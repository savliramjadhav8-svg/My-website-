
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>My GK World</title>

<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #eef3ff;
  color: #222;
}

header {
  background: #1746a2;
  color: white;
  text-align: center;
  padding: 22px 10px;
}

header h1 { margin: 0; }

.container {
  max-width: 750px;
  margin: auto;
  padding: 15px;
}

.card {
  background: white;
  padding: 18px;
  border-radius: 12px;
  margin: 15px 0;
  box-shadow: 0 2px 8px #ccc;
}

input, select {
  width: 100%;
  padding: 13px;
  border: 1px solid #bbb;
  border-radius: 8px;
  margin: 8px 0;
}

button {
  background: #1746a2;
  color: white;
  border: none;
  padding: 12px 16px;
  border-radius: 8px;
  margin: 5px 2px;
  cursor: pointer;
}

button:disabled { opacity: .5; }

.answer { color: green; font-weight: bold; }

.small { color: #666; font-size: 14px; }

footer {
  text-align: center;
  padding: 20px;
  color: #555;
}
</style>
</head>

<body>

<header>
  <h1>🌍 My GK World</h1>
  <p>Learn Something New Every Day</p>
</header>

<div class="container">

  <div class="card">
    <h2>📚 GK Questions</h2>
    <p id="status">Press the button to load questions.</p>
    <button id="loadBtn" onclick="loadQuestions()">
      Load 1000 GK Questions
    </button>
  </div>

  <div class="card">
    <h3>🔎 Search Questions</h3>
    <input id="search" placeholder="Search a question...">
    <select id="category" onchange="renderQuestions()">
      <option value="all">All Categories</option>
      <option value="9">General Knowledge</option>
      <option value="17">Science & Nature</option>
      <option value="18">Computers</option>
      <option value="21">Sports</option>
      <option value="22">Geography</option>
      <option value="23">History</option>
    </select>
  </div>

  <div class="card">
    <h3>📖 Question List</h3>
    <div id="questions">No questions loaded yet.</div>
    <button onclick="previousPage()">Previous</button>
    <span id="page">Page 1</span>
    <button onclick="nextPage()">Next</button>
  </div>

  <div class="card">
    <h2>🧠 GK Quiz</h2>
    <p id="quizQuestion">Load questions to start quiz.</p>
    <div id="quizOptions"></div>
    <p id="quizResult"></p>
    <p id="score">Score: 0</p>
    <button onclick="startQuiz()">Start / Restart Quiz</button>
    <button onclick="nextQuiz()">Next Question</button>
  </div>

</div>

<footer>
  © 2026 My GK World | Learn Every Day
</footer>

<script>

let allQuestions = [];
let pageNumber = 1;
let pageSize = 10;
let quizQuestions = [];
let quizIndex = 0;
let score = 0;
let answered = false;

function decode(text) {
  let t = document.createElement("textarea");
  t.innerHTML = text;
  return t.value;
}

async function loadQuestions() {
  const btn = document.getElementById("loadBtn");
  const status = document.getElementById("status");

  btn.disabled = true;
  allQuestions = [];

  try {
    // Open Trivia DB allows up to 50 questions per request.
    for (let i = 0; i < 20; i++) {
      status.innerText =
        "Loading questions: " + allQuestions.length + " / 1000";

      let response = await fetch(
        "https://opentdb.com/api.php?amount=50&type=multiple"
      );

      if (!response.ok) throw new Error("Network error");

      let data = await response.json();

      if (data.response_code !== 0 ||
          !data.results || data.results.length === 0) {
        break;
      }

      let batch = data.results.map(q => ({
        question: decode(q.question),
        answer: decode(q.correct_answer),
        options: [
          ...q.incorrect_answers.map(decode),
          decode(q.correct_answer)
        ],
        category: q.category,
        categoryId: q.category === "Science & Nature" ? "17" :
                    q.category === "Computers" ? "18" :
                    q.category === "Sports" ? "21" :
                    q.category === "Geography" ? "22" :
                    q.category === "History" ? "23" : "9"
      }));

      allQuestions.push(...batch);

      renderQuestions();

      // Short pause between API requests.
      await new Promise(resolve => setTimeout(resolve, 6000));
    }

    status.innerText =
      "Loaded " + allQuestions.length + " questions successfully!";

    renderQuestions();

  } catch (error) {
    status.innerText =
      "Could not load questions. Check your internet and try again. " +
      "Loaded: " + allQuestions.length;
  }

  btn.disabled = false;
}

function renderQuestions() {
  const search = document.getElementById("search")
    .value.toLowerCase();

  const category = document.getElementById("category").value;

  const filtered = allQuestions.filter(q =>
    q.question.toLowerCase().includes(search) &&
    (category === "all" || q.categoryId === category)
  );

  const start = (pageNumber - 1) * pageSize;
  const list = filtered.slice(start, start + pageSize);

  const box = document.getElementById("questions");
  box.innerHTML = "";

  if (list.length === 0) {
    box.innerText = "No questions found.";
    return;
  }

  list.forEach((q, index) => {
    const div = document.createElement("div");
    div.className = "card";

    const title = document.createElement("p");
    title.innerText = (start + index + 1) + ". " + q.question;

    const categoryText = document.createElement("p");
    categoryText.className = "small";
    categoryText.innerText = q.category;

    const btn = document.createElement("button");
    btn.innerText = "Show Answer";

    const ans = document.createElement("p");
    ans.className = "answer";
    ans.hidden = true;
    ans.innerText = "Answer: " + q.answer;

    btn.onclick = () => {
      ans.hidden = !ans.hidden;
      btn.innerText = ans.hidden ? "Show Answer" : "Hide Answer";
    };

    div.append(title, categoryText, btn, ans);
    box.appendChild(div);
  });

  document.getElementById("page").innerText =
    "Page " + pageNumber + " | Total: " + filtered.length;
}

function nextPage() {
  let search = document.getElementById("search").value.toLowerCase();
  let cat = document.getElementById("category").value;

  let filtered = allQuestions.filter(q =>
    q.question.toLowerCase().includes(search) &&
    (cat === "all" || q.categoryId === cat)
  );

  if (pageNumber * pageSize < filtered.length) {
    pageNumber++;
    renderQuestions();
  }
}

function previousPage() {
  if (pageNumber > 1) {
    pageNumber--;
    renderQuestions();
  }
}

document.getElementById("search").addEventListener("input", () => {
  pageNumber = 1;
  renderQuestions();
});

function startQuiz() {
  if (allQuestions.length === 0) {
    alert("First load questions!");
    return;
  }

  quizQuestions = [...allQuestions]
    .sort(() => Math.random() - 0.5)
    .slice(0, 10);

  quizIndex = 0;
  score = 0;
  showQuiz();
}

function showQuiz() {
  answered = false;

  let q = quizQuestions[quizIndex];

  document.getElementById("quizQuestion").innerText =
    (quizIndex + 1) + ". " + q.question;

  document.getElementById("quizResult").innerText = "";
  document.getElementById("score").innerText = "Score: " + score;

  let options = [...q.options].sort(() => Math.random() - 0.5);
  let box = document.getElementById("quizOptions");
  box.innerHTML = "";

  options.forEach(option => {
    let btn = document.createElement("button");
    btn.innerText = option;
    btn.style.display = "block";
    btn.style.width = "100%";
    btn.onclick = () => checkAnswer(option, q.answer);
    box.appendChild(btn);
  });
}

function checkAnswer(selected, correct) {
  if (answered) return;
  answered = true;

  if (selected === correct) {
    score++;
    document.getElementById("quizResult").innerText =
      "Correct Answer! 🎉";
  } else {
    document.getElementById("quizResult").innerText =
      "Wrong! Correct answer: " + correct;
  }

  document.getElementById("score").innerText = "Score: " + score;
}

function nextQuiz() {
  if (quizQuestions.length === 0) {
    alert("Start the quiz first!");
    return;
  }

  if (quizIndex < quizQuestions.length - 1) {
    quizIndex++;
    showQuiz();
  } else {
    document.getElementById("quizQuestion").innerText =
      "Quiz completed! Your score: " + score + " / 10";

    document.getElementById("quizOptions").innerHTML = "";
    document.getElementById("quizResult").innerText =
      "Well done! 🎉";
  }
}

</script>
</body>
</html>
