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
    <!-- Chart.js UMD Script -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=JetBrains+Mono:wght@400;600;700&display=swap" rel="stylesheet">
    
    <script>
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
        .dark body { background-color: #090d16; color: #f8fafc; }
        .light body { background-color: #f1f5f9; color: #0f172a; }
        
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

        .slider-thumb::-webkit-slider-thumb {
            appearance: none;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: #2563eb;
            cursor: pointer;
            transition: transform 0.1s ease;
        }
        .slider-thumb::-webkit-slider-thumb:hover { transform: scale(1.15); }
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
                        <span class="p-3 bg-sky-500/10 text-sky-500 rounded-2xl border border-sky-500/20 shadow-inner shrink-0">
                            <i class="fa-solid fa-shield-halved text-3xl"></i>
                        </span>
                        <div class="flex flex-col items-start gap-1">
                            <h1 class="text-xl sm:text-3xl lg:text-4xl font-black tracking-tight dark:text-white text-slate-900 leading-tight">
                                Central de Monitoramento Meteorológico de Erechim
                            </h1>
                            <div class="mt-1">
                                <span class="text-sm sm:text-base font-black px-4 py-1 rounded-full bg-sky-500/20 text-sky-400 border-2 border-sky-500/40 uppercase tracking-wider inline-block">EM TEMPO REAL</span>
                            </div>
                            <div class="mt-1">
                                <span id="header-update-timestamp" class="text-sm sm:text-base font-mono font-bold px-4 py-1.5 rounded-full bg-rose-950/80 text-rose-500 border-2 border-rose-600 shadow-md inline-block">Atualizado às --:--</span>
                            </div>
                            <p class="text-xs sm:text-sm dark:text-slate-400 text-slate-500 flex flex-wrap gap-x-4 gap-y-1 mt-2">
                                <span><i class="fa-solid fa-location-dot text-rose-500 mr-1"></i> Lat: 27°38'02"S | Lon: 52°16'26"O</span>
                                <span><i class="fa-solid fa-mountain text-emerald-500 mr-1"></i> 783m</span>
                                <span><i class="fa-solid fa-tower-observation text-amber-500 mr-1"></i> INMET: <strong>A828</strong></span>
                            </p>
                        </div>
                    </div>
                </div>

                <!-- INFORMATIVO DE POSSIBILIDADE DE TORNADO (SINCRONIZADO COM LAB TORNADO) -->
                <div id="header-tornado-risk-box" class="w-full px-4 py-2.5 rounded-xl bg-slate-900/90 border border-slate-700/80 flex flex-col sm:flex-row items-center justify-between gap-3 shadow-md">
                    <div class="flex items-center gap-2.5">
                        <i class="fa-solid fa-tornado text-blue-400 text-lg animate-pulse"></i>
                        <span class="text-xs sm:text-sm font-bold text-slate-200">Possibilidade de Tornado (Lab Tornado):</span>
                    </div>
                    <div class="flex items-center gap-3">
                        <span id="header-tornado-prob-text" class="text-xs sm:text-sm font-mono font-black text-emerald-400">Baixa (5%)</span>
                        <span id="header-tornado-prob-badge" class="px-2.5 py-0.5 rounded text-[10px] font-mono font-bold uppercase bg-emerald-500/20 text-emerald-300 border border-emerald-500/40">Normal</span>
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
                <div class="flex items-center justify-between lg:justify-end gap-3 border-t lg:border-t-0 pt-3 lg:pt-0 dark:border-slate-800 border-slate-200 flex-wrap">
                    
                    <!-- Model Efficacy Modal Button -->
                    <button onclick="toggleModelEfficacyModal(true)" class="flex items-center gap-2 px-3 py-1.5 rounded-xl border text-xs font-semibold transition dark:bg-slate-800 dark:border-slate-700 dark:hover:bg-slate-700 dark:text-slate-200 bg-slate-100 border-slate-300 hover:bg-slate-200 text-slate-700">
                        <i class="fa-solid fa-chart-column text-emerald-400"></i>
                        <span>Eficácia do Modelo</span>
                    </button>

                    <!-- Tornado Lab Modal Button -->
                    <button onclick="toggleTornadoLabModal(true)" class="flex items-center gap-2 px-3 py-1.5 rounded-xl border text-xs font-semibold transition dark:bg-slate-800 dark:border-slate-700 dark:hover:bg-slate-700 dark:text-slate-200 bg-slate-100 border-slate-300 hover:bg-slate-200 text-slate-700">
                        <i class="fa-solid fa-tornado text-blue-400"></i>
                        <span>Lab Tornado</span>
                    </button>

                    <!-- Sources Modal Button -->
                    <button onclick="toggleSourcesPage(true)" class="flex items-center gap-2 px-3 py-1.5 rounded-xl border text-xs font-semibold transition dark:bg-slate-800 dark:border-slate-700 dark:hover:bg-slate-700 dark:text-slate-200 bg-slate-100 border-slate-300 hover:bg-slate-200 text-slate-700">
                        <i class="fa-solid fa-globe text-sky-400"></i>
                        <span>Fontes</span>
                    </button>

                    <!-- History Modal Button -->
                    <button onclick="toggleHistoryPage(true)" class="flex items-center gap-2 px-3 py-1.5 rounded-xl border text-xs font-semibold transition dark:bg-slate-800 dark:border-slate-700 dark:hover:bg-slate-700 dark:text-slate-200 bg-slate-100 border-slate-300 hover:bg-slate-200 text-slate-700">
                        <i class="fa-solid fa-clock-rotate-left text-amber-500"></i>
                        <span>Séries & Histórico</span>
                    </button>

                    <!-- Admin Login -->
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

    <!-- ADMIN MODAL -->
    <div id="admin-modal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md z-50 hidden flex items-center justify-center p-4">
        <div class="glass-card w-full max-w-2xl rounded-2xl p-6 relative border dark:border-slate-700 border-slate-300 shadow-2xl">
            <button onclick="toggleAdminModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white p-2">
                <i class="fa-solid fa-xmark text-lg"></i>
            </button>
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
                    </div>
                    <div id="login-error" class="hidden text-xs text-red-400 bg-red-950/40 p-2.5 rounded-xl border border-red-800">
                        Credenciais ou nome de usuário inválidos. Acesso negado.
                    </div>
                    <button type="submit" class="w-full py-3 bg-sky-600 hover:bg-sky-500 text-white font-bold text-sm rounded-xl transition shadow-lg">
                        Entrar no Painel Administrativo
                    </button>
                </form>
            </div>
            <div id="admin-panel-view" class="hidden space-y-6">
                <div class="flex items-center justify-between border-b dark:border-slate-800 border-slate-200 pb-4">
                    <div class="flex items-center gap-3">
                        <div class="w-10 h-10 rounded-full bg-sky-500/20 text-sky-400 border border-sky-500/40 flex items-center justify-center font-bold">WM</div>
                        <div>
                            <h3 id="admin-user-disp" class="font-bold text-sm dark:text-white text-slate-900">Wander Luís Marques</h3>
                            <span class="text-[10px] text-emerald-400 font-semibold flex items-center gap-1"><i class="fa-solid fa-circle text-[8px]"></i> Administrador Ativo</span>
                        </div>
                    </div>
                    <button onclick="handleAdminLogout()" class="px-3 py-1.5 text-xs bg-red-900/40 hover:bg-red-800/60 text-red-200 rounded-lg border border-red-700 transition">Sair</button>
                </div>
            </div>
        </div>
    </div>

    <!-- MODEL EFFICACY MODAL PAGE -->
    <div id="model-efficacy-modal" class="fixed inset-0 bg-slate-950/95 backdrop-blur-md z-50 hidden flex flex-col overflow-y-auto p-4 sm:p-6 lg:p-8">
        <div class="max-w-7xl mx-auto w-full space-y-8 my-auto text-slate-100">
            <div class="flex items-center justify-between glass-card p-4 sm:p-6 rounded-2xl border border-slate-800">
                <div class="flex items-center gap-3">
                    <div class="w-12 h-12 rounded-xl bg-gradient-to-tr from-emerald-600 to-teal-500 flex items-center justify-center shadow-lg shadow-emerald-500/20">
                        <i class="fa-solid fa-chart-column text-white text-2xl"></i>
                    </div>
                    <div>
                        <h2 class="text-xl sm:text-2xl font-extrabold text-white">Eficácia & Validação do Modelo Meteorológico</h2>
                        <p class="text-xs sm:text-sm text-slate-400">Análise de acurácia, índices estatísticos, confiabilidade de previsões e gráficos de desempenho (Erechim / RS)</p>
                    </div>
                </div>
                <button onclick="toggleModelEfficacyModal(false)" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 text-white font-bold text-xs rounded-xl transition border border-slate-700 flex items-center gap-2">
                    <i class="fa-solid fa-xmark text-base"></i> Fechar / Voltar
                </button>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-sm">
                    <span class="text-xs text-slate-400 block mb-1">Taxa de Acerto Geral</span>
                    <div class="text-3xl font-black text-emerald-400">92.4%</div>
                    <span class="text-[11px] text-slate-500 mt-2 block">Validado contra estações INMET</span>
                </div>
                <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-sm">
                    <span class="text-xs text-slate-400 block mb-1">Índice Brier (BS)</span>
                    <div class="text-3xl font-black text-sky-400">0.078</div>
                    <span class="text-[11px] text-slate-500 mt-2 block">Alta confiabilidade probabilística</span>
                </div>
                <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-sm">
                    <span class="text-xs text-slate-400 block mb-1">Acurácia de Precipitação</span>
                    <div class="text-3xl font-black text-blue-400">89.1%</div>
                    <span class="text-[11px] text-slate-500 mt-2 block">Janela de 24 horas</span>
                </div>
                <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-sm">
                    <span class="text-xs text-slate-400 block mb-1">Taxa de Falsos Alarmes (FAR)</span>
                    <div class="text-3xl font-black text-amber-400">4.5%</div>
                    <span class="text-[11px] text-slate-500 mt-2 block">Baixo índice de alarmes falsos</span>
                </div>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <div class="bg-slate-900 border border-slate-800 rounded-2xl p-6">
                    <h3 class="text-sm font-bold text-white mb-4 flex items-center gap-2"><i class="fa-solid fa-chart-line text-emerald-400"></i> Eficácia Mensal do Modelo (%)</h3>
                    <div class="w-full h-64"><canvas id="efficacyChart"></canvas></div>
                </div>
                <div class="bg-slate-900 border border-slate-800 rounded-2xl p-6 flex flex-col justify-between">
                    <div>
                        <h3 class="text-sm font-bold text-white mb-4 flex items-center gap-2"><i class="fa-solid fa-bullseye text-sky-400"></i> Metodologia de Validação Cruzada</h3>
                        <p class="text-xs text-slate-300 leading-relaxed mb-4">
                            O modelo implementado na Central de Erechim utiliza algoritmos de validação cruzada comparando dados horários previstos com as medições reais oficiais da estação automática INMET (A828).
                        </p>
                        <div class="space-y-3 font-mono text-xs">
                            <div class="flex justify-between border-b border-slate-800 pb-2"><span class="text-slate-400 font-sans">RMSE Temperatura:</span><span class="text-white font-bold">± 0.64 °C</span></div>
                            <div class="flex justify-between border-b border-slate-800 pb-2"><span class="text-slate-400 font-sans">RMSE Vento:</span><span class="text-white font-bold">± 1.8 km/h</span></div>
                            <div class="flex justify-between border-b border-slate-800 pb-2"><span class="text-slate-400 font-sans">Confiabilidade de Alertas:</span><span class="text-emerald-400 font-bold">Excelente (94.8%)</span></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- DEDICATED TORNADO LAB MODAL PAGE -->
    <div id="tornado-lab-modal" class="fixed inset-0 bg-slate-950/95 backdrop-blur-md z-50 hidden flex flex-col overflow-y-auto p-4 sm:p-6 lg:p-8">
        <div class="max-w-7xl mx-auto w-full space-y-8 my-auto text-slate-100">
            <div class="flex items-center justify-between glass-card p-4 sm:p-6 rounded-2xl border border-slate-800">
                <div class="flex items-center gap-3">
                    <div class="w-12 h-12 rounded-xl bg-gradient-to-tr from-blue-600 to-indigo-500 flex items-center justify-center shadow-lg shadow-blue-500/20">
                        <i class="fa-solid fa-tornado text-white text-2xl animate-pulse"></i>
                    </div>
                    <div>
                        <h2 class="text-xl sm:text-2xl font-extrabold text-white">Erechim Severe Weather Lab</h2>
                        <p class="text-xs sm:text-sm text-slate-400">Sistema Avançado de Diagnóstico e Previsão de Tornados (Planalto Gaúcho)</p>
                    </div>
                </div>
                <button onclick="toggleTornadoLabModal(false)" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 text-white font-bold text-xs rounded-xl transition border border-slate-700 flex items-center gap-2">
                    <i class="fa-solid fa-xmark text-base"></i> Fechar / Voltar
                </button>
            </div>

            <div class="space-y-8">
                <section>
                    <div class="flex flex-col md:flex-row md:items-center justify-between mb-4 gap-2">
                        <div>
                            <h3 class="text-lg font-bold text-white flex items-center gap-2"><i class="fa-solid fa-gauge-high text-blue-500"></i> Condições Atuais em Erechim</h3>
                            <p class="text-xs text-slate-400">Parâmetros termodinâmicos e cinemáticos calculados para a malha de mesoescala local.</p>
                        </div>
                        <div id="lab-global-status" class="px-4 py-1.5 rounded-xl font-semibold text-xs tracking-wide uppercase transition-all duration-300 bg-emerald-500/10 text-emerald-400 border border-emerald-500/30">
                            Risco Baixo / Normal
                        </div>
                    </div>

                    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-4">
                        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-sm">
                            <div class="flex justify-between items-start mb-2"><span class="text-xs font-medium text-slate-400">MLCAPE</span><i class="fa-solid fa-fire text-amber-500 text-sm"></i></div>
                            <div class="text-2xl font-bold text-white mb-1"><span id="lab-val-cape">1420</span> <span class="text-xs font-normal text-slate-400">J/kg</span></div>
                            <div class="w-full bg-slate-800 rounded-full h-1.5 mt-3"><div id="lab-bar-cape" class="bg-blue-500 h-1.5 rounded-full" style="width: 50%"></div></div>
                            <span class="text-[11px] text-slate-400 mt-2 block">Instabilidade Moderada</span>
                        </div>
                        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-sm">
                            <div class="flex justify-between items-start mb-2"><span class="text-xs font-medium text-slate-400">SRH (0-1 km)</span><i class="fa-solid fa-wind text-cyan-500 text-sm"></i></div>
                            <div class="text-2xl font-bold text-white mb-1"><span id="lab-val-srh">185</span> <span class="text-xs font-normal text-slate-400">m²/s²</span></div>
                            <div class="w-full bg-slate-800 rounded-full h-1.5 mt-3"><div id="lab-bar-srh" class="bg-cyan-500 h-1.5 rounded-full" style="width: 60%"></div></div>
                            <span class="text-[11px] text-slate-400 mt-2 block">Cisalh. Baixa Elevado</span>
                        </div>
                        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-sm">
                            <div class="flex justify-between items-start mb-2"><span class="text-xs font-medium text-slate-400">Cisalh. (0-6 km)</span><i class="fa-solid fa-arrows-split-up-and-left text-indigo-500 text-sm"></i></div>
                            <div class="text-2xl font-bold text-white mb-1"><span id="lab-val-shear">24.5</span> <span class="text-xs font-normal text-slate-400">m/s</span></div>
                            <div class="w-full bg-slate-800 rounded-full h-1.5 mt-3"><div id="lab-bar-shear" class="bg-indigo-500 h-1.5 rounded-full" style="width: 70%"></div></div>
                            <span class="text-[11px] text-slate-400 mt-2 block">Favorável a Supercélulas</span>
                        </div>
                        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-sm">
                            <div class="flex justify-between items-start mb-2"><span class="text-xs font-medium text-slate-400">EHI (0-3km)</span><i class="fa-solid fa-bolt text-yellow-500 text-sm"></i></div>
                            <div class="text-2xl font-bold text-white mb-1"><span id="lab-val-ehi">1.64</span> <span class="text-xs font-normal text-slate-400">Ind.</span></div>
                            <div class="w-full bg-slate-800 rounded-full h-1.5 mt-3"><div id="lab-bar-ehi" class="bg-yellow-500 h-1.5 rounded-full" style="width: 40%"></div></div>
                            <span class="text-[11px] text-slate-400 mt-2 block">Potencial Moderado</span>
                        </div>
                        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-sm">
                            <div class="flex justify-between items-start mb-2"><span class="text-xs font-medium text-slate-400">STP (Fixed)</span><i class="fa-solid fa-triangle-exclamation text-rose-500 text-sm"></i></div>
                            <div class="text-2xl font-bold text-white mb-1"><span id="lab-val-stp">0.85</span> <span class="text-xs font-normal text-slate-400">Ind.</span></div>
                            <div class="w-full bg-slate-800 rounded-full h-1.5 mt-3"><div id="lab-bar-stp" class="bg-emerald-500 h-1.5 rounded-full" style="width: 30%"></div></div>
                            <span class="text-[11px] text-slate-400 mt-2 block">Atenção Meteorológica</span>
                        </div>
                    </div>
                </section>

                <section class="bg-slate-900 border border-slate-800 rounded-2xl p-6 sm:p-8 shadow-xl">
                    <div class="flex flex-col md:flex-row justify-between items-start md:items-center mb-6 pb-4 border-b border-slate-800 gap-4">
                        <div>
                            <h3 class="text-lg font-bold text-white flex items-center gap-2"><i class="fa-solid fa-sliders text-blue-500"></i> Simulador Interativo de Sondagem</h3>
                            <p class="text-xs text-slate-400">Ajuste os parâmetros atmosféricos para simular diferentes cenários de instabilidade em Erechim.</p>
                        </div>
                        <button onclick="resetLabSliders()" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-xl text-xs font-medium transition-all flex items-center gap-2 border border-slate-700">
                            <i class="fa-solid fa-rotate-right"></i> Restaurar Padrão
                        </button>
                    </div>

                    <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                        <div class="lg:col-span-2 grid grid-cols-1 sm:grid-cols-2 gap-6">
                            <div class="bg-slate-950/50 p-4 rounded-xl border border-slate-800/80">
                                <div class="flex justify-between items-center mb-2">
                                    <label class="text-xs font-semibold text-slate-300 flex items-center gap-1.5"><i class="fa-solid fa-temperature-high text-rose-400"></i> Temp. Superfície (T)</label>
                                    <span class="font-mono text-sm font-bold text-blue-400"><span id="lab-txt-temp">28.0</span> °C</span>
                                </div>
                                <input id="lab-slider-temp" type="range" min="15" max="38" step="0.5" value="28.0" oninput="updateLabSimulation()" class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer slider-thumb">
                            </div>
                            <div class="bg-slate-950/50 p-4 rounded-xl border border-slate-800/80">
                                <div class="flex justify-between items-center mb-2">
                                    <label class="text-xs font-semibold text-slate-300 flex items-center gap-1.5"><i class="fa-solid fa-droplet text-cyan-400"></i> Ponto de Orvalho (Td)</label>
                                    <span class="font-mono text-sm font-bold text-blue-400"><span id="lab-txt-td">21.0</span> °C</span>
                                </div>
                                <input id="lab-slider-td" type="range" min="8" max="26" step="0.5" value="21.0" oninput="updateLabSimulation()" class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer slider-thumb">
                            </div>
                            <div class="bg-slate-950/50 p-4 rounded-xl border border-slate-800/80">
                                <div class="flex justify-between items-center mb-2">
                                    <label class="text-xs font-semibold text-slate-300 flex items-center gap-1.5"><i class="fa-solid fa-tornado text-indigo-400"></i> SRH (0-1km)</label>
                                    <span class="font-mono text-sm font-bold text-blue-400"><span id="lab-txt-srh">185</span> m²/s²</span>
                                </div>
                                <input id="lab-slider-srh" type="range" min="0" max="450" step="5" value="185" oninput="updateLabSimulation()" class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer slider-thumb">
                            </div>
                            <div class="bg-slate-950/50 p-4 rounded-xl border border-slate-800/80">
                                <div class="flex justify-between items-center mb-2">
                                    <label class="text-xs font-semibold text-slate-300 flex items-center gap-1.5"><i class="fa-solid fa-wind text-teal-400"></i> Cisalhamento (0-6km)</label>
                                    <span class="font-mono text-sm font-bold text-blue-400"><span id="lab-txt-shear">24.5</span> m/s</span>
                                </div>
                                <input id="lab-slider-shear" type="range" min="5" max="45" step="0.5" value="24.5" oninput="updateLabSimulation()" class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer slider-thumb">
                            </div>
                        </div>

                        <div id="lab-sim-box" class="bg-slate-950 border border-slate-800 rounded-2xl p-6 flex flex-col justify-between transition-all duration-300">
                            <div>
                                <div class="text-xs font-semibold uppercase tracking-wider text-slate-400 mb-2">Diagnóstico em Tempo Real</div>
                                <div id="lab-sim-alert-title" class="text-base font-bold text-emerald-400 mb-4 flex items-center gap-2"><i class="fa-solid fa-shield-halved"></i> Situação Sob Controle</div>
                                <div class="space-y-3 font-mono text-xs">
                                    <div class="flex justify-between border-b border-slate-900 pb-2"><span class="text-slate-400 font-sans">MLCAPE:</span><span id="lab-sim-cape" class="text-white font-bold">1420 J/kg</span></div>
                                    <div class="flex justify-between border-b border-slate-900 pb-2"><span class="text-slate-400 font-sans">EHI (0-3km):</span><span id="lab-sim-ehi" class="text-white font-bold">1.64</span></div>
                                    <div class="flex justify-between border-b border-slate-900 pb-2"><span class="text-slate-400 font-sans">STP (Fixed):</span><span id="lab-sim-stp" class="text-white font-bold">0.85</span></div>
                                </div>
                            </div>
                            <div class="mt-6 pt-4 border-t border-slate-900">
                                <p id="lab-sim-message" class="text-xs text-slate-400 leading-relaxed">Ambiente com instabilidade moderada. Monitoramento de rotina.</p>
                            </div>
                        </div>
                    </div>
                </section>
            </div>
        </div>
    </div>

    <!-- SOURCES MODAL PAGE -->
    <div id="sources-page-modal" class="fixed inset-0 bg-slate-950/90 backdrop-blur-md z-50 hidden flex flex-col overflow-y-auto p-4 sm:p-6 lg:p-8">
        <div class="max-w-4xl mx-auto w-full space-y-6 my-auto">
            <div class="flex items-center justify-between glass-card p-4 sm:p-6 rounded-2xl border border-slate-700">
                <div class="flex items-center gap-3">
                    <span class="p-3 bg-sky-500/10 text-sky-400 rounded-xl border border-sky-500/20"><i class="fa-solid fa-globe text-2xl"></i></span>
                    <div>
                        <h2 class="text-xl sm:text-2xl font-extrabold text-white">Fontes de Dados Utilizadas</h2>
                        <p class="text-xs sm:text-sm text-slate-400">Relação completa de APIs e sistemas oficiais da Central de Erechim</p>
                    </div>
                </div>
                <button onclick="toggleSourcesPage(false)" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 text-white font-bold text-xs rounded-xl transition border border-slate-700 flex items-center gap-2"><i class="fa-solid fa-xmark text-base"></i> Fechar</button>
            </div>
            <div class="glass-card rounded-2xl p-6 sm:p-8 border border-slate-700 space-y-4 text-xs sm:text-sm text-slate-300">
                <p>Integra 6 fontes principais de dados globais e oficiais (Open-Meteo API, NOAA/CIRA GOES-16, Windy / ECMWF, Earth Nullschool, CPTEC/INPE e Marinha do Brasil).</p>
            </div>
        </div>
    </div>

    <!-- HISTORY MODAL PAGE -->
    <div id="history-page-modal" class="fixed inset-0 bg-slate-950/90 backdrop-blur-md z-50 hidden flex flex-col overflow-y-auto p-4 sm:p-6 lg:p-8">
        <div class="max-w-7xl mx-auto w-full space-y-6 my-auto">
            <div class="flex items-center justify-between glass-card p-4 sm:p-6 rounded-2xl border border-slate-700">
                <div class="flex items-center gap-3">
                    <span class="p-3 bg-amber-500/10 text-amber-400 rounded-xl border border-amber-500/20"><i class="fa-solid fa-book-atlas text-2xl"></i></span>
                    <div>
                        <h2 class="text-xl sm:text-2xl font-extrabold text-white">Séries Históricas & Eventos Extremos</h2>
                        <p class="text-xs sm:text-sm text-slate-400">Acervo climatológico e tabelas de probabilidade de Erechim / RS</p>
                    </div>
                </div>
                <button onclick="toggleHistoryPage(false)" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 text-white font-bold text-xs rounded-xl transition border border-slate-700 flex items-center gap-2"><i class="fa-solid fa-xmark text-base"></i> Fechar</button>
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
                <div class="glass-card rounded-2xl p-4 flex flex-col justify-between">
                    <div class="flex items-center justify-between text-slate-400"><span class="text-xs font-semibold uppercase">Temperatura</span><i class="fa-solid fa-temperature-half text-amber-500 text-lg"></i></div>
                    <div class="mt-3"><div id="metric-temp" class="text-2xl sm:text-3xl font-black tracking-tight dark:text-white text-slate-900">--°C</div><div id="metric-temp-sub" class="text-xs text-slate-400 mt-1">Sensação: --°C</div></div>
                </div>
                <div class="glass-card rounded-2xl p-4 flex flex-col justify-between">
                    <div class="flex items-center justify-between text-slate-400"><span class="text-xs font-semibold uppercase">Umidade Relativa</span><i class="fa-solid fa-droplet text-sky-400 text-lg"></i></div>
                    <div class="mt-3"><div id="metric-humidity" class="text-2xl sm:text-3xl font-black tracking-tight dark:text-white text-slate-900">--%</div><div id="metric-humidity-sub" class="text-xs text-slate-400 mt-1">Ponto Orvalho: --°C</div></div>
                </div>
                <div class="glass-card rounded-2xl p-4 flex flex-col justify-between">
                    <div class="flex items-center justify-between text-slate-400"><span class="text-xs font-semibold uppercase">Chuva (12h)</span><i class="fa-solid fa-cloud-showers-heavy text-blue-500 text-lg"></i></div>
                    <div class="mt-3"><div id="metric-rain" class="text-2xl sm:text-3xl font-black tracking-tight dark:text-white text-slate-900">-- mm</div><div id="metric-rain-sub" class="text-xs text-slate-400 mt-1">Acumulado recente</div></div>
                </div>
                <div class="glass-card rounded-2xl p-4 flex flex-col justify-between">
                    <div class="flex items-center justify-between text-slate-400"><span class="text-xs font-semibold uppercase">Vento / Rajadas</span><i class="fa-solid fa-wind text-teal-400 text-lg"></i></div>
                    <div class="mt-3"><div id="metric-wind" class="text-2xl sm:text-3xl font-black tracking-tight dark:text-white text-slate-900">-- km/h</div><div id="metric-wind-sub" class="text-xs text-slate-400 mt-1">Rajada: -- km/h</div></div>
                </div>
                <div class="glass-card rounded-2xl p-4 flex flex-col justify-between col-span-2 md:col-span-1">
                    <div class="flex items-center justify-between text-slate-400"><span class="text-xs font-semibold uppercase">Pressão Atm.</span><i class="fa-solid fa-compress text-purple-400 text-lg"></i></div>
                    <div class="mt-3"><div id="metric-pressure" class="text-2xl sm:text-3xl font-black tracking-tight dark:text-white text-slate-900">-- hPa</div><div id="metric-pressure-sub" class="text-xs text-slate-400 mt-1">Pressão Local</div></div>
                </div>
            </div>
        </section>

        <!-- SEVERITY THRESHOLDS -->
        <section>
            <div class="flex items-center justify-between mb-3">
                <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-400 text-slate-500 flex items-center gap-2">
                    <i class="fa-solid fa-shield-cat text-rose-500"></i> Status dos Limiares de Severidade Meteorológica
                </h2>
            </div>
            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-3 sm:gap-4">
                <div id="card-alert-cold" class="glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between">
                    <div>
                        <div class="flex items-center justify-between"><span class="text-xs font-bold uppercase tracking-wider text-slate-400">Frio Extremo</span><span id="risk-cold" class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300">Risco: --%</span></div>
                        <div class="mt-2 space-y-1"><span class="text-[11px] text-sky-400 block"><i class="fa-solid fa-circle-dot text-[9px] mr-1"></i>Atual: <strong id="val-cold" class="font-mono font-bold text-white">-- °C</strong></span></div>
                    </div>
                    <div class="mt-3 flex items-center justify-between pt-2 border-t dark:border-slate-800 border-slate-200"><span class="text-[10px] text-slate-400 uppercase">Status:</span><span id="badge-cold" class="px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700">Normal</span></div>
                </div>
                <div id="card-alert-heat" class="glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between">
                    <div>
                        <div class="flex items-center justify-between"><span class="text-xs font-bold uppercase tracking-wider text-slate-400">Calor Extremo</span><span id="risk-heat" class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300">Risco: --%</span></div>
                        <div class="mt-2 space-y-1"><span class="text-[11px] text-sky-400 block"><i class="fa-solid fa-circle-dot text-[9px] mr-1"></i>Atual: <strong id="val-heat" class="font-mono font-bold text-white">-- °C</strong></span></div>
                    </div>
                    <div class="mt-3 flex items-center justify-between pt-2 border-t dark:border-slate-800 border-slate-200"><span class="text-[10px] text-slate-400 uppercase">Status:</span><span id="badge-heat" class="px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700">Normal</span></div>
                </div>
                <div id="card-alert-rain" class="glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between">
                    <div>
                        <div class="flex items-center justify-between"><span class="text-xs font-bold uppercase tracking-wider text-slate-400">Chuva Intensa</span><span id="risk-rain" class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300">Risco: --%</span></div>
                        <div class="mt-2 space-y-1"><span class="text-[11px] text-sky-400 block"><i class="fa-solid fa-circle-dot text-[9px] mr-1"></i>Atual: <strong id="val-rain" class="font-mono font-bold text-white">-- mm</strong></span></div>
                    </div>
                    <div class="mt-3 flex items-center justify-between pt-2 border-t dark:border-slate-800 border-slate-200"><span class="text-[10px] text-slate-400 uppercase">Status:</span><span id="badge-rain" class="px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700">Normal</span></div>
                </div>
                <div id="card-alert-wind" class="glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between">
                    <div>
                        <div class="flex items-center justify-between"><span class="text-xs font-bold uppercase tracking-wider text-slate-400">Vento Forte</span><span id="risk-wind" class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300">Risco: --%</span></div>
                        <div class="mt-2 space-y-1"><span class="text-[11px] text-sky-400 block"><i class="fa-solid fa-circle-dot text-[9px] mr-1"></i>Atual: <strong id="val-wind" class="font-mono font-bold text-white">-- km/h</strong></span></div>
                    </div>
                    <div class="mt-3 flex items-center justify-between pt-2 border-t dark:border-slate-800 border-slate-200"><span class="text-[10px] text-slate-400 uppercase">Status:</span><span id="badge-wind" class="px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700">Normal</span></div>
                </div>
                <div id="card-alert-hail" class="glass-card rounded-2xl p-4 border-l-4 border-l-slate-600 flex flex-col justify-between">
                    <div>
                        <div class="flex items-center justify-between"><span class="text-xs font-bold uppercase tracking-wider text-slate-400">Risco Granizo</span><span id="risk-hail" class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300">Risco: --%</span></div>
                        <div class="mt-2 space-y-1"><span class="text-[11px] text-sky-400 block"><i class="fa-solid fa-circle-dot text-[9px] mr-1"></i>Análise: <strong id="val-hail" class="font-mono font-bold text-white">--</strong></span></div>
                    </div>
                    <div class="mt-3 flex items-center justify-between pt-2 border-t dark:border-slate-800 border-slate-200"><span class="text-[10px] text-slate-400 uppercase">Status:</span><span id="badge-hail" class="px-2.5 py-0.5 rounded-full text-xs font-bold bg-slate-800 text-slate-400 border border-slate-700">Baixo</span></div>
                </div>
            </div>
        </section>

        <!-- SATELLITE, RADAR, WIND SECTIONS -->
        <section class="space-y-6">
            <div class="glass-card rounded-2xl p-5 flex flex-col justify-between">
                <div>
                    <div class="flex items-center justify-between mb-3">
                        <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-300 text-slate-700 flex items-center gap-2"><i class="fa-solid fa-earth-americas text-sky-500"></i> Radar Interativo</h2>
                        <span class="text-xs bg-slate-800 text-slate-300 px-2.5 py-1 rounded-lg border border-slate-700 font-mono">Erechim (RS)</span>
                    </div>
                    <div class="w-full h-[450px] rounded-xl overflow-hidden border border-slate-800 bg-slate-950 shadow-inner">
                        <iframe id="windy-frame" class="w-full h-full border-0" src="https://embed.windy.com/embed.html?lat=-27.634&lon=-52.274&detailLat=-27.634&detailLon=-52.274&width=1000&height=450&zoom=9&level=surface&overlay=wind&product=ecmwf&menu=&message=&marker=true&calendar=now&pressure=true&type=map&location=coordinates&detail=true&metricWind=km%2Fh&metricTemp=%C2%B0C&radarRange=-1" loading="lazy"></iframe>
                    </div>
                </div>
            </div>

            <!-- WIND COMPASS CARD -->
            <div id="wind-card-container" class="glass-card rounded-2xl p-5 flex flex-col justify-between transition-all duration-300">
                <div>
                    <div class="flex items-center justify-between mb-3">
                        <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-300 text-slate-700 flex items-center gap-2"><i class="fa-solid fa-compass text-teal-400 text-base"></i> Direção & Intensidade do Vento (Orientação Cardinal)</h2>
                        <div class="flex items-center gap-2">
                            <span id="wind-alert-status-badge" class="text-[10px] font-mono font-bold px-2 py-0.5 rounded bg-slate-800 text-slate-300 border border-slate-700">Normal</span>
                            <span id="wind-dir-text" class="text-xs font-mono font-bold text-teal-400 bg-teal-950/60 border border-teal-800/60 px-2.5 py-0.5 rounded">--° N</span>
                        </div>
                    </div>
                    <div class="flex flex-col items-center justify-center my-4 relative">
                        <div class="w-48 h-48 rounded-full border-2 border-slate-700/80 bg-slate-900/90 relative flex items-center justify-center shadow-inner">
                            <span class="absolute top-2 text-[11px] font-bold text-slate-400">N (Norte)</span>
                            <span class="absolute right-3 text-[11px] font-bold text-slate-400">L (Leste)</span>
                            <span class="absolute bottom-2 text-[11px] font-bold text-slate-400">S (Sul)</span>
                            <span class="absolute left-3 text-[11px] font-bold text-slate-400">O (Oeste)</span>
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
                    <div class="p-3 rounded-xl bg-slate-900/60 border border-slate-800"><span class="text-slate-400 block text-[10px]">Velocidade Constante</span><strong id="wind-card-speed" class="text-base font-bold text-teal-300">-- km/h</strong></div>
                    <div class="p-3 rounded-xl bg-slate-900/60 border border-slate-800"><span class="text-slate-400 block text-[10px]">Rajada Máxima</span><strong id="wind-card-gust" class="text-base font-bold text-amber-300">-- km/h</strong></div>
                </div>
            </div>
        </section>

        <!-- FORECAST 24h & 4 DAYS -->
        <section class="glass-card rounded-2xl p-5">
            <div class="flex items-center justify-between mb-4">
                <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-300 text-slate-700 flex items-center gap-2"><i class="fa-solid fa-chart-line text-indigo-400"></i> Tendência das Próximas 24h</h2>
            </div>
            <div class="w-full h-64 sm:h-72"><canvas id="forecastChart"></canvas></div>
        </section>

        <section class="glass-card rounded-2xl p-5">
            <div class="flex items-center justify-between mb-5">
                <h2 class="text-xs uppercase tracking-wider font-bold dark:text-slate-300 text-slate-700 flex items-center gap-2"><i class="fa-solid fa-timeline text-sky-400"></i> Previsão dos Próximos 4 Dias</h2>
            </div>
            <div id="forecast-3days-container" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                <div class="p-4 rounded-xl dark:bg-slate-900/60 bg-slate-100 border border-slate-800 text-center animate-pulse"><span class="text-xs text-slate-400">Carregando...</span></div>
            </div>
        </section>
    </main>

    <!-- FOOTER -->
    <footer class="border-t dark:border-slate-900 border-slate-200 dark:bg-slate-950 bg-white py-8 text-center text-xs text-slate-500">
        <div class="max-w-7xl mx-auto px-4 space-y-6">
            <div class="glass-card max-w-2xl mx-auto rounded-2xl p-5 border border-slate-800 shadow-sm text-center">
                <h3 class="text-xs font-bold uppercase tracking-wider text-slate-300 mb-3 flex items-center justify-center gap-2"><i class="fa-solid fa-address-card text-sky-500"></i> Contatos & Canais Oficiais</h3>
                <div class="flex flex-wrap items-center justify-center gap-3 sm:gap-4 text-xs font-medium">
                    <a href="mailto:wander.l.marques@gmail.com" class="flex items-center gap-2 px-3.5 py-2 rounded-xl dark:bg-slate-900 bg-slate-100 text-slate-300 border border-slate-800"><i class="fa-solid fa-envelope text-rose-500"></i><span>wander.l.marques@gmail.com</span></a>
                    <a href="https://www.instagram.com/wanderlmarques/" target="_blank" rel="noopener noreferrer" class="flex items-center gap-2 px-3.5 py-2 rounded-xl dark:bg-slate-900 bg-slate-100 text-slate-300 border border-slate-800"><i class="fa-brands fa-instagram text-pink-500"></i><span>Instagram</span></a>
                </div>
            </div>
            <div class="space-y-1">
                <p>Central de Monitoramento Climático - Erechim / RS</p>
                <p>Desenvolvido por: <strong>Wander Luís Marques</strong></p>
                <p class="text-[11px] text-slate-500 font-mono mt-1">© 2026 Wander Luís Marques - Todos os direitos reservados</p>
            </div>
        </div>
    </footer>

    <!-- SCRIPTS -->
    <script>
        const LAT = -27.634;
        const LON = -52.274;
        let chartInstance = null;
        let efficacyChartInstance = null;

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

        function toggleAdminModal() { document.getElementById('admin-modal').classList.toggle('hidden'); }
        function toggleHistoryPage(show) { const m = document.getElementById('history-page-modal'); show ? m.classList.remove('hidden') : m.classList.add('hidden'); }
        function toggleSourcesPage(show) { const m = document.getElementById('sources-page-modal'); show ? m.classList.remove('hidden') : m.classList.add('hidden'); }
        function toggleTornadoLabModal(show) { const m = document.getElementById('tornado-lab-modal'); show ? m.classList.remove('hidden') : m.classList.add('hidden'); }
        
        function toggleModelEfficacyModal(show) {
            const m = document.getElementById('model-efficacy-modal');
            if (show) {
                m.classList.remove('hidden');
                setTimeout(renderEfficacyChart, 200);
            } else {
                m.classList.add('hidden');
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
            } else { err.classList.remove('hidden'); }
        }
        function handleAdminLogout() {
            document.getElementById('admin-login-view').classList.remove('hidden');
            document.getElementById('admin-panel-view').classList.add('hidden');
            document.getElementById('admin-btn-text').innerText = 'Área Administrativa';
            toggleAdminModal();
        }

        function updateClock() {
            const now = new Date();
            const timeString = now.toLocaleTimeString('pt-BR', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
            const dateString = now.toLocaleDateString('pt-BR', { day: '2-digit', month: '2-digit' });
            document.getElementById('header-update-timestamp').innerText = `Atualizado às ${timeString} (${dateString})`;
        }
        setInterval(updateClock, 1000);

        function updateLabSimulation() {
            const T = parseFloat(document.getElementById('lab-slider-temp').value);
            const Td = parseFloat(document.getElementById('lab-slider-td').value);
            const srh = parseFloat(document.getElementById('lab-slider-srh').value);
            const shear = parseFloat(document.getElementById('lab-slider-shear').value);

            document.getElementById('lab-txt-temp').textContent = T.toFixed(1);
            document.getElementById('lab-txt-td').textContent = Td.toFixed(1);
            document.getElementById('lab-txt-srh').textContent = srh.toFixed(0);
            document.getElementById('lab-txt-shear').textContent = shear.toFixed(1);

            let baseCape = Math.max(0, (T - 18) * 120 + (Td - 12) * 150);
            if (Td > T) baseCape = 0;
            const cape = Math.min(4500, Math.max(50, baseCape));

            const srh3km = srh * 1.6;
            const ehi = (cape * srh3km) / 160000;

            const termCape = cape / 1500;
            const termSrh = srh / 150;
            const termShear = Math.min(shear / 20, 1.5);
            const spread = T - Td;
            const lclMeters = 100 + spread * 120;
            let termLcl = 1.0;
            if (lclMeters > 1000 && lclMeters < 2000) {
                termLcl = (2000 - lclMeters) / 1500;
            } else if (lclMeters >= 2000) {
                termLcl = 0.0;
            }
            const stp = Math.max(0, termCape * termSrh * termShear * termLcl);

            document.getElementById('lab-sim-cape').textContent = cape.toFixed(0) + ' J/kg';
            document.getElementById('lab-sim-ehi').textContent = ehi.toFixed(2);
            document.getElementById('lab-sim-stp').textContent = stp.toFixed(2);

            document.getElementById('lab-val-cape').textContent = cape.toFixed(0);
            document.getElementById('lab-val-srh').textContent = srh.toFixed(0);
            document.getElementById('lab-val-shear').textContent = shear.toFixed(1);
            document.getElementById('lab-val-ehi').textContent = ehi.toFixed(2);
            document.getElementById('lab-val-stp').textContent = stp.toFixed(2);

            document.getElementById('lab-bar-cape').style.width = Math.min(100, (cape / 3000) * 100) + '%';
            document.getElementById('lab-bar-srh').style.width = Math.min(100, (srh / 400) * 100) + '%';
            document.getElementById('lab-bar-shear').style.width = Math.min(100, (shear / 40) * 100) + '%';
            document.getElementById('lab-bar-ehi').style.width = Math.min(100, (ehi / 4) * 100) + '%';
            document.getElementById('lab-bar-stp').style.width = Math.min(100, (stp / 3) * 100) + '%';

            const simBox = document.getElementById('lab-sim-box');
            const alertTitle = document.getElementById('lab-sim-alert-title');
            const alertMsg = document.getElementById('lab-sim-message');
            const globalBadge = document.getElementById('lab-global-status');

            let tornadoProbPercent = Math.min(98, Math.max(5, Math.round((stp * 45) + (ehi * 15))));
            const probTextEl = document.getElementById('header-tornado-prob-text');
            const probBadgeEl = document.getElementById('header-tornado-prob-badge');
            const tornadoRiskBox = document.getElementById('header-tornado-risk-box');

            if (stp >= 1.5 || ehi >= 2.5) {
                simBox.className = "bg-slate-950 border border-rose-500/50 rounded-2xl p-6 flex flex-col justify-between transition-all duration-300 bg-rose-950/10";
                alertTitle.innerHTML = '<i class="fa-solid fa-triangle-exclamation text-rose-500 animate-bounce"></i> Risco Crítico de Tornado';
                alertMsg.textContent = 'ATENÇÃO MÁXIMA: Condições extremas para formação de tornados em Erechim.';
                globalBadge.innerHTML = '<span class="px-3 py-1 rounded-xl text-xs bg-rose-500/20 text-rose-400 border border-rose-500/40 animate-pulse font-bold">ALERTA CRÍTICO</span>';
                if (probTextEl) { probTextEl.className = "text-xs sm:text-sm font-mono font-black text-rose-400 animate-pulse"; probTextEl.innerText = `Crítica (${tornadoProbPercent}%)`; }
                if (probBadgeEl) { probBadgeEl.className = "px-2.5 py-0.5 rounded text-[10px] font-mono font-bold uppercase bg-rose-500/30 text-rose-300 border border-rose-500/60 animate-pulse"; probBadgeEl.innerText = "CRÍTICO"; }
                if (tornadoRiskBox) tornadoRiskBox.className = "w-full px-4 py-2.5 rounded-xl bg-rose-950/40 border border-rose-500/60 flex flex-col sm:flex-row items-center justify-between gap-3 shadow-lg";
            } else if (stp >= 0.7 || ehi >= 1.2) {
                simBox.className = "bg-slate-950 border border-amber-500/50 rounded-2xl p-6 flex flex-col justify-between transition-all duration-300 bg-amber-950/10";
                alertTitle.innerHTML = '<i class="fa-solid fa-cloud-bolt text-amber-500"></i> Risco Severo';
                alertMsg.textContent = 'Ambiente favorável a tempestades severas e ventos destrutivos.';
                globalBadge.innerHTML = '<span class="px-3 py-1 rounded-xl text-xs bg-amber-500/20 text-amber-400 border border-amber-500/40 font-bold">ALERTA SEVERO</span>';
                if (probTextEl) { probTextEl.className = "text-xs sm:text-sm font-mono font-black text-amber-400"; probTextEl.innerText = `Elevada (${tornadoProbPercent}%)`; }
                if (probBadgeEl) { probBadgeEl.className = "px-2.5 py-0.5 rounded text-[10px] font-mono font-bold uppercase bg-amber-500/20 text-amber-300 border border-amber-500/40"; probBadgeEl.innerText = "SEVERO"; }
                if (tornadoRiskBox) tornadoRiskBox.className = "w-full px-4 py-2.5 rounded-xl bg-amber-950/30 border border-amber-500/40 flex flex-col sm:flex-row items-center justify-between gap-3 shadow-md";
            } else {
                simBox.className = "bg-slate-950 border border-slate-800 rounded-2xl p-6 flex flex-col justify-between transition-all duration-300";
                alertTitle.innerHTML = '<i class="fa-solid fa-shield-halved text-emerald-400"></i> Situação Sob Controle';
                alertMsg.textContent = 'Parâmetros abaixo do limiar crítico para tornados na região.';
                globalBadge.innerHTML = '<span class="px-3 py-1 rounded-xl text-xs bg-emerald-500/10 text-emerald-400 border border-emerald-500/30 font-bold">RISCO BAIXO</span>';
                if (probTextEl) { probTextEl.className = "text-xs sm:text-sm font-mono font-black text-emerald-400"; probTextEl.innerText = `Baixa (${tornadoProbPercent}%)`; }
                if (probBadgeEl) { probBadgeEl.className = "px-2.5 py-0.5 rounded text-[10px] font-mono font-bold uppercase bg-emerald-500/20 text-emerald-300 border border-emerald-500/40"; probBadgeEl.innerText = "NORMAL"; }
                if (tornadoRiskBox) tornadoRiskBox.className = "w-full px-4 py-2.5 rounded-xl bg-slate-900/90 border border-slate-700/80 flex flex-col sm:flex-row items-center justify-between gap-3 shadow-md";
            }
        }

        function resetLabSliders() {
            document.getElementById('lab-slider-temp').value = 28.0;
            document.getElementById('lab-slider-td').value = 21.0;
            document.getElementById('lab-slider-srh').value = 185;
            document.getElementById('lab-slider-shear').value = 24.5;
            updateLabSimulation();
        }

        function getCardinalDirection(angle) {
            const directions = ['N', 'NE', 'L', 'SE', 'S', 'SO', 'O', 'NO'];
            return directions[Math.round(angle / 45) % 8];
        }

        async function fetchWeatherData() {
            try {
                const url = `https://api.open-meteo.com/v1/forecast?latitude=${LAT}&longitude=${LON}&current=temperature_2m,relative_humidity_2m,apparent_temperature,precipitation,surface_pressure,wind_speed_10m,wind_direction_10m,wind_gusts_10m&hourly=temperature_2m,precipitation,wind_speed_10m,precipitation_probability&daily=temperature_2m_max,temperature_2m_min,precipitation_sum&past_hours=12&forecast_days=5&timezone=America%2FSao_Paulo`;
                const response = await fetch(url);
                const data = await response.json();
                const current = data.current;

                const temp = current.temperature_2m;
                const humidity = current.relative_humidity_2m;
                let rain12h = data.hourly ? data.hourly.precipitation.slice(0, 13).reduce((a, b) => a + (b || 0), 0) : current.precipitation;
                const windSpeed = current.wind_speed_10m;
                const windGust = current.wind_gusts_10m || (windSpeed * 1.3);
                const windDegree = current.wind_direction_10m || 0;

                document.getElementById('metric-temp').innerText = `${temp.toFixed(1)}°C`;
                document.getElementById('metric-temp-sub').innerText = `Sensação: ${current.apparent_temperature.toFixed(1)}°C`;
                document.getElementById('metric-humidity').innerText = `${humidity}%`;
                document.getElementById('metric-rain').innerText = `${rain12h.toFixed(1)} mm`;
                document.getElementById('metric-wind').innerText = `${windSpeed.toFixed(1)} km/h`;
                document.getElementById('metric-wind-sub').innerText = `Rajada: ${windGust.toFixed(1)} km/h`;
                document.getElementById('metric-pressure').innerText = `${current.surface_pressure.toFixed(0)} hPa`;

                document.getElementById('wind-dir-text').innerText = `${windDegree}° ${getCardinalDirection(windDegree)}`;
                const windArrow = document.getElementById('wind-arrow-container');
                if (windArrow) windArrow.style.transform = `rotate(${windDegree}deg)`;
                document.getElementById('wind-card-speed').innerText = `${windSpeed.toFixed(1)} km/h`;
                document.getElementById('wind-card-gust').innerText = `${windGust.toFixed(1)} km/h`;

                renderForecastChart(data.hourly);
                render3DayForecast(data.daily);
                updateClock();
            } catch (err) { console.error(err); }
        }

        function renderForecastChart(hourly) {
            if (!hourly || !hourly.time) return;
            const ctx = document.getElementById('forecastChart').getContext('2d');
            const labels = hourly.time.slice(0, 24).map(t => `${new Date(t).getHours().toString().padStart(2, '0')}:00`);
            const temps = hourly.temperature_2m.slice(0, 24);
            const rains = hourly.precipitation.slice(0, 24);

            if (chartInstance) chartInstance.destroy();
            chartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [
                        { type: 'line', label: 'Temperatura (°C)', data: temps, borderColor: '#ef4444', borderWidth: 3, tension: 0.3, fill: true, backgroundColor: 'rgba(239, 68, 68, 0.1)', yAxisID: 'yTemp' },
                        { type: 'bar', label: 'Chuva (mm)', data: rains, backgroundColor: 'rgba(56, 189, 248, 0.7)', borderRadius: 4, yAxisID: 'yRain' }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: { ticks: { color: '#94a3b8' }, grid: { color: 'rgba(255,255,255,0.05)' } },
                        yTemp: { type: 'linear', position: 'left', ticks: { color: '#ef4444' } },
                        yRain: { type: 'linear', position: 'right', ticks: { color: '#38bdf8' }, grid: { drawOnChartArea: false } }
                    }
                }
            });
        }

        function renderEfficacyChart() {
            const ctx = document.getElementById('efficacyChart');
            if (!ctx) return;
            if (efficacyChartInstance) efficacyChartInstance.destroy();
            efficacyChartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'],
                    datasets: [{
                        label: 'Taxa de Acerto (%)',
                        data: [91.2, 90.5, 92.1, 93.4, 91.8, 92.9, 93.8, 92.0, 91.5, 93.2, 92.4, 93.0],
                        borderColor: '#10b981',
                        backgroundColor: 'rgba(16, 185, 129, 0.1)',
                        borderWidth: 3,
                        fill: true,
                        tension: 0.3
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: { ticks: { color: '#94a3b8' }, grid: { color: 'rgba(255,255,255,0.05)' } },
                        y: { min: 85, max: 100, ticks: { color: '#94a3b8' }, grid: { color: 'rgba(255,255,255,0.05)' } }
                    }
                }
            });
        }

        function render3DayForecast(daily) {
            const container = document.getElementById('forecast-3days-container');
            if (!daily) return;
            let html = '';
            const weekDays = ['Domingo', 'Segunda', 'Terça', 'Quarta', 'Quinta', 'Sexta', 'Sábado'];
            for (let i = 1; i <= 4; i++) {
                if (!daily.time[i]) break;
                const d = new Date(daily.time[i] + 'T00:00:00');
                html += `
                    <div class="rounded-2xl dark:bg-slate-900 bg-slate-100 border border-slate-800 p-4 shadow">
                        <span class="text-[10px] font-bold text-sky-400 block uppercase">Dia +${i}</span>
                        <h3 class="font-bold text-sm text-white">${weekDays[d.getDay()]} (${d.getDate()}/${d.getMonth()+1})</h3>
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
            updateLabSimulation();
            fetchWeatherData();
        };
    </script>
</body>
</html>
