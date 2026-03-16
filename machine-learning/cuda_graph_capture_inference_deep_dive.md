
# CUDA Graph Capture in Model Inference (Deep Dive)

## 1. Motivation

In GPU inference workloads, a common bottleneck is **kernel launch overhead**.

Traditional execution model:

CPU launches kernels one by one:

CPU → launch kernel → GPU executes
CPU → launch kernel → GPU executes
CPU → launch kernel → GPU executes

For large kernels this overhead is small.

But deep learning inference is different.

Example transformer layer operations:

- LayerNorm
- MatMul
- BiasAdd
- Activation
- ResidualAdd

Each step can become a CUDA kernel.

Example:

50 layers × 6 kernels per layer ≈ **300 kernel launches**

Typical kernel launch overhead:

5–20 microseconds

Total overhead:

300 × 10µs ≈ **3ms launch overhead**

For low-latency systems like:

- search ranking
- recommendation systems
- real-time LLM token generation

this overhead is significant.

---

# 2. CUDA Graph Concept

CUDA Graph turns a sequence of GPU operations into a **static execution graph (DAG)**.

Traditional execution:

CPU launches many kernels individually.

CUDA Graph execution:

CPU launches **one graph**, GPU executes the full sequence internally.

Result:

CPU scheduling overhead reduces from **N launches → 1 launch**.

---

# 3. CUDA Graph Execution Model

CUDA Graph workflow:

Capture → Instantiate → Launch

### Capture

Record GPU operations.

```cpp
cudaStreamBeginCapture(stream);

RunModelKernels();

cudaStreamEndCapture(stream, &graph);
```

During capture CUDA records:

- kernel launches
- memory copies
- memory sets
- synchronization events

These operations form a **Directed Acyclic Graph (DAG)**.

---

### Instantiate

Create executable graph instance.

```cpp
cudaGraphInstantiate(&graphExec, graph);
```

This step:

- validates dependencies
- prepares launch metadata
- builds executable structure

Result:

`cudaGraphExec_t` — reusable executable graph.

---

### Replay

Execute the graph:

```cpp
cudaGraphLaunch(graphExec, stream);
```

GPU executes the whole DAG without CPU scheduling between kernels.

---

# 4. CUDA Graph Internal Structure

CUDA Graph is internally a DAG.

Example:

memcpy  
 ↓  
kernel1  
 ↙   ↘  
kernel2  kernel3  
 ↘   ↙  
kernel4

Node types include:

- CUDA_KERNEL_NODE
- CUDA_MEMCPY_NODE
- CUDA_MEMSET_NODE
- CUDA_EVENT_NODE
- CUDA_HOST_NODE

Dependencies are recorded between nodes.

---

# 5. Capture Modes

CUDA supports three capture modes.

### Global

Most restrictive.

```cpp
cudaStreamCaptureModeGlobal
```

Blocks unsafe CUDA APIs during capture.

### Thread Local

```cpp
cudaStreamCaptureModeThreadLocal
```

Only restricts the current thread.

### Relaxed

```cpp
cudaStreamCaptureModeRelaxed
```

Least restrictive but easier to misuse.

---

# 6. CUDA Graph in Model Inference

Deep learning inference is ideal for CUDA Graph because:

- execution graph is stable
- tensor shapes often fixed
- kernel order deterministic

Typical inference pipeline:

Embedding → Attention → MLP → LayerNorm → Output

Each request executes the **same kernel sequence**, making replay efficient.

---

# 7. Integration in Inference Engines

## TensorRT

Pipeline:

ONNX → TensorRT Engine → CUDA Graph Capture → Replay

TensorRT builds a **static optimized engine**, making CUDA Graph capture very reliable.

---

## ONNX Runtime

CUDA Graph support exists in:

CUDAExecutionProvider

Important files:

- cuda_execution_provider.cc
- cuda_graph.cc

Execution flow:

First Run → Capture Graph
Instantiate GraphExec
Subsequent Runs → Replay

Pseudo logic:

if (!graph_captured)
    capture_graph()
else
    launch_graph()

---

# 8. Constraints

CUDA Graph requires strict execution conditions.

### Static Memory

Tensor addresses must not change between runs.

### Static Shapes

Kernel parameters must remain constant.

### Deterministic Execution

No dynamic control flow such as:

- Loop
- If
- Scan

### Restricted APIs

Operations like:

cudaMalloc  
cudaFree  

are not allowed during capture.

---

# 9. Why CUDA Graph Fails in ONNX Runtime

Typical error:

This session cannot use the graph capture feature
as the model has control flow nodes

Reason:

ONNX graph still contains control flow operators.

TensorRT often removes these during engine optimization, but ONNX Runtime may execute them directly.

Result:

TensorRT capture succeeds while ORT capture fails.

---

# 10. Performance Gains

Typical improvements:

ResNet: 5–10%  
BERT: 10–20%  
GPT inference: 20–30%

Benefits come from:

- reduced kernel launch overhead
- reduced CPU scheduling
- improved GPU utilization

---

# 11. Debugging CUDA Graph

Useful techniques:

CUDA_LAUNCH_BLOCKING=1

ORT_LOG_SEVERITY_LEVEL=0

cudaGraphDebugDotPrint(graph)

This produces a `.dot` file for visualization.

---

# 12. When NOT to Use CUDA Graph

Not suitable for:

- dynamic batch size
- dynamic sequence length
- control-flow heavy models
- variable memory allocation

Examples:

- dynamic LLM batching
- Mixture-of-Experts routing

---

# 13. Key Takeaways

CUDA Graph improves inference by:

- reducing CPU overhead
- improving GPU utilization
- stabilizing latency

Best used when:

- execution graph is static
- tensor shapes fixed
- memory layout stable

Ideal for optimized TensorRT engines.
