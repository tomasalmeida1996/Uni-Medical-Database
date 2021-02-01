; multi-segment executable file template.

data segment
    
    ;menu
	opcao1 db "L)ist all patients" 
	opcao2 db "l)ist patients of a specific pathology"
	opcao3 db "I)nsert a new patience"
	opcao4 db "P)rocess new file to import patients"
	opcao5 db "D)isplay statistics"
	opcao6 db "A)bout"
	opcao7 db "E)xit"
	
	;opcao 1
	listagem1 db "Lista de pacientes vazia"
	listagem2 db "<#number> '<patient name>', <pathology>"
	
	;opcao 2
	listagemPat db "Seleccione uma patologia" 
	listagemPat2 db "<#number> '<patient name>'"
	
	;opcao3
	insert1 db "Name of patient: "
	insert2 db "Pathology: " 
	
	;opcao4
	process1 db "Put the name file to process"
	
	;opcao5 estatistica
	estatistica1 db "Total patients: "
	estatistica2 db "Results:"
	estatistica3 db "Pathology"
	estatistica4 db "Num."
	estatistica5 db "Percentage"
		
	;about
	aboutInfo db "NAME               NUMBER"
	aluno1 db "Tomas Almeida       46134"
	aluno2 db "Bernardo Inocencio  45816" ;25bytes
 
	
	
	mainText db "<<Main>>"
	nextText db "<<Next>>"
	
	buffer1 db 49,?, 49 dup(?)  ;49 bytes para compensar o adicionar do '$'
	buffer2 db 49,?, 49 dup(?)
	buffer3 db 102 dup(?) 
	buffer4 db 3,?, 3 dup(?)
	buffer5 db 3,?, 3 dup(?) 
	
	bufferPat db 49,?, 49 dup(?)
	bufferPat2 db 51 dup(?)
	aux db 1
	aux2 db 1
	
	directory db "..\",0
	 
	FileNameDB db "database.dmp",0  ;"C\:database.dmp",0 

	sizestruct db 102
	nrPag db 0
	nrPac db 0                   ;numero de pacientes da bd
	nrPatologias db 0
	structDB db 5100 dup(?)         ;#numero(1b),<nome>(50b),<patologia>(50b),#utilizacoes(1b)  
	
	structPatologias db 2550 dup(?)   ;<patologia>(50b),#utilizacoes(1b) *50 

    pkey db "press any key...$"
ends

stack segment
    dw   128  dup(0)
ends

code segment
start:
; set segment registers:
    mov ax, data
    mov ds, ax
    mov es, ax

    ; add your code here
    
    call loadDatabase
    call loadPatologias
    
    call Display
            
    
    mov ax, 4c00h ; exit to operating system.
    int 21h    
                                                           
;*****************************************
;output do display no ecra
;*****************************************    
Display proc 
    
    voltar:
    call ModoVideo
    call printDisplay
    
    displayEspera:
        call displayVerifica
        
    cmp al, 1
	je listPage             ;opcao1 do menu
	cmp al, 2
	je listPathology        ;opcao2 do menu
	cmp al, 3
	je insertPage           ;opcao3 do menu
	;cmp al, 4
	;je processPage         ;opcao4 do menu
	cmp al, 5
	je statsPage            ;opcao5 do menu
	cmp al, 6
	je aboutPage            ;opcao6 do menu
	cmp al, 7
	je exit                 ;opcao7 do menu
	
	jmp displayEspera

;***************** List *****************; 

  listPage:
  
    call ModoVideo       ;coloca em modo de video

    cmp nrPac, 0
    je ListSPac 
    
    printListagem:
    mov bl, nrPag   ;para depois usar o numero da pagina qd vamos
                    ;para multiplicarmos e usarmos como nrPac
 
    call ModoVideo
    call listagem
    call verificaListagem  ;verifica em que botao o utilizador carrega
    
    cmp al, 1              ;se carregou em main
    je listFim
    inc nrPag              ;se carregou em next
    jmp printListagem
    
    listSPac:
   	    call listagemSPac
        call verificaListagemSPac ;verifica em que botao o utilizador carrega
        
    listFim:
        mov nrPag, 0
        jmp voltar       
        
;***************** List Pathology *****************;

  listPathology:
  
    call ModoVideo       ;coloca em modo de video

    cmp nrPatologias, 0
    je ListSPatologias 
    
    printListaPatologias:
    mov bl, nrPag   ;para depois usar o numero da pagina qd vamos
                    ;para multiplicarmos e usarmos como nrPac
 
    call ModoVideo
    call listagemPatologias
    call verificaListagemPatologias  ;verifica em que botao o utilizador carrega
    
    cmp al, 10
    jbe list2
    
    cmp al, 11              ;se carregou em main
    je listPatologiasFim
    
    cmp al, 12
    inc nrPag               ;se carregou em next
    jmp printListaPatologias
    
    listSPatologias:
   	    call listagemSPac
        call verificaListagemSPac ;verifica em que botao o utilizador carrega
    
    list2:  ;al tem o numero da patologia escolhida 
        mov bl, al
        mov al, aux2
        ;push ax 
        mov al, bl
        mov bl, nrPag   ;para depois usar o numero da pagina qd vamos
                        ;para multiplicarmos e usarmos como nrPac 
        call ModoVideo
        call PacientesComPatologia 
        mov bl, al             ;ficar com o numero da patologia guardado
        
        call verificaListagem  ;verifica em que botao o utilizador carrega
        
        cmp al, 1              ;se carregou em main
        je listPatologiasFim
        
        inc nrPag              ;se carregou em next
        mov al, bl             ;recarregar o numero da patologia
        jmp list2
        
    listPatologiasFim: 
        ;pop ax
        mov al, 1
        mov aux2, al
        mov nrPag, 0
        jmp voltar       
 
        
;******************* Insert *******************;  

    insertPage:
        call insertPrint 
        
       insertMainLoop:
        call verificaInsert
        
        cmp al, 1
        je adicionaPac
        cmp al, 2
        je guardaEsai
        cmp al, 3
        je guardaEAdiciona
        
        
        adicionaPac:
            call mostraCursor
            ;ler titulo
            mov bh, 0
            mov bl, 1111b
            mov dh, 4   ;posicao yy do cursor
            mov dl, 6   ;posicao xx do cursor
            call setCursorPos   ;para colocar na posicao que queremos
            
            mov dx, offset buffer1
            call readInput  ;faz  flush keyboard buffer and read standard input
            
            ;ler nomes
            mov bh, 0
            mov dl, 6
            mov dh, 8
            call setCursorPos
            
            mov dx, offset buffer2
            call readInput
            
            jmp insertMainLoop
            
            
        guardaEsai:
            call guardaPac
            jmp voltar
            
        
        guardaEAdiciona:
            call guardaPac
            jmp insertPage
            
            
;******************* Process *******************;

    processPage:
        
        call ModoVideo
        call processPrint
        call verificaProcess 
        
        jmp voltar                     
        
        
;***************** Estatistics ******************; 
       
    statsPage:
           
        call ModoVideo 
        call estatisticas
        call verificaEstatistica
        
        cmp al, 1
        je statsPagefim
        
        cmp bl, 0
        je statsPagefim 
        inc nrPag
        jmp statsPage
        
        statsPagefim:
            mov nrPag, 0
            jmp voltar  

               
;******************** About ********************;

    aboutPage:
            call aboutPrint
            call verificaAbout
            jmp voltar 

;********************* Exit ********************; 

    exit: 
    call createDatabase
       ret     
endp
    
;********************************************************************
; Set Video Mode
;Input:
;  AL- Video Mode:
;    00h - text mode. 40x25. 16 colors. 8 pages. 
;    03h - text mode. 80x25. 16 colors. 8 pages.
;    13h - graphical mode. 40x25. 256 colors. 320x200 pixels, 1 page.
;Output:
;destroi: 
;*********************************************************************    
ModoVideo proc
    push ax
    mov al, 13h
    mov ah, 00h
    int 10h
    pop ax
    
    ret
endp

;********************************************************************
;printDisplay
;Input: AL = write mode:
;           bit 0: update cursor after writing;
;           bit 1: string contains attributes.
;       BH = page number.
;       BL = attribute if string contains only characters (bit 1 of AL is zero).
;          8 bit value, low 4 bits set fore color, high 4 bits set background color.
;       CX = number of characters in string (attributes are not counted).
;       DL,DH = column, row at which to start writing.
;       ES:BP points to string to be printed.
;Output:
;destroi: 
;*********************************************************************    

printDisplay proc  
    ;escreve opcao1
    mov al, 0
    mov bh, 0
    mov cx, 18  ;tamanho da string opcao1   
    mov bl, 1111b   ;letras a branco
    mov bh, 0   ;background preto
    mov dl, 2   ;cursor pos xx  
    mov dh, 2   ;cursor pos yy    
    mov bp, offset opcao1   ;string a escrever
    mov ah, 13h
    int 10h  
    
    ;escreve opcao2
    mov cx, 38
    mov dh, 4   ;para mudar pos yy
    mov bp, offset opcao2
    int 10h        
    
    ;escreve opcao3
    mov cx, 22
    mov dh, 6   
    mov bp, offset opcao3
    int 10h

    ;escreve opcao4
    mov cx, 36
    mov dh, 8   
    mov bp, offset opcao4
    int 10h
                   
    ;escreve opcao5
    mov cx, 19
    mov dh, 10  
    mov bp, offset opcao5
    int 10h

    ;escreve opcao6
    mov cx, 6
    mov dh, 12  
    mov bp, offset opcao6
    int 10h 
    
    ;escreve opcao7
    mov cx, 5
    mov dh, 14   ;para mudar pos yy
    mov bp, offset opcao7
    int 10h        
    
                   
    ret
endp   

;****************************************
;displayVerifica
;Rotina que verifica opcao escolhida pelo mouse
;Input: mouse
;Output: AL - opcao escolhida (1-7)
;destroi: 
;**********************************************   
displayVerifica proc
                         
    verificaLoop:         
    
        call getMousePos    ;retorna posicao do mouse cx(xx), dx(yy)

        cmp dx, 10h
        jb verificaLoop     ;verificamos logo se ha hipotese de estar dentro
        cmp dx, 78h         ;dos limites para uma das linhas para otimizar o
        ja verificaLoop     ;tempo de execucao e resposta
        
        ;opcao1?
        mov al, 1           ;antecipando o resultado da proxima linha
        cmp dx, 18h         ;verificar se da pra trocar estas duas linhas ???
        jb fimVerificaLoop
        
        ;opcao2?
        cmp dx, 20h
        jb verificaLoop
        mov al, 2
        cmp dx, 28h
        jb fimVerificaLoop
        
        ;opcao3?
        cmp dx, 30h
        jb verificaLoop
        mov al, 3
        cmp dx, 38h
        jb fimVerificaLoop 
        
        ;opcao4?
        cmp dx, 40h
        jb verificaLoop
        mov al, 4
        cmp dx, 48h
        jb fimVerificaLoop
        
        ;opcao5?
        cmp dx, 50h
        jb verificaLoop
        mov al, 5
        cmp dx, 58h
        jb fimVerificaLoop
        
        ;opcao6?
        cmp dx, 60h
        jb verificaLoop
        mov al, 6
        cmp dx, 68h
        jb fimVerificaLoop 
        
         ;opcao7?
        cmp dx, 70h
        jb verificaLoop
        mov al, 7
        cmp dx, 78h
        jb fimVerificaLoop
                
    fimVerificaLoop:
      ret
endp   
                
;**********************************************************************
;Get Mouse Position and Button pressed
;Input: 
;Output:
;   BX- Button pressed (1 - botão da esquerda, 2 - botão da direita e 3 ambos os botões)
;   CX- posicao xx (coluna)
;   DX- posicao yy (linha)
;destroi: 
;***********************************************************************
getMousePos proc 
    
    push ax 
    push bx 
    
    mov ax,03h 
    verificaBotao:           ;verifica se algum dos botoes esta ativo
        int 33h              ;para poder receber a pos
        cmp bx, 0
        je verificaBotao 
    
    pop bx            
    pop ax
    ret
endp


;**********************************************************************
;set Cursor Position
;Input:
;       bh -> page number (0...7)
;       dl -> coluna xx
;       dh -> linha yy 
;Output:
;destroi: 
;***********************************************************************
setCursorPos proc
    
    push ax
    
    mov ah, 2
    int 10h 
    
    pop ax
    ret
endp
 
;*****************************************************************
;set text-mode cursor shape
;Input: CH = cursor start line (bits 0-4) and options (bits 5-7)
;       CL = bottom cursor line (bits 0-4).
;Output:
;Destroi:
;*****************************************************************
mostraCursor proc
    
    push cx
    
    mov ch, 0
    mov cl, 7   ;show standard blinking text cursor   
    mov ah, 1
    int 10h
    
    pop cx
    ret   
endp
 
;**********************************************************************
;Escreve as informacoes dos pacientes numa pagina
;Input: bl -> nrPag
;Output:         
;destroi: 
;***********************************************************************
listagem proc 
    
    mov al, bl
    mov cl, 10
    mul cl
    inc al
    push ax
    
    mov al, 0
    mov bh, 0    
    mov cx, 39   ;tamanho da string
    mov bl, 1111b   ;cor
    mov dl, 2    ;posicao -x
    mov dh, 2    ;posicao -y
    mov bp, offset listagem2
    call printstrAvancada
    
    pop ax
    mov bh, 0   ;numero da pagina  
    mov cx, 10  ;numero de pacientes maximo por pagina
    mov dl, 2   ;posicao xx do cursor
    mov dh, 4   ;posicao yy do cursor
    
    listagemLoop:
        
        cmp al, nrPac       ;al tem o numero de Pacientes atual
        ja listagemLoopFim  ;chegou ao numero max de pacientes
        
        call setCursorPos   ;mete o cursor na posicao requerida
                            
        call printPac
        
        inc al
        inc dh        ;incrementa a posicao yy para escrever na linha de baixo
        dec cx
        jnz listagemLoop    ;se o cx ainda nao chegou a zero
        
     listagemLoopFim:
        
        mov dh, 16      ;posicao -yy de ambas as strings
        mov bl, 1111b   ;cor de ambas as strings
        mov cx, 8       ;tamanho de ambas as strings
        mov al, 0
        mov bh, 0
        
        mov bp, offset mainText 
        call printstrAvancada
        
        mov dl, 24      ;posicao -xx da string next
        mov bp, offset nextText 
        call printstrAvancada
        
        ret 
endp
 
;**********************************************************************
;Escreve as patologias numa pagina
;Input: bl -> nrPag
;Output:          
;destroi: 
;***********************************************************************
listagemPatologias proc 
    
    mov al, bl
    mov cl, 10
    mul cl
    inc al
    push ax
    
    mov al, 0
    mov bh, 0    
    mov cx, 24   ;tamanho da string
    mov bl, 1111b   ;cor
    mov dl, 2    ;posicao -x
    mov dh, 2    ;posicao -y
    mov bp, offset listagemPat
    call printstrAvancada
    
    pop ax
    mov bh, 0   ;numero da pagina  
    mov cx, 10  ;numero de pacientes maximo por pagina
    mov dl, 2   ;posicao xx do cursor
    mov dh, 4   ;posicao yy do cursor

    listagemPatLoop:
        
        cmp al, nrPatologias       ;al tem o numero de patologias atual
        ja listagemPatLoopFim  ;chegou ao numero max de patologias
        
        call setCursorPos   ;mete o cursor na posicao requerida
                            
        call printPatologia
        
        inc al
        inc dh        ;incrementa a posicao yy para escrever na linha de baixo
        dec cx
        jnz listagemPatLoop    ;se o cx ainda nao chegou a zero
        
     listagemPatLoopFim:
        
        mov dh, 16      ;posicao -yy de ambas as strings
        mov bl, 1111b   ;cor de ambas as strings
        mov cx, 8       ;tamanho de ambas as strings
        mov al, 0
        mov bh, 0
        
        mov bp, offset mainText 
        call printstrAvancada
        
        mov dl, 24      ;posicao -xx da string next
        mov bp, offset nextText 
        call printstrAvancada
        
        ret 
        
endp    

;**********************************************************************
;verifica se utilizador carrega no botao next ou main
;Input: 
;Output:
;destroi: 
;***********************************************************************
verificaListagem proc 
    
    listagemLoop2:
    
    call getMousePos ;devolve cx->xx  dx->yy                      
    
    cmp dx, 127          ;verifica o yy primeiro para otimizar o programa
    jb listagemLoop2
    cmp dx, 135
    ja listagemLoop2 
    
    cmp cx, 32           ;verifica se e o botao main
    jb listagemLoop2
    mov al, 1            ;(antecipado) se sim al=1
    cmp cx, 150
    jb listagemLoop2Fim
    
    cmp cx, 382          ;verifica se e o botao next
    jb listagemLoop2
    cmp cx, 508
    ja listagemLoop2
    mov al, 2            ;(antecipado) se sim al=2
    
    listagemLoop2Fim:
        ret
endp        

;**********************************************************************
;verifica se utilizador carrega no botao next ou main
;Input: 
;Output:
;destroi: 
;***********************************************************************
verificaListagemPatologias proc 
    push bx
    push cx
    push ax  
    
    mov al, 0
    mov al, 10;nrPatologias
    mov cx, 8           ;intervalo de valores do yy de cada linha
    mul cx   
    add ax, 20h
    mov bx, ax          ;bx= numero limite do yy da ultima patologia
    pop ax
    
    listagemLoop3:
    
    call getMousePos ;devolve cx->xx  dx->yy  
    
    cmp dx, 20h           ;verifica se esta acima da primeira opcao
    jb listagemLoop3                                              
    
    cmp dx, bx
    jbe carregou1         ;menor ou igual
                                
    cmp dx, 127          ;verifica o yy primeiro para otimizar o programa
    jb listagemLoop3
    cmp dx, 135
    ja listagemLoop3 
    
    cmp cx, 32           ;verifica se e o botao main
    jb listagemLoop3
    mov al, 11            ;(antecipado) se sim al=1
    cmp cx, 150
    jb listagemLoop3Fim
    
    cmp cx, 382          ;verifica se e o botao next
    jb listagemLoop3
    cmp cx, 508
    ja listagemLoop3
    mov al, 12            ;(antecipado) se sim al=2 
    jmp listagemLoop3Fim
    
    carregou1:     ;podemos utilizar cx, bx=n maximo logo 20<dx<bx
        mov al, 1
        mov cx, 20h  
        
        carregou2:
            add cx, 8
            cmp dx, cx
            jbe listagemLoop3Fim
            inc al
                          
            cmp al, 10
            ja listagemLoop3    ;pois nao pode passar de 10 elementos por pagina 
            
            cmp cx, bx
            je listagemLoop3    ;nao era nenhuma das opcoes
            jmp carregou2      
        
        
    listagemLoop3Fim:
        pop cx
        pop bx 
        ret
endp 


;**********************************************************************
;Escreve as patologias numa pagina
;Input: al contem o numero da patologia referente a structPatologias
;       bl tem o numero da pagina
;Output:          
;destroi: 
;***********************************************************************
PacientesComPatologia proc
    push ax
        
    mov ah, 0
    dec al       ;para aceder a posicao correta
    mov cl, 51
    mul cl       ;al fica com o numero da posicao que queremos aceder
              
    mov si, offset structPatologias
    add si,ax                       
    push ax
    
    ;escreve o nome da patologia especifica
    mov al, 0
    mov bh, 0 
    mov dl, 2    ;posicao -x
    mov dh, 2    ;posicao -y
    mov bh, 0   ;numero da pagina  

    call setCursorPos   ;mete o cursor na posicao requerida
       
    mov ah, 2
                
    mov dl, '<'
    call printchar
    
    mov dx, si
    call printstr

    mov dl, '>'
    call printchar
    
    ;escreve cabecalho
    mov cx, 26   ;tamanho da string
    mov bl, 1111b   ;cor
    mov dl, 2    ;posicao -x
    mov dh, 4    ;posicao -y
    mov bp, offset listagemPat2  
    call setCursorPos
    call printstrAvancada   
    
    mov ah, 0
    mov al, nrPag
    mov cl, 10
    mul cl
    cmp al, 0
    je zero    
    inc al      ;se nao estiver na primeira pagina 
    
    zero: 
    mov cl, 102
    mul cl      ;ax fica com a posicao da estrutura structDB que vamos aceder
   
    mov di, offset structDB
    add di, ax
    add di, 51  
    
    pop ax
    mov bh, 0   ;numero da pagina  
    mov al, 10  ;numero de pacientes maximo por pagina
    mov dl, 2   ;posicao xx do cursor
    mov dh, 5   ;posicao yy do cursor

    mov bl, nrPac
    
    
    patVerifica:       
        mov cx, 49;offset si-$ ;5
        push ax
        
        call strcmp
        
        cmp al, 1
        je escreveNome1
        
        pop ax
        volta1:
            add di, 102     ;proxima hipotese 
            dec bl
            jz patVerificaFim   
                    
            jmp patVerifica
      
    
    escreveNome1:        
        
        sub di, 50 
        inc dh  
        call setCursorPos      
        call escreveNome  
        add di, 50    
        
        pop ax
        dec al
        jz patVerificaFim 
     
        jmp volta1
        
    patVerificaFim:
        
        mov dh, 16      ;posicao -yy de ambas as strings
        mov bl, 1111b   ;cor de ambas as strings
        mov cx, 8       ;tamanho de ambas as strings
        mov al, 0
        mov bh, 0
        
        mov bp, offset mainText 
        call printstrAvancada
        
        mov dl, 24      ;posicao -xx da string next
        mov bp, offset nextText 
        call printstrAvancada       
        
        pop ax
        ret 

endp

;**********************************************************************
;Escreve o nome do paciente pretendido
;Input: 
;Output:
;destroi: 
;***********************************************************************
escreveNome proc    
        push cx
        push ax
        push si        
        push dx      
        
        mov ah, 2
                
        mov dl, '['
        call printchar
        
        mov ah, 0  ;limpa o byte anterior ao apontado por si 
        mov al, aux2
        call escreveNumero
        
        mov dl, ']'
        call printchar
        mov dl, ' '
        call printchar

        mov dl, '"'
        call printchar

        mov dx, di
        call printstr

        mov dl, '"'
        call printchar
        
        inc aux2
    
        pop dx
        pop si
        pop ax
        pop cx
        ret  
        
endp

;**********************************************************************
;Escreve que a lista esta vazia se nao houver pacientes
;Input: 
;Output:
;destroi: 
;***********************************************************************
 listagemSPac proc
    
     mov al, 0
     mov bh, 0
     mov cx, 24  ;tamanho da string
     mov bl, 1111b   ;cor
     mov dl, 2    ;posicao -x
     mov dh, 2    ;posicao -y
     mov bp, offset listagem1
     call printstrAvancada 
    
     mov cx, 8
     mov dh, 14
     mov bp, offset mainText 
     call printstrAvancada	
	
     ret 
endp  	     
            
;**********************************************************************
;verifica se utilizador quer sair da listagem (quando nao ha pacientes)
;Input: 
;Output:
;destroi: 
;***********************************************************************
verificaListagemSPac proc
    
    listagemSPacLoop:
    
    call getMousePos
    
    cmp dx, 111     ;verifica se carrega no botao main
    jb listagemSPacLoop
    cmp dx, 117
    ja listagemSPacLoop

    ret
endp        
        
;**********************************************************************
;escreve o paciente que tem o numero do al
;Input: 
;Output:           
;destroi:         
;***********************************************************************
printPac proc
        push ax                         ;guardar registos, pq sao destruidos a meio
        push dx
        push cx
        
        dec al                          ;dec do al, para a aceder a nPac-1
        mov ah, 0
        mov cl, 102
        mul cl                          ;multiplicacao de al com 102 para obter offset interno da estrutura
      
        mov si, offset structDB
        add si, ax
        mov ah, 2
        
        push si
        
        mov dl, '['
        call printchar
        
        mov ax,[si]
        mov ah, 0  ;limpa o byte anterior ao apontado por si
        mov si, offset buffer3
        call WriteUns  
        mov dx, si
        call printstr
        pop si
        
        mov dl, ']'
        call printchar
        mov dl, ' '
        call printchar
        mov dl, '"'
        call printchar
       
        inc si
        mov dx, si
        call printstr
        
        mov dl, '"'
        call printchar
        mov dl, ','
        call printchar
        mov dl, ' '
        call printchar
        
        add si, 50
        mov dx, si
        call printstr
        
        pop cx
        pop dx
        pop ax
        
        ret
endp

;**********************************************************************
;escreve a patologia
;Input: 
;Output:           
;destroi:         
;***********************************************************************
printPatologia proc
        
        push cx
        push dx
        push ax
        push si
        
        push ax          

        dec al                          ;dec do al, para a aceder a nPac-1
        mov ah, 0
        mov cl, 51
        mul cl                          ;multiplicacao de al com 51 para obter offset interno da estrutura
              
        mov si, offset structPatologias
        add si, ax
        
        pop ax
        mov ah, 2
                
        mov dl, '['
        call printchar
        
        mov ah, 0  ;limpa o byte anterior ao apontado por si
        call escreveNumero
        
        mov dl, ']'
        call printchar
        mov dl, ' '
        call printchar
       
        mov dx, si
        call printstr
        
        pop si
        pop ax
        pop dx
        pop cx
        ret

endp

;****************************************************************
;Descricao: guarda a informacao de um paciente com os dados que o 
;           utilizador introduziu
;Input: buffer1 -> titulo
;       buffer2 -> nomes
;Output:
;Destroi:
;****************************************************************

guardaPac proc
    
    inc nrPac   ;vai ser o numero do novo paciente
    mov di, offset structDB
    mov ax, 0
    mov al, nrPac
    push ax     ;para ficar com nrPac guardado
    
    dec ax      ;decrementa nrPac para acader ao indice da estrutura em nrPac-1
    mov cl, 102  ;estrutura do paciente (1+50+50+1)
    mul cl      ;multiplica nrPac*102 pra aceder a posiçao desse paciente na struct
    
    add di, ax  ;para ter o offset na posicao da struct onde queremos adicionar
    pop bx      ;poe nrPac em bx
    mov [di], bl    ;mete numero do Pac na posicao
    
    cld     ;para podermos usar o rep movsb
    
    inc di  ;para passar para a posicao do nome
    push di
    mov si, offset buffer1  ;para passar titulo para a struct
    mov cx, [si+1]
    mov ch, 0
    add si, 2    
    rep movsb       ;Copia o byte em DS:[SI] para ES:[DI], o titulo    
    mov [di], '$'
    
    pop di
    add di, 50      ;passa para a posicao da patologia
    mov si, offset buffer2  
    mov cx, [si+1]
    mov ch, 0
    push cx
    add si, 2
    rep movsb       ;Copia o byte em DS:[SI] para ES:[DI], os nomes
    mov [di], '$'
    
    pop cx
    call verificaEGuardaPat

    
    ret    
    
endp

;******************************************************************************
;guarda a informcao da patologia(em si) que foi introduzida com o novo paciente
;Input: 
;Output:
;destroi:si di 
;******************************************************************************
verificaEGuardaPat proc  
    push si
    push di
    push bx
    push ax
    
    sub di, cx
    mov si, di
    mov bx, 1 
    mov ax, 0
    call verificaPatologia
    
    pop ax
    pop bx
    pop di
    pop si
    ret    
    
    
endp


;**********************************************************************
;faz load da struct com os tipos de patologias que existem
;Input: 
;Output:
;destroi: 
;***********************************************************************
loadPatologias proc
    
    push ax
    push si
    push di
    
    
    mov si, offset structDB    
    mov ax, 0
    mov al, nrPac
    
    add si, 51      ;passa para a posicao dos nomes
    
    cicloVerificaPatologia:
        cmp ax, 0
        je fimloadPatologias
        mov bx, 0
        call verificaPatologia
        
        dec ax   
        
        mov dx, 0
        mov dl, aux
        add si, 52 
        add si, dx
        
        jmp cicloVerificaPatologia
    
    fimloadPatologias:
        pop di
        pop si
        pop ax
        ret    
    
    
endp    

;**********************************************************************
;verifica se um determinado tipo de patologias ja existe
;Input: 
;Output:
;destroi: 
;***********************************************************************
verificaPatologia proc
    
    push ax
    push dx
    
    
    mov di, offset structPatologias
    mov dx, 0
    mov dl, nrPatologias
    
    cicloPatologia:
        cmp dx, 0
        je fimCicloPatologia
        
        cmp bx, 1
        je label1
        mov cx, 49
        
        
        label1:
        call strcmp         ;compara as duas strings
        cmp ax, 1
        je igual
        
        dec dx
        add di, 51          ;passa para a proxima hipotese de patologia
        jmp cicloPatologia
        
        igual:
            add di, 49      ;para compensar o push e pop do si e di na strcmp
            add si, 49
            inc di          ;mete o di nas utilizacoes
            inc [di]        ;aumenta o numero de utilizacoes
            jmp exitVerifica
            
    fimCicloPatologia:
        call guardaPatologia
        
        exitVerifica:
        
            pop dx
            pop ax
            ret
        
        
endp    


;*******************************************************
;Descricao: guarda a informacao de uma patologia nova          
;Input: 
;Output:
;Destroi:
;*******************************************************
guardaPatologia proc
    
    push cx
    push ax
    
    inc nrPatologias
    mov di, offset structPatologias
    
    mov ax, 0  
    mov al, nrPatologias
    dec ax      ;decrementa para acader ao indice da estrutura em nrPatologias-1
    
    mov cl, 51          ;51=tamanho de uma patologia
    mul cl              ;multiplica nrPatologias*51 pra aceder a posicao desse paciente na struct
    
    add di, ax          ;nova pos da patologia
    push di
    
    cld
    
    mov cx, 49
    mov ch, 0
    rep movsb           ;Copia o byte em DS:[SI] para ES:[DI]    
    mov [di], '$'
    
    pop di
    add di, 49             
    inc di              ;passa para a posicao das utilizacoes
    mov [di], 1         ;mete utilizacoes a 1
    
    pop ax
    pop cx
    ret
    
endp

;**********************************************************************
;compara duas strings com o mesmo tamanho (em cx) situadas em si e di
;Input: 
;Output: 1- caso sejam iguais  0- caso contrario
;destroi: ax
;***********************************************************************
strcmp proc
    push si       
    push di
    push cx       
    
; compare until equal:
        repe cmpsb
        jnz not_equal

        mov al, 1

        jmp exit_here

        not_equal:        
            mov al, 0  
                
        exit_here:        
        pop cx
        pop di
        pop si        
        ret
endp
         
;*******************************************************
;Descricao: Imprime no ecra a estrutura da pagina Insert 
;Input:
;Output:
;Destroi:
;*******************************************************
insertPrint proc  
    
        call ModoVideo
        
        mov al, 0
        mov bl, 1111b
        mov bh, 0
        mov cx, 17  ;tamanho string
        mov dl, 2   ;posicao xx
        mov dh, 2   ;posicao yy
        mov bp, offset insert1
        call printStrAvancada
        
        mov cx, 11  
        mov dh, 6   ;posicao yy
        mov bp, offset insert2
        call printStrAvancada
        
        mov cx, 8   ;tamanho de ambas as strings
        mov dh, 12  ;posicao yy do botao main e do botao next
        
        mov bp, offset mainText
        call printStrAvancada
        
        mov dl, 24  ;posicao xx do botao next
        mov bp, offset nextText
        call printStrAvancada 
        
        ret
endp
        

;**********************************************************************
;verifica se o utilizador quer escrever informacoes sobre o
;paciente e se carrega no botao main ou next
;Input: 
;Output:
;destroi: 
;***********************************************************************
verificaInsert proc
    
    insertLoop:
    
        call getMousePos ;devolve cx->xx  dx->yy
        
        cmp dx, 16     ;verifica se esta dentro dos limites
        jb insertLoop 
        cmp dx, 103     
        ja insertLoop
        
        mov al, 1
        cmp dx, 46     ;verifica se quer escrever
        jb insertLoopFim    
        
      insertLoop2:
        
        cmp dx, 95     ;verifica se esta dentro dos limites
        jb insertLoop  ;dos botoes main e next
        
        cmp cx, 32           ;verifica se e o botao main
        jb insertLoop
        mov al, 2            ;(antecipado) se sim al=2
        cmp cx, 156
        jb insertLoopFim
    
        cmp cx, 382          ;verifica se e o botao next
        jb insertLoop
        cmp cx, 508
        ja insertLoop
        mov al, 3            ;(antecipado) se sim al=3
    
    insertLoopFim: 
        ret
endp

;**************************************************************
;Descricao: Imprime no ecra os pacientes associados ao artigo 
;Input:
;Output:
;Destroi:
;**************************************************************
processPrint proc
    
    mov al, 0
    mov bh, 0   
    mov cx, 39   ;tamanho da string
    mov bl, 1111b   ;cor
    mov dl, 2    ;posicao -x
    mov dh, 2    ;posicao -y
    mov bp, offset listagem2
    call printstrAvancada
    
    mov dh, 4   ;muda de linha para escrever referncias
    
    
    ProcessLoopFim:
        mov al, 0
        mov dh, 20
        mov cx, 8
        mov bp, offset mainText
        call printStrAvancada 
    
        ret    
endp


;**********************************************************************
;verifica se o utilizador carrega no botao main da pagina process
;Input: 
;Output:
;destroi: 
;***********************************************************************
verificaProcess proc  
    
    processLoop:
    
        call getMousePos

        cmp dx, 111     ;verifica se carrega no botao main
        jb processLoop
        cmp dx, 119
        ja processLoop
        cmp cx, 30
        jb processLoop
        cmp cx, 156
        ja processLoop
    
        ret    
endp

;*******************************************************
;Descricao: Imprime as estatisticas do trabalho 
;Input:     bl o nrPatologias que faltam mostrar
;Output:
;Destroi:
;*******************************************************
estatisticas proc
    push bx 
    mov ax, 0
    mov al, nrPag
    mov cl, 6
    mul cl  
    push ax
    mov cl, 51
    mul cl
    push ax
    
    mov al, 0
    mov bh, 0    
    mov bl, 1111b   ;cor
    mov dl, 2    ;posicao -x
    mov dh, 2    ;posicao -y  
    ;estatistica1
    mov cx, 16   ;tamanho da string
    mov bp, offset estatistica1
    call printstrAvancada
    
    ;imprime numero de pacientes
    mov al, nrPac
    
    mov dl, 12h    ;aumenta posicao -x 
    mov dh, 2   
    call setCursorPos      
    call escreveNumero    
    
    ;estatistica2
    mov al, 0
    mov bh, 0    
    mov bl, 1111b   ;cor
    mov dl, 2    ;posicao -x
    mov dh, 4   ;muda de linha 
    mov cx, 8   ;tamanho da string
    mov bp, offset estatistica2
    call printstrAvancada   
    
    ;estatistica3, 4 e 5   9,4,10
    mov dh, 6   ;muda de linha 
    mov cx, 9   ;tamanho da string
    mov bp, offset estatistica3
    call printstrAvancada
    
    mov dl, 12h   
    mov cx, 4   ;tamanho da string
    mov bp, offset estatistica4
    call printstrAvancada
   
    mov dl, 19h   
    mov cx, 10   ;tamanho da string
    mov bp, offset estatistica5
    call printstrAvancada      
    
    mov dh, 8
    pop ax
    mov si, offset structPatologias
    add si, ax  
    pop ax
    mov bx, 0
    pop bx      ;tem o nrPatologias que faltam mostrar
    mov cx, 6   ;numero maximo de patologias por pagina
      
    ;escreve as estatisticas de cada patologia
    escreveEstatistica:
        cmp al, nrPatologias
        jae fimEstatistica

        push ax
                
        mov dl, 2
        push dx                
        call setCursorPos
        mov dx, si
        call printstr       ;escreve o nome
        pop dx
        add si, 50
        add dl, 10h         ;mete na posicao do numero
        mov al, [si]
        call setCursorPos
        call escreveNumero  ;escreve o numero de utilizacoes
        
        add dl, 7h
        call setCursorPos
        call percentagem
        call escreveNumero 
        
        mov dl, '%'
        call printchar

                  
        add dh, 2
        inc si    
        
        pop ax
        inc ax
        
        dec cx
        jz fimEstatistica
        
        dec bx
        jz fimEstatistica

        jmp escreveEstatistica 
        
    fimEstatistica:
        push bx
        mov al, 0
        mov bl, 1111b
        mov bh, 0
        
        mov cx, 8   ;tamanho de ambas as strings
        mov dh, 22  ;posicao yy do botao main e do botao next 
        
        mov dl, 2  ;posicao xx do botao next
        mov bp, offset mainText
        call printStrAvancada
        
        mov dl, 24  ;posicao xx do botao next
        mov bp, offset nextText
        call printStrAvancada 
   
        
        call getMousePos
        pop bx
        ret
    
endp

;*******************************************************
;Descricao: faz a percentagem da parte estatistica 
;Input: bl-> nrPatologias   [si]->numero de utilizacoes
;Output:  al fica com o numero da percentagem
;Destroi:
;*******************************************************
percentagem proc
    
    push cx
    push bx
    push dx  
    
    mov bl, nrPac
    
    mov al, 100
    ;mov cl, bl
    div bl   
    mov bl, al
    
    cmp ah, 0           ;al tem resultado  ah tem o resto da divisao
    je divisaoCerta
    
    ;inc al    
    ;mov bl, al          ;resultado em bl
    mov cl, 10                          
    mov al, ah
    mov ah, 0
    mul cl   
    mov cl, nrPac
    div cl              ;al=resto*100/nrPac
    cmp al, 5
    jb divisaoCerta
    
    inc bl
    
    divisaoCerta:
        mov al, bl
        mov ah, 0
        mov cl, [si]
        mul cl 
        
        pop dx
        pop bx
        pop cx
        ret  
            
endp

;**********************************************************************
;verifica se o utilizador carrega no botao main da pagina estatistics
;Input: 
;Output:
;destroi: 
;***********************************************************************
verificaEstatistica proc  
    
    estatisticaLoop:
    
        call getMousePos     ;devolve cx->xx  dx->yy
                             
        cmp dx, 175          ;verifica se esta dentro dos limites
        jb estatisticaLoop   ;dos botoes main e next
        
        cmp dx, 182          ;verifica se esta dentro dos limites
        ja estatisticaLoop   ;dos botoes main e next

        
        cmp cx, 32           ;verifica se e o botao main
        jb estatisticaLoop
        mov al, 1            ;(antecipado) se sim al=2
        cmp cx, 155
        jb estatisticaLoopFim
    
        cmp cx, 382          ;verifica se e o botao next
        jb estatisticaLoop
        cmp cx, 508
        ja estatisticaLoop
        mov al, 2            ;(antecipado) se sim al=3
    
    estatisticaLoopFim: 
        ret
endp



;*******************************************************
;Descricao: escreve o numero que esta em al no ecra 
;Input: dx tem as coordenadas
;Output: 
;Destroi:
;*******************************************************
escreveNumero proc
    push bx
    push si
    push di
    push dx
    
    mov ah, 0
    mov si, offset buffer4  ;auxiliar em writeUNS
    mov di, offset buffer5  ;fica com o resultado 
    
    call WriteUns           ;al tem o numero si aponta para um buffer e di recebe o numero em string
    
    mov di, offset buffer4  
    mov dx, di
    call printstr
    
    pop dx
    pop di
    pop si
    pop bx
    ret
    
endp

;*******************************************************
;Descricao: Imprime no ecra a informacoes dos alunos 
;Input:
;Output:
;Destroi:
;*******************************************************
aboutPrint proc      
    
    call ModoVideo
    mov al, 0
    mov bh, 0
    mov cx, 25              ;tamanho das 3 strings
    mov bl, 1111b
    
    mov dl, 2
    mov dh, 2
    mov bp, offset aboutInfo
    call printStrAvancada
    
    mov dh, 4
    mov bp, offset aluno1
    call printStrAvancada
    
    mov dh, 6
    mov bp, offset aluno2
    call printStrAvancada
    
    mov dh, 14
    mov cx, 8
    mov bp, offset mainText
    call printStrAvancada
    
    ret
endp

;**********************************************************************
;verifica se o utilizador carrega no botao main da pagina about
;Input: 
;Output:
;destroi: 
;***********************************************************************
verificaAbout proc  
    
    aboutLoop:
    
        call getMousePos

        cmp dx, 111     ;verifica se carrega no botao main
        jb aboutLoop
        cmp dx, 119
        ja aboutLoop
        cmp cx, 30
        jb aboutLoop
        cmp cx, 156
        ja aboutLoop
    
        ret    
endp

;******** FICHEIROS ***********;



;*****************************************************************
;fcreate - cria um ficheiro
;input - DS:DX = buffer com o nome CX- file attributes
;Output - if successful: CF clear, AX=file handle, 
;          else: CF on, AX=error code   
;destroi - 
;*****************************************************************    
fCreate proc
    
     mov ah, 3ch
     int 21h       
     ret
endp


;******************************************************************************************
;fopen - abre um ficheiro
;Descricao: rotina que abre um dado ficheiro com dado nome para, leitura, escrita ou append 
;input - AL= modo de abertura, DS:DX=file name, 
;         BX= origem o cursor(0-inicio,1-atual,2-fim), CX=offset da origem do cursor, 1byte
;output - sucesso: CF clear, AX=file handle, DX:AX= nova posicao de abertura(bytes)
;          else: CF on, AX=error code   
;destroi - 
;******************************************************************************************
fOpen proc 
        
     mov ah, 3dh
     int 21h        ;retorna ax = handle ou error code
     push ax
     jc erro
     push bx
     mov bx, ax     ;handler para seek
     pop ax         ;origem do cursor para seek que estava em bx
     mov dx, cx     ;offset da origem do cursor
     mov cx, 0      ;offset de abertura 8bits mais significativos a zero
     mov ah, 42h
     int 21h        ;file seek
                
    erro:        
       pop ax       
       ret
endp


;*****************************************************************
;fread - le do ficheiro
;Descricao: rotina que le do ficheiro n bytes (que sao guardados no ax)
;Input - BX = file handler, CX = n bytes, DS:DX = buffer para data
;Output - sucesso: CF clear, AX=n bytes lidos, 0 se apanha um EOF
;          else: CF on, AX=error code   
; destroi - 
;***************************************************************** 
fRead proc
    
     mov ah, 3fh
     int 21h
     ret
endp
 

;*****************************************************************
;fwrite - escreve no ficheiro
;Descricao: rotina que escreve no ficheiro 
;Input - BX = file handler, CX = n bytes, DS:DX = data a escrever
;Output - sucesso: CF clear, AX=numero de bytes escritos
;          else: CF on, AX=error code   
;Destroi - 
;*****************************************************************
fWrite proc   
    
     mov ah, 40h
     int 21h
     ret
endp     

;******************************************************************************************
;findNum - encontra um numero dentro do ficheiro 
;Input - BX = file handle   DS:DX = buffer para data  
;Output - sucesso: CF clear, AX = numero encontrado
;          else: CF on, AX=error code   
;******************************************************************************************
findNum proc
    
    push cx
    
    mov cx, 1   ;para procurar apenas um digito de cada vez
    
    findNumLoop:
        call fRead  ;procura no ficheiro
        cmp ax, 0
        jz findNumLoopFim   ;se o ficheiro chegar ao fim (EOF)
        
       ; mov ax,     ;;;encontrar maneira de por o caracter lido
                    ;;;em ax
        cmp al, 30h 
        jb findNumLoop  ;se nao for um numero
        cmp al, 39h
        ja findNumLoop
        
        sub al, 30h ;para transformar numero (caracter) em numero real
        
    findNumLoopFim:
        pop cx
        
        ret 
        
    
    
endp



;******************************************************************************************
;fclose - fecha um ficheiro 
;Input - BX = file handle 
;Output - sucesso: CF clear, AX destroyed
;          else: CF on, AX=error code   
;******************************************************************************************
fClose proc
    
    mov ah,3eh     
    int 21h           
endp 


;********** INPUT E OUTPUT ***********;
 
;*****************************************************************
; WriteUns - output de um numero sem sinal de 2bytes 
; descricao: faz output de um numero guardado em AX
; input - AX=numero ai guardado SI=offset do vetor
; output - di fica com o numero guardado como string
; destroi - ax
;***************************************************************** 
WriteUns proc
    push si
    mov bx, 10
    mov di, si
    mov dx, 0
        
    divisao:  
        div bx
        mov [si], dx ;guarda o resto 
        add [si], 30h           
        push [si] 
        cmp ax, 0 ;verifica se divisao = 0 
        mov dx, 0
        je fimDivisao
        inc si
        jmp divisao
        
    fimDivisao:  
        pop ax
        mov [di],ax
        cmp di, si
        je writeUnsEnd
        inc di
        jmp fimDivisao  
       
    writeUnsEnd:
        mov [di+1],'$'
        pop si    
         
    ret
endp  

;*************************************************************
;descricao: imprime uma string no ecra 
;Input: AL = write mode:
;           bit 0: update cursor after writing;
;           bit 1: string contains attributes.
;       BH = page number.
;       BL = attribute if string contains only characters (bit 1 of AL is zero).
;          8 bit value, low 4 bits set fore color, high 4 bits set background color.
;       CX = number of characters in string (attributes are not counted).
;       DL,DH = column, row at which to start writing.
;       ES:BP points to string to be printed.
;*************************************************************
printstrAvancada proc
    
    push ax    
    mov ah, 13h
    int 10h
    pop ax
    
    ret
    
endp     


;*************************************************************
;descricao: imprime uma string que se encontra em dx no ecra 
;a diferenca para o INT 10h/13h é que esta string termina em $
;*************************************************************
printstr proc
    push ax    
    mov ah, 09h
    int 21h
    pop ax
    ret
    
endp     

;**************************************
;Descricao: imprime um caracter no ecra
;input: dl->caracter pretendido
;**************************************
printchar proc
        
    push ax
    mov al, 0
    mov ah,02h       
    int 21H
    pop ax    
    ret    
    
endp

;**************************************************************
;Descricao: flush keyboard buffer and read standard input
;**************************************************************
readInput proc
    
    push ax
    
    mov al, 0ah  ;number of input function to execute after flushing buffer
    mov ah, 0ch   
    int 21h
 
    pop ax
    ret
endp
        
;***********************************************************************************
;ReadStr - string read
;descricao: rotina que le uma string do teclado para o endereco dado pelo registo DI
;input - teclado = caracter a guardar
;output - DI = endereco da string
;destroi - nada
;***********************************************************************************
ReadStr proc     
    push ax 
L2: 
    mov ah, 01H     ;read character from standard input, result is stored in AL
    int 21h         ;if there is no character in the keyboard buffer, the function waits until any key is pressed
    cmp al,0dH      ;ate carregar enter 
    je fimRead
    mov byte ptr[di], al
    inc di
    jmp L2  
    
    fimRead: 
        pop ax
        ret
   
endp 
        
createDatabase proc
    
     mov dx, offset FileNameDB
     mov cx, 0    ;para funcao fCreate
     call fCreate
     push ax      ;guarda handler
     mov al, 1    ;modo write
     mov bx, 0    ;cursor no inicio do ficheiro
     call fOpen
     
     pop bx       ;recupera handler
     mov cl, 102     ;numero de bytes de cada referencia
     mov al, 1    ;contador de Ref
     mov dx, offset structDB
     
     mov al, nrPac
     mul cl
     mov cx, ax  ;em fWrite cx e numero de bytes a escrever
     call fWrite
     
     ret
endp


loadDatabase proc
    mov al, 0   ;por em modo read
    mov bx, 0   ;cursor no inicio
    mov cx, 0                    
    mov dx, offset fileNameDB
    call fOpen
    
    mov bx, ax  ;handler para a funcao fRead
    mov cx, 102 ; numero de bytes de cada paciente
    mov dx, offset structDB ;para a funcao fRead
    
    loadDatabaseLoop:
        call fRead
        cmp ax, 0   ;verifica se chegou ao fim (EOF)
        jz loadDatabaseLoopFim
        inc nrPac
        add dx, cx  ;para passar para a proxima linha
        
        jmp loadDatabaseLoop
        
     loadDatabaseLoopFim:
        ret
endp
        
ends

end start ; set entry point and stop the assembler.
        
        
        
        
        
        
        
    