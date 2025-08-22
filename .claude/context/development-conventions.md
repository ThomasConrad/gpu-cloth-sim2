# Development Conventions & Code Standards

## Rust Code Style Guidelines

### Project Structure
```
src/
├── main.rs              // Application entry point
├── lib.rs               // Library root with public API
├── app/                 // Application layer
│   ├── mod.rs
│   ├── window.rs        // Window management and event handling
│   ├── renderer.rs      // High-level rendering coordination
│   └── ui.rs            // User interface and interaction
├── physics/             // Physics simulation core
│   ├── mod.rs
│   ├── particle.rs      // Particle system and data structures
│   ├── constraints.rs   // Constraint types and solving
│   ├── collision.rs     // Collision detection and response
│   └── integrator.rs    // Time integration and force application
├── gpu/                 // GPU abstraction and utilities
│   ├── mod.rs
│   ├── device.rs        // wgpu device and queue management
│   ├── buffers.rs       // Buffer creation and management
│   ├── compute.rs       // Compute pipeline abstractions
│   └── shaders.rs       // Shader loading and compilation
├── math/                // Mathematical utilities
│   ├── mod.rs
│   └── linalg.rs        // Linear algebra extensions
└── utils/               // General utilities
    ├── mod.rs
    ├── profiler.rs      // Performance measurement
    └── config.rs        // Configuration management
```

### Naming Conventions

#### Variables and Functions
```rust
// Use snake_case for variables and functions
let particle_count = 1000;
let max_iterations = 10;

fn update_particle_positions(dt: f32) { }
fn solve_distance_constraints() { }
```

#### Types and Structs
```rust
// Use PascalCase for types
struct ParticleSystem { }
struct ClothSimulator { }
enum ConstraintType { }

// Use descriptive names that clarify purpose
struct DistanceConstraint {
    particle_a: u32,
    particle_b: u32,
    rest_length: f32,
}
```

#### Constants and Statics
```rust
// Use SCREAMING_SNAKE_CASE for constants
const DEFAULT_GRAVITY: Vec3 = Vec3::new(0.0, -9.81, 0.0);
const MAX_SOLVER_ITERATIONS: u32 = 10;
const PARTICLE_BUFFER_SIZE: u64 = 1024 * 1024;
```

### Memory Management Patterns

#### Buffer Ownership
```rust
// GPU buffers owned by device abstraction layer
pub struct GpuDevice {
    particle_buffer: wgpu::Buffer,
    constraint_buffer: wgpu::Buffer,
}

// Use explicit lifetime parameters when borrowing
impl<'a> PhysicsSimulator<'a> {
    pub fn new(device: &'a GpuDevice) -> Self { }
}
```

#### Error Handling
```rust
// Use Result<T, E> for fallible operations
pub fn create_compute_pipeline(
    device: &wgpu::Device
) -> Result<wgpu::ComputePipeline, PipelineError> { }

// Use expect() with descriptive messages for initialization
let device = adapter.request_device(&desc, None)
    .await
    .expect("Failed to create wgpu device");
```

### GPU Programming Patterns

#### Compute Shader Organization
```rust
// Group related compute operations
pub struct ClothComputePipelines {
    predict_positions: wgpu::ComputePipeline,
    solve_constraints: wgpu::ComputePipeline,
    update_velocities: wgpu::ComputePipeline,
    compute_normals: wgpu::ComputePipeline,
}

// Use builder pattern for complex pipeline creation
impl ClothComputePipelines {
    pub fn builder() -> ComputePipelineBuilder {
        ComputePipelineBuilder::new()
    }
}
```

#### Buffer Management
```rust
// Use type-safe buffer wrappers
pub struct TypedBuffer<T> {
    buffer: wgpu::Buffer,
    len: usize,
    _phantom: PhantomData<T>,
}

// Implement safe buffer operations
impl<T> TypedBuffer<T> 
where
    T: bytemuck::Pod + bytemuck::Zeroable,
{
    pub fn write(&self, queue: &wgpu::Queue, data: &[T]) { }
    pub fn len(&self) -> usize { self.len }
}
```

## WGSL Shader Conventions

### File Organization
```
shaders/
├── compute/
│   ├── predict_positions.wgsl
│   ├── solve_stretch.wgsl
│   ├── solve_bending.wgsl
│   ├── apply_wind.wgsl
│   └── update_velocities.wgsl
├── render/
│   ├── cloth_vertex.wgsl
│   ├── cloth_fragment.wgsl
│   └── wireframe.wgsl
└── common/
    ├── types.wgsl           // Shared struct definitions
    ├── constants.wgsl       // Shader constants
    └── math.wgsl           // Mathematical functions
```

### Shader Code Style
```wgsl
// Use consistent naming with Rust host code
struct Particle {
    pos: vec3<f32>,
    prev_pos: vec3<f32>,
    vel: vec3<f32>,
    inv_mass: f32,
}

// Group related bindings
@group(0) @binding(0) var<storage, read_write> particles: array<Particle>;
@group(0) @binding(1) var<storage, read> constraints: array<DistanceConstraint>;
@group(0) @binding(2) var<uniform> sim_params: SimulationParams;

// Use descriptive workgroup sizes
@compute @workgroup_size(256)
fn predict_positions(@builtin(global_invocation_id) global_id: vec3<u32>) {
    let particle_id = global_id.x;
    if (particle_id >= arrayLength(&particles)) {
        return;
    }
    
    // Physics computation...
}
```

### Buffer Layout Consistency
```rust
// Ensure Rust and WGSL struct layouts match exactly
#[repr(C)]
#[derive(Copy, Clone, Debug, bytemuck::Pod, bytemuck::Zeroable)]
pub struct Particle {
    pub pos: [f32; 3],
    pub _pad1: f32,      // Explicit padding for alignment
    pub prev_pos: [f32; 3],
    pub _pad2: f32,
    pub vel: [f32; 3],
    pub inv_mass: f32,
}
```

## Performance Guidelines

### GPU Performance Patterns
- **Memory Coalescing**: Access memory in patterns that align with GPU architecture
- **Occupancy**: Size workgroups to maximize GPU utilization (typically 64-256)
- **Bank Conflicts**: Avoid shared memory bank conflicts in complex algorithms
- **Divergent Branching**: Minimize conditional execution within workgroups

### CPU Performance Patterns
```rust
// Use const generics for compile-time optimization
impl<const N: usize> ParticleGrid<N> {
    pub fn new() -> Self { }
}

// Prefer Vec<T> over Vec<Box<T>> for cache efficiency
struct PhysicsState {
    particles: Vec<Particle>,        // Good: contiguous memory
    // constraints: Vec<Box<Constraint>>,  // Avoid: pointer indirection
}
```

### Memory Usage Guidelines
- **Buffer Reuse**: Recycle GPU buffers across frames
- **Pool Allocation**: Use object pools for frequently allocated/deallocated objects
- **Stack Allocation**: Prefer stack-allocated arrays for small, fixed-size data

## Documentation Standards

### API Documentation
```rust
/// Simulates cloth physics using XPBD (Extended Position-Based Dynamics).
/// 
/// This simulator implements a parallel constraint solver that runs entirely
/// on the GPU using compute shaders. The simulation maintains stability
/// across a wide range of timesteps through compliance-based constraints.
/// 
/// # Examples
/// 
/// ```rust
/// let mut simulator = ClothSimulator::new(&device, 50, 50)?;
/// simulator.set_gravity(Vec3::new(0.0, -9.81, 0.0));
/// 
/// // Run simulation step
/// simulator.step(&queue, 0.016)?;
/// ```
pub struct ClothSimulator { }
```

### Code Comments
```rust
// Physics integration: predict positions using current velocities
// and external forces (gravity, wind, user interaction)
for particle in &mut self.particles {
    if particle.inv_mass > 0.0 {
        // Apply forces: F = ma, so a = F * inv_mass
        let acceleration = self.gravity * particle.inv_mass;
        particle.vel += acceleration * dt;
        
        // Semi-implicit Euler: update position with new velocity
        particle.pos += particle.vel * dt;
    }
}
```

### Commit Message Format
```
feat: implement XPBD constraint solver with compliance model

- Add distance constraint projection with configurable stiffness
- Implement iterative Jacobi solver for parallel GPU execution
- Support both equality and inequality constraints
- Maintain numerical stability across wide timestep range

Closes #12
```

## Testing Conventions

### Unit Test Organization
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_particle_integration() {
        // Test basic Verlet integration accuracy
    }
    
    #[test]
    fn test_constraint_projection() {
        // Test XPBD constraint solving
    }
}
```

### Integration Test Patterns
- **GPU Tests**: Verify compute shader behavior with known inputs
- **Performance Tests**: Benchmark key operations and detect regression
- **Visual Tests**: Compare rendered output against reference images
- **Cross-Platform Tests**: Ensure consistent behavior across GPU vendors

## Build and Development Tools

### Cargo Configuration
```toml
[dependencies]
wgpu = "0.18"
winit = "0.29"
glam = { version = "0.24", features = ["bytemuck"] }
bytemuck = { version = "1.0", features = ["derive"] }

[dev-dependencies]
approx = "0.5"      # Floating point comparison in tests
criterion = "0.5"   // Benchmarking framework

[profile.dev]
opt-level = 1       # Reasonable performance for development

[profile.release]
lto = true          # Link-time optimization for maximum performance
```

### Development Workflow
1. **Feature Branch**: Create branch from main for each new feature
2. **Incremental Commits**: Make small, focused commits with clear messages
3. **Performance Check**: Run benchmarks before merging performance-sensitive changes
4. **Documentation Update**: Update API docs and comments with implementation changes
5. **Clean Merge**: Squash commit history for clean main branch log