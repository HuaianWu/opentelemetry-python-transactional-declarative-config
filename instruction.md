"我们通过 OTEL_CONFIG_FILE 在进程启动时一次性配置 traces、metrics、logs 和 propagator。现在只要靠后的 signal 配置有误，前面创建的 provider 就可能已经注册为全局对象，已经启动的 batch processor 或 metric reader 线程也会继续存活；修正配置后在同一进程里重试，还会因为全局 provider 只能设置一次而得到一个无法恢复的半配置状态。请让声明式 SDK 配置的应用具备全有或全无语义。

在任何进程级状态对应用可见之前，整份配置以及它引用的内置组件和第三方 entry point 都应完成构造与校验。任一阶段失败时，已经创建的 processor、reader、exporter 和 provider 必须按正确顺序释放且只释放一次，不能遗留工作线程、网络会话、文件句柄、logging handler 或部分替换的 propagator，随后用修正后的配置重试应能正常工作。成功时各 signal 应作为同一次配置切换生效；未配置的 section、disabled 行为以及现有的一次性全局 provider 约定保持不变。

现有内置传输和通过 entry point 加载的扩展都要保持兼容，包括会启动后台线程或打开外部资源的 processor、reader 和 exporter。请补充同一 signal 内部失败、跨 signal 失败、并发调用和失败后重试的测试，并在声明式配置文档中说明失败时的可见性和清理行为。"
