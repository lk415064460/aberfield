#### iOS崩溃分析

+ 软件异常来源于几个API：kill()，pthread_kill(), 平时开发中遇到的NSException未捕获，abort()函数调用等，都属于这种情况。比如存在pthread_kill方法调用。
+ 硬件异常：硬件产生的信号始于处理器 trap，处理器 trap 是平台相关的。比如我们遇到的野指针崩 溃大部分是硬件异常。
+ Mach异常：这里虽然叫异常，但是要和上面的两种分开来看。我们了解到苹果的内核 xnu 的核心是 Mach , 在 Mach 之上建立了 BSD 层。“Mach 异常” 是 “Mach 异常处理流程” 的简称。
+ UNIX信号：这就是我们常说的信号了，如 SIGBUS SIGSEGV SIGABRT SIGKILL 等。
