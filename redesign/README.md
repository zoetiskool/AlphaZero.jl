# AlphaZero.jl Redesign

## Redesign Objectives

- A codebase that is more accessible and easier to read
  - The file hierarchy reflects the module hierarchy
  - Literate programming using `Pollen`
  - A layered API similar to the one used in FastAI?
- Improved integration with the rest of the ecosystem:
  - Use the `ReinforcementLearning.RLBase` interface for environments
  - Use the `Logging` and `ProgressLogging` modules for logging
  - Use the `PrettyTables` and `Term` packaged for the Terminal UI
- Better support for distributed computing
  - Agent-based architecture for leveraging multiple machines and GPU (the previous version only parallelizes data generation)
- Improved performances
  - The goal is to be competitive with Fabrice Rosay's `AZ.jl` implementation (or even `AlphaGPU`).
- Improved modularity
  - Support for both AlphaZero and MuZero algorithms
  - Support for several MCTS implementations (including a batched MCTS implementation and a full-GPU implementation)
- Batteries included
  - Provide a test suite for new environments
  - Check hyperparameters consistency
  - Provide standard hyperparameter tuning utilities
  - Provide profiling utilities

## Mistakes to fix

- Having a centralized hyperparameters structure is not a good idea as it introduces a lot of coupling in the codebase and prevents switching components easily. Having JSON serialization of all hyperparameters by default is also exceedingly rigid. In general, centralizing all hyperparameters in a serializable structure should happen at a higher API layer.
- The way training statistics are logged using `Report` is too heavy. The logging library should be used for this.
- Submodules are underused, which hurts discoverability.
- Tests are lacking.
- Latency is very high, in part due to making many types unnecessarily parametric.
- Using multithreading for the workers and inference server may lead to bad performances at the GC constantly stops the world. Having multiple processes may be better (i.e. using Distributed).
- Precompilation is broken since we rely on conditional loading (for the CUDA_MEMORY_POOL and USE_KNET flags). We should use `Preferences` instead.
- To help with replicability, random number generators must be passed explicitly.

## Codebase Architecture

## Coding Style

- This codebase enforces the [Blue Style](https://github.com/invenia/BlueStyle).
- Each source file defines a submodule. The files hierarchy perfectly reflects the underlying module hierarchy.
- We use the `Reexport` package so as to ease working with module hierarchies.
- We should make sure that the codebase can be explored using the "Jump to definition" feature of VS-Code.

## Workflow

We use `JuliaFormatter` to format the code on save. To do so, use the following VSCode configuration:

```json
{
    "[julia]": {
      "editor.detectIndentation": false,
      "editor.insertSpaces": true,
      "editor.tabSize": 4,
      "files.insertFinalNewline": true,
      "files.trimFinalNewlines": true,
      "files.trimTrailingWhitespace": true,
      "editor.rulers": [92],
      "editor.formatOnSave": true
    }
}
```

To run the VSCode debugger within the REPL, just write:

```julia
@run function_to_debug()
```

## Dev Plan

- We start implementing a minimal version of AlphaZero as it is easier:
  - Reset MCTS tree everytime for now.