[聯簿-缺項貼.html](https://github.com/user-attachments/files/24345062/-.html)
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>聯簿-缺項貼</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* 字體設定 */
        body { font-family: "Microsoft JhengHei", "Heiti TC", sans-serif; }

        /* 統計表用的座號框 */
        .stat-box {
            width: 3rem; height: 3rem;
            display: flex; align-items: center; justify-content: center;
            font-weight: 700; border-width: 1px; border-radius: 0.5rem;
            font-size: 1.5rem; line-height: 1; background: white;
        }

        /* 列印專用設定 */
        @media print {
            .no-print { display: none !important; }
            body { background: white; padding: 0; margin: 0; }
            * { -webkit-print-color-adjust: exact !important; print-color-adjust: exact !important; }
            
            #resultSection { display: block !important; }
            
            /* 強制三欄排版 */
            .print-grid-3 {
                display: grid !important;
                grid-template-columns: repeat(3, 1fr) !important;
                gap: 1rem !important;
            }
            
            .break-inside-avoid { page-break-inside: avoid; break-inside: avoid; }
            #modeTitle { display: none !important; } /* 列印時連標題都省了，更省墨水 */
        }
    </style>
</head>
<body class="bg-slate-50 min-h-screen flex flex-col items-center py-8 px-4">

    <div class="w-full max-w-7xl mb-6 flex items-center justify-between no-print">
        <a href="index.html" class="flex items-center gap-2 text-slate-500 hover:text-blue-600 font-bold px-4 py-2 rounded-xl hover:bg-white transition-colors">
            ⬅️ 回首頁
        </a>
        <h1 class="text-3xl font-black text-slate-700">📋 聯簿-缺項貼</h1>
        <div class="w-24"></div> 
    </div>

    <div class="w-full max-w-7xl bg-white rounded-2xl p-6 shadow-sm border border-slate-200 mb-8 no-print">
        <div class="mb-3 text-slate-500 font-bold">
            請貼上清單 (支援格式：事項(3.5.))
        </div>
        <textarea id="inputArea" 
            class="w-full h-48 p-4 border-2 border-slate-100 rounded-xl focus:ring-2 focus:ring-blue-500 outline-none text-slate-700 text-lg font-mono leading-relaxed"
            placeholder="請貼上如：
1. ()確認比賽日午餐+接送(3.5.)
2. ()練語競25分鐘(3.5.)
3. ()背練台詞+道具(1.12.15.18.21.22.23.)"
        ></textarea>
        
        <div class="grid grid-cols-2 gap-4 mt-4">
            <button onclick="generateStats()" 
                class="bg-blue-600 hover:bg-blue-700 text-white font-black text-2xl py-4 rounded-xl shadow-lg transition-transform active:scale-98 flex items-center justify-center gap-2">
                📊 名單
            </button>
            
            <button onclick="generateCards()" 
                class="bg-slate-800 hover:bg-slate-900 text-white font-black text-2xl py-4 rounded-xl shadow-lg transition-transform active:scale-98 flex items-center justify-center gap-2">
                📝 貼條
            </button>
        </div>
    </div>

    <div id="resultSection" class="w-full max-w-7xl hidden">
        <div id="modeTitle" class="text-2xl font-bold text-slate-700 mb-4 pl-2 border-l-8 border-slate-800">
            結果
        </div>

        <div id="resultContent"></div>

        <div class="mt-10 flex justify-center gap-4 no-print pb-10">
            <button onclick="window.print()" class="bg-white text-slate-800 border-2 border-slate-200 font-bold px-10 py-3 rounded-full hover:bg-slate-50 shadow-lg flex items-center gap-2">
                🖨️ 列印 / 存圖
            </button>
        </div>
    </div>

    <script>
        const CLASS_SIZE = 24;

        // ★ 解析引擎 ★
        function parseInput() {
            let text = document.getElementById('inputArea').value;
            text = text.replace(/（/g, '(').replace(/）/g, ')'); // 全形轉半形
            
            const lines = text.split('\n');
            const students = {}; 
            for (let i = 1; i <= CLASS_SIZE; i++) students[i] = [];

            let foundData = false;

            lines.forEach(line => {
                let str = line.trim();
                if (!str) return;

                const matchSeat = str.match(/\(([\d\.]+)\)$/);
                
                if (matchSeat) {
                    const seatString = matchSeat[1]; 
                    let content = str.substring(0, matchSeat.index).trim();
                    
                    // 清理編號
                    content = content.replace(/^\d+\.\s*\(\)/, '');
                    content = content.replace(/^\d+\.\s*/, '');
                    content = content.replace(/^\(\)/, ''); 
                    content = content.trim();

                    const seats = seatString.split('.').filter(s => s.trim() !== '').map(Number);
                    
                    seats.forEach(seat => {
                        if (seat >= 1 && seat <= CLASS_SIZE) {
                            students[seat].push(content);
                            foundData = true;
                        }
                    });
                }
            });

            if (!foundData && text.trim().length > 0) {
                alert("⚠️ 讀取不到座號！請確認格式，例如：事項(3.5.)");
            }
            return students;
        }

        // 10色分級
        const colors = {
            0: { bg: 'bg-emerald-50',  tag: 'bg-emerald-100 text-emerald-700', border: 'border-emerald-400', text: 'text-emerald-700' },
            1: { bg: 'bg-cyan-50',     tag: 'bg-cyan-100 text-cyan-700',     border: 'border-cyan-400',    text: 'text-cyan-700' },
            2: { bg: 'bg-sky-50',      tag: 'bg-sky-100 text-sky-700',       border: 'border-sky-400',     text: 'text-sky-700' },
            3: { bg: 'bg-blue-50',     tag: 'bg-blue-100 text-blue-700',     border: 'border-blue-400',    text: 'text-blue-700' },
            4: { bg: 'bg-indigo-50',   tag: 'bg-indigo-100 text-indigo-700', border: 'border-indigo-400',  text: 'text-indigo-700' },
            5: { bg: 'bg-violet-50',   tag: 'bg-violet-100 text-violet-700', border: 'border-violet-400',  text: 'text-violet-700' },
            6: { bg: 'bg-purple-50',   tag: 'bg-purple-100 text-purple-700', border: 'border-purple-400',  text: 'text-purple-700' },
            7: { bg: 'bg-fuchsia-50',  tag: 'bg-fuchsia-100 text-fuchsia-700', border: 'border-fuchsia-400', text: 'text-fuchsia-700' },
            8: { bg: 'bg-pink-50',     tag: 'bg-pink-100 text-pink-700',     border: 'border-pink-400',    text: 'text-pink-700' },
            9: { bg: 'bg-rose-50',     tag: 'bg-rose-100 text-rose-700',     border: 'border-rose-400',    text: 'text-rose-700' },
            10: { bg: 'bg-red-50',     tag: 'bg-red-100 text-red-700',       border: 'border-red-600',     text: 'text-red-700' }
        };

        // 功能一：名單 (統計)
        function generateStats() {
            const students = parseInput();
            const groups = {};
            
            for (let i = 1; i <= CLASS_SIZE; i++) {
                const count = students[i].length;
                if (!groups[count]) groups[count] = [];
                groups[count].push(i);
            }

            const counts = Object.keys(groups).sort((a, b) => b - a);
            
            let html = '<div class="flex flex-col gap-0 border border-slate-200 rounded-xl overflow-hidden bg-white shadow-sm">';
            
            counts.forEach(count => {
                const seats = groups[count];
                seats.sort((a, b) => a - b);
                
                const safeCount = count > 10 ? 10 : count;
                const style = colors[safeCount];
                
                // 極簡文字
                const labelText = (count == 0) ? '缺 0' : `缺 ${count}`;
                const subText = `(${seats.length}人)`;

                html += `
                    <div class="flex flex-row items-stretch border-b border-slate-100 last:border-0 ${style.bg}">
                        <div class="w-32 md:w-40 shrink-0 flex flex-col justify-center items-center p-3 border-r border-slate-100 bg-white/50">
                            <span class="px-3 py-1 rounded-lg text-lg font-black ${style.tag} mb-1">
                                ${labelText}
                            </span>
                            <span class="text-sm font-bold text-slate-500">${subText}</span>
                        </div>
                        <div class="flex-1 p-4 flex flex-wrap items-center gap-3">
                            ${seats.map(seat => `
                                <div class="stat-box ${style.border} ${style.text}">
                                    ${seat}
                                </div>
                            `).join('')}
                        </div>
                    </div>
                `;
            });
            html += '</div>';

            document.getElementById('modeTitle').innerText = '名單';
            document.getElementById('resultContent').className = ''; 
            document.getElementById('resultContent').innerHTML = html;
            document.getElementById('resultSection').classList.remove('hidden');
        }

        // 功能二：貼條 (卡片) - ★★★ 排序修正版 ★★★
        function generateCards() {
            const students = parseInput();
            const today = new Date();
            const dateStr = `${today.getMonth() + 1}/${today.getDate()}`;
            
            // 1. 收集所有有缺項的學生資料
            let cardList = [];
            for (let i = 1; i <= CLASS_SIZE; i++) {
                if (students[i].length > 0) {
                    cardList.push({
                        seat: i,
                        items: students[i],
                        count: students[i].length
                    });
                }
            }

            // 2. ★★★ 排序邏輯：缺越多排越前面 ★★★
            cardList.sort((a, b) => {
                // 先比數量 (大 -> 小)
                if (b.count !== a.count) {
                    return b.count - a.count;
                }
                // 數量一樣，比座號 (小 -> 大)
                return a.seat - b.seat;
            });

            let html = '';
            
            // 3. 生成卡片
            if (cardList.length > 0) {
                cardList.forEach(student => {
                    const count = student.count;
                    const items = student.items;
                    const i = student.seat;

                    const safeCount = count > 10 ? 10 : count;
                    const style = colors[safeCount];
                    const detailText = items.join('、');

                    html += `
                        <div class="break-inside-avoid relative w-full bg-white rounded-xl border-2 ${style.border} shadow-sm overflow-hidden flex flex-col">
                            <div class="flex items-center justify-between p-3 pb-2">
                                <div class="flex items-baseline gap-2">
                                    <span class="text-4xl font-black ${style.text}">${i}</span>
                                    <span class="text-base font-bold text-slate-500">(${count}項)</span>
                                </div>
                                <div class="text-xl font-bold ${style.text}">
                                    ${dateStr}
                                </div>
                            </div>
                            <div class="h-px w-full bg-slate-200 border-b border-${style.border.split('-')[1]}-300 mx-auto w-[95%]"></div>
                            <div class="flex-1 p-3 pt-2">
                                <div class="text-base font-bold text-slate-800 leading-snug text-justify">
                                    ${detailText}
                                </div>
                            </div>
                        </div>
                    `;
                });
            } else {
                html = `<div class="col-span-3 text-center text-slate-400 py-10 font-bold">沒有缺交資料！</div>`;
            }

            document.getElementById('modeTitle').innerText = '貼條';
            // 列印強制三欄
            document.getElementById('resultContent').className = 'grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 print-grid-3';
            document.getElementById('resultContent').innerHTML = html;
            document.getElementById('resultSection').classList.remove('hidden');
        }
    </script>
</body>
</html>
