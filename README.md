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
#conditionals

```cpp
#include <stdio.h>
void test(){}
int main()
{
    int x,y;
    scanf("%d",x);
    scanf("%d",y);
    
    if(x < y){ 

        int zz = 10;
    }
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
  .string "%d"
main:
  pushq %rbp
  movq %rsp, %rbp
  subq $16, %rsp
  movl -4(%rbp), %eax
  movl %eax, %esi
  movl $.LC0, %edi
  movl $0, %eax
  call scanf
  movl -8(%rbp), %eax
  movl %eax, %esi
  movl $.LC0, %edi
  movl $0, %eax
  call scanf
  movl -4(%rbp), %eax
  cmpl -8(%rbp), %eax  // -----------> actual comparison is carried out here
  jge .L3
  movl $10, -12(%rbp)
.L3:
  movl $0, %eax
  leave
  ret
```

compiler optimization
```cpp
#include <stdio.h>
void test(){}
int main()
{
    if(2<3){ 
        int zz = 10;
    }
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
main:
  pushq %rbp
  movq %rsp, %rbp
  movl $10, -4(%rbp)
  movl $0, %eax
  popq %rbp
  ret
  ```
Compiler is smart enough to see that this if branch will always be true so no comparison is performed in oreder to save cpu cycles.


#Jump instruction & loops

```cpp
#include <stdio.h>
void test(){
    for(int i=0;i<10;i++){}
}
int main(){
  return 0;
}
```
output
```nasm
test():
  pushq %rbp
  movq %rsp, %rbp
  movl $0, -4(%rbp)
.L3:
  cmpl $9, -4(%rbp)
  jg .L4
  addl $1, -4(%rbp)
  jmp .L3
.L4:
  nop
  popq %rbp
  ret
main:
  pushq %rbp
  movq %rsp, %rbp
  movl $0, %eax
  popq %rbp
  ret
```
