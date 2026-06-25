# VoltLang

> A from-scratch, production-grade compiler & bytecode interpreter written in modern C++23 — featuring a hand-written recursive-descent parser, a multi-pass semantic analyzer, a register-based virtual machine, and a REPL.

```
volt > let fib = fn(n) => if n < 2 then n else fib(n-1) + fib(n-2);
volt > fib(30)
=> 832040  [executed in 14ms]
```

---

## Architecture

```
Source Code
    │
    ▼
┌─────────┐     ┌─────────┐     ┌──────────┐     ┌─────────────┐     ┌──────────┐
│  Lexer  │────▶│ Parser  │────▶│ Semantic │────▶│  Bytecode   │────▶│   VM /   │
│(Tokenize│     │  (AST)  │     │ Analyzer │     │  Compiler   │     │  Runtime │
│  )      │     │         │     │ (Types+  │     │  (VoltBC)   │     │          │
└─────────┘     └─────────┘     │  Scope)  │     └─────────────┘     └──────────┘
                                └──────────┘
                                      │
                                      ▼
                               ┌────────────┐
                               │    REPL    │
                               │(interactive│
                               │  shell)    │
                               └────────────┘
```

### Components

| Module | Description |
|---|---|
| `Lexer` | Hand-written UTF-8 aware tokenizer with source location tracking |
| `Parser` | Pratt (top-down operator precedence) recursive-descent parser |
| `AST` | Strongly-typed node hierarchy with visitor pattern |
| `Sema` | Multi-pass: name resolution → type inference → borrow checking |
| `CodeGen` | Compiles AST to VoltBC (custom register-based bytecode) |
| `VM` | Stack-less register VM with mark-and-sweep GC |
| `REPL` | Line-editing REPL with history, multi-line input, and `.volt` script loading |

---

## The VoltLang Language

VoltLang is a **statically-typed, expression-oriented** language with Hindley-Milner type inference, first-class functions, and algebraic data types.

### Features

- Type inference (no annotation required in most cases)
- First-class & higher-order functions
- Algebraic Data Types (enums with payloads)
- Pattern matching with exhaustiveness checking
- Closures with captured environments
- Tail-call optimization
- Module system
- Immutable by default (`let`), explicit mutation (`var`)

### Syntax Samples

```volt
// Fibonacci — recursive
let fib = fn(n: Int) -> Int =>
    if n < 2 then n else fib(n-1) + fib(n-2);

// Algebraic data type
type Shape =
    | Circle(radius: Float)
    | Rect(w: Float, h: Float)
    | Triangle(base: Float, height: Float);

// Pattern match
let area = fn(s: Shape) -> Float =>
    match s {
        Circle(r)      => 3.14159 * r * r,
        Rect(w, h)     => w * h,
        Triangle(b, h) => 0.5 * b * h,
    };

// Higher-order / closures
let adder = fn(x: Int) => fn(y: Int) => x + y;
let add5   = adder(5);
add5(10)  // => 15

// Tail-recursive factorial
let fact = fn(n: Int, acc: Int) -> Int =>
    if n <= 1 then acc else fact(n-1, n * acc);
```

---

## Building

### Requirements

- C++23-compliant compiler (GCC ≥ 13 / Clang ≥ 17)
- CMake ≥ 3.26
- Optional: `readline` for REPL line editing

```bash
git clone https://github.com/yourname/voltlang
cd voltlang
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
```

### Running

```bash
# Start the REPL
./build/volt

# Run a script
./build/volt examples/fibonacci.volt

# Dump bytecode (debug)
./build/volt --dump-bc examples/fibonacci.volt

# Dump AST
./build/volt --dump-ast examples/fibonacci.volt
```

---

## Project Structure

```
voltlang/
├── include/
│   ├── lexer/        Token.hpp, Lexer.hpp
│   ├── parser/       Parser.hpp
│   ├── ast/          Nodes.hpp, Visitor.hpp
│   ├── sema/         TypeEnv.hpp, Checker.hpp
│   ├── codegen/      Instruction.hpp, Compiler.hpp
│   └── runtime/      VM.hpp, GC.hpp, Value.hpp
├── src/
│   ├── lexer/        Lexer.cpp
│   ├── parser/       Parser.cpp
│   ├── ast/          ASTPrinter.cpp
│   ├── sema/         TypeChecker.cpp, TypeInfer.cpp
│   ├── codegen/      Compiler.cpp, Disassembler.cpp
│   ├── runtime/      VM.cpp, GC.cpp
│   └── repl/         REPL.cpp
├── tests/
│   ├── unit/         Per-module unit tests (Catch2)
│   └── integration/  End-to-end .volt programs
├── examples/         Sample VoltLang programs
├── docs/             Language spec, VM spec, bytecode format
├── CMakeLists.txt
└── README.md
```

---

## Bytecode Format (VoltBC)

VoltBC is a **register-based** instruction set. Each instruction is 32 bits wide:

```
 31      24 23    16 15     8 7      0
 ┌────────┬────────┬─────────┬────────┐
 │ OPCODE │  dst   │  src1   │  src2  │
 └────────┴────────┴─────────┴────────┘
```

Example disassembly for `fib(5)`:

```
0000  LOAD_INT   r0,  5
0002  CALL       r1,  @fib,  r0
0004  PRINT      r1
```

Full instruction reference in [`docs/bytecode.md`](docs/bytecode.md).

---

## Tests

```bash
# Run all unit + integration tests
ctest --test-dir build --output-on-failure

# Run only lexer tests
ctest --test-dir build -R lexer
```

---

## Roadmap

- [ ] LLVM IR backend (native compilation)
- [ ] Foreign Function Interface (FFI) to C
- [ ] Concurrency primitives (async/await, channels)
- [ ] Standard library (`std.list`, `std.map`, `std.io`)
- [ ] LSP language server

---

## License

MIT — see [LICENSE](LICENSE).
