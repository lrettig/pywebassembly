*WARNING: This is an early demo in need of a code review.*
  

**pyWebAssembly.py**: Closely follows *WebAssembly Specification, Release 1.0, Dec 13, 2017*. Implements Chapter 5 and necessary parts of Chapters 2 and 4. The resulting recursive-descent parser takes a `.wasm` file and builds a syntax tree with nested tuples, lists, and dicts. This process is invertible back to a `.wasm` file. Functions from the specification are named `spec_<func>(...)` and their inverses (of a canonical form) are named `spec_<func>_inv(...)`.


**metering.py**: Injects a function call before each sequence of unbranching instructions, whose argument is the sum of costs of instructions in the sequence. Also injects helper functions, including the metering function which updates the cycle count and traps when a limit is exceeded.

The original C fibonacci function:

Before metering (see `fibonacci.wast` for full module, and the original fibonacci function in C).
```
  (func (;0;) (type 0) (param i32) (result i32)
    (local i32 i32 i32)
    i32.const 1
    set_local 1
    block  ;; label = @1
      get_local 0
      i32.const 1
      i32.lt_s
      br_if 0 (;@1;)
      i32.const 1
      set_local 3
      i32.const 0
      set_local 2
      loop  ;; label = @2
        get_local 2
        get_local 3
        i32.add
        set_local 1
        get_local 3
        set_local 2
        get_local 1
        set_local 3
        get_local 0
        i32.const -1
        i32.add
        tee_local 0
        br_if 0 (;@2;)
      end
    end
    get_local 1)
```


After metering (see `fibonacci_metered.wast` for full module). Notice injected `i32.const n` `call 1` where n is sum of costs of instructions until the next branch (each instruction costs one for this example). The metering function has index `1` in this example -- this index is handled automatically.
```
  (func (;0;) (type 0) (param i32) (result i32)
    (local i32 i32 i32)
    i32.const 3
    call 1
    i32.const 1
    set_local 1
    block  ;; label = @1
      i32.const 4
      call 1
      get_local 0
      i32.const 1
      i32.lt_s
      br_if 0 (;@1;)
      i32.const 5
      call 1
      i32.const 1
      set_local 3
      i32.const 0
      set_local 2
      loop  ;; label = @2
        i32.const 13
        call 1
        get_local 2
        get_local 3
        i32.add
        set_local 1
        get_local 3
        set_local 2
        get_local 1
        set_local 3
        get_local 0
        i32.const -1
        i32.add
        tee_local 0
        br_if 0 (;@2;)
      end
    end
    i32.const 1
    call 1
    get_local 1)
```


**index.html**: Demo to run any file generated by `metering.py` (e.g. `fibonacci_metered.wasm`). User specifies the `.wasm` file, function name, argument, and cycles limit. Because JavaScript doesn't support the `i64` type and we want more than 2^32 cycles, we allow arbitrary precision integers imported and exported to and from the wasm module as a sequence of `i32`s, using helper functions in both JavaScript and injected by `metering.py`. A trap occurs if the cycles limit is exceeded! To run this demo, a server is needed to serve the `.wasm` file: execute `python3 -m http.server` in this directory and visit the given URL in the browser. Notice the 46th Fibonacci numbers overflows `i32`.


**fibonacci...**: Various versions of our demo fibonacci module. The `_metered.wasm` file is generated by `metering.py`. The `.wast` versions are generated by `wasm2wast`.


TODO:

 * Floating point canonicalization code injector.
 * Interpretter as described in chapter 4.
 * Support text format as described in chapter 6.



