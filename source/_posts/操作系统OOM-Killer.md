---
title: 操作系统OOM Killer
date: 2024-06-12 10:43:34
tags:
categories:
---

# 1. OOM Killer 是什么？

OOM Killer（Out of Memory Killer）是 Linux 操作系统内核中的一个机制，用于在系统内存耗尽时，通过终止进程来释放内存资源，从而防止系统崩溃或无法响应。

## 2. OOM Killer 的触发条件

当系统的物理内存和交换空间（swap）都被完全占用，且内核无法再分配内存时，就会触发 OOM（Out of Memory）状态。在这种情况下，系统必须释放一些内存，否则整个系统将无法继续运行。

# 3 OOM Killer 的功能

OOM Killer 的主要功能是在系统内存耗尽时，通过终止某些进程来释放内存。这是一个最后的手段，目的是保证系统的基本功能和响应能力。

# 4. OOM Killer 的选择机制

OOM Killer 选择进程时，会根据一个评分系统来决定终止哪个进程。评分越高的进程，被终止的可能性越大。评分主要基于以下几个因素：

**进程使用的内存量**：使用内存越多的进程，评分越高。

**进程的重要性**：系统关键进程和低优先级进程的评分会有所不同。重要的系统进程一般有较低的评分。

**进程的优先级（OOM调整值）**：可以手动设置进程的优先级，调整它在 OOM Killer 眼中的重要性。

# 5. 4. OOM Killer 的日志信息

当 OOM Killer 终止一个进程时，会在系统日志（通常是 `/var/log/syslog` 或 `/var/log/messages`）中记录一条信息，如你提供的日志条目：

```java
May 28 17:34:42 sith-mom-test-node kernel: Out of memory: Kill process 9281 (java) score 75 or sacrifice child
```

