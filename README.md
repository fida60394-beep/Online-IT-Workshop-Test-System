<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Online IT Workshop Test</title>
<style>
body{font-family:Segoe UI,Tahoma,sans-serif;background:#f4f6fb;margin:0}
.header{background:#2563eb;color:#fff;padding:12px;text-align:center}
.container{max-width:900px;margin:25px auto;background:#fff;padding:25px;border-radius:12px;box-shadow:0 5px 20px rgba(0,0,0,.08)}
h1,h2{text-align:center}
button{background:#2563eb;color:#fff;border:none;padding:10px 18px;border-radius:6px;cursor:pointer;margin:4px}
button:hover{background:#1e4fd1}
input,select{width:100%;padding:8px;margin:6px 0;border:1px solid #ccc;border-radius:6px}
.hidden{display:none}
.nav{display:flex;gap:10px;margin-bottom:15px}
.timer{color:red;font-weight:bold;text-align:right}
.question{background:#f8fafc;padding:12px;border-radius:8px;margin-bottom:15px}
.admin-card{background:#f1f5f9;padding:10px;border-radius:8px;margin:8px 0}
.small-btn{background:#dc2626;padding:5px 10px;font-size:12px}
.result-pass{color:green;font-weight:bold}
.result-fail{color:#dc2626;font-weight:bold}
.certificate{border:4px solid #2563eb;padding:30px;text-align:center;margin-top:20px;border-radius:12px}
.result-list{background:#e0e7ff;padding:10px;margin:5px 0;border-radius:8px}
.review-box{background:#fff7ed;padding:10px;border-radius:8px;margin:10px 0}
</style>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
</head>
<body>

<div class="header">
<img src="logo.PNG" alt="Logo.PNG" alt="" width="6%">
<h1>COLLEGE OF PHYSICIANS & SURGEONS PAKISTAN</h1>
<h1>ONLINE IT WORKSHOP TEST</h1>
</div>

<div class="container">
<div class="nav">
<button onclick="showCandidate()">Candidate Login</button>
<button onclick="showAdminLogin()">Admin Login</button>
</div>

<div id="candidateLogin">
<h1>Candidate Login</h1>
<input type="text" id="studentName" placeholder="Enter your name">
<input type="text" id="CPSPID" placeholder="Enter your CPSP ID">
<input type="text" id="Speciality" placeholder="Enter your Speciality">
<select id="subjectFilter">
<option value="ALL">All Subjects</option>
<option value="Basic Computer & IT">Basic Computer & IT</option>
<option value="MS Word & MS PowerPoint">MS Word & MS PowerPoint</option>
<option value="Basic SPSS">Basic SPSS</option>
</select>
<button onclick="startTest()">Start Test</button>
</div>

<div id="quizBox" class="hidden">
<div id="timer" class="timer">Time: 60s</div>
<h2 id="qCounter"></h2>
<form id="quizForm"></form>
<div class="review-box" id="reviewPanel"></div>
<button onclick="nextQuestion()">Next</button>
<button onclick="submitTest()">Submit Test</button>
</div>

<div id="resultBox" class="hidden">
<h2>Test Result</h2>
<div id="resultText"></div>
<div id="certificateBox"></div>
<button onclick="downloadPDF()">📄 Download Result PDF</button>
<button onclick="logoutCandidate()">Logout</button>
</div>

<div id="adminLogin" class="hidden">
<h1>Admin Login</h1>
<input type="password" id="adminPass" placeholder="Enter admin password">
<button onclick="adminLoginHandler()">Login</button>
</div>

<div id="adminPanel" class="hidden">
<h1>Admin Dashboard</h1>
<button onclick="logoutAdmin()">Logout</button>

<h3>📊 Add / Delete Questions</h3>
<input id="qText" placeholder="Question">
<input id="opt1" placeholder="Option 1">
<input id="opt2" placeholder="Option 2">
<input id="opt3" placeholder="Option 3">
<input id="opt4" placeholder="Option 4">
<select id="correct">
<option value="">Select Correct Answer</option>
<option>Option 1</option>
<option>Option 2</option>
<option>Option 3</option>
<option>Option 4</option>
</select>
<button onclick="addQuestion()">Add Question</button>
<h3>All Questions</h3>
<div id="questionList"></div>

<h3>📥 CSV Question Import</h3>
<input type="file" id="csvFile">
<button onclick="importCSV()">Import CSV</button>

<h3>📊 Result Analytics</h3>
<div id="analytics"></div>
<h3>Candidate Results</h3>
<div id="candidateResults"></div>
</div>

</div>

<script>
function safeJSONParse(key,fallback){try{const raw=localStorage.getItem(key);if(!raw)return fallback;const parsed=JSON.parse(raw);return parsed??fallback}catch(e){console.warn('Corrupted localStorage for',key);return fallback;}}
function ensureArray(value,fallback){return Array.isArray(value)?value:fallback;}

document.addEventListener('DOMContentLoaded',()=>{
window.candidateLogin=document.getElementById('candidateLogin');
window.quizBox=document.getElementById('quizBox');
window.resultBox=document.getElementById('resultBox');
window.adminLogin=document.getElementById('adminLogin');
window.adminPanel=document.getElementById('adminPanel');
window.quizForm=document.getElementById('quizForm');
window.resultText=document.getElementById('resultText');
window.studentName=document.getElementById('studentName');
window.adminPass=document.getElementById('adminPass');
window.analytics=document.getElementById('analytics');
window.candidateResults=document.getElementById('candidateResults');
window.certificateBox=document.getElementById('certificateBox');
window.subjectFilter=document.getElementById('subjectFilter');
window.qCounter=document.getElementById('qCounter');
window.reviewPanel=document.getElementById('reviewPanel');
window.qText=document.getElementById('qText');
window.opt1=document.getElementById('opt1');
window.opt2=document.getElementById('opt2');
window.opt3=document.getElementById('opt3');
window.opt4=document.getElementById('opt4');
window.correct=document.getElementById('correct');
window.questionList=document.getElementById('questionList');
});

const NEGATIVE_MARK=0,QUESTIONS_PER_TEST=25;
const defaultQuestions=[
// Add 25+ questions here

// ...continue to 25+ questions
];
let questions=ensureArray(safeJSONParse('questions',defaultQuestions),defaultQuestions);
let results=ensureArray(safeJSONParse('results',[]),[]);

let currentQuestion=0,timeLeft=60,timer,shuffledQuestions=[],answers=[];

function hideAll(){candidateLogin.classList.add('hidden');quizBox.classList.add('hidden');resultBox.classList.add('hidden');adminLogin.classList.add('hidden');adminPanel.classList.add('hidden');}
function showCandidate(){hideAll();candidateLogin.classList.remove('hidden');}
function showAdminLogin(){hideAll();adminLogin.classList.remove('hidden');}
function adminLoginHandler(){if(adminPass.value==='admin123'){hideAll();adminPanel.classList.remove('hidden');renderAnalytics();renderCandidateResults();renderQuestions();}else alert('Wrong password');}
function logoutAdmin(){location.reload();}
function renderAnalytics(){const total=results.length,pass=results.filter(r=>r.status==='PASS').length,fail=results.filter(r=>r.status==='FAIL').length;analytics.innerHTML=`Attempts: <b>${total}</b><br>Pass: <b>${pass}</b><br>Fail: <b>${fail}</b>`;}
function renderCandidateResults(){candidateResults.innerHTML='';results.forEach((r,i)=>{candidateResults.innerHTML+=`<div class="result-list"><b>${i+1}. ${r.name}</b> — ${Number(r.score).toFixed(1)}% (${r.status})</div>`;});}

function addQuestion(){if(!qText.value.trim())return alert('Enter question');const opts=[opt1.value,opt2.value,opt3.value,opt4.value];const ansIndex=correct.selectedIndex-1;if(ansIndex<0)return alert('Select correct answer');questions.push({q:qText.value,o:opts,a:opts[ansIndex],s:'Computer'});localStorage.setItem('questions',JSON.stringify(questions));renderQuestions();alert('Question added');}
function renderQuestions(){questionList.innerHTML='';questions.forEach((q,i)=>{questionList.innerHTML+=`<div class="admin-card"><b>${i+1}. ${q.q}</b><br><button class="small-btn" onclick="deleteQuestion(${i})">Delete</button></div>`;});}
function deleteQuestion(i){if(!confirm('Delete this question?'))return;questions.splice(i,1);localStorage.setItem('questions',JSON.stringify(questions));renderQuestions();}

function importCSV(){const file=document.getElementById('csvFile').files[0];if(!file)return alert('Select CSV');const reader=new FileReader();reader.onload=e=>{const rows=e.target.result.split('\n');rows.forEach(r=>{const [q,o1,o2,o3,o4,a,s]=r.split(',');if(q&&o1&&a)questions.push({q,o:[o1,o2,o3,o4],a,s:s||'Computer'});});localStorage.setItem('questions',JSON.stringify(questions));alert('CSV Imported');};reader.readAsText(file);}

function shuffle(arr){return arr.sort(()=>Math.random()-0.5);}
function startTest(){if(!studentName.value.trim())return alert('Enter name');if(!Array.isArray(questions)||questions.length===0)return alert('Question bank empty');hideAll();quizBox.classList.remove('hidden');let filtered=subjectFilter.value==='ALL'?questions:questions.filter(q=>q.s===subjectFilter.value);if(!filtered.length)filtered=questions;shuffledQuestions=shuffle([...filtered]).slice(0,QUESTIONS_PER_TEST);answers=new Array(shuffledQuestions.length).fill(null);currentQuestion=0;loadQuestion();startTimer();}
function loadQuestion(){const q=shuffledQuestions[currentQuestion];if(!q)return submitTest();qCounter.innerText=`Question ${currentQuestion+1} / ${shuffledQuestions.length}`;let html=`<div class="question"><b>${q.q}</b><br>`;q.o.forEach(opt=>{html+=`<label><input type="radio" name="q" value="${opt}" ${answers[currentQuestion]===opt?'checked':''}> ${opt}</label><br>`});html+='</div>';quizForm.innerHTML=html;updateReview();timeLeft=60;}
function updateReview(){reviewPanel.innerHTML=answers.map((a,i)=>`<button class="small-btn" onclick="goTo(${i})">${i+1}${a?'✓':''}</button>`).join(' ');}
function saveAnswer(){const sel=document.querySelector('input[name="q"]:checked');if(sel)answers[currentQuestion]=sel.value;}
function startTimer(){clearInterval(timer);timer=setInterval(()=>{timeLeft--;document.getElementById('timer').innerText='Time: '+timeLeft+'s';if(timeLeft<=0)nextQuestion();},1000);}
function nextQuestion(){saveAnswer();currentQuestion++;currentQuestion>=shuffledQuestions.length?submitTest():loadQuestion();}
function skipQuestion(){currentQuestion++;currentQuestion>=shuffledQuestions.length?submitTest():loadQuestion();}
function goTo(i){saveAnswer();currentQuestion=i;loadQuestion();}
function submitTest(){clearInterval(timer);saveAnswer();let correct=0,attempted=0,wrong=0;shuffledQuestions.forEach((q,i)=>{const ans=answers[i];if(ans!==null){attempted++;if(ans===q.a)correct++;else wrong++;}});const total=shuffledQuestions.length;let score=correct-(wrong*NEGATIVE_MARK);if(score<0)score=0;let percent=(score/total)*100;const status=percent>=50?'PASS':'FAIL';results.push({name:studentName.value,score:percent,status});localStorage.setItem('results',JSON.stringify(results));hideAll();resultBox.classList.remove('hidden');resultText.innerHTML=`Name: <b>${studentName.value}</b><br>Total: ${total}<br>Attempted: ${attempted}<br>Correct: ${correct}<br>Incorrect: ${wrong}<br>Score: ${percent.toFixed(1)}%<br>Status: <b>${status}</b>`;certificateBox.innerHTML=status==='PASS'?`<div class="certificate"><h2>🎓 Certificate of Achievement</h2><h1>${studentName.value}</h1><p>has successfully passed the Online IT Workshop Test</p></div>`:'';renderCandidateResults();}
function logoutCandidate(){location.reload();}
async function downloadPDF(){const { jsPDF }=window.jspdf;const doc=new jsPDF();doc.text('Online IT Workshop Test Result',20,20);doc.text(resultText.innerText,20,40);doc.save('result.pdf');}

console.assert(Array.isArray(questions),'Questions must be array');
console.assert(questions.length>=25,'Question bank must have at least 25 questions');
console.assert(typeof startTest==='function','startTest must exist');
console.assert(typeof importCSV==='function','CSV import must exist');
</script>

<footer style="text-align:center;padding:15px;margin-top:20px;color:#666;font-size:13px;">
© <span id="year"></span> Fida Hussain Soomro, Assistant Manager IT, CPSP Gambat. All rights reserved.
</footer>

<script>
// auto year for copyright
try{document.getElementById('year').innerText=new Date().getFullYear();}catch(e){}
</script>

</body>
</html>



