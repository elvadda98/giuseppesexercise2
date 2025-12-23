<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Easy-Going Vocab Practice</title>
    <style>
        :root {
            --primary-color: #2c3e50;
            --secondary-color: #3498db;
            --accent-color: #1abc9c;
            --light-bg: #f9fbfd;
            --dark-text: #2c3e50;
            --success-color: #27ae60;
            --warning-color: #f39c12; /* Yellow for typos */
            --error-color: #e74c3c;
        }

        body { font-family: 'Segoe UI', Tahoma, sans-serif; background: var(--light-bg); color: var(--dark-text); margin: 0; padding-bottom: 50px; }
        header { background: var(--primary-color); color: white; padding: 25px; text-align: center; position: relative; }
        .score-display { position: absolute; top: 25px; right: 25px; background: rgba(255,255,255,0.1); padding: 5px 15px; border-radius: 20px; font-weight: bold; }
        .container { max-width: 950px; margin: 30px auto; padding: 0 20px; }
        
        .nav-buttons { display: flex; gap: 10px; margin-bottom: 25px; flex-wrap: wrap; }
        .nav-button { flex: 1; padding: 12px; border: none; border-radius: 8px; background: #dfe6e9; cursor: pointer; font-weight: bold; transition: 0.3s; }
        .nav-button.active { background: var(--secondary-color); color: white; }

        .exercise-section { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.05); }
        .word-bank { display: flex; flex-wrap: wrap; gap: 12px; background: #f1f4f8; padding: 20px; border-radius: 10px; margin-bottom: 20px; border: 2px dashed var(--secondary-color); }
        .word-bank-item { background: white; padding: 8px 15px; border-radius: 6px; font-weight: bold; box-shadow: 0 2px 5px rgba(0,0,0,0.1); color: var(--primary-color); }

        .gap-sentence { margin: 15px 0; font-size: 1.1em; line-height: 1.6; padding: 10px; border-left: 4px solid transparent; }
        .gap-input { border: none; border-bottom: 2px solid var(--secondary-color); width: 150px; text-align: center; font-size: inherit; outline: none; background: transparent; }
        
        .status-green { color: var(--success-color); font-weight: bold; }
        .status-yellow { color: var(--warning-color); font-weight: bold; }
        .status-red { color: var(--error-color); font-weight: bold; }
        .correction-note { font-size: 0.85em; font-weight: normal; font-style: italic; display: block; margin-top: 4px; }

        .speaker-btn, .mic-btn { background: #f1f4f8; border: none; border-radius: 50%; width: 40px; height: 40px; cursor: pointer; display: inline-flex; align-items: center; justify-content: center; font-size: 1.2em; }
        .action-btn { background: var(--accent-color); color: white; border: none; padding: 12px 30px; border-radius: 8px; cursor: pointer; font-weight: bold; margin-top: 20px; }
    </style>
</head>
<body>

<header>
    <h1>Vocabulary Practice</h1>
    <div class="score-display">Total Score: <span id="score-val">0</span></div>
</header>

<div class="container">
    <div class="nav-buttons">
        <button class="nav-button active" onclick="switchTab(0)">1. Study List</button>
        <button class="nav-button" onclick="switchTab(1)">2. Writing</button>
        <button class="nav-button" onclick="switchTab(2)">3. Pronunciation</button>
    </div>

    <div id="tab-0" class="exercise-section">
        <h2>📚 Vocabulary List</h2>
        <div id="vocab-list"></div>
    </div>

    <div id="tab-1" class="exercise-section" style="display:none">
        <h2>✍️ Writing (Typos Allowed)</h2>
        <div id="writing-bank" class="word-bank"></div>
        <div id="writing-content"></div>
        <button class="action-btn" onclick="checkWriting()">Check Answers</button>
    </div>

    <div id="tab-2" class="exercise-section" style="display:none">
        <h2>🗣️ Pronunciation (Guess & Speak)</h2>
        <div id="pron-bank" class="word-bank"></div>
        <div id="pron-content"></div>
    </div>
</div>

<script>
    const data = [
        { 
            word: "low-season", 
            def: "The time of year when travel is less popular and cheaper.", 
            writingSents: ["I prefer traveling during the ___ because there are fewer crowds.", "Hotels are cheaper during the ___."],
            pronSents: ["Flights are cheaper in the ___.", "We saved money by visiting during the ___."]
        },
        { 
            word: "rusty", 
            def: "Out of practice in a skill; or covered in iron oxide.", 
            writingSents: ["My English is a bit ___ lately.", "That old gate is very ___."],
            pronSents: ["I feel quite ___ at playing chess.", "Watch out for the ___ nails."]
        },
        { 
            word: "replace", 
            def: "To take the place of something.", 
            writingSents: ["I need to ___ my phone battery.", "They decided to ___ the old carpet."],
            pronSents: ["Can you ___ the bulb?", "We should ___ this part."]
        },
        { 
            word: "used to", 
            def: "Becoming accustomed to something.", 
            writingSents: ["I am getting ___ the heat.", "She is slowly getting ___ the new city."],
            pronSents: ["Are you ___ working at night?", "I'm not ___ this noise yet."]
        }
    ];

    let totalScore = 0;

    function getEditDistance(a, b) {
        a = a.replace(/[-\s]/g, ''); b = b.replace(/[-\s]/g, ''); // Ignore dashes/spaces for typo check
        if (a.length === 0) return b.length; 
        if (b.length === 0) return a.length;
        var matrix = [];
        for (var i = 0; i <= b.length; i++) { matrix[i] = [i]; }
        for (var j = 0; j <= a.length; j++) { matrix[0][j] = j; }
        for (i = 1; i <= b.length; i++) {
            for (j = 1; j <= a.length; j++) {
                if (b.charAt(i - 1) == a.charAt(j - 1)) { matrix[i][j] = matrix[i-1][j-1]; } 
                else { matrix[i][j] = Math.min(matrix[i-1][j-1] + 1, Math.min(matrix[i][j-1] + 1, matrix[i-1][j] + 1)); }
            }
        }
        return matrix[b.length][a.length];
    }

    function speak(text) {
        const synth = window.speechSynthesis;
        const utterance = new SpeechSynthesisUtterance(text);
        const voices = synth.getVoices();
        const nativeVoice = voices.find(v => (v.lang === 'en-US' || v.lang === 'en-GB') && v.name.includes('Google')) || voices.find(v => v.lang.includes('en')) || voices[0];
        utterance.voice = nativeVoice;
        utterance.rate = 0.9;
        synth.speak(utterance);
    }

    function switchTab(idx) {
        document.querySelectorAll('.exercise-section').forEach((s, i) => s.style.display = i === idx ? 'block' : 'none');
        document.querySelectorAll('.nav-button').forEach((b, i) => b.classList.toggle('active', i === idx));
    }

    function init() {
        const list = document.getElementById('vocab-list');
        list.innerHTML = data.map(item => `<div class="vocab-item"><div class="word-cell">${item.word}</div><button class="speaker-btn" onclick="speak('${item.word}')">🔊</button><div>${item.def}</div></div>`).join('');

        const wordList = data.map(d => d.word);
        document.getElementById('writing-bank').innerHTML = shuffle([...wordList]).map(w => `<div class="word-bank-item">${w}</div>`).join('');
        document.getElementById('pron-bank').innerHTML = shuffle([...wordList]).map(w => `<div class="word-bank-item">${w}</div>`).join('');

        const wCont = document.getElementById('writing-content');
        wCont.innerHTML = "";
        data.forEach((item, i) => {
            item.writingSents.forEach((s, j) => {
                const id = `w-in-${i}-${j}`;
                wCont.innerHTML += `<div class="gap-sentence">${s.replace('___', `<input type="text" class="gap-input" data-ans="${item.word}" id="${id}">`)} <div id="fb-${id}"></div></div>`;
            });
        });

        const pCont = document.getElementById('pron-content');
        pCont.innerHTML = "";
        data.forEach((item, i) => {
            item.pronSents.forEach((s, j) => {
                const id = `p-fb-${i}-${j}`;
                pCont.innerHTML += `<div class="gap-sentence">${s.replace(item.word, '_________')} <button class="mic-btn" onclick="testSpeech('${item.word}', '${id}')">🎤</button> <div id="${id}" style="color:#888; font-size: 0.8em;">Click & guess!</div></div>`;
            });
        });
    }

    function shuffle(array) {
        let currentIndex = array.length, randomIndex;
        while (currentIndex != 0) {
            randomIndex = Math.floor(Math.random() * currentIndex);
            currentIndex--;
            [array[currentIndex], array[randomIndex]] = [array[randomIndex], array[currentIndex]];
        }
        return array;
    }

    function checkWriting() {
        document.querySelectorAll('.gap-input').forEach(input => {
            const fb = document.getElementById(`fb-${input.id}`);
            const val = input.value.trim().toLowerCase();
            const target = input.dataset.ans.toLowerCase();
            const dist = getEditDistance(val, target);

            if (val === target) {
                fb.innerHTML = `<span class="status-green">✅ Perfect!</span>`;
                totalScore += 1;
            } else if (dist <= 2 || (target.length > 5 && dist <= 3)) {
                fb.innerHTML = `<span class="status-yellow">⚠️ Close!</span> <span class="correction-note">The correct answer is "${input.dataset.ans}"</span>`;
                totalScore += 0.5;
            } else {
                fb.innerHTML = `<span class="status-red">❌ Correct: "${input.dataset.ans}"</span>`;
            }
        });
        updateDisplayScore();
    }

    function testSpeech(word, fbId) {
        const rec = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
        rec.lang = 'en-US'; rec.start();
        document.getElementById(fbId).innerText = "Listening...";
        rec.onresult = (e) => {
            const heard = e.results[0][0].transcript.toLowerCase();
            const target = word.toLowerCase();
            const dist = getEditDistance(heard, target);
            const fb = document.getElementById(fbId);

            if (heard.includes(target)) {
                fb.innerHTML = `<span class="status-green">✅ Perfect!</span>`;
                totalScore += 1;
            } else if (dist <= 3) {
                fb.innerHTML = `<span class="status-yellow">⚠️ Almost! You said "${heard}"</span> <span class="correction-note">Correct word: "${word}"</span>`;
                totalScore += 0.5;
            } else {
                fb.innerHTML = `<span class="status-red">❌ Heard "${heard}". Word: "${word}"</span>`;
            }
            updateDisplayScore();
        };
    }

    function updateDisplayScore() {
        document.getElementById('score-val').innerText = Math.floor(totalScore);
    }

    window.onload = () => {
        if ('speechSynthesis' in window) {
            window.speechSynthesis.onvoiceschanged = init;
            if (window.speechSynthesis.getVoices().length > 0) init();
        } else { init(); }
    };
</script>
</body>
</html>
