# Duckware Team - m00nwalk2(picoCTF 2019)

###### Solved by @0xpics and @JoaoPedroPiceli

> This CTF is about Forensics, steganography, signal transmission 

## Sobre o Desafio

Neste desafio o usuário deve conseguir achar mensagens escondidas através de interpretando arquivos de aúdio com informações desconhecidas, além de também conseguir interpretar as informações recebidas dessas fontes.

## Analisando o Desafio

Para esse desafio, nós recebemos três arquivos de aúdio diferentes: message.wav, clue1.wav, clue2.wav e clue3.wav.

Em cada um desses arquivos um som desconhecido toca,mas não sabemos ainda o que é. Antes de fazermos uma análise precisamos descobrir com o que estamos lidando. Para isso podemos usar o título do desafio como dica "m00nwalk".

Assumindo que o autor não esteja falando do Michael Jackson, podemos ligar esse título ao pouso lunar. Pesquisando um pouco podemos descobrir que as as imagens transmitidas da Lua pelo Apolo 11 foram feitas através de um metódo chamado SSTV, procurando um pouco sobre esse metódo descobrimos essa definição:

*O SSTV, ou Slow Scan Television, é uma técnica única que permite a transmissão de imagens estáticas por rádio. Ao contrário da televisão tradicional que transmite múltiplos quadros por segundo, o SSTV envia uma única imagem em um período de tempo, convertendo essa imagem em tons de áudio para a transmissão. Esta técnica é amplamente utilizada em comunicações de rádio de longa distância, especialmente onde a largura de banda é limitada.*

Perfeito, a descrição bate com que estamos procurando, agora só precisamos de uma maneira de transmitir a imagem.

Para isso usaremos dois programas: QSSTV e o Pavucontrol
 
QSSTV é um programa para receber e transmitir SSTV e HAMDRM (às vezes chamado de DSSTV). Já Pavucontrol é um sistema de aúdio simples e de fácil. Usaremos eles em conjunto, o Pavucontrol para tocar o aúdio e o QSSTV para a transmissão da imagem. Mas para isso precisamos configurá-los antes.

## Configuração

Primeiramente precisamos "conectar" ambos os softwares pare que o aúdio que esteja tocando no Pavucontrol seja transmitido para QSSTV. Para isso usaremos o seguinte comando no Linux:

**Antes de executar esse comando, certifique que ambos os programas estejam abertos**

`pactl load-module module-null-sink sink_name=virtual-cable`

Essa comando é usado para criar um dispositivo de áudio virtual no Pavucontrol, permitindo redirecionar o som entre aplicativos sem precisar de hardware físico.

`pactl` → Ferramenta de linha de comando para interagir com o servidor de som PulseAudio.

`load-module` → Carrega um módulo no PulseAudio.

`module-null-sink` → Cria um dispositivo de saída de áudio virtual (um "buraco negro" de áudio).

`sink_name=virtual-cable` → Define o nome desse dispositivo virtual como "virtual-cable", facilitando a identificação.

Com esse comando funcionando, vamos agora configurar o Pavucontrol. Vá até a sessão Recording e e troque a opção para Monitor of virtual-cable Audio/Sink

[![Captura-de-tela-2025-02-21-031841.png](https://i.postimg.cc/sDWZD6dj/Captura-de-tela-2025-02-21-031841.png)](https://postimg.cc/GHcpXjmV)

Com isso feito vamos para o QSSTV. Vá Options > Configuration > Sound, lá troque a opção de Input e Output Audio Device para pulse -- PulseAudio Sound Server

[![Captura-de-tela-2025-02-21-032358.png](https://i.postimg.cc/mDwsTFBN/Captura-de-tela-2025-02-21-032358.png)](https://postimg.cc/62yS0q7y)

Com isso feito, voltamos para a tela `receive` e marcamos a opção Auto Slant e definimos o Mode para Auto, isso irá garantir que o tipo de transmissão feita será automaticamente cofigurada para a correta para nós.

[![Captura-de-tela-2025-02-21-032853.png](https://i.postimg.cc/JzPY7ckh/Captura-de-tela-2025-02-21-032853.png)](https://postimg.cc/gxLHNh49)

## Solucionando o Desafio

Com tudo pronto, iremos executar o seguinte comando no terminal `paplay -d virtual-cable message.wav`

Esse comando é usado para reproduzir um arquivo de áudio (message.wav) através de um dispositivo de saída específico no Pavucontrol.

`paplay` → Comando para reproduzir arquivos de áudio no PulseAudio.

`-d virtual-cable` → Especifica o dispositivo de saída onde o áudio será reproduzido.

`message.wav` → O arquivo de áudio que será tocado.

Se tudo o ocorrer corretamente, o QSSTV deverá nos dar a seguinte imagem:

[![Whats-App-Image-2025-02-20-at-02-51-30-1-2.jpg](https://i.postimg.cc/zv23XQ8j/Whats-App-Image-2025-02-20-at-02-51-30-1-2.jpg)](https://postimg.cc/q6np1jth)

Essa imagem nos dá a seguinte flag:

>picoCTF{beep_boop_im_in_space}

Essa flag porém não é validada no site do picoCTF, isso significa que precisamos investigar um pouco mais.

Vamos repetir o mesmo processo que usamos com o arquivo message.wav, com os outros arquivos fornecidos e ver o que conseguimos.

**clue1.wav:**

[![Captura-de-tela-2025-02-21-040626.jpg](https://i.postimg.cc/J4t5cVDR/Captura-de-tela-2025-02-21-040626.jpg)](https://postimg.cc/ppbjRSJS)

**clue2.wav:**

[![Captura-de-tela-2025-02-21-041148.jpg](https://i.postimg.cc/Rh26LYy0/Captura-de-tela-2025-02-21-041148.jpg)](https://postimg.cc/p5YL2s2N)

**clue3.wav:**

[![Whats-App-Image-2025-02-20-at-03-09-54.jpg](https://i.postimg.cc/CK0MY0b0/Whats-App-Image-2025-02-20-at-03-09-54.jpg)](https://postimg.cc/21HNFPH9)

Todas as imagens obtidas nos dão dica de como prosseguir no desafio, porém as mais importante são a primeira e a terceira.

A primeira imagem no mostra a seguinte mensagem: `Password:hidden_stegossaurus`. Isso é uma clara a esteganografia, ou em inglês, steganography. A esteganografia é a prática de representar informações dentro de outra mensagem ou objeto físico, de tal maneira que a presença da informação oculta não seria evidente para o exame de uma pessoa desavisada. Arquivos escondidos através desse metódo sempre são acompanhados de uma senha que serve como mais uma camada de proteção ao arquivo, essa senha pode ser vista na imagem.

Agora analisando a terceira imagem nós temos a mensagem `Alan Eliasen the FutureBoy`, procurando sobre isso em uma search engine encontramos o seguinte site:

[![Captura-de-tela-2025-02-21-035304.jpg](https://i.postimg.cc/Bvy0bGKt/Captura-de-tela-2025-02-21-035304.jpg)](https://postimg.cc/DWqDpH5K)

Neste site podemos encontrar uma série de ferramentas, entre elas, uma de esteganografia.

Indo até essa ferramente, encontramos a opção de descodificar uma imagem. Ao escolhermos essa opção, podemos escolher um arquivo .wav para ser descodificado, além de também podermos inserir uma senha. Selecionando o arquivo message.wav para ser descodificado e digitando a senha que descobrimos, conseguimos a flag correta do desafio:

>picoCTF{the_answer_lies_hidden_in_plain_sight}
