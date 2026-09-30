# ARM64 interpreter checks

The ARM64 LLInt object is needed for interpreter entry points, including when
JIT compilation is disabled. Its generated header must match `NDEBUG`: debug
guards contain assertions that are absent from release configurations.

Configure `llint-config` with CMake and run CTest to check header selection:

```
cmake -S darling/tests/llint-config -B /tmp/llint-release -DCMAKE_BUILD_TYPE=Release
ctest --test-dir /tmp/llint-release --output-on-failure
```

Also check Debug, RelWithDebInfo, MinSizeRel, and an empty build type. This tests
selection, not compilation. Compile `llint/LowLevelInterpreter.cpp` through the
normal Darling build for ARM64 and x86_64, with debug and release headers.

`interpreter-execution.c` is a public-API runtime smoke test. Compile it against
the built framework's public headers and link JavaScriptCore; run the resulting
Mach-O executable inside Darling. Require exit zero and the PASS line. It sets
`JSC_useJIT=false` before creating a context, checks loop evaluation, deliberately
throws an exception, and verifies subsequent evaluation. The expected exception
may also appear in console logs. Checks remain active with `NDEBUG` defined.

For regeneration, `darling/scripts/generate-offlineasm.sh ARM64` uses the Darling
source tree, build tools and SDK selected by `DARLING_ROOT`,
`DARLING_BUILD_ROOT`, and `DARLING_SDK_ROOT`. Save the reference headers first,
then use `compare-offlineasm.rb reference.h regenerated.h` to compare generated
code while ignoring comments containing checkout-specific paths. Selecting
X86_64 also generates its required C_LOOP header. Those existing extraction
targets require an appropriate x86/i386 SDK and linker; an ARM64-only staged
installation is not sufficient to validate their regeneration or runtime.
