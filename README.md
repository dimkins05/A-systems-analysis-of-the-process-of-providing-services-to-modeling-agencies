<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Метод идеальной точки — курсовой проект</title>
  <style>
    :root{
      --bg:#0b1220; --card:#111a2e; --text:#e8eefc; --muted:#a9b7d5;
      --line:#233052; --accent:#7aa2ff; --bad:#ff6b6b; --good:#44d7a8;
      --input:#0d162a;
    }
    body{ margin:0; font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial,sans-serif; background:var(--bg); color:var(--text); }
    .wrap{ max-width:1200px; margin:0 auto; padding:18px; }
    h1{ font-size:20px; margin:0 0 12px; }
    h2{ font-size:16px; margin:18px 0 10px; color:var(--text); }
    p,li{ color:var(--muted); line-height:1.35; }
    .row{ display:flex; gap:12px; flex-wrap:wrap; }
    .card{
      background:var(--card); border:1px solid var(--line); border-radius:12px;
      padding:12px; box-shadow:0 0 0 1px rgba(0,0,0,.12) inset;
    }
    .grow{ flex:1 1 520px; }
    .small{ flex:1 1 320px; }
    .actions{ display:flex; gap:10px; flex-wrap:wrap; align-items:center; }
    button{
      background:linear-gradient(180deg,#1b2a52,#152244); border:1px solid var(--line);
      color:var(--text); padding:10px 12px; border-radius:10px; cursor:pointer;
      font-weight:600;
    }
    button:hover{ border-color:#3550a0; }
    button.secondary{ background:transparent; }
    button.danger{ background:linear-gradient(180deg,#5a1e24,#3a1418); border-color:#6d2a33; }
    .note{ font-size:12px; color:var(--muted); margin-top:8px; }
    .warn{ color:#ffd166; }
    .ok{ color:var(--good); }
    .err{ color:var(--bad); }
    label{ display:block; font-size:12px; color:var(--muted); margin-bottom:6px; }
    input, select, textarea{
      width:100%; box-sizing:border-box; background:var(--input); color:var(--text);
      border:1px solid var(--line); border-radius:10px; padding:8px 10px;
      outline:none;
    }
    input:focus, select:focus, textarea:focus{ border-color:#3c64ff; }
    table{ width:100%; border-collapse:separate; border-spacing:0; overflow:hidden; border-radius:12px; border:1px solid var(--line); }
    th, td{ border-bottom:1px solid var(--line); border-right:1px solid var(--line); padding:8px; vertical-align:middle; }
    th:last-child, td:last-child{ border-right:none; }
    tr:last-child td{ border-bottom:none; }
    th{ text-align:left; background:#0f1930; font-size:12px; color:var(--muted); }
    .grid2{ display:grid; grid-template-columns:1fr 1fr; gap:10px; }
    .grid3{ display:grid; grid-template-columns:repeat(3,1fr); gap:10px; }
    .mono{ font-family:ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; }
    .pill{
      display:inline-block; padding:2px 8px; border-radius:999px; font-size:12px;
      border:1px solid var(--line); color:var(--muted);
    }
    .resultBox{
      background:#0d162a; border:1px solid var(--line); border-radius:12px; padding:10px;
    }
    .resultTop{ display:flex; gap:12px; flex-wrap:wrap; align-items:center; justify-content:space-between; }
    .best{ font-weight:800; color:var(--good); }
    .muted{ color:var(--muted); }
    .nowrap{ white-space:nowrap; }
  </style>
</head>
<body>
<div class="wrap">
  <h1>Метод «идеальной точки»</h1>

  <div class="row">
    <div class="card small">
      <h2>Управление</h2>
      <div class="actions">
		<button id="btnLoadDefault">Загрузить тестовые данные</button>
		<button id="btnLoadFile">Загрузить из файла</button>
		<button id="btnSave">Сохранить в файл</button>
		<input type="file" id="inpFile" style="display:none" accept=".json">
        <button id="btnCalc">Рассчитать</button>
        <button class="secondary" id="btnGenerate">Сгенерировать пустые таблицы</button>
        <button class="danger" id="btnClear">Очистить всё</button>
      </div>

      <div style="margin-top:10px" class="grid3">
        <div>
          <label>Альтернатив (N)</label>
          <input id="inpNAlt" type="number" min="1" value="5" />
        </div>
        <div>
          <label>Критериев (M)</label>
          <input id="inpNCrit" type="number" min="1" value="4" />
        </div>
        <div>
          <label>Экспертов (E)</label>
          <input id="inpNExp" type="number" min="1" value="3" />
        </div>
      </div>

      <div class="note">
        <div class="pill">Поддержка max/min</div>
        <div class="pill">Авто-нормировка весов</div>
        <div class="pill">Коэф. вариации</div>
      </div>

      <div id="status" class="note"></div>
    </div>

    <div class="card grow">
      <h2>Кратко</h2>
      <ul>
        <li>Заполните таблицы (или нажмите «Загрузить данные проекта»).</li>
        <li>Нажмите «Рассчитать» — появятся обобщённые оценки, нормированные оценки и расстояния до идеальной точки.</li>
        <li>Лучшая альтернатива — с минимальным расстоянием.</li>
      </ul>
    </div>
  </div>

  <div class="row">
    <div class="card grow" id="blockAlts">
      <h2>Альтернативы</h2>
      <div id="altsTable"></div>
    </div>

    <div class="card grow" id="blockCrits">
      <h2>Критерии</h2>
      <div id="critsTable"></div>
      <div class="note">
        Тип: <span class="mono">max</span> — «чем больше, тем лучше», <span class="mono">min</span> — «чем меньше, тем лучше».
      </div>
    </div>
  </div>

  <div class="row">
    <div class="card grow" id="blockExps">
      <h2>Эксперты</h2>
      <div id="expsTable"></div>
    </div>
  </div>

  <div class="row">
    <div class="card grow" id="blockRatings">
      <h2>Оценки экспертов</h2>
      <div class="note">
        Формат: строка = (критерий, эксперт), столбцы = альтернативы. Значения обычно на шкале 1..10.
      </div>
      <div id="ratingsTable"></div>
    </div>
  </div>

  <div class="row">
    <div class="card grow">
      <h2>Результаты</h2>
      <div class="resultBox">
        <div class="resultTop">
          <div>
            <div class="muted">Лучшая альтернатива</div>
            <div id="bestAlt" class="best">—</div>
          </div>
          <div>
            <div class="muted">Минимальное расстояние</div>
            <div id="bestDist" class="mono">—</div>
          </div>
          <div>
          </div>
        </div>
      </div>

      <h2>Обобщённые оценки (aᵢⱼ) + Kv%</h2>
      <div id="aggTable"></div>

      <h2>Нормированные оценки (qᵢⱼ)</h2>
      <div id="normTable"></div>

      <h2>Расстояния и ранжирование</h2>
      <div id="distTable"></div>
    </div>
  </div>
</div>

<script>
/** ======= ДЕФОЛТНЫЕ ДАННЫЕ ИЗ ПРОЕКТА ======= **/
const DEFAULT_DATA = {
  alts: [
    { id: "x1", name: "Внедрение CRM с модулем кастинга" },
    { id: "x2", name: "Собственное веб-приложение для менеджеров" },
    { id: "x3", name: "Интеграция с внешней AI-платформой" },
    { id: "x4", name: "Надстройка/макрос к существующей БД" },
    { id: "x5", name: "SaaS-решение для модельных агентств" },
  ],
  crits: [
    { id: "k1", name: "Скорость внедрения", weight: 0.35, type: "max" },
    { id: "k2", name: "Стоимость реализации", weight: 0.25, type: "min" }, // в проекте выше балл = ниже затраты
    { id: "k3", name: "Надёжность и техподдержка", weight: 0.20, type: "max" },
    { id: "k4", name: "Качество интеграции", weight: 0.20, type: "max" },
  ],
  exps: [
    { id: "e1", name: "Эксперт 1 (IT-руководитель)", weight: 0.40 },
    { id: "e2", name: "Эксперт 2 (Менеджер по подбору)", weight: 0.35 },
    { id: "e3", name: "Эксперт 3 (Финансовый директор)", weight: 0.25 },
  ],
  // ratings[expId][critId] = [x1..xN]
  ratings: {
    e1: {
      k1: [8,4,7,9,8],
      k2: [6,3,7,9,8],
      k3: [9,7,6,5,8],
      k4: [7,9,8,6,8],
    },
    e2: {
      k1: [9,3,6,8,9],
      k2: [5,2,6,10,7],
      k3: [8,6,7,4,9],
      k4: [8,10,7,5,9],
    },
    e3: {
      k1: [7,2,5,10,8],
      k2: [4,2,5,8,6],
      k3: [8,5,5,3,8],
      k4: [6,8,6,4,7],
    },
  },
  // ожидаемые расстояния из таблицы проекта (для быстрой сверки)
  expectedDistances: { x1:0.323, x2:0.637, x3:0.520, x4:0.806, x5:0.381 }
};

function el(id){ return document.getElementById(id); }
function num(v, def=0){
  const x = (typeof v === "string") ? v.replace(",", ".").trim() : v;
  const n = Number(x);
  return Number.isFinite(n) ? n : def;
}
function fmt(n, d=3){
  if(!Number.isFinite(n)) return "—";
  return n.toFixed(d);
}
function setStatus(html){
  el("status").innerHTML = html;
}

/** ======= ХРАНИЛИЩЕ ТЕКУЩИХ ДАННЫХ (в DOM) ======= **/
function generateEmpty(nAlt, nCrit, nExp){
  const alts = Array.from({length:nAlt}, (_,i)=>({id:`x${i+1}`, name:`Альтернатива x${i+1}`}));
  const crits = Array.from({length:nCrit}, (_,i)=>({id:`k${i+1}`, name:`Критерий k${i+1}`, weight: (1/nCrit), type:"max"}));
  const exps = Array.from({length:nExp}, (_,i)=>({id:`e${i+1}`, name:`Эксперт ${i+1}`, weight:(1/nExp)}));
  const ratings = {};
  for(const e of exps){
    ratings[e.id] = {};
    for(const k of crits){
      ratings[e.id][k.id] = Array.from({length:nAlt}, ()=>5);
    }
  }
  return {alts, crits, exps, ratings};
}

function loadIntoUI(data){
  el("inpNAlt").value = data.alts.length;
  el("inpNCrit").value = data.crits.length;
  el("inpNExp").value = data.exps.length;

  renderAlts(data.alts);
  renderCrits(data.crits);
  renderExps(data.exps);
  renderRatings(data.alts, data.crits, data.exps, data.ratings);

  setStatus(`<span class="ok">Данные загружены.</span>`);
}

function clearAll(){
  el("altsTable").innerHTML = "";
  el("critsTable").innerHTML = "";
  el("expsTable").innerHTML = "";
  el("ratingsTable").innerHTML = "";

  el("aggTable").innerHTML = "";
  el("normTable").innerHTML = "";
  el("distTable").innerHTML = "";
  el("bestAlt").textContent = "—";
  el("bestDist").textContent = "—";
  el("checkInfo").textContent = "—";

  setStatus(`<span class="warn">Очищено. Нажмите «Загрузить данные проекта» или «Сгенерировать пустые таблицы».</span>`);
}

/** ======= РЕНДЕР ТАБЛИЦ ======= **/
function renderAlts(alts){
  const rows = alts.map((a,i)=>`
    <tr>
      <td class="mono nowrap">${a.id}</td>
      <td><input data-alt="name" data-i="${i}" value="${escapeHtml(a.name)}" /></td>
    </tr>`).join("");

  el("altsTable").innerHTML = `
    <table>
      <thead><tr><th style="width:110px">ID</th><th>Название</th></tr></thead>
      <tbody>${rows}</tbody>
    </table>`;
}

function renderCrits(crits){
  const rows = crits.map((c,i)=>`
    <tr>
      <td class="mono nowrap">${c.id}</td>
      <td><input data-crit="name" data-i="${i}" value="${escapeHtml(c.name)}" /></td>
      <td style="width:150px"><input data-crit="weight" data-i="${i}" value="${String(c.weight).replace(".", ",")}" /></td>
      <td style="width:120px">
        <select data-crit="type" data-i="${i}">
          <option value="max" ${c.type==="max"?"selected":""}>max</option>
          <option value="min" ${c.type==="min"?"selected":""}>min</option>
        </select>
      </td>
    </tr>`).join("");

  el("critsTable").innerHTML = `
    <table>
      <thead><tr><th style="width:110px">ID</th><th>Название</th><th>Вес vᵢ</th><th>Тип</th></tr></thead>
      <tbody>${rows}</tbody>
    </table>`;
}

function renderExps(exps){
  const rows = exps.map((e,i)=>`
    <tr>
      <td class="mono nowrap">${e.id}</td>
      <td><input data-exp="name" data-i="${i}" value="${escapeHtml(e.name)}" /></td>
      <td style="width:150px"><input data-exp="weight" data-i="${i}" value="${String(e.weight).replace(".", ",")}" /></td>
    </tr>`).join("");

  el("expsTable").innerHTML = `
    <table>
      <thead><tr><th style="width:110px">ID</th><th>Имя</th><th>Компет. kᵢ</th></tr></thead>
      <tbody>${rows}</tbody>
    </table>`;
}


// функция для контроля корректности ввода
function validateInput(inp) {
  // Заменяем запятую на точку для проверки числа
  const valStr = inp.value.replace(",", ".");
  const val = Number(valStr);

  // Проверка: если не число, пустое значение или меньше 0
  // жесткая граница 1..10
  if (inp.value.trim() === "" || !Number.isFinite(val) || val < 0 || val > 10 ) {
    inp.style.backgroundColor = "var(--bad)";
    inp.style.color = "#fff"; // Белый текст на красном фоне
  } else {
    inp.style.backgroundColor = ""; // Сброс цвета
    inp.style.color = "";
  }
}

function renderRatings(alts, crits, exps, ratings){
  const head = `
    <tr>
      <th style="width:140px">Критерий</th>
      <th style="width:220px">Эксперт</th>
      ${alts.map(a=>`<th class="mono nowrap">${a.id}</th>`).join("")}
    </tr>`;

  const body = crits.map(c=>{
    return exps.map(e=>{
      const arr = (ratings?.[e.id]?.[c.id]) || Array.from({length:alts.length},()=>5);
      const cells = alts.map((_,j)=>`
        <td>
          <input
            class="val-input"
            data-rating="1"
            data-exp="${e.id}"
            data-crit="${c.id}"
            data-alt-index="${j}"
            value="${String(arr[j] ?? 5).replace(".", ",")}"
            oninput="validateInput(this)" 
          />
        </td>`).join("");
      return `
        <tr>
          <td class="mono nowrap">${c.id}</td>
          <td>${escapeHtml(e.name)}</td>
          ${cells}
        </tr>`;
    }).join("");
  }).join("");

  el("ratingsTable").innerHTML = `
    <table>
      <thead>${head}</thead>
      <tbody>${body}</tbody>
    </table>`;
}


function escapeHtml(s){
  return String(s ?? "")
    .replaceAll("&","&amp;")
    .replaceAll("<","&lt;")
    .replaceAll(">","&gt;")
    .replaceAll('"',"&quot;")
    .replaceAll("'","&#039;");
}

/** ======= СЧИТЫВАНИЕ ДАННЫХ ИЗ UI ======= **/
function readFromUI(){
  // alts
  const nAlt = num(el("inpNAlt").value, 0);
  const nCrit = num(el("inpNCrit").value, 0);
  const nExp = num(el("inpNExp").value, 0);

  if(!(nAlt>0 && nCrit>0 && nExp>0)) throw new Error("N/M/E должны быть > 0.");

  const altNameInputs = Array.from(document.querySelectorAll('[data-alt="name"]'));
  const alts = Array.from({length:nAlt}, (_,i)=>({
    id:`x${i+1}`,
    name: altNameInputs[i]?.value?.trim() || `Альтернатива x${i+1}`
  }));

  const critNameInputs = Array.from(document.querySelectorAll('[data-crit="name"]'));
  const critWeightInputs = Array.from(document.querySelectorAll('[data-crit="weight"]'));
  const critTypeSelects = Array.from(document.querySelectorAll('[data-crit="type"]'));
  const crits = Array.from({length:nCrit}, (_,i)=>({
    id:`k${i+1}`,
    name: critNameInputs[i]?.value?.trim() || `Критерий k${i+1}`,
    weight: num(critWeightInputs[i]?.value, 0),
    type: (critTypeSelects[i]?.value === "min") ? "min" : "max"
  }));

  const expNameInputs = Array.from(document.querySelectorAll('[data-exp="name"]'));
  const expWeightInputs = Array.from(document.querySelectorAll('[data-exp="weight"]'));
  const exps = Array.from({length:nExp}, (_,i)=>({
    id:`e${i+1}`,
    name: expNameInputs[i]?.value?.trim() || `Эксперт ${i+1}`,
    weight: num(expWeightInputs[i]?.value, 0)
  }));

  // ratings
  const ratings = {};
  for(const e of exps){
    ratings[e.id] = {};
    for(const c of crits){
      ratings[e.id][c.id] = Array.from({length:nAlt}, ()=>0);
    }
  }
  const rInputs = Array.from(document.querySelectorAll('[data-rating="1"]'));
  for(const inp of rInputs){
    const eId = inp.getAttribute("data-exp");
    const cId = inp.getAttribute("data-crit");
    const j = num(inp.getAttribute("data-alt-index"), 0);
    if(ratings?.[eId]?.[cId] && j>=0 && j<nAlt){
      ratings[eId][cId][j] = num(inp.value, 0);
    }
  }

  return {alts, crits, exps, ratings};
}

/** ======= МАТЕМАТИКА: ИДЕАЛЬНАЯ ТОЧКА ======= **/
function normalizeWeights(arr, field){
  const sum = arr.reduce((s,x)=>s + (Number.isFinite(x[field]) ? x[field] : 0), 0);
  if(sum <= 0){
    const v = 1/arr.length;
    return {norm: arr.map(x=>({...x, [field]: v})), sumBefore: sum, changed:true};
  }
  const norm = arr.map(x=>({...x, [field]: x[field]/sum}));
  const changed = Math.abs(sum - 1) > 1e-9;
  return {norm, sumBefore: sum, changed};
}

function calcIdealPoint(data){
  // 1) Нормируем веса критериев и экспертов (чтобы сумма = 1)
  const critW = normalizeWeights(data.crits, "weight");
  const expW  = normalizeWeights(data.exps,  "weight");

  const crits = critW.norm;
  const exps  = expW.norm;

  const nAlt = data.alts.length;
  const nCrit = crits.length;

  // 2) Обобщенные оценки a[i][j] = sum_e k_e * rating[e][i][j]
  const agg = Array.from({length:nCrit}, ()=>Array.from({length:nAlt}, ()=>0));
  const kv  = Array.from({length:nCrit}, ()=>Array.from({length:nAlt}, ()=>0)); // коэффициент вариации, %

  for(let i=0;i<nCrit;i++){
    const cId = crits[i].id;
    for(let j=0;j<nAlt;j++){
      let mean = 0;
      for(const e of exps){
        const r = data.ratings?.[e.id]?.[cId]?.[j];
        mean += e.weight * num(r, 0);
      }
      agg[i][j] = mean;

      // взвешенная дисперсия относительно mean
      let varw = 0;
      for(const e of exps){
        const r = num(data.ratings?.[e.id]?.[cId]?.[j], 0);
        varw += e.weight * Math.pow(r - mean, 2);
      }
      const sigma = Math.sqrt(varw);
      kv[i][j] = (mean !== 0) ? (sigma / mean) * 100 : 0;
    }
  }

  // 3) Нормировка по критериям (min-max) в [0;1]
  const norm = Array.from({length:nCrit}, ()=>Array.from({length:nAlt}, ()=>0));
  for(let i=0;i<nCrit;i++){
    const row = agg[i];
    const qmin = Math.min(...row);
    const qmax = Math.max(...row);
    for(let j=0;j<nAlt;j++){
      if(Math.abs(qmax - qmin) < 1e-12){
        norm[i][j] = 1; // все одинаковые => считаем равноидеальными
      } else if(crits[i].type === "min"){
        norm[i][j] = (qmax - row[j]) / (qmax - qmin);
      } else {
        norm[i][j] = (row[j] - qmin) / (qmax - qmin);
      }
      // поджать из-за возможных погрешностей
      norm[i][j] = Math.max(0, Math.min(1, norm[i][j]));
    }
  }

  // 4) Расстояние до идеальной точки (идеал = 1 по каждому критерию)
  const dist = Array.from({length:nAlt}, ()=>0);
  for(let j=0;j<nAlt;j++){
    let s = 0;
    for(let i=0;i<nCrit;i++){
      const w = crits[i].weight;
      const d = 1 - norm[i][j];
      s += w * d * d;
    }
    dist[j] = Math.sqrt(s);
  }

  // 5) Ранжирование
  const ranking = data.alts.map((a, j)=>({
    id: a.id, name: a.name, distance: dist[j]
  })).sort((a,b)=>a.distance - b.distance);

  return {crits, exps, agg, kv, norm, dist, ranking, critSum:critW.sumBefore, expSum:expW.sumBefore, critChanged:critW.changed, expChanged:expW.changed};
}

/** ======= ВЫВОД РЕЗУЛЬТАТОВ ======= **/
function renderAgg(data, res){
  const head = `
    <tr>
      <th style="width:110px">Критерий</th>
      <th>Название</th>
      ${data.alts.map(a=>`<th class="mono nowrap">${a.id} (a)</th>`).join("")}
      ${data.alts.map(a=>`<th class="mono nowrap">${a.id} (Kv%)</th>`).join("")}
    </tr>`;
    
  const body = res.crits.map((c,i)=>{
    const aCells = data.alts.map((_,j)=>`<td class="mono">${fmt(res.agg[i][j], 2)}</td>`).join("");
    
    // Проверка Kv > 33%
    const kvCells = data.alts.map((_,j)=> {
        const val = res.kv[i][j];
        // Если вариация > 33%, считаем выборку неоднородной (подсвечиваем)
        const style = (val > 33) ? 'color:var(--bad); font-weight:bold;' : '';
        return `<td class="mono" style="${style}">${fmt(val, 2)}</td>`;
    }).join("");

    return `<tr>
      <td class="mono nowrap">${c.id}</td>
      <td>${escapeHtml(c.name)}</td>
      ${aCells}${kvCells}
    </tr>`;
  }).join("");

  el("aggTable").innerHTML = `<table><thead>${head}</thead><tbody>${body}</tbody></table>`;
}


function renderNorm(data, res){
  const head = `
    <tr>
      <th style="width:110px">Критерий</th>
      <th style="width:160px">Вес vᵢ</th>
      <th style="width:90px">Тип</th>
      ${data.alts.map(a=>`<th class="mono nowrap">${a.id}</th>`).join("")}
    </tr>`;
  const body = res.crits.map((c,i)=>{
    const cells = data.alts.map((_,j)=>`<td class="mono">${fmt(res.norm[i][j], 2)}</td>`).join("");
    return `<tr>
      <td class="mono nowrap">${c.id}</td>
      <td class="mono">${fmt(c.weight, 4)}</td>
      <td class="mono">${c.type}</td>
      ${cells}
    </tr>`;
  }).join("");

  el("normTable").innerHTML = `<table><thead>${head}</thead><tbody>${body}</tbody></table>`;
}

function renderDist(data, res){
  const head = `<tr><th style="width:110px">ID</th><th>Альтернатива</th><th style="width:200px">Расстояние</th><th style="width:120px">Место</th></tr>`;
  const rows = res.ranking.map((r,idx)=>`
    <tr>
      <td class="mono nowrap">${r.id}</td>
      <td>${escapeHtml(r.name)}</td>
      <td class="mono">${fmt(r.distance, 3)}</td>
      <td class="mono">${idx+1}</td>
    </tr>`).join("");
  el("distTable").innerHTML = `<table><thead>${head}</thead><tbody>${rows}</tbody></table>`;

  const best = res.ranking[0];
  el("bestAlt").textContent = `${best.id}: ${best.name}`;
  el("bestDist").textContent = fmt(best.distance, 3);
}


function showWeightWarnings(res){
  const w = [];
  if(res.critChanged){
    w.push(`<div class="warn">Сумма весов критериев была ${fmt(res.critSum,4)} → автоматически нормирована до 1.</div>`);
  }
  if(res.expChanged){
    w.push(`<div class="warn">Сумма компетентностей экспертов была ${fmt(res.expSum,4)} → автоматически нормирована до 1.</div>`);
  }
  if(w.length===0) w.push(`<div class="ok">Веса критериев и экспертов корректны (сумма = 1).</div>`);
  setStatus(w.join(""));
}

/** ======= КНОПКИ ======= **/
el("btnLoadDefault").addEventListener("click", ()=>{
  loadIntoUI(DEFAULT_DATA);
  // сразу посчитать, чтобы было видно
  tryCalculate();
});

el("btnGenerate").addEventListener("click", ()=>{
  const nAlt = Math.max(1, Math.floor(num(el("inpNAlt").value, 5)));
  const nCrit = Math.max(1, Math.floor(num(el("inpNCrit").value, 4)));
  const nExp = Math.max(1, Math.floor(num(el("inpNExp").value, 3)));
  loadIntoUI(generateEmpty(nAlt, nCrit, nExp));
});

el("btnClear").addEventListener("click", clearAll);
el("btnCalc").addEventListener("click", tryCalculate);

function tryCalculate(){
  try{
    const data = readFromUI();
    const res = calcIdealPoint(data);

    showWeightWarnings(res);

    renderAgg(data, res);
    renderNorm(data, res);
    renderDist(data, res);
    <!-- renderCheck(res); -->
  } catch(e){
    setStatus(`<span class="err">Ошибка: ${escapeHtml(e.message || String(e))}</span>`);
  }
}


/** ======= СОХРАНЕНИЕ И ЗАГРУЗКА ======= **/
el("btnSave").addEventListener("click", () => {
  try {
    const data = readFromUI(); // Считываем текущее состояние таблиц
    const json = JSON.stringify(data, null, 2);
    const blob = new Blob([json], {type: "application/json"});
    const url = URL.createObjectURL(blob);
    
    const a = document.createElement("a");
    a.href = url;
    a.download = "ideal_point_project.json";
    a.click();
    URL.revokeObjectURL(url);
  } catch(e) {
    alert("Ошибка сохранения: " + e.message);
  }
});

el("btnLoadFile").addEventListener("click", () => el("inpFile").click());

el("inpFile").addEventListener("change", (e) => {
  const file = e.target.files[0];
  if (!file) return;
  
  const reader = new FileReader();
  reader.onload = (ev) => {
    try {
      const data = JSON.parse(ev.target.result);
      loadIntoUI(data); // Загружаем данные в таблицы и обновляем N, M, E
      tryCalculate();   // Сразу пересчитываем
      setStatus(`<span class="ok">Файл «${file.name}» успешно загружен.</span>`);
      el("inpFile").value = ""; // Сброс, чтобы можно было загрузить тот же файл снова
    } catch(ex) {
      alert("Ошибка чтения файла: " + ex.message);
    }
  };
  reader.readAsText(file);
});


// при первом открытии страницы — загрузить проектные данные
loadIntoUI(DEFAULT_DATA);
tryCalculate();
</script>
</body>
</html>
