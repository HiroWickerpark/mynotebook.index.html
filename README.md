<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>単語帳アプリ</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    h1 { text-align: center; }
    .card { border: 1px solid #ccc; padding: 15px; border-radius: 10px;
            box-shadow: 2px 2px 6px rgba(0,0,0,0.1); max-width: 400px; margin: 10px auto; }
    .hidden { display: none; }
    input, button { margin: 5px 0; padding: 8px; width: 100%; }
    .word-item { margin: 5px 0; padding: 5px; border-bottom: 1px solid #ddd; cursor: pointer; }
  </style>
</head>
<body>

<h1>単語帳アプリ</h1>

<div class="card">
  <h2>単語を追加</h2>
  <input type="text" id="wordInput" placeholder="単語を入力">
  <input type="text" id="meaningInput" placeholder="意味を入力">
  <button id="addButton">追加</button>
</div>

<div class="card">
  <button id="randomButton">ランダム問題を出す</button>
</div>

<!-- 問題ページ -->
<div id="questionPage" class="card hidden">
  <h2>問題</h2>
  <p id="questionWord"></p>
  <button onclick="showAnswer()">答えを見る</button>
</div>

<!-- 答えページ -->
<div id="answerPage" class="card hidden">
  <h2>答え</h2>
  <p id="answerMeaning"></p>
  <button onclick="backToQuestion()">戻る</button>
  <button onclick="nextRandom()">次の問題へ</button>
</div>

<!-- 単語一覧 -->
<div class="card">
  <h2>単語一覧</h2>
  <div id="wordList"></div>
</div>

<script>
  let words = [];
  let currentIndex = 0;

  const wordInput = document.getElementById('wordInput');
  const meaningInput = document.getElementById('meaningInput');
  const addButton = document.getElementById('addButton');
  const randomButton = document.getElementById('randomButton');
  const wordList = document.getElementById('wordList');
  const questionPage = document.getElementById('questionPage');
  const answerPage = document.getElementById('answerPage');
  const questionWord = document.getElementById('questionWord');
  const answerMeaning = document.getElementById('answerMeaning');

  function renderList() {
    wordList.innerHTML = '';
    words.forEach((item, index) => {
      const div = document.createElement('div');
      div.className = 'word-item';
      div.textContent = item.word;
      div.onclick = () => startQuestion(index);
      wordList.appendChild(div);
    });
  }

  function startQuestion(index) {
    currentIndex = index;
    questionWord.textContent = words[index].word;
    questionPage.classList.remove('hidden');
    answerPage.classList.add('hidden');
  }

  function showAnswer() {
    answerMeaning.textContent = words[currentIndex].meaning;
    questionPage.classList.add('hidden');
    answerPage.classList.remove('hidden');
  }

  function backToQuestion() {
    questionPage.classList.remove('hidden');
    answerPage.classList.add('hidden');
  }

  function nextRandom() {
    if (words.length > 0) {
      const randomIndex = Math.floor(Math.random() * words.length);
      startQuestion(randomIndex);
      showAnswer();
    }
  }

  addButton.addEventListener('click', () => {
    const word = wordInput.value.trim();
    const meaning = meaningInput.value.trim();
    if (word && meaning) {
      words.push({ word, meaning });
      renderList();
      wordInput.value = '';
      meaningInput.value = '';
    }
  });

  randomButton.addEventListener('click', () => {
    if (words.length > 0) {
      const randomIndex = Math.floor(Math.random() * words.length);
      startQuestion(randomIndex);
    }
  });

  // Enterキー対応
  [wordInput, meaningInput].forEach(input => {
    input.addEventListener('keypress', e => {
      if (e.key === 'Enter') {
        addButton.click();
      }
    });
  });
</script>
</body>
</html>
