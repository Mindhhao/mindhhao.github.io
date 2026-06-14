# mindhhao.github.io
# 我们制作了一份NBA球员综合评分系统，评分的规则如下：
# 1. 若该球员获得NBA赛季总冠军并且作为前二核心+100积分，若作为首发则+30积分（若同时成立则取更高积分的选项，不重复计算）
# 2. 若该球员获得NBA赛季总冠军总亚军且作为前二核心20积分，若作为首发则+5积分（若同时成立则取更高积分的选项，不重复计算）
# 3. 若该球员获得过常规赛mvp +70积分
# 4. 若该球员获得过总决赛fmvp +50积分
# 5. 若该球员获得过一阵 +20积分、DPOY +20积分、二阵 +15积分、得分王+10积分、全明星+5积分、一防+5积分、三阵+5积分、二防+3积分、进入季后赛+1积分、每过季后赛一轮再+1积分
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>🏀 NBA巨星历史积分动态评测系统 (移动端优化版)</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            -webkit-tap-highlight-color: transparent;
        }
        .custom-scroll::-webkit-scrollbar { width: 4px; height: 4px; }
        .custom-scroll::-webkit-scrollbar-track { background: #f1f1f1; }
        .custom-scroll::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
        .safe-bottom { padding-bottom: calc(env(safe-area-inset-bottom) + 80px); }
    </style>
</head>
<body class="bg-slate-100 text-slate-800 font-sans antialiased min-h-screen flex flex-col">

    <header class="bg-gradient-to-r from-blue-900 to-indigo-950 text-white shadow-md sticky top-0 z-40 px-4 py-3 flex justify-between items-center shrink-0">
        <h1 class="text-base font-bold tracking-wider flex items-center gap-1.5">
            🏀 NBA巨星积分评测系统
        </h1>
        <button onclick="resetData()" class="text-xs bg-red-600/20 text-red-300 border border-red-500/30 px-2 py-1 rounded-md active:bg-red-600/40 transition">
            重置
        </button>
    </header>

    <div class="sticky top-[48px] z-40 bg-white border-b border-slate-200 shadow-sm flex shrink-0">
        <button id="tab-rank" onclick="switchTab('rank')" class="flex-1 py-3 text-center text-sm font-bold border-b-2 border-blue-600 text-blue-600 transition-all">
            🏆 积分排行榜
        </button>
        <button id="tab-detail" onclick="switchTab('detail')" class="flex-1 py-3 text-center text-sm font-medium border-b-2 border-transparent text-slate-500 transition-all flex justify-center items-center gap-1">
            📊 维度与数据检验
            <span id="detail-badge" class="bg-amber-500 text-white text-[10px] px-1.5 py-0.5 rounded-full font-sans hidden">已选</span>
        </button>
    </div>

    <main class="flex-1 overflow-hidden relative">
        
        <div id="panel-rank" class="absolute inset-0 overflow-y-auto px-3 py-4 space-y-4">
            <div class="bg-white rounded-xl shadow-sm border border-slate-200/60 overflow-hidden">
                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse">
                        <thead class="bg-slate-50 border-b border-slate-200 text-slate-600 text-xs font-bold">
                            <tr>
                                <th class="p-3 text-center w-12">排名</th>
                                <th class="p-3">球员</th>
                                <th class="p-3 text-right w-20">总积分</th>
                                <th class="p-3 text-center w-16">操作</th>
                            </tr>
                        </thead>
                        <tbody id="rankTableBody" class="divide-y divide-slate-100 text-xs">
                            </tbody>
                    </table>
                </div>
            </div>
            <div class="h-16"></div> </div>

        <div id="panel-detail" class="absolute inset-0 overflow-y-auto px-3 py-4 space-y-4 hidden">
            <div class="bg-gradient-to-br from-blue-900 to-indigo-900 text-white p-4 rounded-xl shadow-md flex justify-between items-center">
                <div>
                    <p class="text-[10px] text-blue-200 uppercase tracking-wider font-semibold">当前检验对象</p>
                    <h2 id="selectedPlayerName" class="text-base font-bold mt-0.5">-</h2>
                </div>
                <div class="text-right">
                    <p class="text-[10px] text-amber-300 font-semibold">当前总积分</p>
                    <p id="liveScoreTop" class="text-xl font-black text-amber-400">0 分</p>
                </div>
            </div>

            <div class="bg-white rounded-xl shadow-sm p-4 border border-slate-200/60">
                <h3 class="text-xs font-bold text-slate-700 mb-2 flex items-center gap-1">
                    📊 荣誉多维度雷达图
                </h3>
                <div class="w-full max-w-[280px] mx-auto aspect-square flex items-center justify-center">
                    <canvas id="radarChart"></canvas>
                </div>
            </div>

            <form id="editForm" oninput="calculateLiveScore()" class="space-y-4 safe-bottom">
                <input type="hidden" id="playerId">
                
                <div class="bg-amber-50/70 p-3 rounded-xl border border-amber-200/50 space-y-2.5">
                    <h3 class="text-xs font-bold text-amber-900 flex items-center gap-1">⭐ 核心重磅荣誉</h3>
                    <div class="grid grid-cols-2 gap-2">
                        <div>
                            <label class="block text-[11px] text-slate-500 mb-1">常规赛 MVP (+70)</label>
                            <input type="number" id="mvp" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-2 text-xs focus:outline-blue-500 text-center font-bold">
                        </div>
                        <div>
                            <label class="block text-[11px] text-slate-500 mb-1">总决赛 FMVP (+50)</label>
                            <input type="number" id="fmvp" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-2 text-xs focus:outline-blue-500 text-center font-bold">
                        </div>
                    </div>
                </div>

                <div class="bg-blue-50/70 p-3 rounded-xl border border-blue-200/50 space-y-2.5">
                    <h3 class="text-xs font-bold text-blue-900 flex items-center gap-1">🏆 季后赛终极荣誉 (单次内互斥)</h3>
                    <div class="grid grid-cols-2 gap-2">
                        <div>
                            <label class="block text-[11px] text-slate-500 mb-1">冠军-前二核心 (+100)</label>
                            <input type="number" id="champsTop2" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-2 text-xs focus:outline-blue-500 text-center font-bold">
                        </div>
                        <div>
                            <label class="block text-[11px] text-slate-500 mb-1">冠军-纯首发 (+30)</label>
                            <input type="number" id="champsStarter" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-2 text-xs focus:outline-blue-500 text-center font-bold">
                        </div>
                        <div>
                            <label class="block text-[11px] text-slate-500 mb-1">亚军-前二核心 (+20)</label>
                            <input type="number" id="finalsLossTop2" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-2 text-xs focus:outline-blue-500 text-center font-bold">
                        </div>
                        <div>
                            <label class="block text-[11px] text-slate-500 mb-1">亚军-纯首发 (+5)</label>
                            <input type="number" id="finalsLossStarter" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-2 text-xs focus:outline-blue-500 text-center font-bold">
                        </div>
                    </div>
                </div>

                <div class="bg-slate-50 p-3 rounded-xl border border-slate-200 space-y-2.5">
                    <h3 class="text-xs font-bold text-slate-700 flex items-center gap-1">🏅 最佳阵容与单项大奖</h3>
                    <div class="grid grid-cols-3 gap-2">
                        <div>
                            <label class="block text-[10px] text-slate-500 mb-0.5 truncate text-center">最佳一阵(+20)</label>
                            <input type="number" id="allNBA1st" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-1.5 text-xs focus:outline-blue-500 text-center">
                        </div>
                        <div>
                            <label class="block text-[10px] text-slate-500 mb-0.5 truncate text-center">最佳二阵(+15)</label>
                            <input type="number" id="allNBA2nd" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-1.5 text-xs focus:outline-blue-500 text-center">
                        </div>
                        <div>
                            <label class="block text-[10px] text-slate-500 mb-0.5 truncate text-center">最佳三阵(+5)</label>
                            <input type="number" id="allNBA3rd" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-1.5 text-xs focus:outline-blue-500 text-center">
                        </div>
                        <div>
                            <label class="block text-[10px] text-slate-500 mb-0.5 truncate text-center">DPOY (+20)</label>
                            <input type="number" id="dpoy" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-1.5 text-xs focus:outline-blue-500 text-center">
                        </div>
                        <div>
                            <label class="block text-[10px] text-slate-500 mb-0.5 truncate text-center">得分王 (+10)</label>
                            <input type="number" id="scoringTitle" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-1.5 text-xs focus:outline-blue-500 text-center">
                        </div>
                        <div>
                            <label class="block text-[10px] text-slate-500 mb-0.5 truncate text-center">全明星 (+5)</label>
                            <input type="number" id="allStar" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-1.5 text-xs focus:outline-blue-500 text-center">
                        </div>
                    </div>
                    <div class="grid grid-cols-2 gap-2 pt-1 border-t border-slate-200/60">
                        <div>
                            <label class="block text-[10px] text-slate-500 mb-0.5 text-center">最佳一防 (+5)</label>
                            <input type="number" id="allDefense1st" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-1.5 text-xs focus:outline-blue-500 text-center">
                        </div>
                        <div>
                            <label class="block text-[10px] text-slate-500 mb-0.5 text-center">最佳二防 (+3)</label>
                            <input type="number" id="allDefense2nd" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-1.5 text-xs focus:outline-blue-500 text-center">
                        </div>
                    </div>
                </div>

                <div class="bg-emerald-50/70 p-3 rounded-xl border border-emerald-200/50 space-y-2.5">
                    <h3 class="text-xs font-bold text-emerald-900 flex items-center gap-1">📈 季后赛累积征程</h3>
                    <div class="grid grid-cols-2 gap-2">
                        <div>
                            <label class="block text-[11px] text-slate-500 mb-1">入季后赛次数(+1)</label>
                            <input type="number" id="playoffAppearances" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-2 text-xs focus:outline-blue-500 text-center">
                        </div>
                        <div>
                            <label class="block text-[11px] text-slate-500 mb-1">总晋级轮数(+1)</label>
                            <input type="number" id="playoffRoundsWon" min="0" class="w-full bg-white border border-slate-300 rounded-lg p-2 text-xs focus:outline-blue-500 text-center">
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </main>

    <div id="sticky-footer" class="fixed bottom-0 inset-x-0 bg-white/95 backdrop-blur border-t border-slate-200 p-3 shadow-lg z-50 flex items-center justify-between gap-3 pb-[calc(env(safe-area-inset-bottom)+12px)]">
        <div id="footer-left-info" class="flex flex-col">
            <span class="text-[10px] text-slate-400 font-medium">预览球星评分</span>
            <span id="liveScore" class="text-lg font-black text-blue-900">0 分</span>
        </div>
        <button id="footer-btn" type="button" onclick="savePlayerData()" class="flex-1 max-w-xs bg-blue-600 hover:bg-blue-500 active:scale-98 text-white font-bold text-xs py-3 px-4 rounded-xl transition shadow text-center">
            💾 保存当前改动并应用
        </button>
    </div>

    <script>
        // 初始历史前 20 大球员真实荣誉数据
        const initialPlayersData = [
            { id: 1, name: "迈克尔-乔丹 (Michael Jordan)", champsTop2: 6, champsStarter: 0, finalsLossTop2: 0, finalsLossStarter: 0, mvp: 5, fmvp: 6, allNBA1st: 10, allNBA2nd: 1, allNBA3rd: 0, dpoy: 1, scoringTitle: 10, allStar: 14, allDefense1st: 9, allDefense2nd: 0, playoffAppearances: 13, playoffRoundsWon: 30 },
            { id: 2, name: "勒布朗-詹姆斯 (LeBron James)", champsTop2: 4, champsStarter: 0, finalsLossTop2: 6, finalsLossStarter: 0, mvp: 4, fmvp: 4, allNBA1st: 13, allNBA2nd: 3, allNBA3rd: 4, dpoy: 0, scoringTitle: 1, allStar: 20, allDefense1st: 5, allDefense2nd: 1, playoffAppearances: 17, playoffRoundsWon: 41 },
            { id: 3, name: "卡里姆-阿卜杜尔-贾巴尔 (Kareem Abdul-Jabbar)", champsTop2: 6, champsStarter: 0, finalsLossTop2: 4, finalsLossStarter: 0, mvp: 6, fmvp: 2, allNBA1st: 10, allNBA2nd: 5, allNBA3rd: 0, dpoy: 0, scoringTitle: 2, allStar: 19, allDefense1st: 5, allDefense2nd: 6, playoffAppearances: 18, playoffRoundsWon: 37 },
            { id: 4, name: "比尔-拉塞尔 (Bill Russell)", champsTop2: 11, champsStarter: 0, finalsLossTop2: 1, finalsLossStarter: 0, mvp: 5, fmvp: 0, allNBA1st: 3, allNBA2nd: 8, allNBA3rd: 0, dpoy: 0, scoringTitle: 0, allStar: 12, allDefense1st: 1, allDefense2nd: 0, playoffAppearances: 13, playoffRoundsWon: 29 },
            { id: 5, name: "埃尔文-约翰逊 (Magic Johnson)", champsTop2: 5, champsStarter: 0, finalsLossTop2: 4, finalsLossStarter: 0, mvp: 3, fmvp: 3, allNBA1st: 9, allNBA2nd: 1, allNBA3rd: 0, dpoy: 0, scoringTitle: 0, allStar: 12, allDefense1st: 0, allDefense2nd: 0, playoffAppearances: 13, playoffRoundsWon: 32 },
            { id: 6, name: "威尔特-张伯伦 (Wilt Chamberlain)", champsTop2: 2, champsStarter: 0, finalsLossTop2: 4, finalsLossStarter: 0, mvp: 4, fmvp: 1, allNBA1st: 7, allNBA2nd: 3, allNBA3rd: 0, dpoy: 0, scoringTitle: 7, allStar: 13, allDefense1st: 2, text: 0, playoffAppearances: 13, playoffRoundsWon: 22 },
            { id: 7, name: "拉里-伯德 (Larry Bird)", champsTop2: 3, champsStarter: 0, finalsLossTop2: 2, finalsLossStarter: 0, mvp: 3, fmvp: 2, allNBA1st: 9, allNBA2nd: 1, allNBA3rd: 0, dpoy: 0, scoringTitle: 0, allStar: 12, allDefense1st: 0, allDefense2nd: 3, playoffAppearances: 12, playoffRoundsWon: 24 },
            { id: 8, name: "蒂姆-邓肯 (Tim Duncan)", champsTop2: 5, champsStarter: 0, finalsLossTop2: 1, finalsLossStarter: 0, mvp: 2, fmvp: 3, allNBA1st: 10, allNBA2nd: 3, allNBA3rd: 2, dpoy: 0, scoringTitle: 0, allStar: 15, allDefense1st: 8, allDefense2nd: 7, playoffAppearances: 18, playoffRoundsWon: 35 },
            { id: 9, name: "科比-布莱恩特 (Kobe Bryant)", champsTop2: 5, champsStarter: 0, finalsLossTop2: 2, finalsLossStarter: 0, mvp: 1, fmvp: 2, allNBA1st: 11, allNBA2nd: 2, allNBA3rd: 2, dpoy: 0, scoringTitle: 2, allStar: 18, allDefense1st: 9, allDefense2nd: 3, playoffAppearances: 15, playoffRoundsWon: 33 },
            { id: 10, name: "沙奎尔-奥尼尔 (Shaquille O'Neal)", champsTop2: 4, champsStarter: 0, finalsLossTop2: 2, finalsLossStarter: 0, mvp: 1, fmvp: 3, allNBA1st: 8, allNBA2nd: 2, allNBA3rd: 4, dpoy: 0, scoringTitle: 2, allStar: 15, allDefense1st: 0, allDefense2nd: 3, playoffAppearances: 17, playoffRoundsWon: 32 },
            { id: 11, name: "阿基姆-奥拉朱旺 (Hakeem Olajuwon)", champsTop2: 2, champsStarter: 0, finalsLossTop2: 1, finalsLossStarter: 0, mvp: 1, fmvp: 2, allNBA1st: 6, allNBA2nd: 3, allNBA3rd: 3, dpoy: 2, scoringTitle: 0, allStar: 12, allDefense1st: 5, allDefense2nd: 4, playoffAppearances: 15, playoffRoundsWon: 16 },
            { id: 12, name: "斯蒂芬-库里 (Stephen Curry)", champsTop2: 4, champsStarter: 0, finalsLossTop2: 2, finalsLossStarter: 0, mvp: 2, fmvp: 1, allNBA1st: 4, allNBA2nd: 4, allNBA3rd: 2, dpoy: 0, scoringTitle: 2, allStar: 10, allDefense1st: 0, allDefense2nd: 0, playoffAppearances: 9, playoffRoundsWon: 23 },
            { id: 13, name: "凯文-杜兰特 (Kevin Durant)", champsTop2: 2, champsStarter: 0, finalsLossTop2: 2, finalsLossStarter: 0, mvp: 1, fmvp: 2, allNBA1st: 6, allNBA2nd: 5, allNBA3rd: 0, dpoy: 0, scoringTitle: 4, allStar: 14, allDefense1st: 0, allDefense2nd: 0, playoffAppearances: 13, playoffRoundsWon: 22 },
            { id: 14, name: "奥斯卡-罗伯特森 (Oscar Robertson)", champsTop2: 1, champsStarter: 0, finalsLossTop2: 1, finalsLossStarter: 0, mvp: 1, fmvp: 0, allNBA1st: 9, allNBA2nd: 2, allNBA3rd: 0, dpoy: 0, scoringTitle: 1, allStar: 12, allDefense1st: 0, allDefense2nd: 0, playoffAppearances: 10, playoffRoundsWon: 11 },
            { id: 15, name: "卡尔-马龙 (Karl Malone)", champsTop2: 0, champsStarter: 0, finalsLossTop2: 3, finalsLossStarter: 0, mvp: 2, fmvp: 0, allNBA1st: 11, allNBA2nd: 2, allNBA3rd: 1, dpoy: 0, scoringTitle: 0, allStar: 14, allDefense1st: 3, allDefense2nd: 1, playoffAppearances: 19, playoffRoundsWon: 22 },
            { id: 16, name: "莫塞斯-马龙 (Moses Malone)", champsTop2: 1, champsStarter: 0, finalsLossTop2: 1, finalsLossStarter: 0, mvp: 3, fmvp: 1, allNBA1st: 4, allNBA2nd: 4, allNBA3rd: 0, dpoy: 0, scoringTitle: 0, allStar: 13, allDefense1st: 1, allDefense2nd: 1, playoffAppearances: 12, playoffRoundsWon: 10 },
            { id: 17, name: "杰里-韦斯特 (Jerry West)", champsTop2: 1, champsStarter: 0, finalsLossTop2: 8, finalsLossStarter: 0, mvp: 0, fmvp: 1, allNBA1st: 10, allNBA2nd: 2, allNBA3rd: 0, dpoy: 0, scoringTitle: 1, allStar: 14, allDefense1st: 4, allDefense2nd: 1, playoffAppearances: 14, playoffRoundsWon: 22 },
            { id: 18, name: "埃尔金-贝勒 (Elgin Baylor)", champsTop2: 0, champsStarter: 0, finalsLossTop2: 8, finalsLossStarter: 0, mvp: 0, fmvp: 0, allNBA1st: 10, allNBA2nd: 0, allNBA3rd: 0, dpoy: 0, scoringTitle: 0, allStar: 11, allDefense1st: 0, allDefense2nd: 0, playoffAppearances: 14, playoffRoundsWon: 16 },
            { id: 19, name: "扬尼斯-阿德托昆博 (Giannis Antetokounmpo)", champsTop2: 1, champsStarter: 0, finalsLossTop2: 0, finalsLossStarter: 0, mvp: 2, fmvp: 1, allNBA1st: 6, allNBA2nd: 2, allNBA3rd: 0, dpoy: 1, scoringTitle: 0, allStar: 8, allDefense1st: 5, allDefense2nd: 0, playoffAppearances: 9, playoffRoundsWon: 12 },
            { id: 20, name: "尼古拉-约基奇 (Nikola Jokić)", champsTop2: 1, champsStarter: 0, finalsLossTop2: 0, finalsLossStarter: 0, mvp: 3, fmvp: 1, allNBA1st: 4, allNBA2nd: 2, allNBA3rd: 0, dpoy: 0, scoringTitle: 0, allStar: 6, allDefense1st: 0, allDefense2nd: 0, playoffAppearances: 6, playoffRoundsWon: 10 }
        ];

        let players = JSON.parse(localStorage.getItem('nba_players_scores_mobile')) || [...initialPlayersData];
        let radarChart = null;
        let currentSelectedId = 1;
        let activeTab = 'rank';

        // 积分计算核心公式
        function calculateScore(p) {
            let score = 0;
            score += (p.champsTop2 * 100) + (p.champsStarter * 30);
            score += (p.finalsLossTop2 * 20) + (p.finalsLossStarter * 5);
            score += (p.mvp * 70) + (p.fmvp * 50);
            score += (p.allNBA1st * 20) + (p.allNBA2nd * 15) + (p.allNBA3rd * 5);
            score += (p.dpoy * 20) + (p.scoringTitle * 10) + (p.allStar * 5);
            score += (p.allDefense1st * 5) + (p.allDefense2nd * 3);
            score += (p.playoffAppearances * 1) + (p.playoffRoundsWon * 1);
            return score;
        }

        // 切换标签页
        function switchTab(tab) {
            activeTab = tab;
            const tabRank = document.getElementById('tab-rank');
            const tabDetail = document.getElementById('tab-detail');
            const panelRank = document.getElementById('panel-rank');
            const panelDetail = document.getElementById('panel-detail');
            const footerBtn = document.getElementById('footer-btn');

            if (tab === 'rank') {
                tabRank.className = "flex-1 py-3 text-center text-sm font-bold border-b-2 border-blue-600 text-blue-600 transition-all";
                tabDetail.className = "flex-1 py-3 text-center text-sm font-medium border-b-2 border-transparent text-slate-500 transition-all flex justify-center items-center gap-1";
                panelRank.classList.remove('hidden');
                panelDetail.classList.add('hidden');
                footerBtn.innerText = "🔍 选择球员进行检验";
                footerBtn.className = "flex-1 max-w-xs bg-slate-800 text-white font-bold text-xs py-3 px-4 rounded-xl text-center shadow";
                footerBtn.setAttribute("onclick", "switchTab('detail')");
            } else {
                tabRank.className = "flex-1 py-3 text-center text-sm font-medium border-b-2 border-transparent text-slate-500 transition-all";
                tabDetail.className = "flex-1 py-3 text-center text-sm font-bold border-b-2 border-blue-600 text-blue-600 transition-all flex justify-center items-center gap-1";
                panelRank.classList.add('hidden');
                panelDetail.classList.remove('hidden');
                footerBtn.innerText = "💾 保存改动并重新排名";
                footerBtn.className = "flex-1 max-w-xs bg-blue-600 active:bg-blue-700 text-white font-bold text-xs py-3 px-4 rounded-xl text-center shadow shadow-blue-500/20";
                footerBtn.setAttribute("onclick", "savePlayerData()");
            }
        }

        // 渲染排行榜表格
        function renderRankTable() {
            const sortedPlayers = players.map(p => ({ ...p, totalScore: calculateScore(p) }))
                                         .sort((a, b) => b.totalScore - a.totalScore);

            const tbody = document.getElementById("rankTableBody");
            tbody.innerHTML = "";

            sortedPlayers.forEach((player, index) => {
                const isSelected = player.id === currentSelectedId;
                const row = document.createElement("tr");
                row.className = `transition ${isSelected ? 'bg-blue-50/80 font-semibold border-l-4 border-blue-600' : 'active:bg-slate-50'}`;
                
                row.setAttribute("onclick", `selectPlayer(${player.id}, true)`);

                let rankBadge = `<span class="text-slate-500 font-bold">${index + 1}</span>`;
                if(index === 0) rankBadge = `🥇`;
                if(index === 1) rankBadge = `🥈`;
                if(index === 2) rankBadge = `🥉`;

                row.innerHTML = `
                    <td class="p-3 text-center text-sm">${rankBadge}</td>
                    <td class="p-3">
                        <div class="font-medium text-slate-900 break-words max-w-[140px]">${player.name.split(' (')[0]}</div>
                        <div class="text-[10px] text-slate-400 font-normal truncate max-w-[140px]">${player.name.split(' (')[1] ? player.name.split(' (')[1].replace(')', '') : ''}</div>
                    </td>
                    <td class="p-3 text-right font-bold text-slate-900">${player.totalScore}分</td>
                    <td class="p-3 text-center">
                        <span class="inline-block px-2 py-1 text-[10px] bg-blue-50 text-blue-600 font-medium rounded-md border border-blue-100">检验</span>
                    </td>
                `;
                tbody.appendChild(row);
            });
        }

        // 选择球星
        function selectPlayer(id, autoSwitch = false) {
            currentSelectedId = id;
            const player = players.find(p => p.id === id);
            if (!player) return;

            document.getElementById("selectedPlayerName").innerText = player.name;
            document.getElementById("playerId").value = player.id;
            document.getElementById("detail-badge").classList.remove('hidden');

            const fields = [
                'champsTop2', 'champsStarter', 'finalsLossTop2', 'finalsLossStarter',
                'mvp', 'fmvp', 'allNBA1st', 'allNBA2nd', 'allNBA3rd', 
                'dpoy', 'scoringTitle', 'allStar', 'allDefense1st', 'allDefense2nd',
                'playoffAppearances', 'playoffRoundsWon'
            ];
            
            fields.forEach(field => {
                document.getElementById(field).value = player[field] || 0;
            });

            calculateLiveScore();
            updateRadarChart(player);
            renderRankTable();

            if (autoSwitch) {
                switchTab('detail');
            }
        }

        // 实时更新得分数值
        function calculateLiveScore() {
            const tempPlayer = {};
            const fields = [
                'champsTop2', 'champsStarter', 'finalsLossTop2', 'finalsLossStarter',
                'mvp', 'fmvp', 'allNBA1st', 'allNBA2nd', 'allNBA3rd', 
                'dpoy', 'scoringTitle', 'allStar', 'allDefense1st', 'allDefense2nd',
                'playoffAppearances', 'playoffRoundsWon'
            ];
            fields.forEach(field => {
                tempPlayer[field] = parseInt(document.getElementById(field).value) || 0;
            });
            
            const liveScore = calculateScore(tempPlayer);
            document.getElementById("liveScore").innerText = liveScore + " 分";
            document.getElementById("liveScoreTop").innerText = liveScore + " 分";
        }

        // 保存改动
        function savePlayerData() {
            const id = parseInt(document.getElementById("playerId").value);
            const playerIndex = players.findIndex(p => p.id === id);
            if (playerIndex === -1) return;

            const fields = [
                'champsTop2', 'champsStarter', 'finalsLossTop2', 'finalsLossStarter',
                'mvp', 'fmvp', 'allNBA1st', 'allNBA2nd', 'allNBA3rd', 
                'dpoy', 'scoringTitle', 'allStar', 'allDefense1st', 'allDefense2nd',
                'playoffAppearances', 'playoffRoundsWon'
            ];

            fields.forEach(field => {
                players[playerIndex][field] = parseInt(document.getElementById(field).value) || 0;
            });

            localStorage.setItem('nba_players_scores_mobile', JSON.stringify(players));
            
            renderRankTable();
            updateRadarChart(players[playerIndex]);
            
            switchTab('rank');
            
            const toast = document.createElement('div');
            toast.className = "fixed top-4 left-1/2 -translate-x-1/2 bg-slate-900/90 text-white text-xs px-4 py-2.5 rounded-full shadow-lg z-50 transition-opacity font-medium";
            toast.innerText = `更新成功，已重新计算排名！`;
            document.body.appendChild(toast);
            setTimeout(() => { toast.style.opacity = '0'; setTimeout(() => toast.remove(), 300); }, 1500);
        }

        // 重置数据
        function resetData() {
            if(confirm("确定恢复到默认的初始荣誉数据吗？")) {
                localStorage.removeItem('nba_players_scores_mobile');
                players = [...initialPlayersData];
                renderRankTable();
                selectPlayer(currentSelectedId);
                switchTab('rank');
            }
        }

        // 雷达图更新
        function updateRadarChart(player) {
            const labels = ['总冠军分', '核心MVP', '最佳阵容', '防守高阶', '季后赛'];
            const dataValues = [
                (player.champsTop2 * 100) + (player.champsStarter * 30),
                (player.mvp * 70) + (player.fmvp * 50),
                (player.allNBA1st * 20) + (player.allNBA2nd * 15) + (player.allNBA3rd * 5) + (player.scoringTitle * 10),
                (player.dpoy * 20) + (player.allDefense1st * 5) + (player.allDefense2nd * 3),
                (player.playoffAppearances * 1) + (player.playoffRoundsWon * 1)
            ];

            if (radarChart) {
                radarChart.data.datasets[0].label = `维度细分`;
                radarChart.data.datasets[0].data = dataValues;
                radarChart.update();
            } else {
                const ctx = document.getElementById('radarChart').getContext('2d');
                radarChart = new Chart(ctx, {
                    type: 'radar',
                    data: {
                        labels: labels,
                        datasets: [{
                            label: `维度细分`,
                            data: dataValues,
                            backgroundColor: 'rgba(30, 58, 138, 0.15)',
                            borderColor: 'rgba(30, 58, 138, 1)',
                            borderWidth: 1.5,
                            pointBackgroundColor: 'rgba(234, 179, 8, 1)',
                            pointRadius: 3
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: true,
                        plugins: { legend: { display: false } },
                        scales: {
                            r: {
                                angleLines: { display: true },
                                grid: { color: '#e2e8f0' },
                                pointLabels: { font: { size: 10, weight: 'bold' } },
                                suggestedMin: 0,
                                ticks: { display: false, stepSize: 150 }
                            }
                        }
                    }
                });
            }
        }

        window.onload = function() {
            renderRankTable();
            selectPlayer(1, false);
            switchTab('rank'); 
        };
    </script>
</body>
</html>
