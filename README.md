<!DOCTYPE html>
<html lang="ja" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>カレンダー - Standalone Dark Calendar</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        slate: {
                            750: '#293548',
                            850: '#151e2e',
                            950: '#0b0f19',
                        }
                    }
                }
            }
        }
    </script>
    <!-- Lucide Icons CDN -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        body {
            background-color: #0b0f19;
            color: #f1f5f9;
        }
        .calendar-cell {
            min-height: 115px;
        }
        @media (max-width: 640px) {
            .calendar-cell {
                min-height: 80px;
            }
        }
        .event-badge {
            transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .event-badge:hover {
            filter: brightness(1.15);
            transform: translateY(-1px);
        }
        /* スクロールバーのカスタマイズ (ダーク) */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #0b0f19;
        }
        ::-webkit-scrollbar-thumb {
            background: #334155;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #475569;
        }
        /* 背景グラデーション・グロー */
        .bg-glow {
            background: radial-gradient(circle at 50% -20%, rgba(99, 102, 241, 0.15), transparent 50%),
                        radial-gradient(circle at 100% 100%, rgba(139, 92, 246, 0.08), transparent 40%);
        }
    </style>
</head>
<body class="bg-[#0b0f19] text-slate-100 antialiased min-h-screen flex flex-col font-sans bg-glow selection:bg-indigo-500 selection:text-white">

    <!-- ナビゲーションバー -->
    <header class="bg-slate-900/80 backdrop-blur-md border-b border-slate-800/80 sticky top-0 z-30 shadow-lg shadow-black/20">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3.5">
            <div class="flex items-center justify-between flex-wrap gap-3">
                <!-- 左側: タイトル & 年月 & ナビゲーション -->
                <div class="flex items-center space-x-3 sm:space-x-5">
                    <div class="flex items-center space-x-2.5">
                        <div class="w-9 h-9 rounded-xl bg-gradient-to-tr from-indigo-600 to-purple-600 flex items-center justify-center shadow-md shadow-indigo-500/20 ring-1 ring-white/10">
                            <i data-lucide="calendar" class="w-5 h-5 text-white"></i>
                        </div>
                        <div class="hidden sm:block">
                            <span class="text-xl font-extrabold tracking-tight bg-gradient-to-r from-white via-slate-200 to-indigo-300 bg-clip-text text-transparent">
                                CALENDAR
                            </span>
                            <span class="ml-1.5 text-[10px] font-semibold tracking-wider text-indigo-400 uppercase bg-indigo-950/60 border border-indigo-800/60 px-1.5 py-0.5 rounded">
                                HTML
                            </span>
                        </div>
                    </div>

                    <div class="flex items-center bg-slate-950/80 rounded-xl p-1 border border-slate-800 shadow-inner">
                        <button id="prevBtn" class="p-1.5 rounded-lg hover:bg-slate-800 text-slate-400 hover:text-white transition" title="前月">
                            <i data-lucide="chevron-left" class="w-4 h-4"></i>
                        </button>
                        <button id="todayBtn" class="px-3 py-1 text-xs font-semibold text-slate-300 hover:text-white hover:bg-slate-800 rounded-lg transition">
                            今日
                        </button>
                        <button id="nextBtn" class="p-1.5 rounded-lg hover:bg-slate-800 text-slate-400 hover:text-white transition" title="翌月">
                            <i data-lucide="chevron-right" class="w-4 h-4"></i>
                        </button>
                    </div>

                    <h1 id="currentMonthYearLabel" class="text-xl sm:text-2xl font-black text-white tracking-tight">
                        ----年 --月
                    </h1>
                </div>

                <!-- 右側: アクションボタン & 表示切替 -->
                <div class="flex items-center space-x-2 sm:space-x-3">
                    <!-- データバックアップ (エクスポート/インポート) -->
                    <div class="hidden lg:flex items-center space-x-1 text-xs text-slate-400 mr-1">
                        <button id="exportBtn" class="px-2.5 py-1.5 rounded-lg hover:bg-slate-800/80 hover:text-slate-200 transition flex items-center space-x-1" title="予定データをJSONファイルとして保存">
                            <i data-lucide="download" class="w-3.5 h-3.5"></i>
                            <span>バックアップ</span>
                        </button>
                        <button id="importBtn" class="px-2.5 py-1.5 rounded-lg hover:bg-slate-800/80 hover:text-slate-200 transition flex items-center space-x-1" title="JSONファイルから復元">
                            <i data-lucide="upload" class="w-3.5 h-3.5"></i>
                            <span>復元</span>
                        </button>
                        <input type="file" id="importFileInput" accept=".json" class="hidden">
                    </div>

                    <!-- 表示切り替え (グリッド / リスト) -->
                    <div class="flex bg-slate-950/80 rounded-xl p-1 border border-slate-800 text-xs sm:text-sm font-medium text-slate-400 shadow-inner">
                        <button id="viewGridBtn" class="px-3 py-1.5 rounded-lg bg-indigo-600 text-white shadow-md shadow-indigo-600/30 flex items-center space-x-1.5 transition">
                            <i data-lucide="layout-grid" class="w-4 h-4"></i>
                            <span class="hidden md:inline font-semibold">カレンダー</span>
                        </button>
                        <button id="viewListBtn" class="px-3 py-1.5 rounded-lg hover:text-white hover:bg-slate-800/60 flex items-center space-x-1.5 transition">
                            <i data-lucide="list" class="w-4 h-4"></i>
                            <span class="hidden md:inline font-semibold">予定一覧</span>
                        </button>
                    </div>

                    <!-- 予定追加ボタン -->
                    <button id="openNewModalBtn" class="bg-gradient-to-r from-indigo-500 to-purple-600 hover:from-indigo-600 hover:to-purple-700 text-white px-3.5 py-1.5 sm:px-4 sm:py-2 rounded-xl text-sm font-semibold shadow-lg shadow-indigo-500/25 flex items-center space-x-2 transition hover:scale-[1.02] active:scale-[0.98]">
                        <i data-lucide="plus" class="w-4 h-4"></i>
                        <span>予定を追加</span>
                    </button>
                </div>
            </div>
        </div>
    </header>

    <!-- メインコンテンツ -->
    <main class="max-w-7xl mx-auto px-2 sm:px-6 lg:px-8 py-5 sm:py-7 flex-1 w-full">
        <!-- カレンダーグリッド表示 -->
        <div id="calendarViewContainer" class="bg-slate-900/60 backdrop-blur-md rounded-2xl shadow-xl shadow-black/40 border border-slate-800/80 overflow-hidden">
            <!-- 曜日ヘッダー -->
            <div class="grid grid-cols-7 border-b border-slate-800 text-center font-semibold text-xs sm:text-sm bg-slate-950/60 py-1">
                <div class="py-2.5 text-rose-400 flex items-center justify-center space-x-1">
                    <span>日</span>
                </div>
                <div class="py-2.5 text-slate-400">月</div>
                <div class="py-2.5 text-slate-400">火</div>
                <div class="py-2.5 text-slate-400">水</div>
                <div class="py-2.5 text-slate-400">木</div>
                <div class="py-2.5 text-slate-400">金</div>
                <div class="py-2.5 text-sky-400 flex items-center justify-center space-x-1">
                    <span>土</span>
                </div>
            </div>

            <!-- カレンダーの日付セルコンテナ -->
            <div id="calendarDaysGrid" class="grid grid-cols-7 divide-x divide-y divide-slate-800/80 border-b border-slate-800/80">
                <!-- JavaScriptで日付セルを動的生成 -->
            </div>
        </div>

        <!-- 予定一覧リスト表示 (トグル用) -->
        <div id="listViewContainer" class="hidden bg-slate-900/60 backdrop-blur-md rounded-2xl shadow-xl shadow-black/40 border border-slate-800/80 overflow-hidden p-4 sm:p-6">
            <div class="flex items-center justify-between mb-5 flex-wrap gap-3">
                <h2 class="text-lg font-bold text-white flex items-center space-x-2.5">
                    <div class="w-8 h-8 rounded-lg bg-indigo-500/10 border border-indigo-500/20 flex items-center justify-center text-indigo-400">
                        <i data-lucide="list-todo" class="w-4 h-4"></i>
                    </div>
                    <span id="listTitle">今月の予定一覧</span>
                </h2>
                <!-- 検索ボックス -->
                <div class="relative w-full sm:w-72">
                    <i data-lucide="search" class="w-4 h-4 text-slate-500 absolute left-3.5 top-3"></i>
                    <input type="text" id="searchInput" placeholder="予定を検索..." class="w-full pl-10 pr-3.5 py-2 text-sm bg-slate-950/80 border border-slate-800 rounded-xl text-slate-100 placeholder-slate-500 focus:outline-none focus:border-indigo-500 focus:ring-1 focus:ring-indigo-500 transition shadow-inner">
                </div>
            </div>
            <div id="eventsListContent" class="space-y-3">
                <!-- JavaScriptでリストを動的生成 -->
            </div>
        </div>
    </main>

    <!-- モーダル: 予定の追加 / 編集 -->
    <div id="eventModal" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/75 backdrop-blur-md hidden transition-opacity">
        <div class="bg-slate-900 rounded-2xl shadow-2xl shadow-black/80 w-full max-w-lg overflow-hidden transform transition-all border border-slate-800">
            <!-- モーダルヘッダー -->
            <div class="px-6 py-4 border-b border-slate-800 flex items-center justify-between bg-slate-950/60">
                <h3 id="modalTitle" class="text-lg font-bold text-white flex items-center space-x-2">
                    <i data-lucide="calendar-plus" class="w-5 h-5 text-indigo-400"></i>
                    <span>予定を追加</span>
                </h3>
                <button id="closeModalBtn" class="text-slate-400 hover:text-white p-1 rounded-lg hover:bg-slate-800 transition">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>

            <!-- モーダルフォーム -->
            <form id="eventForm" class="p-6 space-y-4">
                <input type="hidden" id="eventId" name="id" value="">

                <!-- タイトル -->
                <div>
                    <label class="block text-xs font-semibold text-slate-400 uppercase tracking-wider mb-1.5">
                        タイトル <span class="text-rose-400">*</span>
                    </label>
                    <input type="text" id="eventTitle" name="title" required placeholder="例: プロジェクトミーティング、旅行、記念日"
                        class="w-full px-3.5 py-2.5 text-sm bg-slate-950 border border-slate-800 rounded-xl text-white placeholder-slate-500 focus:outline-none focus:border-indigo-500 focus:ring-1 focus:ring-indigo-500 transition">
                </div>

                <!-- 日付 -->
                <div>
                    <label class="block text-xs font-semibold text-slate-400 uppercase tracking-wider mb-1.5">
                        日付 <span class="text-rose-400">*</span>
                    </label>
                    <input type="date" id="eventDate" name="event_date" required
                        class="w-full px-3.5 py-2.5 text-sm bg-slate-950 border border-slate-800 rounded-xl text-white focus:outline-none focus:border-indigo-500 focus:ring-1 focus:ring-indigo-500 transition [color-scheme:dark]">
                </div>

                <!-- 終日 & 時間帯 -->
                <div class="bg-slate-950/60 p-3.5 rounded-xl border border-slate-800/80 space-y-3">
                    <div class="flex items-center justify-between">
                        <label for="isAllDay" class="text-sm font-medium text-slate-300 cursor-pointer">終日予定</label>
                        <input type="checkbox" id="isAllDay" name="is_all_day" checked
                            class="w-4 h-4 text-indigo-600 rounded bg-slate-900 border-slate-700 focus:ring-indigo-500 focus:ring-offset-slate-900 cursor-pointer">
                    </div>

                    <div id="timeInputsContainer" class="hidden grid grid-cols-2 gap-3 pt-2 border-t border-slate-800">
                        <div>
                            <label class="block text-xs text-slate-400 mb-1">開始時刻</label>
                            <input type="time" id="startTime" name="start_time"
                                class="w-full px-3 py-1.5 text-sm bg-slate-900 border border-slate-700 rounded-lg text-white focus:outline-none focus:border-indigo-500 transition [color-scheme:dark]">
                        </div>
                        <div>
                            <label class="block text-xs text-slate-400 mb-1">終了時刻</label>
                            <input type="time" id="endTime" name="end_time"
                                class="w-full px-3 py-1.5 text-sm bg-slate-900 border border-slate-700 rounded-lg text-white focus:outline-none focus:border-indigo-500 transition [color-scheme:dark]">
                        </div>
                    </div>
                </div>

                <!-- カラー選択 -->
                <div>
                    <label class="block text-xs font-semibold text-slate-400 uppercase tracking-wider mb-2">ラベルカラー</label>
                    <div class="flex items-center space-x-3.5">
                        <label class="cursor-pointer">
                            <input type="radio" name="color" value="#3b82f6" class="peer sr-only" checked>
                            <div class="w-7 h-7 rounded-full bg-blue-500 peer-checked:ring-4 peer-checked:ring-blue-500/40 shadow-sm transition"></div>
                        </label>
                        <label class="cursor-pointer">
                            <input type="radio" name="color" value="#10b981" class="peer sr-only">
                            <div class="w-7 h-7 rounded-full bg-emerald-500 peer-checked:ring-4 peer-checked:ring-emerald-500/40 shadow-sm transition"></div>
                        </label>
                        <label class="cursor-pointer">
                            <input type="radio" name="color" value="#8b5cf6" class="peer sr-only">
                            <div class="w-7 h-7 rounded-full bg-purple-500 peer-checked:ring-4 peer-checked:ring-purple-500/40 shadow-sm transition"></div>
                        </label>
                        <label class="cursor-pointer">
                            <input type="radio" name="color" value="#06b6d4" class="peer sr-only">
                            <div class="w-7 h-7 rounded-full bg-cyan-500 peer-checked:ring-4 peer-checked:ring-cyan-500/40 shadow-sm transition"></div>
                        </label>
                        <label class="cursor-pointer">
                            <input type="radio" name="color" value="#f59e0b" class="peer sr-only">
                            <div class="w-7 h-7 rounded-full bg-amber-500 peer-checked:ring-4 peer-checked:ring-amber-500/40 shadow-sm transition"></div>
                        </label>
                        <label class="cursor-pointer">
                            <input type="radio" name="color" value="#f43f5e" class="peer sr-only">
                            <div class="w-7 h-7 rounded-full bg-rose-500 peer-checked:ring-4 peer-checked:ring-rose-500/40 shadow-sm transition"></div>
                        </label>
                        <label class="cursor-pointer">
                            <input type="radio" name="color" value="#64748b" class="peer sr-only">
                            <div class="w-7 h-7 rounded-full bg-slate-500 peer-checked:ring-4 peer-checked:ring-slate-500/40 shadow-sm transition"></div>
                        </label>
                    </div>
                </div>

                <!-- メモ・詳細 -->
                <div>
                    <label class="block text-xs font-semibold text-slate-400 uppercase tracking-wider mb-1.5">詳細・メモ</label>
                    <textarea id="eventDescription" name="description" rows="3" placeholder="場所、URL、アジェンダなどを記入"
                        class="w-full px-3.5 py-2.5 text-sm bg-slate-950 border border-slate-800 rounded-xl text-white placeholder-slate-500 focus:outline-none focus:border-indigo-500 focus:ring-1 focus:ring-indigo-500 transition resize-none"></textarea>
                </div>

                <!-- フッターボタングループ -->
                <div class="pt-3 border-t border-slate-800 flex items-center justify-between">
                    <div>
                        <button type="button" id="deleteEventBtn" class="hidden text-rose-400 hover:text-rose-300 hover:bg-rose-950/50 border border-transparent hover:border-rose-900/50 px-3 py-1.5 rounded-xl text-sm font-medium transition flex items-center space-x-1.5">
                            <i data-lucide="trash-2" class="w-4 h-4"></i>
                            <span>削除</span>
                        </button>
                    </div>
                    <div class="flex items-center space-x-2.5">
                        <button type="button" id="cancelModalBtn" class="px-4 py-2 text-sm font-medium text-slate-400 hover:text-white hover:bg-slate-800 rounded-xl transition">
                            キャンセル
                        </button>
                        <button type="submit" id="saveEventBtn" class="bg-gradient-to-r from-indigo-500 to-purple-600 hover:from-indigo-600 hover:to-purple-700 text-white px-5 py-2 rounded-xl text-sm font-semibold shadow-lg shadow-indigo-500/25 transition">
                            保存する
                        </button>
                    </div>
                </div>
            </form>
        </div>
    </div>

    <!-- 通知トースト -->
    <div id="toast" class="fixed bottom-6 right-6 z-50 bg-slate-800/90 backdrop-blur-md text-white px-4 py-3 rounded-xl shadow-2xl border border-slate-700 flex items-center space-x-2.5 transform translate-y-20 opacity-0 transition-all duration-300 pointer-events-none">
        <i id="toastIcon" data-lucide="check-circle" class="w-5 h-5 text-emerald-400"></i>
        <span id="toastMsg" class="text-sm font-medium">保存しました</span>
    </div>

    <!-- JavaScript アプリケーションロジック (完全スタンドアロン) -->
    <script>
        // ==========================================
        // 日本の祝日計算ロジック (JavaScript移植版)
        // ==========================================
        function getJapaneseHolidays(year) {
            const holidays = {};

            function add(dateStr, name) {
                holidays[dateStr] = name;
            }

            function getNthDayOfWeek(y, m, targetW, n) {
                const firstDay = new Date(y, m - 1, 1);
                const firstW = firstDay.getDay();
                const offset = (targetW - firstW + 7) % 7;
                const day = 1 + offset + (n - 1) * 7;
                return `${y}-${String(m).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
            }

            // 固定祝日
            add(`${year}-01-01`, '元日');
            add(`${year}-02-11`, '建国記念の日');
            if (year >= 2020) add(`${year}-02-23`, '天皇誕生日');
            add(`${year}-04-29`, '昭和の日');
            add(`${year}-05-03`, '憲法記念日');
            add(`${year}-05-04`, 'みどりの日');
            add(`${year}-05-05`, 'こどもの日');
            if (year >= 2016) add(`${year}-08-11`, '山の日');
            add(`${year}-11-03`, '文化の日');
            add(`${year}-11-23`, '勤労感謝の日');

            // ハッピーマンデー (第N月曜日)
            add(getNthDayOfWeek(year, 1, 1, 2), '成人の日');
            add(getNthDayOfWeek(year, 7, 1, 3), '海の日');
            add(getNthDayOfWeek(year, 9, 1, 3), '敬老の日');
            add(getNthDayOfWeek(year, 10, 1, 2), 'スポーツの日');

            // 春分の日 (1980〜2099)
            const springDay = Math.floor(20.8431 + 0.242194 * (year - 1980) - Math.floor((year - 1980) / 4));
            add(`${year}-03-${String(springDay).padStart(2, '0')}`, '春分の日');

            // 秋分の日 (1980〜2099)
            const autumnDay = Math.floor(23.2488 + 0.242194 * (year - 1980) - Math.floor((year - 1980) / 4));
            add(`${year}-09-${String(autumnDay).padStart(2, '0')}`, '秋分の日');

            // 国民の休日 (9月の敬老の日と秋分の日に挟まれた日)
            const sept3rdMon = getNthDayOfWeek(year, 9, 1, 3);
            const autumnStr = `${year}-09-${String(autumnDay).padStart(2, '0')}`;
            const diff = (new Date(autumnStr) - new Date(sept3rdMon)) / (1000 * 60 * 60 * 24);
            if (diff === 2) {
                const d = new Date(sept3rdMon);
                d.setDate(d.getDate() + 1);
                add(`${year}-09-${String(d.getDate()).padStart(2, '0')}`, '国民の休日');
            }

            // 振替休日 (日曜日の祝日 -> 翌平日)
            const sortedDates = Object.keys(holidays).sort();
            const substituteHolidays = {};
            for (const dateStr of sortedDates) {
                const parts = dateStr.split('-');
                const d = new Date(parts[0], parts[1] - 1, parts[2]);
                if (d.getDay() === 0) { // 日曜日
                    let next = new Date(d);
                    next.setDate(next.getDate() + 1);
                    let nextStr = `${next.getFullYear()}-${String(next.getMonth() + 1).padStart(2, '0')}-${String(next.getDate()).padStart(2, '0')}`;
                    while (holidays[nextStr] || substituteHolidays[nextStr]) {
                        next.setDate(next.getDate() + 1);
                        nextStr = `${next.getFullYear()}-${String(next.getMonth() + 1).padStart(2, '0')}-${String(next.getDate()).padStart(2, '0')}`;
                    }
                    substituteHolidays[nextStr] = '振替休日';
                }
            }

            return { ...holidays, ...substituteHolidays };
        }

        // ==========================================
        // ローカルストレージ管理 (サーバー不要)
        // ==========================================
        const STORAGE_KEY = 'standalone_dark_calendar_events';

        function loadEvents() {
            try {
                const json = localStorage.getItem(STORAGE_KEY);
                if (json) {
                    return JSON.parse(json);
                }
            } catch (err) {
                console.error('Failed to load events from localStorage:', err);
            }

            // 初回サンプルデータ
            const today = new Date();
            const y = today.getFullYear();
            const m = String(today.getMonth() + 1).padStart(2, '0');
            const d = String(today.getDate()).padStart(2, '0');
            
            const samples = [
                {
                    id: 'sample_1',
                    title: 'プロジェクトキックオフ',
                    description: '新規プロジェクトの概要説明・スケジュール確認',
                    event_date: `${y}-${m}-${d}`,
                    start_time: '13:00',
                    end_time: '14:30',
                    color: '#3b82f6',
                    is_all_day: 0
                },
                {
                    id: 'sample_2',
                    title: '歯医者定期検診',
                    description: '駅前クリニック',
                    event_date: `${y}-${m}-${String(Math.min(28, today.getDate() + 3)).padStart(2, '0')}`,
                    start_time: '18:00',
                    end_time: '19:00',
                    color: '#10b981',
                    is_all_day: 0
                },
                {
                    id: 'sample_3',
                    title: '週末温泉旅行',
                    description: '箱根温泉でリフレッシュ',
                    event_date: `${y}-${m}-${String(Math.min(28, today.getDate() + 6)).padStart(2, '0')}`,
                    start_time: '',
                    end_time: '',
                    color: '#8b5cf6',
                    is_all_day: 1
                }
            ];
            saveEvents(samples);
            return samples;
        }

        function saveEvents(events) {
            try {
                localStorage.setItem(STORAGE_KEY, JSON.stringify(events));
            } catch (err) {
                console.error('Failed to save events to localStorage:', err);
                showToast('保存に失敗しました', true);
            }
        }

        // ==========================================
        // アプリケーション状態
        // ==========================================
        let currentDate = new Date();
        let viewMode = 'grid'; // 'grid' | 'list'
        let allEvents = loadEvents();
        let currentHolidays = {};

        // DOM要素
        const currentMonthYearLabel = document.getElementById('currentMonthYearLabel');
        const prevBtn = document.getElementById('prevBtn');
        const nextBtn = document.getElementById('nextBtn');
        const todayBtn = document.getElementById('todayBtn');
        const calendarDaysGrid = document.getElementById('calendarDaysGrid');
        const viewGridBtn = document.getElementById('viewGridBtn');
        const viewListBtn = document.getElementById('viewListBtn');
        const calendarViewContainer = document.getElementById('calendarViewContainer');
        const listViewContainer = document.getElementById('listViewContainer');
        const eventsListContent = document.getElementById('eventsListContent');
        const listTitle = document.getElementById('listTitle');
        const searchInput = document.getElementById('searchInput');

        // モーダル要素
        const eventModal = document.getElementById('eventModal');
        const modalTitle = document.getElementById('modalTitle');
        const eventForm = document.getElementById('eventForm');
        const eventId = document.getElementById('eventId');
        const eventTitle = document.getElementById('eventTitle');
        const eventDate = document.getElementById('eventDate');
        const isAllDay = document.getElementById('isAllDay');
        const timeInputsContainer = document.getElementById('timeInputsContainer');
        const startTime = document.getElementById('startTime');
        const endTime = document.getElementById('endTime');
        const eventDescription = document.getElementById('eventDescription');
        const openNewModalBtn = document.getElementById('openNewModalBtn');
        const closeModalBtn = document.getElementById('closeModalBtn');
        const cancelModalBtn = document.getElementById('cancelModalBtn');
        const deleteEventBtn = document.getElementById('deleteEventBtn');

        // エクスポート / インポート
        const exportBtn = document.getElementById('exportBtn');
        const importBtn = document.getElementById('importBtn');
        const importFileInput = document.getElementById('importFileInput');

        // トースト通知
        function showToast(message, isError = false) {
            const toast = document.getElementById('toast');
            const toastMsg = document.getElementById('toastMsg');
            const toastIcon = document.getElementById('toastIcon');

            toastMsg.textContent = message;
            if (isError) {
                toastIcon.setAttribute('data-lucide', 'alert-circle');
                toastIcon.className = 'w-5 h-5 text-rose-400';
            } else {
                toastIcon.setAttribute('data-lucide', 'check-circle');
                toastIcon.className = 'w-5 h-5 text-emerald-400';
            }
            lucide.createIcons();

            toast.classList.remove('translate-y-20', 'opacity-0');
            toast.classList.add('translate-y-0', 'opacity-100');

            setTimeout(() => {
                toast.classList.remove('translate-y-0', 'opacity-100');
                toast.classList.add('translate-y-20', 'opacity-0');
            }, 3000);
        }

        // 終日チェックボックス
        isAllDay.addEventListener('change', () => {
            if (isAllDay.checked) {
                timeInputsContainer.classList.add('hidden');
                startTime.value = '';
                endTime.value = '';
            } else {
                timeInputsContainer.classList.remove('hidden');
                if (!startTime.value) startTime.value = '10:00';
                if (!endTime.value) endTime.value = '11:00';
            }
        });

        // 画面の更新
        function updateView() {
            const year = currentDate.getFullYear();
            const month = currentDate.getMonth() + 1;

            currentMonthYearLabel.textContent = `${year}年 ${month}月`;
            listTitle.textContent = `${year}年 ${month}月の予定一覧`;

            // 前年・当年・翌年の祝日をまとめて計算
            currentHolidays = {
                ...getJapaneseHolidays(year - 1),
                ...getJapaneseHolidays(year),
                ...getJapaneseHolidays(year + 1),
            };

            renderCalendar();
            renderList();
        }

        // カレンダーグリッド描画
        function renderCalendar() {
            calendarDaysGrid.innerHTML = '';

            const year = currentDate.getFullYear();
            const month = currentDate.getMonth();

            const firstDayIndex = new Date(year, month, 1).getDay();
            const lastDay = new Date(year, month + 1, 0).getDate();
            const prevLastDay = new Date(year, month, 0).getDate();

            const today = new Date();
            const todayStr = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`;

            // 前月
            for (let i = firstDayIndex; i > 0; i--) {
                const dayNum = prevLastDay - i + 1;
                const prevMonth = month === 0 ? 12 : month;
                const prevYear = month === 0 ? year - 1 : year;
                const dateStr = `${prevYear}-${String(prevMonth).padStart(2, '0')}-${String(dayNum).padStart(2, '0')}`;
                calendarDaysGrid.appendChild(createDayCell(dateStr, dayNum, true));
            }

            // 当月
            for (let dayNum = 1; dayNum <= lastDay; dayNum++) {
                const dateStr = `${year}-${String(month + 1).padStart(2, '0')}-${String(dayNum).padStart(2, '0')}`;
                calendarDaysGrid.appendChild(createDayCell(dateStr, dayNum, false, dateStr === todayStr));
            }

            // 翌月
            const totalCells = firstDayIndex + lastDay;
            const nextDays = (7 - (totalCells % 7)) % 7;
            for (let dayNum = 1; dayNum <= nextDays; dayNum++) {
                const nextMonth = month === 11 ? 1 : month + 2;
                const nextYear = month === 11 ? year + 1 : year;
                const dateStr = `${nextYear}-${String(nextMonth).padStart(2, '0')}-${String(dayNum).padStart(2, '0')}`;
                calendarDaysGrid.appendChild(createDayCell(dateStr, dayNum, true));
            }

            lucide.createIcons();
        }

        // 日付セル生成
        function createDayCell(dateStr, dayNum, isOtherMonth, isToday = false) {
            const cell = document.createElement('div');
            cell.className = `calendar-cell p-1.5 sm:p-2.5 flex flex-col justify-between transition-colors duration-150 group cursor-pointer ${
                isOtherMonth 
                    ? 'bg-slate-950/40 text-slate-600' 
                    : 'bg-slate-900/40 hover:bg-slate-800/50'
            }`;

            cell.addEventListener('click', () => {
                openEventModalForNew(dateStr);
            });

            // ヘッダー (日付番号 & 祝日)
            const header = document.createElement('div');
            header.className = 'flex items-start justify-between mb-1';

            const dayLabel = document.createElement('span');
            dayLabel.className = `text-xs sm:text-sm font-bold rounded-lg w-6 h-6 sm:w-7 sm:h-7 flex items-center justify-center transition ${
                isToday 
                    ? 'bg-gradient-to-tr from-indigo-500 to-purple-600 text-white shadow-md shadow-indigo-500/30 ring-2 ring-indigo-400/50' 
                    : isOtherMonth 
                        ? 'text-slate-600' 
                        : 'text-slate-300 group-hover:text-white group-hover:bg-slate-800/80'
            }`;
            dayLabel.textContent = dayNum;

            const parts = dateStr.split('-');
            const cellDate = new Date(parts[0], parts[1] - 1, parts[2]);
            const dayOfWeek = cellDate.getDay();
            if (!isToday && !isOtherMonth) {
                if (dayOfWeek === 0) dayLabel.classList.add('text-rose-400');
                if (dayOfWeek === 6) dayLabel.classList.add('text-sky-400');
            }

            header.appendChild(dayLabel);

            // 祝日
            const holidayName = currentHolidays[dateStr];
            if (holidayName) {
                const holidayBadge = document.createElement('span');
                holidayBadge.className = 'text-[10px] sm:text-xs text-rose-400 bg-rose-950/50 border border-rose-900/60 px-1.5 py-0.5 rounded font-medium truncate max-w-[70px] sm:max-w-[90px]';
                holidayBadge.title = holidayName;
                holidayBadge.textContent = holidayName;
                header.appendChild(holidayBadge);
                if (!isToday && !isOtherMonth) {
                    dayLabel.classList.remove('text-slate-300', 'text-sky-400');
                    dayLabel.classList.add('text-rose-400');
                }
            }

            cell.appendChild(header);

            // イベント一覧
            const eventsContainer = document.createElement('div');
            eventsContainer.className = 'space-y-1 overflow-y-auto max-h-[85px] sm:max-h-[110px] flex-1';

            const dayEvents = allEvents.filter(e => e.event_date === dateStr);
            // ソート: 終日優先、次に開始時刻順
            dayEvents.sort((a, b) => {
                if (a.is_all_day !== b.is_all_day) return b.is_all_day - a.is_all_day;
                return (a.start_time || '').localeCompare(b.start_time || '');
            });

            dayEvents.forEach(event => {
                const badge = document.createElement('div');
                badge.className = 'event-badge text-[11px] sm:text-xs px-2 py-0.5 rounded-md font-medium truncate flex items-center space-x-1.5 cursor-pointer shadow-sm border border-white/5';
                
                badge.style.backgroundColor = `${event.color}25`;
                badge.style.color = '#f8fafc';
                badge.style.borderLeft = `3.5px solid ${event.color}`;

                let timeText = '';
                if (!event.is_all_day && event.start_time) {
                    timeText = `<span class="opacity-75 font-mono text-[10px] text-slate-300">${event.start_time}</span>`;
                }

                badge.innerHTML = `${timeText}<span class="truncate">${escapeHtml(event.title)}</span>`;
                badge.title = `${event.title}${event.start_time ? ' (' + event.start_time + ')' : ''}`;

                badge.addEventListener('click', (e) => {
                    e.stopPropagation();
                    openEventModalForEdit(event);
                });

                eventsContainer.appendChild(badge);
            });

            cell.appendChild(eventsContainer);
            return cell;
        }

        // リストビュー描画
        function renderList() {
            eventsListContent.innerHTML = '';
            const query = searchInput.value.toLowerCase().trim();

            const year = currentDate.getFullYear();
            const month = currentDate.getMonth() + 1;
            const prefix = `${year}-${String(month).padStart(2, '0')}`;

            // 当月のイベント、または検索クエリがある場合は全体からヒット
            const filteredEvents = allEvents.filter(ev => {
                const matchMonth = query ? true : ev.event_date.startsWith(prefix);
                if (!matchMonth) return false;
                if (!query) return true;
                return ev.title.toLowerCase().includes(query) || (ev.description && ev.description.toLowerCase().includes(query));
            });

            if (filteredEvents.length === 0) {
                eventsListContent.innerHTML = `
                    <div class="py-16 text-center text-slate-500">
                        <i data-lucide="calendar-x" class="w-12 h-12 mx-auto mb-3 opacity-40"></i>
                        <p class="text-sm font-medium">${query ? '該当する予定は見つかりませんでした' : '今月の予定はありません'}</p>
                    </div>
                `;
                lucide.createIcons();
                return;
            }

            // 日付ごとにグループ化
            const grouped = {};
            filteredEvents.forEach(ev => {
                if (!grouped[ev.event_date]) grouped[ev.event_date] = [];
                grouped[ev.event_date].push(ev);
            });

            const sortedDates = Object.keys(grouped).sort();

            sortedDates.forEach(dateStr => {
                const parts = dateStr.split('-');
                const d = new Date(parts[0], parts[1] - 1, parts[2]);
                const dayOfWeekStr = ['日', '月', '火', '水', '木', '金', '土'][d.getDay()];
                const holiday = currentHolidays[dateStr];

                const dayGroup = document.createElement('div');
                dayGroup.className = 'border border-slate-800 rounded-xl overflow-hidden bg-slate-950/40 mb-3.5';

                let dayHeaderClass = 'text-slate-300';
                if (d.getDay() === 0 || holiday) dayHeaderClass = 'text-rose-400 font-semibold';
                else if (d.getDay() === 6) dayHeaderClass = 'text-sky-400 font-semibold';

                dayGroup.innerHTML = `
                    <div class="px-4 py-2.5 bg-slate-900/80 border-b border-slate-800 flex items-center justify-between">
                        <div class="flex items-center space-x-2.5">
                            <span class="font-bold text-sm ${dayHeaderClass}">${dateStr} (${dayOfWeekStr})</span>
                            ${holiday ? `<span class="text-xs bg-rose-950/60 text-rose-400 border border-rose-900/50 px-2 py-0.5 rounded-full">${holiday}</span>` : ''}
                        </div>
                        <button class="text-xs text-indigo-400 hover:text-indigo-300 font-medium add-date-btn transition" data-date="${dateStr}">
                            + 予定追加
                        </button>
                    </div>
                    <div class="p-2.5 space-y-2 bg-slate-950/20"></div>
                `;

                const eventListDiv = dayGroup.querySelector('.space-y-2');
                grouped[dateStr].forEach(ev => {
                    const item = document.createElement('div');
                    item.className = 'p-3 rounded-xl border border-slate-800/80 hover:border-slate-700 hover:bg-slate-850/80 transition cursor-pointer flex items-center justify-between bg-slate-900/60';
                    
                    let timeLabel = ev.is_all_day ? '終日' : `${ev.start_time || ''} ${ev.end_time ? '〜 ' + ev.end_time : ''}`;

                    item.innerHTML = `
                        <div class="flex items-center space-x-3.5">
                            <span class="w-3 h-3 rounded-full shadow-sm" style="background-color: ${ev.color}; box-shadow: 0 0 8px ${ev.color}80;"></span>
                            <div>
                                <h4 class="text-sm font-semibold text-slate-100">${escapeHtml(ev.title)}</h4>
                                ${ev.description ? `<p class="text-xs text-slate-400 mt-0.5">${escapeHtml(ev.description)}</p>` : ''}
                            </div>
                        </div>
                        <div class="text-right">
                            <span class="text-xs font-mono font-medium text-slate-300 bg-slate-800/90 border border-slate-700/60 px-2.5 py-1 rounded-lg">${timeLabel}</span>
                        </div>
                    `;

                    item.addEventListener('click', () => openEventModalForEdit(ev));
                    eventListDiv.appendChild(item);
                });

                dayGroup.querySelector('.add-date-btn').addEventListener('click', () => {
                    openEventModalForNew(dateStr);
                });

                eventsListContent.appendChild(dayGroup);
            });

            lucide.createIcons();
        }

        // 新規登録モーダル
        function openEventModalForNew(defaultDate = null) {
            modalTitle.innerHTML = `<i data-lucide="calendar-plus" class="w-5 h-5 text-indigo-400"></i><span>予定を追加</span>`;
            deleteEventBtn.classList.add('hidden');
            eventForm.reset();
            eventId.value = '';

            const targetDate = defaultDate || new Date().toISOString().split('T')[0];
            eventDate.value = targetDate;
            isAllDay.checked = true;
            timeInputsContainer.classList.add('hidden');

            document.querySelector('input[name="color"][value="#3b82f6"]').checked = true;

            lucide.createIcons();
            eventModal.classList.remove('hidden');
            setTimeout(() => eventTitle.focus(), 50);
        }

        // 編集モーダル
        function openEventModalForEdit(event) {
            modalTitle.innerHTML = `<i data-lucide="edit" class="w-5 h-5 text-indigo-400"></i><span>予定を編集</span>`;
            deleteEventBtn.classList.remove('hidden');
            
            eventId.value = event.id;
            eventTitle.value = event.title;
            eventDate.value = event.event_date;
            isAllDay.checked = !!event.is_all_day;
            startTime.value = event.start_time || '';
            endTime.value = event.end_time || '';
            eventDescription.value = event.description || '';

            if (isAllDay.checked) {
                timeInputsContainer.classList.add('hidden');
            } else {
                timeInputsContainer.classList.remove('hidden');
            }

            const colorRadio = document.querySelector(`input[name="color"][value="${event.color}"]`);
            if (colorRadio) {
                colorRadio.checked = true;
            } else {
                document.querySelector('input[name="color"][value="#3b82f6"]').checked = true;
            }

            lucide.createIcons();
            eventModal.classList.remove('hidden');
            setTimeout(() => eventTitle.focus(), 50);
        }

        // モーダルを閉じる
        function closeModal() {
            eventModal.classList.add('hidden');
            eventForm.reset();
            eventId.value = '';
        }

        // 予定の保存 (新規または更新)
        eventForm.addEventListener('submit', (e) => {
            e.preventDefault();

            const id = eventId.value || 'evt_' + Date.now();
            const payload = {
                id: id,
                title: eventTitle.value.trim(),
                event_date: eventDate.value,
                is_all_day: isAllDay.checked ? 1 : 0,
                start_time: startTime.value,
                end_time: endTime.value,
                color: document.querySelector('input[name="color"]:checked')?.value || '#3b82f6',
                description: eventDescription.value.trim()
            };

            const existingIndex = allEvents.findIndex(ev => ev.id === id);
            if (existingIndex >= 0) {
                allEvents[existingIndex] = payload;
                showToast('予定を更新しました');
            } else {
                allEvents.push(payload);
                showToast('予定を登録しました');
            }

            saveEvents(allEvents);
            closeModal();
            updateView();
        });

        // 予定の削除
        deleteEventBtn.addEventListener('click', () => {
            const id = eventId.value;
            if (!id) return;
            if (!confirm('この予定を削除してもよろしいですか？')) return;

            allEvents = allEvents.filter(ev => ev.id !== id);
            saveEvents(allEvents);
            showToast('予定を削除しました');
            closeModal();
            updateView();
        });

        // ナビゲーション操作
        prevBtn.addEventListener('click', () => {
            currentDate.setMonth(currentDate.getMonth() - 1);
            updateView();
        });

        nextBtn.addEventListener('click', () => {
            currentDate.setMonth(currentDate.getMonth() + 1);
            updateView();
        });

        todayBtn.addEventListener('click', () => {
            currentDate = new Date();
            updateView();
        });

        openNewModalBtn.addEventListener('click', () => openEventModalForNew());
        closeModalBtn.addEventListener('click', closeModal);
        cancelModalBtn.addEventListener('click', closeModal);

        eventModal.addEventListener('click', (e) => {
            if (e.target === eventModal) closeModal();
        });

        window.addEventListener('keydown', (e) => {
            if (e.key === 'Escape' && !eventModal.classList.contains('hidden')) {
                closeModal();
            }
        });

        // 表示切り替え
        viewGridBtn.addEventListener('click', () => {
            viewMode = 'grid';
            viewGridBtn.className = 'px-3 py-1.5 rounded-lg bg-indigo-600 text-white shadow-md shadow-indigo-600/30 flex items-center space-x-1.5 transition font-semibold';
            viewListBtn.className = 'px-3 py-1.5 rounded-lg hover:text-white hover:bg-slate-800/60 flex items-center space-x-1.5 transition font-semibold text-slate-400';
            calendarViewContainer.classList.remove('hidden');
            listViewContainer.classList.add('hidden');
        });

        viewListBtn.addEventListener('click', () => {
            viewMode = 'list';
            viewListBtn.className = 'px-3 py-1.5 rounded-lg bg-indigo-600 text-white shadow-md shadow-indigo-600/30 flex items-center space-x-1.5 transition font-semibold';
            viewGridBtn.className = 'px-3 py-1.5 rounded-lg hover:text-white hover:bg-slate-800/60 flex items-center space-x-1.5 transition font-semibold text-slate-400';
            calendarViewContainer.classList.add('hidden');
            listViewContainer.classList.remove('hidden');
            renderList();
        });

        // 検索
        searchInput.addEventListener('input', () => {
            renderList();
        });

        // エクスポート
        exportBtn.addEventListener('click', () => {
            const dataStr = 'data:text/json;charset=utf-8,' + encodeURIComponent(JSON.stringify(allEvents, null, 2));
            const downloadAnchor = document.createElement('a');
            downloadAnchor.setAttribute('href', dataStr);
            downloadAnchor.setAttribute('download', `calendar_backup_${new Date().toISOString().split('T')[0]}.json`);
            document.body.appendChild(downloadAnchor);
            downloadAnchor.click();
            downloadAnchor.remove();
            showToast('予定データをエクスポートしました');
        });

        // インポート
        importBtn.addEventListener('click', () => {
            importFileInput.click();
        });

        importFileInput.addEventListener('change', (e) => {
            const file = e.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = (event) => {
                try {
                    const imported = JSON.parse(event.target.result);
                    if (Array.isArray(imported)) {
                        allEvents = imported;
                        saveEvents(allEvents);
                        updateView();
                        showToast(`${imported.length} 件の予定を復元しました`);
                    } else {
                        throw new Error('データの形式が不正です');
                    }
                } catch (err) {
                    alert('ファイルの読み込みに失敗しました: ' + err.message);
                }
            };
            reader.readAsText(file);
            importFileInput.value = '';
        });

        // HTMLエスケープヘルパー
        function escapeHtml(str) {
            if (!str) return '';
            return str
                .replace(/&/g, '&amp;')
                .replace(/</g, '&lt;')
                .replace(/>/g, '&gt;')
                .replace(/"/g, '&quot;')
                .replace(/'/g, '&#039;');
        }

        // 初回ロード
        updateView();
    </script>
</body>
</html>
