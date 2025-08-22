# Physics Architecture & Algorithms

## XPBD (Extended Position-Based Dynamics) Core

### Simulation Loop Overview
```
Per Frame:
1. Predict Positions    → Apply forces, integrate velocity
2. Constraint Solving   → Iterative constraint projection (5-10 iterations)
3. Velocity Update      → Derive velocities from position changes
4. Additional Forces    → Wind, user interaction
5. Collision Response   → Project out of collision primitives
```

### Mathematical Foundation
- **Position-Based**: Primary state is particle position, velocity derived
- **Constraint Projection**: Iterative relaxation of constraint violations
- **Compliance Model**: Soft constraints via compliance parameters
- **Stability**: Unconditionally stable for any timestep

## Data Structures

### Particle System
```rust
struct Particle {
    pos: Vec3,           // Current predicted position
    prev_pos: Vec3,      // Previous frame position
    vel: Vec3,           // Current velocity (derived)
    inv_mass: f32,       // Inverse mass (0 = pinned)
}
```

### Constraint Types

#### Stretch Constraints (Distance)
```rust
struct StretchConstraint {
    particle_a: u32,     // First particle index
    particle_b: u32,     // Second particle index
    rest_length: f32,    // Target distance
    compliance: f32,     // Stiffness control [0,∞)
}
```

#### Bending Constraints (Angle)
```rust
struct BendConstraint {
    particles: [u32; 4], // Quad of vertices forming two triangles
    rest_angle: f32,     // Target dihedral angle
    compliance: f32,     // Bending stiffness
}
```

## GPU Compute Pipeline Architecture

### Compute Shader Organization
1. **predict_positions.wgsl** - Force integration and position prediction
2. **solve_stretch.wgsl** - Distance constraint projection
3. **solve_bending.wgsl** - Bending constraint projection  
4. **solve_collisions.wgsl** - Collision constraint projection
5. **apply_wind.wgsl** - Aerodynamic force computation
6. **update_velocities.wgsl** - Velocity derivation from positions
7. **compute_normals.wgsl** - Vertex normal calculation for rendering

### Buffer Layout Strategy
```
Storage Buffers:
├── particles_buffer      // Main particle state
├── constraints_stretch   // Distance constraints
├── constraints_bend      // Bending constraints
├── triangle_indices      // Mesh topology for rendering/wind
├── collision_planes      // Static collision geometry
├── collision_spheres     // Dynamic collision objects
└── simulation_params     // Per-frame parameters (uniforms)
```

## Constraint Solving Algorithm

### Jacobi vs Gauss-Seidel
- **Choice**: Jacobi iteration for GPU parallelism
- **Trade-off**: Slower convergence vs. perfect parallelization
- **Mitigation**: More iterations, optimized constraint ordering

### XPBD Constraint Projection
```
For each constraint:
1. Compute current constraint error C(x)
2. Calculate constraint gradient ∇C(x)  
3. Compute Lagrange multiplier λ with compliance
4. Apply position corrections Δx = -λ * ∇C(x) / |∇C|²
```

### Convergence Strategy
- **Iterations**: 5-10 per frame for typical stiffness
- **Adaptive**: Monitor constraint error, adjust iteration count
- **Ordering**: Stretch → Bend → Collision for stability

## Physics Parameters

### Global Simulation
- **timestep**: Fixed 16.67ms (60 FPS) or dynamic
- **gravity**: World-space acceleration vector
- **damping**: Velocity damping coefficient [0,1]
- **substeps**: Multiple physics steps per frame for stability

### Material Properties
- **stretch_compliance**: Distance constraint softness [0,∞)
- **bend_compliance**: Bending resistance [0,∞) 
- **mass_distribution**: Per-particle mass variation
- **pin_constraints**: Fixed position constraints

### Environmental Forces
- **wind_velocity**: 3D wind vector in world space
- **air_resistance**: Aerodynamic drag coefficient
- **external_forces**: User interaction, magnetic fields, etc.

## Collision System Architecture

### Collision Primitives
- **Infinite Planes**: Ground, walls (normal + distance)
- **Spheres**: Dynamic rigid bodies (center + radius)
- **Signed Distance Fields**: Complex static geometry
- **Self-Collision**: Particle-particle distance constraints

### Collision Response
- **Projection Method**: Move particles out of collision volumes
- **Restitution**: Velocity-based bouncing behavior
- **Friction**: Tangential velocity reduction on contact
- **Penetration Depth**: Distance-based force magnitude

## Performance Optimization Strategies

### Memory Access Patterns
- **Coalesced Access**: Align data access to GPU memory architecture
- **Bank Conflicts**: Minimize shared memory bank conflicts
- **Cache Locality**: Group related data for temporal locality

### Compute Dispatch Optimization
- **Workgroup Size**: Match hardware thread group size (32/64)
- **Occupancy**: Balance registers/shared memory for maximum throughput
- **Load Balancing**: Distribute work evenly across compute units

### Algorithmic Optimizations
- **Spatial Acceleration**: Broad-phase collision detection
- **Constraint Graph Coloring**: Parallel constraint solving
- **Level-of-Detail**: Reduce computation for distant cloth
- **Temporal Coherence**: Exploit frame-to-frame similarity