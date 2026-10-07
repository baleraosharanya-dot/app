const symbols = ["🌙", "⭐", "☀️", "🌈", "🍀", "🪐", "🎵", "🔥"];

const board = document.getElementById("game-board");
const movesLabel = document.getElementById("moves");
const timerLabel = document.getElementById("timer");
const matchesLabel = document.getElementById("matches");
const message = document.getElementById("message");
const restartButton = document.getElementById("restart-button");

let moves = 0;
let matchedPairs = 0;
let seconds = 0;
let timerId = null;
let flippedCards = [];
let isLocked = false;
let hasStarted = false;

function shuffle(array) {
  const copy = [...array];

  for (let i = copy.length - 1; i > 0; i -= 1) {
    const randomIndex = Math.floor(Math.random() * (i + 1));
    [copy[i], copy[randomIndex]] = [copy[randomIndex], copy[i]];
  }

  return copy;
}

function formatTime(totalSeconds) {
  const minutes = Math.floor(totalSeconds / 60)
    .toString()
    .padStart(2, "0");
  const remainingSeconds = (totalSeconds % 60).toString().padStart(2, "0");
  return `${minutes}:${remainingSeconds}`;
}

function updateStats() {
  movesLabel.textContent = String(moves);
  matchesLabel.textContent = `${matchedPairs} / ${symbols.length}`;
  timerLabel.textContent = formatTime(seconds);
}

function startTimer() {
  if (timerId) {
    return;
  }

  timerId = setInterval(() => {
    seconds += 1;
    updateStats();
  }, 1000);
}

function stopTimer() {
  if (timerId) {
    clearInterval(timerId);
    timerId = null;
  }
}

function setMessage(text) {
  message.textContent = text;
}

function buildBoard() {
  const cards = shuffle([...symbols, ...symbols]);

  board.innerHTML = "";

  cards.forEach((symbol, index) => {
    const card = document.createElement("button");
    card.type = "button";
    card.className = "card";
    card.setAttribute("data-symbol", symbol);
    card.setAttribute("aria-label", `Card ${index + 1}`);
    card.innerHTML = `
      <div class="card-inner">
        <span class="card-face card-front">?</span>
        <span class="card-face card-back">${symbol}</span>
      </div>
    `;

    card.addEventListener("click", () => handleCardClick(card));
    board.appendChild(card);
  });
}

function revealCard(card) {
  card.classList.add("flipped");
  card.disabled = true;
}

function hideCard(card) {
  card.classList.remove("flipped");
  card.disabled = false;
}

function handleCardClick(card) {
  if (isLocked || card.classList.contains("flipped") || card.classList.contains("matched")) {
    return;
  }

  if (!hasStarted) {
    startTimer();
    hasStarted = true;
  }

  revealCard(card);
  flippedCards.push(card);

  if (flippedCards.length === 2) {
    moves += 1;
    updateStats();
    isLocked = true;

    const [firstCard, secondCard] = flippedCards;

    if (firstCard.dataset.symbol === secondCard.dataset.symbol) {
      firstCard.classList.add("matched");
      secondCard.classList.add("matched");
      flippedCards = [];
      matchedPairs += 1;
      updateStats();
      isLocked = false;

      if (matchedPairs === symbols.length) {
        stopTimer();
        setMessage(`You won in ${moves} moves and ${formatTime(seconds)}!`);
      } else {
        setMessage("Nice match! Keep going!");
      }
      return;
    }

    setMessage("Not a match — try again!");

    setTimeout(() => {
      hideCard(firstCard);
      hideCard(secondCard);
      flippedCards = [];
      isLocked = false;
    }, 700);
  }
}

function resetGame() {
  moves = 0;
  matchedPairs = 0;
  seconds = 0;
  flippedCards = [];
  isLocked = false;
  hasStarted = false;
  stopTimer();
  setMessage("Find all the matching pairs!");
  updateStats();
  buildBoard();
}

restartButton.addEventListener("click", resetGame);
updateStats();
buildBoard();
