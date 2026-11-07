<!DOCTYPE html>
<html lang="pt-BR" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel Meteorológico  — Erechim / RS</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Chart.js UMD Script for global Chart constructor support -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=JetBrains+Mono:wght@400;600;700&display=swap" rel="stylesheet">
    
    <script>
        // Global handler functions defined early to prevent ReferenceErrors during DOM rendering
        function handleSatelliteError(img) {
            if (typeof satSources !== 'undefined' && satSources[currentSatChannel] && satSources[currentSatChannel].length > 1) {
                img.src = satSources[currentSatChannel][1];
            } else {
                if (typeof switchSatChannel === 'function') {
                    switchSatChannel('interactive');
                }
            }
        }

        function reloadSatelliteImage() {
            const img = document.getElementById('satellite-img');
            const iframe = document.getElementById('sat-iframe-view');
            if (typeof currentSatChannel !== 'undefined' && currentSatChannel === 'interactive') {
                if (img) img.classList.add('hidden');
                if (iframe) {
                    iframe.classList.remove('hidden');
                    iframe.src = satSources.interactive[0];
                }
                return;
            }
            if (iframe) iframe.classList.add('hidden');
            if (img) {
                img.classList.remove('hidden');
                img.src = satSources[currentSatChannel][0] + `?t=${new Date().getTime()}`;
            }
        }
    </script>

    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        brand: {
                            50: '#f0f9ff',
                            500: '#0284c7',
                            900: '#0c4a6e',
                        }
                    },
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                        mono: ['JetBrains Mono', 'monospace'],
                    }
                }
            }
        }
    </script>
    <style>
        body {
            transition: background-color 0.3s, color 0.3s;
            background-color: #090d16;
            color: #f8fafc;
        }
        .dark body {
            background-color: #090d16;
            color: #f8fafc;
        }
        .light body {
            background-color: #f1f5f9;
            color: #0f172a;
        }
        
        .glass-card {
            backdrop-filter: blur(12px);
            transition: all 0.2s ease-in-out;
        }
        .dark .glass-card {
            background: rgba(15, 23, 42, 0.75);
            border: 1px solid rgba(255, 255, 255, 0.08);
        }
        .light .glass-card {
            background: rgba(255, 255, 255, 0.85);
            border: 1px solid rgba(0, 0, 0, 0.08);
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
        }

        .pulse-danger {
            animation: pulse-danger-anim 2s infinite;
        }
        @keyframes pulse-danger-anim {
            0% { box-shadow: 0 0 0 0px rgba(255, 40, 0, 0.9); }
            50% { box-shadow: 0 0 0 16px rgba(255, 40, 0, 0); }
            100% { box-shadow: 0 0 0 0px rgba(255, 40, 0, 0); }
        }

        .border-blink {
            animation: blink-border-anim 1.2s infinite ease-in-out;
        }
        @keyframes blink-border-anim {
            0%, 100% { border-color: #ff2800; }
            50% { border-color: #ffffff; }
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between selection:bg-sky-500 selection:text-white">

    <!-- HEADER BAR -->
    <header class="border-b dark:border-slate-800 border-slate-200 dark:bg-slate-900/95 bg-white/95 backdrop-blur-md relative z-50 shadow-md">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3 sm:py-4">
            <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between gap-4">
                
                <!-- Title & Station Info -->
                <div class="flex items-center justify-between lg:justify-start gap-3">
                    <div class="flex items-center gap-3">
                        <span class="p-3 bg-sky-500/10 text-sky-500 rounded-2xl border border-sky-500/20 shadow-inner">
                            <i class="fa-solid fa-shield-halved text-3xl"></i>
                        </span>
                        <div>
                            <h1 class="text-xl sm:text-3xl lg:text-4xl font-black tracking-tight dark:text-white text-slate-900 flex flex-wrap items-center gap-2">
                                Central de Monitoramento Meteorológico de Erechim
                                <span class="text-xs sm:text-sm font-extrabold px-3 py-1 rounded-full bg-sky-500/15 text-sky-400 border border-sky-500/30 uppercase tracking-wide">AO VIVO</span>
                            </h1>
                            <p class="text-xs sm:text-sm dark:text-slate-400 text-slate-500 flex flex-wrap gap-x-4 gap-y-1 mt-1">
                                <span><i class="fa-solid fa-location-dot text-rose-500 mr-1"></i> Lat: 27°38'02"S | Lon: 52°16'26"O</span>
                                <span><i class="fa-solid fa-mountain text-emerald-500 mr-1"></i> 783m</span>
                                <span><i class="fa-solid fa-tower-observation text-amber-500 mr-1"></i> INMET: <strong>A828</strong></span>
                            </p>
                        </div>
                    </div>
                </div>

                <!-- INDICADOR DE TENDÊNCIA E ALERTA DAS PRÓXIMAS 6 HORAS -->
                <div id="header-6h-alert" class="flex-1 max-w-xl mx-auto lg:mx-4 px-4 py-3 rounded-xl bg-emerald-600/20 border-2 border-emerald-500 text-emerald-100 transition shadow-lg flex flex-row items-center justify-between gap-3">
                    <div class="flex items-center gap-2">
                        <i class="fa-solid fa-clock-rotate-left text-emerald-400 text-base"></i>
                        <span class="text-xs sm:text-sm font-black uppercase tracking-wider text-emerald-300">Próximas 6 horas:</span>
                    </div>
                    <div id="content-6h-alert" class="text-xs sm:text-sm font-bold text-emerald-100 flex-1 truncate">
                        <span>Analisando condições meteorológicas...</span>
                    </div>
                    <span id="label-6h-span" class="text-[10px] sm:text-xs font-mono font-bold px-2.5 py-1 rounded bg-emerald-500 text-slate-950 uppercase shrink-0">SEM ALERTA</span>
                </div>

                <!-- Admin & Controls -->
                <div class="flex items-center justify-between lg:justify-end gap-3 border-t lg:border-t-0 pt-3 lg:pt-0 dark:border-slate-800 border-slate-200">
                    
                    <!-- New Page / Modal Button for Historical & Statistical Data -->
                    <button onclick="toggleHistoryPage(true)" class="flex items-center gap-2 px-3 py-1.5 rounded-xl border text-xs font-semibold transition dark:bg-slate-800 dark:border-slate-700 dark:hover:bg-slate-700 dark:text-slate-200 bg-slate-100 border-slate-300 hover:bg-slate-200 text-slate-700">
                        <i class="fa-solid fa-clock-rotate-left text-amber-500"></i>
                        <span>Séries & Histórico</span>
                    </button>

                    <!-- Admin Login / Status Trigger -->
                    <div id="admin-status-area">
                        <button onclick="toggleAdminModal()" class="flex items-center gap-2 px-3 py-1.5 rounded-xl border text-xs font-semibold transition dark:bg-slate-800 dark:border-slate-700 dark:hover:bg-slate-700 dark:text-slate-200 bg-slate-100 border-slate-300 hover:bg-slate-200 text-slate-700">
                            <i class="fa-solid fa-user-shield text-sky-500"></i>
                            <span id="admin-btn-text">Área Administrativa</span>
                        </button>
                    </div>

                    <!-- Theme Toggle -->
                    <button onclick="toggleTheme()" class="p-2 rounded-xl dark:bg-slate-800 dark:text-amber-400 bg-slate-100 text-slate-700 hover:opacity-80 transition" title="Alternar Tema">
                        <i id="theme-icon" class="fa-solid fa-moon"></i>
                    </button>

                    <!-- Refresh Button -->
                    <button id="btn-refresh" onclick="fetchWeatherData()" class="flex items-center gap-2 px-3.5 py-1.5 bg-sky-600 hover:bg-sky-500 text-white font-medium text-xs rounded-xl shadow-md transition active:scale-95">
                        <i id="refresh-icon" class="fa-solid fa-rotate-right"></i>
                        <span class="hidden sm:inline">Atualizar</span>
                    </button>
                </div>

            </div>
        </div>
    </header>

    <!-- ADMIN MODAL & AUTHENTICATION -->
    <div id="admin-modal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md z-50 hidden flex items-center justify-center p-4">
        <div class="glass-card w-full max-w-2xl rounded-2xl p-6 relative border dark:border-slate-700 border-slate-300 shadow-2xl">
            <button onclick="toggleAdminModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white p-2">
                <i class="fa-solid fa-xmark text-lg"></i>
            </button>

            <!-- Login View -->
            <div id="admin-login-view" class="space-y-4">
                <div class="flex items-center gap-3">
                    <span class="p-3 bg-sky-500/10 text-sky-400 rounded-xl border border-sky-500/20">
                        <i class="fa-solid fa-lock text-xl"></i>
                    </span>
                    <div>
                        <h2 class="text-xl font-bold dark:text-white text-slate-900">Acesso Restrito — Administrador</h2>
                        <p class="text-xs text-slate-400">Entre com credenciais autorizadas de gestão do sistema.</p>
                    </div>
                </div>

                <form onsubmit="handleAdminLogin(event)" class="space-y-4 mt-4">
                    <div>
                        <label class="block text-xs font-semibold uppercase text-slate-400 mb-1">Nome do Gestor / Administrador</label>
                        <input type="text" id="login-name" value="Wander Luís Marques" placeholder="Digite seu nome completo" required class="w-full px-4 py-2.5 rounded-xl dark:bg-slate-900 dark:border-slate-700 dark:text-white bg-slate-50 border-slate-300 text-slate-900 text-sm focus:ring-2 focus:ring-sky-500 outline-none">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold uppercase text-slate-400 mb-1">Senha de Acesso Exclusiva</label>
                        <input type="password" id="login-pass" placeholder="••••••••" required class="w-full px-4 py-2.5 rounded-xl dark:bg-slate-900 dark:border-slate-700 dark:text-white bg-slate-50 border-slate-300 text-slate-900 text-sm focus:ring-2 focus:ring-sky-500 outline-none">
                        <span class="text-[10px] text-slate-500 mt-1 block"><i class="fa-solid fa-lock text-amber-500 mr-1"></i>Acesso restrito estritamente ao administrador autorizado.</span>
                    </div>
                    <div id="login-error" class="hidden text-xs text-red-400 bg-red-950/40 p-2.5 rounded-xl border border-red-800">
                        Credenciais ou nome de usuário inválidos. Acesso negado.
                    </div>
                    <button type="submit" class="w-full py-3 bg-sky-600 hover:bg-sky-500 text-white font-bold text-sm rounded-xl transition shadow-lg">
                        Entrar no Painel Administrativo
                    </button>
                </form>
            </div>

            <!-- Dashboard Controls -->
            <div id="admin-panel-view" class="hidden space-y-6">
                <div class="flex items-center justify-between border-b dark:border-slate-800 border-slate-200 pb-4">
                    <div class="flex items-center gap-3">
                        <div class="w-10 h-10 rounded-full bg-sky-500/20 text-sky-400 border border-sky-500/40 flex items-center justify-center font-bold">
                            WM
                        </div>
                        <div>
                            <h3 id="admin-user-disp" class="font-bold text-sm dark:text-white text-slate-900">Wander Luís Marques</h3>
                            <span class="text-[10px] text-emerald-400 font-semibold flex items-center gap-1">
                                <i class="fa-solid fa-circle text-[8px]"></i> Administrador do Sistema Ativo
                            </span>
                        </div>
                    </div>
                    <button onclick="handleAdminLogout()" class="px-3 py-1.5 text-xs bg-red-900/40 hover:bg-red-800/60 text-red-200 rounded-lg border border-red-700 transition">
                        Sair
                    </button>
                </div>

                <!-- Admin Control Tabs -->
                <div class="space-y-4">
                    <h4 class="text-xs uppercase tracking-wider font-bold text-slate-400">Controles do Gestor:</h4>
                    
                    <!-- Emergency Override Alert -->
                    <div class="p-4 rounded-xl dark:bg-slate-900/80 bg-slate-100 border border-amber-500/30 space-y-3">
                        <label class="text-xs font-bold text-amber-400 flex items-center gap-2">
                            <i class="fa-solid fa-bullhorn"></i> Emitir Comunicado/Alerta Manual de Emergência
                        </label>
                        <textarea id="admin-custom-alert-text" rows="2" placeholder="Digite o aviso de utilidade pública para a população..." class="w-full p-3 rounded-lg dark:bg-slate-950 dark:text-white bg-white text-slate-900 border dark:border-slate-700 border-slate-300 text-xs outline-none"></textarea>
                        <div class="flex items-center justify-between">
                            <label class="flex items-center gap-2 text-slate-700 dark:text-slate-300 cursor-pointer text-xs">
                                <input type="checkbox" id="admin-override-active" class="rounded border-slate-700 text-sky-500 focus:ring-0">
                                Ativar Banner de Emergência Manual
                            </label>
                            <button onclick="saveAdminBannerOverride()" class="px-3 py-1.5 bg-amber-600 hover:bg-amber-500 text-white font-semibold text-xs rounded-lg transition">
                                Publicar Comunicado
                            </button>
                        </div>
                    </div>

                    <!-- Config threshold metrics -->
                    <div class="p-4 rounded-xl dark:bg-slate-900/80 bg-slate-100 border dark:border-slate-800 border-slate-200 space-y-3">
                        <h5 class="text-xs font-bold text-slate-300">Ajustar Gatilhos de Alerta Automático</h5>
                        <div class="grid grid-cols-2 gap-3 text-xs">
                            <div>
                                <span class="text-slate-400 block mb-1">Temp Mínima Frio (°C):</span>
                                <input type="number" id="cfg-temp-min" value="5" class="w-full px-3 py-1.5 rounded dark:bg-slate-950 bg-white border dark:border-slate-700 border-slate-300">
                            </div>
                            <div>
                                <span class="text-slate-400 block mb-1">Temp Máxima Calor (°C):</span>
                                <input type="number" id="cfg-temp-max" value="28" class="w-full px-3 py-1.5 rounded dark:bg-slate-950 bg-white border dark:border-slate-700 border-slate-300">
                            </div>
                            <div>
                                <span class="text-slate-400 block mb-1">Chuva (mm/12h):</span>
                                <input type="number" id="cfg-rain-limit" value="30" class="w-full px-3 py-1.5 rounded dark:bg-slate-950 bg-white border dark:border-slate-700 border-slate-300">
                            </div>
                            <div>
                                <span class="text-slate-400 block mb-1">Vento Limite (km/h):</span>
                                <input type="number" id="cfg-wind-limit" value="40" class="w-full px-3 py-1.5 rounded dark:bg-slate-950 bg-white border dark:border-slate-700 border-slate-300">
                            </div>
                        </div>
                        <button onclick="saveThresholdConfigs()" class="w-full py-2 bg-sky-600 hover:bg-sky-500 text-white font-bold text-xs rounded-lg transition mt-2">
                            Salvar Configurações de Gatilho
                        </button>
                    </div>
                </div>
            </div>

        </div>
    </div>

    <!-- DEDICATED HISTORICAL & STATISTICAL DATA PAGE / MODAL -->
    <div id="history-page-modal" class="fixed inset-0 bg-slate-950/90 backdrop-blur-md z-50 hidden flex flex-col overflow-y-auto p-4 sm:p-6 lg:p-8">
        <div class="max-w-7xl mx-auto w-full space-y-6 my-auto">
            
            <!-- Modal Header -->
            <div class="flex items-center justify-between glass-card p-4 sm:p-6 rounded-2xl border dark:border-slate-700 border-slate-300">
                <div class="flex items-center gap-3">
                    <span class="p-3 bg-amber-500/10 text-amber-400 rounded-xl border border-amber-500/20">
                        <i class="fa-solid fa-book-atlas text-2xl"></i>
                    </span>
                    <div>
                        <h2 class="text-xl sm:text-2xl font-extrabold dark:text-white text-slate-900">Séries Históricas & Eventos Extremos</h2>
                        <p class="text-xs sm:text-sm text-slate-400">Acervo climatológico e tabelas de probabilidade de Erechim / RS</p>
                    </div>
                </div>
                <button onclick="toggleHistoryPage(false)" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 text-white font-bold text-xs rounded-xl transition border border-slate-700 flex items-center gap-2">
                    <i class="fa-solid fa-xmark text-base"></i> Fechar / Voltar
                </button>
            </div>

            <!-- SÉRIES HISTÓRICAS E EVENTOS EXTREMOS INTRO SECTION -->
            <div class="glass-card rounded-2xl p-5 border dark:border-slate-700 border-slate-300 shadow-xl">
                <div class="flex items-center gap-3 mb-4 border-b dark:border-slate-800 border-slate-200 pb-3">
                    <span class="p-2.5 bg-amber-500/10 text-amber-400 rounded-xl border border-amber-500/20">
                        <i class="fa-solid fa-book-atlas text-xl"></i>
                    </span>
                    <div>
                        <h3 class="text-sm sm:text-base font-bold dark:text-white text-slate-900">Séries Históricas e Eventos Extremos</h3>
                        <p class="text-xs text-slate-400">Compilado climatológico e acervo estatístico de monitoramento em Erechim / RS</p>
                    </div>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 text-xs">
                    <div class="p-4 rounded-xl dark:bg-slate-900/80 bg-slate-100 border dark:border-slate-800 border-slate-200 space-y-2">
                        <span class="font-bold text-sky-400 flex items-center gap-1.5"><i class="fa-solid fa-temperature-arrow-down"></i> Recordes de Frio</span>
                        <p class="text-slate-400 leading-relaxed">Temperaturas mínimas históricas registradas abaixo de -4°C com geadas severas e episódios de neve esporádica na região do Alto Uruguai.</p>
                    </div>
                    <div class="p-4 rounded-xl dark:bg-slate-900/80 bg-slate-100 border dark:border-slate-800 border-slate-200 space-y-2">
                        <span class="font-bold text-blue-400 flex items-center gap-1.5"><i class="fa-solid fa-cloud-showers-heavy"></i> Precipitação Acumulada</span>
                        <p class="text-slate-400 leading-relaxed">Ciclos chuvosos severos com acumulados mensais superiores a 300mm associados a sistemas frontais persistentes e bloqueios atmosféricos.</p>
                    </div>
                    <div class="p-4 rounded-xl dark:bg-slate-900/80 bg-slate-100 border dark:border-slate-800 border-slate-200 space-y-2">
                        <span class="font-bold text-amber-400 flex items-center gap-1.5"><i class="fa-solid fa-wind"></i> Ventos e Rajadas</span>
                        <p class="text-slate-400 leading-relaxed">Linhas de instabilidade com rajadas frontais e convectivas ultrapassando 90 km/h, monitoradas em tempo real pelos radares e estações oficiais.</p>
                    </div>
                </div>
            </div>

            <!-- Statistical Table Section -->
            <div class="glass-card rounded-2xl p-5 border dark:border-slate-700 border-slate-300">
                <div class="flex items-center justify-between mb-4">
                    <h3 class="text-xs uppercase tracking-wider font-bold dark:text-slate-300 text-slate-700 flex items-center gap-2">
                        <i class="fa-solid fa-chart-pie text-emerald-400"></i> Probabilidade Estatística & Climatologia Histórica (Erechim / RS)
                    </h3>
                </div>
                <div class="overflow-x-auto">
                    <table class="w-full text-left text-xs dark:text-slate-300 text-slate-700">
                        <thead class="dark:bg-slate-900/90 bg-slate-200/80 dark:text-slate-400 text-slate-600 font-semibold uppercase tracking-wider border-b dark:border-slate-800 border-slate-300">
                            <tr>
                                <th class="py-3 px-4">Período / Estação</th>
                                <th class="py-3 px-4">Chuva Acumulada &gt; 150mm/mês</th>
                                <th class="py-3 px-4">Probab. Granizo Convectivo</th>
                                <th class="py-3 px-4">Rajadas de Vento &gt; 50 km/h</th>
                                <th class="py-3 px-4">Frio Extremo / Geada (&lt; 3°C)</th>
                                <th class="py-3 px-4">Calor Extremo (&gt; 30°C)</th>
                            </tr>
                        </thead>
                        <tbody class="divide-y dark:divide-slate-800 divide-slate-200 font-mono">
                            <tr class="hover:bg-slate-800/40 transition">
                                <td class="py-3 px-4 font-bold text-sky-400">Verão (Dez - Mar)</td>
                                <td class="py-3 px-4 text-emerald-400">68% <span class="text-[10px] text-slate-400 font-sans block">Chuvas Convectivas</span></td>
                                <td class="py-3 px-4 text-amber-400">22% <span class="text-[10px] text-slate-400 font-sans block">Risco Moderado</span></td>
                                <td class="py-3 px-4 text-amber-400">35% <span class="text-[10px] text-slate-400 font-sans block">Temporais de Verão</span></td>
                                <td class="py-3 px-4 text-slate-500">0% <span class="text-[10px] text-slate-400 font-sans block">Raro</span></td>
                                <td class="py-3 px-4 text-rose-400 font-bold">78% <span class="text-[10px] text-slate-400 font-sans block">Frequente</span></td>
                            </tr>
                            <tr class="hover:bg-slate-800/40 transition">
                                <td class="py-3 px-4 font-bold text-amber-400">Outono (Abr - Jun)</td>
                                <td class="py-3 px-4 text-emerald-400">74% <span class="text-[10px] text-slate-400 font-sans block">Frentes Frias Frequentes</span></td>
                                <td class="py-3 px-4 text-slate-400">12% <span class="text-[10px] text-slate-400 font-sans block">Baixo Risco</span></td>
                                <td class="py-3 px-4 text-slate-300">28% <span class="text-[10px] text-slate-400 font-sans block">Moderado</span></td>
                                <td class="py-3 px-4 text-sky-400 font-bold">52% <span class="text-[10px] text-slate-400 font-sans block">Início das Geadas</span></td>
                                <td class="py-3 px-4 text-slate-400">15% <span class="text-[10px] text-slate-400 font-sans block">Veranicos Ocasionais</span></td>
                            </tr>
                            <tr class="hover:bg-slate-800/40 transition">
                                <td class="py-3 px-4 font-bold text-indigo-400">Inverno (Jul - Set)</td>
                                <td class="py-3 px-4 text-emerald-400">82% <span class="text-[10px] text-slate-400 font-sans block">Ciclones & Frentes Extratropicais</span></td>
                                <td class="py-3 px-4 text-amber-400">18% <span class="text-[10px] text-slate-400 font-sans block">Choque Térmico</span></td>
                                <td class="py-3 px-4 text-rose-400 font-bold">48% <span class="text-[10px] text-slate-400 font-sans block">Ventos de Quadrante Sul</span></td>
                                <td class="py-3 px-4 text-sky-400 font-bold">85% <span class="text-[10px] text-slate-400 font-sans block">Elevada Frequência de Geada</span></td>
                                <td class="py-3 px-4 text-slate-500">2% <span class="text-[10px] text-slate-400 font-sans block">Muito Raro</span></td>
                            </tr>
                            <tr class="hover:bg-slate-800/40 transition">
                                <td class="py-3 px-4 font-bold text-rose-400">Primavera (Out - Nov)</td>
                                <td class="py-3 px-4 text-rose-400 font-bold">89% <span class="text-[10px] text-slate-400 font-sans block">Pico Pluviométrico Anual</span></td>
                                <td class="py-3 px-4 text-rose-400 font-bold">42% <span class="text-[10px] text-slate-400 font-sans block">Pico Histórico Granizo</span></td>
                                <td class="py-3 px-4 text-rose-400 font-bold">54% <span class="text-[10px] text-slate-400 font-sans block">Rajadas Severas</span></td>
                                <td class="py-3 px-4 text-slate-400">14% <span class="text-[10px] text-slate-400 font-sans block">Geadas Tardias</span></td>
                                <td class="py-3 px-4 text-amber-400">45% <span class="text-[10px] text-slate-400 font-sans block">Elevação Gradual</span></td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>

            <!-- Historical Table Section -->
            <div class="glass-card rounded-2xl p-5 border dark:border-slate-700 border-slate-300">
                <div class="flex items-center justify-between mb-4">
                    <h3 class="text-xs uppercase tracking-wider font-bold dark:text-slate-300 text-slate-700 flex items-center gap-2">
                        <i class="fa-solid fa-clock-rotate-left text-amber-500"></i> Histórico de Eventos Extremos Registrados em Erechim
                    </h3>
                </div>
                <div class="overflow-x-auto">
                    <table class="w-full text-left text-xs dark:text-slate-300 text-slate-700">
                        <thead class="dark:bg-slate-900/80 bg-slate-200/80 dark:text-slate-400 text-slate-600 font-semibold uppercase tracking-wider border-b dark:border-slate-800 border-slate-300">
                            <tr>
                                <th class="py-3 px-4">Data</th>
                                <th class="py-3 px-4">Tipo de Evento</th>
                                <th class="py-3 px-4">Valores Registrados</th>
                                <th class="py-3 px-4">Impactos / Observações</th>
                            </tr>
                        </thead>
                        <tbody id="modal-history-table-body" class="divide-y dark:divide-slate-800 divide-slate-200">
                        </tbody>
                    </table>
                </div>
            </div>

        </div>
    </div>

    <!-- MAIN CONTAINER -->
    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pt-6 pb-6 space-y-6 flex-grow w-full">

        <!-- EMERGENCY PUBLIC ALERT BANNER -->
        <div id="alert-banner" class="hidden rounded-2xl p-5 sm:p-6 bg-white border-4 border-[#ff2800] text-[#ff2800] flex items-center gap-4 sm:gap-6 shadow-2xl pulse-danger border-blink">
            <i class="fa-solid fa-triangle-exclamation text-4xl sm:text-5xl text-[#ff2800] shrink-0 animate-bounce"></i>
            <div class="w-full">
                <div class="flex items-center justify-between mb-1.5">
                    <h3 class="font-black text-sm sm:text-base text-[#ff2800] uppercase tracking-wider">Atenção: Evento Extremo / Comunicado Meteorológico</h3>
                    <span id="alert-type-tag" class="text-xs uppercase font-extrabold px-3 py-1 rounded-full bg-[#ff2800] text-white shadow">Alerta Oficial</span>
                </div>
                <p id="alert-banner-text" class="text-lg sm:text-2xl font-black tracking-tight text-[#ff2800] leading-snug">Condições meteorológicas extremas detectadas para o município de Erechim.</p>
            </div>
        </div>

        <!-- CURRENT WEATHER METRICS -->
        <section>
            <div class="flex items-center justify-between mb-3">
                <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-400 text-slate-500 flex items-center gap-2">
                    <i class="fa-solid fa-gauge-high text-sky-500"></i> Condições em Tempo Real — Estação Erechim
                </h2>
                <span class="text-xs dark:text-slate-500 text-slate-400 font-mono" id="last-update-label">Atualizado: --:--</span>
            </div>

            <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-3 sm:gap-4">
                
                <!-- Temp -->
                <div class="glass-card rounded-2xl p-4 flex flex-col justify-between">
                    <div class="flex items-center justify-between text-slate-400">
                        <span class="text-xs font-semibold uppercase">Temperatura</span>
                        <i class="fa-solid fa-temperature-half text-amber-500 text-lg"></i>
                    </div>
                    <div class="mt-3">
                        <div id="metric-temp" class="text-2xl sm:text-3xl font-black tracking-tight dark:text-white text-slate-900">--°C</div>
                        <div id="metric-temp-sub" class="text-xs text-slate-400 mt-1">Sensação: --°C</div>
                    </div>
                </div>

                <!-- Umidade -->
                <div class="glass-card rounded-2xl p-4 flex flex-col justify-between">
                    <div class="flex items-center justify-between text-slate-400">
                        <span class="text-xs font-semibold uppercase">Umidade Relativa</span>
                        <i class="fa-solid fa-droplet text-sky-400 text-lg"></i>
                    </div>
                    <div class="mt-3">
                        <div id="metric-humidity" class="text-2xl sm:text-3xl font-black tracking-tight dark:text-white text-slate-900">--%</div>
                        <div id="metric-humidity-sub" class="text-xs text-slate-400 mt-1">Ponto Orvalho: --°C</div>
                    </div>
                </div>

                <!-- Chuva -->
                <div class="glass-card rounded-2xl p-4 flex flex-col justify-between">
                    <div class="flex items-center justify-between text-slate-400">
                        <span class="text-xs font-semibold uppercase">Chuva (12h)</span>
                        <i class="fa-solid fa-cloud-showers-heavy text-blue-500 text-lg"></i>
                    </div>
                    <div class="mt-3">
                        <div id="metric-rain" class="text-2xl sm:text-3xl font-black tracking-tight dark:text-white text-slate-900">-- mm</div>
                        <div id="metric-rain-sub" class="text-xs text-slate-400 mt-1">Acumulado recente</div>
                    </div>
                </div>

                <!-- Vento -->
                <div class="glass-card rounded-2xl p-4 flex flex-col justify-between">
                    <div class="flex items-center justify-between text-slate-400">
                        <span class="text-xs font-semibold uppercase">Vento / Rajadas</span>
                        <i class="fa-solid fa-wind text-teal-400 text-lg"></i>
                    </div>
                    <div class="mt-3">
                        <div id="metric-wind" class="text-2xl sm:text-3xl font-black tracking-tight dark:text-white text-slate-900">-- km/h</div>
                        <div id="metric-wind-sub" class="text-xs text-slate-400 mt-1">Rajada: -- km/h</div>
                    </div>
                </div>

                <!-- Pressão -->
                <div class="glass-card rounded-2xl p-4 flex flex-col justify-between col-span-2 md:col-span-1">
                    <div class="flex items-center justify-between text-slate-400">
                        <span class="text-xs font-semibold uppercase">Pressão Atm.</span>
                        <i class="fa-solid fa-compress text-purple-400 text-lg"></i>
                    </div>
                    <div class="mt-3">
                        <div id="metric-pressure" class="text-2xl sm:text-3xl font-black tracking-tight dark:text-white text-slate-900">-- hPa</div>
                        <div id="metric-pressure-sub" class="text-xs text-slate-400 mt-1">Pressão Local</div>
                    </div>
                </div>

            </div>
        </section>

        <!-- EVENTOS EXTREMOS - THRESHOLDS CHECK WITH EXPLICIT LABELS -->
        <section>
            <div class="flex items-center justify-between mb-3">
                <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-400 text-slate-500 flex items-center gap-2">
                    <i class="fa-solid fa-shield-cat text-rose-500"></i> Status dos Limiares de Severidade Meteorológica
                </h2>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-3 sm:gap-4">
                
                <!-- Frio -->
                <div id="card-alert-cold" class="glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between">
                    <div>
                        <div class="flex items-center justify-between">
                            <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Frio Extremo</span>
                            <span id="risk-cold" class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300">Risco: --%</span>
                        </div>
                        <div class="mt-2 space-y-1">
                            <span class="text-[11px] text-sky-400 block"><i class="fa-solid fa-circle-dot text-[9px] mr-1"></i>Dado Atual da Estação: <strong id="val-cold" class="font-mono font-bold text-white">-- °C</strong></span>
                            <span class="text-[11px] text-slate-400 block"><i class="fa-solid fa-triangle-exclamation text-[9px] mr-1 text-amber-400"></i>Gatilho de Severidade: <strong id="lbl-cfg-cold" class="dark:text-slate-300 text-slate-700 font-mono">&lt; 5.0 °C</strong></span>
                        </div>
                    </div>
                    <div class="mt-3 flex items-center justify-between pt-2 border-t dark:border-slate-800 border-slate-200">
                        <span class="text-[10px] text-slate-400 uppercase">Status:</span>
                        <span id="badge-cold" class="px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700">Normal</span>
                    </div>
                </div>

                <!-- Calor -->
                <div id="card-alert-heat" class="glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between">
                    <div>
                        <div class="flex items-center justify-between">
                            <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Calor Extremo</span>
                            <span id="risk-heat" class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300">Risco: --%</span>
                        </div>
                        <div class="mt-2 space-y-1">
                            <span class="text-[11px] text-sky-400 block"><i class="fa-solid fa-circle-dot text-[9px] mr-1"></i>Dado Atual da Estação: <strong id="val-heat" class="font-mono font-bold text-white">-- °C</strong></span>
                            <span class="text-[11px] text-slate-400 block"><i class="fa-solid fa-triangle-exclamation text-[9px] mr-1 text-amber-400"></i>Gatilho de Severidade: <strong id="lbl-cfg-heat" class="dark:text-slate-300 text-slate-700 font-mono">&gt; 28.0 °C</strong></span>
                        </div>
                    </div>
                    <div class="mt-3 flex items-center justify-between pt-2 border-t dark:border-slate-800 border-slate-200">
                        <span class="text-[10px] text-slate-400 uppercase">Status:</span>
                        <span id="badge-heat" class="px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700">Normal</span>
                    </div>
                </div>

                <!-- Chuva -->
                <div id="card-alert-rain" class="glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between">
                    <div>
                        <div class="flex items-center justify-between">
                            <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Chuva Intensa</span>
                            <span id="risk-rain" class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300">Risco: --%</span>
                        </div>
                        <div class="mt-2 space-y-1">
                            <span class="text-[11px] text-sky-400 block"><i class="fa-solid fa-circle-dot text-[9px] mr-1"></i>Dado Atual da Estação: <strong id="val-rain" class="font-mono font-bold text-white">-- mm</strong></span>
                            <span class="text-[11px] text-slate-400 block"><i class="fa-solid fa-triangle-exclamation text-[9px] mr-1 text-amber-400"></i>Gatilho de Severidade: <strong id="lbl-cfg-rain" class="dark:text-slate-300 text-slate-700 font-mono">&gt; 30mm/12h</strong></span>
                        </div>
                    </div>
                    <div class="mt-3 flex items-center justify-between pt-2 border-t dark:border-slate-800 border-slate-200">
                        <span class="text-[10px] text-slate-400 uppercase">Status:</span>
                        <span id="badge-rain" class="px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700">Normal</span>
                    </div>
                </div>

                <!-- Vento -->
                <div id="card-alert-wind" class="glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between">
                    <div>
                        <div class="flex items-center justify-between">
                            <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Vento Forte</span>
                            <span id="risk-wind" class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300">Risco: --%</span>
                        </div>
                        <div class="mt-2 space-y-1">
                            <span class="text-[11px] text-sky-400 block"><i class="fa-solid fa-circle-dot text-[9px] mr-1"></i>Dado Atual da Estação: <strong id="val-wind" class="font-mono font-bold text-white">-- km/h</strong></span>
                            <span class="text-[11px] text-slate-400 block"><i class="fa-solid fa-triangle-exclamation text-[9px] mr-1 text-amber-400"></i>Gatilho de Severidade: <strong id="lbl-cfg-wind" class="dark:text-slate-300 text-slate-700 font-mono">&gt; 40 km/h</strong></span>
                        </div>
                    </div>
                    <div class="mt-3 flex items-center justify-between pt-2 border-t dark:border-slate-800 border-slate-200">
                        <span class="text-[10px] text-slate-400 uppercase">Status:</span>
                        <span id="badge-wind" class="px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700">Normal</span>
                    </div>
                </div>

                <!-- Granizo -->
                <div id="card-alert-hail" class="glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between">
                    <div>
                        <div class="flex items-center justify-between">
                            <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Risco Granizo</span>
                            <span id="risk-hail" class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300">Risco: --%</span>
                        </div>
                        <div class="mt-2 space-y-1">
                            <span class="text-[11px] text-sky-400 block"><i class="fa-solid fa-circle-dot text-[9px] mr-1"></i>Dado Atual da Estação: <strong id="val-hail" class="font-mono font-bold text-white">--</strong></span>
                            <span class="text-[11px] text-slate-400 block"><i class="fa-solid fa-triangle-exclamation text-[9px] mr-1 text-amber-400"></i>Gatilho de Severidade: <strong class="dark:text-slate-300 text-slate-700 font-mono">Convectivo</strong></span>
                        </div>
                    </div>
                    <div class="mt-3 flex items-center justify-between pt-2 border-t dark:border-slate-800 border-slate-200">
                        <span class="text-[10px] text-slate-400 uppercase">Status:</span>
                        <span id="badge-hail" class="px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700">Baixo</span>
                    </div>
                </div>

            </div>
        </section>

        <!-- SATELLITE, RADAR, SYNOPTIC & WIND SECTIONS (ONE PER LINE) -->
        <section class="space-y-6">
            
            <!-- 1. RADAR INTERATIVO (Linha Inteira) -->
            <div class="glass-card rounded-2xl p-5 flex flex-col justify-between">
                <div>
                    <div class="flex items-center justify-between mb-3">
                        <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-300 text-slate-700 flex items-center gap-2">
                            <i class="fa-solid fa-earth-americas text-sky-500"></i> Radar Interativo
                        </h2>
                        <span class="text-xs bg-slate-800 text-slate-300 px-2.5 py-1 rounded-lg border border-slate-700 font-mono">
                            Erechim (RS)
                        </span>
                    </div>
                    
                    <div class="w-full h-[450px] rounded-xl overflow-hidden border dark:border-slate-800 border-slate-300 bg-slate-950 shadow-inner">
                        <iframe 
                            id="windy-frame"
                            class="w-full h-full border-0" 
                            src="https://embed.windy.com/embed.html?lat=-27.634&lon=-52.274&detailLat=-27.634&detailLon=-52.274&width=1000&height=450&zoom=9&level=surface&overlay=wind&product=ecmwf&menu=&message=&marker=true&calendar=now&pressure=true&type=map&location=coordinates&detail=true&metricWind=km%2Fh&metricTemp=%C2%B0C&radarRange=-1" 
                            loading="lazy">
                        </iframe>
                    </div>
                </div>

                <div class="flex flex-wrap items-center justify-between gap-2 mt-4 text-xs">
                    <span class="text-slate-400">Camadas:</span>
                    <div class="flex flex-wrap gap-1.5">
                        <button onclick="changeWindyLayer('wind')" class="px-2.5 py-1 rounded-lg bg-sky-950 border border-sky-800 text-sky-200 transition text-[11px]">
                            <i class="fa-solid fa-wind mr-1"></i> Vento
                        </button>
                        <button onclick="changeWindyLayer('rain')" class="px-2.5 py-1 rounded-lg bg-blue-950 border border-blue-800 text-blue-200 transition text-[11px]">
                            <i class="fa-solid fa-cloud-rain mr-1"></i> Chuva
                        </button>
                        <button onclick="changeWindyLayer('thunder')" class="px-2.5 py-1 rounded-lg bg-purple-950 border border-purple-800 text-purple-200 transition text-[11px]">
                            <i class="fa-solid fa-bolt mr-1"></i> Raios
                        </button>
                    </div>
                </div>
            </div>

            <!-- 2. SATÉLITE GOES-16 (Linha Inteira) -->
            <div class="glass-card rounded-2xl p-5 flex flex-col justify-between">
                <div>
                    <div class="flex items-center justify-between mb-3">
                        <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-300 text-slate-700 flex items-center gap-2">
                            <i class="fa-solid fa-satellite text-emerald-500"></i> Imagem de Satélite
                        </h2>
                        <span id="sat-mode-badge" class="text-[10px] px-2 py-0.5 rounded bg-emerald-950 text-emerald-300 border border-emerald-800 font-mono uppercase">
                            INFRAVERMELHO
                        </span>
                    </div>

                    <div class="relative w-full h-[450px] rounded-xl overflow-hidden border dark:border-slate-800 border-slate-300 bg-slate-950 flex items-center justify-center group">
                        <iframe id="sat-iframe-view" class="w-full h-full border-0 hidden" src="about:blank" loading="lazy"></iframe>
                        <img id="satellite-img" src="https://cdn.star.nesdis.noaa.gov/GOES16/ABI/SECTOR/ssa/13/1000x1000.jpg" alt="Satélite GOES-16 Erechim" class="w-full h-full object-cover rounded-xl transition duration-300 group-hover:scale-105" onerror="handleSatelliteError(this)">
                        
                        <div class="absolute bottom-2 left-2 right-2 flex justify-between items-center bg-slate-950/80 px-2.5 py-1.5 rounded-lg border border-slate-800 text-[10px] font-mono text-slate-300 backdrop-blur-sm z-10">
                            <span id="sat-status-label" class="flex items-center gap-1.5">
                                <span class="w-2 h-2 rounded-full bg-emerald-500 animate-ping"></span> GOES-16 NOAA / CIRA
                            </span>
                            <button onclick="reloadSatelliteImage()" class="hover:text-sky-400 transition" title="Recarregar Imagem">
                                <i class="fa-solid fa-rotate-right"></i>
                            </button>
                        </div>
                    </div>
                </div>

                <div class="flex gap-2 mt-4 text-xs">
                    <button onclick="switchSatChannel('ir')" class="py-1.5 px-3 rounded bg-sky-950 text-sky-200 border border-sky-800 hover:bg-sky-900 transition font-medium text-center">
                        Infravermelho
                    </button>
                    <button onclick="switchSatChannel('geocolor')" class="py-1.5 px-3 rounded dark:bg-slate-800 dark:text-slate-200 bg-slate-200 text-slate-700 hover:opacity-80 transition font-medium text-center">
                        GeoColor
                    </button>
                    <button onclick="switchSatChannel('interactive')" class="py-1.5 px-3 rounded dark:bg-slate-800 dark:text-slate-200 bg-slate-200 text-slate-700 hover:opacity-80 transition font-medium text-center">
                        Radar Interativo
                    </button>
                </div>
            </div>

            <!-- 3. CARTA SINÓTICA (Linha Inteira) -->
            <div class="glass-card rounded-2xl p-5 flex flex-col justify-between">
                <div>
                    <div class="flex items-center justify-between mb-3">
                        <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-300 text-slate-700 flex items-center gap-2">
                            <i class="fa-solid fa-globe text-indigo-400"></i> Earth Nullschool (Globo Meteorológico Global e Correntes de Vento)
                        </h2>
                        <span class="text-xs font-mono text-indigo-300 bg-indigo-950/60 px-2.5 py-1 rounded-lg border border-indigo-800">
                            Erechim (RS) - Lat: -27.63, Lon: -52.27
                        </span>
                    </div>

                    <div class="relative w-full h-[450px] rounded-xl overflow-hidden border dark:border-slate-800 border-slate-300 bg-slate-950 flex items-center justify-center group">
                        <iframe 
                            class="w-full h-full border-0" 
                            src="https://earth.nullschool.net/#current/wind/surface/level/orthographic=-52.27,-27.63,3000" 
                            loading="lazy">
                        </iframe>
                        <div class="absolute bottom-2 left-2 right-2 flex justify-between items-center bg-slate-950/80 px-2.5 py-1.5 rounded-lg border border-slate-800 text-[10px] font-mono text-slate-300 backdrop-blur-sm z-10 pointer-events-none">
                            <span>Visualização Interativa de Ventos e Correntes Globais</span>
                            <a href="https://earth.nullschool.net/#current/wind/surface/level/orthographic=-52.27,-27.63,3000" target="_blank" class="hover:text-sky-400 underline pointer-events-auto">Abrir Tela Cheia</a>
                        </div>
                    </div>
                </div>
            </div>

            <!-- 4. DIREÇÃO & INTENSIDADE DO VENTO (Linha Inteira) -->
            <div id="wind-card-container" class="glass-card rounded-2xl p-5 flex flex-col justify-between transition-all duration-300">
                <div>
                    <div class="flex items-center justify-between mb-3">
                        <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-300 text-slate-700 flex items-center gap-2">
                            <i class="fa-solid fa-compass text-teal-400 text-base"></i> Direção & Intensidade do Vento (Orientação Cardinal)
                        </h2>
                        <div class="flex items-center gap-2">
                            <span id="wind-alert-status-badge" class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300 border border-slate-700">Normal</span>
                            <span id="wind-dir-text" class="text-xs font-mono font-bold text-teal-400 bg-teal-950/60 border border-teal-800/60 px-2.5 py-0.5 rounded">
                                --° N
                            </span>
                        </div>
                    </div>

                    <div class="flex flex-col items-center justify-center my-4 relative">
                        <!-- Compass Rose Visual with Explicit Direction Arrow -->
                        <div class="w-48 h-48 rounded-full border-2 border-slate-700/80 bg-slate-900/90 relative flex items-center justify-center shadow-inner">
                            <span class="absolute top-2 text-[11px] font-bold text-slate-400">N (Norte)</span>
                            <span class="absolute right-3 text-[11px] font-bold text-slate-400">L (Leste)</span>
                            <span class="absolute bottom-2 text-[11px] font-bold text-slate-400">S (Sul)</span>
                            <span class="absolute left-3 text-[11px] font-bold text-slate-400">O (Oeste)</span>

                            <!-- Rotating Arrow Indicator with Lateral/Horizontal Directional Label -->
                            <div id="wind-arrow-container" class="w-36 h-36 absolute flex items-center justify-center transition-transform duration-700 ease-out">
                                <div class="w-3 h-28 bg-gradient-to-t from-sky-500 via-teal-400 to-rose-500 rounded-full flex flex-col justify-between items-center shadow-2xl border border-white/20 relative">
                                    <div class="w-0 h-0 border-x-8 border-x-transparent border-b-[16px] border-b-rose-400 -mt-2"></div>
                                    <span id="wind-arrow-label" class="text-[9px] font-black text-slate-950 bg-white/95 px-1.5 py-0.5 rounded shadow absolute whitespace-nowrap -right-12 top-1/2 -translate-y-1/2 border border-slate-700">DIREÇÃO</span>
                                    <div class="w-3 h-3 rounded-full bg-white shadow-md mb-0.5"></div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="grid grid-cols-2 gap-3 text-center text-xs mt-2">
                    <div class="p-3 rounded-xl bg-slate-900/60 border border-slate-800">
                        <span class="text-slate-400 block text-[10px]">Velocidade Constante</span>
                        <strong id="wind-card-speed" class="text-base font-bold text-teal-300">-- km/h</strong>
                    </div>
                    <div class="p-3 rounded-xl bg-slate-900/60 border border-slate-800">
                        <span class="text-slate-400 block text-[10px]">Rajada Máxima</span>
                        <strong id="wind-card-gust" class="text-base font-bold text-amber-300">-- km/h</strong>
                    </div>
                </div>
            </div>

        </section>

        <!-- FORECAST 24h & TREND GRAPH -->
        <section class="glass-card rounded-2xl p-5">
            <div class="flex items-center justify-between mb-4">
                <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-300 text-slate-700 flex items-center gap-2">
                    <i class="fa-solid fa-chart-line text-indigo-400"></i> Tendência das Próximas 24h (Temperatura & Precipitação)
                </h2>
                <span class="text-xs text-slate-400">Modelo Open-Meteo High Precision</span>
            </div>

            <div class="w-full h-64 sm:h-72">
                <canvas id="forecastChart"></canvas>
            </div>
        </section>

        <!-- 4-DAY GRAPHIC WEATHER FORECAST TIMELINE -->
        <section class="glass-card rounded-2xl p-5">
            <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3 mb-5">
                <div>
                    <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-300 text-slate-700 flex items-center gap-2">
                        <i class="fa-solid fa-timeline text-sky-400"></i> Previsão dos Próximos 4 Dias
                    </h2>
                    <p class="text-xs text-slate-400">Linha do tempo gráfica com amplitude térmica, volumes acumulados e intensidade de ventos.</p>
                </div>
                <span class="text-xs text-slate-400 font-mono bg-slate-900/60 px-3 py-1 rounded-lg border border-slate-800 flex items-center gap-1.5">
                    <i class="fa-solid fa-calendar-week text-sky-400"></i> +4 Dias Pós-Hoje
                </span>
            </div>

            <div id="forecast-3days-container" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                <div class="p-4 rounded-xl dark:bg-slate-900/60 bg-slate-100 border dark:border-slate-800 border-slate-200 text-center animate-pulse">
                    <span class="text-xs text-slate-400">Carregando linha do tempo de 4 dias...</span>
                </div>
            </div>
        </section>

    </main>

    <!-- FOOTER -->
    <footer class="border-t dark:border-slate-900 border-slate-200 dark:bg-slate-950 bg-white py-8 text-center text-xs text-slate-500">
        <div class="max-w-7xl mx-auto px-4 space-y-6">
            
            <div class="glass-card max-w-2xl mx-auto rounded-2xl p-5 border dark:border-slate-800 border-slate-200 shadow-sm text-center">
                <h3 class="text-xs font-bold uppercase tracking-wider dark:text-slate-300 text-slate-700 mb-3 flex items-center justify-center gap-2">
                    <i class="fa-solid fa-address-card text-sky-500 text-sm"></i> Contatos & Canais Oficiais
                </h3>
                
                <div class="flex flex-wrap items-center justify-center gap-3 sm:gap-4 text-xs font-medium">
                    <a href="mailto:wander.l.marques@gmail.com" class="flex items-center gap-2 px-3.5 py-2 rounded-xl dark:bg-slate-900 bg-slate-100 hover:bg-sky-500/10 hover:text-sky-500 dark:text-slate-300 text-slate-700 border dark:border-slate-800 border-slate-200 transition">
                        <i class="fa-solid fa-envelope text-rose-500"></i>
                        <span>wander.l.marques@gmail.com</span>
                    </a>
                    <a href="https://www.instagram.com/wanderlmarques/" target="_blank" rel="noopener noreferrer" class="flex items-center gap-2 px-3.5 py-2 rounded-xl dark:bg-slate-900 bg-slate-100 hover:bg-pink-500/10 hover:text-pink-500 dark:text-slate-300 text-slate-700 border dark:border-slate-800 border-slate-200 transition">
                        <i class="fa-brands fa-instagram text-pink-500"></i>
                        <span>Instagram</span>
                    </a>
                    <a href="https://www.facebook.com/WanderLuisMarques" target="_blank" rel="noopener noreferrer" class="flex items-center gap-2 px-3.5 py-2 rounded-xl dark:bg-slate-900 bg-slate-100 hover:bg-blue-500/10 hover:text-blue-500 dark:text-slate-300 text-slate-700 border dark:border-slate-800 border-slate-200 transition">
                        <i class="fa-brands fa-facebook text-blue-600"></i>
                        <span>Facebook</span>
                    </a>
                    <a href="https://www.linkedin.com/in/wander-lu%C3%ADs-marques-71a3651aa/" target="_blank" rel="noopener noreferrer" class="flex items-center gap-2 px-3.5 py-2 rounded-xl dark:bg-slate-900 bg-slate-100 hover:bg-sky-600/10 hover:text-sky-400 dark:text-slate-300 text-slate-700 border dark:border-slate-800 border-slate-200 transition">
                        <i class="fa-brands fa-linkedin text-sky-600"></i>
                        <span>LinkedIn</span>
                    </a>
                </div>
            </div>

            <div>
                <p>Central de Monitoramento Climático - Erechim / RS</p>
                <p class="mt-1">Desenvolvido por: <strong>Wander Luís Marques</strong></p>
            </div>
        </div>
    </footer>

    <!-- JAVASCRIPT & LOGIC -->
    <script>
        const LAT = -27.634;
        const LON = -52.274;
        let chartInstance = null;

        let thresholds = { cold: 5.0, heat: 28.0, rain: 30.0, wind: 40.0 };

        const satSources = {
            ir: ["https://cdn.star.nesdis.noaa.gov/GOES16/ABI/SECTOR/ssa/13/1000x1000.jpg"],
            geocolor: ["https://cdn.star.nesdis.noaa.gov/GOES16/ABI/SECTOR/ssa/GEOCOLOR/1000x1000.jpg"],
            interactive: ["https://embed.windy.com/embed.html?lat=-27.634&lon=-52.274&detailLat=-27.634&detailLon=-52.274&width=1000&height=450&zoom=7&level=surface&overlay=satellite&product=satellite&menu=&message=&marker=true&calendar=now&pressure=false&type=map&location=coordinates&detail=false&metricWind=km%2Fh&metricTemp=%C2%B0C&radarRange=-1"]
        };

        let currentSatChannel = 'ir';

        const historicalEvents = [
            { date: "23/11/2025", type: "Tempestade Severa de Granizo", value: "Pedras > 5 cm", impact: "Danos em telhados e lavouras na área urbana e rural de Erechim." },
            { date: "02/05/2024", type: "Chuva Torrencial / Alagamento", value: "118.4 mm / 24h", impact: "Transbordamento de córregos no Centro, Rio Tigre e Bairro Esperança." },
            { date: "16/01/2024", type: "Vendaval Severo", value: "94.2 km/h", impact: "Destelhamento nos bairros Três Vendas e Koller." },
            { date: "12/10/2023", type: "Queda de Granizo Convectivo", value: "Pedras de 3 a 5 cm", impact: "Danos em coberturas na zona rural e nos bairros São Cristóvão e Copas Verdes." },
            { date: "13/07/2023", type: "Frio Extremo / Geada Severa", value: "-1.2 °C", impact: "Geada generalizada em todo o município." }
        ];

        function toggleTheme() {
            const html = document.documentElement;
            const themeIcon = document.getElementById('theme-icon');
            if (html.classList.contains('dark')) {
                html.classList.remove('dark');
                html.classList.add('light');
                themeIcon.classList.replace('fa-moon', 'fa-sun');
            } else {
                html.classList.remove('light');
                html.classList.add('dark');
                themeIcon.classList.replace('fa-sun', 'fa-moon');
            }
        }

        function toggleAdminModal() {
            const modal = document.getElementById('admin-modal');
            modal.classList.toggle('hidden');
        }

        function toggleHistoryPage(show) {
            const modal = document.getElementById('history-page-modal');
            if (show) {
                modal.classList.remove('hidden');
            } else {
                modal.classList.add('hidden');
            }
        }

        function handleAdminLogin(event) {
            event.preventDefault();
            const name = document.getElementById('login-name').value.trim().toLowerCase();
            const pass = document.getElementById('login-pass').value;
            const err = document.getElementById('login-error');
            
            if ((name.includes("wander") || name.includes("marques")) && pass === 'AMinhaSenhaeh@1') {
                err.classList.add('hidden');
                document.getElementById('admin-login-view').classList.add('hidden');
                document.getElementById('admin-panel-view').classList.remove('hidden');
                document.getElementById('admin-btn-text').innerText = 'Admin Ativo';
            } else {
                err.classList.remove('hidden');
            }
        }

        function handleAdminLogout() {
            document.getElementById('admin-login-view').classList.remove('hidden');
            document.getElementById('admin-panel-view').classList.add('hidden');
            document.getElementById('admin-btn-text').innerText = 'Área Administrativa';
            toggleAdminModal();
        }

        function changeWindyLayer(layer) {
            document.getElementById('windy-frame').src = `https://embed.windy.com/embed.html?lat=${LAT}&lon=${LON}&detailLat=${LAT}&detailLon=${LON}&width=1000&height=450&zoom=9&level=surface&overlay=${layer}&product=ecmwf&menu=&message=&marker=true&calendar=now&pressure=true&type=map&location=coordinates&detail=true&metricWind=km%2Fh&metricTemp=%C2%B0C&radarRange=-1`;
        }

        function switchSatChannel(channel) {
            currentSatChannel = channel;
            reloadSatelliteImage();
        }

        function getCardinalDirection(angle) {
            const directions = ['N', 'NE', 'L', 'SE', 'S', 'SO', 'O', 'NO'];
            return directions[Math.round(angle / 45) % 8];
        }

        async function fetchWeatherData() {
            try {
                const url = `https://api.open-meteo.com/v1/forecast?latitude=${LAT}&longitude=${LON}&current=temperature_2m,relative_humidity_2m,apparent_temperature,precipitation,surface_pressure,wind_speed_10m,wind_direction_10m,wind_gusts_10m,weather_code&hourly=temperature_2m,precipitation,wind_speed_10m,precipitation_probability&daily=weather_code,temperature_2m_max,temperature_2m_min,precipitation_sum,wind_speed_10m_max,precipitation_probability_max&past_hours=12&forecast_days=5&timezone=America%2FSao_Paulo`;
                const response = await fetch(url);
                const data = await response.json();
                const current = data.current;

                const temp = current.temperature_2m;
                const humidity = current.relative_humidity_2m;
                let rain12h = data.hourly ? data.hourly.precipitation.slice(0, 13).reduce((a, b) => a + (b || 0), 0) : current.precipitation;
                const windSpeed = current.wind_speed_10m;
                const windGust = current.wind_gusts_10m || (windSpeed * 1.3);
                const windDegree = current.wind_direction_10m || 0;

                // Update Metrics UI safely with element checks
                const setElemText = (id, text) => {
                    const el = document.getElementById(id);
                    if (el) el.innerText = text;
                };

                setElemText('metric-temp', `${temp.toFixed(1)}°C`);
                setElemText('metric-temp-sub', `Sensação: ${current.apparent_temperature.toFixed(1)}°C`);
                setElemText('metric-humidity', `${humidity}%`);
                setElemText('metric-rain', `${rain12h.toFixed(1)} mm`);
                setElemText('metric-wind', `${windSpeed.toFixed(1)} km/h`);
                setElemText('metric-wind-sub', `Rajada: ${windGust.toFixed(1)} km/h`);
                setElemText('metric-pressure', `${current.surface_pressure.toFixed(0)} hPa`);

                setElemText('wind-dir-text', `${windDegree}° ${getCardinalDirection(windDegree)}`);
                
                const windArrow = document.getElementById('wind-arrow-container');
                if (windArrow) windArrow.style.transform = `rotate(${windDegree}deg)`;

                setElemText('wind-card-speed', `${windSpeed.toFixed(1)} km/h`);
                setElemText('wind-card-gust', `${windGust.toFixed(1)} km/h`);

                // Update Threshold Cards with live values
                updateThresholdStatus(temp, rain12h, windSpeed, humidity);
                update6HourAlert(data.hourly);

                renderForecastChart(data.hourly);
                render3DayForecast(data.daily);

                const now = new Date();
                setElemText('last-update-label', `Atualizado: ${now.toLocaleTimeString('pt-BR', { hour: '2-digit', minute: '2-digit' })}`);
            } catch (err) {
                console.error(err);
            }
        }

        function update6HourAlert(hourly) {
            if (!hourly || !hourly.temperature_2m) return;
            const next6Temps = hourly.temperature_2m.slice(0, 7);
            const next6Rains = hourly.precipitation ? hourly.precipitation.slice(0, 7) : [];
            const next6Probs = hourly.precipitation_probability ? hourly.precipitation_probability.slice(0, 7) : [];
            
            const maxRain6h = next6Rains.reduce((a, b) => a + (b || 0), 0);
            const maxProb6h = next6Probs.length > 0 ? Math.max(...next6Probs) : 0;
            
            const alertBox = document.getElementById('header-6h-alert');
            const alertContent = document.getElementById('content-6h-alert');
            const alertSpan = document.getElementById('label-6h-span');
            
            if (maxRain6h > 15 || maxProb6h > 60) {
                alertBox.className = "flex-1 max-w-xl mx-auto lg:mx-4 px-4 py-3 rounded-xl bg-blue-600/20 border-2 border-blue-500 text-blue-100 transition shadow-lg flex flex-row items-center justify-between gap-3";
                alertContent.innerHTML = `Chuva de <strong>${maxRain6h.toFixed(1)} mm</strong> (Prob: <strong>${maxProb6h}%</strong>) nas próximas 6h.`;
                alertSpan.innerText = "ALERTA CHUVA";
                alertSpan.className = "text-[10px] sm:text-xs font-mono font-bold px-2.5 py-1 rounded bg-blue-500 text-slate-950 uppercase shrink-0";
            } else {
                alertBox.className = "flex-1 max-w-xl mx-auto lg:mx-4 px-4 py-3 rounded-xl bg-emerald-600/20 border-2 border-emerald-500 text-emerald-100 transition shadow-lg flex flex-row items-center justify-between gap-3";
                alertContent.innerHTML = `Estável. Prob. chuva: <strong>${maxProb6h}%</strong> (Acum: ${maxRain6h.toFixed(1)}mm).`;
                alertSpan.innerText = "SEM ALERTA";
                alertSpan.className = "text-[10px] sm:text-xs font-mono font-bold px-2.5 py-1 rounded bg-emerald-500 text-slate-950 uppercase shrink-0";
            }
        }

        function updateThresholdStatus(temp, rain, wind, humidity) {
            const valCold = document.getElementById('val-cold');
            const badgeCold = document.getElementById('badge-cold');
            const cardCold = document.getElementById('card-alert-cold');
            const riskCold = document.getElementById('risk-cold');
            
            // Cold risk calculation
            let coldRisk = temp <= thresholds.cold ? 100 : Math.max(0, Math.round(100 - ((temp - thresholds.cold) / 15) * 100));
            if (riskCold) {
                riskCold.innerText = `Risco: ${coldRisk}%`;
                riskCold.className = coldRisk > 50 ? "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-sky-950 text-sky-300 border border-sky-800" : "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-400";
            }
            if (valCold) valCold.innerText = `${temp.toFixed(1)} °C`;
            if (temp <= thresholds.cold) {
                if (badgeCold) { badgeCold.innerText = "Alerta Frio"; badgeCold.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-sky-600 text-white shadow"; }
                if (cardCold) cardCold.className = "glass-card rounded-2xl p-4 border-l-4 border-l-sky-500 flex flex-col justify-between";
            } else {
                if (badgeCold) { badgeCold.innerText = "Normal"; badgeCold.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700"; }
                if (cardCold) cardCold.className = "glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between";
            }

            const valHeat = document.getElementById('val-heat');
            const badgeHeat = document.getElementById('badge-heat');
            const cardHeat = document.getElementById('card-alert-heat');
            const riskHeat = document.getElementById('risk-heat');
            
            let heatRisk = temp >= thresholds.heat ? 100 : Math.max(0, Math.round((temp / thresholds.heat) * 80));
            if (riskHeat) {
                riskHeat.innerText = `Risco: ${heatRisk}%`;
                riskHeat.className = heatRisk > 70 ? "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-amber-950 text-amber-300 border border-amber-800" : "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-400";
            }
            if (valHeat) valHeat.innerText = `${temp.toFixed(1)} °C`;
            if (temp >= thresholds.heat) {
                if (badgeHeat) { badgeHeat.innerText = "Alerta Calor"; badgeHeat.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-amber-600 text-white shadow"; }
                if (cardHeat) cardHeat.className = "glass-card rounded-2xl p-4 border-l-4 border-l-amber-500 flex flex-col justify-between";
            } else {
                if (badgeHeat) { badgeHeat.innerText = "Normal"; badgeHeat.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700"; }
                if (cardHeat) cardHeat.className = "glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between";
            }

            const valRain = document.getElementById('val-rain');
            const badgeRain = document.getElementById('badge-rain');
            const cardRain = document.getElementById('card-alert-rain');
            const riskRain = document.getElementById('risk-rain');
            
            let rainRisk = Math.min(100, Math.round((rain / thresholds.rain) * 100));
            if (riskRain) {
                riskRain.innerText = `Risco: ${rainRisk}%`;
                riskRain.className = rainRisk >= 100 ? "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-blue-950 text-blue-300 border border-blue-800" : "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-400";
            }
            if (valRain) valRain.innerText = `${rain.toFixed(1)} mm`;
            if (rain >= thresholds.rain) {
                if (badgeRain) { badgeRain.innerText = "Alerta Chuva"; badgeRain.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-blue-600 text-white shadow"; }
                if (cardRain) cardRain.className = "glass-card rounded-2xl p-4 border-l-4 border-l-blue-500 flex flex-col justify-between";
            } else {
                if (badgeRain) { badgeRain.innerText = "Normal"; badgeRain.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700"; }
                if (cardRain) cardRain.className = "glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between";
            }

            const valWind = document.getElementById('val-wind');
            const badgeWind = document.getElementById('badge-wind');
            const cardWind = document.getElementById('card-alert-wind');
            const riskWind = document.getElementById('risk-wind');

            // Wind card elements
            const windCardContainer = document.getElementById('wind-card-container');
            const windAlertBadge = document.getElementById('wind-alert-status-badge');
            
            let windRisk = Math.min(100, Math.round((wind / thresholds.wind) * 100));
            if (riskWind) {
                riskWind.innerText = `Risco: ${windRisk}%`;
                riskWind.className = windRisk >= 100 ? "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-teal-950 text-teal-300 border border-teal-800" : "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-400";
            }
            if (valWind) valWind.innerText = `${wind.toFixed(1)} km/h`;

            // Advanced Wind Gust Alerts (>43km/h blinking red/white, 35-40 red, 30-35 yellow)
            if (wind > 43) {
                if (badgeWind) { badgeWind.innerText = "ALERTA CRÍTICO (>43)"; badgeWind.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-[#ff2800] text-white shadow"; }
                if (cardWind) cardWind.className = "glass-card rounded-2xl p-4 border-l-4 border-l-[#ff2800] flex flex-col justify-between";
                if (windCardContainer) {
                    windCardContainer.className = "glass-card rounded-2xl p-5 flex flex-col justify-between border-2 border-[#ff2800] border-blink pulse-danger";
                }
                if (windAlertBadge) {
                    windAlertBadge.innerText = "CRÍTICO >43KM/H";
                    windAlertBadge.className = "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-[#ff2800] text-white animate-bounce";
                }
            } else if (wind > 35 && wind <= 43) {
                if (badgeWind) { badgeWind.innerText = "Alerta Vermelho"; badgeWind.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-rose-600 text-white shadow"; }
                if (cardWind) cardWind.className = "glass-card rounded-2xl p-4 border-l-4 border-l-rose-500 flex flex-col justify-between";
                if (windCardContainer) {
                    windCardContainer.className = "glass-card rounded-2xl p-5 flex flex-col justify-between border-2 border-rose-500 shadow-lg";
                }
                if (windAlertBadge) {
                    windAlertBadge.innerText = "ALERTA VERMELHO";
                    windAlertBadge.className = "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-rose-600 text-white";
                }
            } else if (wind >= 30 && wind <= 35) {
                if (badgeWind) { badgeWind.innerText = "Alerta Amarelo"; badgeWind.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-amber-500 text-slate-950 font-bold shadow"; }
                if (cardWind) cardWind.className = "glass-card rounded-2xl p-4 border-l-4 border-l-amber-400 flex flex-col justify-between";
                if (windCardContainer) {
                    windCardContainer.className = "glass-card rounded-2xl p-5 flex flex-col justify-between border-2 border-amber-500 shadow-lg";
                }
                if (windAlertBadge) {
                    windAlertBadge.innerText = "ALERTA AMARELO";
                    windAlertBadge.className = "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-amber-500 text-slate-950 font-bold";
                }
            } else {
                if (badgeWind) { badgeWind.innerText = "Normal"; badgeWind.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700"; }
                if (cardWind) cardWind.className = "glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between";
                if (windCardContainer) {
                    windCardContainer.className = "glass-card rounded-2xl p-5 flex flex-col justify-between";
                }
                if (windAlertBadge) {
                    windAlertBadge.innerText = "Normal";
                    windAlertBadge.className = "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300 border border-slate-700";
                }
            }

            const valHail = document.getElementById('val-hail');
            const badgeHail = document.getElementById('badge-hail');
            const cardHail = document.getElementById('card-alert-hail');
            const riskHail = document.getElementById('risk-hail');
            
            let hailScore = (temp > 24 && humidity > 65) ? Math.min(95, Math.round((humidity * temp) / 10)) : 15;
            if (riskHail) {
                riskHail.innerText = `Risco: ${hailScore}%`;
                riskHail.className = hailScore > 65 ? "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-rose-950 text-rose-300 border border-rose-800" : "text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-400";
            }
            if (valHail) valHail.innerText = `${hailScore}%`;
            if (hailScore > 65) {
                if (badgeHail) { badgeHail.innerText = "Alto Risco"; badgeHail.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-rose-600 text-white shadow"; }
                if (cardHail) cardHail.className = "glass-card rounded-2xl p-4 border-l-4 border-l-rose-500 flex flex-col justify-between";
            } else if (hailScore > 40) {
                if (badgeHail) { badgeHail.innerText = "Moderado"; badgeHail.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-amber-500 text-slate-950 font-bold shadow"; }
                if (cardHail) cardHail.className = "glass-card rounded-2xl p-4 border-l-4 border-l-amber-400 flex flex-col justify-between";
            } else {
                if (badgeHail) { badgeHail.innerText = "Baixo"; badgeHail.className = "px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700"; }
                if (cardHail) cardHail.className = "glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between";
            }
        }

        function renderForecastChart(hourly) {
            if (!hourly || !hourly.time) return;
            const canvasEl = document.getElementById('forecastChart');
            if (!canvasEl) return;
            const ctx = canvasEl.getContext('2d');
            const labels = [];
            const temps = hourly.temperature_2m.slice(0, 24);
            const rains = hourly.precipitation.slice(0, 24);

            const dayNames = ['Dom', 'Seg', 'Ter', 'Qua', 'Qui', 'Sex', 'Sáb'];

            hourly.time.slice(0, 24).forEach((tStr, index) => {
                const dateObj = new Date(tStr);
                const hour = dateObj.getHours().toString().padStart(2, '0');
                
                if (hour === '00' && index > 0) {
                    const dayName = dayNames[dateObj.getDay()];
                    const dayNum = dateObj.getDate().toString().padStart(2, '0');
                    const monthNum = (dateObj.getMonth() + 1).toString().padStart(2, '0');
                    labels.push(`| 00:00 (${dayName} ${dayNum}/${monthNum})`);
                } else {
                    labels.push(`${hour}:00`);
                }
            });

            if (chartInstance) chartInstance.destroy();
            chartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [
                        { 
                            type: 'line', 
                            label: 'Temperatura (°C)', 
                            data: temps, 
                            borderColor: '#ef4444', 
                            backgroundColor: 'rgba(239, 68, 68, 0.15)',
                            borderWidth: 3, 
                            pointBackgroundColor: '#ef4444',
                            pointHoverRadius: 6,
                            tension: 0.35,
                            fill: true,
                            yAxisID: 'yTemp' 
                        },
                        { 
                            type: 'bar', 
                            label: 'Chuva (mm)', 
                            data: rains, 
                            backgroundColor: 'rgba(56, 189, 248, 0.7)', 
                            borderRadius: 4,
                            yAxisID: 'yRain' 
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: { 
                            ticks: { 
                                color: function(context) {
                                    const val = context.tick ? context.tick.label : '';
                                    return (val && val.includes('|')) ? '#ef4444' : '#94a3b8';
                                },
                                font: function(context) {
                                    const val = context.tick ? context.tick.label : '';
                                    return (val && val.includes('|')) ? { weight: 'bold' } : {};
                                }
                            },
                            grid: { color: 'rgba(255, 255, 255, 0.05)' }
                        },
                        yTemp: { 
                            type: 'linear', 
                            position: 'left', 
                            ticks: { color: '#ef4444' },
                            grid: { color: 'rgba(239, 68, 68, 0.1)' }
                        },
                        yRain: { 
                            type: 'linear', 
                            position: 'right', 
                            ticks: { color: '#38bdf8' },
                            grid: { drawOnChartArea: false }
                        }
                    }
                }
            });
        }

        function renderHistoricalEvents() {
            const tbody = document.getElementById('modal-history-table-body');
            if (!tbody) return;
            let html = '';
            historicalEvents.forEach(item => {
                html += `
                    <tr class="hover:bg-slate-800/40 transition">
                        <td class="py-3 px-4 font-mono font-bold text-sky-400 whitespace-nowrap">${item.date}</td>
                        <td class="py-3 px-4 font-semibold text-slate-200">${item.type}</td>
                        <td class="py-3 px-4 font-mono text-amber-300">${item.value}</td>
                        <td class="py-3 px-4 text-slate-300">${item.impact}</td>
                    </tr>
                `;
            });
            tbody.innerHTML = html;
        }

        function render3DayForecast(daily) {
            const container = document.getElementById('forecast-3days-container');
            if (!container || !daily) return;
            let html = '';
            const weekDays = ['Domingo', 'Segunda', 'Terça', 'Quarta', 'Quinta', 'Sexta', 'Sábado'];
            for (let i = 1; i <= 4; i++) {
                if (!daily.time[i]) break;
                const d = new Date(daily.time[i] + 'T00:00:00');
                html += `
                    <div class="rounded-2xl dark:bg-slate-900 bg-slate-100 border dark:border-slate-800 border-slate-200 p-4 shadow">
                        <span class="text-[10px] font-bold text-sky-400 block uppercase">Dia +${i}</span>
                        <h3 class="font-bold text-sm dark:text-white text-slate-900">${weekDays[d.getDay()]} (${d.getDate()}/${d.getMonth()+1})</h3>
                        <div class="mt-3 text-xs font-mono flex justify-between">
                            <span class="text-sky-400">Min: ${daily.temperature_2m_min[i].toFixed(1)}°C</span>
                            <span class="text-rose-400">Máx: ${daily.temperature_2m_max[i].toFixed(1)}°C</span>
                        </div>
                        <div class="mt-2 text-xs text-slate-400">Chuva: <strong class="text-sky-300">${daily.precipitation_sum[i].toFixed(1)} mm</strong></div>
                    </div>
                `;
            }
            container.innerHTML = html;
        }

        window.onload = function() {
            renderHistoricalEvents();
            fetchWeatherData();
        };
    </script>
</body>
</html>
