# DuckWare Team - Web Gauntlet 2 (picoCTF 2021)
###### Solved by @xpics and @JoaoPedroPiceli
>This CTF is about SQL Injection, Web exploitation

# Desafio: Injeção SQL com Filtros Específicos

## Introdução
Você se depara com um site que aparenta ser familiar e que possui uma área de administração protegida. Os links importantes são:

- **Página de Login:** [http://mercury.picoctf.net:26215/](http://mercury.picoctf.net:26215/)
[![imagem-2025-02-13-125542694.png](https://i.postimg.cc/VkDcStz6/imagem-2025-02-13-125542694.png)](https://postimg.cc/FYd687d5)
- **Página com Filtro:** [http://mercury.picoctf.net:26215/filter.php](http://mercury.picoctf.net:26215/filter.php)
- **Filtro:** or and true false union like = > < ; -- /* */ admin

## Descrição do Desafio
O desafio explora uma vulnerabilidade de injeção SQL no formulário de login. Contudo, um filtro implementado na página `filter.php` bloqueia determinados termos e símbolos, exigindo uma abordagem diferenciada para a injeção.

## Estratégia de Solução

### Manipulação do Campo de Usuário
O login padrão requer o nome `"admin"`, mas esse valor é filtrado. Para contornar essa restrição, podemos quebrar a palavra e uní-la utilizando o operador `||`, resultando em:

- **Username:** `ad'||'min`

### Manipulação do Campo de Senha
Normalmente, usaríamos uma condição como `‘ OR ‘1’=’1` para forçar a entrada, mas a palavra **OR** também está filtrada. Assim, optamos por uma expressão que sempre avalia como verdadeira sem utilizar os termos bloqueados:

- **Password:** `1' IS NOT '2`

Utilizando esses valores, o sistema autentica o usuário e exibe a mensagem:  
*“Parabéns! Você ganhou! Confira filter.php”*

[![imagem-2025-02-13-125958242.png](https://i.postimg.cc/2S5KKKfk/imagem-2025-02-13-125958242.png)](https://postimg.cc/GHZKBz2V)

## Recuperando a Bandeira
Após o login bem-sucedido, recarregue a página `filter.php` para visualizar o código-fonte que contém a bandeira do desafio. Segue o trecho de código relevante:

```php
session_start();

if (!isset($_SESSION["winner2"])) {
    $_SESSION["winner2"] = 0;
}
$win = $_SESSION["winner2"];
$view = ($_SERVER["PHP_SELF"] == "/filter.php");

if ($win === 0) {
    $filter = array("or", "and", "true", "false", "union", "like", "=", ">", "<", ";", "--", "/*", "*/", "admin");
    if ($view) {
        echo "Filters: ".implode(" ", $filter)."<br/>";
    }
} else if ($win === 1) {
    if ($view) {
        highlight_file("filter.php");
    }
    $_SESSION["winner2"] = 0;        // <- Don't refresh!
} else {
    $_SESSION["winner2"] = 0;
}

// picoCTF{0n3_m0r3_t1m3_fc0f841ee8e0d3e1f479f1a01a617ebb}

```

### Flag:

```picoCTF{0n3_m0r3_t1m3_fc0f841ee8e0d3e1f479f1a01a617ebb} ```


