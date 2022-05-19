<div align='center'>

  <img src='https://user-images.githubusercontent.com/66398400/169051355-edd50385-e66d-4255-a6ef-e60581b30019.gif'/>

# Rust Memory Layout

Describing the Memory Layout in Rust Language

<br><br><br>

</div>

<table width="100%">
  <tr>
  <td width="20%" align=center>

<h1>Kernel Virtual Memory Space Overview</h1>

<kbd>
 <h3 align=center>&nbsp;&nbsp;&nbsp;HIGH MEMORY ADDRESS&nbsp;&nbsp;&nbsp;</h3>
</kbd>

...

| <h3>[STACK](#stack)<br>⟱</h3>                                            |
| ------------------------------------------------------------------------ |
| <div align=center>**main()**</div>                                       |
| <img width=200><br><br><div align=center>free memory</div><br><br></img> |
| <h3 align=center>⟰<br>[HEAP](#heap)</h3>                                 |
| <div align=center>**Block Started by Symbol**</div>                      |
| <div align=center>**Data Segment**</div>                                 |
| <div align=center>**Text Segment**</div>                                 |

...

<kbd>
 <h3 align=center>&nbsp;&nbsp;&nbsp;LOW MEMORY ADDRESS&nbsp;&nbsp;&nbsp;</h3>
</kbd>

  </td>

  <td width="80%">

# [Memory Address Range](#memory-address-range)

### Basic x86-64 Linux ELF Binary

- The memory address range is bounded the the `word` size of the `CPU`.
- In a **64-bits** processor the `word` size is **64/8 or 8 bytes**.
- A **32-bits** processor can only address up to **2^32 or ~4GB** of [byte-addressable](https://en.wikipedia.org/wiki/Byte_addressing) memory. On the other hand **64-bits** processor can address from **0 to 2^(64-1) or 16 billion GB**. Filling up the total memory is equivalent to turn every **0 bit into 1**
- On **64-bits** `CPU` only **48-bits or ~281TB** is used for memory addressing with the remaining **16 bits** of the virtual address required to be all 0's or all 1's.
- Using the above as example only **1-bit or ~141T** is used for `kernelspace` and the rest is used for `userspace` memory.

# [Text Segment](#text-segment)

- The text segment, aka code segment, is where the Rust code is compiled by LLVM into machine code and then execute instructions

# [Data Segment](#data-segment)

- The data segment contains the initalized variables

# [BSS](#bss)

- The Block Started by Symbol (BSS) contains the unitialized variables

  </td>
  </tr>
</table>

🦀 **Depicting the memory allocation:**

1.  A `Stack Frame` is created and a `pointer` **increases by 8 bytes (for a 64-bit OS)** to keep track of the running program. The `main()` has the variable `a` which is a `i32` so its `4 bytes` in size to be reserved in its `Stack Frame`.

2.  Next the variable `b` calls the function `double()` which creates another `Stack Frame` and the `pointer` **increases 8 bytes** again.

3.  In `double()` function a parameter `n` is a `i32` so it needs another `4 bytes` to be added on its own `Stack Frame`. The `double()` function run and returns a value. The `double()` function has a **return address to be stored** in the variable `b` inside `main()` and its a `i32` so another `4 bytes` again. The function is terminated and the **stack pointer decreases 8 bytes** deallocating `double()` to be **overwritten by another function in future**.

4.  The variable `b` receives the value returned from `double()` and the `main()` function ends and the whole program terminates.

<br><br><br>

# The [STACK](#stack)

> SAMPLE FUNCTION

```rust
fn main() {
  let a = 48;
  let b = double(a);
  println!("{b}");
}

fn double(n: i32) -> i32 {
  n * 2
}
```

<table width="100%">
  <tr>
  <td width="20%" align=center>

| <h3>[STACK](#stack) (8MB)<br>⟱</h3>                                                                                                   |
| ------------------------------------------------------------------------------------------------------------------------------------- |
| <div align=center>**main()**<br><br><kbd>&nbsp;**a = 48**&nbsp;</kbd><br><kbd>&nbsp;**b = 96**&nbsp;</kbd></div>                      |
| <div align=center>**double()**<br><br><kbd>&nbsp;**n = 48**&nbsp;</kbd><br><kbd>&nbsp;**fn return addr 0x12f = 96**&nbsp;</kbd></div> |
| <img width=200><br><br><div align=center>free memory</div><br><br></img>                                                              |
| <h3 align=center>⟰<br>[HEAP](#heap)</h3>                                                                                              |

  </td>

  <td width="80%">

# [STACK](#stack) Properties

- The `stack` is a **static memory** that requires a **fixed known size** at compile time and is stored inside the binary after compilation.
- The `stack` memory **grows downwards** starting from a [higher address](#high-memory-address) of around `0x7fffffffffff`.
- It is an abstraction that runs processes on almost every computer.
- Each process starts a **single thread by default** and each process has it's **own separate stack**.
- Rust `stack` size limit is **8MB** in a **64-bit** system but the default **start size is 2MB**.
- If your executition exceeds the stack limit you will be confronted with a **"stack overflow"** compile error.
- The stack **pointer size** is determined its **data type**. In this case `i32` needs **4 bytes**.
- Because of it's **static** nature it doesn't need any additional step to change it's size using system calls which **speeds up the execution**.

  </td>
  </tr>
</table>

🦀 **Depicting the memory allocation:**

1.  A `Stack Frame` is created and a `pointer` **increases by 8 bytes (for a 64-bit OS)** to keep track of the running program. The `main()` has the variable `a` which is a `i32` so its `4 bytes` in size to be reserved in its `Stack Frame`.

2.  Next the variable `b` calls the function `double()` which creates another `Stack Frame` and the `pointer` **increases 8 bytes** again.

3.  In `double()` function a parameter `n` is a `i32` so it needs another `4 bytes` to be added on its own `Stack Frame`. The `double()` function run and returns a value. The `double()` function has a **return address to be stored** in the variable `b` inside `main()` and its a `i32` so another `4 bytes` again. The function is terminated and the **stack pointer decreases 8 bytes** deallocating `double()` to be **overwritten by another function in future**.

4.  The variable `b` receives the value returned from `double()` and the `main()` function ends and the whole program terminates.

<br><br><br>

# The [HEAP](#heap)

> SAMPLE FUNCTION

```rust
fn main() {
  let heap = boxed_value();
  println!("{heap}");
}

fn boxed_value() -> Box<i32> {
  let result = Box::new(99);
  result
}
```

<table width="100%">
  <tr>
  <td width="20%" align=center>

| <h3>[STACK](#stack) (8MB)<br>⟱</h3>                                                                                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------ |
| <div align=center>**main()**<br><br><kbd>&nbsp;**heap = 0x6f32**&nbsp;</kbd></div>                                                               |
| <div align=center>**boxed_value()**<br><br><kbd>&nbsp;**result = 99**&nbsp;</kbd><br><kbd>&nbsp;**fn return addr 0x6f32 (99)**&nbsp;</kbd></div> |
| <img width=200><br><br><div align=center>free memory</div><br><br></img>                                                                         |
| <div align=center><kbd>&nbsp;**heap addr 0x6f32 (99)**&nbsp;</kbd></div>                                                                         |
| <h3 align=center>⟰<br>[HEAP](#heap)</h3>                                                                                                         |

  </td>

  <td width="80%">

# [HEAP](#heap) Properties

- The `heap` is another abstraction that, unlike `stack`, has a **flexible size** that can change over time (like in **run time**).
- Instead of one `Stack Frame` for each thread the `heap` memory region has a large **shared memory with the `stack`**.
- The `heap` memory **grows upwards** starting from the [lower address](#low-memory-address).
- The `heap` is **not stored** in the binary and is discarded after the program execution.
- Its size limit is bounded the system's memory.
- The `heap` **value size** is determined by its **data type**.
- Because of it's **dyanmic** nature it needs to make system calls to ask for more space as soon as requested thus adding a little overhead. This turns `heap` slower than `stack` memory.

  </td>
  </tr>
</table>

🦀 **Depicting the memory allocation:**

1.  The `main()` function creates a new `Stack Frame`. The **stack pointer increases 8 bytes** and the variable `heap` calls the function `boxed_value()`.

2.  In the `boxed_value()` the **stack pointer increses 8 bytes again** and we have a local variable called `result` that has a `Box` with a value inside. The **`Box` acts as a pointer** so the return value is **8 bytes** (the `heap` variable in `main()` will be the same **8 bytes but as an address to the `Heap Memory`**). Inside the `Box` we have a value `99` which is `i32` so we need to reserve **4 bytes but this time on the heap**. The function ends, **deallocates and decrases the stack pointer by 8 bytes**.

3.  The `heap` variable in `main()` function now receives a **copied address** of the value stored in the other memory region, the `Heap Memory`. Then the funtion runs and terminates the program.
