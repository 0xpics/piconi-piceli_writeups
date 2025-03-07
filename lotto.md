# Duckware Team - lotto (pwnable)

###### Solved by @0xpics and @JoaoPedroPiceli

>This CTF is about pwn, code vulnerability

## Sobre o desafio

O desafio tem como objetivo fazer com que usuário explore as vulnerabilidades "escondidas" em um código e tome vantagem delas sem ajuda de ferramentas externas.

## O Desafio

Para realizar o desafio o usuário deve conectar a um desktop fornecido pelo site. Ao se conectar o usuário irá se deparar com esse arquivo `lotto`

Ao executá-lo a seguinte mensagem irá aparece:

__Lotto Rule__

__lotto is consisted with 6 random natural numbers less than 46__

__your goal is to match lotto numbers as many as you can__

__if you win lottery for *1st place*, you will get reward__

__for more details, follow the link below
http://www.nlotto.co.kr/counsel.do?method=playerGuide#buying_guide01__

__mathematical chance to win this game is known to be 1/8145060__

Ao colocarmos um sequência aleatória de 6 números como requisitado conseguimos a seguinte mensagem:

`bad luck...`

Colocando outras sequências aleatórias conseguimos o mesmo resultado, vamos analisar o código então.

## O Código

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>

unsigned char submit[6];

void play(){
        
        int i;
        printf("Submit your 6 lotto bytes : ");
        fflush(stdout);

        int r;
        r = read(0, submit, 6);

        printf("Lotto Start!\n");
        //sleep(1);

        // generate lotto numbers
        int fd = open("/dev/urandom", O_RDONLY);
        if(fd==-1){
                printf("error. tell admin\n");
                exit(-1);
        }
        unsigned char lotto[6];
        if(read(fd, lotto, 6) != 6){
                printf("error2. tell admin\n");
                exit(-1);
        }
        for(i=0; i<6; i++){
                lotto[i] = (lotto[i] % 45) + 1;         // 1 ~ 45
        }
        close(fd);
        
        // calculate lotto score
        int match = 0, j = 0;
        for(i=0; i<6; i++){
                for(j=0; j<6; j++){
                        if(lotto[i] == submit[j]){
                                match++;
                        }
                }
        }

        // win!
        if(match == 6){
}
        unsigned char lotto[6];
        if(read(fd, lotto, 6) != 6){
                printf("error2. tell admin\n");
                exit(-1);
        }
        for(i=0; i<6; i++){
                lotto[i] = (lotto[i] % 45) + 1;         // 1 ~ 45
        }
        close(fd);
        
        // calculate lotto score
        int match = 0, j = 0;
        for(i=0; i<6; i++){
                for(j=0; j<6; j++){
                        if(lotto[i] == submit[j]){
                                match++;
                        }
                }
        }

        // win!
        if(match == 6){
                system("/bin/cat flag");
        }
        else{
                printf("bad luck...\n");
        }

}

void help(){
        printf("- nLotto Rule -\n");
        printf("nlotto is consisted with 6 random natural numbers less than 46\n");
        printf("your goal is to match lotto numbers as many as you can\n");
        printf("if you win lottery for *1st place*, you will get reward\n");
        printf("for more details, follow the link below\n");
        printf("http://www.nlotto.co.kr/counsel.do?method=playerGuide#buying_guide01\n\n");
        printf("mathematical chance to win this game is known to be 1/8145060.\n");
}

int main(int argc, char* argv[]){

        // menu
        unsigned int menu;

        while(1){
}
        else{
                printf("bad luck...\n");
        }

}

void help(){
        printf("- nLotto Rule -\n");
        printf("nlotto is consisted with 6 random natural numbers less than 46\n");
        printf("your goal is to match lotto numbers as many as you can\n");
        printf("if you win lottery for *1st place*, you will get reward\n");
        printf("for more details, follow the link below\n");
        printf("http://www.nlotto.co.kr/counsel.do?method=playerGuide#buying_guide01\n\n");
        printf("mathematical chance to win this game is known to be 1/8145060.\n");
}

int main(int argc, char* argv[]){

        // menu
        unsigned int menu;

        while(1){

                printf("- Select Menu -\n");
                printf("1. Play Lotto\n");
                printf("2. Help\n");
                printf("3. Exit\n");

                scanf("%d", &menu);

                switch(menu){
                        case 1:
                                play();
                                break;
                        case 2:
                                help();
                                break;
                        case 3:
                                printf("bye\n");
                                return 0;
                        default:
                                printf("invalid menu\n");
                                break;
                }
        }
        return 0;

}
```

Analisando o código conseguimos achar o problema na lógica de comparação do código. O problema está na forma como os números submetidos pelo usuário (submit) são comparados com os números gerados aleatoriamente (lotto). Essa lógica é falha e pode ser explorada de maneiras não intencionais.

```
int match = 0, j = 0;
for(i=0; i<6; i++){
    for(j=0; j<6; j++){
        if(lotto[i] == submit[j]){
            match++;
        }
    }
}
```

Aqui, o programa compara cada número gerado (lotto[i]) com todos os números submetidos pelo usuário (submit[j]). Isso significa que, para cada número gerado, ele verifica se ele existe em qualquer posição do array submit.
Problemas com a Lógica de Comparação

__1. Contagem de Correspondências Inflada:__

A lógica não verifica se os números correspondentes estão nas mesmas posições ou se são únicos.

Por exemplo, se o número 1 aparecer várias vezes no array submit, ele será contado várias vezes como uma correspondência, mesmo que o número 1 apareça apenas uma vez no array lotto.

Isso significa que a variável match pode ser incrementada além do número real de correspondências.

__2. Falta de Verificação de Unicidade:__

O código não garante que os números submetidos pelo usuário sejam únicos. Se o usuário enviar o mesmo número várias vezes (por exemplo, 111111), ele pode inflar artificialmente a contagem de correspondências.

Isso permite que o usuário "ganhe" o jogo sem realmente acertar todos os números gerados.

__3. Lógica de Vitória:__

Para ganhar, o programa verifica se match == 6. No entanto, como a contagem de correspondências pode ser inflada, o usuário pode enviar uma entrada como 111111 e, se o número 1 estiver presente no array lotto, ele terá 6 correspondências (mesmo que o número 1 apareça apenas uma vez no array lotto).

## A solução

Porém quando tentamos realizar tal metódo inserindo varias vezes `111111` não conseguimos a flag, isso é graças a função a `submit`. Essa função não lê valores como 1,2,3,etc... com os seus valores decimais e sim em ASCII. O valor de 1 em ASCII é 49, isso fere uma das regras do jogo `lotto is consisted with 6 random natural numbers less than 46`, ou seja, precisamos achar caracteres em ASCII que possuem valores abaixo de 46 para explorarmos a vulnerabilidade de maneira correta.

Aqui estão alguns dos caracteres que funcionariam:

`!` (33)

`"` (34)

`#` (35)

`$` (36)

`%` (37)

`&` (38)

`'` (39)

`(`(40)

`)` (41)

`*` (42)

`+` (43)

`,` (44)

`-` (45)

Pessoalmente eu usei `!` para esse desafio, repetindo algumas vezes o processo de inserir `!!!!!!` conseguimos a flag:

>sorry mom... I FORGOT to check duplicate numbers... :(
