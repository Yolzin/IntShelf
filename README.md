<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Интерактивный библиотечный стеллаж</title>
  <style>
    :root{
      --bg:#0f172a;
      --panel:#111827ee;
      --panel-2:#0b1222;
      --accent:#60a5fa;
      --muted:#94a3b8;
      --text:#e5e7eb;
      --wood-1: #f3e6d0; /* светлая древесина */
      --wood-2: #e6d1b3;
      --dark-bg: #2b2f33; /* темно-серый фон позади */
      --shadow: 0 10px 30px rgba(0,0,0,.35);
      --radius: 18px;
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{margin:0;background:linear-gradient(180deg,var(--dark-bg),#0b1222);color:var(--text);font:16px/1.5 system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, Noto Sans}

    .app{display:grid;grid-template-columns:340px 1fr;grid-template-rows:auto 1fr;gap:14px;height:100%;padding:14px}
    .toolbar{grid-column:1 / -1;display:flex;flex-wrap:wrap;align-items:center;gap:10px;background:linear-gradient(180deg, #0b1222, #0a1020);border:1px solid #1f2937;border-radius:var(--radius);padding:12px 14px;box-shadow:var(--shadow)}
    .sidebar{background:linear-gradient(180deg, var(--panel), var(--panel-2));border:1px solid #1f2937;border-radius:var(--radius);padding:12px;box-shadow:var(--shadow);overflow:auto}
    .stage-wrap{position:relative;background:var(--dark-bg);border:1px solid #1f2937;border-radius:var(--radius);box-shadow:var(--shadow);overflow:hidden}

    .toolbar .group{display:flex;align-items:center;gap:8px;padding:8px 10px;background:#0b1222;border:1px solid #1f2937;border-radius:12px}
    .toolbar label{font-size:12px;color:var(--muted)}
    .toolbar input[type="range"]{accent-color:var(--accent)}
    .btn{appearance:none;border:none;border-radius:12px;padding:10px 12px;background:#111827;color:var(--text);border:1px solid #243045;cursor:pointer;transition:.2s transform,.2s background;display:inline-flex;align-items:center;gap:8px}
    .btn:hover{background:#172033}
    .btn:active{transform:translateY(1px)}
    .btn.secondary{background:#0b1222}
    .btn.ghost{background:transparent;border-color:#1f2937}
    .btn.ok{background:linear-gradient(180deg, #10b981, #059669);border-color:#047857}
    .btn.warn{background:linear-gradient(180deg, #f59e0b, #d97706);border-color:#b45309}
    .btn.err{background:linear-gradient(180deg, #ef4444, #dc2626);border-color:#b91c1c}
    .sep{width:1px;height:28px;background:#283142;margin:0 6px;border-radius:99px}

    .block{background:#0b1222;border:1px solid #1f2937;border-radius:14px;padding:12px;margin-bottom:12px}
    .block h3{margin:0 0 8px;font-size:14px;color:#cbd5e1}
    .field{display:grid;grid-template-columns:90px 1fr;gap:8px;align-items:center;margin:8px 0}
    .field label{font-size:13px;color:var(--muted)}
    .field input,.field select,.field textarea{width:100%;padding:8px 10px;border-radius:10px;border:1px solid #243045;background:#0a1326;color:var(--text)}
    .field input[type="color"]{padding:0;height:38px}
    .hint{font-size:12px;color:#93a0b4}

    .stage{position:absolute;inset:0;transform-origin:0 0;display:flex;align-items:center;justify-content:center;padding:60px}
    .rack{position:relative;border-radius:16px;border:1px solid #243045;box-shadow:inset 0 0 0 1px #0f1a2e, 0 8px 30px rgba(0,0,0,.35);background:linear-gradient(180deg,#0b1222,#0d1528);min-width:420px}
    .rack-header{display:flex;align-items:center;justify-content:space-between;gap:8px;padding:8px 10px;background:#0a1326;border-bottom:1px solid #1f2937;border-top-left-radius:16px;border-top-right-radius:16px}
    .rack-body{position:relative;padding:12px;background:var(--dark-bg);border-bottom-left-radius:16px;border-bottom-right-radius:16px}

    /* Shelf grid - cells are strictly square */
    .shelf{position:relative;margin:10px 0;border-radius:12px;background:linear-gradient(180deg,var(--wood-1),var(--wood-2));border:1px solid #d8cdb6;padding:8px;overflow:hidden}
    .shelf-inner{position:relative;display:grid;gap:6px}
    .shelf-inner.square{grid-auto-rows:1fr}
    .shelf-bg{position:absolute;inset:0;background-size:cover;background-position:center;opacity:.6;pointer-events:none}
    .shelf-label{position:absolute;left:8px;bottom:6px;background:rgba(0,0,0,.45);backdrop-filter: blur(4px);border:1px solid #1f2937;border-radius:8px;padding:2px 6px;font-size:12px;color:#0b1222}

    .cell{position:relative;border-radius:8px;background:transparent;border:1px solid rgba(0,0,0,.08);display:flex;align-items:flex-end;justify-content:center;user-select:none;aspect-ratio:1 / 1;overflow:visible}
    .cell.highlight{outline:2px solid var(--accent);outline-offset:-2px}
    .book{position:relative;bottom:2px;border-radius:6px;display:flex;flex-direction:column;justify-content:flex-end;align-items:center;padding:6px 8px;gap:4px;color:#0b1222;cursor:grab;border:1px solid rgba(0,0,0,.2);width:70%;height:86%;box-shadow:inset 0 0 0 1px rgba(255,255,255,.12), 0 6px 18px rgba(0,0,0,.25)}
    .book:active{cursor:grabbing}
    .book .spine{writing-mode:horizontal-tb;text-orientation:mixed;font-weight:700;letter-spacing:.02em;font-size:12px;text-align:center;max-height:100%;overflow:hidden;color:#0b1222}
    .badge{position:absolute;top:6px;right:6px;font-size:10px;background:rgba(255,255,255,.75);color:#0b1222;border-radius:999px;padding:2px 6px}

    .dropzone{display:flex;align-items:center;justify-content:center;gap:10px;padding:12px;border:1px dashed #334155;border-radius:12px;background:#0a1326;color:#94a3b8;text-align:center}

    .row{display:flex;gap:8px;align-items:center}
    .row.wrap{flex-wrap:wrap}
    .grow{flex:1}
    .right{margin-left:auto}
    .small{font-size:12px;color:var(--muted)}
  </style>
</head>
<body>
  <div class="app">
    <div class="toolbar">
      <div class="group">
        <button class="btn ok" id="newRackBtn">➕ Новый стеллаж</button>
        <button class="btn" id="addShelfTop">➕ Полка сверху</button>
        <button class="btn" id="addShelfLeft">➕ Полка слева</button>
        <button class="btn" id="addShelfRight">➕ Полка справа</button>
        <button class="btn" id="addShelfBottom">➕ Полка снизу</button>
        <button class="btn secondary" id="addBookBtn">📚 Добавить книгу</button>
        <div class="sep"></div>
        <button class="btn ghost" id="saveBtn">💾 Сохранить</button>
        <button class="btn ghost" id="loadBtn">📂 Загрузить</button>
        <button class="btn err" id="resetBtn">🗑 Сброс</button>
      </div>
      <div class="group">
        <label for="zoomRange">Масштаб</label>
        <input type="range" id="zoomRange" min="20" max="200" step="1" value="100"/>
        <span id="zoomVal" class="small">100%</span>
      </div>
      <div class="group">
        <label>Прокрутка колесом: масштаб</label>
        <label class="small">Зажмите <kbd>Space</kbd> для перетаскивания</label>
      </div>
    </div>

    <aside class="sidebar">
      <div class="block" id="rackProps">
        <h3>Параметры стеллажа</h3>
        <div class="field">
          <label>Название</label>
          <input id="rackTitle" placeholder="Например: Домашняя библиотека"/>
        </div>
        <div class="field">
          <label>Ширина (клетки)</label>
          <input type="number" id="rackCols" min="1" max="20" value="6" />
        </div>
        <div class="field">
          <label>Высота полки</label>
          <input type="number" id="shelfRows" min="1" max="3" value="1" />
        </div>
        <div class="field">
          <label>Ширина книги</label>
          <input type="number" id="bookSpan" min="1" max="3" value="1" />
        </div>
        <div class="row wrap">
          <button class="btn" id="applyGrid">Применить</button>
          <button class="btn ghost" id="centerRack">Центрировать</button>
        </div>
      </div>

      <div class="block" id="bookForm">
        <h3>Создать книгу вручную</h3>
        <div class="field"><label>Название</label><input id="bookTitle" placeholder="Название на корешке"/></div>
        <div class="field"><label>Автор</label><input id="bookAuthor" placeholder="Автор (необязательно)"/></div>
        <div class="field"><label>Цвет</label><input type="color" id="bookColor" value="#9ae6b4"/></div>
        <div class="field"><label>Толщина (клеток)</label><input type="number" id="bookWidth" min="1" max="3" value="1"/></div>
        <div class="row wrap">
          <button class="btn ok" id="createBookBtn">Добавить книгу</button>
        </div>
        <p class="hint">Книгу можно перетаскивать мышью между клетками.</p>
      </div>

      <div class="block">
        <h3>Добавить по фото полки (авто-распознавание)</h3>
        <div class="dropzone" id="dropzone">Перетащите фото сюда или нажмите для выбора</div>
        <input type="file" id="fileInput" accept="image/*" hidden />
        <div class="field"><label>Непрозрачность</label><input type="range" id="photoOpacity" min="20" max="100" value="60"/></div>
        <div class="row wrap">
          <button class="btn" id="clearPhoto">Очистить фото</button>
        </div>
        <p class="hint">Фото будет подложкой полки. Код попытается распознать количество книг и разместить их автоматически.</p>
      </div>

      <div class="block">
        <h3>Подпись полки</h3>
        <div class="field"><label>Текст</label><input id="shelfLabelText" placeholder="Например: Фантастика"/></div>
        <div class="row wrap">
          <button class="btn" id="applyShelfLabel">Применить к выбранной полке</button>
          <button class="btn ghost" id="deleteShelfBtn">Удалить полку</button>
        </div>
        <p class="hint">Подпись появляется в нижнем левом углу полки. Двойной клик по полке — выбрать её.</p>
      </div>

      <div class="block">
        <h3>Экспорт / Импорт</h3>
        <div class="row wrap">
          <button class="btn ghost" id="exportJson">⬇️ Экспорт JSON</button>
          <button class="btn ghost" id="importJson">⬆️ Импорт JSON</button>
          <input type="file" id="importFile" accept="application/json" hidden />
        </div>
      </div>

      <p class="small">Подсказки: колесо мыши — масштаб, Space + перетаскивание — перемещение стеллажа. Переключение между полками — стрелки в шапке стеллажа или двойной клик.</p>
    </aside>

    <div class="stage-wrap">
      <div class="stage" id="stage"></div>
    </div>
  </div>

  <script>
    // --- State ---
    const state = { zoom:1, pan:{x:0,y:0}, racks:[], selection:{ rackId:null, shelfId:null } };
    const stage = document.getElementById('stage');
    const zoomRange = document.getElementById('zoomRange');
    const zoomVal = document.getElementById('zoomVal');

    const uid = () => Math.random().toString(36).slice(2,9);
    const qs = (sel,root=document) => root.querySelector(sel);
    const qsa = (sel,root=document) => Array.from(root.querySelectorAll(sel));
    const clamp = (v,min,max) => Math.max(min, Math.min(max, v));

    function toast(msg){ const t=document.createElement('div'); t.textContent=msg; Object.assign(t.style,{position:'fixed',bottom:'16px',right:'16px',background:'rgba(17,24,39,.95)',border:'1px solid #1f2937',padding:'10px 12px',borderRadius:'10px',boxShadow:'0 10px 30px rgba(0,0,0,.35)'}); document.body.appendChild(t); setTimeout(()=>t.remove(),1600); }

    // --- Rack & Shelf ---
    function newRack(){ const rack = { id:uid(), title:'Стеллаж', cols:6, shelves:[] }; state.racks.push(rack); state.selection.rackId = rack.id; addShelf(rack.id,1,'bottom'); render(); }

    function addShelf(rackId, rows=1, position='bottom', refShelfId=null){
      const rack = state.racks.find(r=>r.id===rackId); if(!rack) return;
      const shelf = { id:uid(), rows, label:'', bg:null, opacity:0.6, cells:[] };
      const total = rack.cols * rows; for(let i=0;i<total;i++) shelf.cells.push(null);
      if(position==='top') rack.shelves.unshift(shelf);
      else if(position==='before'){
        const idx=rack.shelves.findIndex(s=>s.id===refShelfId); rack.shelves.splice(idx===-1?0:idx,0,shelf);
      } else if(position==='after'){
        const idx=rack.shelves.findIndex(s=>s.id===refShelfId); rack.shelves.splice(idx===-1?rack.shelves.length:idx+1,0,shelf);
      } else rack.shelves.push(shelf);
      state.selection = { rackId: rack.id, shelfId: shelf.id };
      render();
    }

    function removeShelf(rackId, shelfId){ const rack = state.racks.find(r=>r.id===rackId); if(!rack) return; const idx=rack.shelves.findIndex(s=>s.id===shelfId); if(idx!==-1){ rack.shelves.splice(idx,1); state.selection.shelfId = rack.shelves[idx]? rack.shelves[idx].id : rack.shelves[idx-1]?.id || null; render(); } }

    function currentRack(){ return state.racks.find(r=>r.id===state.selection.rackId) || state.racks[0]; }
    function currentShelf(){ const rack=currentRack(); if(!rack) return null; return rack.shelves.find(s=>s.id===state.selection.shelfId) || rack.shelves[0] || null; }

    // --- Render ---
    function render(){ stage.innerHTML=''; const rack = currentRack(); if(!rack) return;
      const rackEl=document.createElement('div'); rackEl.className='rack'; rackEl.style.width = Math.max(420, rack.cols*90)+'px';

      const header=document.createElement('div'); header.className='rack-header';
      const title = document.createElement('input'); title.value = rack.title; title.className='grow'; title.style.background='#0b1222'; title.style.border='1px solid #1f2937'; title.style.color='var(--text)'; title.style.borderRadius='10px'; title.style.padding='6px 10px';
      title.addEventListener('input', ()=>{ rack.title = title.value; const sb = qs('#rackTitle'); if(sb) sb.value = title.value; });

      const nav = document.createElement('div'); nav.className='row';
      nav.appendChild(btn('◀ Стеллаж',()=> switchRack(-1), 'btn ghost'));
      nav.appendChild(btn('Стеллаж ▶',()=> switchRack(1), 'btn ghost'));
      nav.appendChild(btn('▲ Полка',()=> switchShelf(-1), 'btn ghost'));
      nav.appendChild(btn('▼ Полка',()=> switchShelf(1), 'btn ghost'));
      nav.appendChild(btn('✏️ Редактировать полку', ()=>{ const s=currentShelf(); if(!s) return; state.selection.shelfId=s.id; qs('#shelfLabelText').value=s.label||''; qs('#photoOpacity').value = Math.round((s.opacity||0.6)*100); toast('Выбрана полка для редактирования'); }, 'btn'));

      header.appendChild(title); header.appendChild(nav);

      const body = document.createElement('div'); body.className='rack-body';

      rack.shelves.forEach((shelf, si)=>{
        const shelfEl = document.createElement('div'); shelfEl.className='shelf'; shelfEl.dataset.shelfId = shelf.id;
        shelfEl.ondblclick = ()=>{ state.selection = { rackId: rack.id, shelfId: shelf.id }; render(); };

        const bg = document.createElement('div'); bg.className='shelf-bg'; if(shelf.bg){ bg.style.backgroundImage = `url(${shelf.bg})`; } bg.style.opacity = shelf.opacity; shelfEl.appendChild(bg);

        const inner = document.createElement('div'); inner.className='shelf-inner square'; inner.style.gridTemplateColumns = `repeat(${rack.cols}, 1fr)`;

        for(let i=0;i<rack.cols*shelf.rows;i++){
          const cell = document.createElement('div'); cell.className='cell'; cell.dataset.index=i;
          cell.addEventListener('dragover', e=>{ e.preventDefault(); cell.classList.add('highlight'); });
          cell.addEventListener('dragleave', ()=> cell.classList.remove('highlight'));
          cell.addEventListener('drop', e=> onDropBook(e, rack, shelf, i));

          const data = shelf.cells[i];
          if(data && !data.ghost){ cell.appendChild(bookEl(data)); }
          inner.appendChild(cell);
        }

        const label = document.createElement('div'); label.className='shelf-label'; label.textContent = shelf.label || '';
        shelfEl.appendChild(inner); shelfEl.appendChild(label);
        if(state.selection.shelfId === shelf.id) shelfEl.style.outline = '3px solid var(--accent)';
        body.appendChild(shelfEl);
      });

      rackEl.appendChild(header); rackEl.appendChild(body); stage.appendChild(rackEl);

      // sync sidebar
      const sbTitle = qs('#rackTitle'); if(sbTitle) sbTitle.value = rack.title;
      const cs = currentShelf(); if(cs){ qs('#shelfRows').value = cs.rows; qs('#shelfLabelText').value = cs.label||''; qs('#photoOpacity').value = Math.round((cs.opacity||0.6)*100); }

      applyPanZoom();
    }

    function btn(text,onClick,cls='btn'){ const b=document.createElement('button'); b.className=cls; b.textContent=text; b.onclick=onClick; return b; }

    function bookEl(book){ const el=document.createElement('div'); el.className='book'; el.draggable=true; el.dataset.bookId = book.id; el.style.background = `linear-gradient(180deg, ${shade(book.color,30)}, ${shade(book.color,-10)})`; el.addEventListener('dragstart', e=>{ e.dataTransfer.setData('text/plain', book.id); e.dataTransfer.effectAllowed='move'; }); el.addEventListener('dblclick', ()=> editBook(book)); const spine=document.createElement('div'); spine.className='spine'; spine.textContent = book.title || 'Без названия'; const badge=document.createElement('div'); badge.className='badge'; badge.textContent = book.author? short(book.author) : (book.span||1)+'×'; el.appendChild(spine); el.appendChild(badge); return el; }

    function short(s){ return s.length>12 ? s.slice(0,12)+'…' : s; }

    function shade(hex,percent){ if(!hex) hex='#8ab4f8'; let c = hex.replace('#',''); if(c.length===3) c = c.split('').map(ch=>ch+ch).join(''); const n=parseInt(c,16); let r=(n>>16)&255,g=(n>>8)&255,b=n&255; r=clamp(Math.round(r*(100+percent)/100),0,255); g=clamp(Math.round(g*(100+percent)/100),0,255); b=clamp(Math.round(b*(100+percent)/100),0,255); return '#'+((1<<24)+(r<<16)+(g<<8)+b).toString(16).slice(1); }

    // --- Book placement helpers ---
    function placeBook(shelf,index,book){
      const span=book.span||1;
      // bounds check
      if(index < 0 || index + span > shelf.cells.length) return false;
      for(let i=0;i<span;i++){
        if(shelf.cells[index+i] !== null) return false;
      }
      shelf.cells[index] = {...book};
      for(let i=1;i<span;i++) shelf.cells[index+i] = { ghost:true, spanRef:book.id };
      return true;
    }

    function removeBookFromShelves(bookId){ state.racks.forEach(r=> r.shelves.forEach(s=>{ for(let i=0;i<s.cells.length;i++){ const c=s.cells[i]; if(!c) continue; if((c.id && c.id===bookId) || (c.spanRef && c.spanRef===bookId)) s.cells[i]=null; } })); }

    function findBookLocation(bookId){ for(const r of state.racks){ for(const s of r.shelves){ for(let i=0;i<s.cells.length;i++){ const c=s.cells[i]; if(!c) continue; if(c.id === bookId) return {rack:r,shelf:s,index:i,book:c}; } } } return null; }

    // --- Drag/drop ---
    function onDropBook(e,rack,shelf,cellIndex){ e.preventDefault(); e.currentTarget.classList.remove('highlight'); const bookId = e.dataTransfer.getData('text/plain'); if(!bookId) return; const found = findBookLocation(bookId); if(!found) return; // remove old
      removeBookFromShelves(bookId);
      const book = found.book; const span = book.span||1; // try place
      // bounds + occupancy check
      if(cellIndex < 0 || cellIndex + span > shelf.cells.length){ toast('Недостаточно места'); placeBook(found.shelf, found.index, book); render(); return; }
      for(let i=0;i<span;i++){
        if(shelf.cells[cellIndex+i] !== null){ toast('Недостаточно места'); // restore old
          placeBook(found.shelf, found.index, book); render(); return; }
      }
      placeBook(shelf, cellIndex, book); render(); }

    function editBook(book){ const t=prompt('Название на корешке:', book.title||''); if(t===null) return; book.title=t; const a=prompt('Автор (необязательно):', book.author||''); if(a===null) return; book.author=a; const c=prompt('HEX цвет (напр. #8ab4f8):', book.color||'#8ab4f8'); if(!c) return; book.color=c; render(); }

    // --- Pan & Zoom ---
    function applyPanZoom(){ stage.style.transform = `translate(${state.pan.x}px, ${state.pan.y}px) scale(${state.zoom})`; }
    zoomRange.addEventListener('input', ()=>{ state.zoom = +zoomRange.value/100; zoomVal.textContent = Math.round(state.zoom*100)+'%'; applyPanZoom(); });
    stage.addEventListener('wheel', (e)=>{ e.preventDefault(); const prev=state.zoom; const delta = -Math.sign(e.deltaY)*0.05; state.zoom = clamp(state.zoom + delta, 0.2, 2); zoomRange.value = Math.round(state.zoom*100); zoomVal.textContent = Math.round(state.zoom*100)+'%'; const rect = stage.getBoundingClientRect(); const dx = (e.clientX - rect.left - state.pan.x) / prev; const dy = (e.clientY - rect.top - state.pan.y) / prev; state.pan.x = e.clientX - rect.left - dx*state.zoom; state.pan.y = e.clientY - rect.top - dy*state.zoom; applyPanZoom(); }, {passive:false});
    let isPanning=false, panStart={x:0,y:0}, stageStart={x:0,y:0}; window.addEventListener('keydown',(e)=>{ if(e.code==='Space'){ isPanning=true; document.body.style.cursor='grab'; }}); window.addEventListener('keyup',(e)=>{ if(e.code==='Space'){ isPanning=false; document.body.style.cursor=''; }});
    stage.addEventListener('mousedown',(e)=>{ if(!isPanning) return; panStart={x:e.clientX,y:e.clientY}; stageStart={...state.pan}; document.body.style.cursor='grabbing'; });
    window.addEventListener('mousemove',(e)=>{ if(!isPanning || panStart.x===0) return; state.pan.x = stageStart.x + (e.clientX - panStart.x); state.pan.y = stageStart.y + (e.clientY - panStart.y); applyPanZoom(); });
    window.addEventListener('mouseup',()=>{ panStart={x:0,y:0}; document.body.style.cursor=isPanning?'grab':''; });

    // --- UI bindings ---
    document.getElementById('newRackBtn').onclick = newRack;
    document.getElementById('addShelfTop').onclick = ()=>{ const r=currentRack(); if(r) addShelf(r.id, parseInt(qs('#shelfRows').value)||1, 'top'); };
    document.getElementById('addShelfBottom').onclick = ()=>{ const r=currentRack(); if(r) addShelf(r.id, parseInt(qs('#shelfRows').value)||1, 'bottom'); };
    document.getElementById('addShelfLeft').onclick = ()=>{ const r=currentRack(); const cs=currentShelf(); if(r) addShelf(r.id, parseInt(qs('#shelfRows').value)||1, 'before', cs? cs.id : null); };
    document.getElementById('addShelfRight').onclick = ()=>{ const r=currentRack(); const cs=currentShelf(); if(r) addShelf(r.id, parseInt(qs('#shelfRows').value)||1, 'after', cs? cs.id : null); };
    document.getElementById('addBookBtn').onclick = ()=>{ const cs=currentShelf(); if(!cs) return; const span = clamp(parseInt(qs('#bookSpan').value)||1,1,3); const idx = cs.cells.findIndex(c=>c===null); if(idx===-1){ toast('Нет свободной клетки'); return; } const book={ id:uid(), title:'Новая книга', author:'', color:'#8ab4f8', span }; placeBook(cs, idx, book); render(); };

    document.getElementById('createBookBtn').onclick = ()=>{ const title=qs('#bookTitle').value.trim(); const author=qs('#bookAuthor').value.trim(); const color=qs('#bookColor').value||'#9ae6b4'; const span=clamp(parseInt(qs('#bookWidth').value)||1,1,3); const cs=currentShelf(); if(!cs) return; const idx = cs.cells.findIndex(c=>c===null); if(idx===-1){ toast('Нет свободной клетки'); return; } const book={ id:uid(), title, author, color, span }; if(placeBook(cs, idx, book)) render(); };

    document.getElementById('applyGrid').onclick = ()=>{ const rack=currentRack(); if(!rack) return; const cols = Math.max(1, Math.min(20, parseInt(qs('#rackCols').value||rack.cols))); rack.cols = cols; rack.shelves.forEach(s=>{ const rows = Math.max(1, Math.min(3, parseInt(qs('#shelfRows').value||s.rows))); const newCells = Array(cols*rows).fill(null); // reflow: keep non-ghost books
      let idx=0; s.cells.forEach(c=>{ if(c && !c.ghost){ if(idx<newCells.length){ newCells[idx] = c; idx += c.span||1; } } }); s.rows = rows; s.cells = newCells; }); render(); };

    document.getElementById('centerRack').onclick = ()=>{ state.pan={x:0,y:0}; state.zoom=1; zoomRange.value=100; zoomVal.textContent='100%'; applyPanZoom(); };
    document.getElementById('saveBtn').onclick = ()=>{ localStorage.setItem('shelf-app', JSON.stringify(state)); toast('Сохранено'); };
    document.getElementById('loadBtn').onclick = ()=>{ const raw=localStorage.getItem('shelf-app'); if(!raw){ toast('Нет сохранений'); return;} try{ const obj=JSON.parse(raw); Object.assign(state, obj); render(); toast('Загружено'); }catch(e){ alert('Не удалось загрузить'); } };
    document.getElementById('resetBtn').onclick = ()=>{ if(confirm('Сбросить всё?')){ localStorage.removeItem('shelf-app'); location.reload(); } };

    function switchRack(dir){ const i = state.racks.findIndex(r=>r.id===state.selection.rackId); if(i===-1) return; const j=(i+dir+state.racks.length)%state.racks.length; state.selection.rackId = state.racks[j].id; state.selection.shelfId = state.racks[j].shelves[0]?.id||null; render(); }
    function switchShelf(dir){ const rack=currentRack(); if(!rack) return; const i=rack.shelves.findIndex(s=>s.id===state.selection.shelfId); const j = clamp(i+dir,0,rack.shelves.length-1); state.selection.shelfId = rack.shelves[j]?.id || rack.shelves[0]?.id; render(); }

    // --- Shelf editing from sidebar ---
    qs('#rackTitle').addEventListener('input', ()=>{ const r=currentRack(); if(!r) return; r.title = qs('#rackTitle').value; const headerInput = stage.querySelector('.rack-header input.grow'); if(headerInput) headerInput.value = r.title; });

    qs('#shelfRows').addEventListener('change', ()=>{ const cs=currentShelf(); const rack=currentRack(); if(!cs || !rack) return; cs.rows = clamp(parseInt(qs('#shelfRows').value)||1,1,3); const cols = rack.cols; const newCells = Array(cols*cs.rows).fill(null); let idx=0; cs.cells.forEach(c=>{ if(c && !c.ghost && idx<newCells.length){ newCells[idx] = c; idx += c.span||1; }}); cs.cells = newCells; render(); });

    qs('#applyShelfLabel').onclick = ()=>{ const s=currentShelf(); if(!s) return; s.label = qs('#shelfLabelText').value; render(); };
    qs('#deleteShelfBtn').onclick = ()=>{ const r=currentRack(); const s=currentShelf(); if(!r||!s) return; if(confirm('Удалить полку?')) removeShelf(r.id,s.id); };

    // --- Photo upload and auto-recognition ---
    const dz = document.getElementById('dropzone'); const fileInput = document.getElementById('fileInput'); dz.addEventListener('click', ()=> fileInput.click()); dz.addEventListener('dragover', e=>{ e.preventDefault(); dz.style.borderColor='var(--accent)'; }); dz.addEventListener('dragleave', ()=> dz.style.borderColor='#334155'); dz.addEventListener('drop', e=>{ e.preventDefault(); dz.style.borderColor='#334155'; if(e.dataTransfer.files[0]) handleFile(e.dataTransfer.files[0]); }); fileInput.addEventListener('change', e=>{ if(e.target.files[0]) handleFile(e.target.files[0]); });

    function handleFile(file){ const reader=new FileReader(); reader.onload = ()=>{ const s = currentShelf(); if(!s) return; s.bg = reader.result; render(); detectBooksFromImage(reader.result, s, currentRack().cols).then(books=>{ if(books && books.length){ // clear shelf cells
            s.cells = Array(currentRack().cols * s.rows).fill(null);
            // map book widths to spans and place sequentially
            let cursor=0; books.forEach(b=>{ if(cursor>=s.cells.length) return; const span = clamp(b.span||1,1,3); if(placeBook(s,cursor,{ id:uid(), title:b.title||'Фото-книга', author:'', color:b.color||'#c2b280', span })) cursor += span; }); render(); toast('Распознано и размещено книг: '+books.length); } else { toast('Не удалось распознать книги, фото добавлено как фон'); } }); }; reader.readAsDataURL(file); }

    document.getElementById('photoOpacity').addEventListener('input',(e)=>{ const s=currentShelf(); if(!s) return; s.opacity = (+e.target.value)/100; render(); });
    document.getElementById('clearPhoto').onclick = ()=>{ const s=currentShelf(); if(!s) return; s.bg=null; render(); };

    // --- Simple image processing to detect vertical separators (naive) ---
    async function detectBooksFromImage(dataUrl, shelf, cols){ return new Promise((resolve)=>{
      const img = new Image(); img.onload = ()=>{
        // draw smaller for speed
        const W = Math.min(900, img.width);
        const H = Math.round(img.height * (W/img.width));
        const c = document.createElement('canvas'); c.width = W; c.height = H; const ctx = c.getContext('2d'); ctx.drawImage(img,0,0,W,H);
        try{
          const id = ctx.getImageData(0,0,W,H).data;
          const colScores = new Float32Array(W);
          for(let x=1;x<W;x++){
            let ssum=0;
            for(let y=0;y<H;y++){
              const i=(y*W+x)*4; const iL=(y*W+(x-1))*4;
              const lum = 0.2126*id[i]+0.7152*id[i+1]+0.0722*id[i+2];
              const lumL = 0.2126*id[iL]+0.7152*id[iL+1]+0.0722*id[iL+2];
              ssum += Math.abs(lum - lumL);
            }
            colScores[x]=ssum/ H;
          }
          // smooth
          const smooth = new Float32Array(W);
          const k = 9; const half = Math.floor(k/2);
          for(let x=0;x<W;x++){ let ssum=0, cnt=0; for(let j=-half;j<=half;j++){ const xi = clamp(x+j,0,W-1); ssum += colScores[xi]; cnt++; } smooth[x] = ssum/cnt; }

          // find peaks
          let mean=0; for(let x=0;x<W;x++) mean += smooth[x]; mean /= W;
          let thresh = Math.max(0.8, mean*1.4);
          const peaks = [];
          for(let x=2;x<W-2;x++){
            if(smooth[x] > thresh && smooth[x] >= smooth[x-1] && smooth[x] >= smooth[x+1]) peaks.push(x);
          }
          // reduce nearby peaks
          const filtered = [];
          peaks.forEach(p=>{ if(!filtered.length || p - filtered[filtered.length-1] > 12) filtered.push(p); });
          // boundaries include 0 and W
          const bounds = [0, ...filtered, W];
          const segments = [];
          for(let i=0;i<bounds.length-1;i++){ const left=bounds[i], right=bounds[i+1]; const width = right-left; if(width>10) segments.push({left,right,width}); }

          // if segments seems too many (like > cols*2) fallback to splitting by cols
          let books = [];
          if(segments.length>=1 && segments.length<=cols*2){
            // map segments to spans proportionally to cols
            const totalW = segments.reduce((a,b)=>a+b.width,0);
            let remainingCols = cols;
            segments.forEach((seg, idx)=>{
              let span = Math.max(1, Math.round(cols * (seg.width / totalW)));
              // keep last segment adjusted
              if(idx === segments.length-1) span = remainingCols;
              else remainingCols -= span;
              books.push({ span });
            });
            // ensure at least 1 book
            if(books.length===0) books.push({span:cols});
          } else {
            // fallback: evenly split into cols (every book 1 span)
            books = Array.from({length:Math.min(cols, 12)}, ()=>({span:1}));
          }

          resolve(books);
        }catch(err){ console.error(err); resolve(null); }
      };
      img.onerror = ()=> resolve(null);
      img.src = dataUrl;
    }); }

    // --- Export / Import ---
    document.getElementById('exportJson').onclick = ()=>{ const data = JSON.stringify(state, null, 2); const blob = new Blob([data], {type:'application/json'}); const url = URL.createObjectURL(blob); const a=document.createElement('a'); a.href=url; a.download='shelf-layout.json'; a.click(); URL.revokeObjectURL(url); };
    document.getElementById('importJson').onclick = ()=> qs('#importFile').click();
    document.getElementById('importFile').addEventListener('change', (e)=>{ const file = e.target.files[0]; if(!file) return; const reader=new FileReader(); reader.onload = ()=>{ try{ const obj=JSON.parse(reader.result); Object.assign(state, obj); render(); toast('Импортировано'); }catch(err){ alert('Неверный JSON'); } }; reader.readAsText(file); });

    // --- Init ---
    (function init(){ const saved = localStorage.getItem('shelf-app'); if(saved){ try{ const obj = JSON.parse(saved); Object.assign(state, obj); }catch{} } if(state.racks.length===0){ newRack(); } else { render(); } })();
  </script>
</body>
</html>
