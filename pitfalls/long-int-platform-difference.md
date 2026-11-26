[← C++ Field Guide](../README.md)

## long int can be 4 or 8 bytes on 64-bit platform

Operating systems are using different data models, despite the same bit system. The size of `long` type can be mistakenly attributed to 64-bit systems, but that is not the case. See [Data models at cppreference.com](https://en.cppreference.com/cpp/language/types#Data_models):

> **Data models**
>
> The choices made by each implementation about the sizes of the fundamental types are collectively known as data model. Four data models found wide acceptance:
> 
> 32 bit systems:
> - LP32 or 2/4/4 (int is 16-bit, long and pointer are 32-bit)
>   - Win16 API
> - ILP32 or 4/4/4 (int, long, and pointer are 32-bit);
>   - Win32 API
>   - Unix and Unix-like systems (Linux, macOS)
> 
> 64 bit systems:
> - LLP64 or 4/4/8 (int and long are 32-bit, pointer is 64-bit)
>   - Win32 API (also called the Windows API) with compilation target 64-bit ARM (AArch64) or x86-64 (a.k.a. x64)
> - LP64 or 4/8/8 (int is 32-bit, long and pointer are 64-bit)
>   - Unix and Unix-like systems (Linux, macOS)
> 
> Other models are very rare. For example, ILP64 (8/8/8: int, long, and pointer are 64-bit) only appeared in some early 64-bit Unix systems (e.g. UNICOS on Cray). 

This means that knowing your code runs on 64-bit system is not enough to rely on `long` being consistently 4 or 8 bytes. This can cause the same code to produce different results when compiled for 64-bit Windows (MSVC) and 64-bit Linux/macOS (GCC, Clang).
