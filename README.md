# OleksiiPick.github.io

<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Конструктор Ледового Бура v5.1</title>
    <style>
        /* --- ОБЩИЕ СТИЛИ --- */
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #334155; color: #f1f5f9; display: flex; justify-content: center; align-items: flex-start; padding: 20px; min-height: 100vh; box-sizing: border-box; user-select: none; }
        #app-wrapper { display: flex; flex-direction: column; align-items: center; gap: 20px; width: 100%; }
        h1 { color: #e2e8f0; text-shadow: 2px 2px 4px #000; margin-bottom: 5px; }
        
        /* --- КОНТЕЙНЕР БУРА --- */
        #drill-wrapper { width: 1000px; max-width: 95vw; height: 500px; position: relative; background-color: #1e293b; border-radius: 15px; box-shadow: 0 10px 20px rgba(0,0,0,0.5); overflow: hidden; transition: height 0.3s; }
        .layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; }
        #background-layer { background-size: cover; background-position: center; background-color: #0f172a; }
        #chassis-layer { background-size: 100% 100%; background-repeat: no-repeat; opacity: 0.9; }
        #slots-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: auto; }

        /* --- СТИЛИ СЛОТА --- */
        .slot { position: absolute; background-size: cover; background-position: center; border: 2px solid rgba(100, 116, 139, 0.5); border-radius: 4px; box-sizing: border-box; display: flex; justify-content: center; align-items: center; background-color: rgba(30, 41, 59, 0.5); transition: all 0.2s; }
        .mode-play .slot { cursor: pointer; }
        .mode-play .slot:hover { border-color: #f1f5f9; box-shadow: 0 0 10px rgba(148, 163, 184, 0.5); }
        .mode-edit .slot { cursor: grab; border: 2px dashed #f59e0b; background-color: rgba(245, 158, 11, 0.1); }
        .mode-edit .slot:active { cursor: grabbing; }
        
        .resize-handle { display: none; position: absolute; bottom: 0; right: 0; width: 15px; height: 15px; background-color: #f59e0b; cursor: se-resize; border-top-left-radius: 4px; }
        .mode-edit .resize-handle { display: block; }
        .delete-slot-btn { display: none; position: absolute; top: -10px; right: -10px; width: 20px; height: 20px; background-color: #ef4444; color: white; border-radius: 50%; text-align: center; line-height: 20px; font-size: 12px; cursor: pointer; z-index: 10; box-shadow: 0 2px 4px rgba(0,0,0,0.5); }
        .mode-edit .delete-slot-btn { display: block; }

        .slot.is-offline::after { content: '!'; font-weight: bold; font-size: 2em; color: rgba(255,255,255,0.8); position: absolute; top: 0; left: 0; width: 100%; height: 100%; background-color: rgba(220, 38, 38, 0.5); display: flex; justify-content: center; align-items: center; pointer-events: none; }

        .slot-ui-icon { position: absolute; top: 2px; left: 2px; width: 24px; height: 24px; background-color: rgba(0,0,0,0.7); color: white; border-radius: 50%; display: flex; justify-content: center; align-items: center; font-size: 14px; cursor: pointer; z-index: 5; border: 1px solid rgba(255,255,255,0.3); }
        
        .slot-bonus { position: absolute; bottom: 2px; right: 2px; background-color: rgba(16, 185, 129, 0.9); color: white; padding: 1px 4px; border-radius: 3px; font-size: 0.8em; pointer-events: none; box-shadow: 0 2px 4px rgba(0,0,0,0.5); }
        .slot-bonus.buffed { background-color: #d97706; }
        .slot-bonus.offline-bonus { background-color: #475569; color: #94a3b8; text-decoration: line-through; }
        
        .boost-icon { position: absolute; bottom: 2px; left: 2px; font-size: 12px; z-index: 5; text-shadow: 0 0 3px black; cursor: help; }

        /* --- UI PANEL --- */
        #ui-panel { background-color: #1e293b; padding: 15px 25px; border-radius: 10px; box-shadow: 0 5px 15px rgba(0,0,0,0.4); display: flex; flex-wrap: wrap; gap: 15px; align-items: center; width: 1000px; max-width: 95vw; box-sizing: border-box; }
        .stats-group { display: flex; gap: 15px; border-right: 1px solid #475569; padding-right: 15px; }
        .controls-group { display: flex; gap: 10px; flex-wrap: wrap; }
        .mode-toggle { display: flex; align-items: center; gap: 10px; background-color: #0f172a; padding: 5px 10px; border-radius: 5px; border: 1px solid #475569; }
        
        button { background-color: #475569; color: #e2e8f0; border: none; padding: 8px 12px; border-radius: 5px; cursor: pointer; transition: 0.2s; }
        button:hover { background-color: #64748b; }
        button.active { background-color: #f59e0b; color: #0f172a; font-weight: bold; }
        button.danger { background-color: #991b1b; }
        button.success { background-color: #059669; }
        
        /* --- MODALS --- */
        .hidden { display: none !important; }
        .modal { position: fixed; inset: 0; background: rgba(0,0,0,0.8); z-index: 100; display: flex; justify-content: center; align-items: center; backdrop-filter: blur(3px); }
        .modal-content { background: #1e293b; padding: 25px; border-radius: 10px; width: 950px; max-width: 95%; max-height: 85vh; overflow-y: auto; display: flex; flex-direction: column; }
        
        /* Module Library UI */
        #module-list-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); gap: 10px; }
        .module-card { background: #334155; padding: 10px; border-radius: 5px; cursor: pointer; border: 1px solid transparent; position: relative; }
        .module-card:hover { border-color: #f1f5f9; background: #475569; }
        .module-card img { width: 100%; height: 80px; object-fit: cover; background: #000; margin-bottom: 5px; }
        
        /* Two Column Layout (Crew & Module Editors) */
        .split-view { display: flex; gap: 20px; height: 500px; }
        .list-col { flex: 1; overflow-y: auto; border-right: 1px solid #475569; padding-right: 10px; }
        .edit-col { flex: 1.5; padding-left: 10px; overflow-y: auto; display: flex; flex-direction: column; gap: 10px; }

        /* Form Elements */
        .form-group { display: flex; flex-direction: column; gap: 5px; margin-bottom: 10px; }
        .form-group label { font-size: 0.9em; color: #94a3b8; }
        input, select, textarea { background: #0f172a; border: 1px solid #475569; color: white; padding: 8px; border-radius: 5px; font-family: inherit; }
        textarea { resize: vertical; min-height: 80px; }
        
        /* Skills List in Module Editor */
        .mod-skill-row { display: flex; gap: 5px; align-items: center; margin-bottom: 5px; background: #0f172a; padding: 5px; border-radius: 4px; }
        
        .crew-item { padding: 10px; background: #334155; border-radius: 5px; margin-bottom: 8px; cursor: pointer; display: flex; justify-content: space-between; align-items: center; }
        .crew-item.selected { border-color: #f59e0b; background: #475569; border: 1px solid #f59e0b; }

        #context-menu { position: absolute; background: #1e293b; border: 1px solid #64748b; padding: 5px; z-index: 200; border-radius: 5px; min-width: 150px; }
        #context-menu div { padding: 8px 12px; cursor: pointer; display: flex; align-items: center; gap: 8px; }
        #context-menu div:hover { background: #334155; }
        
        #toast { position: fixed; bottom: 30px; left: 50%; transform: translateX(-50%); background-color: #059669; color: white; padding: 10px 20px; border-radius: 20px; box-shadow: 0 5px 15px rgba(0,0,0,0.5); opacity: 0; pointer-events: none; transition: opacity 0.3s; z-index: 300; }
        #toast.show { opacity: 1; }
        
        #confirm-modal { z-index: 400; }
        #confirm-box { background: #1e293b; padding: 20px; border-radius: 10px; text-align: center; border: 1px solid #475569; box-shadow: 0 10px 30px rgba(0,0,0,0.8); }

        /* Details Modal Specifics */
        .details-header { display: flex; gap: 20px; margin-bottom: 20px; border-bottom: 1px solid #475569; padding-bottom: 20px; }
        .details-img { width: 150px; height: 150px; object-fit: cover; border-radius: 8px; border: 2px solid #475569; background: #000; }
        .details-info { flex: 1; }
        .details-desc { font-style: italic; color: #cbd5e1; margin: 10px 0; background: #334155; padding: 10px; border-radius: 5px; }
        .calc-box { background: #0f172a; padding: 15px; border-radius: 8px; margin-top: 15px; border: 1px solid #334155; font-family: monospace; font-size: 0.95em; }
        .calc-row { display: flex; justify-content: space-between; margin-bottom: 5px; border-bottom: 1px dashed #334155; padding-bottom: 2px; }
        .calc-row:last-child { border-bottom: none; }
        .calc-row strong { color: #f59e0b; }
        .calc-note { color: #94a3b8; font-size: 0.8em; display: block; margin-top: 2px; }

    </style>
</head>
<body class="mode-play">

    <div id="app-wrapper">
        <h1>Конструктор Бура v5.1</h1>
        
        <div id="ui-panel">
            <div class="stats-group">
                <span title="Лимит экипажа">👥 <span id="crew-val">0</span>/<span id="crew-max">0</span></span>
                <span title="Потребление / Выработка">⚡ <span id="energy-val">0</span>/<span id="energy-max">0</span></span>
            </div>
            
            <div class="mode-toggle">
                <span>Режим:</span>
                <button id="mode-play-btn" class="active">Игра</button>
                <button id="mode-edit-btn">Каркас</button>
            </div>

            <div class="controls-group">
                <button id="add-slot-btn" class="hidden">➕ Слот</button>
                <button id="bg-settings-btn">🖼️ Фон</button>
                <button id="modules-btn">⚙️ Модули</button>
                <button id="crew-btn">👤 Экипаж</button>
                <button id="save-btn">💾 Сохр.</button>
                <button id="load-btn">📂 Загр.</button>
            </div>
        </div>
        
        <div id="background-controls" class="hidden" style="margin-top: 10px; display: flex; gap: 10px; width: 100%; justify-content: center;">
            <input type="text" id="bg-url-input" placeholder="URL или ./images/bg.jpg" style="width: 300px;">
            <input type="file" id="bg-file-input" accept="image/*">
            <button id="apply-bg-btn">Применить</button>
        </div>

        <div id="drill-wrapper">
            <div id="background-layer" class="layer"></div>
            <div id="chassis-layer" class="layer"></div>
            <div id="slots-layer"></div>
        </div>
    </div>

    <!-- Модалка Выбора Модуля -->
    <div id="module-select-modal" class="modal hidden">
        <div class="modal-content" style="height: auto; max-height: 80vh;">
            <h2>Установить модуль</h2>
            <div id="module-list-grid"></div>
            <button onclick="closeModal('module-select-modal')" style="margin-top:15px">Закрыть</button>
        </div>
    </div>

    <!-- Модалка ПОДРОБНОСТЕЙ (New) -->
    <div id="module-details-modal" class="modal hidden">
        <div class="modal-content">
            <div id="details-content">
                <!-- Заполняется JS -->
            </div>
            <div style="display:flex; gap:10px; margin-top:20px;">
                <button onclick="changeModuleFromDetails()" style="flex:1">🔄 Заменить модуль</button>
                <button onclick="removeModuleFromDetails()" class="danger">🗑️ Снять модуль</button>
                <button onclick="closeModal('module-details-modal')" style="flex:1">Закрыть</button>
            </div>
        </div>
    </div>

    <!-- Модалка Редактора Модулей -->
    <div id="module-editor-modal" class="modal hidden">
        <div class="modal-content">
            <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:15px;">
                <h2 style="margin:0">Редактор Модулей</h2>
                <button onclick="createNewModule()" class="success">➕ Новый чертёж</button>
            </div>
            <div class="split-view">
                <!-- Список модулей -->
                <div class="list-col" id="mod-edit-list"></div>
                
                <!-- Форма редактирования -->
                <div class="edit-col">
                    <div id="mod-edit-form" class="hidden">
                        <input type="hidden" id="me-id">
                        <div class="form-group">
                            <label>Название</label>
                            <input type="text" id="me-name" onchange="updateCurrentModule()">
                        </div>
                        <div class="form-group">
                            <label>Описание</label>
                            <textarea id="me-desc" onchange="updateCurrentModule()" placeholder="Краткое художественное или техническое описание..."></textarea>
                        </div>
                        <div class="form-group">
                            <label>Картинка (URL)</label>
                            <input type="text" id="me-img" onchange="updateCurrentModule()" placeholder="https://...">
                        </div>
                        <div style="display:flex; gap:10px;">
                            <div class="form-group" style="flex:1">
                                <label>Цена (Энергия)</label>
                                <input type="number" id="me-cost" value="0" onchange="updateCurrentModule()">
                            </div>
                        </div>

                        <hr style="width:100%; border-color:#475569">
                        
                        <!-- Настройка Ресурсов -->
                        <h4>Производство Ресурсов</h4>
                        <div style="display:flex; gap:10px;">
                            <div class="form-group" style="flex:2">
                                <label>Тип ресурса</label>
                                <select id="me-res-type" onchange="updateCurrentModule()">
                                    <option value="none">Нет</option>
                                    <option value="energy">Энергия (Лимит)</option>
                                    <option value="crew">Экипаж (Койки)</option>
                                </select>
                            </div>
                            <div class="form-group" style="flex:1">
                                <label>Значение</label>
                                <input type="number" id="me-res-val" value="0" onchange="updateCurrentModule()">
                            </div>
                        </div>
                        <div class="form-group">
                            <label style="display:flex; align-items:center; gap:10px; background:#0f172a; padding:10px; border-radius:5px;">
                                <input type="checkbox" id="me-res-boost" onchange="updateCurrentModule()" style="width:auto;">
                                <span>Бонус экипажа увеличивает выработку?</span>
                            </label>
                            <small style="color:#94a3b8">Если включено, бонус навыка (целое число) добавляется к ресурсу.</small>
                        </div>

                        <hr style="width:100%; border-color:#475569">

                        <!-- Настройка Навыков -->
                        <h4>Требуемые Навыки</h4>
                        <div id="me-skills-list"></div>
                        <button onclick="addSkillToModule()" style="margin-top:5px; font-size:0.9em;">+ Добавить требование</button>

                        <div style="margin-top:auto; padding-top:20px; display:flex; justify-content:space-between;">
                            <button onclick="deleteCurrentModule()" class="danger">Удалить модуль</button>
                        </div>
                    </div>
                    <div id="mod-edit-placeholder" style="color:#94a3b8; text-align:center; margin-top:50px;">
                        Выберите модуль слева или создайте новый.
                    </div>
                </div>
            </div>
            <button onclick="closeModal('module-editor-modal')" style="margin-top:15px; width: 100%;">Закрыть</button>
        </div>
    </div>

    <!-- Модалка Экипажа -->
    <div id="crew-modal" class="modal hidden">
        <div class="modal-content">
            <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:15px;">
                <h2 style="margin:0">Управление Экипажем</h2>
                <button onclick="createNewCrewMember()" class="success">➕ Новый сотрудник</button>
            </div>
            <div class="split-view">
                <div class="list-col"><div id="crew-roster-list"></div></div>
                <div class="edit-col">
                    <h3 id="edit-title">Выберите сотрудника</h3>
                    <div id="edit-form" class="hidden">
                        <input type="hidden" id="edit-id">
                        <div class="form-group"><label>Имя</label><input type="text" id="edit-name"></div>
                        <div class="form-group">
                            <label>Категория (Роль)</label>
                            <select id="edit-role">
                                <option value="command">👑 Управляющий</option>
                                <option value="support">🤝 Сопровождающий</option>
                                <option value="tech">🔧 Технический</option>
                                <option value="military">⚔️ Военизированный</option>
                                <option value="other">📦 Прочий</option>
                            </select>
                        </div>
                        <div class="form-group">
                            <label style="display:flex; align-items:center; gap:10px; cursor:pointer; background:#0f172a; padding:10px; border-radius:5px; border:1px solid #475569;">
                                <input type="checkbox" id="edit-is-leader" style="width:auto;">
                                <span>⭐ Назначить ГЛАВОЙ категории</span>
                            </label>
                        </div>
                        <div class="form-group">
                            <label>Навыки</label>
                            <div id="edit-skills-list"></div>
                            <div style="display:flex; gap:5px; margin-top:5px;">
                                <input type="text" id="new-skill-name" placeholder="Название навыка" style="flex:2">
                                <input type="number" id="new-skill-val" placeholder="Знач." style="width:60px">
                                <button onclick="addSkillFromEdit()">+</button>
                            </div>
                        </div>
                        <div style="margin-top:auto; display:flex; justify-content:space-between; padding-top:20px;">
                            <button onclick="deleteCurrentCrew()" class="danger">Удалить</button>
                            <button onclick="saveCurrentCrew()" class="success">Сохранить</button>
                        </div>
                    </div>
                </div>
            </div>
            <button onclick="closeModal('crew-modal')" style="margin-top:15px; width: 100%;">Закрыть</button>
        </div>
    </div>

    <!-- Confirm Modal -->
    <div id="confirm-modal" class="modal hidden">
        <div id="confirm-box">
            <h3>Вы уверены?</h3>
            <p>Это действие нельзя отменить.</p>
            <div style="display:flex; gap:20px; justify-content:center; margin-top:20px;">
                <button onclick="confirmAction(true)" class="danger">Да</button>
                <button onclick="confirmAction(false)">Отмена</button>
            </div>
        </div>
    </div>

    <div id="context-menu" class="hidden"></div>
    <div id="toast">Сохранено!</div>

    <script>
        // --- CONSTANTS ---
        const ROLES = {
            command: { label: 'Управляющий', icon: '👑' },
            support: { label: 'Сопровождающий', icon: '🤝' },
            tech: { label: 'Технический', icon: '🔧' },
            military: { label: 'Военный', icon: '⚔️' },
            other: { label: 'Прочий', icon: '📦' }
        };

        const DEFAULT_MODULES = {
            empty: { id:'empty', name: 'Пусто', img: '', description:'', type: 'none', energyCost: 0, resourceType:'none', resourceVal:0, skills:[] },
            wall: { id:'wall', name: 'Стена', img: 'https://placehold.co/100x100/334155/94a3b8?text=Wall', description:'Утепленная стена, защищающая от внешней стужи.', energyCost: 0, resourceType:'none', resourceVal:0, skills:[] },
            engine_s: { id:'engine_s', name: 'Паровой Котел', img: 'https://placehold.co/100x100/7f1d1d/f1f5f9?text=Steam', description:'Простой паровой агрегат. Сжигает топливо для выработки энергии.', energyCost: 0, resourceType:'energy', resourceVal:10, skills:[] },
            bunks: { id:'bunks', name: 'Кубрик', img: 'https://placehold.co/100x100/047857/f1f5f9?text=Bunks', description:'Жилой модуль с двухъярусными кроватями.', energyCost: 2, resourceType:'crew', resourceVal:4, skills:[] },
            workshop: { 
                id:'workshop', name: 'Мастерская', img: 'https://placehold.co/100x100/581c87/f1f5f9?text=Shop', description:'Позволяет проводить ремонт и обслуживание оборудования.', energyCost: 5, resourceType:'none', resourceVal:0, 
                skills: [{name:'Механика', type:'mandatory'}] 
            }
        };

        // --- STATE ---
        let appState = {
            isEditMode: false,
            bgImage: 'https://placehold.co/1000x500/0f172a/334155?text=Background', 
            moduleLibrary: {}, 
            slots: [], 
            crew: [], 
            leaders: { command: null, support: null, tech: null, military: null, other: null }
        };

        let dragTarget = null, resizeTarget = null;
        let startX, startY, startLeft, startTop, startWidth, startHeight;
        let editingCrewId = null;
        let editingModuleId = null;
        let pendingAction = null;
        let detailSlotIndex = -1; // New: slot currently being viewed in Details modal

        // --- INIT ---
        function init() {
            loadData();
            if (Object.keys(appState.moduleLibrary).length === 0) {
                appState.moduleLibrary = JSON.parse(JSON.stringify(DEFAULT_MODULES));
            }
            if (appState.slots.length === 0) createDefaultLayout();
            render();
            setupDragAndDrop();
        }

        function createDefaultLayout() {
            appState.slots = [
                { id: 1, x: 10, y: 20, w: 15, h: 20, moduleId: 'engine_s', crewId: null },
                { id: 2, x: 30, y: 20, w: 15, h: 20, moduleId: 'bunks', crewId: null },
                { id: 3, x: 50, y: 20, w: 15, h: 20, moduleId: 'wall', crewId: null }
            ];
        }

        // --- RENDER ---
        function render() {
            document.getElementById('background-layer').style.backgroundImage = `url('${appState.bgImage}')`;
            renderSlots();
            updateStats();
        }

        function renderSlots() {
            const container = document.getElementById('slots-layer');
            container.innerHTML = '';
            const stats = calculateStats(); 

            appState.slots.forEach((slot, index) => {
                const module = appState.moduleLibrary[slot.moduleId] || appState.moduleLibrary.empty;
                const isPowered = stats.poweredIndices.includes(index);
                
                const el = document.createElement('div');
                el.className = 'slot';
                
                if (!isPowered && module.energyCost > 0) {
                    el.classList.add('is-offline');
                    el.title = "НЕТ ПИТАНИЯ!";
                }
                
                el.style.left = slot.x + '%';
                el.style.top = slot.y + '%';
                el.style.width = slot.w + '%';
                el.style.height = slot.h + '%';
                if (module.img) el.style.backgroundImage = `url('${module.img}')`;

                if (!appState.isEditMode) {
                    const crewBtn = document.createElement('div');
                    crewBtn.className = 'slot-ui-icon';
                    if (slot.crewId) {
                        const person = appState.crew.find(c => c.id === slot.crewId);
                        crewBtn.innerHTML = person ? ROLES[person.role].icon : '❓';
                        crewBtn.title = person ? `${person.name} (${ROLES[person.role].label})` : 'Unknown';
                    } else {
                        crewBtn.innerHTML = '➕';
                        crewBtn.title = "Назначить";
                    }
                    crewBtn.onclick = (e) => { e.stopPropagation(); openAssignmentMenu(e, index); };
                    el.appendChild(crewBtn);
                    
                    const bonusData = calculateComplexBonus(slot, module);
                    if (bonusData.total > 0) {
                        const bonusBadge = document.createElement('div');
                        bonusBadge.className = 'slot-bonus';
                        if (!isPowered) bonusBadge.classList.add('offline-bonus');
                        else if (bonusData.hasLeaderBuff) bonusBadge.classList.add('buffed');
                        bonusBadge.innerText = '+' + bonusData.total.toFixed(2);
                        el.appendChild(bonusBadge);
                    }

                    if (module.resourceBoost && isPowered && bonusData.total >= 1 && module.resourceType !== 'none') {
                        const boostIcon = document.createElement('div');
                        boostIcon.className = 'boost-icon';
                        const boostVal = Math.floor(bonusData.total);
                        boostIcon.innerText = `⏫+${boostVal}`;
                        el.appendChild(boostIcon);
                    }
                    
                    // CLICK LOGIC: Empty -> Select, Filled -> Details
                    el.onclick = () => {
                        if (!slot.moduleId || slot.moduleId === 'empty') {
                            openSlotModuleSelect(index);
                        } else {
                            openModuleDetails(index);
                        }
                    };
                }

                if (appState.isEditMode) {
                    const handle = document.createElement('div');
                    handle.className = 'resize-handle';
                    handle.onmousedown = (e) => startResize(e, index);
                    el.appendChild(handle);
                    const delBtn = document.createElement('div');
                    delBtn.className = 'delete-slot-btn';
                    delBtn.innerText = '×';
                    delBtn.onclick = (e) => { e.stopPropagation(); requestDeleteSlot(index); };
                    el.appendChild(delBtn);
                    el.onmousedown = (e) => startDrag(e, index);
                }
                container.appendChild(el);
            });
        }

        // --- DETAILS LOGIC (NEW) ---
        function openModuleDetails(index) {
            detailSlotIndex = index;
            const slot = appState.slots[index];
            const module = appState.moduleLibrary[slot.moduleId];
            const stats = calculateStats();
            const isPowered = stats.poweredIndices.includes(index);
            const person = slot.crewId ? appState.crew.find(c => c.id === slot.crewId) : null;
            
            const content = document.getElementById('details-content');
            
            // Build Breakdown HTML
            let calcHTML = '';
            let skillsHTML = '';
            
            if (module.skills && module.skills.length > 0) {
                skillsHTML = `<div style="margin: 10px 0;"><strong>Требуемые навыки:</strong><ul>`;
                module.skills.forEach(s => {
                    skillsHTML += `<li>${s.name} <span style="color:#94a3b8">(${s.type === 'mandatory' ? 'Обязательный' : 'Опциональный'})</span></li>`;
                });
                skillsHTML += `</ul></div>`;
                
                // Bonus Calc Breakdown
                if (person) {
                    calcHTML += `<div class="calc-box">`;
                    calcHTML += `<div class="calc-row"><span>Назначен:</span> <strong>${person.name}</strong></div>`;
                    
                    // 1. Personal
                    let pBonus = calculatePersonSkillPower(person, module.skills);
                    calcHTML += `<div class="calc-row"><span>Личный бонус навыков:</span> <span>+${pBonus.toFixed(2)}</span></div>`;
                    
                    // 2. Leader
                    const leaderId = appState.leaders[person.role];
                    if (leaderId && leaderId !== person.id) {
                        const leader = appState.crew.find(c => c.id === leaderId);
                        if (leader) {
                            let lBonus = calculatePersonSkillPower(leader, module.skills);
                            calcHTML += `<div class="calc-row"><span class="calc-note">Сравнение с Главой (${leader.name}):</span> <span class="calc-note">${lBonus.toFixed(2)}</span></div>`;
                            
                            if (lBonus > pBonus) {
                                const buff = Math.ceil((lBonus/4) / 0.25) * 0.25;
                                calcHTML += `<div class="calc-row"><span>Бонус Лидера (1/4):</span> <strong>+${buff.toFixed(2)}</strong></div>`;
                                
                                let total = pBonus + buff;
                                if (total > lBonus) {
                                    total = lBonus;
                                    calcHTML += `<div class="calc-row"><span style="color:#ef4444">Кап Лидера (max):</span> <span>${lBonus.toFixed(2)}</span></div>`;
                                }
                                calcHTML += `<div class="calc-row" style="border-top:1px solid #475569; margin-top:5px; padding-top:5px;"><span>Итоговый бонус:</span> <strong style="color:#10b981">+${total.toFixed(2)}</strong></div>`;
                            } else {
                                calcHTML += `<div class="calc-row"><span class="calc-note">Бонус главы не применяется (Сотрудник опытнее или равен)</span></div>`;
                            }
                        }
                    } else {
                         calcHTML += `<div class="calc-row" style="border-top:1px solid #475569; margin-top:5px; padding-top:5px;"><span>Итоговый бонус:</span> <strong style="color:#10b981">+${pBonus.toFixed(2)}</strong></div>`;
                    }
                    
                    if (module.resourceBoost && module.resourceType !== 'none') {
                        const finalBonus = calculateComplexBonus(slot, module).total;
                        const boost = Math.floor(finalBonus);
                        calcHTML += `<div class="calc-row" style="margin-top:10px; color:#3b82f6"><span>⏫ Буст ресурса (${module.resourceType}):</span> <strong>+${boost}</strong></div>`;
                    }
                    
                    calcHTML += `</div>`;
                } else {
                    calcHTML += `<div class="calc-box" style="color:#94a3b8; text-align:center;">Экипаж не назначен. Бонусы не действуют.</div>`;
                }
            }

            // Power Status
            let powerStatus = isPowered ? '<span style="color:#10b981">⚡ В сети</span>' : '<span style="color:#ef4444">⚡ НЕТ ПИТАНИЯ</span>';
            if (module.energyCost === 0) powerStatus = '<span style="color:#10b981">⚡ Автономный</span>';

            content.innerHTML = `
                <div class="details-header">
                    <img src="${module.img || 'https://placehold.co/150?text=No+Img'}" class="details-img">
                    <div class="details-info">
                        <h2 style="margin:0">${module.name}</h2>
                        <div style="margin-top:5px;">${powerStatus}</div>
                        <div style="margin-top:5px; font-size:0.9em;">
                            ${module.energyCost > 0 ? `Потребление: <strong>${module.energyCost}</strong>` : ''}
                            ${module.resourceType !== 'none' ? `Производство (${module.resourceType}): <strong>${module.resourceVal}</strong>` : ''}
                        </div>
                    </div>
                </div>
                <div class="details-desc">${module.description || 'Нет описания.'}</div>
                ${skillsHTML}
                ${calcHTML}
            `;
            
            document.getElementById('module-details-modal').classList.remove('hidden');
        }

        function changeModuleFromDetails() {
            closeModal('module-details-modal');
            openSlotModuleSelect(detailSlotIndex);
        }

        function removeModuleFromDetails() {
            if(confirm("Снять модуль?")) {
                appState.slots[detailSlotIndex].moduleId = 'empty';
                appState.slots[detailSlotIndex].crewId = null;
                render();
                saveData();
                closeModal('module-details-modal');
            }
        }

        // --- MATH & LOGIC ---
        function calculateComplexBonus(slot, module) {
            let result = { total: 0, hasLeaderBuff: false, raw: 0 };
            if (!slot.crewId || !module.skills || module.skills.length === 0) return result;
            const person = appState.crew.find(c => c.id === slot.crewId);
            if (!person) return result;

            const personalTotal = calculatePersonSkillPower(person, module.skills);
            let finalTotal = personalTotal;
            const leaderId = appState.leaders[person.role];
            
            if (leaderId && leaderId !== person.id) {
                const leader = appState.crew.find(c => c.id === leaderId);
                if (leader) {
                    const leaderTotal = calculatePersonSkillPower(leader, module.skills);
                    if (leaderTotal > personalTotal) {
                        const buff = Math.ceil((leaderTotal / 4) / 0.25) * 0.25;
                        let potentialSum = personalTotal + buff;
                        if (potentialSum > leaderTotal) potentialSum = leaderTotal;
                        if (potentialSum > finalTotal) {
                            finalTotal = potentialSum;
                            result.hasLeaderBuff = true;
                        }
                    }
                }
            }
            result.total = finalTotal;
            return result;
        }

        function calculatePersonSkillPower(person, requiredSkills) {
            let mandatorySum = 0; let mandatoryCount = 0; let optionalStack = 0;
            requiredSkills.forEach(req => {
                const pSkill = person.skills.find(s => s.name.trim().toLowerCase() === req.name.trim().toLowerCase());
                const val = pSkill ? pSkill.val : 0;
                if (req.type === 'mandatory') { mandatorySum += val; mandatoryCount++; } 
                else {
                    const standardBonus = Math.floor(val / 2) * 0.25;
                    optionalStack += Math.ceil((standardBonus / 4) / 0.25) * 0.25;
                }
            });
            let mandatoryBonus = 0;
            if (mandatoryCount > 0) mandatoryBonus = Math.floor((mandatorySum / mandatoryCount) / 2) * 0.25;
            return mandatoryBonus + optionalStack;
        }

        function calculateStats() {
            let energyProduced = 0, energyUsed = 0, poweredIndices = [];
            appState.slots.forEach(s => {
                const m = appState.moduleLibrary[s.moduleId];
                if(m && m.resourceType === 'energy') energyProduced += parseInt(m.resourceVal);
            });
            let currentEnergy = energyProduced;
            appState.slots.forEach((s, idx) => {
                const m = appState.moduleLibrary[s.moduleId];
                if (!m) return;
                const cost = parseInt(m.energyCost || 0);
                if (cost === 0) poweredIndices.push(idx);
                else if (currentEnergy >= cost) { currentEnergy -= cost; energyUsed += cost; poweredIndices.push(idx); }
            });
            let finalEnergy = 0, finalCrew = 0;
            appState.slots.forEach((s, idx) => {
                const m = appState.moduleLibrary[s.moduleId];
                if (!m) return;
                const isPowered = poweredIndices.includes(idx);
                let val = parseInt(m.resourceVal || 0);
                if (isPowered && m.resourceBoost && s.crewId) val += Math.floor(calculateComplexBonus(s, m).total);
                if (isPowered || m.energyCost === 0) {
                    if (m.resourceType === 'energy') finalEnergy += val;
                    if (m.resourceType === 'crew') finalCrew += val;
                }
            });
            return { produced: finalEnergy, consumed: energyUsed, poweredIndices, maxCrew: finalCrew };
        }

        function updateStats() {
            const stats = calculateStats();
            document.getElementById('energy-val').innerText = stats.consumed;
            document.getElementById('energy-max').innerText = stats.produced;
            document.getElementById('crew-val').innerText = appState.crew.length;
            document.getElementById('crew-max').innerText = stats.maxCrew;
        }

        // --- MODULE EDITOR ---
        document.getElementById('modules-btn').onclick = () => { renderModuleEditorList(); document.getElementById('mod-edit-form').classList.add('hidden'); document.getElementById('mod-edit-placeholder').classList.remove('hidden'); document.getElementById('module-editor-modal').classList.remove('hidden'); };

        function renderModuleEditorList() {
            const list = document.getElementById('mod-edit-list'); list.innerHTML = '';
            Object.values(appState.moduleLibrary).forEach(mod => {
                const item = document.createElement('div'); item.className = 'crew-item'; 
                if (editingModuleId === mod.id) item.classList.add('selected');
                item.innerHTML = `<div style="display:flex; align-items:center; gap:10px;"><img src="${mod.img}" style="width:30px; height:30px; object-fit:cover; border-radius:3px; background:#000;"><span>${mod.name}</span></div>`;
                item.onclick = () => loadModuleForEditing(mod.id); list.appendChild(item);
            });
        }
        function createNewModule() {
            const newId = 'mod_' + Date.now();
            appState.moduleLibrary[newId] = { id: newId, name: 'Новый модуль', img: '', description:'', energyCost: 0, resourceType: 'none', resourceVal: 0, resourceBoost: false, skills: [] };
            saveData(); renderModuleEditorList(); loadModuleForEditing(newId);
        }
        function loadModuleForEditing(id) {
            editingModuleId = id; const mod = appState.moduleLibrary[id];
            renderModuleEditorList();
            document.getElementById('mod-edit-placeholder').classList.add('hidden'); document.getElementById('mod-edit-form').classList.remove('hidden');
            document.getElementById('me-id').value = mod.id; document.getElementById('me-name').value = mod.name;
            document.getElementById('me-desc').value = mod.description || '';
            document.getElementById('me-img').value = mod.img; document.getElementById('me-cost').value = mod.energyCost;
            document.getElementById('me-res-type').value = mod.resourceType; document.getElementById('me-res-val').value = mod.resourceVal;
            document.getElementById('me-res-boost').checked = mod.resourceBoost || false;
            renderModuleSkills(mod.skills);
        }
        function renderModuleSkills(skills) {
            const container = document.getElementById('me-skills-list'); container.innerHTML = '';
            skills.forEach((s, idx) => {
                const row = document.createElement('div'); row.className = 'mod-skill-row';
                row.innerHTML = `<input type="text" value="${s.name}" onchange="updateModSkill(${idx}, 'name', this.value)" style="flex:2" placeholder="Навык"><select onchange="updateModSkill(${idx}, 'type', this.value)" style="flex:1"><option value="mandatory" ${s.type === 'mandatory' ? 'selected' : ''}>Обяз.</option><option value="optional" ${s.type === 'optional' ? 'selected' : ''}>Опц.</option></select><button class="danger" onclick="removeModSkill(${idx})" style="padding:5px 8px;">×</button>`;
                container.appendChild(row);
            });
        }
        function updateCurrentModule() {
            if (!editingModuleId) return;
            const mod = appState.moduleLibrary[editingModuleId];
            mod.name = document.getElementById('me-name').value; mod.description = document.getElementById('me-desc').value;
            mod.img = document.getElementById('me-img').value; mod.energyCost = parseInt(document.getElementById('me-cost').value) || 0;
            mod.resourceType = document.getElementById('me-res-type').value; mod.resourceVal = parseInt(document.getElementById('me-res-val').value) || 0;
            mod.resourceBoost = document.getElementById('me-res-boost').checked;
            saveData(); renderModuleEditorList(); render();
        }
        function addSkillToModule() { if (!editingModuleId) return; const mod = appState.moduleLibrary[editingModuleId]; if (!mod.skills) mod.skills = []; mod.skills.push({ name: '', type: 'mandatory' }); renderModuleSkills(mod.skills); saveData(); }
        window.updateModSkill = (idx, field, val) => { const mod = appState.moduleLibrary[editingModuleId]; mod.skills[idx][field] = val; saveData(); };
        window.removeModSkill = (idx) => { const mod = appState.moduleLibrary[editingModuleId]; mod.skills.splice(idx, 1); renderModuleSkills(mod.skills); saveData(); };
        function deleteCurrentModule() { if(!editingModuleId) return; pendingAction = { type: 'deleteModule', id: editingModuleId }; document.getElementById('confirm-modal').classList.remove('hidden'); }

        // --- SLOT & SELECTION ---
        let selectionSlotIndex = -1;
        function openSlotModuleSelect(index) {
            selectionSlotIndex = index;
            const grid = document.getElementById('module-list-grid'); grid.innerHTML = '';
            Object.values(appState.moduleLibrary).forEach(mod => {
                const card = document.createElement('div'); card.className = 'module-card';
                card.innerHTML = `<img src="${mod.img || ''}"><strong>${mod.name}</strong><br><small>${mod.energyCost > 0 ? '⚡'+mod.energyCost : (mod.resourceType==='energy' ? '⚡+'+mod.resourceVal : 'Free')}</small>`;
                card.onclick = () => { appState.slots[selectionSlotIndex].moduleId = mod.id; appState.slots[selectionSlotIndex].crewId = null; render(); closeModal('module-select-modal'); saveData(); };
                grid.appendChild(card);
            });
            document.getElementById('module-select-modal').classList.remove('hidden');
        }

        // --- CREW EDITOR (Existing) ---
        document.getElementById('crew-btn').onclick = () => { renderCrewList(); document.getElementById('edit-form').classList.add('hidden'); document.getElementById('edit-title').innerText = "Выберите сотрудника"; document.getElementById('crew-modal').classList.remove('hidden'); };
        function renderCrewList() {
            const list = document.getElementById('crew-roster-list'); list.innerHTML = '';
            const sortedCrew = [...appState.crew].sort((a, b) => a.role.localeCompare(b.role));
            sortedCrew.forEach(c => {
                const item = document.createElement('div'); item.className = 'crew-item'; if (editingCrewId === c.id) item.classList.add('selected');
                const roleInfo = ROLES[c.role]; const leaderIcon = appState.leaders[c.role] === c.id ? '⭐' : '';
                item.innerHTML = `<div style="display:flex; align-items:center; gap:10px;"><span style="font-size:1.5em">${roleInfo.icon}</span><div><div style="font-weight:bold;">${c.name} ${leaderIcon}</div><div style="font-size:0.8em; color:#cbd5e1">${roleInfo.label}</div></div></div>`;
                item.onclick = () => loadCrewForEditing(c.id); list.appendChild(item);
            });
        }
        function createNewCrewMember() { const newId = 'c' + Date.now(); appState.crew.push({ id: newId, name: 'Сотрудник', role: 'other', skills: [] }); saveData(); renderCrewList(); loadCrewForEditing(newId); }
        function loadCrewForEditing(id) {
            editingCrewId = id; renderCrewList(); const person = appState.crew.find(c => c.id === id); if (!person) return;
            document.getElementById('edit-form').classList.remove('hidden'); document.getElementById('edit-title').innerText = person.name; document.getElementById('edit-id').value = person.id; document.getElementById('edit-name').value = person.name; document.getElementById('edit-role').value = person.role; document.getElementById('edit-is-leader').checked = (appState.leaders[person.role] === person.id); renderCrewSkills(person.skills);
        }
        function renderCrewSkills(skills) {
            const container = document.getElementById('edit-skills-list'); container.innerHTML = '';
            skills.forEach((s, idx) => {
                const row = document.createElement('div'); row.className = 'mod-skill-row';
                row.innerHTML = `<input type="text" value="${s.name}" onchange="updateCrewSkill(${idx}, 'name', this.value)" style="flex:2"><input type="number" value="${s.val}" onchange="updateCrewSkill(${idx}, 'val', this.value)" style="width:60px"><button class="danger" onclick="removeCrewSkill(${idx})" style="padding:5px 8px;">×</button>`;
                container.appendChild(row);
            });
        }
        window.updateCrewSkill = (idx, f, v) => { const p = appState.crew.find(c => c.id === editingCrewId); if(f==='val') v=parseInt(v); p.skills[idx][f] = v; saveData(); };
        window.removeCrewSkill = (idx) => { const p = appState.crew.find(c => c.id === editingCrewId); p.skills.splice(idx, 1); renderCrewSkills(p.skills); saveData(); };
        function addSkillFromEdit() { const name = document.getElementById('new-skill-name').value; const val = parseInt(document.getElementById('new-skill-val').value); if(name && val && editingCrewId) { const p = appState.crew.find(c => c.id === editingCrewId); p.skills.push({name, val}); document.getElementById('new-skill-name').value=''; document.getElementById('new-skill-val').value=''; renderCrewSkills(p.skills); saveData(); } }
        function deleteCurrentCrew() { if(!editingCrewId) return; pendingAction = { type: 'deleteCrew', id: editingCrewId }; document.getElementById('confirm-modal').classList.remove('hidden'); }
        function saveCurrentCrew() {
            if(!editingCrewId) return; const p = appState.crew.find(c => c.id === editingCrewId); p.name = document.getElementById('edit-name').value;
            const oldRole = p.role; const newRole = document.getElementById('edit-role').value; p.role = newRole; const isL = document.getElementById('edit-is-leader').checked;
            if (oldRole !== newRole && appState.leaders[oldRole] === p.id) appState.leaders[oldRole] = null;
            if (isL) appState.leaders[newRole] = p.id; else if (appState.leaders[newRole] === p.id) appState.leaders[newRole] = null;
            saveData(); renderCrewList(); render(); showToast('Сохранено');
        }

        // --- GLOBAL ACTIONS ---
        function requestDeleteSlot(index) { pendingAction = { type: 'deleteSlot', index: index }; document.getElementById('confirm-modal').classList.remove('hidden'); }
        function confirmAction(isConfirmed) {
            document.getElementById('confirm-modal').classList.add('hidden');
            if (isConfirmed && pendingAction) {
                if (pendingAction.type === 'deleteSlot') appState.slots.splice(pendingAction.index, 1);
                else if (pendingAction.type === 'deleteCrew') { appState.crew = appState.crew.filter(c => c.id !== pendingAction.id); appState.slots.forEach(s => { if(s.crewId === pendingAction.id) s.crewId = null; }); Object.keys(appState.leaders).forEach(r => { if(appState.leaders[r] === pendingAction.id) appState.leaders[r] = null; }); editingCrewId = null; document.getElementById('edit-form').classList.add('hidden'); }
                else if (pendingAction.type === 'deleteModule') { delete appState.moduleLibrary[pendingAction.id]; editingModuleId = null; document.getElementById('mod-edit-form').classList.add('hidden'); document.getElementById('mod-edit-placeholder').classList.remove('hidden'); renderModuleEditorList(); }
                saveData(); render(); showToast('Выполнено');
            }
            pendingAction = null;
        }

        // --- DRAG, DROP, SYSTEM ---
        function setupDragAndDrop() {
            window.addEventListener('mousemove', (e) => {
                const c = document.getElementById('drill-wrapper'), r = c.getBoundingClientRect();
                if (dragTarget !== null) { appState.slots[dragTarget].x = startLeft + ((e.clientX - startX) / r.width) * 100; appState.slots[dragTarget].y = startTop + ((e.clientY - startY) / r.height) * 100; renderSlots(); }
                if (resizeTarget !== null) { appState.slots[resizeTarget].w = Math.max(3, startWidth + ((e.clientX - startX) / r.width) * 100); appState.slots[resizeTarget].h = Math.max(3, startHeight + ((e.clientY - startY) / r.height) * 100); renderSlots(); }
            });
            window.addEventListener('mouseup', () => { if(dragTarget!==null||resizeTarget!==null) saveData(); dragTarget=null; resizeTarget=null; });
        }
        function startDrag(e, index) { if(e.target.className.includes('resize') || e.target.className.includes('delete')) return; dragTarget=index; startX=e.clientX; startY=e.clientY; startLeft=appState.slots[index].x; startTop=appState.slots[index].y; }
        function startResize(e, index) { e.stopPropagation(); resizeTarget=index; startX=e.clientX; startY=e.clientY; startWidth=appState.slots[index].w; startHeight=appState.slots[index].h; }
        
        function openAssignmentMenu(e, index) {
            const menu = document.getElementById('context-menu'); menu.innerHTML = ''; menu.style.left = e.pageX + 'px'; menu.style.top = e.pageY + 'px';
            const sortedCrew = [...appState.crew].sort((a,b) => a.role.localeCompare(b.role));
            if(sortedCrew.length===0) menu.innerHTML='<div style="color:#aaa">Нет экипажа</div>';
            sortedCrew.forEach(c => {
                const item = document.createElement('div'); const isBusy = appState.slots.some((s, i) => s.crewId === c.id && i !== index);
                item.innerHTML = `${ROLES[c.role].icon} ${c.name} ${isBusy?'<small style="color:red">(Занят)</small>':''}`;
                if(!isBusy) item.onclick = () => { appState.slots[index].crewId = c.id; render(); menu.classList.add('hidden'); saveData(); showToast('Назначен'); };
                else { item.style.opacity=0.5; item.style.cursor='default'; }
                menu.appendChild(item);
            });
            const un = document.createElement('div'); un.innerText='❌ Снять'; un.style.borderTop='1px solid #475569';
            un.onclick = () => { appState.slots[index].crewId=null; render(); menu.classList.add('hidden'); saveData(); };
            menu.appendChild(un); menu.classList.remove('hidden');
        }

        document.getElementById('mode-play-btn').onclick = () => { appState.isEditMode=false; updateModeUI(); };
        document.getElementById('mode-edit-btn').onclick = () => { appState.isEditMode=true; updateModeUI(); };
        function updateModeUI() { document.body.className = appState.isEditMode ? 'mode-edit' : 'mode-play'; document.getElementById('mode-play-btn').className = !appState.isEditMode?'active':''; document.getElementById('mode-edit-btn').className = appState.isEditMode?'active':''; const btn = document.getElementById('add-slot-btn'); appState.isEditMode ? btn.classList.remove('hidden') : btn.classList.add('hidden'); render(); }
        document.getElementById('add-slot-btn').onclick = () => { appState.slots.push({ id:Date.now(), x:40, y:40, w:10, h:10, moduleId:'empty', crewId:null }); render(); saveData(); };
        document.getElementById('bg-settings-btn').onclick = () => document.getElementById('background-controls').classList.toggle('hidden');
        document.getElementById('apply-bg-btn').onclick = () => { const u = document.getElementById('bg-url-input').value; if(u){appState.bgImage=u; render(); saveData();} };
        document.addEventListener('click', (e) => { if(!e.target.closest('#context-menu') && !e.target.closest('.slot-ui-icon')) document.getElementById('context-menu').classList.add('hidden'); });
        window.closeModal = (id) => document.getElementById(id).classList.add('hidden');
        function showToast(m) { const t=document.getElementById('toast'); t.innerText=m; t.classList.add('show'); setTimeout(()=>t.classList.remove('show'),2000); }
        function saveData() { localStorage.setItem('drillApp_v5_1', JSON.stringify(appState)); }
        function loadData() { const d = localStorage.getItem('drillApp_v5_1'); if(d) { appState = JSON.parse(d); if(!appState.moduleLibrary) appState.moduleLibrary={}; } }
        document.getElementById('save-btn').onclick = () => { saveData(); showToast('Сохранено'); };
        document.getElementById('load-btn').onclick = () => { loadData(); init(); };

        init();
    </script>
</body>
</html>
