<!DOCTYPE html>
<html lang="ka">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title> GOI'S CASSINO 2 💎</title>
    <style>
        :root {
            --gold: #FFD700;
            --bg: #0E0E1A;
            --panel: #1E1E2E;
            --text: #EEE;
            --accent: #FFB700;
            --jackpot: #FF1493;
        }
        
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Arial, sans-serif;
            font-weight: 600;
        }
        
        body {
            background: var(--bg);
            color: var(--text);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 1rem;
        }
        
        .slot-app {
            width: 100%;
            max-width: 400px;
        }
        
        .logo {
            text-align: center;
            font-size: 2rem;
            color: var(--gold);
            margin-bottom: 0.5rem;
        }
        
        .panel {
            display: flex;
            justify-content: space-between;
            background: var(--panel);
            padding: 12px 16px;
            border-radius: 8px;
            margin-bottom: 12px;
        }
        
        .panel div {
            text-align: center;
        }
        
        .label {
            font-size: 0.75rem;
            color: #BBB;
        }
        
        .val {
            font-size: 1.2rem;
            margin-top: 4px;
        }
        
        .val small {
            font-size: 0.7rem;
            margin-left: 4px;
        }
        
        .jackpot {
            background: var(--jackpot);
            padding: 4px 8px;
            border-radius: 4px;
            margin-top: 4px;
            font-weight: 800;
        }
        
        .grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            margin-bottom: 12px;
        }
        
        .cell {
            aspect-ratio: 1/1;
            background: #1B1B2F;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 2.5rem;
            border-radius: 6px;
            border: 2px solid transparent;
            transition: all 0.3s;
        }
        
        .cell.win {
            box-shadow: 0 0 12px var(--gold);
            border-color: var(--gold);
            animation: pulse 0.6s 2;
        }
        
        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.1); }
        }
        
        .controls {
            display: flex;
            justify-content: center;
            gap: 10px;
        }
        
        button {
            font-size: 1rem;
            padding: 10px 14px;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-weight: 600;
            transition: all 0.2s;
        }
        
        .bet-btn {
            background: var(--panel);
            color: var(--text);
            width: 44px;
            height: 44px;
        }
        
        button:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }
        
        #spin {
            background: var(--accent);
            color: var(--bg);
            flex: 1;
        }
        
        #message {
            height: 24px;
            text-align: center;
            margin-top: 8px;
            font-size: 1rem;
            color: var(--gold);
        }
    </style>
</head>
<body>
    <div class="slot-app" aria-live="polite">
        <div class="logo">3×3 VIP სლოტი</div>
        
        <div class="panel">
            <div>
                <div class="label">ბალანსი</div>
                <div class="val" id="balance">0 <small>₾</small></div>
            </div>
            <div>
                <div class="label">ფსონი</div>
                <div class="val" id="bet">0 <small>₾</small></div>
            </div>
            <div>
                <div class="label">მოგება</div>
                <div class="val" id="potential">0 <small>₾</small></div>
                <div class="jackpot" id="jackpot">ჯეკპოტი: 0 ₾</div>
            </div>
        </div>
        
        <div class="grid" id="grid"></div>
        
        <div class="controls">
            <button id="dec">−</button>
            <button id="spin">SPIN</button>
            <button id="inc">+</button>
        </div>
        
        <div id="message" role="status"></div>
    </div>

    <script>
        // კონფიგურაცია
        const cfg = {
            symbols: ['🍒', '🍋', '🍊', '🍇', '🍉', '7', '💎'],
            min: 5,       // ყველაზე დაბალი ფსონი (შეცვლილი)
            max: 500,     // ყველაზე მაღალი ფსონი (შეცვლილი)
            step: 5,      // ფსონის ცვლილების საფეხური (შეცვლილი)
            initBal: 1000, // საწყისი ბალანსი (შეცვლილი)
            lines: [
                [0, 1, 2], [3, 4, 5], [6, 7, 8], // ჰორიზონტალური
                [0, 3, 6], [1, 4, 7], [2, 5, 8], // ვერტიკალური
                [0, 4, 8], [2, 4, 6]              // დიაგონალური
            ],
            pay: { 
                '💎💎💎': 15,
                '777': 10,
                '🍒🍒🍒': 5,
                '🍋🍋🍋': 3,
                '🍊🍊🍊': 3,
                '🍇🍇🍇': 3,
                '🍉🍉🍉': 3,
                'default': 2
            },
            jackpotFee: 0.02 // 2% of each bet goes to jackpot
        };

        // მონაცემები
        let bal = cfg.initBal;
        let bet = cfg.min;
        let spinning = false;
        let jackpot = 0;

        // DOM ელემენტები
        const D = {
            balance: document.getElementById('balance'),
            bet: document.getElementById('bet'),
            potential: document.getElementById('potential'),
            grid: document.getElementById('grid'),
            message: document.getElementById('message'),
            dec: document.getElementById('dec'),
            inc: document.getElementById('inc'),
            spin: document.getElementById('spin'),
            jackpot: document.getElementById('jackpot')
        };

        // ინიციალიზაცია
        function init() {
            buildGrid();
            update();
            
            // მოვლენების დამატება
            D.dec.addEventListener('click', () => { 
                bet = Math.max(cfg.min, bet - cfg.step); 
                update(); 
            });
            
            D.inc.addEventListener('click', () => { 
                bet = Math.min(cfg.max, bet + cfg.step); 
                update(); 
            });
            
            D.spin.addEventListener('click', spin);
        }

        // გრიდის აგება
        function buildGrid() {
            D.grid.innerHTML = '';
            for (let i = 0; i < 9; i++) {
                const cell = document.createElement('div');
                cell.className = 'cell';
                cell.textContent = '?';
                D.grid.appendChild(cell);
            }
        }

        // ინტერფეისის განახლება
        function update() {
            D.balance.textContent = bal;
            D.bet.textContent = bet;
            D.potential.textContent = bet * Math.max(...Object.values(cfg.pay));
            D.jackpot.textContent = `ჯეკპოტი: ${jackpot.toFixed(2)} ₾`;
            D.spin.disabled = spinning || bal < bet;
            D.dec.disabled = spinning || bet <= cfg.min;
            D.inc.disabled = spinning || bet >= cfg.max;
        }

        // სპინის ფუნქციონალი
        async function spin() {
            if (spinning || bal < bet) return;
            
            spinning = true;
            bal -= bet;
            jackpot += bet * cfg.jackpotFee;
            update();
            
            D.message.textContent = 'სპინი...';
            
            // ანიმაცია
            const cells = [...D.grid.children];
            let spins = 0;
            const totalSpins = 15;
            
            const spinInterval = setInterval(() => {
                cells.forEach(cell => {
                    cell.textContent = cfg.symbols[Math.floor(Math.random() * cfg.symbols.length)];
                    cell.classList.remove('win');
                });
                
                if (++spins >= totalSpins) {
                    clearInterval(spinInterval);
                    finishSpin();
                }
            }, 100);
        }

        // სპინის დასრულება
        function finishSpin() {
            const result = [];
            const cells = [...D.grid.children];
            
            // შედეგის გენერირება
            for (let i = 0; i < 9; i++) {
                const symbol = cfg.symbols[Math.floor(Math.random() * cfg.symbols.length)];
                cells[i].textContent = symbol;
                result[i] = symbol;
            }
            
            // მოგების შემოწმება
            const winResult = checkWin(result);
            
            if (winResult.total > 0) {
                // მოგებული ხაზების მონიშვნა
                winResult.wins.forEach(win => {
                    win.line.forEach(idx => {
                        cells[idx].classList.add('win');
                    });
                });
                
                // ჯეკპოტის შემოწმება
                if (result.includes('💎💎💎')) {
                    winResult.total += jackpot;
                    jackpot = 0;
                    D.message.textContent = `ჯეკპოტი! მოიგე ${winResult.total}₾!`;
                } else {
                    D.message.textContent = `მოიგე ${winResult.total}₾!`;
                }
                
                bal += winResult.total;
            } else {
                D.message.textContent = 'სცადე ისევ!';
            }
            
            spinning = false;
            update();
        }

        // მოგების შემოწმება
        function checkWin(result) {
            let totalWin = 0;
            const wins = [];
            
            cfg.lines.forEach(line => {
                const [a, b, c] = line.map(idx => result[idx]);
                
                if (a === b && b === c) {
                    const combo = a + b + c;
                    const multiplier = cfg.pay[combo] || cfg.pay['default'];
                    const winAmount = bet * multiplier;
                    
                    totalWin += winAmount;
                    wins.push({
                        line,
                        combo,
                        multiplier,
                        amount: winAmount
                    });
                }
            });
            
            return { total: totalWin, wins };
        }

        // ინიციალიზაციის დაწყება
        init();
    </script>
</body>
</html>
