# 01 XLA compile

ut 调用bt栈：

#0  xla::mlu::IrEmitterUnnested::CalculateReductionLaunchInfo (this=0x555555ac4080, mlu_device_info=..., input_shape=..., fused_computation=0x555555b55d50, data_alignment=..., 
    reduce_launch_info=...) at tensorflow/compiler/xla/service/mlu/ir_emitter_unnested.cc:7688
#1  0x00007ffff0798d98 in xla::mlu::IrEmitterUnnested::EmitMLUReduction (this=0x555555ac4080, mlir_input=...) at tensorflow/compiler/xla/service/mlu/ir_emitter_unnested.cc:8527
#2  0x00007ffff079b03f in xla::mlu::IrEmitterUnnested::EmitReductionFromOrToContiguousDimensions (this=0x555555ac4080, mlir_input=...)
    at tensorflow/compiler/xla/service/mlu/ir_emitter_unnested.cc:8761
#3  0x00007ffff075d69a in xla::mlu::IrEmitterUnnested::EmitFusionFromMlir (this=0x555555ac4080, mlir_input=...) at tensorflow/compiler/xla/service/mlu/ir_emitter_unnested.cc:2897
#4  0x00007ffff079f9ba in xla::mlu::IrEmitterUnnested::EmitOp (this=0x555555ac4080, mlir_input=...) at tensorflow/compiler/xla/service/mlu/ir_emitter_unnested.cc:9276
#5  0x00007ffff07a0a07 in xla::mlu::IrEmitterUnnested::EmitLmhloRegion (this=0x555555ac4080, region=0x555555b10c90) at tensorflow/compiler/xla/service/mlu/ir_emitter_unnested.cc:9368
#6  0x00007ffff1a8a952 in xla::mlu::CompileModuleToLlvmIrImpl (hlo_module=0x555555a82dc0, llvm_context=0x7fffffffd110, target_triple="mlisa-cambricon-bang", 
    data_layout="e-p:64:64:64-i64:64-v16:16-v32:32-n16:32:64", platform_name="MLU", mlu_device_info=..., bang_compute_capability=..., can_share_buffer_function=..., pointer_size=8, 
    profile_index_map=0x0, llvm_module=0x7fffffffd128, buffer_assignment=0x7fffffffd130, thunk_schedule=0x7fffffffd138, constants=0x7fffffffd180)
    at tensorflow/compiler/xla/service/mlu/mlu_compiler.cc:772
#7  0x00007ffff1a8dab3 in xla::mlu::MluCompiler::RunBackend (this=0x555555a25c60, module=std::unique_ptr<xla::HloModule> = {...}, stream_exec=0x7fff94000e30, options=...)
    at tensorflow/compiler/xla/service/mlu/mlu_compiler.cc:1074
#8  0x00007ffff2883eaf in xla::Compiler::RunBackend (this=0x555555a25c60, module=std::unique_ptr<xla::HloModule> = {...}, executor=0x7fff94000e30, device_allocator=0x555555aa49f0)
    at ./tensorflow/compiler/xla/service/compiler.h:217
#9  0x00007ffff287cab3 in xla::HloRunner::CreateExecutable (this=0x555555a25510, module=std::unique_ptr<xla::HloModule> = {...}, run_hlo_passes=false)
    at tensorflow/compiler/xla/service/hlo_runner.cc:446
#10 0x00007ffff287913b in xla::HloRunner::ExecuteWithDeviceBuffers (this=0x555555a25510, module=std::unique_ptr<xla::HloModule> = {...}, arguments=..., run_hlo_passes=false, profile=0x0)
    at tensorflow/compiler/xla/service/hlo_runner.cc:160
#11 0x00007ffff28788f9 in xla::HloRunner::Execute (this=0x555555a25510, module=std::unique_ptr<xla::HloModule> = {...}, arguments=..., run_hlo_passes=false, profile=0x0)
    at tensorflow/compiler/xla/service/hlo_runner.cc:103
#12 0x00007ffff29fbf06 in xla::HloRunnerInterface::Execute (this=0x555555a25510, module=std::unique_ptr<xla::HloModule> = {...}, arguments=..., run_hlo_passes=false)
    at ./tensorflow/compiler/xla/service/hlo_runner_interface.h:124
#13 0x00007ffff29f3399 in xla::HloTestBase::RunAndCompareInternal(std::unique_ptr<xla::HloModule, std::default_delete<xla::HloModule> >, absl::lts_2020_09_23::Span<xla::Literal* const>, absl::lts_2020_09_23::optional<xla::ErrorSpec> const&, bool, std::function<void (xla::HloModule*)> const&) (this=0x555555a25500, module=std::unique_ptr<xla::HloModule> = {...}, arguments=..., 
    error=..., run_hlo_passes=false, reference_preprocessor=...) at tensorflow/compiler/xla/tests/hlo_test_base.cc:277
#14 0x00007ffff29f38d1 in xla::HloTestBase::RunAndCompareNoHloPasses(std::unique_ptr<xla::HloModule, std::default_delete<xla::HloModule> >, absl::lts_2020_09_23::Span<xla::Literal* const>, absl::lts_2020_09_23::optional<xla::ErrorSpec> const&, std::function<void (xla::HloModule*)> const&) (this=0x555555a25500, module=std::unique_ptr<xla::HloModule> = {...}, arguments=..., 
    error=..., reference_preprocessor=...) at tensorflow/compiler/xla/tests/hlo_test_base.cc:308
#15 0x00007ffff29f3c75 in xla::HloTestBase::RunAndCompareNoHloPasses(std::unique_ptr<xla::HloModule, std::default_delete<xla::HloModule> >, absl::lts_2020_09_23::optional<xla::ErrorSpec> const&, std::function<void (xla::HloModule*)> const&) (this=0x555555a25500, module=std::unique_ptr<xla::HloModule> = {...}, error=..., reference_preprocessor=...)
    at tensorflow/compiler/xla/tests/hlo_test_base.cc:340
#16 0x000055555562a699 in xla::(anonymous namespace)::ReduceFusionTest_module_0003_fused_computation_193_Test::TestBody (this=0x555555a25500)
    at tensorflow/compiler/xla/service/mlu/tests/reduce_fusion_row_vectorization_test.cc:1910
#17 0x00007ffff21abeaa in testing::internal::HandleSehExceptionsInMethodIfSupported<testing::Test, void> (object=0x555555a25500, method=&virtual testing::Test::TestBody(), 
    location=0x7ffff21bbf13 "the test body") at external/com_google_googletest/googletest/src/gtest.cc:2424
#18 0x00007ffff21a781c in testing::internal::HandleExceptionsInMethodIfSupported<testing::Test, void> (object=0x555555a25500, method=&virtual testing::Test::TestBody(), 
    location=0x7ffff21bbf13 "the test body") at external/com_google_googletest/googletest/src/gtest.cc:2460
#19 0x00007ffff2193cd4 in testing::Test::Run (this=0x555555a25500) at external/com_google_googletest/googletest/src/gtest.cc:2499
#20 0x00007ffff2194553 in testing::TestInfo::Run (this=0x555555a16660) at external/com_google_googletest/googletest/src/gtest.cc:2675
#21 0x00007ffff2194bed in testing::TestSuite::Run (this=0x55555573f710) at external/com_google_googletest/googletest/src/gtest.cc:2803
#22 0x00007ffff219ecdd in testing::internal::UnitTestImpl::RunAllTests (this=0x555555a16730) at external/com_google_googletest/googletest/src/gtest.cc:5241
#23 0x00007ffff21acf2d in testing::internal::HandleSehExceptionsInMethodIfSupported<testing::internal::UnitTestImpl, bool> (object=0x555555a16730

## 1 RunBackend函数被调用的地方：
	tensorflow\compiler\xla\service\service.cc StatusOr<std::vector<std::unique_ptr<Executable>>> Service::BuildExecutables函数

    StatusOr<std::unique_ptr<Executable>> Service::BuildExecutable函数
## 2 BuildExecutables函数走读分析：
### 2.1 输入的重要参数：onst std::vector<const HloModuleProto*>& module_protos
重点看一下HloModuleProto的数据结构(正确性存疑)
``` c++
// Serialization of HloModule.
message HloModuleProto {
    string name = 1;
    string entry_computation_name = 2;
    int64 entry_computation_id = 6;
    // The array of computations is always in a valid dependency order, where
    // callees appear before their callers.
    repeated HloComputationProto computations = 3;
    // The host program shape (with layout) of the entry computation.
    xla.ProgramShapeProto host_program_shape = 4;
    // The id of this module.
    int64 id = 5;
    // The schedule for this module.
    HloScheduleProto schedule = 7;
    // Describes alias information between inputs and outputs.
    HloInputOutputAliasProto input_output_alias = 8;
    DynamicParameterBindingProto dynamic_parameter_binding = 9;
    repeated CrossProgramPrefetch cross_program_prefetches = 10;
    // True if the module contains dynamic computation.
    bool is_dynamic = 11;
}
```
		
### 2.2 Dump module相关信息：
	
### 2.3 递归从proto文件构建module：
``` c++
TF_ASSIGN_OR_RETURN( auto module, CreateModuleFromProto(*proto, config, run_backend_only));
```
进入到`CreateModuleFromProto`函数当中观察一下, 核心函数：
```c
  TF_ASSIGN_OR_RETURN(std::unique_ptr<HloModule> module,
                      HloModule::CreateFromProto(proto, module_config));
```
核心函数是`CreateFromProto`, 在`HloModule`, `HloComputation`, `HloInstruction` 三级结构体都有实现，逐级递归调用。

在`HloModule`会遍历module内所有的computation，然后对每一个computation调用`HloComputation::CreateFromProto`，具体代码如下：
``` c++
  absl::flat_hash_map<int64, HloComputation*> computation_map;
  absl::flat_hash_map<HloComputation*, int64> to_proto_id;
  std::vector<std::unique_ptr<HloComputation>> computations;
  HloComputation* entry = nullptr;
  for (const HloComputationProto& computation_proto : proto.computations()) {
    TF_ASSIGN_OR_RETURN(
        std::unique_ptr<HloComputation> computation,
        HloComputation::CreateFromProto(computation_proto, computation_map,
                                        prohibit_empty_literal));
    CHECK_NE(computation.get(), nullptr);
    int64 computation_id = computation_proto.id();
    TF_RET_CHECK(computation_id != -1);
    TF_RET_CHECK(!ContainsKey(computation_map, computation_id));
    computation_map[computation_id] = computation.get();
    to_proto_id[computation.get()] = computation_id;
    if (computation_id == proto.entry_computation_id()) {
      entry = computation.get();
    }
    computations.push_back(std::move(computation));
  }
  TF_RET_CHECK(entry != nullptr);
```



### 2.4 调用各种backend实例化的compiler的Compile函数
``` c++
std::vector<std::unique_ptr<Executable>> executables;
if (!run_backend_only) {
TF_ASSIGN_OR_RETURN(executables, backend->compiler()->Compile(
                                        std::move(module_group),
                                        std::move(executors), options));
} else {
auto modules = module_group->ConsumeModules();
for (std::unique_ptr<HloModule>& module : modules) {
    TF_ASSIGN_OR_RETURN(std::unique_ptr<Executable> executable,
                        backend->compiler()->RunBackend(
                            std::move(module), executors[0][0], options));
    executables.push_back(std::move(executable));
}
```
### 2.5 暂时不知道具体作用，需要细看一下代码。
``` c++
for (size_t i = 0; i < module_protos.size(); ++i) {
    const auto& debug_opts = module_configs[i]->debug_options();
    if (DumpingEnabledForHloModule(module_protos[i]->name(), debug_opts) &&
        debug_opts.xla_dump_hlo_snapshots()) {
        executables[i]->set_hlo_proto(std::move(hlo_protos[i]));
    }
}
```
		
		
	
## 3 tensorflow\compiler\xla\service\hlo_runner.cc StatusOr<std::unique_ptr<Executable>> HloRunner::CreateExecutable 函数
