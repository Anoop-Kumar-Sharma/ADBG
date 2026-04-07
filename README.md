# 🛡️ ADBG — Advanced Anti-Debugging & Syscall Library

ADBG is a low-level Windows anti-debugging and anti-analysis library built using NT internals, direct syscalls, and execution-level tricks.

It is designed to detect, disrupt, and prevent:

* User-mode debuggers
* Kernel-assisted debugging
* Dynamic analysis tools
* Instrumentation frameworks
* Runtime attach attempts

---

## ⚙️ Core Capabilities

### 🔍 Debugger Detection

* PEB checks (`BeingDebugged`, `NtGlobalFlag`)
* Debug object & debug port detection
* Remote debugger detection
* Kernel debugger detection (`NtSystemDebugControl`)
* Process heap flags / force flags
* Parent process heuristics

---

### 🧠 Breakpoint Detection

* Hardware breakpoints (DR0–DR7)
* Page exception breakpoints
* Guard page execution checks
* Stack read anomalies
* Continuous debug register validation

---

### ⚡ Direct Syscalls Engine

* Hash-based syscall resolution
* Dynamic export parsing from `ntdll`
* Cross-compiler syscall stubs (MSVC / GCC)
* SysWhispers-style implementation

```c
DbgNtCreateThreadEx(...);
DbgNtQuerySystemInformation(...);
```

---

### 🧵 Thread & Execution Control

* Native thread creation (`NtCreateThreadEx`)
* Hidden threads (`ThreadHideFromDebugger`)
* Thread enumeration & manipulation
* Debug register sanitization across threads

---

### 🧬 Anti-Attach Protection (Advanced)

One of the strongest parts of this library.

#### ✔ TLS Callback Protection

* Executes before `main()`
* Detects debugger attach via thread injection

#### ✔ `DbgUiRemoteBreakin` Detection

* Scans thread start addresses
* Terminates if debugger attach thread is detected

#### ✔ Runtime Patching

* Hooks:

  * `DbgUiRemoteBreakin` → redirected to `__fastfail`
  * `DbgBreakPoint` → patched to `ret`

#### ✔ Hardware Breakpoint Reset

* Enumerates all threads
* Clears DR registers across process

---

### 🚨 Exception-Based Detection

* Vectored exception handler (`handler.c`)
* Detects:

  * Single-step exceptions
  * Hardware breakpoint usage
  * Hooked `KiUserExceptionDispatcher`

```c
LONG CALLBACK VectoredDebuggerCheck(...)
```

---

### 🧠 Memory & Integrity

* `MEM_WRITE_WATCH` detection
* Module CRC verification (`hasher.c`)
* Hook detection via memory inspection
* Stack integrity validation

---

### 🖥️ Environment Detection

* Debugger window detection (x64dbg, WinDbg, etc.)
* Job object detection (sandboxing)
* Suspicious parent process detection
* Duplicate handle anomalies

---

### ⏱️ Timing & Behavior

* RDTSC timing checks
* Execution delay detection
* Anti-instrumentation heuristics

---

## 📂 Project Structure

```
/core
  syscall.*, syscall-core.asm, syscalls.h
  syscall_converter.py

/anti_debug
  dbgpresent.*, dbgobj.*, procdbgport.*, rdbgpresent.*
  ntglobalflag.*

/breakpoints
  hwbreakp.*, hwbreakp2.*
  membreakp.*, pgexcbp.*
  raiseexc.*

/process
  prochpflag.*, prochpforceflag.*
  duphnd.*, prothnd.*, clshandle.*
  opnproc.*, job.*, prntproc.*

/execution
  thrmng.*, handler.*
  atcptr.*   ← TLS + anti-attach core

/system
  kerneldbg.*, sysdbgctl.*
  window.*, timing.*, ntldt.*

/memory
  vrtalloc.*, readstck.*
  hasher.*, loadlib.*

/firmware
  dbgp.c  ← ACPI DBGP detection
```

---

## 🚀 Usage

### Initialize Protection (Recommended)

```c
#include "adbg.h"

int main() {
    StartDebugProtection();   // continuous monitoring
    StartAttachProtection();  // anti-attach hardening
}
```

---

### Manual Checks

```c
if (CheckOpenProcess()) return -1;
if (ProtectedHandle()) return -1;
if (NtSystemDebugControl()) return -1;
```

---

### Hidden Thread Example

```c
HANDLE hThread = DbgCreateThread(
    GetCurrentProcess(),
    0,
    MyThread,
    NULL,
    0,
    NULL,
    NULL
);
```

---

## ⚡ Design Philosophy

### 1. Syscall-first

Avoid user-mode API hooks entirely.

### 2. Multi-vector detection

No single bypass → attacker must defeat many layers.

### 3. Early execution

TLS callbacks trigger before main logic.

### 4. Fail-fast

Uses `__fastfail()` to immediately terminate.

---

## ⚠️ Limitations

* Windows-only (NT-based)
* Some techniques may trigger:

  * AV / EDR alerts
  * Sandbox detections
* Syscall numbers must match OS version

---

## 🔐 Disclaimer

This project is intended for:

* Software protection
* Anti-reverse engineering research
* Security education

Do not use for malicious purposes.

---

## 📌 Future Work

* Hypervisor detection
* ETW bypass
* AMSI bypass
* Kernel-mode extension
* Anti-VM heuristics

---

## 🧪 Debug Mode

Compile with `_DEBUG`:

```
[!] Debugger detected in: <function>
```

---

## 📜 License

MIT / Custom
