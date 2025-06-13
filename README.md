# RecuperaSenha.PHP
Criação do recuperasenha.php

<?php
// Inclui arquivos de segurança e conexão com o banco de dados //
include('segurancazero.php');
include('conn.php');

// Verifica se o formulário foi enviado via POST //
if($_SERVER['REQUEST_METHOD'] == "POST"){

    $email = $_POST['email']; // Captura o e-mail informado pelo usuário //

    // Verifica se o e-mail existe na base de dados //
    $email = mysqli_real_escape_string($link, $email); // Protege contra SQL Injection //
    $sql = "SELECT COUNT(*) FROM tb_usuarios WHERE email_usuario = '$email'";
    $result = mysqli_query($link, $sql);
    $contagem = mysqli_fetch_array($result);

    if($contagem[0] == 0){
        // E-mail não encontrado: redireciona com mensagem de erro //
        header('location: recuperasenha.php?msg=Email invalido!');
        exit();
    }

    // E-mail válido: busca ID e telefone do usuário //
    $sql = "SELECT id_usuario, telefone_usuario FROM tb_usuarios WHERE email_usuario = '$email'";
    $result = mysqli_query($link, $sql);
    $tbl = mysqli_fetch_array($result);
    $id = $tbl[0];
    $tel = $tbl[1];

    // Gera código de recuperação aleatório de 6 dígitos //
    session_start(); // Inicia a sessão para armazenar o ID do usuário //
    $code = rand(100000, 999999);

    // Armazena o código no banco de dados associado ao usuário //
    $code = mysqli_real_escape_string($link, $code); // Protege contra SQL Injection //
    $sql = "UPDATE tb_usuarios SET recupera_usuario = '$code' WHERE id_usuario = $id";
    mysqli_query($link, $sql);
    mysqli_close($link); // Fecha a conexão com o banco //
    // Inclui a chave da API do CallMeBot (deve ser definida em outro lugar) //

    // Prepara dados para envio via WhatsApp usando CallMeBot //
    $phone = '+55' . $tel; // Formata o número com DDI do Brasil //
    $apikey = $chaveCB; // Chave da API (deve estar definida em outro lugar) //
    $message = "Seu codigo de recuperação de senha é: *$code*";

    // Monta a URL da API com os parâmetros //
    $url = 'https://api.callmebot.com/whatsapp.php?source=php&phone=' . $phone . '&text=' . urlencode($message) . '&apikey=' . $apikey;

    // Envia a requisição para a API //
    $html = file_get_contents($url);

    // Armazena o ID do usuário na sessão para a próxima etapa da recuperação //
    session_start(); // Garante que a sessão esteja iniciada //
    $_SESSION['id_recupera'] = $id;

    // Redireciona para a próxima etapa da recuperação //
    header('Location: recuperasenha2.php');
}
?>

<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Recuperar Senha</title>
    <link rel="stylesheet" href="lista.css">
</head>
<body>
    <h1>Recuperar Senha</h1>
    <br><br>
    <!-- Exibe mensagens de erro ou sucesso, se houver -->
    <?php
    include('msg_user.php');
    ?>
    <!-- Formulário para o usuário informar o e-mail -->
    <form action="recuperasenha.php" method="post">
        <label for="email">Email:</label>
        <input type="email" name="email" id="email" required>
        <br><br>
        <input type="submit" value="Enviar Codigo">
    </form>
</body>
</html>
