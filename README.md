<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🎨 الرسام الجامعي - أونلاين</title>
    <script src="https://cdn.ably.com/lib/ably.min-1.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Cairo:wght@400;700&display=swap');
        body { font-family: 'Cairo', sans-serif; text-align: center; background: #eef2f3; margin: 0; padding: 10px; color: #333; }
        .container { max-width: 500px; margin: auto; }
        canvas { background: white; border: 4px solid #2c3e50; touch-action: none; border-radius: 15px; box-shadow: 0 10px 20px rgba(0,0,0,0.1); width: 100%; max-width: 350px; height: 400px; }
        .controls { margin: 15px 0; display: flex; justify-content: center; gap: 10px; flex-wrap: wrap; }
        #word-display { font-size: 1.1rem; font-weight: bold; color: #d35400; background: white; padding: 10px; border-radius: 10px; margin-bottom: 10px; border: 2px dashed #d35400; }
        button { padding: 12px 18px; font-size: 14px; border: none; border-radius: 8px; cursor: pointer; color: white; font-weight: bold; transition: 0.3s; }
        .btn-start { background: #27ae60; }
        .btn-clear { background: #e74c3c; }
        #status { font-size: 0.8rem; color: #7f8c8d; margin-top: 5px; }
        .player-count { background: #3498db; color: white; padding: 5px 15px; border-radius: 20px; display: inline-block; margin-bottom: 10px; font-size: 0.9rem; }
    </style>
</head>
<body>

    <div class="container">
        <h2>🎨 الرسام الجامعي (أونلاين)</h2>
        <div class="player-count" id="player-count">جاري الاتصال...</div>
        <div id="word-display">دوس "ابدأ" والكل هيعرف الكلمة!</div>
        
        <canvas id="canvas" width="350" height="400"></canvas>

        <div class="controls">
            <button class="btn-start" onclick="sendNewWord()">كلمة جديدة للكل</button>
            <button class="btn-clear" onclick="sendClear()">مسح عند الكل</button>
        </div>

        <div id="status">أي حاجة بترسمها بتظهر عند صحابك دلوقتي!</div>
    </div>

    <script>
        // مفتاح تشغيل الخدمة الأونلاين (تجريبي)
        const ably = new Ably.Realtime('8O_m_A.nInM5g:Sizk6Tf0fS0l3u9fS99p2C2mJk0LpE-pLpWv_Y-9E8Y');
        const channel = ably.channels.get('drawing-game');

        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        const wordDisplay = document.getElementById('word-display');
        const playerCount = document.getElementById('player-count');
        let drawing = false;

        const words = ["سكشن", "فاينل", "الدكتورة", "شيت العملي", "كافيين", "الأتوبيس", "المدرج", "المحاضرة"];

        // تحديث عدد اللاعبين
        channel.presence.subscribe('enter', updateCount);
        channel.presence.subscribe('leave', updateCount);
        function updateCount() {
            channel.presence.get((err, members) => {
                playerCount.textContent = "لاعبين متصلين: " + (members ? members.length : 1);
            });
        }
        channel.presence.enter();

        // استقبال الرسم من الآخرين
        channel.subscribe('draw', (message) => {
            const data = message.data;
            if (data.type === 'start') {
                ctx.beginPath();
                ctx.moveTo(data.x, data.y);
            } else if (data.type === 'move') {
                drawOnCanvas(data.x, data.y);
            }
        });

        channel.subscribe('control', (message) => {
            if (message.data.action === 'clear') localClear();
            if (message.data.action === 'word') localNewWord(message.data.word);
        });

        function getPos(e) {
            const rect = canvas.getBoundingClientRect();
            let clientX, clientY;
            if (e.touches) {
                clientX = e.touches[0].clientX;
                clientY = e.touches[0].clientY;
            } else {
                clientX = e.clientX;
                clientY = e.clientY;
            }
            return { x: clientX - rect.left, y: clientY - rect.top };
        }

        function startDrawing(e) {
            drawing = true;
            const { x, y } = getPos(e);
            ctx.beginPath();
            ctx.moveTo(x, y);
            channel.publish('draw', { type: 'start', x, y });
        }

        function moveDrawing(e) {
            if (!drawing) return;
            const { x, y } = getPos(e);
            drawOnCanvas(x, y);
            channel.publish('draw', { type: 'move', x, y });
        }

        function stopDrawing() {
            drawing = false;
            ctx.beginPath();
        }

        function drawOnCanvas(x, y) {
            ctx.lineWidth = 4;
            ctx.lineCap = "round";
            ctx.strokeStyle = "#2c3e50";
            ctx.lineTo(x, y);
            ctx.stroke();
            ctx.beginPath();
            ctx.moveTo(x, y);
        }

        function sendNewWord() {
            const randomWord = words[Math.floor(Math.random() * words.length)];
            channel.publish('control', { action: 'word', word: randomWord });
        }

        function localNewWord(word) {
            wordDisplay.textContent = "ارسم: " + word;
            localClear();
        }

        function sendClear() {
            channel.publish('control', { action: 'clear' });
        }

        function localClear() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.beginPath();
        }

        canvas.addEventListener('touchstart', (e) => { startDrawing(e); e.preventDefault(); });
        canvas.addEventListener('touchmove', (e) => { moveDrawing(e); e.preventDefault(); });
        canvas.addEventListener('touchend', stopDrawing);
        canvas.addEventListener('mousedown', startDrawing);
        canvas.addEventListener('mousemove', moveDrawing);
        canvas.addEventListener('mouseup', stopDrawing);
    </script>
</body>
</html>
# Varnursing
