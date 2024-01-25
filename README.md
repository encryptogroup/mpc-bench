# MPC-Bench

A distributed benchmarking tool for multiple secure multi-party computation (MPC) frameworks.

Currently supported are:
- [MP-SPDZ](https://github.com/data61/MP-SPDZ)
- [ABY](https://github.com/encryptogroup/ABY)
- [MOTION](https://github.com/encryptogroup/MOTION) and
- [SEEC](https://github.com/encryptogroup/SEEC)

This tool knows how to build the frameworks, compiles circuits, and executes an MPC either locally or remotely with configurably matrices of input values. We parse and unify the performance metrics of the frameworks and additionally collect wall-time measurements and max heap utilization using [heaptrack](https://github.com/KDE/heaptrack).

This project is written in Rust and needs a recent stable toolchain. The easiest way to install this is via the official [rustup](https://rustup.rs) toolchain manager.

## Dependencies
This tool depends on an unreleased version of [heaptrack](https://github.com/KDE/heaptrack) for memory profiling. Clone heaptrack into a folder adjacent to MPC-Bench (configurable via the CLI interface) and build it (`heaptrack_gui` not needed, only `heaptrack` and `heaptrack_print`).
Tested with commit: 4ae7d3c

## Running the tool
After installing Rust and recursively cloning this repository including submodules, run
```shell
cargo run -- --help
```
to build and run the benchmarking tool.

## Specifying benchmarks
Benchmarks are declaratively specified in a [toml](https://toml.io/en/) file.

```toml
# Network settings for all bechmarks, can be overwritten per benchmark
net_settings = ["RESET", "LAN", "WAN"]
# Number of repetitions per benchmark configuration
repeat = 5

[[bench]]
# Either SEEC, MP-SPDZ, MOTION, or ABY
framework = "SEEC"
# Which binary to build and execute. Must be a Cargo example
# for SEEC, an `.mpc` file in `Programs/Source` for MP-SPDZ (omit .mpc
# extension here), or a `src/examples` for ABY/MOTION.
target = "bristol"
# Tag for this benchmark configuration
tag = "seec_aes_ctr_no_setup"
# compile_flags are arguments that are supplied to every 
# compilation. Only usable for SEEC and MP-SPDZ
compile_flags = ["../../../../../circuits/advanced/aes_128.bristol"]
# flags are supplied to every benchmark configuration run
flags = ["--insecure-setup"]
# restrict the number of cores via taskset. 0 corresponds to no restriction
# cores contributes to benchmark configuration set
cores = [0,1]
# compile_args contribute to the benchmark configuration set. 
# If multiple compile_args keys are specified, we benchmark
# the product set of these arguments (including other options
# like the net_setting and cores)
[bench.compile_args]
--simd = ["1", "10", "100"]
```

Further examples are contained in the [bench-config.toml](./bench_config.toml) file. Paths are relative to the binary they are supplied to. E.g. because compiled SEEC `bristol` binary is located at `frameworks/SEEC/target/release/examples/bristol`, we need to go up 5 directories to get to the project root.

## Remote execution
While this tool supports local benchmarking for quick results, its main purpose is the remote execution and coordination on two servers. For this,
the necessary configuration can be supplied as a `toml` file.
Example (see [remote_conf.toml](./remote_conf.toml)):
```toml
# Jump host to use for the ssh connections
jump = "JUMP_HOST"
# IP:PORT of the server (id = 0) in the MPC, must be reachable from the client at this address
server = "IP:PORT"
# IP:PORT of the client (id = 1) in the MPC
client = "IP:PORT"
# path on the remote hosts where mpc-bench directory is rsynced to
remote_path = "/where/to/clone"
# ssh destination of the server and client
remote_hosts = ["user@server1.domain", "user@server2.domain"]
# directory of locally build heaptack main branch https://github.com/KDE/heaptrack
# as MPC-bench requires changes not present in published packages
heaptrack_dir = "/dir/to/heaptrack/"
# command to enable the lan setting emulation
lan_cmd = "sudo tc_lan10"
# command to enable the wan emulation
wan_cmd = "sudo tc_wan"
# command to reset the emulation
reset_net_cmd = "sudo tc_off"
```