# GPU Cloth Simulation Project - CLAUDE.md

> Think carefully and implement the most concise solution that changes as little code as possible.

## Project-Specific Instructions

This is a GPU cloth simulation project built with Rust, wgpu, and winit. Key principles:

### Development Approach
- **Performance First**: Always maintain 60+ FPS target during development
- **GPU-Native**: All physics computation happens on GPU via compute shaders
- **Incremental**: Follow the 6-milestone development plan strictly
- **Documentation**: Document all mathematical concepts and GPU programming techniques

### Architecture Principles  
- **Separation of Concerns**: Clean separation between physics, rendering, and application layers
- **Type Safety**: Use Rust's type system to prevent GPU buffer management errors
- **Memory Efficiency**: Structure-of-Arrays layout for optimal GPU memory access
- **Cross-Platform**: Ensure compatibility across Windows, macOS, and Linux

### Code Quality Standards
- **Mathematical Accuracy**: Verify physics algorithms against reference implementations
- **Numerical Stability**: Test edge cases and parameter ranges thoroughly
- **Performance Validation**: Profile GPU performance after any changes to compute shaders
- **Memory Management**: Use RAII patterns for GPU resource cleanup

### Context Management
- **Always Prime Context**: Run `/context:prime` at start of work sessions
- **Update Progress**: Update `/context/project-status.md` after completing milestones
- **Document Decisions**: Record architectural choices and algorithm modifications

## Repository Information
- **Repository**: thomasconrad/gpu-cloth-sim2
- **Primary Branch**: main
- **Language**: Rust (edition 2021)
- **GPU API**: wgpu (WebGPU abstraction)
- **Target Platforms**: Desktop (Windows/macOS/Linux) with future WebAssembly support

## Testing

Run comprehensive tests before committing:
- `cargo test` - Unit tests for mathematical operations
- `cargo bench` - Performance benchmarks to detect regression
- GPU validation with different hardware when available
- Cross-platform compatibility verification

## Code Style

Follow the project's Rust conventions defined in `.claude/context/development-conventions.md`:
- Use snake_case for variables and functions
- Use PascalCase for types and structs
- Use SCREAMING_SNAKE_CASE for constants
- Comprehensive documentation for all public APIs
- Explicit error handling with descriptive messages
