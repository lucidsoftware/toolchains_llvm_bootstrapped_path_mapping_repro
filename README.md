# toolchains_llvm_bootstrapped path mapping repro

This is a minimal repro case to demonstrate that `toolchains_llvm_bootstrapped` fails to compile cc targets when [path mapping](https://github.com/bazelbuild/bazel/discussions/22658) `--experimental_output_paths=strip` is enabled.

## How to repro

1. Build the following target:
```bash
bazel build //hello
```

2. An error like the following will occur. It won't always be an error for `libc++.a` - sometimes the error is for `libunwind.a`, `Scrt1.o`, `libclang_rt.builtins.a`, `libc++abi.a`, `libld.so.2`, or another similar file.
```
ERROR: /home/redacted/.cache/bazel/_bazel_redacted/c1aba044061d2d9d86019fc7d2ccee63/external/llvm+/runtimes/libcxx/BUILD.bazel:28:13: RunBinary external/llvm+/runtimes/libcxx/libc++.a failed: (Exit 1): llvm-ar failed: error executing RunBinary command (from _run_binary rule target @@llvm+//runtimes/libcxx:c++) external/llvm++http_archive+llvm-toolchain-minimal-21.1.8-linux-amd64/bin/llvm-ar rc bazel-out/k8-fastbuild/bin/external/llvm+/runtimes/libcxx/libc++.a '--format=gnu'

Use --sandbox_debug to see verbose messages from the sandbox and retain the sandbox build root for debugging
external/llvm++http_archive+llvm-toolchain-minimal-21.1.8-linux-amd64/bin/llvm-ar: error: bazel-out/k8-fastbuild/bin/external/llvm+/runtimes/libcxx/libc++.a: No such file or directory
```

3. Uncomment these two lines in .bazelrc
```
common --modify_execution_info=RunBinary=-supports-path-mapping
common --modify_execution_info=CopyToDirectory=-supports-path-mapping
```

Then build `//hello` again.

This time the build passes.
