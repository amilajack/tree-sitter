# Fuzzing tree-sitter

The tree-sitter fuzzing support requires 1) the `libFuzzer` runtime library and 2) a recent version of clang

## libFuzzer

The main fuzzing logic is implemented by `libFuzzer` which is part of the LLVM project but is not shipped by distros. It will need to be built from source but does not require building the _whole_ LLVM project. LLVM can be downloaded from llvm.org using SVN or [llvm-mirror](https://github.com/llvm-mirror/llvm) using git. `libFuzzer` can be built as, e.g.:

```
cd ~/src
git clone https://github.com/llvm-mirror/llvm
cd llvm/lib/Fuzzer
./build.sh
```

## clang

Using libFuzzer requires a reasonably new version of `clang` and will probably _not_ work with your system-installed version. The easiest way to get started is to use the version provided by the Chromium team. Instructions are available at [libFuzzer.info](http://libfuzzer.info).

The fuzzers can then be built with:
```
export CLANG_DIR=$HOME/src/third_party/llvm-build/Release+Asserts/bin
CC="$CLANG_DIR/clang" CXX="$CLANG_DIR/clang++" LINK="$CLANG_DIR/clang++" \
  LIB_FUZZER_PATH=$HOME/src/llvm/lib/Fuzzer/libFuzzer.a \
  ./script/build_fuzzers
```

This will generate a separate fuzzer for each grammar defined in `test/fixtures/grammars` and will be instrumented with [AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) and [UndefinedBehaviorSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html). Individual fuzzers can be built with, for example, `./script/build_fuzzers python ruby`.

The `run-fuzzer` script handles running an individual fuzzer with a sensible default set of arguments:
```
./script/run-fuzzer <grammar-name> <extra libFuzzer arguments...>
```

which will log information to stdout. Failing testcases and a fuzz corpus will be saved to `fuzz-results/<grammar-name>`. The most important extra `libFuzzer` options are `-jobs` and `-workers` which allow parallel fuzzing. This is can done with, e.g.:
```
./script/run-fuzzer <grammer-name> -jobs=32 -workers=32
```

The testcase can be used to reproduce the crash by running:
```
./script/reproduce <grammar-name> <path-to-testcase>
```
