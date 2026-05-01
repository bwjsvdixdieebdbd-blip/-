<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>حياة الشوارع والمال</title>
    <style>
        :root { --main-bg: #0f0f0f; --accent: #27ae60; --danger: #c0392b; }
        body { font-family: 'Cairo', sans-serif; background: var(--main-bg); color: #e0e0e0; margin: 0; padding: 15px; }
        .header { background: #1e1e1e; padding: 15px; border-radius: 12px; border-bottom: 3px solid var(--accent); margin-bottom: 15px; font-size: 0.9em; }
        .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .stat-box { background: #2a2a2a; padding: 8px; border-radius: 8px; text-align: center; }
        .game-screen { background: #181818; padding: 20px; border-radius: 15px; box-shadow: 0 10px 20px rgba(0,0,0,0.5); min-height: 400px; }
        .location-tag { color: var(--accent); font-weight: bold; text-transform: uppercase; margin-bottom: 10px; display: block; }
        .story-p { line-height: 1.8; font-size: 1.1em; margin-bottom: 25px; }
        .btn { display: block; width: 100%; padding: 15px; margin: 10px 0; border: none; border-radius: 8px; background: #333; color: white; font-size: 16px; cursor: pointer; transition: 0.3s; text-align: right; }
        .btn-trade { border-right: 5px solid #2ecc71; }
        .btn-gang { border-right: 5px solid #e74c3c; }
        .btn:active { transform: scale(0.98); background: #444; }
        .money-up { color: #2ecc71; font-weight: bold; }
        .danger-text { color: #e74c3c; }
    </style>
</head>
<body>

    <div class="header">
        <div class="stats-grid">
            <div class="stat-box">💰 المال: <span id="money">50</span>$</div>
            <div class="stat-box">⭐ السمعة: <span id="rep">0</span></div>
            <div class="stat-box">📦 بضائع: <span id="inv">0</span></div>
            <div class="stat-box">⚠️ خطر: <span id="risk">0</span>%</div>
        </div>
    </div>

    <div class="game-screen" id="screen">
        <span class="location-tag">📍 وسط المدينة - البداية</span>
        <p class="story-p" id="text">أنت تقف في الميدان الكبير. على اليمين يقع "حي المال والأعمال" حيث التجارة والسياسة، وعلى اليسار "المنطقة الصناعية القديمة" حيث القوة تفرضها العصابات.</p>
        <div id="options">
            <button class="btn btn-trade" onclick="goToTrade()">الذهاب لحي التجارة (طريق الثراء)</button>
            <button class="btn btn-gang" onclick="goToGang()">الذهاب للحي القديم (طريق العصابات)</button>
        </div>
    </div>

<script>
    let player = { money: 50, rep: 0, inv: 0, risk: 0 };

    function updateUI() {
        document.getElementById('money').innerText = player.money;
        document.getElementById('rep').innerText = player.rep;
        document.getElementById('inv').innerText = player.inv;
        document.getElementById('risk').innerText = player.risk;
        if(player.money <= 0 && player.inv == 0) {
            gameOver("إفلاس تام! لم تعد تملك شيئاً في هذه المدينة.");
        }
    }

    // --- مسار التجارة ---
    function goToTrade() {
        renderScene("📍 سوق الجملة", 
            "هنا يمكنك البدء كتاجر صغير. هل تشتري بضاعة مهربة بسعر رخيص أم بضاعة قانونية؟", [
            { t: "شراء بضاعة قانونية (-40$)", c: () => { 
                if(player.money >= 40) { player.money-=40; player.inv+=2; player.rep+=5; updateUI(); goToMarket(); }
            }},
            { t: "شراء بضاعة مهربة (-20$)", c: () => { 
                if(player.money >= 20) { player.money-=20; player.inv+=3; player.risk+=15; updateUI(); goToMarket(); }
            }}
        ]);
    }

    function goToMarket() {
        renderScene("📍 ساحة البيع", "لديك بضاعة الآن. هل تبيعها في السوق الرسمي ببطء، أم في السوق السوداء فوراً؟", [
            { t: "بيع رسمي (+60$، خفض خطر)", c: () => { player.money+=60; player.inv=0; player.risk-=5; updateUI(); goToTrade(); }},
            { t: "بيع في السوق السوداء (+100$)", c: () => { player.money+=100; player.inv=0; player.risk+=10; updateUI(); goToTrade(); }}
        ]);
    }

    // --- مسار العصابات ---
    function goToGang() {
        renderScene("📍 زقاق المهجورين", "زعيم محلي يراقبك.. 'تبدو جائعاً للقوة يا فتى'. يعرض عليك حماية شحنة مشبوهة.", [
            { t: "حماية الشحنة (خطر عالي، +150$)", c: () => { 
                if(Math.random() > 0.4) { player.money+=150; player.rep+=20; player.risk+=30; updateUI(); goToGang(); }
                else { player.money-=30; player.risk+=50; alert("داهمتكم الشرطة! هربت بصعوبة وخسرت مالك."); updateUI(); goToGang(); }
            }},
            { t: "سرقة بائع متجول (+30$، -10 سمعة)", c: () => { player.money+=30; player.rep-=10; player.risk+=5; updateUI(); goToGang(); }}
        ]);
    }

    function renderScene(loc, txt, opts) {
        document.getElementById('screen').innerHTML = `
            <span class="location-tag">${loc}</span>
            <p class="story-p">${txt}</p>
            <div id="options"></div>
        `;
        opts.forEach(o => {
            let b = document.createElement('button');
            b.className = 'btn';
            b.innerText = o.t;
            b.onclick = o.c;
            document.getElementById('options').appendChild(b);
        });
    }

    function gameOver(msg) {
        document.getElementById('screen').innerHTML = `<h2>انتهت اللعبة</h2><p>${msg}</p><button class="btn" onclick="location.reload()">محاولة مرة أخرى</button>`;
    }
</script>
</body>
</html>
