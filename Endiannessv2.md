# DuckWare Team - Endianness v2 (picoCTF 2021)
###### Solved by @xpics and @JoaoPedroPiceli  
>This CTF is about Binary Exploitation, Endianness  

# Desafio: Convertendo Endianness  

## Introdução  
Neste desafio, precisamos converter um valor hexadecimal entre **Big Endian** e **Little Endian** para obter a flag.  

- **Arquivo do Desafio:** [endianness](https://play.picoctf.org/practice/challenge/415)  

## O que é Endianness?  
Endianness define a ordem dos **bytes** em que os dados são armazenados na memória.  
- **Big Endian:** O byte mais significativo (MSB) vem primeiro.  
- **Little Endian:** O byte menos significativo (LSB) vem primeiro.  

Exemplo:  
- **Big Endian:** `0x12 34 56 78`  
- **Little Endian:** `0x78 56 34 12`  

## Estratégia de Solução  
Primeiramente, abrimos o arquivo com a ferramenta **ghex** (`$ ghex challengefile`) e analisamos seus bytes em hexadecimal. Logo percebemos que a estrutura dos primeiros bytes do arquivo parece incorreta para um formato de imagem válido.  

### Comparação com uma Imagem PNG Válida  
Para verificar se o arquivo deveria ser uma imagem, buscamos uma **imagem PNG válida** e a analisamos no mesmo editor hexadecimal.  

- **Arquivo do desafio:**  
[![imagem-do-desafio.png](https://i.postimg.cc/WbVr91LZ/imagem-do-desafio.png)](https://postimg.cc/z3dVv5fD)  

- **Imagem PNG válida:**  
[![imagem-comun.png](https://i.postimg.cc/ZY8vyzB5/imagem-comun.png)](https://postimg.cc/HjLkfK9q)  

### Identificação do Problema  
Comparando os dois arquivos, percebemos que os bytes da imagem do desafio estão **invertidos**, ou seja, a imagem está corrompida devido a uma conversão errada de **endianness**. Para corrigir isso, precisamos reverter a ordem dos bytes para restaurar o arquivo corretamente.  

### Corrigindo os Bytes  
Para reverter a ordem dos bytes, utilizamos a ferramenta **CyberChef**. O primeiro passo foi carregar o arquivo corrompido no CyberChef e aplicar a **função de swap endianne**.  

- **Aplicação da função no CyberChef:**  
[![inicio-da-solu-o.png](https://i.postimg.cc/1tZ594Kc/inicio-da-solu-o.png)](https://postimg.cc/SXDpdS3J)  

Utilizando a ferramenta:  

  
[![solu-o1.png](https://i.postimg.cc/HsLWWqbK/solu-o1.png)](https://postimg.cc/c6VGX59M)  

### Convertendo de Hexadecimal para RAW  
Depois de corrigir a ordem dos bytes, precisamos transformar os dados de **hexadecimal para formato bruto (raw)**. Isso é necessário porque:  
1. A imagem estava salva como uma sequência de caracteres hexadecimais.  
2. Para ser exibida corretamente, precisamos convertê-la de volta para um **arquivo binário real**, que pode ser aberto por um visualizador de imagens.  

Fizemos isso no **CyberChef**, utilizando a função **Swap endianness** para converter os dados de hexadecimal para binário (raw).  

- **Conversão para formato RAW:**  
[![Capturar.png](https://i.postimg.cc/Qt0LRMYM/Capturar.png)](https://postimg.cc/xJJFm92r)  

Após essa conversão, baixamos o arquivo e abrimos a imagem, onde encontramos a flag.  

- **Flag revelada na imagem:**  
[![flag.png](https://i.postimg.cc/DytVVH61/flag.png)](https://postimg.cc/VdBZn73N)  

### Flag:  
```picoCTF{cert!f1Ed_Nd!4n_s0rrY_3nDian_94cc03f3}```  
