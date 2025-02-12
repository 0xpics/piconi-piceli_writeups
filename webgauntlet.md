# DuckWare Team - Web Gauntlet (picoCTF 2020)
###### Solved by @xpics and @JoaoPedroPiceli
>This CTF is about SQL Injection, Web exploitation

## Sobre o desafio
O desafio Web Gauntlet tem como objetivo explorar vulnerabilidades em ambientes SQLite. Composto por cinco fases, o desafio exige que o usuário faça login na conta admin utilizando SQL Injection. No entanto, a cada fase, novos filtros são introduzidos para bloquear determinados tipos de injeções, aumentando gradualmente a complexidade.

## Pré-análise 
Antes do início "oficial" do primeiro round, na página do desafio do site do picoCTF, o usuário encontrará a descrição do desafio e dois links.

>Can you beat the filters? Log in as admin

O primeiro link levará o usuário para o site onde o CTF ocorrerá.

`http://jupiter.challenges.picoctf.org:29164/ `

Lá o usuário irá se deparar com uma tela de login.

![Descrição da imagem](https://i.postimg.cc/Gm34bkG8/Captura-de-tela-2025-02-11-150300.png)

Ao tentar logar como admin o usuário verá a seguinte imagem.

[![Captura-de-tela-2025-02-12-062345.png](https://i.postimg.cc/kgkPTCJJ/Captura-de-tela-2025-02-12-062345.png)](https://postimg.cc/PCz7qBv9)

Na tela temos a nossa primeira pista de como deveremos agir:

`SELECT * FROM users WHERE username='admin' AND password='aa' `

Isso é um metódo de consulta SQL que busca informações na tabela `users` onde o `username` seja `'admin'` e a `password` seja `'aa'` (ou as credenciais que o usuário colocou).

* `SELECT *`: Significa que todos os campos da tabela users serão retornados.

* `FROM users`: Indica que a busca será feita na tabela `users`.

* `WHERE username='admin'`: Filtra apenas registros onde o nome de usuário seja `admin`.

* `AND password='aa'`: Adiciona mais um filtro, exigindo que a senha seja aa.

Se houver um usuário com essas credenciais no banco de dados, essa consulta retornará os dados dele. Caso contrário, o resultado será vazio.

Assim podemos concluir que a injeção ocorrerá através do campo `Username`

Antes de prosseguirmos retornaremos a tela da página do desafio do site do picoCTF e iremos explorar o segundo link.

`http://jupiter.challenges.picoctf.org:29164/filter.php`

Ao clicarmos no link, somos redirecionados a um site com uma unica mensagem.

>Round1: or

Ao analisarmos o endereço de link podemos notar que o nome do diretório é `filter.php`, combinando essa informação com a mensagem do site podemos ter a interpretação inicial que `or` é um argumento para injeção de SQL que teremos que fazer e que ela está sendo filtrada de alguma maneira. 

Sem mais conteúdo na página, podemos voltar para a página de login e finalmente começar o desafio.

## Round 1

No momento nossos únicos recursos são o metódo de consulta e o argumento `or`, vamos testar o mesmo no campo de login e ver o que aparece.

[![Captura-de-tela-2025-02-12-070508.png](https://i.postimg.cc/kX2bSND0/Captura-de-tela-2025-02-12-070508.png)](https://postimg.cc/VdcvQCqg)

Nada aparece, porém esse "nada" é algo, pois diferente da vez que em que colocamos `admin` o método de consulta apareceu no canto da tela além da mensagem `Invalide username/password` na tela de login. Assim, podemos assumir que nossa interpretação inicial sobre a filtragem está correta e que deveremos conseguir uma injeção que não usa o argumento `or` em sua composição.

Iremos usar então a injeção `admin'--` mas antes vamos explicar o que ela faz:

Ao inserir `admin'--`, o sistema gera uma consulta assim:

`SELECT * FROM users WHERE username = 'admin'--' AND password = 'aa';`

O `--` indica um comentário em SQL, ignorando o restante da linha.

A consulta fica equivalente a:

`SELECT * FROM users WHERE username = 'admin';`

O sistema verifica apenas se o usuário "admin" existe, ignorando a parte da senha.

Ao colocarmos a o input na caixa do `username` recebemos a segunte mensagem:

[![Captura-de-tela-2025-02-12-072847.png](https://i.postimg.cc/LsxPfxBM/Captura-de-tela-2025-02-12-072847.png)](https://postimg.cc/3yDWQCcL)

Isso mostra que nossa injeção foi correta e podemos prosseguir para o próximo round.

## Round 2

No round 2 temos a mesma tela que o round anterior, vamos testar a nossa injeção e ver o que acontece.

[![Captura-de-tela-2025-02-12-073609.png](https://i.postimg.cc/63GrCDWG/Captura-de-tela-2025-02-12-073609.png)](https://postimg.cc/LY2gpQZH)

Temos a mesma resposta que obtemos quando `or` foi filtrado, porém se testarmos apenas `admin` ou `admin'` podemos ver que recebemos a mensagem de username inválido. Então podemos assumir `--` foi filtrado e que devemos achar outra injeção para "logarmos" na conta admin.

Testaremos então a injeção `admin';`, ela essenciamente faz o mesmo do que a última injeção. O `;` indica um comentário em SQL, ignorando o restante da linha, ignorando a senha e logando no usuário `admin`.

Ao logarmos com essa injeção nós conseguimos acesso ao `admin` e podemos prosseguir para o Round 3.

## Round 3

Novamente em uma tela igual, colocaremos a mesma injeção mais uma vez e ver que resposta obtivemos.

E de maneira surpreendente nós conseguimos logar com essa injeção. Podemos assumir que conseguimos achar um input que passa pelo filtro tanto do Round 2 quanto do Round 3

## Round 4

No Round 4 ao tentarmos logar com essa injeção nós somos filtrados novamente.

Porém ao checarmos separadamente cada parte do nosso input podemos ver o filtro não ocorre com `;`.

[![Captura-de-tela-2025-02-12-080146.png](https://i.postimg.cc/L5PhCpwj/Captura-de-tela-2025-02-12-080146.png)](https://postimg.cc/PvttJBtq)

Mas sim com `admin`.

[![Captura-de-tela-2025-02-12-080657.png](https://i.postimg.cc/SK5ywPWd/Captura-de-tela-2025-02-12-080657.png)](https://postimg.cc/bGxcn3VS)

Porém não podemos logar sem o usuário `admin`, então nesse Round precisaremos achar uma maneira de contornar esse filtro usando o usário `admin` mesmo assim.

Para isso podemos testar a injeção `ad'||'min';`.

Nos bancos de dados que suportam `||` como operador de concatenação, a expressão:

`'ad' || 'min'`

é interpretada como `admin`. Ou seja, o banco de dados junta as strings "ad" e "min", formando "admin".

Se o código SQL vulnerável for algo como:

`SELECT * FROM users WHERE username = 'ad'||'min' AND password = 'aa';`

Isso equivale a:

`SELECT * FROM users WHERE username = 'admin' AND password = 'aa';`

Se houver um usuário chamado "admin", a consulta será válida.

Ao testarmos essa injeção conseguimos a seguinte resposta:

[![Captura-de-tela-2025-02-12-081946.png](https://i.postimg.cc/qvsyB8m2/Captura-de-tela-2025-02-12-081946.png)](https://postimg.cc/4Y3mBhR3)

A injeção funcionou e prosseguimos para o quinto e último Round.

## Round 5

Como fazemos em todos os Rounds, testaremos a injeção anterior para descobrir com qual filtro estamos lidando.

[![Captura-de-tela-2025-02-12-083008.png](https://i.postimg.cc/TY4fySs6/Captura-de-tela-2025-02-12-083008.png)](https://postimg.cc/tYWLwryS)

E assim como aconteceu no Round 3, a nossa injeção passou pelo filtro de dois Rounds. Assim concluímos o desafio passamos para o Round 6, agora é só mudar o diretório do site com o `filter.php` mostrado no campo de login, lá acharemos a nossa Flag.

## Fim do desafio

Ao sermos redirecionados nós deparamos com um código.

```
 <?php
session_start();

if (!isset($_SESSION["round"])) {
    $_SESSION["round"] = 1;
}
$round = $_SESSION["round"];
$filter = array("");
$view = ($_SERVER["PHP_SELF"] == "/filter.php");

if ($round === 1) {
    $filter = array("or");
    if ($view) {
        echo "Round1: ".implode(" ", $filter)."<br/>";
    }
} else if ($round === 2) {
    $filter = array("or", "and", "like", "=", "--");
    if ($view) {
        echo "Round2: ".implode(" ", $filter)."<br/>";
    }
} else if ($round === 3) {
    $filter = array(" ", "or", "and", "=", "like", ">", "<", "--");
    // $filter = array("or", "and", "=", "like", "union", "select", "insert", "delete", "if", "else", "true", "false", "admin");
    if ($view) {
        echo "Round3: ".implode(" ", $filter)."<br/>";
    }
} else if ($round === 4) {
    $filter = array(" ", "or", "and", "=", "like", ">", "<", "--", "admin");
    // $filter = array(" ", "/**/", "--", "or", "and", "=", "like", "union", "select", "insert", "delete", "if", "else", "true", "false", "admin");
    if ($view) {
        echo "Round4: ".implode(" ", $filter)."<br/>";
    }
} else if ($round === 5) {
    $filter = array(" ", "or", "and", "=", "like", ">", "<", "--", "union", "admin");
    // $filter = array("0", "unhex", "char", "/*", "*/", "--", "or", "and", "=", "like", "union", "select", "insert", "delete", "if", "else", "true", "false", "admin");
    if ($view) {
        echo "Round5: ".implode(" ", $filter)."<br/>";
    }
} else if ($round >= 6) {
    if ($view) {
        highlight_file("filter.php");
    }
} else {
    $_SESSION["round"] = 1;
}

// picoCTF{y0u_m4d3_1t_a3ed4355668e74af0ecbb7496c8dd7c5}
?> 
```
Nele, encontramos dois elementos essenciais: primeiro, os filtros aplicados em cada round.

* `Round 1:"or"`
* `Round 2:"or", "and", "like", "=", "--"`
* `Round 3:" ", "or", "and", "=", "like", ">", "<", "--"`
* `Round 4:" ", "or", "and", "=", "like", ">", "<", "--", * "admin"`
* `Round 5:" ", "or", "and", "=", "like", ">", "<", "--", "union", "admin"`

E por último, no fim do código, a flag:

>// picoCTF{y0u_m4d3_1t_a3ed4355668e74af0ecbb7496c8dd7c5}
