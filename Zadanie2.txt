%macro pushd 0
    push eax
    push ebx
    push ecx
    push edx
%endmacro

%macro popd 0
    pop edx
    pop ecx
    pop ebx
    pop eax
%endmacro


%macro print 2
    pushd
    mov edx, %1
    mov ecx, %2
    mov ebx, 1
    mov eax, 4
    int 0x80
    popd
%endmacro


%macro floatnum 1
    print dlen, dot
    mov dl, %1
    %%_divloop:
        mov al, dh
        mov dh, 10
        mul dh 
        div cl
        mov dh, ah
        mov ah, 0
        dprint
        cmp dh, 0
        jz %%_endloop
        dec dl
        mov eax, 0
        cmp dl, 0
        jnz %%_divloop
    %%_endloop:
%endmacro
    

%macro dprint 0
    pushd
   
    mov ecx, 10
    mov bx, 0

    %%_divide:
        mov edx, 0
        div ecx
        push dx 
        inc bx
        test eax, eax
        jnz %%_divide
        
        mov cx, bx
        
    %%_digit:
        pop ax
        add ax, '0'
        mov [result], ax
        print 1, result
        dec cx
        cmp cx, 0
        jg %%_digit    
    
    popd
%endmacro


section .text

global _start

_start:

    mov ebx, alen
    mov cl, alen/4
    dec ebx
    mov eax, 0
    
_loop:
    mov dl, [array1 + ebx]
    mov dh, [array2 + ebx]
    sub dl, dh
    add al, dl
    dec ebx
    cmp ebx, -1
    jnz _loop
    
    print rlen, res   
    cmp al, 0
    jl _negative
    jmp _division

    
_negative:
    neg al
    print mlen, minus

_division:
    div cl
    mov dl, al
    mov dh, ah
    mov eax, 0
    mov al, dl
    mov ebx, 0
    dprint
    cmp dh, 0
    jz  _end  

    floatnum 3
    
_end:    
    
    print nlen, newline
    print len, message
    print nlen, newline
    
    mov eax, 1
    int   0x80

section .data
    array1 dd 5, 3, 2, 6, 1, 7, 4
    alen equ $ - array1
    array2 dd 0, 10, 1, 9, 2, 8, 5
    res db "Result: "
    rlen equ $ - res
    message db "Done"
    len equ $ - message
    minus db "-"
    mlen equ $ - minus
    dot db "."
    dlen equ $ - dot
    newline db 0xA, 0xD
    nlen equ $ - newline
    
section .bss
    result resb 1