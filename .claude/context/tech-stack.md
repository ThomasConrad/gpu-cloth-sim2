# Technology Stack & Dependencies

## Core Technologies

### Rust Ecosystem
- **Language**: Rust 2021 edition for memory safety and performance
- **Graphics**: wgpu for GPU compute and rendering abstraction
- **Windowing**: winit for cross-platform window management and input
- **Math**: glam for GPU-friendly linear algebra (SIMD optimized)
- **Memory**: bytemuck for safe buffer layout and transfers

### GPU Programming
- **Shading Language**: WGSL (WebGPU Shading Language)
- **Compute Architecture**: Storage buffers with compute shader dispatches
- **Rendering**: Vertex pulling from compute buffer outputs
- **API**: WebGPU standard via wgpu-rs bindings

## Architecture Patterns

### GPU-Centric Design
- **Storage Buffers**: Primary data containers for particles, constraints
- **Compute Pipelines**: Separate shaders for each physics pass
- **Command Encoding**: Multi-pass per-frame command buffer construction
- **Zero-Copy Rendering**: Direct visualization from physics buffers

### Data Layout Optimization
- **Structure of Arrays (SoA)**: GPU-friendly memory access patterns
- **Alignment**: Proper buffer alignment for GPU memory coalescing
- **Repr(C)**: C-compatible struct layout for Rust-GPU interop

### Parallel Computing Patterns
- **Jacobi Iteration**: Parallel constraint solving without race conditions
- **Workgroup Sizing**: Optimized dispatch sizes (128-256 threads)
- **Memory Access**: Coalesced reads/writes for maximum bandwidth

## Development Tools

### Build System
- **Cargo**: Standard Rust package manager and build system
- **Features**: Conditional compilation for debug/release optimizations

### Debugging & Profiling
- **wgpu-rs**: Built-in validation layers for GPU debugging
- **Renderdoc**: Graphics debugging and profiling
- **Tracy**: CPU/GPU profiling integration

### Development Environment
- **IDE**: Any Rust-compatible editor with LSP support
- **Version Control**: Git with standard Rust .gitignore patterns
- **CI/CD**: GitHub Actions for cross-platform testing

## Platform Support

### Target Platforms
- **Primary**: Desktop (Windows, macOS, Linux)
- **Secondary**: Web via WebAssembly + WebGPU
- **Future**: Mobile via native wgpu backends

### GPU Requirements
- **Minimum**: Compute shader support (GL 4.3 / D3D11 / Vulkan 1.0)
- **Recommended**: Modern GPU with high memory bandwidth
- **Storage**: Support for large storage buffers (>100MB)

## Performance Considerations

### Memory Management
- **Buffer Reuse**: Persistent GPU buffers across frames
- **Staging Buffers**: Efficient CPU-GPU data transfers
- **Memory Pools**: Pre-allocated buffer management

### Compute Optimization
- **Occupancy**: Maximize GPU utilization with proper workgroup sizes
- **Memory Bandwidth**: Optimize data layout for cache efficiency
- **Instruction Throughput**: Minimize branching in compute shaders

### Rendering Efficiency
- **Instancing**: Batch rendering operations
- **Level of Detail**: Distance-based mesh simplification
- **Culling**: Frustum and occlusion culling for large scenes