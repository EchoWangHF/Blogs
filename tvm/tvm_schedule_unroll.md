# unroll
## 功能说明
unroll是一种常见的循环优化方法，减分支预测失败减少，如果循环体内语句没有数据相关，增加了并发执行的机会，也有利于指令流水线的调度。
### code example :
```python
import tvm

n = 1024
A = tvm.te.placeholder((n, n), name='A')
B = tvm.te.placeholder((n, n), name='B')
C = tvm.te.compute((n, n), lambda i, j: A[i, j] + B[i, j], name='C')

s = tvm.te.create_schedule(C.op)

xo, xi = s[C].split(s[C].op.axis[0], factor=4)

print(tvm.lower(s, [A, B, C], simple_mode=True))
print("---------cutting line---------")

s[C].unroll(xi)

print(tvm.lower(s, [A, B, C], simple_mode=True))
```
### code result :
```python
produce C {
  for (i.outer, 0, 256) {
    for (i.inner, 0, 4) {
      for (j, 0, 1024) {
        C[(((i.outer*4096) + (i.inner*1024)) + j)] = (A[(((i.outer*4096) + (i.inner*1024)) + j)] + B[(((i.outer*4096) + (i.inner*1024)) + j)])
      }
    }
  }
}

---------cutting line---------
produce C {
  for (i.outer, 0, 256) {
    for (j, 0, 1024) {
      C[((i.outer*4096) + j)] = (A[((i.outer*4096) + j)] + B[((i.outer*4096) + j)])
    }
    for (j, 0, 1024) {
      C[(((i.outer*4096) + j) + 1024)] = (A[(((i.outer*4096) + j) + 1024)] + B[(((i.outer*4096) + j) + 1024)])
    }
    for (j, 0, 1024) {
      C[(((i.outer*4096) + j) + 2048)] = (A[(((i.outer*4096) + j) + 2048)] + B[(((i.outer*4096) + j) + 2048)])
    }
    for (j, 0, 1024) {
      C[(((i.outer*4096) + j) + 3072)] = (A[(((i.outer*4096) + j) + 3072)] + B[(((i.outer*4096) + j) + 3072)])
    }
  }
}
```
## 原理说明
unroll的schedule源码比较简单，主要功能在pass当中实现的，pass实现逻辑在`unroll_loop.cc`文档当中。
### schedule code :
```c++
Stage& Stage::unroll(IterVar var) {  // NOLINT(*)
  SetAttrIterType(operator->(), var, kUnrolled);
  return *this;
}
```
### pass code :
```c++
// pass start:
Pass UnrollLoop() {
  auto pass_func = [=](PrimFunc f, IRModule m, PassContext ctx) {
    auto* n = f.CopyOnWrite();
    auto cfg = ctx->GetConfig<UnrollLoopConfig>("tir.UnrollLoop");
    if (!cfg.defined()) {
      cfg = AttrsWithDefaultValues<UnrollLoopConfig>();
    }
    //进入UnrollLoop重载函数的接口：
    n->body = UnrollLoop(std::move(f->body), cfg.value());
    return f;
  };
  return CreatePrimFuncPass(pass_func, 0, "tir.UnrollLoop", {});
}

//UnrollLoop重载函数
Stmt UnrollLoop(Stmt stmt, UnrollLoopConfig cfg) {
  //这里会进入 LoopUnroller 类的构造函数，同时因为重写了继承了 StmtMutator 类，该类对 operator()进行了重写。
  Stmt ret = LoopUnroller(cfg->auto_max_step, cfg->auto_max_depth, cfg->auto_max_extent,
                          cfg->explicit_unroll)(stmt);
  if (!ret.same_as(stmt)) {
    return ConvertSSA(ret);
  } else {
    return ret;
  }
}

// class TVM_DLL StmtMutator ：：
Stmt operator()(Stmt stmt) {
    allow_copy_on_write_ = true;
    return VisitStmt(stmt);
}

//在执行 VisitStmt 的时候，会调用 LoopUnroller 类自己实现的 VisitStmt，最终会调用 Unroll函数 ：
Stmt Unroll(const ForNode* op) {
    int value = GetExtent(op);
    // For loop must have a constant integer extent
    ICHECK_NE(value, -1) << "loop doesn't have a constant integer extent";
    if (value == 0) return Evaluate(0);
    Stmt body = op->body;
    Map<Var, PrimExpr> vmap;
    Array<Stmt> unrolled;
    for (int i = 0; i < value; ++i) {
        vmap.Set(op->loop_var, op->min + make_const(op->loop_var.dtype(), i));
        Stmt step = Substitute(body, vmap);
        unrolled.push_back(step);
    }
    return SeqStmt::Flatten(unrolled);
}
```
上述函数实现的功能比较简单，我们用`gdb`进行入到`Unroll`函数, 看了一下各个步骤的操作：

```c++
//`ForNode *op`:
unrolled (i.inner, 0, 4) {
    for (j, 0, 1024) {
        C[(((i.outer*4096) + (i.inner*1024)) + j)] = (A[(((i.outer*4096) + (i.inner*1024)) + j)] + B[(((i.outer*4096) + (i.inner*1024)) + j)])
    }
}

//`value` : `4`

//`body`:
for (j, 0, 1024) {
    C[(((i.outer*4096) + (i.inner*1024)) + j)] = (A[(((i.outer*4096) + (i.inner*1024)) + j)] + B[(((i.outer*4096) + (i.inner*1024)) + j)])
}
// `op->loop_var` : `i.inner`
```
对比最终的结果：`unroll` pass的功能就是基于`Substitute`函数，实现`i.inner`与`i`值的替换，每替换一次，生成一个`stmt`。


# parallel
## 功能说明
parallel将指定iter的for循环替换为parallel操作，从而在GPU以外的CPU等设备上实现并行。

### code example :
```python
import tvm
n = 1024
m = 1024

A = tvm.placeholder((n, m), name='A')
l = tvm.reduce_axis((0, m), name = 'l')

B = tvm.compute((n,), lambda i: tvm.sum(A[i, l], axis=l), name='B')

s = tvm.create_schedule(B.op)

print(tvm.lower(s, [A, B], simple_mode=True))
print("---------cutting line---------")

s[B].parallel(B.op.reduce_axis[0])
print(tvm.lower(s, [A, B], simple_mode=True))
```
### code result :
```python
produce B {
  for (i, 0, 1024) {
    B[i] = 0f
    for (l, 0, 1024) {
      B[i] = (B[i] + A[((i*1024) + l)])
    }
  }
}

---------cutting line---------
produce B {
  for (i, 0, 1024) {
    B[i] = 0f
    parallel (l, 0, 1024) {
      B[i] = (B[i] + A[((i*1024) + l)])
    }
  }
}
```
## 原理说明

### schedule code :
```c++
Stage& Stage::parallel(IterVar var) {  // NOLINT(*)
  SetAttrIterType(operator->(), var, kParallelized);
  return *this;
}
```
使用`parallel` 会IterType设置为`kParallelized`，在使用中转化为`kParallel`。`kParallel`是`ForNode` type的一种，其他的type比如有:`kUnrolled`和`kSerial`。`kParallel`的作用主要是在`CodeGen`部分作为标记,根据`ForNode` type的种类，生成对应的kernel。
```c++
void CodeGenCPU::VisitStmt_(const ForNode* op) {
  ICHECK(is_zero(op->min));
  if (op->kind == ForKind::kSerial || op->kind == ForKind::kUnrolled) {
    CodeGenLLVM::VisitStmt_(op);
  } else if (op->kind == ForKind::kParallel) {
    // 当ForKind == kParallel，CodeGen会做特殊处理。
    if (parallel_env_.penv == nullptr) {
      CreateParallelLaunch(For(op->loop_var, op->min, op->extent, op->kind, op->body,
                               op->thread_binding, op->annotations),
                           0, std::string("loop_parallel_") + op->loop_var->name_hint.c_str());
    } else {
      // already in parallel env.
      ICHECK(parallel_env_.task_id.defined());
      ICHECK(parallel_env_.num_task.defined());
      ICHECK(parallel_env_.penv != nullptr);
      DataType t = op->extent.dtype();
      PrimExpr num_task = cast(t, parallel_env_.num_task);
      PrimExpr task_id = cast(t, parallel_env_.task_id);
      ICHECK(!parallel_env_.in_parallel_loop)
          << "Nested parallel loop is not supported by threadpool, try fuse them instead";
      parallel_env_.in_parallel_loop = true;
      if (parallel_env_.stride_pattern) {
        CreateSerialFor(MakeValue(task_id), MakeValue(op->extent), MakeValue(num_task),
                        op->loop_var, op->body);
      } else {
        PrimExpr step = (op->extent + num_task - make_const(t, 1)) / num_task;
        PrimExpr begin = min(task_id * step, op->extent);
        PrimExpr end = min((task_id + make_const(t, 1)) * step, op->extent);
        CreateSerialFor(MakeValue(begin), MakeValue(end),
                        llvm::ConstantInt::getSigned(GetLLVMType(end), 1), op->loop_var, op->body);
      }
      parallel_env_.in_parallel_loop = false;
      ++parallel_env_.parallel_loop_count;
    }
  } else {
    LOG(FATAL) << "cannot handle for type " << op->kind;
  }
}
```

# pragma
## 功能说明
添加标签，编译时可以将`InterVal` 替换为标签所对应的原语或者其他，比如`unrool`, `vectorize`.


### schedule code :
```c++
Stage& Stage::pragma(IterVar var, const std::string& pragma_type,
                     const PrimExpr& pragma_value) {  // NOLINT(*)
  if (pragma_type == "unroll") {
    this->unroll(var);
  } else if (pragma_type == "vectorize") {
    this->vectorize(var);
  } else {
    UpdateIterVarAttr(operator->(), var, [pragma_type, pragma_value](IterVarAttrNode* n) {
      n->pragma_keys.push_back(tir::StringImm(pragma_type));
      n->pragma_values.push_back(pragma_value);
    });
  }
  return *this;
}
```
