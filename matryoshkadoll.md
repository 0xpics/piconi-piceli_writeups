# Duckware Team - Matryoshka doll(picoCTF 2021)

###### Solved by @0xpics and @JoaoPedroPiceli

> This CTF is about Forensics, steganography

## Sobre o desafio

O objetivo desse desafio é fazer com que usuário consiga achar arquivos escondidos dentro de outros arquivos, nesse caso, uma imagem.

## O Desafio

Na página do desafio nos é dada a seguinte descrição: `As bonecas Matryoshka são um conjunto de bonecas de madeira de tamanho decrescente colocadas umas dentro das outras. Qual é o última?`. Também nos é dado um arquivo para baixar, quando baixamos conseguimos a imagem abaixo:


[![dolls.png](https://i.postimg.cc/tTRtXpgh/dolls.png)](https://postimg.cc/NyVrDhmM)

Existem muitas maneiras de esconder arquivos em uma imagem, porém graças a definição do que é uma boneca Matryoshka, podemos interpretar que o arquivo que estamos procurando realmente está "dentro" da imagem.

Para checarmos essa suposição podemos usar a ferramenta binwalk.O binwalk é uma ferramenta de análise forense e engenharia reversa usada para inspecionar e extrair dados ocultos em arquivos binários. Ele é amplamente utilizado em análise de firmware, imagens de disco, arquivos comprimidos e esteganografia.

Assim, utilizando o seguinte comando `binwalk -e dolls.jpg`(o -e nos ajudará a extrair automaticamente arquivos embutidos dentro de um binário) conseguimos a seguinte resposta:

`Zip archive data, at least v2.0 to extract, compressed size: 378942, uncompressed size: 383937, name: base_images/2_c.jpg`

Sucesso! Achamos uma imagem escondida, para conseguirmos acessá-la, usaremos dois comandos:

Usaremos `dd` para isolar a parte ZIP da imagem e extraí-la:

`dd if=dolls.jpg bs=1 skip=272492 of=hidden.zip`

E então unzip para desecompactar o arquivo:

`unzip hidden.zip`

Com isso obteremos a seguinte resposta:

`Archive:  hidden.zip
  inflating: base_images/2_c.jpg  `
  
Agora poderemos acessar a imagem escondida. Indo até seu diretório, encontramos essa imagem:

[![Whats-App-Image-2025-02-20-at-23-35-56.jpg](https://i.postimg.cc/6qtjM2Ng/Whats-App-Image-2025-02-20-at-23-35-56.jpg)](https://postimg.cc/VrZW6NvW)

Mais uma boneca, significa que precisamos cavar mais fundo para achar a flag.

Para isso podemos usar o mesmo processo usado anteriormente.

Fazendo isso obteremos essa resposta:`187707        0x2DD3B         Zip archive data, at least v2.0 to extract, compressed size: 196042, uncompressed size: 201444, name: base_images/3_c.jpg`

Achamos mais uma imagem, fazendo o mesmo processo para acessa-la, conseguimos essa imagem:

[![Whats-App-Image-2025-02-20-at-23-37-17.jpg](https://i.postimg.cc/NFDNSssM/Whats-App-Image-2025-02-20-at-23-37-17.jpg)](https://postimg.cc/N2y7rwrq)

Outra boneca, repetindo o mesmo processo de descoberta e acesso nessa imagem conseguiremos essa nova imagem:

[![Whats-App-Image-2025-02-20-at-23-38-33.jpg](https://i.postimg.cc/fWfgJKcN/Whats-App-Image-2025-02-20-at-23-38-33.jpg)](https://postimg.cc/NLLxnRCJ)

Mais uma boneca, porém dessa vez ao repetirmos o processo anterior conseguimos uma nova resposta:

`79578         0x136DA         Zip archive data, at least v2.0 to extract, compressed size: 62, uncompressed size: 81, name: flag.txt`

Achamos o arquivo da flag, fazendo o mesmo processo que usamos com a imagem para acessar o arquivo, obtemos a flag:

>picoCTF{4cf7ac000c3fb0fa96fb92722ffb2a32}
