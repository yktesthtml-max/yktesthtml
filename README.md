<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>YK Test</title>
<style>
  body { font-family: Arial,sans-serif; text-align:center; background:linear-gradient(to right,#4facfe,#00f2fe); margin:0; padding:0; }
  .container { max-width:600px; margin:40px auto; background:#fff; padding:20px; border-radius:12px; box-shadow:0 4px 10px rgba(0,0,0,0.2); }
  button { margin:10px; padding:12px 20px; border:none; border-radius:8px; background:#4facfe; color:white; font-size:16px; cursor:pointer; }
  button:hover { background:#00c6ff; }
  #question { margin:20px 0; font-size:18px; }
  .option { display:block; margin:8px 0; padding:10px; border-radius:8px; border:1px solid #ddd; cursor:pointer; }
  .option:hover { background:#f0f0f0; }
  #feedback { margin:15px 0; font-weight:bold; }
  #backBtn { margin-top:20px; background:#ff5252; }
  input { width:80%; padding:8px; margin:5px; border-radius:6px; border:1px solid #ccc; }
  .score { margin-top:10px; font-weight:600; }
</style>
</head>
<body>

<div class="container" id="loginScreen">
  <h2>YK TEST</h2>
  <h2>Yapan:Yasin Emir Kahya</h2>
  <h2>Destekliyen: Muhammet Yunus Mercimek</h2>
  <input type="text" id="username" placeholder="Kullanıcı Adı - yktest" value="">
  <input type="password" id="password" placeholder="Şifre - yktest" value="">
  <br><br>
  <button onclick="login()">Giriş</button>
</div>

<div class="container" id="menuScreen" style="display:none;">
  <h2>Ders Seç</h2>
  <button onclick="startQuiz('matematik')">Matematik</button>
  <button onclick="startQuiz('turkce')">Türkçe</button>
  <button onclick="startQuiz('fen')">Fen</button>
  <button onclick="startQuiz('inkilap')">İnkılap</button>
  <button onclick="startQuiz('din')">Din</button>
  <button onclick="startQuiz('ingilizce')">İngilizce</button>
</div>

<div class="container" id="quizScreen" style="display:none;">
  <div id="question"></div>
  <div id="options"></div>
  <div id="feedback"></div>
  <div class="score">
    Doğru: <span id="cCount">0</span> | Yanlış: <span id="wCount">0</span>
  </div>
  <button id="nextBtn" onclick="nextQuestion()" style="display:none;">Sonraki Soru</button>
  <button id="backBtn" onclick="goBack()">Geri</button>
</div>

<script>
const USERNAME = "yktest";
const PASSWORD = "yktest";

const sources = {
  matematik: "https://raw.githubusercontent.com/yasinemirkahya1122-del/yktestsorular/refs/heads/main/matematik.json",
  turkce: "https://raw.githubusercontent.com/yasinemirkahya1122-del/yktestsorular/refs/heads/main/t%C3%BCrk%C3%A7e.json",
  fen: "https://raw.githubusercontent.com/yasinemirkahya1122-del/yktestsorular/refs/heads/main/fen.json",
  inkilap: "https://raw.githubusercontent.com/yasinemirkahya1122-del/yktestsorular/refs/heads/main/ink%C4%B1lap.json",
  din: "https://raw.githubusercontent.com/yasinemirkahya1122-del/yktestsorular/refs/heads/main/din.json",
  ingilizce:"https://raw.githubusercontent.com/yasinemirkahya1122-del/yktestsorular/refs/heads/main/ingilizce.json"
};

let questions = [];
let current = 0;
let correct = 0;
let wrong = 0;
let currentSubject = null;

// localStorage yükleme
function loadState() {
  const saved = JSON.parse(localStorage.getItem("ykTestState")||"{}");
  if(saved.subject) currentSubject = saved.subject;
  if(saved.current) current = saved.current;
  if(saved.correct) correct = saved.correct;
  if(saved.wrong) wrong = saved.wrong;
}
function saveState() {
  localStorage.setItem("ykTestState", JSON.stringify({
    subject: currentSubject,
    current: current,
    correct: correct,
    wrong: wrong
  }));
}

// login
function login() {
  const u = document.getElementById("username").value.trim();
  const p = document.getElementById("password").value.trim();
  if(u === USERNAME && p === PASSWORD){
    document.getElementById("loginScreen").style.display = "none";
    if(currentSubject){
      startQuiz(currentSubject); // kaldığı yerden devam
    } else {
      document.getElementById("menuScreen").style.display = "block";
    }
  } else {
    alert("Hatalı kullanıcı adı veya şifre!");
  }
}

async function startQuiz(subject) {
  currentSubject = subject;
  await loadQuestions(subject);
  document.getElementById("menuScreen").style.display = "none";
  document.getElementById("quizScreen").style.display = "block";
  showQuestion();
}

// load questions from github (simple fetch)
async function loadQuestions(subject) {
  const savedQs = localStorage.getItem("ykQuestions_" + subject);
  if(savedQs){
    questions = JSON.parse(savedQs);
    return;
  }
  const url = sources[subject];
  try {
    const res = await fetch(url);
    questions = await res.json();
    localStorage.setItem("ykQuestions_" + subject, JSON.stringify(questions));
  } catch(e){
    alert("Sorular yüklenemedi!");
    questions = [];
  }
}

function showQuestion() {
  if(current >= questions.length){
    document.getElementById("question").innerHTML = "Tebrikler! Tüm soruları bitirdiniz 📝<br>Yakında yeni sorular gelecek. BAŞARILAR";
    document.getElementById("options").innerHTML = "";
    document.getElementById("nextBtn").style.display = "none";
    return;
  }
  const q = questions[current];
  document.getElementById("question").innerText = q.Soru;
  document.getElementById("options").innerHTML = "";
  q.Şıklar.forEach((opt,i) => {
    const btn = document.createElement("button");
    btn.innerText = opt;
    btn.className = "option";
    btn.onclick = () => checkAnswer(opt, q["Doğru cevap"]);
    document.getElementById("options").appendChild(btn);
  });
  document.getElementById("feedback").innerText = "";
  document.getElementById("nextBtn").style.display = "none";
  document.getElementById("cCount").innerText = correct;
  document.getElementById("wCount").innerText = wrong;
}

function checkAnswer(selected, correctAnswer) {
  if(selected === correctAnswer){
    correct++;
    document.getElementById("feedback").innerText = "Doğru ✅";
  } else {
    wrong++;
    document.getElementById("feedback").innerText = "Yanlış ❌";
  }
  document.getElementById("nextBtn").style.display = "block";
  saveState();
}

function nextQuestion(){
  current++;
  showQuestion();
  saveState();
}

function goBack(){
  document.getElementById("quizScreen").style.display = "none";
  document.getElementById("menuScreen").style.display = "block";
  saveState();
}

// sayfa yenilenince state yükle
window.onload = () => {
  loadState();
};
</script>
</body>
</html>
