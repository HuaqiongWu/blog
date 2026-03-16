
# EnableCudaGraphCapture Implementation Analysis (ONNX Runtime)

## Overview

ONNX Runtime provides optional support for **CUDA Graph capture** to reduce GPU kernel launch overhead during model inference.

This functionality is mainly implemented inside the CUDA Execution Provider.

Key source files:

- cuda_execution_provider.cc
- cuda_graph.cc
- cuda_graph.h

The feature is enabled through the provider option:

```
enable_cuda_graph = 1
```

When enabled, ONNX Runtime will attempt to:

1. Capture the GPU execution graph during the first inference run
2. Instantiate a CUDA Graph executable
3. Replay the graph for subsequent runs

---

# 1. High-Level Execution Flow

When a model session runs:

```
Session::Run()
   ↓
ExecutionProvider::Compute()
   ↓
CUDAExecutionProvider
   ↓
(optional) CUDA Graph Capture
```

Simplified logic:

```
if (enable_cuda_graph)
{
    if (!graph_captured)
        CaptureGraph();
    else
        ReplayGraph();
}
else
{
    NormalExecution();
}
```

---

# 2. Provider Option

The feature is enabled through CUDAExecutionProvider options.

Example (C++ API):

```cpp
OrtCUDAProviderOptions options;
options.enable_cuda_graph = 1;

session_options.AppendExecutionProvider_CUDA(options);
```

Internally this value is stored inside:

```
CUDAExecutionProviderInfo
```

During provider initialization.

---

# 3. Capture Trigger

Graph capture normally occurs during the **first Run() call**.

Pseudo flow inside CUDAExecutionProvider:

```
Run()
{
    if (cuda_graph_enabled && !graph_initialized)
    {
        BeginCapture();
        ExecuteGraph();
        EndCapture();
        InstantiateGraph();
    }
    else if (graph_initialized)
    {
        LaunchCapturedGraph();
    }
}
```

The first execution both runs the model **and records the CUDA graph**.

---

# 4. Capture Implementation

Capture begins using CUDA Stream capture APIs.

```cpp
cudaStreamBeginCapture(stream, cudaStreamCaptureModeGlobal);
```

During capture, all GPU operations are recorded:

- kernel launches
- memory copies
- memory sets
- synchronization events

The recorded operations form a **CUDA Graph DAG**.

After execution completes:

```cpp
cudaStreamEndCapture(stream, &graph);
```

The result is:

```
cudaGraph_t graph
```

---

# 5. Graph Instantiation

After capture, ONNX Runtime instantiates the executable graph:

```cpp
cudaGraphInstantiate(&graph_exec, graph);
```

The executable graph:

```
cudaGraphExec_t
```

contains optimized launch metadata.

This object is cached inside the execution provider.

---

# 6. Graph Replay

Subsequent inference calls skip normal execution and replay the graph:

```cpp
cudaGraphLaunch(graph_exec, stream);
```

This reduces CPU overhead because:

```
many kernel launches → one graph launch
```

---

# 7. Graph Lifetime Management

ONNX Runtime stores graph objects inside a structure similar to:

```
CudaGraphState
{
    cudaGraph_t graph;
    cudaGraphExec_t graph_exec;
    bool graph_captured;
}
```

Lifecycle:

```
capture → instantiate → replay
```

Graph is destroyed when the execution provider shuts down.

---

# 8. Memory Address Constraint

CUDA Graph replay requires **stable memory addresses**.

Therefore ONNX Runtime must guarantee:

- input tensor memory pointers remain stable
- output tensor memory pointers remain stable

If tensor allocation changes between runs, the graph becomes invalid.

This is one reason CUDA Graph only works with **static memory planning**.

---

# 9. Why CUDA Graph May Fail

ONNX Runtime disables CUDA Graph if certain operators are detected.

Example error:

```
This session cannot use the graph capture feature
as the model has control flow nodes which can't be supported
```

Unsupported operators include:

- Loop
- If
- Scan

These introduce **dynamic execution paths**.

CUDA Graph requires deterministic execution.

---

# 10. Interaction With TensorRT Execution Provider

TensorRT EP behaves differently.

TensorRT workflow:

```
ONNX → TensorRT Engine → CUDA Graph
```

TensorRT converts the graph into a **static execution engine**.

ONNX Runtime may still contain:

- dynamic operators
- CPU fallback nodes

This prevents full graph capture.

Therefore:

```
trtexec may support CUDA Graph
but ONNX Runtime TensorRT EP may not
```

---

# 11. Debugging CUDA Graph in ORT

Useful debugging options:

Enable verbose logging:

```
ORT_LOG_SEVERITY_LEVEL=0
```

Check CUDA errors:

```
CUDA_LAUNCH_BLOCKING=1
```

Inspect capture failure points in:

```
cuda_execution_provider.cc
```

---

# 12. Performance Impact

Typical improvements depend on model characteristics.

Models with many small kernels benefit most:

Example improvements:

| Model | Latency Improvement |
|------|---------------------|
| ResNet | 5–10% |
| BERT | 10–20% |
| GPT inference | 20–30% |

Performance gain comes from:

- fewer kernel launches
- reduced CPU scheduling
- improved GPU utilization

---

# 13. Key Takeaways

CUDA Graph capture in ONNX Runtime:

- records GPU kernel execution once
- instantiates a reusable CUDA graph
- replays the graph for future runs

Advantages:

- reduced CPU overhead
- lower latency
- more stable inference performance

Limitations:

- requires static shapes
- requires stable memory addresses
- cannot support dynamic control flow
