# RecuperaSenha.PHP
Criação do recuperasenha.php

<?php
include('segurancazero.php');
include('conn.php');
if($_SERVER['REQUEST_METHOD']== "POST"){
    $email = $_POST['email'];
    $sql = "SELECT COUNT(*) FROM tb_usuarios WHERE email_usuario = '$email'";
    $result = mysqli_query($link,$sql);
    $contagem = mysqli_fetch_array($result);

    if($contagem[0] == 0){
        //email invalido!//
        header('location: recuperasenha.php?msg=Email invalido!');
        exit();
    }
    $sql = "SELECT id_usuario, telefone_usuario FROM tb_usuarios WHERE
    email_usuario = '$email'";
    $result = mysqli_query($link,$sql);
    $tbl = mysqli_fetch_array($result);
    $id = $tbl[0];
    $tel = $tbl[1];
    $code = rand(100000,999999);
    $sql = "UPDATE tb_usuarios SET recupera_usuario = '$code' WHERE
    id_usuario = $id";
    mysqli_query($link,$sql);
    mysqli_close($link);

    $phone='+55'  . $tel;
    $apikey= $chaveCB;
    $message="Seu codigo de recuperação de senha é: *$code*";

    $url='https://api.callmebot.com/whatsapp.php?source=php&phone='.$phone.'&text='.urlencode($message).'&apikey='.$apikey;
    $html=file_get_contents($url);
    $_SESSION['id_recupera'] = $id;
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
    <?php
    include('msg_user.php');
    ?>
    <form action="recuperasenha.php" method="post">
        <label for="email">Email:</label>
        <input type="email" name="email" id="email" required>
        <br><br>
        <input type="submit" value="Enviar Codigo">
    </form>
</body>
</html>
