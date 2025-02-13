# Duckware Team - Java Script Kiddie(picoCTF 2019)

###### Solved by @xpics and @JoaoPedroPiceli

>This CTF is about Java Script, PNG File structure, Web Exploitation

## Sobre o desafio
O objetivo do desafio é fazer usuário compreenda a estrutura de um código Java Script conseguindo encontrar falhas e padrões no código afim de conseguir resolver o desafio. Além disso o desafio também propõe a compreensão da estrutura de arquivos PNG analisando números em bytes e em formato hexadecimal.

## O desafio

Ao sermos redirecionados para a página do desafio nos deparamos com a seguinte tela:

[![Captura-de-tela-2025-02-12-183714.png](https://i.postimg.cc/MGFPX5Dc/Captura-de-tela-2025-02-12-183714.png)](https://postimg.cc/gw3qTv1d)

Uma tela em branco com apenas uma barra e um botão escrito `submit`. Quando colocamos um input aleátorio como `aa` e apertarmos em `submit` nada acontece, testando o mesmo com o `11` para descobrir se um input com número faz alguma diferença, descobrimos algo novo:

[![Captura-de-tela-2025-02-12-184207.png](https://i.postimg.cc/SRWDThp8/Captura-de-tela-2025-02-12-184207.png)](https://postimg.cc/BPnTbRbQ)

Um arquivo de imagem "quebrada" aparece, podemos assumir então que inputs que consistem em números farão algum tipo de imagem surgir. Abrindo a imagem em uma nova guia nos deparamos com o seguinte:

[![Captura-de-tela-2025-02-12-184558.png](https://i.postimg.cc/FK8phRr3/Captura-de-tela-2025-02-12-184558.png)](https://postimg.cc/3yjmB3Dx)

Na página não há muito que analisar, além da comprovação que de fato se trata de uma imagem corrompida. Testando outros inputs com outros tamanhos e números sempre recebemos o mesmo tipo de imagem quebrada. Vamos partir então para a análise do código fonte.

```
<html>
	<head>    
		<script src="jquery-3.3.1.min.js"></script>
		<script>
			var bytes = [];
			$.get("bytes", function(resp) {
				bytes = Array.from(resp.split(" "), x => Number(x));
			});

			function assemble_png(u_in){
				var LEN = 16;
				var key = "0000000000000000";
				var shifter;
				if(u_in.length == LEN){
					key = u_in;
				}
				var result = [];
				for(var i = 0; i < LEN; i++){
					shifter = key.charCodeAt(i) - 48;
					for(var j = 0; j < (bytes.length / LEN); j ++){
						result[(j * LEN) + i] = bytes[(((j + shifter) * LEN) % bytes.length) + i]
					}
				}
				while(result[result.length-1] == 0){
					result = result.slice(0,result.length-1);
				}
				document.getElementById("Area").src = "data:image/png;base64," + btoa(String.fromCharCode.apply(null, new Uint8Array(result)));
				return false;
			}
		</script>
	</head>
	<body>

		<center>
			<form action="#" onsubmit="assemble_png(document.getElementById('user_in').value)">
				<input type="text" id="user_in">
				<input type="submit" value="Submit">
			</form>
			<img id="Area" src=""/>
		</center>

	</body>
</html>

```

O código nos mostra algumas coisas sobre o desafio: 

Esse código é uma página HTML que recebe um conjunto de bytes do servidor, processa-os com base em uma chave de entrada do usuário e exibe uma imagem PNG gerada dinamicamente. Vamos analisar os principais pontos de interesse:
1. Comportamento do Código

    Carregamento de Bytes do Servidor:
* A linha `$.get("bytes", function(resp) { ... });` faz uma requisição GET para `"bytes"`, esperando uma resposta com bytes separados por espaços.
        
* Esses bytes são armazenados no array `bytes` como números.

2. Montagem da Imagem PNG:
        
* Quando o usuário insere uma chave (string de 16 caracteres) e submete o formulário, a função `assemble_png()` é chamada.
        
* A string de entrada é usada para modificar a ordem dos bytes, aplicando um deslocamento (`shifter`).
        
* O array `result` é preenchido reorganizando os bytes originais de acordo com a entrada.
        
* Os bytes são então convertidos para Base64 e atribuídos ao `src` da imagem no elemento `<img id="Area">`.

3. Pontos Importantes

* Transformação dos Bytes:
        
* A reorganização dos bytes depende do deslocamento determinado pela chave de entrada do usuário.
        
* O deslocamento é baseado na conversão dos caracteres da string em números: `shifter = key.charCodeAt(i) - 48;`.
        
* Isso significa que a chave deve conter caracteres numéricos (`0-9`) para funcionar corretamente.

    * Possível Objetivo do Código:
        
* Parece que a página esconde uma imagem nos bytes recebidos do servidor, e o usuário precisa fornecer a chave correta para reordená-los corretamente e restaurar a imagem original.

    * Base64 Encoding:
* `btoa(String.fromCharCode.apply(null, new Uint8Array(result)))` transforma os bytes em uma string Base64, permitindo que o navegador os interprete como uma imagem PNG.

Com essa análise podemos tentar resolver o código de duas maneiras, arrumando o código para que ele mostre a imagem de maneira correta, ou descobrir os 16 caracteres necessários para que a imagem correta seja mostrada. Para esse exercício, solucionaremos descobrindo os números corretos.

## A solução

Para começarmos a tentar descobrir os números da imagem primeiramente iremos entender como uma imagem PNG funciona:

Todo PNG é formado por uma série de números em hex, divididos em 4 categorias:PNG signature, Image header, Image data, Image end. Esses números ficam distribuídos em diferentes posições. 

[![Captura-de-tela-2025-02-12-214051.png](https://i.postimg.cc/W35PFJW5/Captura-de-tela-2025-02-12-214051.png)](https://postimg.cc/WFqyxtPZ)

Essa imagem representa o PNG de um único pixel vermelhor, representando assim o mínimo de números em Hex que cada imagem deve possuir. No código nos é mostrado que a chave que gerará a imagem deve ter ter um tamanho de 16 caracteres(`var LEN = 16;`) e com cada um deles sendo um caractere númerico de 0 a 9.

Graças a imagem, podemos concluir que os 16 caracteres buscados serão os 16 primeiros números da imagem: `89 50 4E 47 0D 0A 1A 0A 00 00 00 0D 49 48 44 52`. Se os 16 caracteres buscados são esses, como iremos transforma-los em números de 0 a 9? 

Antes de descobrirmos isso, precisamos conseguir os nosso números em Hex antes. Para isso converteremos os bytes da imagem em hex, para encontrarmos esses bytes só precisamos trocar a url do sit, substituindo o diretório `/?#` pelo `/bytes`.

Fazendo isso encontraremos essa sequência:

`156 255 80 255 117 10 239 248 152 253 120 232 36 127 116 255 151 235 25 172 215 0 56 102 219 174 30 15 36 188 93 90 249 36 32 45 123 73 191 151 236 241 151 68 144 250 157 130 1 180 20 85 213 2 157 248 68 255 250 13 60 66 249 82 187 157 29 222 29 30 0 252 126 251 95 0 174 72 194 108 29 101 70 21 121 40 26 132 73 119 254 237 73 192 96 219 137 80 89 71 0 145 1 152 0 69 254 71 0 65 68 35 0 0 119 114 13 222 68 119 1 0 78 40 155 111 95 90 164 0 26 2 0 245 186 0 84 0 0 233 145 46 110 49 48 16 78 223 135 64 197 10 0 120 0 243 252 62 144 188 21 61 1 110 148 208 22 114 160 31 156 17 45 59 72 237 74 218 0 8 32 123 136 65 179 150 32 56 206 43 240 9 156 225 69 54 226 158 106 148 62 48 1 232 173 0 239 248 243 206 82 255 241 252 56 55 152 132 108 181 78 254 175 251 60 183 38 231 63 123 204 48 43 13 131 4 113 75 243 215 32 200 144 195 29 233 196 63 3 190 139 207 89 28 107 159 185 101 59 120 121 12 245 116 64 96 250 187 241 234 231 207 213 239 119 191 233 71 205 127 144 40 251 253 173 186 246 10 227 252 202 242 163 74 237 33 75 49 205 74 154 165 126 231 30 231 232 199 118 65 211 98 204 7 250 244 141 155 243 123 82 137 252 35 183 201 132 91 252 37 244 56 188 86 125 103 216 248 215 146 144 149 21 164 233 219 127 127 207 208 30 154 111 203 63 127 141 231 146 5 20 4 81 239 38 36 19 191 63 61 183 223 215 205 210 239 168 135 148 201 39 248 212 191 160 151 116 19 150 99 249 141 111 188 0 225 193 61 73 140 160 56 23 53 48 5 99 100 175 250 125 151 253 12 150 85 41 72 206 97 52 79 88 196 130 26 157 254 185 181 42 146 217 255 24 125 155 88 111 116 167 62 238 36 52 95 57 54 126 233 184 143 46 183 234 73 183 108 163 228 218 233 129 44 169 191 74 0 30 126 245 10 249 245 241 65 191 245 73 209 50 140 26 72 132 223 181 204 200 123 185 186 183 218 175 228 249 75 180 91 229 252 193 203 187 253 52 166 28 117 119 13 238 134 74 227 127 71 251 237 50 191 61 76 230 90 241 178 221 233 202 254 211 228 156 60 202 241 71 49 24 90 187 3 245 247 159 124 157 250 227 18 150 50 49 101 86 235 162 234 57 124 108 116 245 226 190 28 43 129 220 86 245 85 107 38 215 223 119 242 72 140 213 103 209 194 70 30 96 111 204 128 234 55 184 247 205 49 227 5 220 101 80 171 155 217 87 33 26 173 127 187 128 253 215 111 203 54 210 243 29 237 148 204 235 202 131 191 191 211 157 54 147 104 188 87 4 251 25 17 185 219 247 124 135 228 176 223 135 196 157 130 215 206 124 122 136 248 28 23 175 56 104 209 253 47 161 236 61 252 147 140 86 102 185 82 110 231 91 251 245 216 243 254 236 176 127 134 31 135 152 251 90 0 216 127 102 56 99 56 64 204 61 95`

Com esses números em mão faremos a conversão de bytes para hex:

[![Captura-de-tela-2025-02-12-221901.png](https://i.postimg.cc/t44vJMQG/Captura-de-tela-2025-02-12-221901.png)](https://postimg.cc/1fbrYM3v)

Com os números hex em nossa posse, nós iremos propriamente organiza-los em um software chamada `HxD`. Ao fazermos isso, nós iremos nos deperar com essa tela:

[![Captura-de-tela-2025-02-12-222359.png](https://i.postimg.cc/8zcwNfM1/Captura-de-tela-2025-02-12-222359.png)](https://postimg.cc/TK8ncwtS)

Nela podemos ver que os 16 caracteres não eram uma coincidência, eles representam o número de colunas dos números em hex, com isso podemos assumir que os caracteres de 0 a 9 representarão a posição de cada número nas linhas.

Para descobrirmos a chave, nós iremos procurar os 16 caracteres que destacamos anteriormente, e quando essa posição for achada, colocaremos o número subtraido por 1 na chave, pois no código é dito que `result = result.slice(0,result.length-1)`, ex. Se a posição do número for 7, seu número na chave será 6.

[![Captura-de-tela-2025-02-12-222359.png](https://i.postimg.cc/FzqzS6WC/Captura-de-tela-2025-02-12-222359.png)](https://postimg.cc/30X35tH2)

Fazendo esse processo nós teremos essa chave:

>6696705967835463

Ao inserirmos ela no site teremos a seguinte imagem:

[![Captura-de-tela-2025-02-12-225035.png](https://i.postimg.cc/52p0qjRF/Captura-de-tela-2025-02-12-225035.png)](https://postimg.cc/XrG4nN3V)

Escaneando o QR Code obtemos a flag:

>picoCTF{ce96332407ef1381c0bcc59e4154afc8}
