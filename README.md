🌟 Introduction: A New Paradigm for C-code – the .desf formatIn the world of software development, there is a constant need for optimization and universality. Traditional approaches to compiling C-code into dynamic libraries (.so for Linux/Unix, .dll for Windows) have significant drawbacks: they depend on a specific operating system and processor architecture, which greatly complicates portability and deployment.It is precisely to solve these problems that the idea of creating .desf (Description File) – a new, platform-independent intermediate format – emerged.The main mission of .desf is to ensure execution optimization and complete platform independence, abandoning standard compiled libraries. This approach is particularly relevant for improving work with large scripts, for example, in the BYM environment, and for running code on a wide variety of devices, from powerful servers to embedded systems and even IoT devices (yes, we are talking about that "refrigerator"!).The c2desf project includes:.desf format specification: A detailed description of the innovative representation of C-code.c2desf utility: A tool for automatic conversion of source C-code into .desf format..desf Virtual Machine (VM): A high-performance interpreter for executing .desf files on any supported platform.✨ Key Features and Benefits🌐 Platform Independence: Run your C-code anywhere! From Windows, Linux, macOS to Android, iOS, and microcontrollers. .desf eliminates the need to compile for each individual architecture.🚀 Execution Optimization:Size: Aggressive minification, intelligent dead code removal, and compact command representation significantly reduce the size of the final file (potentially up to 30-50% savings compared to original C).Speed: The use of SIMD instructions (NEON, AVX2), caching, and potential JIT compilation of "hot" code sections ensure high performance.🛡️ Enhanced Protection:Multi-level obfuscation: Identifier renaming, string encryption, "muddy code" injection, Control Flow Flattening.Anti-debugging techniques: Complicating analysis and reverse engineering.Non-standard format: The .desf format itself is a barrier to standard decompilation tools.💡 Innovative Concept: Abandoning traditional binary libraries in favor of an interpreted, but optimized representation.🔧 Automation: The c2desf utility automates the entire conversion process.🧩 Modularity and Extensibility: Clear project structure and potential for plugins.🔒 Security for IoT: Comprehensive measures for secure code execution on connected devices, including cryptography, authentication, and isolation.Why is .desf better than traditional .so/.dll?CharacteristicTraditional .so/.dll.desf FormatSizeOften large, compiler-dependentSignificantly smaller due to optimizations and compact command representation.Protection from analysisRelatively easy to decompile (IDA Pro, Ghidra)Significantly higher level of protection through obfuscation and unknown bytecode.Platform IndependenceAbsent, separate compilation requiredComplete, thanks to the interpreter/VM.PortabilityLimited by architecture and OSMaximum, from servers to microcontrollers.🧠 .desf Concept: Execution via Command InterpretationUnlike direct compilation into machine code for a specific architecture, .desf uses a different approach.A .desf file is not a binary file of an incomprehensible format, but a specially structured C-code (or text-binary/binary file in the final version). Its key feature is that all the logic of the original C-program is converted into a sequence of simple commands.These commands are stored in an array inside the generated file (if it is C-code) or in a specialized structure (if it is a binary format). Program execution occurs by iterating through this command array in a for loop (in the conceptual C-representation) and interpreting them using a large switch-case construct in the .desf virtual machine. Each case is responsible for executing a specific command (e.g., assign a value to a variable, call a function, jump to another part of the code).Optimization and obfuscation at the conceptual levelTo achieve high performance and reduce code size, as well as to complicate analysis, obfuscation and optimization are applied already at the .desf generation stage:Name Shortening: All variable and function names are replaced with single-character or short identifiers (e.g., myCounter → @a, calculateSum → F1). This is implemented using #define macros in the intermediate C-code or stored in the .desf file's symbol table.Command Structure: Program logic (operators, function calls) is converted into numerical command codes, which are stored in an enum (e.g., enum CMD { SET_A=0, PRINT=1, ADD=2, JUMP_IF_ZERO=3, EXIT=4 };).This approach allows a standard C-compiler (e.g., gcc), used to compile the VM itself, to effectively optimize the generated switch-case code, using all the capabilities of the target architecture (caching, branch prediction, etc.), while the .desf file itself remains compilable by standard means without external dependencies (or executed by the VM without prior compilation on the target platform).📄 .desf Format in DetailThe .desf format is the foundation of the entire project. Its specification defines how C-code is transformed into a set of commands understandable by the virtual machine.Example of .desf file structure (conceptual C-representation)This is a simplified example illustrating the idea. The real .desf can be binary.// File: program.desf (compiled as a regular C-file: gcc program.desf -o program)
// OR: this file is input data for the .desf VM

#include <stdio.h> // Include standard libraries as needed (in VM this will be abstracted)

// --- Obfuscation via macros (or symbol table in .desf) ---
#define var_a vm_registers[0] // Variable 'counter' is now 'var_a' (element of VM registers array)
#define var_b vm_registers[1] // Another variable
#define F_print printf      // Function 'printf' is now 'F_print' (external function call via VM)

// --- Command Codes (defined in .desf specification) ---
enum DESF_CMD {
    CMD_SET_VAR = 0,   // Command: set variable (register) value
    CMD_ADD_VARS = 1,  // Command: add two variables, result in the first
    CMD_PRINT_VAR = 2, // Command: print variable
    CMD_EXIT_VM = 3    // Command: terminate VM execution
    // ... and many other commands for loops, conditions, function calls, etc.
};

// --- Array for storing obfuscated variable values (VM registers) ---
// int vm_registers[MAX_REGISTERS] = {0}; // Managed by VM

// --- Sequence of commands and their arguments (heart of .desf file) ---
// Each command can have a different number of arguments.
// For example: { CMD_ID, ARG1_TYPE, ARG1_VAL, ARG2_TYPE, ARG2_VAL, ... }
// ARG_TYPE can be: CONSTANT, REGISTER_IDX, LABEL_IDX etc.
int desf_commands[] = {
    CMD_SET_VAR, /*reg_idx*/ 0, /*value*/ 5,     // var_a = 5
    CMD_SET_VAR, /*reg_idx*/ 1, /*value*/ 10,    // var_b = 10
    CMD_ADD_VARS, /*dest_reg*/ 0, /*src_reg1*/ 0, /*src_reg2*/ 1, // var_a = var_a + var_b
    CMD_PRINT_VAR, /*reg_idx*/ 0,                 // F_print("...", var_a)
    CMD_EXIT_VM
};

int desf_command_count = sizeof(desf_commands) / sizeof(int); // Not exactly, depends on structure

// --- Main execution logic (implemented in .desf VM) ---
/*
int main_vm_loop() {
    int pc = 0; // Program Counter in VM

    // VM command execution loop
    while (desf_commands[pc] != CMD_EXIT_VM && pc < desf_command_count) { // Exit condition can be more complex
        int current_command_code = desf_commands[pc];
        switch (current_command_code) {
            case CMD_SET_VAR:
                // vm_registers[desf_commands[pc+1]] = desf_commands[pc+2];
                // pc += 3; // Move to next command + arguments
                break;
            case CMD_ADD_VARS:
                // vm_registers[desf_commands[pc+1]] = vm_registers[desf_commands[pc+2]] + vm_registers[desf_commands[pc+3]];
                // pc += 4;
                break;
            case CMD_PRINT_VAR:
                // F_print("Result: %d\n", vm_registers[desf_commands[pc+1]]);
                // pc += 2;
                break;
            // ... other cases for other commands ...
            default:
                // Handling unknown command (optional)
                // F_print("Unknown command: %d\n", desf_commands[pc]);
                return 1; // Error
        }
    }
    return 0; // Successful completion
}
*/
Command Specification (commands.h)Key to the consistent operation of the generator (c2desf) and the interpreter (VM) is the include/desf/commands.h file (or similar specification file), which defines the ".desf format contract".Main elements of the specification:Format version and "magic number": For identification and compatibility.#define DESF_VERSION 0x00010000 // Version 1.0.0
#define DESF_MAGIC   0xDEADBEEF // "Magic" number for file identification
Limitations: For example, maximum number of commands, operand/label length.Command Types (DesfCommandType): An extended list of commands covering:Arithmetic operations (integer, floating-point, vector).Logical operations.Comparison operations.Control flow operations (unconditional and conditional jumps, subroutine calls, returns).Loop operations (start, end, iteration).Memory operations (load, store, allocate).I/O operations (abstracted from specific stdio implementation).Debugging commands (e.g., DESF_BREAKPOINT).Command Structure (DesfCommand): Defines how each command is stored in the file.   // Example of a possible command structure
typedef struct {
    uint8_t type;     // Command type (from DesfCommandType)
    uint8_t flags;    // Additional flags for the command
    union {
        struct { // For commands with operands
            DesfOperand op1;
            DesfOperand op2;
            DesfOperand op3; // Some commands require 3 operands
        } operands;
        char* label; // For jump commands (label)
        // ... other data variants
    } data;
} DesfCommand;

// Example of operand structure
typedef struct {
    DesfOperandType type; // Operand type: constant, register, memory address, label
    union {
        int32_t constant_val_i32;
        float   constant_val_f32;
        uint16_t register_idx;
        uint32_t memory_addr;
        char* label_name;
        // ...
    } value;
} DesfOperand;
File Header Structure (DesfHeader): Contains metadata (version, magic number, section sizes, symbol table, string table, etc.).Generation Macros: Convenient C-macros (DESF_SET, DESF_ADD, etc.) to simplify code generation (if .desf is generated as a C-file).Evolution of commands.h and "proactive" improvements:During development, the format specification underwent changes for improvement:Solving the stdio problem: I/O functions are moved to a separate desf_stdio.c module with conditional compilation (#ifdef DESF_STANDALONE) for complete independence from the standard C library on the target platform.Correct operands: The DesfCommand structure was adapted to support up to three operands required for some arithmetic commands.Structure optimization: Use of union for operand fields and labels for more efficient memory usage.Documentation: Detailed comments for each DesfCommandType explaining its purpose and expected operands.Plugin support: A mechanism for registering external functions (DesfPluginFunc), allowing .desf capabilities to be extended without modifying the interpreter's core.Built-in debugger: DESF_BREAKPOINT() macro that generates an int $3 instruction (on x86) to trigger the system debugger (active only in #ifdef DESF_DEBUG_MODE mode).Basic command encryption: Example desf_encrypt_command function using a simple XOR to demonstrate the possibility of protecting commands.🛠️ c2desf Utility: Automation of ConversionTo make the process of creating .desf files convenient, the c2desf utility is being developed. Its task is to automatically analyze the input C-code and generate the corresponding .desf file (i.e., C-code with a switch-case structure in an intermediate representation, or directly a binary/text-binary .desf).c2desf Utility ArchitectureManaging External DependenciesIsolation: Third-party libraries are placed in the dependencies/ directory.Documentation:dependencies/dependencies_rule_docs.md: Explains the policy for adding, licensing requirements (preference for MIT/BSD/Apache), updating, and removing.dependencies/dependency_list.md: Dynamic list of all used libraries (name, author, GitHub, license, version, purpose). For example, uthash.h (BSD-2-Clause).🚀 Installation and Usage (Conceptual)Detailed instructions will be provided with the first releases.General approach to c2desf installation: * Clone the repository:
   git clone [https://github.com/Ferki-git-creator/c2desf/](https://github.com/Ferki-git-creator/c2desf/)
cd c2desf
Install Python dependencies (frontend):  # Recommended to use a virtual environment
python -m venv venv
source venv/bin/activate  # for Linux/macOS
# venv\Scripts\activate    # for Windows

# If Poetry is used
poetry install
# Or pip (if requirements.txt or pyproject.toml without Poetry)
# pip install -r requirements.txt
# pip install .
Build C/C++ components (backend - VM and Cython bindings):
   # If CMake is used
mkdir build && cd build
cmake ..
make
cd ..

# Or via setup_bindings.py for Cython extensions

python setup_bindings.py build_ext --inplace
Using the c2desf utility:# Basic syntax (conceptual)
c2desf input.c -o output.desf [OPTIONS]

# Example options:
# --obfuscate-level=high  # Obfuscation level
# --optimize=neon         # Optimization for NEON
# --target=embedded       # Generation for embedded systems
# --output-format=binary  # Output file format (binary/text)
# --debug                 # Include debugging information
Running .desf files with VM:# Conceptual VM launch
desf_vm output.desf
🗺️ Roadmap (Development Plan)✅ Stage 1: Foundation[x] Finalize .desf format specification (v1.0).[x] Develop a basic C-code parser on libclang (Python).[x] Implement the main set of arithmetic and control flow operations in VM (C/C++).[x] Basic .desf generator (text format).[x] Unit tests for key components.⏳ Stage 2: Feature Expansion and Optimization[ ] Implement all planned .desf commands.[ ] Advanced obfuscation algorithms (Control Flow Flattening, "muddy code").[ ] CFG optimization (dead code removal).[ ] Support for binary .desf format.[ ] Basic SIMD (NEON/AVX) support in VM.[ ] Integration tests.🎯 Stage 3: Advanced Capabilities and Stabilization[ ] Implement JIT-compiler for "hot spots" in VM.[ ] Develop GUI for AST/CFG visualization and interactive debugging.[ ] Plugin support for VM.[ ] Extended platform support (including RISC-V, mobile OS).[ ] Load tests and benchmarks.[ ] Detailed API and user documentation.🌟 Stage 4: Ecosystem and Community[ ] Create packages for easy installation (PyPI, Docker).[ ] Active community engagement, feedback processing.[ ] Consideration of commercial support models or extended features (Open Core).📜 LicenseThis project is licensed under the MIT License. See the LICENSE file for details.Third-party libraries used in the project have their own licenses, which are detailed in the dependencies/dependency_list.md file.

