<script>
  import { onMount } from "svelte";

  let defaultFlashcards = [
    { question: "le frère", answer: "de broer", seen: false, correct: null },
    { question: "la sœur", answer: "de zus", seen: false, correct: null },
    { question: "le cousin", answer: "de neef", seen: false, correct: null },
    { question: "la cousine", answer: "de nicht", seen: false, correct: null },
    { question: "le grand-père", answer: "de opa", seen: false, correct: null },
    { question: "la grand-mère", answer: "de oma", seen: false, correct: null },

    { question: "drôle", answer: "grappig", seen: false, correct: null },
    { question: "chouette", answer: "leuk", seen: false, correct: null },
    { question: "bien", answer: "goed", seen: false, correct: null },
    { question: "pas mal", answer: "niet slecht", seen: false, correct: null },
    { question: "l'ami", answer: "de vriend", seen: false, correct: null },
    { question: "l'amie", answer: "de vriendin", seen: false, correct: null },

    { question: "le weekend", answer: "het weekend", seen: false, correct: null },
    { question: "hier", answer: "gisteren", seen: false, correct: null },
    { question: "le semaine", answer: "de week", seen: false, correct: null },
    { question: "dernier", answer: "vorige", seen: false, correct: null },
    { question: "prochaine", answer: "volgende", seen: false, correct: null },

    { question: "jouer", answer: "spelen", seen: false, correct: null },
    { question: "le but", answer: "het doelpunt", seen: false, correct: null },
    { question: "l'équipe", answer: "het team", seen: false, correct: null },
    { question: "perdu", answer: "verloren", seen: false, correct: null },
    { question: "gagné", answer: "gewonnen", seen: false, correct: null },
  ];

  let userFlashcards = [];
  let shuffledCards = [];
  let incorrectCards = [];
  let currentIndex = 0;
  let flipped = false;
  let seenCount = 0;
  let score = 0;
  let timer;
  let answerInput = "";
  let feedbackMessage = "";

  let newQuestion = "";
  let newAnswer = "";

  function shuffleCards() {
    if (!defaultFlashcards && !userFlashcards) return;
    shuffledCards = [...defaultFlashcards, ...userFlashcards].sort(() => Math.random() - 0.5);
    resetQuiz();
  }

  function flipCard() {
    flipped = !flipped;
  }

  function checkAnswer() {
    if (!shuffledCards || !shuffledCards[currentIndex]) return;
    
    if (answerInput.trim().toLowerCase() === shuffledCards[currentIndex].answer.toLowerCase()) {
      shuffledCards[currentIndex].correct = true;
      feedbackMessage = "Correct!";
      score += 1; // +1 point for correct answer
    } else {
      shuffledCards[currentIndex].correct = false;
      incorrectCards.push(shuffledCards[currentIndex]);
      feedbackMessage = "Wrong! Try again.";
      score -= 1; // -1 point for wrong answer
    }
    answerInput = "";
  }

  function nextCard() {
    if (!shuffledCards || !shuffledCards[currentIndex]) return;
    
    if (shuffledCards[currentIndex].correct === null) return;

    if (!shuffledCards[currentIndex].seen) {
      shuffledCards[currentIndex].seen = true;
      seenCount++;
    }

    flipped = false;
    feedbackMessage = "";

    if (currentIndex + 1 < shuffledCards.length) {
      currentIndex++;
    } else if (incorrectCards.length > 0) {
      shuffledCards = [...incorrectCards];
      incorrectCards = [];
      currentIndex = 0;
      seenCount = 0;
    } else {
      alert("You have completed all cards!");
      resetQuiz();
    }

    startTimer();
  }

  function startTimer() {
    if (!shuffledCards || shuffledCards.length === 0) return;
    clearTimeout(timer);
    timer = setTimeout(() => {
      flipped = true;
    }, 10000);
  }

  function addFlashcard() {
    if (newQuestion && newAnswer) {
      userFlashcards.push({ question: newQuestion, answer: newAnswer, seen: false, correct: null });
      localStorage.setItem("userFlashcards", JSON.stringify(userFlashcards));
      newQuestion = "";
      newAnswer = "";
      shuffleCards();
    }
  }

  function resetQuiz() {
    seenCount = 0;
    currentIndex = 0;
    incorrectCards = [];
    score = 0; // Reset score when restarting
    
    if (shuffledCards && shuffledCards.length > 0) {
      shuffledCards.forEach(card => {
        card.seen = false;
        card.correct = null;
      });
      startTimer();
    }
  }

  function clearUserCards() {
    localStorage.removeItem("userFlashcards");
    userFlashcards = [];
    shuffleCards();
  }

  onMount(() => {
    // Load flashcards from localStorage only in browser environment
    userFlashcards = JSON.parse(localStorage.getItem("userFlashcards")) || [];
    // Then shuffle cards
    shuffleCards();
  });
</script>

<div class="container">
  <h1>Flashcards</h1>

  <div class="progress-bar">
    <div class="progress" style="width: {(shuffledCards.length > 0 ? (seenCount / shuffledCards.length) * 100 : 0)}%;"></div>
  </div>

  <p><strong>Seen:</strong> {seenCount} / {shuffledCards.length || 0}</p>
  <p><strong>Score:</strong> {score}</p>

  {#if shuffledCards.length > 0 && currentIndex >= 0 && currentIndex < shuffledCards.length && shuffledCards[currentIndex]}
    <div class="flashcard" on:click={flipCard} on:keydown={(e) => e.key === 'Enter' && flipCard()} role="button" tabindex="0">
      <div class="card-content" class:flipped={flipped}>
        <div class="front">{shuffledCards[currentIndex].question}</div>
        <div class="back">{shuffledCards[currentIndex].answer}</div>
      </div>
    </div>
  {:else}
    <div class="flashcard">
      <div class="card-content">
        <div class="front">Loading flashcards...</div>
      </div>
    </div>
  {/if}

  <input 
    type="text" 
    bind:value={answerInput} 
    placeholder="Type your answer" 
    disabled={!shuffledCards || shuffledCards.length === 0 || !shuffledCards[currentIndex]}
  >
  <button 
    on:click={checkAnswer} 
    disabled={!shuffledCards || shuffledCards.length === 0 || !shuffledCards[currentIndex]}
  >
    Check Answer
  </button>
  <p>{feedbackMessage}</p>

  <button 
    on:click={nextCard} 
    disabled={!shuffledCards || shuffledCards.length === 0 || !shuffledCards[currentIndex]}
  >
    Next Card
  </button>
  <button 
    on:click={resetQuiz} 
    class="reset" 
    disabled={!shuffledCards || shuffledCards.length === 0}
  >
    Reset Quiz
  </button>

  <div class="add-card">
    <h2>Add a Flashcard</h2>
    <input type="text" bind:value={newQuestion} placeholder="Question">
    <input type="text" bind:value={newAnswer} placeholder="Answer">
    <button on:click={addFlashcard}>Add</button>
  </div>

  <button on:click={clearUserCards} class="clear">Clear Saved Cards</button>
</div>

<style>
  body {
    font-family: Helvetica, Arial, sans-serif;
    text-align: center;
    background-color: #f4f4f4;
  }

  .container {
    max-width: 400px;
    margin: 20px auto;
  }

  .progress-bar {
    width: 100%;
    height: 15px;
    background-color: #ddd;
    border-radius: 5px;
    margin-bottom: 10px;
    overflow: hidden;
  }

  .progress {
    height: 100%;
    background-color: #28a745;
    transition: width 0.3s ease-in-out;
  }

  .flashcard {
    width: 450px;
    height: 100px;
    perspective: 1000px;
    cursor: pointer;
    margin: 20px auto;
  }

  .card-content {
    width: 100%;
    height: 100%;
    position: relative;
    transform-style: preserve-3d;
    transition: transform 0.5s;
  }

  .flipped {
    transform: rotateY(180deg);
  }

  .front, .back {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    backface-visibility: hidden;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 10px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    font-size: 18px;
    font-weight: bold;
  }

  .front { background: #ffffff; color: #333; }
  .back { background: #007bff; color: white; transform: rotateY(180deg); }
  button { 
    margin-top: 10px; 
    padding: 10px 20px; 
    cursor: pointer; 
    background: #28a745; 
    color: white; 
    border: none; 
    border-radius: 5px; 
  }
  button:disabled {
    background: #cccccc;
    cursor: not-allowed;
    opacity: 0.7;
  }
  .reset { background: #dc3545; }
  .reset:disabled { background: #e89a9e; }
  .clear { background: #ff5733; }
  .clear:disabled { background: #ffb399; }
  input:disabled {
    background-color: #f0f0f0;
    cursor: not-allowed;
  }
</style>

