# Artifact evaluation instructions

## Getting started guide

### Intro

This artifact is prepared as docker image. The docker image contains `eff` compiler build from source along with additional external benchmark files and working version of multicore ocaml compiler used for running benchmarks. Dockerfile was tested on MacOS (version v19.03.13-beta2) and Windows machine.

### Instructions

- Install docker for your [operating system](https://docs.docker.com/engine/install/), any recent enough version will do.
- Before any operation, make sure, that `docker-deamon` or docker service is running.
- Obtain Docker image. This can be done in two ways:

  - Use prebuilt image available on Dockerhub (faster, recommended):
    - Pull image using `docker pull j00sko/eff-oopsla-artifact:oopsla2021`.
  - Build image locally(slower).
    - The repository contains the same `Dockerfile` used to build image on dockerhub.
    - Run `docker build -t j00sko/eff-oopsla-artifact:oopsla2021 .`. Build can take up to 30 minutes depending on your internet connection and computer power. Warnings from opam that "running as root is not recommended" are expected and should be ignored.

- Test successful build by running `docker run j00sko/eff-oopsla-artifact:oopsla2021 ./eff.exe examples/random.eff`. This runs simple example and should produce:

```
[WARNING] Running as root is not recommended
- : int = 10
- : int = 10
val choose_all : 'a => 'a list = <handler>
- : int list = [10; 5; 20; 15]
val choose_all2 : 'a => 'a list = <handler>
- : int list list = [[10; 5]; [20; 15]]
- : int list list = [[10; 20]; [5; 15]]
- : int list list = [[10; 20]; [10; 15]; [5; 20]; [5; 15]]
```

Warnings about running as root can safely be ignored.

## Step by Step Instructions

### Intro

Dockerfile starts with original ocaml docker image and install necessary libraries for building `Eff` compiler presented in the paper. Pinned version of `Eff` is compiled from scratch and an executable is created. Finally an additional switch for Multicore ocaml on which benchmarks are run is installed and set as default.

Pinned version of `eff` is practically the same as the one used for the paper with same minor differences:

- Library `delimcc` is excluded since it doesn't work with multicore ocaml.
- Benchmarks from `Handlers in action` and  `Eff in Ocaml` are commented out. These benchmarks are a few orders of magnitude slower than the ones presented in paper and provide no additional information, while greatly increasing benchmarking time.
- Additional flag is added to optionally compile multicore ocaml benchmarks on both versions `4.10` (already in paper) and `4.12` (newly added).
- Bash script helpers are added to enable easier access to functionality and artifact testing.
- Number of runs for benchmarks is lowered due to overhead induced by container. Number of runs can be increased by setting parameters in `misc/code-generation-benchmarks/display-results/benchmark.ml` in line 12 for human readable output and `misc/code-generation-benchmarks/generate-graphs/graphs.ml` in lines 6 and 8 for graph generation.

Any commands can also be executed directly in source code repo available at [https://github.com/matijapretnar/eff/tree/explicit-effect-subtyping](https://github.com/matijapretnar/eff/tree/explicit-effect-subtyping), provided that you have ocaml with necessary libraries installed (following instructions in eff repo or Dockerfile).

Execute command `docker run -it j00sko/eff-oopsla-artifact:oopsla2021 bash`, to get access to docker.

### Available commands

Unless otherwise states, all commands are run in docker. This is also the same format for commands if you were to use repository locally.

- Generate data for benchmark results presented in paper:
`make graphs`.

  This runs all benchmarks and regenerates table files located in `/eff/misc/code-generation-benchmarks/generate-graphs/tables/`. Filename consists of benchmark name (`count`, `interpreter-state`, ...) and benchmark backend (`generated` for generated ocaml code, `native`, `capabilities`, ...).

  Each file has a header with user readable information and space separated list of values: input parameter and execution time ratio with respect to native ocaml version.

- Run benchmarks and output benchmark data in human readable format
`make benchmark`.

  This runs the same benchmarks as presented in paper but outputs human readable `bechamel` results.

  Specific benchmarks can be excluded and tried on different parameters by modifying file `misc/code-generation-benchmarks/benchmark-suite/benchmark_config.ml`. Multicore OCaml specific config is available in `misc/code-generation-benchmarks/benchmark-suite/config.multicore.ml`.

- Compile custom `eff` source code in file `INPUT_FILENAME` using compiler presented in paper with all optimizations and write resulting `OCaml` program to file `OUTPUT_FILENAME`
`/eff.exe --no-stdlib --compile-plain-ocaml INPUT_FILENAME > OUTPUT_FILENAME.ml`.

  Command `./eff.exe --no-stdlib --compile-plain-ocaml misc/code-generation-benchmarks/benchmark-suite/loop.eff > out.ml` compiles all loop benchmarks presented in paper.

  Resulting file has depends on `ocamlHeader.ml` located in `ocamlHeader/ocamlHeader.ml` and to use it, you have to compile them together `ocaml ocamlHeader/ocamlHeader.ml out.ml` or just manually copy contents from the header to generated file and use it as is.

- Extend and change the compiler.

  Full compiler source code is available and users can make their own modifications and test the compiler. We encourage users to use standard OCaml compiler for this by running `opam switch 4.12; eval $(opam env)`. Running `make` recompiles `Eff` compiler and running `make generate_benchmarks` regenerates `Eff` based benchmarks.

  *WARNING*: Docker file system is non persistent, so any changes made in container will disappear after container exists. We suggest using local repository for compiler modifications.

### Claims

Claims supported by artifact:

- Prototype implementation of optimizing compiler as an extension of `Eff`.

  Artifact provide straightforward way to compile `Eff` source files with experimental version of compiler and using it as a normal OCaml source (see: Compile custom `Eff` source code).

Claims not/partially by artifact:

- Due to virtualization overhead and lowered benchmark runs, performance claims might not be reproducible in docker environment and the measurement results suffer from additional noise caused by low number of repeats.

### Remarks

Running `make test` with multicore version of compiler fails due to missing `ocamlformat` library. See section `Extend and change the compiler` on how to switch to standard OCaml.
