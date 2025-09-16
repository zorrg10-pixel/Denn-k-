<!DOCTYPE html>
<html lang="sk">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
  <title>Denník – Minimal</title>
  <style>
    body {
      margin: 0;
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
      background: #f9fafb;
    }
    header {
      position: fixed;
      top: 0; left: 0; right: 0;
      background: #e6eafe;
      border-bottom: 1px solid #ccc;
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 10px;
      padding: 10px;
      z-index: 1000;
    }
    main {
      padding: 90px 15px 20px;
    }
    #editor {
      width: min(900px, 96vw);
      margin: 0 auto;
      background: #fff;
      border: 1px solid #ccc;
      border-radius: 8px;
      padding: 20px;
      min-height: 70vh;
      outline: none;
      font-size: 18px;
      line-height: 1.6;
    }
    button, select {
      padding: 6px 12px;
      border-radius: 6px;
      border: 1px solid #ccc;
      background: #fff;
      cursor: pointer;
    }
    .date {
      font-weight: bold;
      color: #555;
    }
    .modal-overlay {
      position: fixed;
      inset: 0;
      display: none;
      align-items: center;
      justify-content: center;
      background: rgba(0,0,0,.35);
      z-index: 2000;
    }
    .modal-overlay.show { display: flex; }
    .modal {
      background: #fff;
      border-radius: 8px;
      border: 1px solid #ccc;
      width: min(800px, 90vw);
      max-height: 80vh;
      display: flex;
      flex-direction: column;
    }
    .modal header {
      padding: 10px;
      border-bottom: 1px solid #ddd;
      background: #f9fafb;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .modal .content {
      padding: 10px;
      overflow: auto;
    }
    .rowTpl {
      display: grid;
      grid-template-columns: 1fr auto auto auto;
      gap: 6px;
      margin-bottom: 6px;
      align-items: center;
    }
    .opts-pop {
      position: fixed;
      top: 70px; right: 10px;
      background: #fff;
      border: 1px solid #ccc;
      border-radius: 6px;
      padding: 6px;
      display: none;
      gap: 6px;
      z-index: 1500;
    }
    .opts-pop.show { display: flex; }
  </style>
</head>
<body>
  <header>
    <button id="decreaseFont">A-</button>
    <button id="prevDay">⟵</button>
    <button id="todayBtn">Dnes</button>
    <button id="nextDay">⟶</button>
    <button id="increaseFont">A+</button>
    <select id="fontFamily">
      <option value="" disabled selected>Font</option>
      <option value="system-ui,-apple-system,'Segoe UI',Roboto,Arial,sans-serif">Predvolený</option>
      <option value="Georgia, 'Times New Roman', Times, serif">Georgia</option>
      <option value="Arial, Helvetica, sans-serif">Arial</option>
      <option value="'Courier New', Courier, monospace">Monospace</option>
    </select>
    <button id="saveTemplateBtn">Uložiť šablónu</button>
    <button id="manageTemplatesBtn">Šablóny</button>
    <button id="optionsBtn">Možnosti ▾</button>
    <span class="date" id="dateDisplay"></span>
  </header>

  <main>
    <div id="editor" contenteditable="true"></div>
  </main>

  <!-- Možnosti -->
  <div class="opts-pop" id="opts">
    <button id="optBold"><b>Hrúbka</b></button>
    <button id="optOL">1·2·3</button>
    <input type="color" id="optColor" value="#000000" />
  </div>

  <!-- Šablóny -->
  <div class="modal-overlay" id="tplModal">
    <div class="modal">
      <header>
        <strong>Šablóny</strong>
        <button id="closeTplModal">Zavrieť</button>
      </header>
      <div class="content">
        <div id="tplList"></div>
        <hr/>
        <input id="newTplName" placeholder="Názov novej šablóny" />
        <button id="createTplFromCurrent">Uložiť aktuálnu</button>
      </div>
    </div>
  </div>

  <script>
    const editor = document.getElementById('editor');
    const dateDisplay = document.getElementById('dateDisplay');
    const prevDayBtn = document.getElementById('prevDay');
    const nextDayBtn = document.getElementById('nextDay');
    const todayBtn = document.getElementById('todayBtn');
    const increaseFontBtn = document.getElementById('increaseFont');
    const decreaseFontBtn = document.getElementById('decreaseFont');
    const fontFamilySel = document.getElementById('fontFamily');
    const optionsBtn = document.getElementById('optionsBtn');
    const opts = document.getElementById('opts');
    const optBold = document.getElementById('optBold');
    const optOL = document.getElementById('optOL');
    const optColor = document.getElementById('optColor');

    const saveTemplateBtn = document.getElementById('saveTemplateBtn');
    const manageTemplatesBtn = document.getElementById('manageTemplatesBtn');
    const tplModal = document.getElementById('tplModal');
    const tplList  = document.getElementById('tplList');
    const closeTplModal = document.getElementById('closeTplModal');
    const createTplFromCurrent = document.getElementById('createTplFromCurrent');
    const newTplName = document.getElementById('newTplName');

    let currentDate = new Date();
    function fmt(date){ return date.toISOString().split('T')[0]; }
    function keyFor(date){ return 'notes-' + fmt(date); }

    function loadDay(){
      const html = localStorage.getItem(keyFor(currentDate));
      editor.innerHTML = html || '';
      dateDisplay.textContent = fmt(currentDate);
    }
    function saveDay(){
      if(editor.textContent.trim().length === 0){
        localStorage.removeItem(keyFor(currentDate));
      } else {
        localStorage.setItem(keyFor(currentDate), editor.innerHTML);
      }
    }
    editor.addEventListener('input', saveDay);

    prevDayBtn.addEventListener('click', ()=>{ currentDate.setDate(currentDate.getDate()-1); loadDay(); });
    nextDayBtn.addEventListener('click', ()=>{ currentDate.setDate(currentDate.getDate()+1); loadDay(); });
    todayBtn.addEventListener('click', ()=>{ currentDate = new Date(); loadDay(); });

    increaseFontBtn.addEventListener('click', ()=> document.execCommand("fontSize", false, "5"));
    decreaseFontBtn.addEventListener('click', ()=> document.execCommand("fontSize", false, "2"));
    fontFamilySel.addEventListener('change', ()=> document.execCommand("fontName", false, fontFamilySel.value));

    optionsBtn.addEventListener('click', ()=> opts.classList.toggle('show'));
    optBold.addEventListener('click', ()=> document.execCommand('bold'));
    optOL.addEventListener('click', ()=> document.execCommand('insertOrderedList'));
    optColor.addEventListener('input', ()=> document.execCommand('foreColor', false, optColor.value));

    // Šablóny
    function readTemplates(){ return JSON.parse(localStorage.getItem("templates")||"[]"); }
    function writeTemplates(arr){ localStorage.setItem("templates", JSON.stringify(arr)); }
    function renderTplList(){
      const list = readTemplates();
      tplList.innerHTML = "";
      list.forEach((tpl,i)=>{
        const row = document.createElement("div"); row.className="rowTpl";
        row.innerHTML = `<span>${tpl.name}</span>
          <button onclick="applyTpl(${i},'add')">Pridať</button>
          <button onclick="applyTpl(${i},'replace')">Nahradiť</button>
          <button onclick="delTpl(${i})">Vymazať</button>`;
        tplList.appendChild(row);
      });
    }
    function applyTpl(i,mode){
      const tpl = readTemplates()[i];
      if(mode==="replace"){ editor.innerHTML = tpl.html; }
      else { editor.innerHTML += "<p></p>"+tpl.html; }
      saveDay();
    }
    function delTpl(i){
      const arr = readTemplates();
      arr.splice(i,1); writeTemplates(arr); renderTplList();
    }
    saveTemplateBtn.addEventListener('click', ()=>{
      const name = prompt("Názov šablóny:")||"Šablóna";
      const arr = readTemplates();
      arr.push({name, html: editor.innerHTML});
      writeTemplates(arr);
    });
    manageTemplatesBtn.addEventListener('click', ()=>{ renderTplList(); tplModal.classList.add('show'); });
    closeTplModal.addEventListener('click', ()=> tplModal.classList.remove('show'));
    createTplFromCurrent.addEventListener('click', ()=>{
      const arr = readTemplates();
      arr.push({name:newTplName.value||"Šablóna", html:editor.innerHTML});
      writeTemplates(arr); renderTplList();
    });

    window.onload = ()=> loadDay();
  </script>
</body>
</html>