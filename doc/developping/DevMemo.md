# Designe
## Concept
```puml
===Init phase====
user   -> vscode:	start debug
vscode -> QEMU: 	run QEMU
vscode -> GDB: 		run GDB
GDB    -> QEMU: 	attach
(optional)
GDB    -> QEMU: 	run initial GDB script (.gdbinit, -x some_gdbscript.txt)

===Debugging phase====
' break
user   -> vscode: 	set breakpoint
vscode -> GDB: 		set breakpoint via DAP(debug adapter protocol)
GDB    -> QEMU: 	set breakpoint

' run
user   -> vscode: 	run(or, step in/out, continu)
vscode -> GDB: 		run(or, step in/out, continu) via DAP
GDB    -> QEMU: 	run(or, step in/out, continu)

' watch
user   -> vscode: 	watch
vscode -> GDB: 		watch via DAP
GDB    -> QEMU: 	watch
GDB    <- QEMU: 	watch value
vscode <- GDB: 		watch value via DAP
user   <- vscode: 	watch
```

```
vscode -> GDB: 
```

### entry point 
cmd -> src/extension.ts
dbg -> src/qemu_gdb.ts

### file structure
```
src/
	debugger/
		qemu_gdb.ts
		gdb/
			backend.ts
			gdb_expansion.ts
			mi_parse.ts
			mi2.ts
			mibase.ts
```

### glossary
- MI2
	- GNU Debugger Machine Interface
	- gdb を実行するとterminal的なIFになるが、そのIFのこと(?)
	- ref
	Debugging with GDB - gdb/miインターフェイス
https://www.asahi-net.or.jp/~wg5k-ickw/html/online/gdb-5.0/gdb-ja_20.html

## Understanding DAP

# Environment
- OS: Windows 10 (2004)
- QEMU: 
- RTOS:
	- FreeRTOS
		- target: Demo\CORTEX_M0+_LPC51U68_GCC_IAR_KEIL
		- 


## Reference
### QEMU
QEMU
https://www.qemu.org/

### GDB
GDB Documentation
https://www.gnu.org/software/gdb/documentation/

### Debug Adapter Protocol
Official page for Debug Adapter Protocol
https://microsoft.github.io/debug-adapter-protocol/

### vscode extension API
VS Code API | Visual Studio Code Extension API
https://code.visualstudio.com/api/references/vscode-api

# TODO
## MUST
## WANT
