<html lang="pt-PT">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Guia Med Connect</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f8fafc; }
        .hide-scrollbar::-webkit-scrollbar { display: none; }
        .hide-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        .touch-target { min-height: 44px; min-width: 44px; }
        
        #calendar-container {
            overflow-x: auto;
            cursor: grab;
            user-select: none;
            scroll-behavior: smooth;
        }
        #calendar-container:active { cursor: grabbing; }
        
        #calendar-ribbon {
            display: flex;
            gap: 0.75rem;
            padding: 0.5rem 1.5rem;
            width: max-content;
            align-items: center;
        }

        .calendar-item {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-width: 55px;
            padding: 1rem 0;
            border-radius: 1.25rem;
            transition: all 0.2s;
            border: 1px solid #7283fb;
            background: #2fdfa8;
        }
        .calendar-item.selected {
            background-color: #c4f1e3;
            color: white;
            border-color: #2563eb;
            box-shadow: 0 4px 12px rgba(37, 99, 235, 0.2);
            transform: scale(1.05);
        }
        .calendar-item.is-today {
            border: 2px solid #2563eb;
            min-width: 65px;
            padding: 1.2rem 0;
            transform: scale(1.1);
            position: relative;
        }
        .calendar-item.is-today::after {
            content: "HOJE";
            position: absolute;
            top: -8px;
            background: #2563eb;
            color: white;
            font-size: 7px;
            font-weight: 900;
            padding: 2px 5px;
            border-radius: 4px;
        }

        .autocomplete-suggestions {
            position: absolute;
            top: 100%;
            left: 0;
            right: 0;
            background: white;
            border: 1px solid #e2e8f0;
            border-radius: 0.75rem;
            margin-top: 0.5rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
            z-index: 100;
            max-height: 200px;
            overflow-y: auto;
        }
        .suggestion-item { padding: 0.75rem 1rem; cursor: pointer; border-bottom: 1px solid #f1f5f9; }
        .suggestion-item:hover { background-color: #eff6ff; color: #2563eb; }

        .int-btn.selected { background-color: #2563eb !important; color: white !important; border-color: #2563eb !important; }

        .switch {
            position: relative;
            display: inline-block;
            width: 40px;
            height: 20px;
        }
        .switch input { opacity: 0; width: 0; height: 0; }
        .slider {
            position: absolute;
            cursor: pointer;
            top: 0; left: 0; right: 0; bottom: 0;
            background-color: #cbd5e1;
            transition: .4s;
            border-radius: 20px;
        }
        .slider:before {
            position: absolute;
            content: "";
            height: 16px; width: 16px;
            left: 2px; bottom: 2px;
            background-color: white;
            transition: .4s;
            border-radius: 50%;
        }
        input:checked + .slider { background-color: #2563eb; }
        input:checked + .slider:before { transform: translateX(20px); }

        @media print {
            body { background: white; }
            nav, header, button, .switch, #modal-add, #interaction-warning { display: none !important; }
            .max-w-md { max-width: 100% !important; box-shadow: none !important; }
            #main-content { padding: 0 !important; }
            #view-history { display: block !important; }
            #history-content-consumption, #history-content-purchase { display: block !important; }
            .print-only { display: block !important; }
            .no-print { display: none !important; }
        }
        .print-only { display: none; }
    </style>
</head>
<body class="text-slate-800 antialiased selection:bg-blue-200">

    <div id="toast-msg" class="fixed top-5 left-1/2 -translate-x-1/2 bg-red-600 text-white px-6 py-3 rounded-2xl shadow-2xl font-bold hidden z-[1000] items-center gap-3">
        <span id="toast-icon">⚠️</span> <span id="toast-text">Erro</span>
    </div>

    <div class="max-w-md mx-auto bg-white min-h-screen shadow-2xl relative flex flex-col overflow-hidden pb-20">
        
        <header class="bg-blue-50 pt-8 pb-6 rounded-b-3xl shadow-sm z-10 no-print">
            <div class="px-6 flex justify-between items-center mb-4">
                <div>
                    <h1 class="text-2xl font-bold text-blue-900" id="current-month-display">...</h1>
                    <p class="text-xs text-blue-600 font-medium">GUIA MED controle de medicação <span id="admin-badge" class="hidden ml-2 bg-black text-white px-1 rounded uppercase">ADMIN</span></p>
                </div>
                <button onclick="requestAdminAccess()" class="h-10 w-10 bg-white rounded-full flex items-center justify-center shadow-sm border border-blue-100 touch-target">👤</button>
            </div>

            <div id="calendar-container" class="hide-scrollbar">
                <div id="calendar-ribbon"></div>
            </div>
        </header>

        <main class="flex-1 overflow-y-auto p-6" id="main-content">
            <div class="print-only mb-8 text-center">
                <h1 class="text-2xl font-bold text-slate-900">Relatório Guia Med Connect</h1>
                <p class="text-sm text-slate-500" id="print-date">Gerado em: --/--/----</p>
            </div>

            <section id="view-agenda" class="block no-print">
                <div id="stock-alerts-container" class="hidden mb-6">
                    <h3 class="text-[10px] font-black text-red-500 uppercase tracking-widest mb-2 flex items-center gap-1">⚠️ Reposição Urgente</h3>
                    <div id="stock-alerts-list" class="flex flex-col gap-2"></div>
                </div>

                <div class="mb-4">
                    <h2 class="text-xl font-semibold text-slate-800">Sua Agenda</h2>
                    <p class="text-sm text-slate-500" id="selected-date-label">Carregando...</p>
                </div>
                <div id="agenda-list" class="flex flex-col gap-4"></div>
            </section>

            <section id="view-inventory" class="hidden no-print">
                <div class="mb-4 flex justify-between items-start">
                    <div>
                        <h2 class="text-xl font-semibold text-slate-800">Inventário</h2>
                        <p class="text-sm text-slate-500">Controle de estoque.</p>
                    </div>
                    <button onclick="openAddModal()" class="touch-target bg-green-400 text-white rounded-full px-4 py-2 font-medium shadow-md">➕ Novo</button>
                </div>
                <div id="inventory-list" class="flex flex-col gap-3"></div>
            </section>

            <section id="view-history" class="hidden">
                <div class="mb-6 flex justify-between items-end no-print">
                    <div>
                        <h2 class="text-xl font-semibold text-slate-800">Histórico</h2>
                        <p class="text-sm text-slate-500">Relatórios de consumo e compras.</p>
                    </div>
                    <button onclick="exportToPDF()" class="bg-slate-800 text-white text-[10px] font-bold py-2 px-4 rounded-lg flex items-center gap-1 hover:bg-black transition-colors">
                        📄 Gerar PDF
                    </button>
                </div>

                <div class="flex gap-2 mb-6 bg-slate-100 p-1 rounded-xl no-print">
                    <button onclick="switchHistorySubTab('consumption')" id="sub-tab-consumption" class="flex-1 py-2 text-xs font-bold rounded-lg bg-white shadow-sm text-blue-600 transition-all">Consumo</button>
                    <button onclick="switchHistorySubTab('purchase')" id="sub-tab-purchase" class="flex-1 py-2 text-xs font-bold rounded-lg text-slate-500 transition-all">Compras</button>
                </div>

                <div id="history-content-consumption" class="block">
                    <div class="bg-blue-50 p-4 rounded-2xl mb-4 no-print">
                        <p class="text-[10px] font-black text-blue-600 uppercase mb-2">Resumo de Consumo</p>
                        <p class="text-xs text-slate-600">Registro detalhado dos medicamentos consumidos.</p>
                    </div>
                    <h3 class="print-only text-lg font-bold border-b pb-2 mb-4">Relatório de Consumo Detalhado</h3>
                    <div id="consumption-report-list" class="flex flex-col gap-3"></div>
                </div>

                <div id="history-content-purchase" class="hidden">
                    <div class="bg-orange-50 p-4 rounded-2xl mb-4 mt-6 no-print">
                        <p class="text-[10px] font-black text-orange-600 uppercase mb-2">Lista de Compras</p>
                        <p class="text-xs text-slate-600">Itens com estoque igual ou inferior a 5 unidades.</p>
                    </div>
                    <h3 class="print-only text-lg font-bold border-b pb-2 mb-4 mt-8">Lista de Reposição / Compras</h3>
                    <div id="purchase-report-list" class="flex flex-col gap-3"></div>
                </div>
            </section>
        </main>

        <nav class="absolute bottom-0 w-full bg-white border-t border-slate-100 px-6 py-3 flex items-center z-20 pb-safe no-print">
            <div class="w-1/3 flex justify-start">
                <button onclick="switchTab('inventory')" id="btn-inventory" class="tab-btn text-slate-400 flex flex-col items-center gap-1 touch-target p-2">
                    <span class="text-2xl">💊</span><span class="text-xs font-medium">Estoque</span>
                </button>
            </div>
            <div class="w-1/3 flex justify-center">
                <button onclick="goHome()" id="btn-agenda" class="tab-btn active text-blue-600 flex flex-col items-center gap-1 touch-target p-2">
                    <span class="text-2xl">📅</span><span class="text-xs font-medium">Agenda</span>
                </button>
            </div>
            <div class="w-1/3 flex justify-end">
                <button onclick="switchTab('history')" id="btn-history" class="tab-btn text-slate-400 flex flex-col items-center gap-1 touch-target p-2">
                    <span class="text-2xl">📊</span><span class="text-xs font-medium">Histórico</span>
                </button>
            </div>
        </nav>

        <!-- Modal -->
        <div id="modal-add" class="fixed inset-0 bg-slate-900/50 z-50 hidden flex flex-col justify-end">
            <div class="bg-white w-full max-w-md mx-auto rounded-t-3xl p-6 transform transition-transform translate-y-full overflow-y-auto max-h-[90vh]" id="modal-add-content">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-xl font-bold text-slate-800">Novo Agendamento</h2>
                    <button onclick="closeAddModal()" class="text-2xl text-slate-400">✖</button>
                </div>
                <form id="form-add-med" class="flex flex-col gap-4">
                    <div class="relative">
                        <label class="block text-[11px] font-bold text-slate-500 uppercase mb-1">Medicamento</label>
                        <input type="text" id="med-name" autocomplete="off" required oninput="handleNameInput()" class="w-full border border-slate-300 rounded-xl p-3 focus:ring-2 focus:ring-blue-500 outline-none" placeholder="Ex: Paracetamol">
                        <div id="autocomplete-list" class="autocomplete-suggestions hidden"></div>
                    </div>
                    <div class="grid grid-cols-2 gap-4">
                        <div class="relative">
                            <input type="text" id="med-dose" required oninput="handleDoseInput()" onfocus="handleDoseInput()" class="w-full border border-slate-300 rounded-xl p-3 focus:ring-2 focus:ring-blue-500 outline-none" placeholder="Dose (Ex: 500mg)">
                            <div id="dose-autocomplete-list" class="autocomplete-suggestions hidden"></div>
                        </div>
                        <select id="med-type" class="border border-slate-300 rounded-xl p-3 bg-white">
                            <option value="💊">Comprimido</option>
                            <option value="💧">Gotas</option>
                            <option value="💉">Injetável</option>
                        </select>
                    </div>

                    <div class="flex items-center justify-between p-4 bg-slate-50 rounded-xl border border-slate-100">
                        <div>
                            <p class="text-sm font-bold text-slate-700">Medicamento Contínuo</p>
                            <p class="text-[10px] text-slate-400">Repetir horários na agenda</p>
                        </div>
                        <label class="switch">
                            <input type="checkbox" id="med-continuous">
                            <span class="slider"></span>
                        </label>
                    </div>

                    <div class="bg-blue-50 p-4 rounded-xl">
                        <label class="block text-[11px] font-bold text-blue-900 uppercase mb-2">Início, Hora & Intervalo</label>
                        <div class="grid grid-cols-2 gap-2 mb-3">
                            <input type="date" id="med-start-date" class="w-full border border-slate-300 rounded-xl p-3 text-xs">
                            <input type="time" id="med-time" required class="w-full border border-slate-300 rounded-xl p-3 text-xs">
                        </div>
                        <div class="grid grid-cols-5 gap-2">
                            <button type="button" onclick="setIntervalVal(4)" class="int-btn bg-white py-2 rounded-lg text-xs font-bold border border-slate-200 transition">4h</button>
                            <button type="button" onclick="setIntervalVal(6)" class="int-btn bg-white py-2 rounded-lg text-xs font-bold border border-slate-200 transition">6h</button>
                            <button type="button" onclick="setIntervalVal(8)" class="int-btn bg-white py-2 rounded-lg text-xs font-bold border border-slate-200 transition">8h</button>
                            <button type="button" onclick="setIntervalVal(12)" class="int-btn bg-white py-2 rounded-lg text-xs font-bold border border-slate-200 transition">12h</button>
                            <button type="button" onclick="setIntervalVal(24)" class="int-btn bg-white py-2 rounded-lg text-xs font-bold border border-slate-200 transition">24h</button>
                        </div>
                        <input type="hidden" id="med-interval" value="0">
                    </div>
                    
                    <div id="stock-field-container">
                        <label class="block text-[11px] font-bold text-slate-500 uppercase mb-1">Quantidade em Estoque</label>
                        <input type="number" id="med-stock" required min="0" class="w-full border border-slate-300 rounded-xl p-3" placeholder="Adicione quantidade">
                    </div>

                    <button type="submit" class="w-full bg-blue-600 text-white rounded-xl py-4 font-bold text-lg shadow-lg">Salvar</button>
                </form>
            </div>
        </div>
    </div>

    <script>
        const monthNames = ['Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho', 'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro'];
        const dayNames = ['Dom', 'Seg', 'Ter', 'Qua', 'Qui', 'Sex', 'Sáb'];
        
        const DOSAGE_CATALOG = { 
            'dipirona': ['500mg', '1g', '500mg/ml'],
            'paracetamol': ['500mg', '750mg', '1g', '200mg/ml'],
            'ibuprofeno': ['200mg', '400mg', '600mg'],
            'abafil': ['100mg', '200mg'],
            'abatenolo': ['25mg', '50mg', '100mg'],
            'acido acetilsalicilico': ['100mg', '500mg'],
            'alivium': ['100mg/ml', '400mg', '600mg'],
            'alprazolam': ['0.25mg', '0.5mg', '1mg', '2mg'],
            'amoxicilina': ['250mg', '500mg', '875mg'],
            'amoxicilina + clavulanato': ['500mg+125mg', '875mg+125mg'],
            'anlodipino': ['5mg', '10mg'],
            'atenolol': ['25mg', '50mg', '100mg'],
            'atorvastatina': ['10mg', '20mg', '40mg', '80mg'],
            'azitromicina': ['500mg', '600mg', '900mg'],
            'benegrip': ['Drágeas'],
            'buscopan': ['10mg', 'compositum', 'gotas'],
            'captopril': ['12.5mg', '25mg', '50mg'],
            'cefalexina': ['250mg', '500mg'],
            'cetoprofeno': ['50mg', '100mg', 'gotas'],
            'citalopram': ['20mg', '40mg'],
            'clonazepam': ['0.5mg', '2mg', '2.5mg/ml'],
            'cloridrato de sertralina': ['25mg', '50mg', '100mg'],
            'desloratadina': ['5mg', '0.5mg/ml'],
            'dexametasona': ['0.5mg', '4mg', 'elixir'],
            'diazepam': ['5mg', '10mg'],
            'diclofenaco': ['50mg', '75mg', '100mg'],
            'dorflex': ['Comprimidos'],
            'enalapril': ['5mg', '10mg', '20mg'],
            'escitalopram': ['10mg', '15mg', '20mg'],
            'espiral': ['25mg', '50mg', '100mg'],
            'fluoxetina': ['20mg'],
            'furosemida': ['40mg'],
            'glibenclamida': ['5mg'],
            'glifage': ['500mg', '850mg', '1g'],
            'hidroclorotiazida': ['25mg', '50mg'],
            'insulina': ['NPH', 'Regular', 'Lispro'],
            'isordil': ['5mg', '10mg'],
            'levotiroxina': ['25mcg', '50mcg', '75mcg', '100mcg', '112mcg', '125mcg'],
            'loratadina': ['10mg', '1mg/ml'],
            'losartana': ['25mg', '50mg', '100mg'],
            'metformina': ['500mg', '850mg', '1g'],
            'metoprolol': ['25mg', '50mg', '100mg'],
            'nimesulida': ['100mg', 'gotas'],
            'novalgina': ['500mg', '1g'],
            'omeprazol': ['10mg', '20mg', '40mg'],
            'pantoprazol': ['20mg', '40mg'],
            'paracetamol + codeina': ['500mg+30mg'],
            'prednisona': ['5mg', '20mg', '40mg'],
            'propranolol': ['10mg', '40mg', '80mg'],
            'quetiapina': ['25mg', '100mg', '200mg'],
            'rivotril': ['0.5mg', '2mg', 'gotas'],
            'rosuvastatina': ['5mg', '10mg', '20mg'],
            'selozok': ['25mg', '50mg', '100mg'],
            'sinvastatina': ['10mg', '20mg', '40mg'],
            'spironolactona': ['25mg', '50mg', '100mg'],
            'tadalafila': ['5mg', '20mg'],
            'tramadol': ['50mg', '100mg', 'gotas'],
            'venlafaxina': ['37.5mg', '75mg', '150mg'],
            'victoza': ['6mg/ml'],
            'vitamina d': ['1000ui', '2000ui', '7000ui', '50000ui'],
            'warfarina': ['5mg'],
            'xarelto': ['10mg', '15mg', '20mg'],
            'zolpidem': ['5mg', '10mg']
        };

        let state = {
            meds: JSON.parse(localStorage.getItem('medsafe_v15')) || [],
            selectedDate: new Date().toISOString().split('T')[0],
            isAdmin: false,
            activeSubTab: 'consumption'
        };

        const container = document.getElementById('calendar-container');
        let isDown = false, startX, scrollLeft;

        container.addEventListener('mousedown', (e) => { isDown = true; startX = e.pageX - container.offsetLeft; scrollLeft = container.scrollLeft; });
        container.addEventListener('mouseleave', () => { isDown = false; });
        container.addEventListener('mouseup', () => { isDown = false; });
        container.addEventListener('mousemove', (e) => { if (!isDown) return; e.preventDefault(); const x = e.pageX - container.offsetLeft; const walk = (x - startX) * 2; container.scrollLeft = scrollLeft - walk; });

        function showToast(text, icon = '⚠️') {
            const toast = document.getElementById('toast-msg');
            document.getElementById('toast-text').innerText = text;
            document.getElementById('toast-icon').innerText = icon;
            toast.classList.remove('hidden'); toast.classList.add('flex');
            setTimeout(() => { toast.classList.add('hidden'); toast.classList.remove('flex'); }, 3000);
        }

        window.handleNameInput = function() {
            const input = document.getElementById('med-name');
            const list = document.getElementById('autocomplete-list');
            const val = input.value.toLowerCase();
            list.innerHTML = '';
            
            if(!val) { list.classList.add('hidden'); return; }
            
            const matches = Object.keys(DOSAGE_CATALOG).filter(k => k.includes(val));
            if(matches.length > 0) {
                list.classList.remove('hidden');
                matches.forEach(m => {
                    const div = document.createElement('div');
                    div.className = "suggestion-item";
                    div.innerText = m.charAt(0).toUpperCase() + m.slice(1);
                    div.onclick = () => {
                        input.value = div.innerText;
                        list.classList.add('hidden');
                        window.handleDoseInput();
                    };
                    list.appendChild(div);
                });
            } else { list.classList.add('hidden'); }
        };

        window.handleDoseInput = function() {
            const medName = document.getElementById('med-name').value.toLowerCase();
            const input = document.getElementById('med-dose');
            const list = document.getElementById('dose-autocomplete-list');
            list.innerHTML = '';

            if(DOSAGE_CATALOG[medName]) {
                list.classList.remove('hidden');
                DOSAGE_CATALOG[medName].forEach(d => {
                    const div = document.createElement('div');
                    div.className = "suggestion-item";
                    div.innerText = d;
                    div.onclick = () => {
                        input.value = d;
                        list.classList.add('hidden');
                    };
                    list.appendChild(div);
                });
            } else { list.classList.add('hidden'); }
        };

        window.setIntervalVal = function(val) {
            document.getElementById('med-interval').value = val;
            document.querySelectorAll('.int-btn').forEach(btn => {
                if(btn.innerText === val + 'h') btn.classList.add('selected');
                else btn.classList.remove('selected');
            });
        };

        document.addEventListener('click', (e) => {
            if(!e.target.closest('.relative')) {
                document.getElementById('autocomplete-list').classList.add('hidden');
                document.getElementById('dose-autocomplete-list').classList.add('hidden');
            }
        });

        function requestAdminAccess() {
            if(state.isAdmin) {
                state.isAdmin = false;
                document.getElementById('admin-badge').classList.add('hidden');
                showToast('Logout realizado', '👋');
                renderInventory();
                return;
            }
            const user = prompt("Usuário:");
            if (user === null) return;
            const pass = prompt("Senha:");
            if(user === "admin" && pass === "guiamed2025") {
                state.isAdmin = true;
                document.getElementById('admin-badge').classList.remove('hidden');
                showToast('Acesso administrador liberado', '🔐');
                renderInventory();
            } else {
                showToast('Dados incorretos');
            }
        }

        function generateCalendar() {
            const ribbon = document.getElementById('calendar-ribbon');
            ribbon.innerHTML = '';
            const today = new Date();
            const todayStr = today.toISOString().split('T')[0];
            for (let i = -7; i < 23; i++) {
                const date = new Date();
                date.setDate(today.getDate() + i);
                const dateStr = date.toISOString().split('T')[0];
                const isSelected = state.selectedDate === dateStr;
                const isToday = dateStr === todayStr;
                const item = document.createElement('div');
                item.className = `calendar-item ${isSelected ? 'selected' : ''} ${isToday ? 'is-today' : ''}`;
                item.onclick = () => { state.selectedDate = dateStr; generateCalendar(); renderAgenda(); };
                item.innerHTML = `<span class="text-[10px] font-bold uppercase opacity-60">${dayNames[date.getDay()]}</span><span class="text-lg font-bold">${date.getDate()}</span>`;
                ribbon.appendChild(item);
                if (isSelected) {
                    document.getElementById('current-month-display').innerText = monthNames[date.getMonth()];
                    document.getElementById('selected-date-label').innerText = date.toLocaleDateString('pt-PT', { weekday: 'long', day: 'numeric', month: 'long' });
                }
            }
        }

        function scrollToToday() {
            const todayEl = document.querySelector('.is-today');
            if(todayEl) {
                const scrollPos = todayEl.offsetLeft - (container.offsetWidth / 2) + (todayEl.offsetWidth / 2);
                container.scrollLeft = scrollPos;
            }
        }

        function goHome() {
            const todayStr = new Date().toISOString().split('T')[0];
            state.selectedDate = todayStr;
            switchTab('agenda');
            generateCalendar();
            renderAgenda();
            scrollToToday();
        }

        function renderAgenda() {
            const list = document.getElementById('agenda-list');
            list.innerHTML = '';
            const daily = state.meds.filter(m => {
                if(m.continuous) return state.selectedDate >= m.date;
                return m.date === state.selectedDate;
            }).sort((a,b) => a.time.localeCompare(b.time));
            
            if (daily.length === 0) { 
                list.innerHTML = `<div class="text-center py-10 opacity-30"><p>Sem medicação para este dia.</p></div>`; 
                return; 
            }

            daily.forEach(med => {
                const isTakenToday = med.continuous 
                    ? (med.takenDates && med.takenDates.some(d => d.date === state.selectedDate)) 
                    : med.taken;
                
                const outOfStock = med.stock <= 0;
                const card = document.createElement('div');
                card.className = `p-4 rounded-2xl flex items-center justify-between border-2 transition-all ${isTakenToday ? 'bg-emerald-50 border-emerald-100 opacity-75' : 'bg-white border-slate-100 shadow-sm'}`;
                card.innerHTML = `
                    <div class="flex items-center gap-3">
                        <div class="h-10 w-10 bg-blue-50 rounded-lg flex items-center justify-center text-xl">${med.type}</div>
                        <div>
                            <div class="flex items-center gap-1">
                                <h4 class="font-bold text-sm ${isTakenToday ? 'line-through text-emerald-800' : 'text-slate-800'}">${med.name}</h4>
                            </div>
                            <p class="text-[10px] font-bold text-blue-500 uppercase">${med.time} • ${med.dose}</p>
                        </div>
                    </div>
                    <button onclick="${outOfStock && !isTakenToday ? `showToast('Estoque esgotado para ${med.name}')` : `toggleTaken(${med.id})`}" 
                            class="h-8 w-8 rounded-full border-2 flex items-center justify-center transition-all 
                            ${isTakenToday ? 'bg-emerald-500 border-emerald-500 text-white' : (outOfStock ? 'border-red-200 bg-red-50 text-red-300' : 'border-slate-200 text-transparent')}">✓</button>
                `;
                list.appendChild(card);
            });
            checkStock();
        }

        function renderInventory() {
            const list = document.getElementById('inventory-list');
            list.innerHTML = '';
            state.meds.forEach(med => {
                const card = document.createElement('div');
                card.className = "bg-white p-4 rounded-2xl border border-slate-100 shadow-sm flex justify-between items-center";
                card.innerHTML = `
                    <div class="flex items-center gap-3">
                        <div class="text-2xl">${med.type}</div>
                        <div>
                            <p class="font-bold text-sm text-slate-800">${med.name}</p>
                            <p class="text-[10px] text-slate-500 uppercase font-bold">${med.dose}</p>
                        </div>
                    </div>
                    <div class="flex items-center gap-4">
                        <div class="text-right">
                            <p class="text-[10px] font-black text-slate-400 uppercase tracking-tighter">Estoque</p>
                            <p class="font-bold text-lg ${med.stock <= 5 ? 'text-red-500' : 'text-blue-600'}">${med.stock}</p>
                        </div>
                        ${state.isAdmin ? `
                        <button onclick="deleteMed(${med.id})" class="h-10 w-10 bg-red-50 text-red-500 rounded-full flex items-center justify-center hover:bg-red-500 hover:text-white transition-all">
                            🗑️
                        </button>` : ''}
                    </div>
                `;
                list.appendChild(card);
            });
        }

        function deleteMed(id) {
            if(!state.isAdmin) return;
            state.meds = state.meds.filter(m => m.id !== id);
            save();
            showToast('Medicamento removido', '🗑️');
        }

        function toggleTaken(id) {
            const today = new Date().toISOString().split('T')[0];
            if (state.selectedDate > today) { showToast('Não é possível marcar horários futuros'); return; }
            
            const medIndex = state.meds.findIndex(m => m.id === id);
            if (medIndex === -1) return;
            
            const med = state.meds[medIndex];
            const timestamp = new Date().toLocaleString('pt-PT');

            if (med.continuous) {
                if (!med.takenDates) med.takenDates = [];
                const idx = med.takenDates.findIndex(d => d.date === state.selectedDate);
                
                if (idx === -1) {
                    med.takenDates.push({ date: state.selectedDate, log: timestamp });
                    med.stock--;
                } else {
                    med.takenDates.splice(idx, 1);
                    med.stock++;
                }
            } else {
                if (!med.taken) {
                    med.taken = true; 
                    med.takenAt = timestamp;
                    med.stock--;
                } else {
                    med.taken = false; 
                    med.takenAt = null;
                    med.stock++;
                }
            }
            save();
        }

        function checkStock() {
            const alertsList = document.getElementById('stock-alerts-list');
            const alertsCont = document.getElementById('stock-alerts-container');
            alertsList.innerHTML = '';
            const lowStock = state.meds.filter(m => m.stock <= 5);
            if(lowStock.length > 0) {
                alertsCont.classList.remove('hidden');
                lowStock.forEach(m => {
                    const div = document.createElement('div');
                    div.className = "bg-red-50 border-l-4 border-red-500 p-2 text-[10px] flex justify-between";
                    div.innerHTML = `<span><strong>${m.name}</strong> (${m.dose})</span> <span class="font-bold">${m.stock} restando</span>`;
                    alertsList.appendChild(div);
                });
            } else { alertsCont.classList.add('hidden'); }
        }

        function save() { 
            localStorage.setItem('medsafe_v15', JSON.stringify(state.meds)); 
            renderAgenda(); 
            renderInventory(); 
            renderHistory();
        }

        function switchTab(tab) {
            document.querySelectorAll('section').forEach(s => s.classList.add('hidden'));
            document.getElementById(`view-${tab}`).classList.remove('hidden');
            document.querySelectorAll('.tab-btn').forEach(b => { 
                b.classList.remove('text-blue-600', 'active'); 
                b.classList.add('text-slate-400'); 
            });
            const targetBtn = document.getElementById(`btn-${tab}`);
            if (targetBtn) { targetBtn.classList.add('text-blue-600', 'active'); }
            
            if(tab === 'inventory') renderInventory();
            if(tab === 'history') renderHistory();
            if(tab === 'agenda') { generateCalendar(); renderAgenda(); }
        }

        function renderHistory() {
            const consumptionList = document.getElementById('consumption-report-list');
            const purchaseList = document.getElementById('purchase-report-list');
            consumptionList.innerHTML = ''; purchaseList.innerHTML = '';
            
            const logs = [];
            state.meds.forEach(m => {
                if (m.continuous && m.takenDates) {
                    m.takenDates.forEach(td => logs.push({ name: m.name, dose: m.dose, log: td.log }));
                } else if (!m.continuous && m.taken && m.takenAt) {
                    logs.push({ name: m.name, dose: m.dose, log: m.takenAt });
                }
            });

            if (logs.length === 0) {
                consumptionList.innerHTML = '<p class="text-center text-slate-400 text-xs py-4">Nenhum registro encontrado.</p>';
            } else {
                logs.sort((a,b) => b.log.localeCompare(a.log)).forEach(l => {
                    const item = document.createElement('div');
                    item.className = "bg-white p-3 rounded-xl border border-slate-100 flex justify-between items-center";
                    item.innerHTML = `<div><p class="font-bold text-xs">${l.name}</p><p class="text-[9px] text-slate-400">${l.dose}</p></div><p class="text-[9px] font-bold text-blue-600">${l.log}</p>`;
                    consumptionList.appendChild(item);
                });
            }

            const lowStock = state.meds.filter(m => m.stock <= 5);
            if (lowStock.length === 0) {
                purchaseList.innerHTML = '<p class="text-center text-slate-400 text-xs py-4">Estoque em dia!</p>';
            } else {
                lowStock.forEach(m => {
                    const item = document.createElement('div');
                    item.className = "bg-white p-3 rounded-xl border border-slate-100 flex justify-between items-center";
                    item.innerHTML = `<div><p class="font-bold text-xs">${m.name}</p><p class="text-[9px] text-slate-400">${m.dose}</p></div><div class="text-right"><p class="text-[9px] text-orange-600 font-bold">Estoque: ${m.stock}</p></div>`;
                    purchaseList.appendChild(item);
                });
            }
        }

        function switchHistorySubTab(tab) {
            state.activeSubTab = tab;
            document.getElementById('history-content-consumption').classList.add('hidden');
            document.getElementById('history-content-purchase').classList.add('hidden');
            document.getElementById('sub-tab-consumption').className = "flex-1 py-2 text-xs font-bold rounded-lg text-slate-500 transition-all";
            document.getElementById('sub-tab-purchase').className = "flex-1 py-2 text-xs font-bold rounded-lg text-slate-500 transition-all";
            
            document.getElementById(`history-content-${tab}`).classList.remove('hidden');
            document.getElementById(`sub-tab-${tab}`).className = "flex-1 py-2 text-xs font-bold rounded-lg bg-white shadow-sm text-blue-600 transition-all";
        }

        function exportToPDF() {
            document.getElementById('print-date').innerText = `Gerado em: ${new Date().toLocaleString('pt-PT')}`;
            window.print();
        }

        function openAddModal() {
            const modal = document.getElementById('modal-add');
            modal.classList.remove('hidden');
            setTimeout(() => document.getElementById('modal-add-content').classList.remove('translate-y-full'), 10);
            document.getElementById('med-start-date').value = state.selectedDate;
        }

        function closeAddModal() {
            document.getElementById('modal-add-content').classList.add('translate-y-full');
            setTimeout(() => document.getElementById('modal-add').classList.add('hidden'), 300);
        }

        document.getElementById('form-add-med').onsubmit = (e) => {
            e.preventDefault();
            const newMed = {
                id: Date.now(),
                name: document.getElementById('med-name').value,
                dose: document.getElementById('med-dose').value,
                type: document.getElementById('med-type').value,
                date: document.getElementById('med-start-date').value,
                time: document.getElementById('med-time').value,
                continuous: document.getElementById('med-continuous').checked,
                stock: parseInt(document.getElementById('med-stock').value) || 0,
                interval: parseInt(document.getElementById('med-interval').value) || 0,
                taken: false
            };
            state.meds.push(newMed);
            save();
            closeAddModal();
            e.target.reset();
            document.querySelectorAll('.int-btn').forEach(b => b.classList.remove('selected'));
        };

        window.onload = () => {
            generateCalendar();
            scrollToToday();
            renderAgenda();
        };
    </script>
</body>
</html>
