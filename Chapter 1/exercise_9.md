# Chapter 1

## Exercise 9

Sample L. Explain what function `sub_1000CEA0` does and then decompile it back to C.

---

### Disassembly

```c
┌ 45: fcn.1000cea0 (LPSTR arg_8h, int32_t arg_ch);
│ `- args(sp[0x4..0x8])
│           0x1000cea0      55             push ebp
│           0x1000cea1      8bec           mov ebp, esp
│           0x1000cea3      57             push edi
│           0x1000cea4      8b7d08         mov edi, dword [arg_8h]
│           0x1000cea7      33c0           xor eax, eax
│           0x1000cea9      83c9ff         or ecx, 0xffffffff          ; -1
│           0x1000ceac      f2ae           repne scasb al, byte es:[edi]
│           0x1000ceae      83c101         add ecx, 1
│           0x1000ceb1      f7d9           neg ecx
│           0x1000ceb3      83ef01         sub edi, 1
│           0x1000ceb6      8a450c         mov al, byte [arg_ch]
│           0x1000ceb9      fd             std
│           0x1000ceba      f2ae           repne scasb al, byte es:[edi]
│           0x1000cebc      83c701         add edi, 1
│           0x1000cebf      3807           cmp byte [edi], al
│       ┌─< 0x1000cec1      7404           je 0x1000cec7
│       │   0x1000cec3      33c0           xor eax, eax
│      ┌──< 0x1000cec5      eb02           jmp 0x1000cec9
│      ││   ; CODE XREF from fcn.1000cea0 @ 0x1000cec1(x)
│      │└─> 0x1000cec7      8bc7           mov eax, edi
│      │    ; CODE XREF from fcn.1000cea0 @ 0x1000cec5(x)
│      └──> 0x1000cec9      fc             cld
│           0x1000ceca      5f             pop edi
│           0x1000cecb      c9             leave
└           0x1000cecc      c3             ret

```

### Initial Observations

1. `arg_8h` is likely a pointer to a string, as a `scasb` instruction is used on it. It is also used with a `repne` on zero (al = 0). Assuming the direction flag is cleared, this is likely looking for a terminating null byte within a string. - Initializing ecx to 0xFFFFFFFF ensure that `repne` does not end early, and when a null byte is reached, ecx = -length - 1. By applying `add ecx, 1` and `neg ecx`, the actual length is obtained. Lastly, edi is decremented, so it is pointing at the last character before the null byte.

####

2. Next, the direction flag is set. Another `repne scasb` instruction iterates through the original string in the other direction, this time comparing with al = `arg_ch`. If the character is found, increment edi to move back to it. ecx already contains the string length to ensure the `scasb` halts.

####

3. “Finally, `byte [edi]` is compared with arg_ch to ensure that a match was actually found. If the byte at EDI equals arg_ch, the function returns EDI; otherwise, it returns 0.”

### Function Summary

At a high level, this function takes a pointer to a string, a character, and returns the address of the last occurrence of that character.


### Decompilation

```c
__stdcall char* find_last_char(char* str, char c) {
    char* p = str;
    while (*p != '\0') {
        p++;
    }
    p--;  
    while (p >= str && *p != c) {
        p--;
    }
    if (*p == ch) {
        return p;
    } 
    return 0;
}
```

