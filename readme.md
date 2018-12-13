## Por que o Asterisk consome 100% da CPU?

Eu não sei !

Muitas pessoas me perguntaram isso algumas vezes ultimamente e minha resposta é sempre “eu não sei”. No entanto, o ps pode fornecer mais informações sobre isso. Na verdade, isso funciona para qualquer aplicativo que você tenha e deseja depurar porque está ficando maluco.

Primeiro, verifique qual segmento (Asterisk é um aplicativo multi-threaded) está ficando louco.

` ps -LlFm -p 'pidof asterisk'` 

Isso deve mostrar a porcentagem de CPU que está sendo usada por cada thread do Asterisk na coluna chamada "C" e, em seguida, anote o valor da coluna LWP para o segmento em que você está interessado. (LWP é um número de processo leve, grosso modo, o id do segmento). Agora que você tem o id do segmento, você precisa saber o que esse segmento está fazendo.

`pstack 'pidof asterisk' > /tmp/asterisk.stack.txt`

Isso fará com que o processo de asterisco despeje o estado da pilha no arquivo /tmp/asterisk.stack.txt. Se você não tem o comando pstack do google para ele, eu acho que no CentOS é tão fácil quanto o yum install pstack.

Em seguida, abra o arquivo e procure o LWP que você acabou de escrever. Espero que você encontre algumas dicas que permitam que você saiba como evitá-lo ou pelo menos muito mais informações para postar em bugs.digium.com

**ATUALIZAÇÃO:**  
Um dos caras que fez essa pergunta me contou o que achou:

Thread 10 (Thread 0x41d8f940 (LWP 3406)):  

> 0 0x00000033ce2ca436 in poll () from /lib64/libc.so.6  
> 1 0x00000000004933c0 in ast_io_wait ()  
> 2 0x00002aaabd9510cd in network_thread ()  
> 3 0x00000000004f8b2c in dummy_start ()  
> 4 0x00000033cee06367 in start_thread () from /lib64/libpthread.so.0  
> 5 0x00000033ce2d2f7d in clone () from /lib64/libc.so.6

Um grep -rI “network_thread” no código-fonte do Asterisk revela que esta função pertence a chan_iax.c, desabilitar o chan_iax.so em modules.conf é uma boa solução para o seu problema, no entanto depuração adicional seria necessária para determinar porque o thread do monitor está dando voltas assim.

> Desempenho do g729 retirado do site da Digium: Testes internos com dois processadores Intel Xeon 1.8GHz permitiram 60 chamadas simultâneas . Os processadores Dual Xeon de 2,8 GHz permitiram 80 chamadas simultâneas .

Saiba Mais:
https://www.voip-info.org/asterisk-dimensioning/


> Ajuste de desempenho

`[threadpool]
;
; For a busy 8 core PBX, these settings are probably safe.
;
initial_size = 10
idle_timeout_sec = 120
;
; The notes about the pjsip max size apply here as well.  Increasing to 100 threads is probably
; safe, but anything more will probably cause the same thrashing and memory over-utilization,
max_size = 60`

https://wiki.asterisk.org/wiki/display/AST/Performance+Tuning