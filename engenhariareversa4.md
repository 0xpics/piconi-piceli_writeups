# Duckware Team - Engenharia Reversa (4)

###### Solved by @0xpics and @JoaoPedroPiceli

> This CTF is about Reverse Engineering, code analysis

## Sobre o desafio

Neste desafio, o usuário deve utilizar ferramentas de análise de código para examinar a programação, compreender sua lógica e interpretar corretamente suas funcionalidades, a fim de alcançar o objetivo proposto.

## O Desafio

Nos é dado um executável para baixar, ao executarmos o programa nos pede uma senha, colocando um input aleátorio recebemos a mensagem "Bad Boy".

Sem muito mais o que explorar, vamos para a análise do código.

## Análise do código em Ghidra

Abrindo o código em Ghidra, vamos em busca da main, aquela que carrega o código principal. Como a main não está clara vamos exlplorar a função `entry` que é a mais próxima que conseguimos. Nela há esse código:

```
void processEntry entry(undefined8 param_1,undefined8 param_2)

{
  undefined auStack_8 [8];
  
  __libc_start_main(FUN_001010c0,param_2,&stack0x00000008,FUN_00101260,FUN_001012d0,param_1,
                    auStack_8);
  do {
                    /* WARNING: Do nothing block with infinite loop */
  } while( true );
}
```

Não há nada muito útil aqui para nós além do destaque para a função `FUN_001010c0`, vamos até ela então.

Nessa função se encontra o seguinte código:

```
undefined8 FUN_001010c0(void)

{
  long in_FS_OFFSET;
  int local_118;
  short local_114;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  __printf_chk(1,"Enter the password: ");
  __isoc99_scanf("%255s",&local_118);
  if ((local_118 == 0x30783468) && (local_114 == 0x72)) {
    puts("Good boy!");
  }
  else {
    puts("Bad boy!");
  }
  if (local_10 == *(long *)(in_FS_OFFSET + 0x28)) {
    return 0;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```

Ótimo, achamos a parte do código que se encontra a programção da senha, então a senha correta também deve estar aqui.

Analisando um pouco código podemos ver que, através da concatenação dos valores `local_118 == 0x30783468` e `local_114 == 0x72` conseguimos uma resposta positiva "Good boy!", este deve ser o output para a senha correta.

Então para solucionarmos o exercício precisamos descriptografar a senha correta dos valores encontrados.

## Solucionado o desafio

Os valores encontrados estão claramente em formato hexadecimal, quando fazemos a conversão do primeiro valor para ASCII conseguimos `0x4h`, já do segundo obtemos `r`, porém ao juntarmos o que encontramos e tentarmos coloca-los como senha, conseguimos a resposta negativa novamente, há algo mais na criptografia então.

Precisamos lembrar que quando valores hexadecimais são alocados na memória eles não são interpretados pela "maquina" com hex, e sim por little-endiadness, então precisamos fazer essa conversão primeiro e só ai converter para ASCII.

Primeiro valor:

`0x30783468` = `0x68 0x34 0x78 0x30`

`0x68 0x34 0x78 0x30` = `h4x0`

Segundo valor:

`0x72` = `0x72 0x00`
`0x72 0x00` = `r`

Juntando os dois conseguimos `h4x0r`. Ao testarmos a senha conseguimos o seguinte resultado:

[![Whats-App-Image-2025-03-10-at-02-39-05.jpg](https://i.postimg.cc/kXNbDPkD/Whats-App-Image-2025-03-10-at-02-39-05.jpg)](https://postimg.cc/KRzzswcy)
