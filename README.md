"use strict";

const game = document.getElementById("game");
const playerElement = document.getElementById("player");
const npcElement = document.getElementById("npc");
const hintElement = document.getElementById("hint");

const scoreElement = document.getElementById("score");
const fragmentsElement = document.getElementById("fragments");

const dialogueElement = document.getElementById("dialogue");
const dialogueTextElement = document.getElementById("dialogueText");
const dialogueActionButton = document.getElementById("dialogueAction");

const challengeElement = document.getElementById("challenge");
const challengePromptElement = document.getElementById("challengePrompt");
const challengeOptionsElement = document.getElementById("challengeOptions");
const challengeStatusElement = document.getElementById("challengeStatus");

const player = {
  x: 65,
  y: 330,
  width: 44,
  height: 48,
  speed: 3.5
};

const npc = {
  x: 0,
  y: 0,
  width: 44,
  height: 48
};

const missions = [
  {
    title: "Missão 1: O registro perdido",
    prompt: "Encontre o registro de Dália na tabela. Qual é a sequência correta?",
    steps: [
      "Abrir a tabela de habitantes",
      "Identificar a coluna Nome",
      "Filtrar os registros pelo nome Dália",
      "Ler os dados do registro encontrado"
    ]
  },
  {
    title: "Missão 2: Organizando os dados",
    prompt: "A biblioteca precisa encontrar todos os livros de programação. O que fazer?",
    steps: [
      "Abrir a tabela de livros",
      "Identificar a coluna Categoria",
      "Filtrar pela categoria Programação",
      "Conferir os livros encontrados"
    ]
  },
  {
    title: "Missão 3: Alerta de segurança",
    prompt: "Um terminal recebeu uma mensagem suspeita. Qual é a atitude segura?",
    steps: [
      "Verificar o remetente da mensagem",
      "Não abrir links ou anexos suspeitos",
      "Avisar a guardiã digital",
      "Apagar a mensagem suspeita"
    ]
  }
];

const pressedKeys = new Set();

let score = 0;
let fragments = 0;
let expectedStepIndex = 0;
let currentMission = null;
let dialogMode = "intro";
let gamePaused = false;

function placeNpc() {
  const maxX = game.clientWidth - npc.width - 30;
  const maxY = game.clientHeight - npc.height - 30;

  npc.x = Math.min(600, Math.max(280, maxX));
  npc.y = Math.min(210, Math.max(150, maxY));

  npcElement.style.left = `${npc.x}px`;
  npcElement.style.top = `${npc.y}px`;
}

function renderPlayer() {
  const maxX = game.clientWidth - player.width;
  const maxY = game.clientHeight - player.height;

  player.x = Math.max(0, Math.min(player.x, maxX));
  player.y = Math.max(0, Math.min(player.y, maxY));

  playerElement.style.left = `${player.x}px`;
  playerElement.style.top = `${player.y}px`;
}

function isNearNpc() {
  const playerCenterX = player.x + player.width / 2;
  const playerCenterY = player.y + player.height / 2;
  const npcCenterX = npc.x + npc.width / 2;
  const npcCenterY = npc.y + npc.height / 2;

  return Math.hypot(playerCenterX - npcCenterX, playerCenterY - npcCenterY) < 85;
}

function updateHint() {
  if (gamePaused) return;

  if (isNearNpc()) {
    hintElement.textContent = "Pressione E para falar com Dália.";
  } else {
    hintElement.textContent = "Explore a biblioteca e encontre Dália.";
  }
}

function openDialogue() {
  gamePaused = true;
  dialogMode = "intro";

  if (fragments >= missions.length) {
    dialogueTextElement.textContent =
      "Dália: Você recuperou todos os fragmentos do Código Central. A Biblioteca dos Dados está salva. Parabéns!";
    dialogueActionButton.textContent = "Jogar novamente";
    dialogMode = "victory";
  } else if (fragments > 0) {
    dialogueTextElement.textContent =
      `Dália: Obrigada pela ajuda! Você recuperou ${fragments} de 3 fragmentos. Vamos continuar com a próxima missão?`;
    dialogueActionButton.textContent = "Ver missão";
    dialogMode = "nextMission";
  } else {
    dialogueTextElement.textContent =
      "Dália: Olá, desenvolvedor! Sou a guardiã da Biblioteca dos Dados. O Bug Supremo misturou informações importantes. Você pode me ajudar a recuperar o Código Central?";
    dialogueActionButton.textContent = "Ver missão";
  }

  dialogueElement.classList.remove("hidden");
  dialogueActionButton.focus();
}

function startMission() {
  const mission = missions[fragments];
  if (!mission) {
    showVictory();
    return;
  }

  currentMission = mission;
  challengePromptElement.textContent = mission.prompt;
  challengeStatusElement.textContent = "Escolha a primeira etapa.";
  challengeOptionsElement.replaceChildren();

  expectedStepIndex = 0;
  const options = shuffle(mission.steps);

  options.forEach((step) => {
    const button = document.createElement("button");
    button.type = "button";
    button.className = "choice-button";
    button.textContent = step;
    button.addEventListener("click", () => selectStep(step, button));
    challengeOptionsElement.appendChild(button);
  });

  dialogueElement.classList.add("hidden");
  challengeElement.classList.remove("hidden");
  gamePaused = true;
}

function selectStep(step, button) {
  const correctStep = currentMission.steps[expectedStepIndex];

  if (step === correctStep) {
    button.disabled = true;
    expectedStepIndex += 1;

    if (expectedStepIndex === currentMission.steps.length) {
      completeMission();
    } else {
      challengeStatusElement.textContent = "Correto! Escolha a próxima etapa.";
    }
    return;
  }

  expectedStepIndex = 0;
  challengeStatusElement.textContent = "Essa não é a próxima etapa. Tente a sequência novamente.";

  challengeOptionsElement.querySelectorAll("button").forEach((option) => {
    option.disabled = false;
  });
}

function completeMission() {
  score += 100;
  fragments += 1;

  scoreElement.textContent = String(score);
  fragmentsElement.textContent = `${fragments}/${missions.length}`;

  challengeElement.classList.add("hidden");

  if (fragments >= missions.length) {
    showVictory();
    return;
  }

  dialogueTextElement.textContent =
    `Dália: Muito bem! Você concluiu a missão e recuperou um fragmento. Pontuação: ${score} pontos.`;
  dialogueActionButton.textContent = "Continuar";
  dialogMode = "missionComplete";
  dialogueElement.classList.remove("hidden");
  dialogueActionButton.focus();
}

function showVictory() {
  challengeElement.classList.add("hidden");
  dialogueTextElement.textContent =
    `Dália: Excelente trabalho! Você recuperou os 3 fragmentos e salvou a Biblioteca dos Dados. Pontuação final: ${score} pontos.`;
  dialogueActionButton.textContent = "Jogar novamente";
  dialogMode = "victory";
  dialogueElement.classList.remove("hidden");
  dialogueActionButton.focus();
}

function closeDialogue() {
  dialogueElement.classList.add("hidden");
  gamePaused = false;
  updateHint();
}

function resetGame() {
  score = 0;
  fragments = 0;
  expectedStepIndex = 0;
  currentMission = null;

  scoreElement.textContent = "0";
  fragmentsElement.textContent = `0/${missions.length}`;

  player.x = 65;
  player.y = Math.max(80, game.clientHeight - player.height - 45);

  dialogueElement.classList.add("hidden");
  challengeElement.classList.add("hidden");
  gamePaused = false;

  renderPlayer();
  updateHint();
}

function shuffle(items) {
  const shuffled = [...items];

  for (let index = shuffled.length - 1; index > 0; index -= 1) {
    const randomIndex = Math.floor(Math.random() * (index + 1));
    [shuffled[index], shuffled[randomIndex]] = [shuffled[randomIndex], shuffled[index]];
  }

  return shuffled;
}

dialogueActionButton.addEventListener("click", () => {
  if (dialogMode === "intro" || dialogMode === "nextMission") {
    dialogueTextElement.textContent =
      `Dália: ${missions[fragments].prompt} Organize as etapas na ordem correta para completar a missão.`;
    dialogueActionButton.textContent = "Iniciar desafio";
    dialogMode = "missionBriefing";
    return;
  }

  if (dialogMode === "missionBriefing") {
    startMission();
    return;
  }

  if (dialogMode === "missionComplete") {
    closeDialogue();
    return;
  }

  if (dialogMode === "victory") {
    resetGame();
    openDialogue();
  }
});

document.addEventListener("keydown", (event) => {
  const key = event.key.toLowerCase();

  if (["arrowup", "arrowdown", "arrowleft", "arrowright", " "].includes(key)) {
    event.preventDefault();
  }

  if (key === "e" && !gamePaused && isNearNpc()) {
    openDialogue();
    return;
  }

  if (!gamePaused) {
    pressedKeys.add(key);
  }
});

document.addEventListener("keyup", (event) => {
  pressedKeys.delete(event.key.toLowerCase());
});

function gameLoop() {
  if (!gamePaused) {
    let moveX = 0;
    let moveY = 0;

    if (pressedKeys.has("arrowup") || pressedKeys.has("w")) moveY -= 1;
    if (pressedKeys.has("arrowdown") || pressedKeys.has("s")) moveY += 1;
    if (pressedKeys.has("arrowleft") || pressedKeys.has("a")) moveX -= 1;
    if (pressedKeys.has("arrowright") || pressedKeys.has("d")) moveX += 1;

    if (moveX !== 0 && moveY !== 0) {
      moveX *= Math.SQRT1_2;
      moveY *= Math.SQRT1_2;
    }

    player.x += moveX * player.speed;
    player.y += moveY * player.speed;

    renderPlayer();
    updateHint();
  }

  requestAnimationFrame(gameLoop);
}

window.addEventListener("resize", () => {
  placeNpc();
  renderPlayer();
  updateHint();
});

placeNpc();
player.y = Math.max(80, game.clientHeight - player.height - 45);
renderPlayer();
updateHint();
gameLoop();

README.md content to add:

# CodeWorld — Biblioteca dos Dados

CodeWorld é um minijogo educativo sobre Ciência da Computação, desenvolvido em HTML5, CSS3 e JavaScript. O jogador explora a Biblioteca dos Dados, interage com a NPC Dália e resolve desafios de lógica e organização de informações.

## Como jogar

1. Abra o arquivo `index.html` em um navegador.
2. Use as setas do teclado ou `WASD` para mover o personagem.
3. Aproxime-se de Dália e pressione `E` para interagir.
4. Complete as missões na ordem correta para recuperar os fragmentos do Código Central.
5. Acumule pontos e avance para as próximas etapas.

## Objetivo

Recuperar os fragmentos do Código Central, restaurando a ordem dos dados e ajudando a salvar a Biblioteca dos Dados.

## Tecnologias

- HTML5
- CSS3
- JavaScript

## Estrutura do projeto

- `index.html` — estrutura da interface do jogo
- `style.css` — visual do cenário e dos elementos do jogo
- `script.js` — lógica de movimento, interação e desafios

## Autor

Projeto desenvolvido como protótipo educativo para apresentação do minigame CodeWorld.
