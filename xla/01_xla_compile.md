# 01 XLA compile
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
