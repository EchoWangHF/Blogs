# tf2xla
相关代码走读，以tf2.10分支为基础，彼时openxla已经发布，正在摸索源代码，后续与openxla变动较大的地方会更新。

## 1. Graph Ir to HloModuleProto
graph 从 `tensorflow/core/common_runtime/graph_execution_state.cc`文件的 `GraphExecutionState::InitBaseGraph` 函数开始，会经过一些列的pass优化，这些pass有的是tf 框架的pass，也有一些是xla jit的pass。

xla对graph ir的pass主要是跟atuo clustering 相关的，注册入口在`jit/jit_compilation_pass_registration.cc`，后续可以抽时间深读一下。

运行pass的入口为`tensorflow/core/common_runtime/optimization_registry.cc`文件的`OptimizationPassRegistry::RunGrouping`


