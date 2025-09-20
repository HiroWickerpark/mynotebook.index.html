<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>単語帳アプリ（完成版）</title>
  <style>
    body { font-family: sans-serif; text-align: center; padding: 20px; }
    input { margin: 5px; padding: 5px; }
    button { margin: 5px; padding: 6px 12px; }
    .card {
      border: 1px solid #333; border-radius: 10px;
      width: 250px; height: 150px; margin: 20px auto;
      display: flex; align-items: center; justify-content: center;
      font-size: 22px; cursor: pointer; background: #f8f8f8;
    }
    ul { list-style: none; padding: 0; }
    li { margin: 5px 0; }
    .delete-btn { margin-left: 10px; color: red; }
  </style>
</head>
<body>
  <h1>単語帳（完成版）</h1>

  <h2>単語を追加</h2>
  <input id="term" placeholder="単語">
  <input id="meaning" placeholder="意味">
  <button onclick="addWord()">追加</button>

  <h2>登録済みの単語</h2>
  <ul id="wordList"></ul>

  <h2>フラッシュカード</h2>
  <div id="card" class="card" onclick="flipCard()">ここに表示</div>
  <button onclick="prevCard()">前へ</button>
  <button onclick="nextCard()">次へ</button>
  <br>
  <button onclick="markHard()">苦手</button>
  <button onclick="markEasy()">得意</button>

  <h2>データ管理</h2>
  <button onclick="exportWords()">エクスポート</button>
  <input type="file" accept="application/json" onchange="importWords(event)">

  <h2>学習履歴</h2>
  <div id="history">今日の学習: なし</div>

  <script>
    let words = JSON.parse(localStorage.getItem("words")) || [];
    let history = JSON.parse(localStorage.getItem("history")) || {};
    let currentIndex = 0;
    let showMeaning = false;

    function saveWords() {
      localStorage.setItem("words", JSON.stringify(words));
    }

    function saveHistory() {
      localStorage.setItem("history", JSON.stringify(history));
    }

    function addWord() {
      const term = document.getElementById("term").value.trim();
      const meaning = document.getElementById("meaning").value.trim();
      if (term && meaning) {
        words.push({term, meaning, score: 3}); // 初期スコアは3
        saveWords();
        document.getElementById("term").value = "";
        document.getElementById("meaning").value = "";
        renderList();
      }
    }

    function deleteWord(index) {
      words.splice(index, 1);
      saveWords();
      renderList();
      showCard();
    }

    function renderList() {
      const list = document.getElementById("wordList");
      list.innerHTML = "";
      words.forEach((w, i) => {
        const li = document.createElement("li");
        li.textContent = `${w.term} - ${w.meaning} (score:${w.score})`;
        const btn = document.createElement("button");
        btn.textContent = "削除";
        btn.className = "delete-btn";
        btn.onclick = () => deleteWord(i);
        li.appendChild(btn);
        list.appendChild(li);
      });
    }

    function showCard() {
      if (words.length === 0) {
        document.getElementById("card").textContent = "単語がありません";
        return;
      }
      const word = words[currentIndex];
      document.getElementById("card").textContent = showMeaning ? word.meaning : word.term;
    }

    function flipCard() {
      showMeaning = !showMeaning;
      showCard();
    }

    function nextCard() {
      if (words.length > 0) {
        currentIndex = weightedRandomIndex();
        showMeaning = false;
        showCard();
      }
    }

    function prevCard() {
      if (words.length > 0) {
        currentIndex = (currentIndex - 1 + words.length) % words.length;
        showMeaning = false;
        showCard();
      }
    }

    // 苦手／得意
    function markHard() {
      if (words[currentIndex].score > 1) {
        words[currentIndex].score--;
      }
      saveWords();
      renderList();
      addHistory("hard");
    }

    function markEasy() {
      if (words[currentIndex].score < 5) {
        words[currentIndex].score++;
      }
      saveWords();
      renderList();
      addHistory("easy");
    }

    // 学習履歴の追加
    function addHistory(action) {
      const today = new Date().toISOString().slice(0,10);
      if (!history[today]) {
        history[today] = {hard: 0, easy: 0};
      }
      history[today][action]++;
      saveHistory();
      renderHistory();
    }

    function renderHistory() {
      const today = new Date().toISOString().slice(0,10);
      const stats = history[today] || {hard:0, easy:0};
      document.getElementById("history").textContent =
        `今日の学習: 苦手 ${stats.hard} / 得意 ${stats.easy}`;
    }

    // スコアに応じた重み付きランダム
    function weightedRandomIndex() {
      let weights = words.map(w => 6 - w.score); // scoreが低いほど重みが大きい
      let total = weights.reduce((a, b) => a + b, 0);
      let r = Math.random() * total;
      for (let i = 0; i < weights.length; i++) {
        if (r < weights[i]) return i;
        r -= weights[i];
      }
      return 0;
    }

    // エクスポート
    function exportWords() {
      const data = JSON.stringify(words, null, 2);
      const blob = new Blob([data], {type: "application/json"});
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "words.json";
      a.click();
      URL.revokeObjectURL(url);
    }

    // インポート
    function importWords(event) {
      const file = event.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = function(e) {
        try {
          const imported = JSON.parse(e.target.result);
          words = words.concat(imported);
          saveWords();
          renderList();
          showCard();
        } catch (err) {
          alert("読み込みエラー: JSON形式を確認してください");
        }
      };
      reader.readAsText(file);
    }

    renderList();
    showCard();
    renderHistory();
  </script>
</body>
</html>
