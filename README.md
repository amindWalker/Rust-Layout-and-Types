
<div align='center'>

  <img src='https://user-images.githubusercontent.com/66398400/169051355-edd50385-e66d-4255-a6ef-e60581b30019.gif'/>
  
  # Rust-Memory-Layout
  Describing the Memory Layout in Rust Language

</div>

<div align=center>

  ## Basic x86-64 Linux ELF-64 Binary Format
  
<kbd>

<h1 align=center>&nbsp;&nbsp;&nbsp;Kernel Virtual Memory Space Overview&nbsp;&nbsp;&nbsp;</h1>
    
  |<h3 align=center>HIGH MEMORY ADDRESS</h3>
  |--
    
  ...
    
  |<h3>STACK<br>⟱</h3>|
  |--
  |<div align=center>**main()**</div>
  |<div align=center>**double()**</div>
  |<img width=300 /><br><br><br><br><br><br><br><br><br><br>|
  |<h3 align=center>⟰<br>HEAP</h3>|
  |<h3 align=center>Block Started by Symbol <br>(BSS)</h3>|
  |<h3 align=center>Data Segment <br>(variables)</h3>|
  |<h3 align=center>Text Segment <br>(your code)</h3>|
  
  ...
    
  |<h3>LOW MEMORY ADDRESS</h3>|
  |--
  
  </div>

</kbd>

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
  <td width="50%">
  <div align=center>

<kbd>

<h1 align=center>&nbsp;&nbsp;&nbsp;Stack (8MB)&nbsp;&nbsp;&nbsp;</h1>
    
  |<h3>STACK<br>⟱</h3>|
  |--
  |<div align=center><h3>main()</h3><h3><kbd>&nbsp;a = 48&nbsp;</kbd></h3></div>
  |<div align=center><h3>double()<h3><h3><kbd>&nbsp;i = 48&nbsp;</kbd></h3><h3><kbd>&nbsp;return address = 0x12f&nbsp;</kbd></h3></div>
  |<img width=200 /><br><br><br><br><br><br><br><br><br><br>|
  
</kbd>

</div>
  
  </td>
    
  <td width="50%">
    
# [STACK](#stack) Properties
* The `stack` is a **static memory** that requires a **fixed known size** at compile time and is stored inside the binary after compilation. 
* The `stack` memory **grows downwards** starting from a [higher address](#high-memory-address) of around `0x7fffffffffff`.
* It is an abstraction that runs processes on almost every computer. 
* Each process starts a **single thread by default** and each process has it's **own separate stack**. 
* Rust `stack` size limit is **8MB** in a **64-bit** system but the default **start size is 2MB**. 
* If your executition exceeds the stack limit you will be confronted with a **"stack overflow"** compile error.
* The stack **pointer size** is determined its **data type**. In this case `i32` needs **4 bytes**.
* Because of it's **static** nature it doesn't need any additional step to change it's size using system calls which **speeds up the execution**.
    
  </td>
  </tr>
</table>
 
 🦀 **Depicting the memory allocation:**
  
 1. A `Stack Frame` is created and a `pointer` **increases by 8 bytes (for a 64-bit OS)** to keep track of the running program. The `main()` has the variable `a` which is a `i32` so its `4 bytes` in size to be reserved in its `Stack Frame`.
 
 2. Next the variable `b` calls the function `double()` which creates another `Stack Frame` and the `pointer` **increases 8 bytes** again.
 
 3. In `double()` function a parameter `n` is a `i32` so it needs another `4 bytes` to be added on its own `Stack Frame`. The `double()` function run and returns a value. The `double()` function has a **return address to be stored** in the variable `b` inside `main()` and its a `i32` so another `4 bytes` again. The function is terminated and the **stack pointer decreases 8 bytes** deallocating `double()` to be **overwritten by another function in future**.
 
 4. The variable `b` receives the value returned from `double()` and the `main()` function ends and the whole program terminates.


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

 🦀 **Depicting the memory allocation:**
 
 1. The `main()` function creates a new `Stack Frame`. The **stack pointer increases 8 bytes** and the variable `heap` calls the function `boxed_value()`.
 
 2. In the `boxed_value()` the **stack pointer increses 8 bytes again** and we have a local variable called `result` that has a `Box` with a value inside. The **`Box` acts as a pointer** so the return value is **8 bytes** (the `heap` variable in `main()` will be the same **8 bytes but as an address to the `Heap Memory`**). Inside the `Box` we have a value `99` which is `i32` so we need to reserve **4 bytes but this time on the heap**. The function ends, **deallocates and decrases the stack pointer by 8 bytes**.
 
 3. The `heap` variable in `main()` function now receives a **copied address** of the value stored in the other memory region, the `Heap Memory`. Then the funtion runs and terminates the program.
  
<div align=center>

<kbd>

<h1 align=center>&nbsp;&nbsp;&nbsp;HEAP (SYSTEM SIZE)&nbsp;&nbsp;&nbsp;</h1>

  |<h3>STACK<br>⟱</h3>|
  |--
  |<div align=center><h3>main()</h3><h3><kbd>&nbsp;heap = 0x6f32&nbsp;</kbd></h3></div>
  |<div align=center><h3>boxed_value()</h3><h3><kbd>&nbsp;result = 0x6f32&nbsp;</kbd></h3></div>
  |<img width=300 /><br><br><br><br><br><br><br><br><br><br>|
  |<div align=center><h3><kbd>&nbsp;heap address 0x6f32 = 99&nbsp;</kbd></h3></div>
  |<h3 align=center>⟰<br>HEAP</h3>|
  
</kbd>
  
</div>

<br>
  
# [HEAP](#heap) Properties
* The `heap` is another abstraction that, unlike `stack`, has a **flexible size** that can change over time (like in **run time**).
* Instead of one `Stack Frame` for each thread the `heap` memory region has a large **shared memory with the `stack`**.
* The `heap` memory **grows upwards** starting from the [lower address](#low-memory-address).
* The `heap` is **not stored** in the binary and is discarded after the program execution.
* Its size limit is bounded the system's memory. 
* The `heap` **value size** is determined by its **data type**.
* Because of it's **dyanmic** nature it needs to make system calls to ask for more space as soon as requested thus adding a little overhead. This turns `heap` slower than `stack` memory.


