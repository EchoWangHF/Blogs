# tf2xla
相关代码走读，以tf2.10分支为基础，彼时openxla已经发布，正在摸索源代码，后续与openxla变动较大的地方会更新。

## 1. Graph Ir to HloModuleProto
graph 从 `tensorflow/core/common_runtime/graph_execution_state.cc`文件的 `GraphExecutionState::InitBaseGraph` 函数开始，会经过一些列的pass优化，这些pass有的是tf 框架的pass，也有一些是xla jit的pass。

xla对graph ir的pass主要是跟atuo clustering 相关的，注册入口在`jit/jit_compilation_pass_registration.cc`，后续可以抽时间深读一下。

运行pass的入口为`tensorflow/core/common_runtime/optimization_registry.cc`文件的`OptimizationPassRegistry::RunGrouping`

tf graph op 到 xla op的编译运行流程可以从下面这个函数开始了解 ：
``` c++
// loc: jit/xla_compile_on_demand_op.cc
void XlaCompileOnDemandOp::Compute(OpKernelContext* ctx) {
  const XlaCompiler::CompilationResult* result;
  xla::LocalExecutable* executable;
  ResourceVarsSnapshot variable_args;
  XlaCompilationCache* cache;
  OP_REQUIRES(ctx, ctx->function_library(),
              errors::Internal("Function library missing"));
  // VLOG(0) << "@@@@@@@@ Start Compile";
  OP_REQUIRES_OK(ctx,
                 Compile(ctx, &result, &cache, &variable_args, &executable));

  // Hold the reference to the JIT during evaluation. (We could probably
  // free it sooner because the ResourceMgr will retain a reference, but
  // this is more obviously correct.)
  core::ScopedUnref cache_ref(cache);
  // VLOG(0) << "@@@@@@@@ Start Run";
  OP_REQUIRES_OK(ctx, Run(ctx, cache, result, executable, variable_args));
}
// #### class XlaCompileOnDemandOp : public OpKernel
```
有几个比较重要的数据结构可以关注一下：

`using CompilationResult = ::tensorflow::XlaCompilationResult; `

`xla::LocalExecutable`

`XlaCompilationCache`



