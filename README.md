# Trabalho.Forca
PHP
<?php
session_start();

// Palavras para o jogo
$palavras = ["php", "javascript", "forca", "programacao", "desenvolvimento", "computador", "algoritmo"];

// Inicializar jogo se não existir
if (!isset($_SESSION['palavra'])) {
    $_SESSION['palavra'] = $palavras[array_rand($palavras)];
    $_SESSION['letras_adivinhadas'] = [];
    $_SESSION['erros'] = 0;
    $_SESSION['max_erros'] = 6;
}

$palavra = $_SESSION['palavra'];
$letras_adivinhadas = $_SESSION['letras_adivinhadas'];
$erros = $_SESSION['erros'];
$max_erros = $_SESSION['max_erros'];

// Processar palpite
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['letra'])) {
    $letra = strtolower(trim($_POST['letra']));
    if (ctype_alpha($letra) && strlen($letra) === 1 && !in_array($letra, $letras_adivinhadas)) {
        $letras_adivinhadas[] = $letra;
        $_SESSION['letras_adivinhadas'] = $letras_adivinhadas;
        
        if (strpos($palavra, $letra) === false) {
            $_SESSION['erros']++;
            $erros++;
        }
    }
    // Reset game
    if (isset($_POST['reset'])) {
        session_destroy();
        header("Location: index.php");
        exit;
    }
}

// Verificar vitória ou derrota
$palavra_exibida = '';
$completa = true;
for ($i = 0; $i < strlen($palavra); $i++) {
    if (in_array($palavra[$i], $letras_adivinhadas)) {
        $palavra_exibida .= $palavra[$i] . ' ';
    } else {
        $palavra_exibida .= '_ ';
        $completa = false;
    }
}

$game_over = $erros >= $max_erros;
$vitoria = $completa && !$game_over;
?>

<!DOCTYPE html>
<html lang="pt">
<head>
    <meta charset="UTF-8">
    <title>Jogo da Forca - PHP</title>
    <style>
        body { font-family: Arial; text-align: center; background: #f4f4f4; }
        .forca { font-size: 2.5em; letter-spacing: 12px; margin: 20px 0; }
        input, button { padding: 10px; font-size: 1.1em; }
    </style>
</head>
<body>
    <h1>Jogo da Forca (PHP Server-side)</h1>
    <div class="forca"><?php echo $palavra_exibida; ?></div>
    <p>Erros: <?php echo $erros; ?>/<?php echo $max_erros; ?></p>
    
    <?php if ($game_over): ?>
        <h2>Game Over! A palavra era: <strong><?php echo $palavra; ?></strong></h2>
    <?php elseif ($vitoria): ?>
        <h2>🎉 Parabéns! Você venceu!</h2>
    <?php else: ?>
        <form method="post">
            <input type="text" name="letra" maxlength="1" required autofocus>
            <button type="submit">Adivinhar Letra</button>
        </form>
    <?php endif; ?>
    
    <form method="post">
        <button type="submit" name="reset" value="1">Novo Jogo</button>
    </form>
</body>
</html>

-----------------------------------
JavaScript

<!DOCTYPE html>
<html lang="pt">
<head>
    <meta charset="UTF-8">
    <title>Jogo da Forca - JavaScript</title>
    <style>
        body { 
            font-family: Arial; 
            text-align: center; 
            background: #f4f4f4; 
        }
        .forca { 
            font-size: 2.5em; 
            letter-spacing: 12px; 
            margin: 20px 0; 
        }
        #canvas { 
            border: 2px solid #333; 
            margin: 20px auto; 
            display: block; 
            background: #fff;
        }
        input, button { 
            padding: 12px; 
            font-size: 1.1em; 
            margin: 5px;
        }
    </style>
</head>
<body>
    <h1>Jogo da Forca (JavaScript Client-side)</h1>
    <canvas id="canvas" width="250" height="280"></canvas>
    <div id="palavra" class="forca"></div>
    <p id="erros">Erros: 0/6</p>
    
    <input type="text" id="letra" maxlength="1" autofocus>
    <button onclick="adivinhar()">Adivinhar</button>
    <button onclick="novoJogo()">Novo Jogo</button>

    <script>
        const palavras = ["php", "javascript", "forca", "programacao", "desenvolvimento", "computador", "algoritmo"];
        let palavra = '';
        let letrasAdivinhadas = [];
        let erros = 0;
        const maxErros = 6;
        let canvas, ctx;

        function iniciarJogo() {
            palavra = palavras[Math.floor(Math.random() * palavras.length)];
            letrasAdivinhadas = [];
            erros = 0;
            canvas = document.getElementById('canvas');
            ctx = canvas.getContext('2d');
            atualizarTela();
        }

        function desenharForca() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.strokeStyle = '#333';
            ctx.lineWidth = 8;

            // Base
            ctx.beginPath();
            ctx.moveTo(30, 250); ctx.lineTo(220, 250); ctx.stroke();
            // Poste
            ctx.moveTo(60, 250); ctx.lineTo(60, 40); ctx.stroke();
            ctx.moveTo(60, 40); ctx.lineTo(160, 40); ctx.stroke();
            // Corda
            ctx.moveTo(160, 40); ctx.lineTo(160, 80); ctx.stroke();

            // Corpo conforme erros
            if (erros >= 1) { // Cabeça
                ctx.beginPath(); ctx.arc(160, 105, 20, 0, Math.PI * 2); ctx.stroke();
            }
            if (erros >= 2) { // Corpo
                ctx.moveTo(160, 125); ctx.lineTo(160, 200); ctx.stroke();
            }
            if (erros >= 3) { // Braço esquerdo
                ctx.moveTo(160, 140); ctx.lineTo(120, 170); ctx.stroke();
            }
            if (erros >= 4) { // Braço direito
                ctx.moveTo(160, 140); ctx.lineTo(200, 170); ctx.stroke();
            }
            if (erros >= 5) { // Perna esquerda
                ctx.moveTo(160, 200); ctx.lineTo(130, 240); ctx.stroke();
            }
            if (erros >= 6) { // Perna direita
                ctx.moveTo(160, 200); ctx.lineTo(190, 240); ctx.stroke();
            }
        }

        function atualizarTela() {
            desenharForca();
            
            let exibida = '';
            let completa = true;
            for (let letra of palavra) {
                if (letrasAdivinhadas.includes(letra)) {
                    exibida += letra + ' ';
                } else {
                    exibida += '_ ';
                    completa = false;
                }
            }
            document.getElementById('palavra').textContent = exibida.trim();
            document.getElementById('erros').textContent = `Erros: ${erros}/${maxErros}`;

            if (erros >= maxErros) {
                setTimeout(() => alert('💀 Game Over! A palavra era: ' + palavra), 100);
            } else if (completa) {
                setTimeout(() => alert('🎉 Parabéns! Você venceu!'), 100);
            }
        }

        function adivinhar() {
            const input = document.getElementById('letra');
            let letra = input.value.toLowerCase().trim();
            input.value = '';

            if (letra.length === 1 && /^[a-z]$/.test(letra) && !letrasAdivinhadas.includes(letra)) {
                letrasAdivinhadas.push(letra);
                if (!palavra.includes(letra)) {
                    erros++;
                }
                atualizarTela();
            }
        }

        function novoJogo() {
            iniciarJogo();
        }

        // Iniciar o jogo
        window.onload = iniciarJogo;
    </script>
</body>
</html>
