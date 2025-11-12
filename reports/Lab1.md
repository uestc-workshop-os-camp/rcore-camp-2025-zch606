### 实现功能

>该实验完成可以简单分为两部分，一部分是对`Task`模块的更改，一部分是`syscall_trace`的实现。

##### `Task`结构的完善

在`TaskControlBlock`结构内加上`pub task_syscall_count: [usize;MAX_SYSCALL_NUM]`用来记录该任务内各个系统调用的次数。
并在`task/mod.rs`内为`TaskManager`实现我们接下来使用的“get, add系统调用次数”的两个方法，并封装成函数`pub fn current_add_syscall_count(syscall_id: usize) `,
`pub fn current_get_syscall_count(syscall_id: usize) -> Option<usize> `,供后续使用。

#####  `sys_trace`实现

利用上述封装的函数，我们按要求实现`sys_trace`，按需调用`current_get_syscall_count`即可，并在`syscall`函数中调用`current_add_syscall_count(syscall_id: usize)`来增加调用次数


### 简答作业

###### 1.出错行为

 **SBI**: RustSBI version 0.3.0-alpha.2

```
[kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x804003a4, kernel killed it.
[kernel] IllegalInstruction in application, kernel killed it.
[kernel] IllegalInstruction in application, kernel killed it.

```

- **ch2b_bad_address**: 
  - 错误行为：访问非法内存地址，触发 Load/Store Page Fault 异常
- **ch2b_bad_instruction**: 
  - 错误行为：在用户态执行 `sret` 指令（S 态特权指令），触发 Illegal Instruction 异常，进入 trap handler
- **ch2b_bad_register**: 
  - 错误行为：在用户态访问 `sstatus` 寄存器（S 态 CSR），触发 Illegal Instruction 异常

###### 2. `__alltraps` 和 `__restore`

1.刚进入 `__restore` 时，`sp` 代表了什么值。请指出 `__restore` 的两种使用情景?

-   **sp 的值**: 指向当前任务的内核栈，该位置存储着 TrapContext 结构
-   **__restore 的两种使用情景**： 任务第一次开始执行：从任务初始化时的 TrapContext 恢复； 从中断/异常返回：从 trap handler 返回到用户态

2.处理的三种寄存器?

- **sstatus**: 保存处理器的状态信息。意义：SPP 位决定了执行 `sret` 后进入的特权级
- **sepc**: 保存发生异常时的程序计数器。意义：用于返回到用户态继续执行
- **sscratch**: 存储栈指针。意义：用于在用户态和内核态之间交换 sp 寄存器。

3.为何跳过?

- x2 (sp): 栈指针需要特殊处理，不能在这里直接恢复，因为当前正在使用内核栈
- x4 (tp): 线程指针通常在内核初始化后保持不变，不需要在任务切换时恢复

4.`csrrw sp, sscratch, sp`后?

- sscratch->kernel stack, sp->user stack

5.`__restore`：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？

- `sret`指令切换回用户态
- `sret` 指令根据 `sstatus` 寄存器的 SPP 字段决定返回的特权级
- 在 __restore 中我们已经将 `sstatus` 设置为用户态
- 同时 `sret` 会将 `sepc` 的值加载到 pc，开始执行用户态代码




6.L13:该指令之后，`sp` 和 `sscratch` 中的值分别有什么意义？

-  sp->kernel stack, sscratch->user stack   (完成用户栈与内核栈的切换)


7.从 U 态进入 S 态是哪一条指令发生的？
	`ecall`

