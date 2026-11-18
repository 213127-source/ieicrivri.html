<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>キッズ・ミステリー・チャレンジ</title>
    <style>
        body {
            font-family: 'Hiragino Maru Gothic Pro', 'M PLUS 1p', sans-serif;
            background-color: #e0f7fa;
            color: #333;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        #game-container {
            width: 650px;
            background: white;
            padding: 40px;
            border-radius: 20px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            text-align: center;
            border: 5px solid #4dd0e1;
        }
        h1 { color: #0097a7; margin-bottom: 20px; }
        
        .course-select { display: flex; gap: 15px; justify-content: center; margin-bottom: 30px; }
        .course-btn {
            padding: 15px 25px;
            border: none;
            border-radius: 50px;
            font-size: 1.1em;
            cursor: pointer;
            transition: 0.3s;
            font-weight: bold;
            color: white;
        }
        .sea { background-color: #29b6f6; }
        .mountain { background-color: #66bb6a; }
        .city { background-color: #ffa726; }
        .course-btn:hover { transform: scale(1.1); }

        #puzzle-area { display: none; }
        .question-box {
            background-color: #f1f8e9;
            padding: 25px;
            border-radius: 15px;
            margin-bottom: 20px;
            font-size: 1.3em;
            line-height: 1.6;
            border: 2px dashed #aed581;
        }
        input[type="text"] {
            padding: 12px;
            width: 60%;
            border: 2px solid #ccc;
            border-radius: 8px;
            font-size: 1.1em;
            margin-bottom: 15px;
            text-align: center;
        }
        button.submit-btn {
            padding: 12px 30px;
            background-color: #0097a7;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 1.1em;
            font-weight: bold;
        }
        #feedback { margin-top: 15px; font-weight: bold; font-size: 1.2em; min-height: 30px; }
        .correct { color: #2e7d32; }
        .wrong { color: #c62828; }

        #result-area { display: none; font-size: 2.5em; font-weight: bold; color: #0097a7; animation: popIn 1s; }
        @keyframes popIn { 
            0% { transform: scale(0.5); opacity: 0; } 
            80% { transform: scale(1.1); } 
            100% { transform: scale(1); opacity: 1; } 
        }
    </style>
</head>
<body>

<div id="game-container">
    <h1>🎓 キッズ・ミステリー・チャレンジ</h1>

    <div id="select-area">
        <p>どのコースに挑戦しますか？</p>
        <div class="course-select">
            <button class="course-btn sea" onclick="startCourse('sea')">🌊 海コース</button>
            <button class="course-btn mountain" onclick="startCourse('mountain')">⛰️ 山コース</button>
            <button class="course-btn city" onclick="startCourse('city')">🏙️ 都会コース</button>
        </div>
    </div>

    <div id="puzzle-area">
        <div id="progress">問題 <span id="q-num">1</span> / 4</div>
        <div class="question-box" id="question-text"></div>
        <input type="text" id="answer-input" placeholder="こたえを にゅうりょう">
        <br>
        <button class="submit-btn" onclick="checkAnswer()">せいかい チェック！</button>
        <div id="feedback"></div>
    </div>

    <div id="result-area"></div>
</div>

<script>
    const courses = {
        sea: [
            { q: "【海コース Q1】1円玉が5枚と5円玉が1枚あります。合わせると何円？", a: ["10", "じゅうえん"] },
            { q: "【海コース Q2】「忘れる」の反対語は何？（ひらがなで）", a: ["おぼえる", "覚える"] },
            { q: "【海コース Q3】分速100mで歩くと、1km進むのに何分かかる？", a: ["10", "じゅっぷん"] },
            { q: "【海コース Q4】学校でテストが終わった後、先生が見ているものは？", a: ["せいせき", "成績", "てんすう", "点数"] }
        ],
        mountain: [
            { q: "【山コース Q1】1kgから200gを引くと何g？", a: ["800", "はっぴゃく"] },
            { q: "【山コース Q2】氷を触るとどんな感じ？（ひらがな2文字）", a: ["つめたい", "冷たい", "さむい", "寒い"] },
            { q: "【山コース Q3】ピラミッドの側面になっている図形の名前は？", a: ["さんかくけい", "三角形", "triangle"] },
            { q: "【山コース Q4】一緒にゲームをしたり外で遊んだりする人は？", a: ["ともだち", "友達", "フレンド"] }
        ],
        city: [
            { q: "【都会コース Q1】10 × 10 の答えは？", a: ["100", "ひゃく"] },
            { q: "【都会コース Q2】「左」の反対の方向は？", a: ["みぎ", "右", "right"] },
            { q: "【都会コース Q3】線路の上を走る、たくさんの人が乗れる乗り物は？", a: ["でんしゃ", "電車", "れっしゃ", "列車"] },
            { q: "【都会コース Q4】学校の授業が終わった後、家でやる勉強のこと", a: ["しゅくだい", "宿題", "ホームワーク"] }
        ]
    };

    const finalMessages = {
        sea: "ポケモン",
        mountain: "遊んでくれて",
        city: "ありがとう！"
    };

    let currentCourse = [];
    let currentCourseName = "";
    let currentIndex = 0;

    function startCourse(courseName) {
        currentCourse = courses[courseName];
        currentCourseName = courseName;
        currentIndex = 0;
        document.getElementById('select-area').style.display = 'none';
        document.getElementById('puzzle-area').style.display = 'block';
        showQuestion();
    }

    function showQuestion() {
        if (currentIndex >= currentCourse.length) {
            showResult();
            return;
        }
        document.getElementById('q-num').innerText = currentIndex + 1;
        document.getElementById('question-text').innerText = currentCourse[currentIndex].q;
        document.getElementById('answer-input').value = "";
        document.getElementById('feedback').innerText = "";
        document.getElementById('answer-input').focus();
    }

    function checkAnswer() {
        const input = document.getElementById('answer-input').value.trim().toLowerCase();
        const answers = currentCourse[currentIndex].a.map(a => a.toLowerCase());
        const feedback = document.getElementById('feedback');

        if (answers.includes(input)) {
            feedback.innerText = "せいかい！ よくできました 👍";
            feedback.className = "correct";
            setTimeout(() => {
                currentIndex++;
                showQuestion();
            }, 1200);
        } else {
            feedback.innerText = "ざんねん... もういちど やってみよう！";
            feedback.className = "wrong";
        }
    }

    function showResult() {
        document.getElementById('puzzle-area').style.display = 'none';
        const resultArea = document.getElementById('result-area');
        resultArea.style.display = 'block';
        resultArea.innerText = finalMessages[currentCourseName];
    }

    document.getElementById("answer-input").addEventListener("keyup", function(event) {
        if (event.key === "Enter") checkAnswer();
    });
</script>

</body>
</html>
