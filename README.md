<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Taaltrainer</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { 
            font-family: ui-sans-serif, system-ui, -apple-system, sans-serif;
            background-color: #F8FAFC;
            color: #000000;
        }

        h1, h2, h3, p, span, button {
            color: #000000 !important;
        }

        .card-contrast {
            background-color: #FFFFFF;
            border: 3px solid #000000;
            box-shadow: 5px 5px 0px #000000;
        }

        .option-card {
            background-color: #FFFFFF;
            border: 3px solid #000000;
            transition: all 0.1s;
            cursor: pointer;
            -webkit-tap-highlight-color: transparent;
            box-shadow: 4px 4px 0px #000000;
        }

        .option-card:active {
            transform: translate(2px, 2px);
            box-shadow: 1px 1px 0px #000000;
        }

        .correct { 
            background-color: #22C55E !important; 
            box-shadow: none !important;
            transform: translate(2px, 2px);
        }
        .wrong { 
            background-color: #EF4444 !important; 
            box-shadow: none !important;
            transform: translate(2px, 2px);
        }

        .progress-bg {
            background-color: #FFFFFF;
            border: 3px solid #000000;
        }

        .locked-card {
            background-color: #E2E8F0;
            border: 3px solid #000000;
            opacity: 0.6;
            cursor: not-allowed;
        }

        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body class="p-0 m-0">

    <div id="app">
        <!-- Dashboard -->
        <div id="dashboard" class="max-w-md mx-auto p-6 pb-24">
            <header class="flex justify-between items-center mb-10 pt-6">
                <div class="card-contrast p-3 px-6 rounded-2xl">
                    <p class="text-[10px] font-black uppercase tracking-widest opacity-60">Totaal XP</p>
                    <p id="display-xp" class="text-3xl font-black">0</p>
                </div>
            </header>

            <!-- Niveaus -->
            <div class="flex gap-2 overflow-x-auto no-scrollbar mb-10 pb-2">
                <button onclick="setCategory('A1')" id="nav-A1" class="px-6 py-2 border-2 border-black rounded-full font-black uppercase text-sm bg-yellow-400 shadow-[2px_2px_0px_#000]">A1</button>
                <button onclick="setCategory('A2')" id="nav-A2" class="px-6 py-2 border-2 border-black rounded-full font-black uppercase text-sm bg-white">A2</button>
                <button onclick="setCategory('B1')" id="nav-B1" class="px-6 py-2 border-2 border-black rounded-full font-black uppercase text-sm bg-white">B1</button>
                <button onclick="setCategory('B2')" id="nav-B2" class="px-6 py-2 border-2 border-black rounded-full font-black uppercase text-sm bg-white">B2</button>
                <button onclick="setCategory('C1')" id="nav-C1" class="px-6 py-2 border-2 border-black rounded-full font-black uppercase text-sm bg-white">C1</button>
                <button onclick="setCategory('C2')" id="nav-C2" class="px-6 py-2 border-2 border-black rounded-full font-black uppercase text-sm bg-white">C2</button>
            </div>
            
            <div id="levels-list" class="space-y-6"></div>

            <!-- Support -->
            <div class="mt-16 pt-8 border-t-4 border-black">
                <div class="card-contrast rounded-3xl p-6 mb-20">
                    <h2 class="text-xl font-black uppercase mb-4 text-center">Contact</h2>
                    <input type="text" id="contact-name" placeholder="NAAM" class="w-full mb-3 p-4 border-2 border-black rounded-xl font-bold outline-none focus:bg-slate-50">
                    <textarea id="contact-msg" rows="3" placeholder="BERICHT..." class="w-full mb-4 p-4 border-2 border-black rounded-xl font-bold outline-none focus:bg-slate-50"></textarea>
                    <button onclick="sendEmail()" class="w-full bg-black text-white p-4 rounded-xl font-black uppercase tracking-widest active:bg-gray-800 transition-colors">Verzenden</button>
                </div>
            </div>
        </div>

        <!-- Quiz -->
        <div id="quiz-overlay" class="hidden fixed inset-0 bg-white z-50 flex flex-col">
            <div class="p-4 flex items-center justify-between border-b-4 border-black">
                <button onclick="confirmExit()" class="text-3xl font-black p-2">✕</button>
                <div class="flex-1 mx-4 progress-bg h-6 rounded-full overflow-hidden">
                    <div id="progress-bar" class="bg-blue-500 h-full transition-all" style="width: 0%"></div>
                </div>
                <div id="quiz-counter" class="text-sm font-black w-24 text-right italic">1 / 100</div>
            </div>
            
            <div class="flex-1 flex flex-col p-6 items-center justify-center bg-slate-50">
                <div class="w-full max-w-sm">
                    <p id="q-label" class="text-[10px] font-black uppercase mb-2 opacity-40 tracking-widest"></p>
                    <h2 id="q-text" class="text-2xl font-black mb-10 leading-tight"></h2>
                    <div id="options-container" class="space-y-4"></div>
                </div>
            </div>
        </div>

        <!-- Resultaten -->
        <div id="result-screen" class="hidden fixed inset-0 z-[60] flex flex-col items-center justify-center p-8 text-center border-[12px] border-black transition-colors duration-500">
            <div id="result-emoji" class="text-8xl mb-4"></div>
            <h2 id="result-title" class="text-4xl font-black uppercase mb-2"></h2>
            <p id="result-msg" class="font-bold mb-8 text-lg px-4"></p>
            <div class="bg-white border-4 border-black p-6 rounded-3xl mb-12 shadow-[8px_8px_0px_#000] min-w-[240px]">
                <p class="text-[10px] font-black uppercase mb-1 tracking-tighter opacity-50">Resultaat</p>
                <p id="result-score" class="text-6xl font-black">0/100</p>
                <p class="text-[10px] font-black uppercase mt-4 opacity-70 border-t border-black/10 pt-2">80 punten nodig voor volgend level</p>
            </div>
            <button onclick="returnToDashboard()" class="w-full max-w-xs py-5 rounded-2xl bg-black text-white font-black uppercase text-xl active:scale-95 shadow-[4px_4px_0px_#444]">Doorgaan</button>
        </div>
    </div>

    <script>
        const CATS = ["A1", "A2", "B1", "B2", "C1", "C2"];
        const THEMES = {
            "A1": "bg-emerald-400", "A2": "bg-sky-400", "B1": "bg-indigo-400", 
            "B2": "bg-amber-500", "C1": "bg-rose-500", "C2": "bg-yellow-400"
        };

        const VOCAB_GROUPS = {
            "greeting": [
                {p: "Olá", n: "Hallo"}, {p: "Bom dia", n: "Goedemorgen"}, {p: "Boa tarde", n: "Goedemiddag"},
                {p: "Boa noite", n: "Goedenavond"}, {p: "Até logo", n: "Tot ziens"}, {p: "Adeus", n: "Vaarwel"}
            ],
            "basic": [
                {p: "Obrigado", n: "Bedankt"}, {p: "Por favor", n: "Alstublieft"}, {p: "Sim", n: "Ja"},
                {p: "Não", n: "Nee"}, {p: "Com licença", n: "Excuseer mij"}, {p: "Desculpe", n: "Sorry"}
            ],
            "food": [
                {p: "A água", n: "Het water"}, {p: "O pão", n: "Het brood"}, {p: "A maçã", n: "De appel"},
                {p: "O café", n: "De koffie"}, {p: "Tenho fome", n: "Ik heb honger"}, {p: "Tenho sede", n: "Ik heb dorst"}
            ],
            "family": [
                {p: "A mãe", n: "De moeder"}, {p: "O pai", n: "De vader"}, {p: "O irmão", n: "De broer"},
                {p: "A irmã", n: "De zus"}, {p: "O filho", n: "De zoon"}, {p: "A filha", n: "De dochter"}
            ],
            "place": [
                {p: "A casa", n: "Het huis"}, {p: "Onde fica?", n: "Waar is het?"}, {p: "A rua", n: "De straat"},
                {p: "A cidade", n: "De stad"}, {p: "O país", n: "Het land"}, {p: "A escola", n: "De school"}
            ],
            "time": [
                {p: "Ontem", n: "Gisteren"}, {p: "Hoje", n: "Vandaag"}, {p: "Amanhã", n: "Morgen"},
                {p: "A semana", n: "De week"}, {p: "O mês", n: "De maand"}, {p: "O ano", n: "Het jaar"}
            ],
            "abstract": [
                {p: "Desenvolvimento", n: "Ontwikkeling"}, {p: "Consequência", n: "Gevolg"}, {p: "Oportunidade", n: "Kans"},
                {p: "Desafiador", n: "Uitdagend"}, {p: "Inexorável", n: "Onverbiddelijk"}, {p: "Efêmero", n: "Vluchtig"}
            ]
        };

        const CAT_TO_GROUPS = {
            "A1": ["greeting", "basic", "food"],
            "A2": ["greeting", "food", "family", "place"],
            "B1": ["place", "time", "family"],
            "B2": ["time", "abstract"],
            "C1": ["abstract"],
            "C2": ["abstract"]
        };

        let currentCat = "A1";
        let state = JSON.parse(localStorage.getItem('trainer_clean_v1')) || {
            xp: 0,
            unlocked: { "A1-1": true },
            bestScores: {}
        };

        let activeQuiz = null;

        function getLevelContent(cat) {
            const groups = CAT_TO_GROUPS[cat];
            const data = [];
            const allWordsInCat = groups.flatMap(g => VOCAB_GROUPS[g]);
            
            // Genereer 100 vragen zonder technische nummers
            for (let i = 0; i < 100; i++) {
                const wordObj = allWordsInCat[i % allWordsInCat.length];
                const groupName = groups.find(g => VOCAB_GROUPS[g].includes(wordObj));
                
                data.push({
                    p: wordObj.p,
                    n: wordObj.n,
                    g: groupName
                });
            }
            return data;
        }

        function setCategory(cat) {
            currentCat = cat;
            CATS.forEach(c => {
                const el = document.getElementById(`nav-${c}`);
                if (c === cat) {
                    el.className = "px-6 py-2 border-2 border-black rounded-full font-black uppercase text-sm bg-yellow-400 shadow-[2px_2px_0px_#000]";
                } else {
                    el.className = "px-6 py-2 border-2 border-black rounded-full font-black uppercase text-sm bg-white";
                }
            });
            renderDashboard();
        }

        function renderDashboard() {
            document.getElementById('display-xp').innerText = state.xp;
            const list = document.getElementById('levels-list');
            list.innerHTML = '';

            for (let i = 1; i <= 10; i++) {
                const id = `${currentCat}-${i}`;
                const unlocked = state.unlocked[id];
                const score = state.bestScores[id] || 0;
                
                const card = document.createElement('div');
                if (unlocked) {
                    card.className = "p-6 rounded-3xl border-4 border-black flex items-center gap-6 bg-white cursor-pointer shadow-[6px_6px_0px_#000] active:translate-x-1 active:translate-y-1 active:shadow-none transition-all";
                    card.onclick = () => startQuiz(id);
                } else {
                    card.className = "p-6 rounded-3xl border-4 border-black flex items-center gap-6 locked-card";
                }

                card.innerHTML = `
                    <div class="w-16 h-16 rounded-2xl flex items-center justify-center text-2xl border-4 border-black ${unlocked ? THEMES[currentCat] : 'bg-gray-300'}">
                        ${score >= 80 ? '✓' : i}
                    </div>
                    <div class="flex-1">
                        <div class="flex justify-between items-center mb-2">
                            <h3 class="font-black text-xl uppercase tracking-tighter">Level ${i}</h3>
                            <span class="font-black text-sm">${score}/100</span>
                        </div>
                        <div class="w-full bg-gray-100 border-2 border-black h-4 rounded-full overflow-hidden">
                            <div class="bg-blue-600 h-full transition-all duration-700" style="width: ${score}%"></div>
                        </div>
                    </div>
                `;
                list.appendChild(card);
            }
        }

        function startQuiz(id) {
            const [cat] = id.split('-');
            const raw = getLevelContent(cat);
            
            const questions = raw.map(item => {
                // Slimme afleiders uit dezelfde groep
                let distractors = VOCAB_GROUPS[item.g]
                    .filter(x => x.n !== item.n)
                    .sort(() => 0.5 - Math.random())
                    .slice(0, 3)
                    .map(x => x.n);
                
                // Vul aan indien nodig
                if (distractors.length < 3) {
                    const extra = CAT_TO_GROUPS[cat]
                        .flatMap(g => VOCAB_GROUPS[g])
                        .filter(x => x.n !== item.n && !distractors.includes(x.n))
                        .sort(() => 0.5 - Math.random())
                        .slice(0, 3 - distractors.length)
                        .map(x => x.n);
                    distractors = [...distractors, ...extra];
                }
                
                const options = [item.n, ...distractors].sort(() => 0.5 - Math.random());
                return {
                    q: `Vertaal: "${item.p}"`,
                    a: options,
                    c: options.indexOf(item.n)
                };
            });

            activeQuiz = { id, questions, index: 0, score: 0 };
            document.getElementById('quiz-overlay').classList.remove('hidden');
            renderQuestion();
        }

        function renderQuestion() {
            const q = activeQuiz.questions[activeQuiz.index];
            document.getElementById('q-label').innerText = `${activeQuiz.id} • Vraag ${activeQuiz.index + 1}`;
            document.getElementById('q-text').innerText = q.q;
            document.getElementById('quiz-counter').innerText = `${activeQuiz.index + 1} / 100`;
            document.getElementById('progress-bar').style.width = `${((activeQuiz.index) / 100) * 100}%`;

            const container = document.getElementById('options-container');
            container.innerHTML = '';

            q.a.forEach((opt, i) => {
                const btn = document.createElement('button');
                btn.className = "option-card w-full p-5 rounded-2xl font-black text-lg text-left leading-tight";
                btn.innerText = opt;
                btn.onclick = () => {
                    const btns = document.querySelectorAll('.option-card');
                    btns.forEach(b => b.onclick = null);
                    
                    if (i === q.c) {
                        btn.classList.add('correct');
                        activeQuiz.score++;
                        setTimeout(next, 250);
                    } else {
                        btn.classList.add('wrong');
                        btns[q.c].classList.add('correct');
                        setTimeout(next, 800);
                    }
                };
                container.appendChild(btn);
            });
        }

        function next() {
            activeQuiz.index++;
            if (activeQuiz.index < 100) {
                renderQuestion();
            } else {
                finishQuiz();
            }
        }

        function finishQuiz() {
            const finalScore = activeQuiz.score;
            const success = finalScore >= 80;
            const screen = document.getElementById('result-screen');
            
            state.bestScores[activeQuiz.id] = Math.max(state.bestScores[activeQuiz.id] || 0, finalScore);
            state.xp += finalScore;

            if (success) {
                screen.className = "fixed inset-0 z-[60] flex flex-col items-center justify-center p-8 text-center border-[12px] border-black bg-green-400";
                document.getElementById('result-emoji').innerText = "⭐";
                document.getElementById('result-title').innerText = "Geslaagd";
                document.getElementById('result-msg').innerText = "Je hebt het volgende niveau vrijgespeeld.";
                unlockNext(activeQuiz.id);
            } else {
                screen.className = "fixed inset-0 z-[60] flex flex-col items-center justify-center p-8 text-center border-[12px] border-black bg-red-400";
                document.getElementById('result-emoji').innerText = "!";
                document.getElementById('result-title').innerText = "Gezakt";
                document.getElementById('result-msg').innerText = "Minimaal 80 punten nodig.";
            }

            document.getElementById('result-score').innerText = `${finalScore}/100`;
            document.getElementById('quiz-overlay').classList.add('hidden');
            screen.classList.remove('hidden');
            save();
        }

        function unlockNext(id) {
            const [cat, num] = id.split('-');
            const n = parseInt(num);
            if (n < 10) {
                state.unlocked[`${cat}-${n+1}`] = true;
            } else {
                const idx = CATS.indexOf(cat);
                if (idx < CATS.length - 1) {
                    state.unlocked[`${CATS[idx+1]}-1`] = true;
                }
            }
        }

        function confirmExit() {
            if (confirm("Stopzetten? Geen opslag.")) {
                document.getElementById('quiz-overlay').classList.add('hidden');
            }
        }

        function returnToDashboard() {
            document.getElementById('result-screen').classList.add('hidden');
            renderDashboard();
        }

        function save() {
            localStorage.setItem('trainer_clean_v1', JSON.stringify(state));
        }

        function sendEmail() {
            const n = document.getElementById('contact-name').value;
            const m = document.getElementById('contact-msg').value;
            if(!n || !m) return;
            window.location.href = `mailto:plettingrobin@gmail.com?subject=Feedback&body=Naam: ${n}%0A%0A${m}`;
        }

        renderDashboard();
    </script>
</body>
</html>

