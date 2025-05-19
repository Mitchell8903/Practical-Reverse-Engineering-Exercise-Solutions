# Chapter 1

## Exercise 8

**Sample H. Decompile `sub_11732` and expllain the most likely programming construct used in the original code.**

**Tool used:** Radare2

### Disassembly

```
┌ 65: fcn.0001172e (int32_t arg_8h);
│ `- args(sp[0x4..0x4])
│           0x0001172e      56             push esi
│           0x0001172f      8b742408       mov esi, dword [arg_8h]
│           0x00011733      4e             dec esi
│       ┌─< 0x00011734      7429           je 0x1175f
│       │   0x00011736      4e             dec esi
│      ┌──< 0x00011737      741c           je 0x11755
│      ││   0x00011739      4e             dec esi
│     ┌───< 0x0001173a      740f           je 0x1174b
│     │││   0x0001173c      83ee09         sub esi, 9
│    ┌────< 0x0001173f      752a           jne 0x1176b
│    ││││   0x00011741      8b7008         mov esi, dword [eax + 8]
│    ││││   0x00011744      d1ee           shr esi, 1
│    ││││   0x00011746      83c00c         add eax, 0xc                ; 12
│   ┌─────< 0x00011749      eb1c           jmp 0x11767
│   │││││   ; CODE XREF from fcn.0001172e @ 0x1173a(x)
│   ││└───> 0x0001174b      8b703c         mov esi, dword [eax + 0x3c]
│   ││ ││   0x0001174e      d1ee           shr esi, 1
│   ││ ││   0x00011750      83c05e         add eax, 0x5e               ; 94
│   ││┌───< 0x00011753      eb12           jmp 0x11767
│   │││││   ; CODE XREF from fcn.0001172e @ 0x11737(x)
│   │││└──> 0x00011755      8b703c         mov esi, dword [eax + 0x3c]
│   │││ │   0x00011758      d1ee           shr esi, 1
│   │││ │   0x0001175a      83c044         add eax, 0x44               ; 68
│   │││┌──< 0x0001175d      eb08           jmp 0x11767
│   │││││   ; CODE XREF from fcn.0001172e @ 0x11734(x)
│   ││││└─> 0x0001175f      8b703c         mov esi, dword [eax + 0x3c]
│   ││││    0x00011762      d1ee           shr esi, 1
│   ││││    0x00011764      83c040         add eax, 0x40               ; 64
│   ││││    ; CODE XREFS from fcn.0001172e @ 0x11749(x), 0x11753(x), 0x1175d(x)
│   └─└└──> 0x00011767      8931           mov dword [ecx], esi
│    │      0x00011769      8902           mov dword [edx], eax
│    │      ; CODE XREF from fcn.0001172e @ 0x1173f(x)
│    └────> 0x0001176b      5e             pop esi
└           0x0001176c      c20400         ret 4
```

### Initial Observations

- Two arguments, `eax` and `arg_8h`
- Stores results in memory pointed to by `ecx` and `edx`
- The jumps resemble a switch statement

### Decompilation

```c
void __usercall sub_1172e(struct _s1* eax_val, int arg_8h, int* ecx_out, int* edx_out) {
    int i;      //esi 
    int a;      //eax 

    switch (arg_8h) {
        case 1:
            i = eax_val->off_3ch >> 1;
            a = (int)eax_val + 0x40;
            break;
        case 2:
            i = eax_val->off_3ch >> 1;
            a = (int)eax_val + 0x44;
            break;
        case 3:
            i = eax_val->off_3ch >> 1;
            a = (int)eax_val + 0x5e;
            break;
        case 0xc:
            return;  
        default:
            i = eax_val->off_8h >> 1;
            a = (int)eax_val + 0x0c;
            break;
    }

    *ecx_out = i;
    *edx_out = a;
}
```

