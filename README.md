; Cis-11A
;Bubble Sort Program
        .ORIG   x3000

        JSR     INIT            ; Initialize stack pointer and other registers
        JSR     GET_INPUT       ; Get user input
        JSR     BUBBLE_SORT     ; Sort the numbers
        JSR     DISPLAY         ; Display the sorted numbers
        HALT

; Subroutine: INIT
; Description: Initialize the stack pointer and other registers
INIT    LD      R6, STACK_PTR_INIT ; Load stack pointer initial value
        RET

; Subroutine: GET_INPUT
; Description: Get 8 numbers from the user
GET_INPUT   LEA     R0, PROMPT_MSG     ; Load prompt message address
            PUTS                    ; Display prompt message
            AND     R1, R1, #0      ; Clear R1 (counter)
GET_LOOP    ADD     R2, R1, #-8     ; Check if 8 numbers are entered
            BRz     GET_INPUT_END   ; If 8 numbers are entered, exit loop
            JSR     GET_NUM         ; Get number from user
            LEA     R3, NUMBERS     ; Load base address of NUMBERS array
            ADD     R3, R3, R1      ; Calculate address to store number
            STR     R0, R3, #0      ; Store the number in the array
            ADD     R1, R1, #1      ; Increment counter
            BR      GET_LOOP

GET_INPUT_END
            RET

; Subroutine: GET_NUM
; Description: Get a number from the user
GET_NUM     AND     R0, R0, #0      ; Clear R0 (accumulating the number)
            AND     R1, R1, #0      ; Clear R1 (digit counter)
            LD      R2, ASCII_ZERO
GET_NUM_LOOP
            GETC                    ; Get character from keyboard
            OUT                     ; Echo character
            ADD     R3, R0, #0
            ADD     R0, R0, R0      ; Multiply R0 by 10
            ADD     R0, R0, R0
            ADD     R0, R0, R0
            ADD     R0, R0, R3      ; Add the previous result
            ADD     R3, R0, R2      ; Convert ASCII to digit
            ADD     R0, R0, R3      ; Add the digit to R0
            ADD     R3, R3, #-10    ; Check if the character is a digit
            BRn     GET_NUM_END     ; If not a digit, end input
            BRzp    GET_NUM_LOOP    ; Otherwise, continue input

GET_NUM_END 
            RET

; Subroutine: BUBBLE_SORT
; Description: Sort the numbers using Bubble Sort algorithm
BUBBLE_SORT AND     R4, R4, #0      ; Clear R4 (outer loop counter)
            LD      R5, NUM_COUNT   ; Load number of elements
SORT_OUTER  ADD     R4, R5, #-1     ; Outer loop counter = num_elements - 1
            BRz     SORT_END        ; If outer loop counter is zero, exit
SORT_INNER  AND     R2, R2, #0      ; Clear R2 (swap flag)
            LEA     R1, NUMBERS     ; Load base address of array
            ADD     R3, R1, #0      ; Initialize pointer for inner loop
            ADD     R6, R5, #-1     ; Inner loop counter
COMPARE     LDR     R0, R3, #0      ; Load current element
            LDR     R1, R3, #1      ; Load next element
            NOT     R2, R1          ; Negate next element
            ADD     R2, R2, #1      ; Two's complement of next element
            ADD     R2, R0, R2      ; R2 = current - next
            BRZP    NOSWAP          ; If current <= next, no swap
            STR     R1, R3, #0      ; Swap: store next in current
            STR     R0, R3, #1      ; Swap: store current in next
            ADD     R2, R2, #1      ; Set swap flag
NOSWAP      ADD     R3, R3, #1      ; Move to next pair
            ADD     R6, R6, #-1     ; Decrement inner loop counter
            BRP     COMPARE         ; If inner loop counter > 0, repeat
            ADD     R5, R5, #-1     ; Decrement outer loop counter
            BRp     SORT_OUTER      ; Repeat outer loop
SORT_END    RET

; Subroutine: DISPLAY
; Description: Display the sorted numbers
DISPLAY     AND     R1, R1, #0      ; Clear counter
DISPLAY_LOOP
            LEA     R3, NUMBERS     ; Load base address of NUMBERS array
            ADD     R2, R3, R1      ; Calculate address to load number
            LDR     R0, R2, #0      ; Load number from array
            JSR     DISP_NUM        ; Display the number
            LD      R0, NEWLINE     ; Load newline character
            OUT                     ; Display newline
            ADD     R1, R1, #1      ; Increment counter
            ADD     R3, R1, #-8     ; Check if all numbers are displayed
            BRZ     DISPLAY_END     ; If yes, exit loop
            BR      DISPLAY_LOOP

DISPLAY_END RET

; Subroutine: DISP_NUM
; Description: Display a number
DISP_NUM    JSR     PUSH            ; Save R0 on stack
            JSR     PUSH            ; Save R1 on stack
            AND     R2, R2, #0      ; Clear R2
            LEA     R1, NUM_BUFFER  ; Load address of NUM_BUFFER
            ADD     R1, R1, #4      ; Point to the end of the buffer
            STR     R2, R1, #0      ; Null-terminate the buffer

DISP_NUM_LOOP
            AND     R2, R0, #0      ; Clear R2
            ADD     R3, R0, #0      ; Set R3 to the value in R0
DIV_LOOP    ADD     R3, R3, #-10    ; Subtract 10 from R3
            BRn     DIV_DONE        ; If result is negative, division is done
            ADD     R2, R2, #1      ; Increment R2 (quotient)
            BR      DIV_LOOP        ; Repeat division

DIV_DONE    ADD     R3, R3, #10     ; Add 10 back to R3 to get the remainder
            ADD     R0, R2, #0      ; Set R0 to quotient
            LD      R4, ASCII_ZERO  ; Load ASCII '0' to R4
            ADD     R3, R3, R4      ; Convert remainder to ASCII
            STR     R3, R1, #-1     ; Store digit in buffer
            ADD     R1, R1, #-1     ; Move to the next position
            BRz     DISP_NUM_END    ; If quotient is 0, end loop
            ADD     R0, R2, #0      ; Set R0 to quotient for next division
            BR      DISP_NUM_LOOP

DISP_NUM_END
            ADD     R1, R1, #1      ; Point to the first digit
            PUTS                    ; Display the buffer
            JSR     POP             ; Restore R1
            JSR     POP             ; Restore R0
            RET

; Stack operations
PUSH        ADD     R6, R6, #-1     ; Decrement stack pointer
            STR     R0, R6, #0      ; Store R0 on stack
            RET

POP         LDR     R0, R6, #0      ; Load value from stack
            ADD     R6, R6, #1      ; Increment stack pointer
            RET

; Data Section
STACK_PTR_INIT  .FILL x4000         ; Initial stack pointer value
NUM_COUNT       .FILL #8            ; Number of elements
NUMBERS         .BLKW #8            ; Space for 8 numbers
PROMPT_MSG      .STRINGZ "Enter 8 numbers (0-100): "
ASCII_ZERO      .FILL x0030         ; ASCII code for '0'
NEWLINE         .FILL x000A         ; ASCII code for newline
NUM_BUFFER      .BLKW #5            ; Buffer for displaying numbers

        .END
