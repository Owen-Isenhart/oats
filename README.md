# oats (Offensively Audited Target Syntax)
## Language Design & Architecture Specification
---

## 1. Executive Summary: The Adversarial Paradigm
OATS is a natively compiled, systems-level programming language that introduces a completely novel paradigm: **The Adversarial Compiler**. 

While traditional compilers use static analysis to ensure types match and syntax is correct, the OATS compiler acts as an active, autonomous red team. During compilation, the compiler spins up an internal Virtual Machine (The Arena) and actively attacks the intermediate representation (IR) of the code. It attempts to violate developer-defined invariants and memory safety rules using SMT solvers, machine learning heuristics, and mutational fuzzing. If the compiler can exploit the code, the build fails and provides the developer with the exact exploit chain.

**Alternative Acronym Meanings:**
* *Obligation Assertion & Testing System* (Focus on Contract-Oriented design)
* *Open Assurance Toolchain Standard* (Enterprise/Infrastructure friendly)

---

## 2. Core Language Semantics & Philosophy

### 2.1 Paradigm: Imperative Systems + Contract-Oriented
OATS is designed for highly critical edge systems, robotics, and low-level infrastructure. It gives the developer dangerous tools (mutability, pointers, raw memory access) but wraps them in strict, mathematically verifiable contracts.

### 2.2 Memory Management
To maintain extreme runtime performance, OATS does **not** use Garbage Collection (GC). It relies on a deterministic lifecycle:
* **Linear/Affine Types (Ownership):** Similar to Rust, variables have strict owners.
* **Temporal Decay:** Built-in primitives for variables that expire logically after a set Time-To-Live (TTL), useful for cryptographic hygiene.

### 2.3 Syntax & Contracts (Design by Contract)
Developers write imperative logic accompanied by strictly enforced invariant blocks.

```rust
// Example OATS Syntax
contract secure_transfer(amount: u64, sender: &User, receiver: &User) {
    // Pre-conditions (Must be true to execute)
    require!(sender.balance >= amount);
    
    // Invariants (Must be true at all times during execution)
    invariant total_funds {
        sender.balance + receiver.balance == PREV(sender.balance + receiver.balance);
    }

    // Logic
    sender.balance -= amount;
    receiver.balance += amount;
}
```

---

## 3. The Adversarial Compiler Pipeline
Building the OATS compiler is the most complex part of the language. It operates in stages:

### Stage 1: Lexical Analysis & Parsing
Standard conversion of source text into an Abstract Syntax Tree (AST).

### Stage 2: IR Generation & The Arena (The Novelty)
The AST is lowered into a proprietary Intermediate Representation (OATS-IR). Before this IR is passed to the backend, it enters **The Arena**, where the internal Red Team attacks it:
* **Layer 1 (The Mathematician - SMT Solvers):** Tools like Z3 symbolically execute the IR to prove that variables cannot exceed their bounds or violate invariants.
* **Layer 2 (The Hacker - ML Agents):** A lightweight, embedded inference model analyzes the IR state machine and generates attack heuristics based on known vulnerability patterns.
* **Layer 3 (The Brute Force - Mutational Fuzzer):** Using the heuristics from Layer 2, a highly optimized engine blasts thousands of mutated inputs into the internal IR Virtual Machine to attempt state-corruption.

### Stage 3: Code Generation (Backend)
If the code survives The Arena, the IR is passed to an LLVM backend to be aggressively optimized and compiled into bare-metal machine code (`.elf`, `.exe`, `.macho`).

### The Developer Experience (DX)
If compilation fails, OATS does not just throw an error; it generates an incident report:
`[FATAL] invariant 'total_funds' compromised via integer overflow payload on line 14.`

---

## 4. The Standard Library (`std`)
Because OATS requires an ML inference engine inside the compiler, that engine is safely exposed to developers at runtime without requiring massive third-party dependencies. OATS is "batteries-included" for Edge AI and Security.

### 4.1 `std::ml::infer` (Zero-Dependency AI)
A built-in, lightweight tensor execution engine. Developers can load pre-trained weights (e.g., ONNX, GGML) and run inference natively.
* *Use Case:* Running local anomaly detection or computer vision on edge hardware.

### 4.2 `std::security::monitor` (Runtime Defenses)
Allows developers to attach statistical monitors to network streams or memory buffers to detect real-time attacks mirroring what the compiler tested.

### 4.3 `std::prob` (Probabilistic Types)
Native data structures representing probability distributions rather than discrete values, making it ideal for sensor fusion and fuzzy logic.

### 4.4 `std::contracts`
Core primitives for defining obligations, assertions, and state machine transitions.

---

## 5. Ecosystem & Tooling

### 5.1 Package Manager: `silo`
Instead of pulling packages blindly from the internet, the OATS package manager (`silo`) operates on a Zero-Trust model. 
* **Fuzz-on-Import:** When a developer imports a third-party library, the Adversarial Compiler treats the imported code as hostile and attempts to exploit its boundaries before allowing it to interact with the host project.

### 5.2 Build System: `forge`
Handles orchestration of the SMT solvers, caching safe IR representations (so you don't re-fuzz unchanged code every time you hit compile), and linking.

---

## 6. Implementation Roadmap
To bring OATS to life, the following components must be built from scratch:

1. **The Formal Grammar:** Defining the syntax, keywords, and AST structures.
2. **The Lexer/Parser:** Writing the frontend (likely in Rust or C++ for speed) to parse OATS syntax.
3. **The OATS-IR & VM:** Designing the byte-code representation and the hyper-fast internal Virtual Machine used for testing.
4. **The Arena Integration:** Hooking Z3 (SMT) and a custom ML inference engine into the compilation pipeline.
5. **The LLVM Bridge:** Translating OATS-IR into LLVM-IR for final machine code generation.
6. **The Standard Library:** Writing the core `std` modules using unsafe OATS code wrapped in ultra-strict contracts.
7. **The Toolchain:** Building `silo` (package manager), `forge` (build tool), and the language server (LSP) for VSCode/IDE integration.
