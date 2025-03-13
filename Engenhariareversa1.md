# DuckWare Team - Engenharia Reversa  
###### Solved by @JoaoPedroPiceli and @xpics :)  

> Este CTF aborda a exploração de vulnerabilidades analisando códigos através do Ghidra.  

---

# 🔍 Análise de Engenharia Reversa  

Este CTF aborda a análise de um **Crackme** utilizando engenharia reversa para identificar a lógica do programa e descobrir a senha correta.  

## 📂 Ponto de Entrada do Programa (`entry`)  

O código de entrada do programa:  

```c
undefined8 entry(void) {
  undefined4 local_10;
  undefined4 local_c;

  local_c = 0;
  _printf("Crackme Level 0x02 (created by Nox)\n");
  _printf("\nEnter the passphrase: ");
  _scanf("%99d", &local_10);
  _check(local_10, 0x388e);
  return 1;
}
```  

### 🧐 Análise:  
- O programa exibe "Crackme Level 0x02 (created by Nox)".  
- Solicita ao usuário uma senha numérica (`%99d`).  
- A senha é armazenada em `local_10` e enviada para a função `_check`.  
- O valor `0x388e` (14478 em decimal) é comparado na função `_check`.  

### ✅ Conclusão:  
A senha correta é **14478**.  

## 🔐 Verificação da Senha (`_check`)  

```c
void _check(int param_1, int param_2) {
  if (param_1 == param_2) {
    _shift("Znk&vgyy}uxj&oy&iuxxkiz''\x06\x06");
  } else {
    _shift("Ot|groj&vgyy}uxj4\x06");
  }
  return;
}
```  

### 🧐 Análise:  
- A senha inserida (`param_1`) é comparada com `param_2` (14478).  
- Se correta, passa a string `"Znk&vgyy}uxj&oy&iuxxkiz''\x06\x06"` para `_shift`.  
- Se errada, passa `"Ot|groj&vgyy}uxj4\x06"` para `_shift`.  

## 🔓 Descriptografia da Mensagem (`_shift`)  

```c
void _shift(char *param_1) {
  size_t sVar1;
  int local_84;
  char local_78 [104];
  long local_10;

  local_10 = *(long *)PTR____stack_chk_guard_100004000;
  local_84 = 0;
  sVar1 = _strlen(param_1);
  for (; local_84 < (int)sVar1; local_84 = local_84 + 1) {
    local_78[local_84] = param_1[local_84] - 6;
  }
  local_78[local_84] = '\0';
  _printf("\n%s", local_78);
  if (*(long *)PTR____stack_chk_guard_100004000 == local_10) {
    return;
  }
  ___stack_chk_fail();
}
```  

### 🧐 Análise:  
- A função `_shift` desloca cada caractere da string **-6 posições na tabela ASCII**.  
- O resultado é armazenado em `local_78` e exibido.  

## 🏆 Obtendo a Mensagem Secreta  

A função `_shift` aplica uma **cifra de César invertida** (-6). Para descriptografar a mensagem, basta somar **6** a cada caractere da string recebida. O seguinte script Python faz isso automaticamente:  

```python
ciphertext = "Znk&vgyy}uxj&oy&iuxxkiz''\x06\x06"
decrypted = "".join(chr(ord(c) - 6) for c in ciphertext)
print(decrypted)
```  

### 🖼️ Saída esperada:  

Após inserir a senha correta e rodar o script, obtemos:  

[![87069262-d5e4-4b72-926d-18b7701cf4cc.jpg](https://i.postimg.cc/mZP8hQ9p/87069262-d5e4-4b72-926d-18b7701cf4cc.jpg)](https://postimg.cc/mhGQ5187)  

🖥️ **Nota:** Este executável foi compilado para **MacOS**.  

[![eng-reverse1.png](https://i.postimg.cc/HsQ073kz/eng-reverse1.png)](https://postimg.cc/D8w49rs4)  

[![reverse2.png](https://i.postimg.cc/L82tkfqp/reverse2.png)](https://postimg.cc/PPVvTCz3)  

### 🔑 Flag Final:  

```bash
The password is correct
```  

🎉 Isso conclui a análise do desafio! 🚀  
