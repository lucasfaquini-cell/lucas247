<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Final Fantasy - Batalha Avançada</title>
    <style>
        /* Estilo da Fonte de Pixel Art */
        @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');
        
        body {
            font-family: 'Press Start 2P', cursive;
            background-color: #000;
            color: #fff;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            overflow: hidden;
        }

        .game-screen {
            display: none;
            position: relative;
            width: 800px;
            height: 600px;
            background-image: url('https://i.imgur.com/K432JtD.png');
            background-size: cover;
            border: 8px solid #545c68;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.7);
        }

        .hero-sprite, .enemy-sprite {
            position: absolute;
            background-size: contain;
            background-repeat: no-repeat;
            background-position: center;
        }
        
        .hero-sprite {
            width: 120px;
            height: 120px;
            background-image: url('https://i.imgur.com/nJgqL4A.png');
            bottom: 250px;
            left: 100px;
        }

        .enemy-sprite {
            width: 150px;
            height: 150px;
            top: 150px;
            right: 150px;
        }

        .battle-ui {
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 200px;
            display: flex;
            color: #000;
        }

        .message-box {
            background-color: #a8b0b8;
            border: 5px solid #000;
            width: 60%;
            height: 100%;
            display: flex;
            align-items: center;
            padding: 20px;
            box-sizing: border-box;
            font-size: 14px;
        }

        .info-box {
            background-color: #a8b0b8;
            border: 5px solid #000;
            border-left: none;
            width: 40%;
            height: 100%;
            padding: 20px;
            box-sizing: border-box;
            text-align: left;
        }

        .info-box h3 {
            margin: 0;
            font-size: 16px;
        }

        .hp-mp-bar {
            width: 100%;
            height: 15px;
            background-color: #4a4a4a;
            border: 2px solid #000;
            margin-top: 5px;
            margin-bottom: 10px;
            position: relative;
        }

        .hp-bar {
            height: 100%;
            background-color: #ff0000;
            transition: width 0.5s;
        }

        .mp-bar {
            height: 100%;
            background-color: #0000ff;
            transition: width 0.5s;
        }

        .battle-options {
            position: absolute;
            bottom: 0;
            right: 0;
            width: 40%;
            height: 100%;
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 10px;
            padding: 20px;
            box-sizing: border-box;
            background-color: #a8b0b8;
            border: 5px solid #000;
            border-bottom: none;
        }

        .battle-options button, .character-menu button {
            background-color: #545c68;
            color: #fff;
            font-family: 'Press Start 2P', cursive;
            font-size: 14px;
            border: 2px solid #fff;
            padding: 10px;
            cursor: pointer;
            transition: background-color 0.3s;
        }

        .battle-options button:hover:not(:disabled), .character-menu button:hover {
            background-color: #3d454f;
        }
        
        .battle-options button:disabled {
            cursor: not-allowed;
            opacity: 0.5;
        }

        /* Menu de Seleção de Personagem */
        .character-menu {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            width: 800px;
            height: 600px;
            background-color: #333;
            border: 8px solid #545c68;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.7);
            text-align: center;
        }

        .character-menu h2 {
            margin-bottom: 20px;
        }

        .character-list {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 20px;
            margin-bottom: 30px;
        }

        .character-card {
            background-color: #444;
            border: 2px solid #fff;
            padding: 15px;
            cursor: pointer;
            transition: transform 0.2s;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .character-card:hover {
            transform: scale(1.05);
        }

        .character-card h4 {
            margin-top: 0;
            margin-bottom: 5px;
            color: #ffcc00;
        }

        .character-card p {
            margin: 0;
            font-size: 12px;
        }
        
        .character-card img {
            width: 80px;
            height: 80px;
            image-rendering: pixelated;
            margin-bottom: 10px;
        }
    </style>
</head>
<body>
    <div class="character-menu" id="character-menu">
        <h2>Escolha seu Herói</h2>
        <div class="character-list" id="character-list">
            </div>
    </div>
    
    <div class="game-screen" id="game-screen">
        <div class="hero-sprite"></div>
        <div class="enemy-sprite"></div>

        <div class="battle-ui">
            <div class="message-box">
                <p id="battle-log">O que você fará?</p>
            </div>
            <div class="info-box">
                <h3><span id="hero-name"></span></h3>
                <div class="hp-mp-bar">
                    <div class="hp-bar" id="hero-hp-bar"></div>
                </div>
                <h3><span id="enemy-name"></span></h3>
                <div class="hp-mp-bar">
                    <div class="hp-bar" id="enemy-hp-bar"></div>
                </div>
            </div>
        </div>

        <div class="battle-options" id="battle-options">
            <button id="attack-btn">ATACAR</button>
            <button id="magic-btn">MAGIA</button>
            <button id="item-btn">ITEM</button>
            <button id="run-btn">FUGIR</button>
        </div>
    </div>

    <script>
        // === DADOS DO JOGO ===
        const heroes = {
            warrior: {
                name: "Guerreiro",
                hp: 150,
                maxHp: 150,
                mp: 20,
                maxMp: 20,
                attackDamage: () => Math.floor(Math.random() * (25 - 20 + 1)) + 20,
                magicDamage: () => 0,
                sprite: 'https://i.imgur.com/nJgqL4A.png'
            },
            mage: {
                name: "Mago",
                hp: 90,
                maxHp: 90,
                mp: 100,
                maxMp: 100,
                attackDamage: () => Math.floor(Math.random() * (12 - 8 + 1)) + 8,
                magicDamage: () => Math.floor(Math.random() * (45 - 35 + 1)) + 35,
                sprite: 'https://i.imgur.com/L4R6mFv.png'
            },
            thief: {
                name: "Ladrão",
                hp: 110,
                maxHp: 110,
                mp: 30,
                maxMp: 30,
                attackDamage: () => Math.floor(Math.random() * (20 - 15 + 1)) + 15,
                magicDamage: () => 0,
                sprite: 'https://i.imgur.com/1E4Lz0A.png'
            }
        };

        const enemies = [
            {
                name: "Goblin",
                hp: 100,
                maxHp: 100,
                attackDamage: () => Math.floor(Math.random() * (15 - 10 + 1)) + 10,
                sprite: 'https://i.imgur.com/mYgKk5g.png'
            },
            {
                name: "Wolf",
                hp: 80,
                maxHp: 80,
                attackDamage: () => Math.floor(Math.random() * (20 - 15 + 1)) + 15,
                sprite: 'https://i.imgur.com/D4s2G5O.png'
            },
            {
                name: "Slime",
                hp: 150,
                maxHp: 150,
                attackDamage: () => Math.floor(Math.random() * (8 - 5 + 1)) + 5,
                sprite: 'https://i.imgur.com/x08kH2A.png'
            }
        ];

        let hero;
        let enemy;
        const mpCost = 10;

        // === ELEMENTOS DO DOM ===
        const characterMenuEl = document.getElementById("character-menu");
        const characterListEl = document.getElementById("character-list");
        const gameScreenEl = document.getElementById("game-screen");

        const heroSpriteEl = document.querySelector(".hero-sprite");
        const enemySpriteEl = document.querySelector(".enemy-sprite");
        const heroNameEl = document.getElementById("hero-name");
        const enemyNameEl = document.getElementById("enemy-name");
        const heroHpBarEl = document.getElementById("hero-hp-bar");
        const enemyHpBarEl = document.getElementById("enemy-hp-bar");
        const battleLogEl = document.getElementById("battle-log");
        const buttons = document.querySelectorAll("#battle-options button");
        const attackBtn = document.getElementById("attack-btn");
        const magicBtn = document.getElementById("magic-btn");
        const itemBtn = document.getElementById("item-btn");
        const runBtn = document.getElementById("run-btn");

        // === FUNÇÕES DE LÓGICA DO JOGO ===

        function updateUI() {
            const heroHpPercent = (hero.hp / hero.maxHp) * 100;
            const enemyHpPercent = (enemy.hp / enemy.maxHp) * 100;
            
            heroHpBarEl.style.width = `${heroHpPercent}%`;
            enemyHpBarEl.style.width = `${enemyHpPercent}%`;

            heroHpBarEl.style.backgroundColor = heroHpPercent < 20 ? 'red' : heroHpPercent < 50 ? 'orange' : 'green';
            enemyHpBarEl.style.backgroundColor = enemyHpPercent < 20 ? 'red' : enemyHpPercent < 50 ? 'orange' : 'green';
        }

        function logMessage(message) {
            battleLogEl.textContent = message;
        }

        function toggleButtons(disabled) {
            buttons.forEach(button => button.disabled = disabled);
        }

        function heroTurn(action) {
            toggleButtons(true);
            let damage = 0;
            let message = "";
            let enemyDies = false;

            if (action === "attack") {
                damage = hero.attackDamage();
                enemy.hp -= damage;
                message = `${hero.name} ataca o ${enemy.name}, causando ${damage} de dano!`;
            } else if (action === "magic") {
                if (hero.mp >= mpCost) {
                    damage = hero.magicDamage();
                    if (damage > 0) {
                        hero.mp -= mpCost;
                        enemy.hp -= damage;
                        message = `${hero.name} usa FOGO, causando ${damage} de dano!`;
                    } else {
                         // Se o personagem não tiver magia, exibe mensagem diferente
                         message = `${hero.name} não sabe usar magia!`;
                         toggleButtons(false);
                         return;
                    }
                } else {
                    message = `Não há MP suficiente!`;
                    toggleButtons(false);
                    return;
                }
            } else if (action === "item") {
                hero.hp = Math.min(hero.hp + 30, hero.maxHp);
                message = `${hero.name} usa uma Poção e recupera 30 HP!`;
            } else if (action === "run") {
                const success = Math.random() < 0.5;
                if (success) {
                    logMessage(`${hero.name} fugiu com sucesso!`);
                    setTimeout(() => alert("Você fugiu da batalha."), 1000);
                    return;
                } else {
                    logMessage(`Falha na fuga!`);
                    setTimeout(enemyTurn, 1500);
                    return;
                }
            }
            
            logMessage(message);
            updateUI();
            
            if (enemy.hp <= 0) {
                enemy.hp = 0;
                logMessage(`${enemy.name} foi derrotado! Você VENCEU!`);
                setTimeout(() => alert("Parabéns, você venceu a batalha!"), 1000);
                return;
            }

            setTimeout(enemyTurn, 1500);
        }

        function enemyTurn() {
            logMessage(`${enemy.name} está atacando...`);
            setTimeout(() => {
                const damage = enemy.attackDamage();
                hero.hp -= damage;
                logMessage(`${enemy.name} ataca, causando ${damage} de dano ao ${hero.name}!`);
                updateUI();
                
                if (hero.hp <= 0) {
                    hero.hp = 0;
                    logMessage(`${hero.name} foi derrotado... Fim de Jogo.`);
                    setTimeout(() => alert("Você foi derrotado."), 1000);
                    return;
                }

                setTimeout(() => {
                    logMessage("O que você fará?");
                    toggleButtons(false);
                }, 1500);
            }, 1000);
        }

        // === LÓGICA DE INICIALIZAÇÃO ===

        function createCharacterCards() {
            for (const key in heroes) {
                const heroData = heroes[key];
                const card = document.createElement("div");
                card.className = "character-card";
                card.dataset.hero = key;
                card.innerHTML = `
                    <h4>${heroData.name}</h4>
                    <img src="${heroData.sprite}" alt="${heroData.name} sprite">
                    <p>HP: ${heroData.maxHp}</p>
                    <p>MP: ${heroData.maxMp}</p>
                `;
                card.addEventListener("click", () => selectCharacter(key));
                characterListEl.appendChild(card);
            }
        }

        function selectCharacter(heroKey) {
            hero = { ...heroes[heroKey] }; // Cria uma cópia para não alterar o objeto original
            
            // Esconde o menu e mostra a tela de jogo
            characterMenuEl.style.display = "none";
            gameScreenEl.style.display = "block";

            // Sorteia um inimigo aleatório
            const randomEnemyIndex = Math.floor(Math.random() * enemies.length);
            enemy = { ...enemies[randomEnemyIndex] };

            // Configura os sprites e nomes
            heroSpriteEl.style.backgroundImage = `url(${hero.sprite})`;

