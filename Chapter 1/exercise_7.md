# Chapter 1

## Exercise 7

Sample H. The function `sub_10BB6` has a loop searching for something. First recover the function prototype and then infer the types based on the context. Hint: You should probably have a copy of the PE specification nearby.

----

**Tool used:** Radare2

### Function Disassembly

```c
┌ 69: fcn.00010bb2 (int32_t arg_4h);
│ `- args(sp[0x4..0x4])
│           0x00010bb2      8b442404       mov eax, dword [arg_4h]
│           0x00010bb6      53             push ebx
│           0x00010bb7      56             push esi
│           0x00010bb8      8b703c         mov esi, dword [eax + 0x3c]
│           0x00010bbb      03f0           add esi, eax
│           0x00010bbd      0fb74614       movzx eax, word [esi + 0x14]
│           0x00010bc1      33db           xor ebx, ebx
│           0x00010bc3      66395e06       cmp word [esi + 6], bx
│           0x00010bc7      57             push edi
│           0x00010bc8      8d7c3018       lea edi, [eax + esi + 0x18]
│       ┌─< 0x00010bcc      761d           jbe 0x10beb
│       │   ; CODE XREF from fcn.00010bb2 @ 0x10be9(x)
│      ┌──> 0x00010bce      ff742414       push dword [arg_4h]
│      ╎│   0x00010bd2      57             push edi
│      ╎│   0x00010bd3      ff15a4690100   call dword [0x169a4]
│      ╎│   0x00010bd9      85c0           test eax, eax
│      ╎│   0x00010bdb      59             pop ecx
│      ╎│   0x00010bdc      59             pop ecx
│     ┌───< 0x00010bdd      7414           je 0x10bf3
│     │╎│   0x00010bdf      0fb74606       movzx eax, word [esi + 6]
│     │╎│   0x00010be3      83c728         add edi, 0x28               ; 40
│     │╎│   0x00010be6      43             inc ebx
│     │╎│   0x00010be7      3bd8           cmp ebx, eax
│     │└──< 0x00010be9      72e3           jb 0x10bce
│     │ │   ; CODE XREF from fcn.00010bb2 @ 0x10bcc(x)
│     │ └─> 0x00010beb      33c0           xor eax, eax
│     │     ; CODE XREF from fcn.00010bb2 @ 0x10bf5(x)
│     │ ┌─> 0x00010bed      5f             pop edi
│     │ ╎   0x00010bee      5e             pop esi
│     │ ╎   0x00010bef      5b             pop ebx
│     │ ╎   0x00010bf0      c20800         ret 8
│     │ ╎   ; CODE XREF from fcn.00010bb2 @ 0x10bdd(x)
│     └───> 0x00010bf3      8bc7           mov eax, edi
└       └─< 0x00010bf5      ebf6           jmp 0x10bed
```

### Function Prototype
```
int __stdcall sub_10bb2(struct _arg_4h *arg_4h)
```

### Preliminary Analysis & Function Behaviour

1. The code from `0x00010bb2` to `0x00010bcc` appear to retrieve a structure pointer, `arg_4h`, access a substructure member, and compare a word member at `+6` to 0. If the value < 0 , return 0.

####

2. The structure and an address retrieved from `arg_4h` are passed to a dynamically loaded function `fun_169a4`. If the return code from this function is 0, return the original address passed to `fun_169a4`
    - `fun_169a4` unusually cleans the stack by popping `ecx` twice, instead of a typical `add esp, 8`. This also indicates that the function likely uses `cdecl` calling convention.

####

3. ebx is incremented by 1 and edi (address paramter for `fun_169a4`) is incremeneted by 28h (40). The same member of the substructure at `+6` is then compared with ebx, 

### PE Format

The structure `_arg_4h` points to a Windows PE image. This is hinted by the question, and the immediate access of `[arg_4h + 0x3C]` 
The image structure is as follows:

```c
arg_4h → IMAGE_DOS_HEADER                           (base)
            IMAGE_NT_HEADERS                        (base + [0x3C])
                Signature ("PE\0\0")                (0x3C + 0x00)
                IMAGE_FILE_HEADER        
                    Machine                         (0x3C + 0x04)
                    NumberOfSections                (0x3C + 0x06)   !
                    timeDateStamp                   (0x3C + 0x08)
                    PointerToSymbolTable            (0x3C + 0x0C)
                    NumberOfSymbols                 (0x3C + 0x10)
                    SizeOfOptionalHeader            (0x3C + 0x14)   !
                    Characteristics                 (0x3C + 0x16)
                IMAGE_OPTIONAL_HEADER               
                    (<SizeOfOptionalHeader> bytes)  (0x3C + 0x18)
                    /* not relevant here */
                IMAGE_SECTION_HEADER         
                    ...
                    Section_i                       (28h (40) bytes each)          
                        NAme                        (0x3C + 0x18 + SizeOfOptionalHeader + <0x28 * section_idx>)
                        VirtualSize                 (0x3C + 0x18 + SizeOfOptionalHeader + <0x28 * section_idx> + 0x08)
                        VirtualAddress              (0x3C + 0x18 + SizeOfOptionalHeader + <0x28 * section_idx> + 0x0C)
                        SizeOfRawData               (0x3C + 0x18 + SizeOfOptionalHeader + <0x28 * section_idx> + 0x10)
                        PointerToRawData            (0x3C + 0x18 + SizeOfOptionalHeader + <0x28 * section_idx> + 0x14)
                        PointerToRelocations        (0x3C + 0x18 + SizeOfOptionalHeader + <0x28 * section_idx> + 0x18)
                        PointerToLinenumbers        (0x3C + 0x18 + SizeOfOptionalHeader + <0x28 * section_idx> + 0x1c)
                        NumberOfRelocations         (0x3C + 0x18 + SizeOfOptionalHeader + <0x28 * section_idx> + 0x20)
                        NumberOfLinenumbers         (0x3C + 0x18 + SizeOfOptionalHeader + <0x28 * section_idx> + 0x22)
                        Characteristics             (0x3C + 0x18 + SizeOfOptionalHeader + <0x28 * section_idx> + 0x24)
                    ...

                                                    

                            
```
- Source: https://learn.microsoft.com/en-us/windows/win32/debug/pe-format#ms-dos-stub-image-only

### Decompiled Function

Given the behaviour and implied PE format, the decompiled function can include more detailed names:

```c
struct _IMAGE_SECTION_HEADER __stdcall *sub_10bb2(struct _IMAGE_DOS_HEADER *image_base) {
    struct _IMAGE_NT_HEADERS *nt = (struct _IMAGE_NT_HEADERS *)((uint8_t *)image_base + image_base->e_lfanew);  
    short SizeOfOptionalHdr = nt->FileHeader.SizeOfOptionalHeader;
    short NumberOfSections = nt->FileHeader.NumberOfSections;

    if (NumberOfSections <= 0) {
        return 0;
    }

    struct _IMAGE_SECTION_HEADER *currentSectionHeader = (struct _IMAGE_SECTION_HEADER *)((uint8_t *)&nt->OptionalHeader + SizeOfOptionalHdr);

    for (int i = 0; i < NumberOfSections; i++) {
        if (fun_0x169a4(currentSectionHeader, image_base) == 0) {
            return currentSectionHeader;
        }
        currentSectionHeader++;

    }
    return 0;
}
```

### Function Summary

This function loops through each section in a PE file and applies a helper function on each one. If any section matches the condition defined by that function, return it.