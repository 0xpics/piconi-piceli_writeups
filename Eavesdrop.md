# DuckWare Team - Eavesdrop (picoCTF 2021)
###### Solved by @xpics and @JoaoPedroPiceli  
>This CTF é sobre Análise de Tráfego de Rede e Sniffing de Pacotes

# Desafio: Interceptando Comunicação em Rede

## Introdução  
Neste desafio, recebemos um arquivo de captura de tráfego de rede (**.pcap**) e precisamos analisar os pacotes para encontrar a flag escondida.

- **Arquivo do Desafio:** [capture.pcap](https://play.picoctf.org/practice/challenge/264)

## Solução  
Primeiro, abri o arquivo disponibilizado pelo site utilizando a ferramenta **Wireshark** e encontrei a seguinte tela:

[![inicio.png](https://i.postimg.cc/yYJmgbHQ/inicio.png)](https://postimg.cc/JGLDSKtZ)

Conforme estudado em redes, pude identificar que o protocolo TCP é utilizado para comunicações. Ao analisar os pacotes do protocolo TCP, encontrei a seguinte mensagem:

[![tela2.png](https://i.postimg.cc/w3K7sFXZ/tela2.png)](https://postimg.cc/vxzQJrSt)

Observando a comunicação, percebi que se tratava de uma troca entre duas pessoas. Durante esse diálogo, um dos interlocutores menciona a existência de um arquivo na porta 9002 e fornece a seguinte instrução para acessá-lo:

**sigh openssl des3 -d -salt -in file.des3 -out file.txt -k supersecretpassword123**

Ao acessar a porta 9002, encontrei:

[![Capturar2.png](https://i.postimg.cc/4d9K4rtV/Capturar2.png)](https://postimg.cc/8JTktXZz)

Após baixar o arquivo e executar o comando mencionado, obtive o seguinte resultado:

[![Capturar3.png](https://i.postimg.cc/SKXRVwg3/Capturar3.png)](https://postimg.cc/PLjd5FsQ)

## Flag  
picoCTF{nc_73115_411_5786acc3}
