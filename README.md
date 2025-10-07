This is Forked source code from 1.60 version of UE Explorer owned and made by Eliot. 
The main goal of this update is to bring custom unreal script compiler that is NOT violated with original UCC.exe/UnrealForntend.exe by Epic Games and reversed.

Why it is not easy to do?
A lot of commercial games powered by ue3 had modified low-level engine c++ laguage. None of all licensed sourced unreal buiilds had code of UnrealFrontend.exe. It is main GUI program used for compiling script and so on. This official Epic Games Compiler is not designed for "Native Functions" due to game specific modifications. The custom c# compiler is intended to convert that decompiled uc to bytecode. Finding the best matching Opcode for game recompile support is hard to do without leaked game sources.
So I managed to do many random operation codes until some work for compiliation process. 

<img width="366" height="413" alt="2025-10-07_13h14_33" src="https://github.com/user-attachments/assets/247b2e94-feb5-4a2c-9e15-fb382a97727b" />

As you can see added selectable "experimental opcodes".  By default is comes with 10 different examples you can test with different games, it is possible to enable multiple opcodes so it check for best one so other are skipped.
You can add and test your own written opcode in "UC C# COMPILER v6/config/opcodes.json" file then wire with UI here UE "Explorer/UnrealScript/ScriptCompiler.cs"  (reads UEEX_OPCODES env / applies policy)
This upcoming feature gonna be released in 0.2 version of UEModExplorer fork.



UnrealScript Mini Compiler (UE Explorer Extended)

Lightweight experimental compiler that takes a restricted UnrealScript subset, builds an AST/IR, emits legacy UE3 bytecode, and patches compiled functions back into an existing .u / .upk package. Designed for rapid mod iteration (patch‑in‑place) without full package rebuilding or export relocation.

Key Components
Lexer & Parser: Produces AST for class members and function bodies (subset: functions, returns, simple statements; expanding).
IR Builder: Converts AST nodes into a linear intermediate instruction list.
BytecodeEmitter: Translates IR to engine bytecode tokens, manages labels / jumps.
BytecodeWriter: Validates, optimizes (padding + duplicate return stripping), then overwrites function script slots inside the target package.
Optimizer (size oriented): Removes EX_Nothing (0x0B) padding, collapses consecutive return tokens, enforces single terminal return.
Patch Pipeline: Queues per‑function replacements, applies either in a new file or in-place with safety checks (size, offsets, bounds).
Truncation (optional / unsafe): If enabled, oversized output is forcibly truncated to existing slot length and tail‑patched with a return token.
Supported / Recognized Opcodes (current subset)
Return tokens: 0x04, 0x53 (legacy), 0xFF (compiler synthetic return placeholder accepted by validator).
Padding / No‑op: 0x0B (EX_Nothing) — stripped by optimizer unless needed for fixed slot padding.
(Additional tokens mapped internally via opcodes table; unimplemented tokens pass through only if already present in original bytecode.)
Bytecode Rules Enforced
Must end in a recognized Return token.
Replacement size must be <= original slot unless truncation enabled.
ScriptOffset interpretation: absolute file position = Export.SerialOffset + Function.ScriptOffset.
Shorter replacements are padded with 0x0B to preserve slot size (avoids shifting later exports).
Patch Modes
New File (recommended): Writes a copy with patched functions and leaves original intact.
In-Place: Opens original package, overwrites script ranges directly (creates backup first if configured).
Fallback: On in-place failure, temp-file write + atomic replace.
Safety & Validation
Size check before overwrite.
Boundary verification against file length.
Return opcode validation.
Optional optimization pass before considering truncation.
Console / UI reporting: per-function skip reasons (expanded, no match, mismatch).
Current Language Subset
Function declarations with parameter lists (parsed).
Return statements.
Basic expression handling (literals / simple placeholders).
Local var declaration syntax stubbed (parses, not yet emitted).
Unsupported constructs: loops, full switch, complex expressions, struct ops, state code.
Typical Workflow
Load package & class.
Edit or inject UnrealScript subset in editor.
Compile (shows function sizes).
Apply Patch (choose in-place or new file).
Re-open or run game to test.

Limitations
No relocation for larger functions (real expansion) yet.
Truncation feature can remove logic (dangerous) — off by default.
Minimal semantic/type checking.
Control flow beyond straight-line + return largely unimplemented.
Roadmap
Proper relocation: grow function script, adjust dependent offsets safely.
Control flow: if/else, loops, switch lowering, label resolution.
Parameter & local variable load/store emission.
Expression tree expansion (operators, function calls).
Peephole patterns (dead store removal, constant folding).
Safer return opcode normalization (emit canonical engine opcode instead of 0xFF).
Opcode coverage table auto-generated from engine version metadata.
Extending
Add AST nodes (e.g., ForStmt) → map in IRBuilder → implement Emit in BytecodeEmitter.
Update opcode map (opcodes.json or table) so validator accepts new tokens.
For relocation: augment BytecodeWriter to (a) allocate new buffer, (b) shift subsequent export SerialOffsets, (c) patch name/export tables & summary.
Build / Integration
Pure C# (.NET) inside UE Explorer fork.
No external dependencies besides existing UELib core.
Toggle unsafe truncation via UI checkbox (sets BytecodeWriter.AllowTruncateOversize).
Disclaimer
Experimental tooling; generated bytecode should be verified via re-decompile. Back up all original packages before in-place patching.

Need sections for relocation design or opcode table format next.
