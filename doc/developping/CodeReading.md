# Native debug
WebFreak001/code-debug: Native debugging for VSCode
https://github.com/WebFreak001/code-debug

## file structure
```shell
.
├── backend
│   ├── backend.ts		 # !
│   ├── gdb_expansion.ts # !
│   ├── linux
│   │   └── console.ts
│   ├── mi2
│   │   ├── mi2.ts		# !
│   │   ├── mi2lldb.ts
│   │   └── mi2mago.ts
│   └── mi_parse.ts
├── frontend
│   └── extension.ts	 # ! コマンド宣言とか
├── gdb.ts				  # ! Debug sessionの管理(init, launch, attach)
├── lldb.ts
├── mago.ts
├── mibase.ts
└── test
    ├── runTest.ts
    └── suite
        ├── gdb_expansion.test.ts
        ├── index.ts
        └── mi_parse.test.ts
```

```puml
MI2DebugSession ←has- MI2<ここでDAPを介してGDBとやり取りしている
↑abstruct
GDBDebugSession
	initializeRequest
	launchRequest
	attachRequest
```


### 大事そうなところ
mi2.tsのMI2クラスで実装されているのが、
```typescript
   this.process.on("exit", (() => { this.emit("quit"); }).bind(this));			
```
file: src/backend/mi2/mi2.ts, line:62-63

mibase.tsのMI2DebugSession class内でMI2インスタンスに対してbindされている

```typescript
  this.miDebugger.on("quit", this.quitEvent.bind(this));
```
(file: src/mibase.ts, line:48)
		

file: src/mibase.ts
line:25-59
code:
```typescript
export class MI2DebugSession extends DebugSession {
	protected variableHandles = new Handles<string | VariableObject | ExtendedVariable>(VAR_HANDLES_START);
	protected variableHandlesReverse: { [id: string]: number } = {};
	protected useVarObjects: boolean;
	protected quit: boolean;
	protected attached: boolean;
	protected needContinue: boolean;
	protected isSSH: boolean;
	protected trimCWD: string;
	protected switchCWD: string;
	protected started: boolean;
	protected crashed: boolean;
	protected debugReady: boolean;
	protected miDebugger: MI2;
	protected commandServer: net.Server;
	protected serverPath: string;

	public constructor(debuggerLinesStartAt1: boolean, isServer: boolean = false) {
		super(debuggerLinesStartAt1, isServer);
	}

	protected initDebugger() {
		this.miDebugger.on("launcherror", this.launchError.bind(this));
		this.miDebugger.on("quit", this.quitEvent.bind(this));
		this.miDebugger.on("exited-normally", this.quitEvent.bind(this));
		this.miDebugger.on("stopped", this.stopEvent.bind(this));
		this.miDebugger.on("msg", this.handleMsg.bind(this));
		this.miDebugger.on("breakpoint", this.handleBreakpoint.bind(this));
		this.miDebugger.on("step-end", this.handleBreak.bind(this));
		this.miDebugger.on("step-out-end", this.handleBreak.bind(this));
		this.miDebugger.on("step-other", this.handleBreak.bind(this));
		this.miDebugger.on("signal-stop", this.handlePause.bind(this));
		this.miDebugger.on("thread-created", this.threadCreatedEvent.bind(this));
		this.miDebugger.on("thread-exited", this.threadExitedEvent.bind(this));
		this.sendEvent(new InitializedEvent());
```

ここらへんはおまじないかな？

file: src/gdb.ts
line:195
code:
```typescript
DebugSession.run(GDBDebugSession);
```

## 気づいた点
```puml
DebugSession デバッグのセッションを管理するクラス
Runtime デバッグ対象のプロセス

DebugSession <|- Runtime

DebugSession 
	// 
	// stop系
	stopOnEntry -> entry
	stopOnStep -> step
	stopOnBreakpoint -> breakpoint
	stopOnDataBreakpoint -> data breakpoint
	stopOnException -> exception
	// 出力系
	output
	// 終了系
	end
	// その他
	breakpointValidated
```

```puml
initalizedRequest
↓
configrationDoneRequest
↓
launchRequest
```

```puml
User -> DebugSession :*Request
DebugSession -> Runtime: any action(e.g. continue, step, stepIn)
User <- DebugSession :sendResponse
```

ここらへんから

```typescript
  // setup event handlers
		this._runtime.on('stopOnEntry', () => {
			this.sendEvent(new StoppedEvent('entry', QemuGdbDebugSession.threadID));
		});
		// User => vscode => nextRequest => 
		this._runtime.on('stopOnStep', () => {
			this.sendEvent(new StoppedEvent('step', QemuGdbDebugSession.threadID));
		}); 
		this._runtime.on('stopOnBreakpoint', () => {
			this.sendEvent(new StoppedEvent('breakpoint', QemuGdbDebugSession.threadID));
		}); // 
```
(file: src/qemuGgbDebugSession.ts, line:67-76)

