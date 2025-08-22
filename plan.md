# Cloth Simulator Plan (Rust + wgpu + winit)

## Overview
We will build a GPU-accelerated cloth simulator in Rust using:
- **wgpu** for compute + rendering
- **winit** for windowing and input
- **XPBD (Extended Position-Based Dynamics)** as the simulation model
- **Compute shaders** for physics passes
- **Storage buffers** for particles, constraints, and topology
- **Render pipeline** for previewing directly from GPU buffers

---

## Core Simulation Algorithm

### Simulation Steps (per frame)
1. **Predict positions**  
   - Update velocities with external forces (gravity, wind).
   - Move positions forward in time.

2. **Iterative constraint solving** (XPBD, multiple iterations):
   - Stretch constraints (edges).
   - Bending constraints (triangle adjacency).
   - Attachment constraints (pin fixed vertices).
   - Collision constraints (planes, spheres, SDF).

3. **Update velocities**  
   - Recalculate velocity as `(new_pos - prev_pos) / dt`.

4. **Wind forces**  
   - Compute per-triangle normals.
   - Apply impulses to connected particles.

5. **Normal calculation (optional for rendering)**  
   - Average adjacent triangle normals to build smooth shading.

---

## GPU Data Layout

### Particle Buffer
- `pos.xyz` (predicted position)
- `inv_mass` (0 for pinned, >0 otherwise)
- `prev_pos.xyz` (last step)
- `vel.xyz` (current velocity)

### Constraint Buffers
- **Edges**: pairs of vertex indices, rest length, compliance.  
- **Bends**: quad vertex sets, rest angle or distance, compliance.  
- **Attachments**: vertex indices with fixed positions.  

### Collision Buffers
- **Planes**: normal and distance.  
- **Spheres**: center and radius.  
- (Optional) SDF fields.

### Topology Buffer
- **Triangles**: vertex indices for wind + rendering.  

### Parameters (Uniform Buffer)
- Time step `dt`
- Substeps
- Iteration count
- Gravity vector
- Damping
- Wind direction and strength

---

## GPU Pipelines (WGSL Compute Shaders)

- **Predict**: integrate forces and predict positions.
- **Stretch Solver**: enforce edge lengths with XPBD.
- **Bend Solver**: enforce bending constraints.
- **Collision Solver**: project particles out of colliders.
- **Wind**: apply aerodynamic forces.
- **Velocity Update**: recompute velocities after constraints.
- **Normals (optional)**: per-vertex normals for shading.

Each kernel reads/writes directly to storage buffers.  
Workgroup size: tune around `128–256`.

---

## Rendering

- **Vertex pulling**: read particle positions from storage buffer directly in vertex shader.  
- **Index buffer**: triangles define rendering topology.  
- **Shading**: basic lighting using normals (flat or smoothed).  
- **Wireframe overlay (optional)** for debugging.  

---

## Host (Rust) Responsibilities

### Initialization
- Create window and surface with **winit**.
- Initialize **wgpu::Device** and **Queue**.
- Build compute pipelines for each simulation step.
- Build render pipeline for drawing cloth mesh.
- Allocate and upload initial buffers:
  - Particle grid
  - Edge and bend constraints
  - Triangle indices
  - Collision objects

### Simulation Dispatch
- Encode a command buffer per frame:
  1. Run **predict pass**.
  2. Run **constraint passes** in loop (`iters`).
  3. Run **collision pass**.
  4. Run **wind pass**.
  5. Run **velocity update pass**.
- Then issue a render pass that reads the particle buffer.

### Input/Interaction
- Handle **resize events** with winit.
- Allow toggling **pinned vertices** with mouse clicks.
- Add sliders/controls for parameters (gravity, stiffness, iterations).

---

## Implementation Milestones

1. **MVP**
   - Grid of particles
   - Stretch constraints only
   - Gravity + plane collision
   - Simple shading
   - Runs at 60 FPS

2. **Enhanced Cloth**
   - Add bending constraints
   - Add wind forces
   - Compute normals for smooth shading

3. **Collision System**
   - Support spheres
   - Add simple SDF collisions

4. **User Interaction**
   - Pin/unpin vertices with mouse
   - Adjust parameters at runtime

5. **Optimizations**
   - Profile workgroup sizes
   - Add constraint graph coloring if needed
   - Introduce substeps for stability at larger time steps

---

## Key Techniques
- Use **SoA (structure of arrays)** layout for GPU-friendly memory access.
- Use **Jacobi-style parallel constraint solving** to avoid atomics.
- Use **XPBD compliance** for stable stiffness control independent of dt.
- Use **command encoder with multiple compute passes** per frame.
- Minimize CPU-GPU transfers: keep everything in GPU storage buffers.
- Use **bytemuck + #[repr(C)]** for buffer uploads from Rust.

---

## Debugging and Verification
- Visualize constraint error (color edges by stretch).
- Render normals for shading verification.
- Toggle wireframe rendering for mesh connectivity.
- Clamp edge length ratios to avoid numerical explosion.
- Add NaN guards in shaders.

---

