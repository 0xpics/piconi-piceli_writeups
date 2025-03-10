# Duckware Team - Engenharia Reversa (3)

###### Solved by @0xpics and @JoaoPedroPiceli

>This CTF is about Reverse Engineering, code analysis

## Sobre o desafio

Neste desafio o usuário tem que, através de ferramentas análise de código, conseguir interpretar corretamente a progamação para atingir o objetivo do desafio.

## O Desafio

Nos é dado um executável para baixar, este executável tem como extensão .exe, ou seja, só roda no Windows. Ao executarmos em um Desktop Windows o programa nos pede uma senha, colocando um input aleátorio recebemos uma mensagem negativa.

Sem muito mais o que explorar, vamos para a análise do código.

## Análise do código em Ghidra

Abrindo o código em Ghidra, vamos em busca da main, aquela que carrega o código principal, porém ela nem sempre está de fácil acesso então precisamo explorar um pouco.

Cavucando no Ghidra achamos uma função que se destaca das outras `FUN_14000b1f0`, de todas as funções no código, esta é a mais única, uma vez que só ela termina a sequênica 1400 com o caractere b. Analisando a função encontramos o seguinte código:

```
bool FUN_14000b1f0(void)

{
  int iVar1;
  undefined8 uVar2;
  char *_Str2;
  char local_27 [31];
  
  FUN_14000153e();
  uVar2 = FUN_14000148a("1: 1&t <1t$5\'\'#;&0ntT4q\'4Tqgf\'TTe 7",0x54);
  FUN_140001400(uVar2);
  FUN_140001447(&DAT_14000d000,local_27);
  _Str2 = (char *)FUN_14000148a("\"gpPGcG*+wBq}b]VP[Zdy$ej\\jyj\"g",0x13);
  iVar1 = strcmp(local_27,_Str2);
  if (iVar1 != 0) {
    FUN_140001400("Wrong password!\n");
  }
  else {
    FUN_140001400("Correct password!\n");
  }
  return iVar1 != 0;
}

```

Perfeito, apesar de ser um pouco de díficil de entender conseguimos perceber algo mais facilmente no código, as frases "Wrong password!" e "Correct password!". Perfeito, isso significa que a parte do programa que configura a senha está aqui, estamos no local correto.

No código também temos duas strings interessantes: `"1: 1&t <1t$5\'\'#;&0ntT4q\'4Tqgf\'TTe 7"` e `"\"gpPGcG*+wBq}b]VP[Zdy$ej\\jyj\"g"`, uma dessas duas strings deve ser a senha que procuramos, porém ao testarmos qualquer umas das duas no programa recebemos a mensagem de senha incorreta, isso significa que elas estão criptografadas.

## Solucionado o desafio

A criptografia que estamos lidando pode ser qualquer uma, porém ao lado das strings temos algo muito interessante, os valores `0x54` e `0x13`, estes valores podem ser associados a criptografia XOR, uma vez que eles irão agir como keys mudarão o padrão de criptografia, uma característica padrão do XOR.

Para testar essa teoria podemos utilizar o código abaixo:

```
def xor_decrypt(encoded, key):
    return ''.join(chr(ord(c) ^ key) for c in encoded)

encoded1 = "1: 1&t <1t$5\'\'#;&0ntT4q\'4Tqgf\'TTe 7"
encoded2 = "\"gpPGcG*+wBq}b]VP[Zdy$ej\\jyj\"g"

key1 = 0x54
key2 = 0x13

print(xor_decrypt(encoded1, key1))
print(xor_decrypt(encoded2, key2))
```

Executando o código conseguimos as seguintes senhas:

String 1: `%s` %32s  1tc

String 2: `1tcCTpT98dQbnqNECHIwj7vyOyjy1t`

Não é possível através do write up porém a string 1 veio cheio de caracteres "quebrados", então vamos testar a string 2.

Executando o código e colocando a senha quando pedido coneguimos a seguinte resposta:

[![Whats-App-Image-2025-03-10-at-02-04-39.jpg](https://i.postimg.cc/zGjDTtfz/Whats-App-Image-2025-03-10-at-02-04-39.jpg)](https://postimg.cc/K11y25Lw)
