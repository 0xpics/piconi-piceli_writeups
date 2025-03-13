# DuckWare Team - Engenharia Reversa2
###### Solved by @JoaoPedroPiceli and @xpics:)  

> Este CTF aborda exploração de vulnerabilidades analisando codigos atrávez do ghidra.

# 🔍 Análise Detalhada dos Códigos - Engenharia Reversa  

> Este desafio aborda a análise de um sistema de geração e verificação de senhas.  

---

## 🔑 Função `generate_password`

```c
void generate_password(long param_1, char *param_2) {
  size_t sVar1;
  ulong local_18;
  
  sVar1 = strlen(param_2);
  for (local_18 = 0; local_18 < sVar1; local_18 = local_18 + 1) {
    *(char *)(local_18 + param_1) = param_2[local_18] + '\x03';
  }
  *(undefined *)(sVar1 + param_1) = 0;
  return;
}
```

### 🧐 Análise:
- A senha é gerada a partir de uma string de entrada (`param_2`).
- Cada caractere da string é deslocado **+3** na tabela ASCII.
- A senha transformada é armazenada em `param_1`.

#### 📌 Exemplo de Transformação:

| Entrada  | ASCII | Saída | ASCII |
|----------|-------|--------|-------|
| 's'      | 115   | 'v'    | 118   |
| 'e'      | 101   | 'h'    | 104   |
| 'x'      | 120   | '{'    | 123   |
| 'y'      | 121   | '|'    | 124   |
| '1'      | 49    | '4'    | 52    |
| '3'      | 51    | '6'    | 54    |
| '3'      | 51    | '6'    | 54    |
| '7'      | 55    | ':'    | 58    |

✅ **Senha gerada:** `vh{|466:`

---

## 🔐 Função `encrypt_decrypt`

```c
void encrypt_decrypt(long param_1, byte param_2) {
  int local_c;
  
  for (local_c = 0; *(char *)(param_1 + local_c) != '\0'; local_c = local_c + 1) {
    *(byte *)(param_1 + local_c) = *(byte *)(param_1 + local_c) ^ param_2;
  }
  return;
}
```

### 🧐 Análise:
- A função aplica um **XOR** entre cada byte da string e uma chave (`param_2`).
- A operação é **reversível**, ou seja, aplicar novamente recupera a string original.

#### 📌 Exemplo de XOR:

Chave usada: `0xAA` (binário: `10101010`)

| Entrada  | ASCII | XOR `0xAA` | Resultado |
|----------|-------|-----------|-----------|
| 'v'      | 118   | 0xAA      | 0x92      |
| 'h'      | 104   | 0xAA      | 0xE2      |
| '{'      | 123   | 0xAA      | 0x91      |
| '|'      | 124   | 0xAA      | 0x8E      |
| '4'      | 52    | 0xAA      | 0xE6      |
| '6'      | 54    | 0xAA      | 0xE4      |
| '6'      | 54    | 0xAA      | 0xE4      |
| ':'      | 58    | 0xAA      | 0xE0      |

✅ **Senha criptografada:** `\x92\xE2\x91\x8E\xE6\xE4\xE4\xE0`

---

## 🔄 Fluxo do Programa

1. **Geração da Senha:**
   - `generate_password("sexy1337")` gera `vh{|466:`.

2. **Criptografia:**
   - `encrypt_decrypt("vh{|466:", 0xAA)` resulta em `\x92\xE2\x91\x8E\xE6\xE4\xE4\xE0`.

3. **Entrada do Usuário:**
   - O usuário insere uma senha, armazenada em `local_88`.

4. **Comparacão:**
   - A senha do usuário é criptografada e comparada com `\x92\xE2\x91\x8E\xE6\xE4\xE4\xE0`.
   - Se forem iguais, o acesso é concedido.

---

## 🔓 Conclusão

- A senha correta a ser inserida pelo usuário é `vh{|466:`.
- Isso ocorre porque o programa não espera que a entrada esteja criptografada, apenas compara diretamente com a versão gerada.

[![imagem-2025-03-12-222053355.png](https://i.postimg.cc/FFj5753B/imagem-2025-03-12-222053355.png)](https://postimg.cc/dDtSxfW2)

🎯 **Exploração:** Se for possível modificar a chave `0xAA`, poderíamos alterar a senha esperada!

🚀 Isso conclui a análise da verificação de senha! 😃
