# My-Mempool: High-Performance Concurrent Memory Pool

**My-Mempool** 是一个基于 C++ 实现的高性能并发内存池。它专为高并发场景设计，结合了 **哈希桶（Hash Bucket）** 分级管理和 **无锁（Lock-free）** 数据结构，旨在显著减少频繁申请/释放小块内存带来的系统开销（Syscall）和内存碎片问题。

## ✨ 核心特性

* **⚡ 高并发友好**: 核心的空闲链表（Free List）操作采用 CAS (Compare-And-Swap) 原子操作实现无锁化（Lock-free），在多线程环境下极大降低了锁竞争。
* **📦 哈希桶分级管理**: 采用 `HashBucket` 策略，将内存块按大小（8字节为单位）对齐。支持 8B 到 512B 的小对象快速分配，减少内存碎片。
* **🔄 自动扩容**: 当内存池耗尽时，自动向操作系统申请新的大块内存（Block），并通过互斥锁（Mutex）保证扩容时的线程安全。
* **🛡️ 智能回退机制**: 对于超过 512字节 的大对象，自动回退到原生 `new`/`delete`，保证通用性。
* **🚀 易用的模板接口**: 提供类似 `new` 的模板函数 `newElement<T>()` 和 `deleteElement<T>()`，自动处理对象构造与析构。

## 🏗️ 架构设计

1.  **HashBucket**: 内存池的顶层入口。它管理着 64 个 `MemoryPool` 实例（`MEMORY_POOL_NUM`），分别对应 8B, 16B, ..., 512B 的内存槽大小。根据用户请求的 size，通过哈希算法映射到对应的 Pool 中。
2.  **MemoryPool**: 具体的内存管理单元。
    * **FreeList**: 一个原子链表，存储被释放回来的空闲槽位。优先从此分配，使用 CAS 操作保证原子性。
    * **Block List**: 向 OS 申请的大块内存链表。当 FreeList 为空且当前 Block 用完时，申请新的 Block。
3.  **Slot**: 内存分配的最小单元。

## 🛠️ 构建与运行

本项目基于 C++11 标准。

### 环境要求
* C++11 或更高版本编译器 (GCC/Clang/MSVC)
* Linux / Windows / macOS

### 编译示例 (Linux/Git Bash)

假设当前处于项目根目录：

```bash
# 编译测试程序
g++ -o mempool_test v1/tests/UnitTest.cpp v1/src/MemoryPool.cpp -I v1/include -lpthread -std=c++11

# 运行测试
./mempool_test
