---
layout: page
title: "Dependency Inversion Principle"
permalink: /solid/dip
---


## Definition
1. High-level modules should not depend on low-level modules. Both should depend on abstractions.
2. Abstractions should not depend on details. Details should depend on abstractions.

## Example
Consider a File class that needs to perform I/O operations:
### Without DIP (tightly coupled):

    class File {
        fun write(data: String) {
            // Directly depends on concrete implementation
            val fd = SystemCalls.open("/path/file")
            SystemCalls.write(fd, data)
            SystemCalls.close(fd)
        }
    }

Problem: File class is tightly coupled to system calls. Can't test without real file operations. Can't reuse on different platforms.

### With DIP (loosely coupled):

    // Abstraction (owned by high-level module)
    interface IODriver {
        fun open(path: String): Int
        fun write(handle: Int, data: ByteArray): Int
        fun close(handle: Int)
    }
    
    // High-level module depends on abstraction
    class File(private val driver: IODriver) {
        fun write(data: String) {
            val handle = driver.open(path)
            driver.write(handle, data.toByteArray())
            driver.close(handle)
        }
    }
    
    // Low-level module implements abstraction
    class SystemIODriver : IODriver {
        override fun open(path: String) = // OS-specific implementation
        override fun write(handle: Int, data: ByteArray) = // System calls
        override fun close(handle: Int) = // Platform details
    }
    
    // Usage - injection at runtime
    val file = File(SystemIODriver())  // Production
    val testFile = File(MockIODriver()) // Testing


```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
````
## The Dependency Flow Reversal
### Traditional flow (without DIP):
    File → SystemIODriver → OS System Calls (high-level depends on low-level)

### Inverted flow (with DIP):
    File → IODriver (interface)
             ↑
    SystemIODriver → OS System Calls

## Polymorphism
Polymorphism is the key mechanism that makes DIP work. The `File` class doesn't know or care which concrete `IODriver`
