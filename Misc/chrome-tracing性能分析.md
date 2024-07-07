Chrome Trace Event Format（通常称为`chrome-trace`）是一种用于记录和查看浏览器（主要是Chrome）和其他应用程序性能的JSON格式。这种格式用于Chrome的性能分析工具，如Chrome DevTools中的Performance tab和其他基于Trace Event的工具，例如`systrace`。它允许开发者收集和分析浏览器或应用程序在运行时的详细事件和性能指标。
# 格式

Chrome Trace格式主要基于一系列的“事件”对象，每个对象记录了一个特定的操作或发生的事情，这些事件对象被封装在一个大的JSON数组中。每个事件对象通常包含以下字段：

- `name`：事件的名称。
- `cat`：事件的类别，用于对事件进行分组。
- `ph`：事件的阶段（Phase），例如`B`（开始）、`E`（结束）、`X`（持续事件）等。
- `ts`：时间戳，表示事件发生的时间，单位微秒。
- `dur`：持续时间，单位微秒；
- `pid`：进程ID，表示事件发生在哪个进程中。
- `tid`：线程ID，表示事件发生在哪个线程中。
- `args`：参数，一个包含与事件相关的额外信息的对象。

示例：
```json
[
  {
    "name": "myEvent",
    "cat": "category",
    "ph": "B",
    "ts": 123456,
    "pid": 1,
    "tid": 2,
    "args": {
      "param1": "value1",
      "param2": "value2"
    }
  },
  {
    "name": "myEvent",
    "cat": "category",
    "ph": "E",
    "ts": 1234567,
    "pid": 1,
    "tid": 2,
    "args": {}
  }
]
```

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240407155148.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240407155229.png)

# 使用

要使用Chrome浏览器的`chrome://tracing`工具读取和可视化JSON格式的性能日志文件，请按照以下步骤操作：

1. **打开Chrome浏览器**：启动Google Chrome浏览器。

2. **访问`chrome://tracing`**：在浏览器的地址栏中输入`chrome://tracing`，然后按回车键。这将打开Chrome的跟踪工具界面。

3. **加载性能日志文件**：
   - 在`chrome://tracing`界面的左上角，点击`Load`按钮。
   - 在弹出的文件选择对话框中，找到并选择你的性能日志文件（例如，之前提到的`performance_log.json`）。
   - 点击`Open`来加载文件。

4. **查看和分析性能数据**：
   - 文件加载完成后，你的性能数据将在`chrome://tracing`界面中可视化显示。
   - 你可以通过拖动和缩放来查看不同的时间段和事件详情。
   - 工具提供了多种视图和过滤选项，以帮助你分析性能数据。

5. **使用高级功能**：
   - `chrome://tracing`工具还提供了一些高级功能，如选择特定的线程或事件进行查看，使用搜索功能找到特定的事件，以及设置时间范围过滤器等。
   - 你可以通过界面上的按钮和输入框来访问这些功能。

通过这些步骤，你可以利用Chrome的`chrome://tracing`工具来读取和分析你的C++程序性能日志文件。这个工具提供了一个强大且直观的方式来理解程序的运行时性能特征，帮助你识别瓶颈和优化机会。