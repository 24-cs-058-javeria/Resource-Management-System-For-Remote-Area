.model small
.stack 100h

.data
    ; Main Menu
    msg_menu    db 13,10,'....RESOURCE MANAGEMENT SYSTEM....',13,10
                db '1. Add New Record',13,10
                db '2. View Record',13,10
                db '3. Update Record',13,10
                db '4. Remove Record',13,10
                db '5. Exit ',13,10,'Enter your choice: $',13,10
    
    p_id        db 13,10,'Enter Serial No: $'
    err_id      db 13,10,'Error: Sr# already exists! Try again.$'
    not_found   db 13,10,'Error: Sr# not found!$'
    msg_upd     db 13,10,'Record found! Enter new details:$'
    msg_rem     db 13,10,'Record removed!$'
    
    p_name      db 13,10,'Enter Name: $'
    p_members   db 13,10,'Family Members (Digit + Enter): $'
    p_water     db 13,10,'Water Needed (Ltrs): $'
    p_flour     db 13,10,'Flour Needed (kg): $'
    p_pulses    db 13,10,'Pulses Needed (kg): $'
    
    ; Table Formatting
   table_head  db 13,10,13,10,'Sr#  Name       Mem   Wat   Flr   Pls',13,10
                db '---------------------------------------$'
    total_line  db 13,10,'---------------------------------------',13,10
                db 'GRAND TOTALS:  $'
    gap         db '     $'
    break       db 13, 10, '$'
    
    ; Data Storage
    list_id     db 10 dup(0)
    list_mem    db 10 dup(0)
    list_wat    db 10 dup(0)
    list_flr    db 10 dup(0)
    list_pls    db 10 dup(0)
    list_names  db 100 dup(' ') 
    
    count       dw 0 
    sum_mem     dw 0
    sum_wat     dw 0
    sum_flr     dw 0
    sum_pls     dw 0 
    box         db 0 
    temp_id     db 0

.code
start:
    mov ax,@data
    mov ds,ax

main_loop:
    mov ah,9
    mov dx,offset msg_menu
    int 21h

    mov ah,1
    int 21h
    cmp al,'1'
    je input
    cmp al,'2'
    je output
    cmp al,'3'      
    je update
    cmp al,'4'
    je remove
    cmp al,'5'
    je exit
    jmp main_loop

;DATA ENTRY 
input:
    mov bx, count
get_unique_id:
    mov ah,9
    mov dx,offset p_id
    int 21h
    call get_input 
    mov cx, count        
    cmp cx, 0
    je save_id           
    mov si, 0          
check_loop:
    cmp al, list_id[si]
    je show_id_error     
    inc si
    loop check_loop
    jmp save_id         

show_id_error:
    mov ah,9
    mov dx,offset err_id
    int 21h
    jmp get_unique_id    

save_id:
    mov list_id[bx], al
    mov ah,9
    mov dx,offset p_name
    int 21h
    mov ax, bx ; BX index is used
    mov dl, 10
    mul dl
    mov si, ax 
read_name:
    mov ah,1
    int 21h
    cmp al,13
    je name_ready
    mov list_names[si], al
    inc si
    jmp read_name
name_ready:
    mov ah,9
    mov dx,offset p_members
    int 21h
    call get_input
    mov list_mem[bx], al
    mov ah, 0
    add sum_mem, ax

    mov ah,9
    mov dx,offset p_water
    int 21h
    call get_input
    mov list_wat[bx], al
    mov ah, 0
    add sum_wat, ax

    mov ah,9
    mov dx,offset p_flour
    int 21h
    call get_input
    mov list_flr[bx], al
    mov ah, 0
    add sum_flr, ax

    mov ah,9
    mov dx,offset p_pulses
    int 21h
    call get_input
    mov list_pls[bx], al
    mov ah, 0
    add sum_pls, ax

    inc count
    jmp main_loop

;  UPDATE FEATURE 
update:
    mov ah,9
    mov dx,offset p_id
    int 21h
    call get_input
    mov temp_id, al
    mov cx, count
    mov si, 0
find_u:
    mov al, temp_id
    cmp al, list_id[si]
    je do_upd
    inc si
    loop find_u
    mov ah,9
    mov dx,offset not_found
    int 21h
    jmp main_loop
do_upd:
    ; subtract old  values 
    mov al, list_mem[si]
    mov ah,0
    sub sum_mem, ax
    mov al, list_wat[si]
    mov ah,0
    sub sum_wat, ax
    mov al, list_flr[si]
    mov ah,0
    sub sum_flr, ax
    mov al, list_pls[si]
    mov ah,0
    sub sum_pls, ax
    
    mov bx, si
    dec count ; so the count does not double after the update
    mov ah,9
    mov dx,offset msg_upd
    int 21h
    jmp save_id

;  REMOVE FEATURE 
remove:
    mov ah,9
    mov dx,offset p_id
    int 21h
    call get_input
    mov temp_id, al
    mov cx, count
    mov si, 0
find_r:
    mov al, temp_id
    cmp al, list_id[si]
    je do_rem
    inc si
    loop find_r
    mov ah,9
    mov dx,offset not_found
    int 21h
    jmp main_loop
do_rem:
    ; Subtract totals
    mov al, list_mem[si]
    mov ah,0
    sub sum_mem, ax
    mov al, list_wat[si]
    mov ah,0
    sub sum_wat, ax
    mov al, list_flr[si]
    mov ah,0
    sub sum_flr, ax
    mov al, list_pls[si]
    mov ah,0
    sub sum_pls, ax
    
shift_loop:
    mov di, si
    inc di
    mov al, list_id[di]
    mov list_id[si], al
    mov al, list_mem[di]
    mov list_mem[si], al
    mov al, list_wat[di]
    mov list_wat[si], al
    mov al, list_flr[di]
    mov list_flr[si], al
    mov al, list_pls[di]
    mov list_pls[si], al
    inc si
    mov ax, count
    cmp si, ax
    jne shift_loop
    
    dec count
    mov ah,9
    mov dx,offset msg_rem
    int 21h
    jmp main_loop

;  DATA DISPLAY 
output:
    mov ah,9
    mov dx,offset table_head
    int 21h
    mov cx, count
    cmp cx, 0
    je main_loop
    mov bx, 0
print_rows:
    mov ah,9
    mov dx,offset break
    int 21h
    mov al, list_id[bx]
    add al, '0'
    mov dl, al
    mov ah, 2
    int 21h
    mov ah,9
    mov dx,offset gap
    int 21h
    mov ax, bx
    mov dl, 10
    mul dl
    mov si, ax
    mov di, 0
show_name:
    mov dl, list_names[si]
    mov ah, 2
    int 21h
    inc si
    inc di
    cmp di, 10
    jne show_name
    mov al, list_mem[bx]
    call print_item
    mov al, list_wat[bx]
    call print_item
    mov al, list_flr[bx]
    call print_item
    mov al, list_pls[bx]
    call print_item
    inc bx
    loop print_rows
    mov ah,9
    mov dx,offset total_line
    int 21h
    mov ax, sum_mem
    call print_grand
    mov ax, sum_wat
    call print_grand
    mov ax, sum_flr
    call print_grand
    mov ax, sum_pls
    call print_grand
    jmp main_loop

;  HELPER FUNCTIONS(procedures) 
get_input proc
    mov ah,1
    int 21h
    sub al, '0'
    mov box, al
wait_enter:
    mov ah,1
    int 21h
    cmp al, 13
    jne wait_enter
    mov al, box
    ret
get_input endp

print_item proc
    mov box, al
    mov ah,9
    mov dx,offset gap
    int 21h
    mov al, box
    add al, '0'
    mov dl, al
    mov ah, 2
    int 21h
    ret
print_item endp

print_grand proc
    mov dl, 10
    div dl 
    mov si, ax 
    mov dl, al
    add dl, '0'
    mov ah, 2
    int 21h
    mov ax, si
    mov dl, ah
    add dl, '0'
    mov ah, 2
    int 21h
    mov ah,9
    mov dx,offset gap
    int 21h
    ret
print_grand endp

exit:
    mov ah,4ch
    int 21h
end start
