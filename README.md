```cpp

#include <stdio.h>
int main()
{
  return 0;
}
```
output:
```objdump
main:
  pushq %rbp
  movq %rsp, %rbp
  movl $0, %eax
  popq %rbp
  ret
  ```
  
  ```cpp
#include <stdio.h>
void test(){}
int main()
{
  printf("hello, world\n");
  return 0;
}
```
```nasm
test():
  pushq %rbp
  movq %rsp, %rbp
  nop
  popq %rbp
  ret
.LC0:
  .string "hello, world"
main:
  pushq %rbp
  movq %rsp, %rbp
  movl $.LC0, %edi
  call puts
  movl $0, %eax
  popq %rbp
  ret
 ```
 
 Now, declare a variable inside test() and see how the assembly output changes
 ```cpp
 void test(){
    int x = 20;
}
```

```nasm
test():
  pushq %rbp
  movq %rsp, %rbp
  movl $20, -4(%rbp)  // --------> space to accomodate a new variable on stack
  nop
  popq %rbp
  ret
.LC0:
  .string "hello, world"
main:
  pushq %rbp
  movq %rsp, %rbp
  movl $.LC0, %edi
  call puts
  movl $0, %eax
  popq %rbp
  ret
  ```

Now declare another variable inside test() & observe the output

```cpp
void test(){
    int x = 20;
    int y = 30;
}
```

output:
```nasm
test():
  pushq %rbp
  movq %rsp, %rbp
  movl $20, -4(%rbp)
  movl $30, -8(%rbp)
  nop
  popq %rbp
  ret
.LC0:
  .string "hello, world"
main:
  pushq %rbp
  movq %rsp, %rbp
  movl $.LC0, %edi
  call puts
  movl $0, %eax
  popq %rbp
  ret
```

Very important point to notice is that ***stack grows downwards***

doing some operation (multiplication):
```cpp
void test(){
    int x = 20;
    int y = 30;
    int z = x*y;
    
}
```

output:
```nasm
test():
  pushq %rbp
  movq %rsp, %rbp
  movl $20, -4(%rbp)
  movl $30, -8(%rbp)
  movl -4(%rbp), %eax  // ----> store the content stored at 4 bytes from stack pointer into **eax**
  imull -8(%rbp), %eax // ----> multiply the content stored at 8 bytes from stack to the content stored in eax & store resut in **eax**
  movl %eax, -12(%rbp)
  nop
  popq %rbp
  ret
.LC0:
  .string "hello, world"
main:
  pushq %rbp
  movq %rsp, %rbp
  movl $.LC0, %edi
  call puts
  movl $0, %eax
  popq %rbp
  ret
```
