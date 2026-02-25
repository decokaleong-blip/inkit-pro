<!DOCTYPE html>
<html lang="zh-HK">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover" />
<title>stocktracer — 股票交易紀錄 (測試版)</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
<style>
:root{
  --bg:#071026;
  --card:#081227;
  --muted:#9aa4b2;
  --accent:#06b6d4;
  --accent-2:#7c3aed;
  --glass: rgba(255,255,255,0.03);
  --success:#16a34a;
  --danger:#ef4444;
}
*{box-sizing:border-box}
html,body{height:100%}
body{
  margin:0;
  font-family:Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  background: linear-gradient(180deg, #041022 0%, #071026 60%);
  color:#e6eef8;
  -webkit-font-smoothing:antialiased;
  -moz-osx-font-smoothing:grayscale;
  padding:18px;
}
/* container */
.wrapper{max-width:980px;margin:0 auto}
.header{
  display:flex;align-items:center;gap:12px;margin-bottom:12px;
}
.logo{width:46px;height:46px;border-radius:10px;background:linear-gradient(135deg,var(--accent),var(--accent-2));display:flex;align-items:center;justify-content:center;font-weight:700;box-shadow:0 6px 18px rgba(7,10,20,0.6)}
.title-block{flex:1}
.title{font-size:18px;font-weight:700}
.subtitle{color:var(--muted);font-size:13px;margin-top:3px}

/* cards */
.card{
  background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
  border:1px solid rgba(255,255,255,0.03);
  padding:14px;
  border-radius:12px;
  box-shadow: 0 6px 30px rgba(2,6,23,0.6);
  margin-bottom:12px;
}
.row{display:flex;gap:8px;align-items:center}
.col{display:flex;flex-direction:column;gap:8px}

/* form */
.form-grid{display:grid;grid-template-columns:repeat(2,1fr);gap:8px}
@media (max-width:640px){
  .form-grid{grid-template-columns:1fr}
}
label{font-size:12px;color:var(--muted);margin-bottom:6px}
input[type="text"], input[type="number"], input[type="date"], select, textarea{
  background:var(--glass);
  border:1px solid rgba(255,255,255,0.04);
  color:var(--muted);
  padding:10px 12px;border-radius:8px;
  outline:none;font-size:14px;
  width:100%;
}
input:focus, select:focus, textarea:focus{
  box-shadow:0 6px 18px rgba(7,10,20,0.6), 0 0 0 4px rgba(6,182,212,0.06);
  border-color:rgba(6,182,212,0.18);
  color:#e9f8fb;
}
.actions{display:flex;gap:8px;flex-wrap:wrap}
.button{
  background:linear-gradient(90deg,var(--accent),var(--accent-2));
  color:#051220;padding:10px 14px;border:none;border-radius:10px;font-weight:700;
  cursor:pointer;
}
.ghost{background:transparent;border:1px solid rgba(255,255,255,0.04);color:var(--muted);padding:9px 12px;border-radius:10px;cursor:pointer}

/* table */
.table-wrap{overflow:auto;max-height:320px;margin-top:12px;}
.table{width:100%;border-collapse:collapse;min-width:680px}
.table th{position:sticky;top:0;background:rgba(6,10,20,0.6);backdrop-filter: blur(6px);z-index:2;color:var(--muted);font-size:13px;padding:10px;text-align:left}
.table td{padding:10px;border-top:1px solid rgba(255,255,255,0.02);font-size:13px;color:#dbeafe}
.row-actions{display:flex;gap:8px}

/* summary */
.summary-grid{display:grid;grid-template-columns:1fr 1fr;gap:8px}
@media (max-width:640px){
  .summary-grid{grid-template-columns:1fr}
}
.hold-card{padding:10px;border-radius:10px;background:linear-gradient(180deg, rgba(255,255,255,0.015), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.02);display:flex;justify-content:space-between;align-items:center}
.ticker{font-weight:700}
.small{font-size:13px;color:var(--muted)}
.badge-positive{color:var(--success)}
.badge-negative{color:var(--danger)}

/* footer */
.footer{display:flex;justify-content:space-between;align-items:center;gap:12px;padding:10px;border-radius:10px}
@media (max-width:640px){
  .footer{flex-direction:column;align-items:flex-start;gap:6px}
}

/* utility */
.center{display:flex;align-items:center;justify-content:center}
.note{font-size:13px;color:var(--muted);margin-top:8px}
.input-inline{display:flex;gap:8px}
.small-btn{padding:8px 10px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:var(--muted);cursor:pointer}
</style>
</head>
<body>
<div class="wrapper">
  <div class="header">
    <div class="logo">ST</div>
    <div class="title-block">
      <div class="title">stocktracer</div>
      <div class="subtitle">股票交易紀錄 • 測試版</div>
    </div>
    <div style="display:flex;gap:8px;align-items:center">
      <button id="exportBtn" class="button" style="padding:8px 10px;font-size:13px">匯出</button>
      <label class="ghost" style="display:inline-flex;align-items:center;gap:8px;cursor:pointer;font-size:13px;padding:8px 10px">
        匯入
        <input id="importFile" type="file" accept=".csv" style="display:none">
      </label>
      <button id="clearAll" class="ghost" title="清除所有資料" style="font-size:13px;padding:8px 10px">清除</button>
    </div>
  </div>

  <div class="card">
    <div style="display:flex;justify-content:space-between;align-items:center;gap:8px;margin-bottom:8px">
      <div style="font-weight:700">新增 / 修改 交易</div>
      <div class="small">請填寫正確資料，手機上建議使用日期選擇器</div>
    </div>
    <div class="form-grid">
      <div>
        <label>交易日期</label>
        <input id="tradeDate" type="date">
      </div>
      <div>
        <label>股票代號 (4 位數)</label>
        <input id="ticker" type="text" placeholder="0001">
      </div>
      <div>
        <label>買 / 賣</label>
        <select id="side"><option value="BUY">買入</option><option value="SELL">賣出</option></select>
      </div>
      <div>
        <label>股數</label>
        <input id="qty" type="number" min="1" step="1" value="100">
      </div>
      <div>
        <label>成交價 (每股)</label>
        <input id="price" type="number" min="0" step="0.0001" value="1.00">
      </div>
      <div>
        <label>手續費</label>
        <input id="fee" type="number" min="0" step="0.01" value="0.00">
      </div>
      <div style="grid-column:1 / -1">
        <label>備註（選填）</label>
        <input id="note" type="text" placeholder="例如：長線 / 日內交易">
      </div>
    </div>

    <div style="display:flex;justify-content:space-between;align-items:center;margin-top:10px;gap:8px;flex-wrap:wrap">
      <div class="actions">
        <button id="addBtn" class="button">新增紀錄</button>
        <button id="updateBtn" class="button" style="display:none;background:linear-gradient(90deg,#f59e0b,#ef4444)">儲存修改</button>
        <button id="cancelEdit" class="ghost" style="display:none">取消</button>
      </div>
      <div class="small">總記錄: <span id="totalCount">0</span> • 持股類別: <span id="tickerCount">0</span></div>
    </div>

    <div class="note">股票代號只接受 4 位數字（如 0001）。所有資料存於本機，請定期匯出備份。</div>

    <div class="table-wrap" style="margin-top:12px">
      <table class="table" id="tradesTable">
        <thead>
          <tr>
            <th>日期</th><th>代號</th><th>買/賣</th><th>股數</th><th>價格</th><th>手續費</th><th>總額</th><th style="text-align:right">操作</th>
          </tr>
        </thead>
        <tbody id="tradesBody"></tbody>
      </table>
    </div>
  </div>

  <div class="card">
    <div style="display:flex;justify-content:space-between;align-items:center;gap:8px;margin-bottom:8px">
      <div style="font-weight:700">持股總覽</div>
      <div class="small">可手動設定市價以計算未實現盈虧</div>
    </div>

    <div style="display:flex;gap:8px;flex-wrap:wrap;margin-bottom:8px">
      <input id="marketPriceTicker" type="text" placeholder="0001" style="flex:1;padding:10px;border-radius:8px;background:var(--glass);border:1px solid rgba(255,255,255,0.03);color:var(--muted)">
      <input id="marketPrice" type="number" min="0" step="0.0001" placeholder="最新價格" style="width:140px;padding:10px;border-radius:8px;background:var(--glass);border:1px solid rgba(255,255,255,0.03);color:var(--muted)">
      <button id="setMarket" class="button" style="padding:9px 12px">更新價格</button>
    </div>

    <div class="summary-grid" id="summaryList"></div>

    <div class="note" style="margin-top:10px">備註：未實現盈虧以手動輸入之「最新價格」計算。自動即時報價需串接 API（需後端）。</div>
  </div>

  <div class="card footer">
    <div>
      <div style="font-weight:700">測試版 • stocktracer</div>
      <div class="small">開發提示：可匯出/匯入 CSV，共享單檔即可讓他人測試。</div>
    </div>
    <div style="text-align:right">
      <div class="small">創作者：黃延杰</div>
    </div>
  </div>
</div>

<script>
(function(){
  const qs = (s, el=document) => el.querySelector(s);
  const qsa = (s, el=document) => Array.from(el.querySelectorAll(s));
  const LS_KEY = 'stocktracer_trades_v1';

  // elements
  const tradeDate = qs('#tradeDate');
  const ticker = qs('#ticker');
  const side = qs('#side');
  const qty = qs('#qty');
  const price = qs('#price');
  const fee = qs('#fee');
  const note = qs('#note');
  const addBtn = qs('#addBtn');
  const tradesBody = qs('#tradesBody');
  const exportBtn = qs('#exportBtn');
  const importFile = qs('#importFile');
  const clearAll = qs('#clearAll');
  const summaryList = qs('#summaryList');
  const totalCount = qs('#totalCount');
  const tickerCount = qs('#tickerCount');
  const marketPriceTicker = qs('#marketPriceTicker');
  const marketPrice = qs('#marketPrice');
  const setMarket = qs('#setMarket');
  const updateBtn = qs('#updateBtn');
  const cancelEdit = qs('#cancelEdit');

  let trades = [];
  let editingId = null;

  function uid(){ return 't_'+Math.random().toString(36).slice(2,9); }
  function nowDate(){ return new Date().toISOString().slice(0,10); }
  function save(){ localStorage.setItem(LS_KEY, JSON.stringify(trades)); render(); }
  function load(){
    const raw = localStorage.getItem(LS_KEY);
    if(raw) {
      try { trades = JSON.parse(raw) || []; } catch(e){ trades = []; }
    } else {
      trades = [];
    }
  }

  function formatNum(n){
    if(n === null || n === undefined || isNaN(n)) return '-';
    return Number(n).toLocaleString(undefined, {minimumFractionDigits:2, maximumFractionDigits:4});
  }

  function rowTotal(r){ return (r.qty * r.price) + (Number(r.fee) || 0); }

  function render(){
    tradesBody.innerHTML = '';
    trades.forEach(r=>{
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${r.date}</td>
        <td>${r.ticker}</td>
        <td>${r.side === 'BUY' ? '買入' : '賣出'}</td>
        <td>${r.qty}</td>
        <td>${formatNum(r.price)}</td>
        <td>${formatNum(r.fee)}</td>
        <td>${formatNum(rowTotal(r))}</td>
        <td style="text-align:right">
          <div class="row-actions">
            <button class="ghost edit" data-id="${r.id}">編輯</button>
            <button class="ghost del" data-id="${r.id}">刪除</button>
          </div>
        </td>
      `;
      tradesBody.appendChild(tr);
    });

    qsa('.edit').forEach(btn=> btn.addEventListener('click', ()=> startEdit(btn.dataset.id)));
    qsa('.del').forEach(btn=> btn.addEventListener('click', ()=> {
      if(!confirm('確定要刪除此筆紀錄？')) return;
      trades = trades.filter(t => t.id !== btn.dataset.id);
      save();
    }));

    // summary
    const map = {};
    trades.forEach(r=>{
      if(!map[r.ticker]) map[r.ticker] = {ticker:r.ticker, qty:0, cost:0};
      const m = map[r.ticker];
      if(r.side === 'BUY'){
        m.qty += Number(r.qty);
        m.cost += (Number(r.qty) * Number(r.price)) + (Number(r.fee) || 0);
      } else {
        m.qty -= Number(r.qty);
        const avg = m.qty > 0 ? (m.cost / m.qty) : 0;
        m.cost -= avg * Number(r.qty);
      }
    });
    const holdings = Object.values(map).filter(h => h.qty !== 0);

    summaryList.innerHTML = '';
    holdings.forEach(h=>{
      const avgCost = h.qty !== 0 ? (h.cost / h.qty) : 0;
      const mpKey = 'MP_stocktracer_'+h.ticker;
      const mp = localStorage.getItem(mpKey);
      const lastPrice = mp ? Number(mp) : null;
      const unreal = lastPrice !== null ? ( (lastPrice - avgCost) * h.qty ) : null;

      const div = document.createElement('div');
      div.className = 'hold-card';
      div.innerHTML = `
        <div>
          <div class="ticker">${h.ticker} <span class="small" style="margin-left:8px">股數 ${h.qty}</span></div>
          <div class="small" style="margin-top:6px">平均成本: <strong style="margin-left:6px">${formatNum(avgCost)}</strong></div>
          <div class="small" style="margin-top:4px">市價: <strong style="margin-left:6px">${lastPrice !== null ? formatNum(lastPrice) : '-'}</strong></div>
          <div class="small" style="margin-top:4px">未實現: <strong style="margin-left:6px">${unreal !== null ? (unreal>=0 ? '<span class="badge-positive">'+formatNum(unreal)+'</span>' : '<span class="badge-negative">'+formatNum(unreal)+'</span>') : '-'}</strong></div>
        </div>
        <div style="display:flex;flex-direction:column;gap:8px;align-items:flex-end">
          <div style="display:flex;gap:6px">
            <button class="small-btn btn-set-price" data-t="${h.ticker}">設市價</button>
            <button class="small-btn btn-view" data-t="${h.ticker}">查看</button>
          </div>
        </div>
      `;
      summaryList.appendChild(div);
    });

    totalCount.textContent = trades.length;
    tickerCount.textContent = Object.keys(map).length;
  }

  function validateForm(){
    const tk = (ticker.value || '').trim();
    if(!tradeDate.value){ alert('請輸入交易日期'); return false; }
    if(!/^\d{4}$/.test(tk)){ alert('股票代號請輸入 4 位數字，例如 0001'); return false; }
    if(!qty.value || Number(qty.value) <= 0){ alert('請輸入股數'); return false; }
    if(!price.value || Number(price.value) < 0){ alert('請輸入正確成交價'); return false; }
    return true;
  }

  function clearForm(){
    tradeDate.value = nowDate();
    ticker.value = '';
    side.value = 'BUY';
    qty.value = 100;
    price.value = 1.00;
    fee.value = 0.00;
    note.value = '';
    editingId = null;
    updateBtn.style.display = 'none';
    cancelEdit.style.display = 'none';
    addBtn.style.display = 'inline-block';
  }

  function addTrade(){
    if(!validateForm()) return;
    const r = {
      id: uid(),
      date: tradeDate.value,
      ticker: ticker.value.trim(),
      side: side.value,
      qty: Number(qty.value),
      price: Number(price.value),
      fee: Number(fee.value) || 0,
      note: note.value || ''
    };
    trades.push(r);
    save();
    clearForm();
  }

  function startEdit(id){
    const r = trades.find(t=>t.id === id);
    if(!r) return;
    editingId = id;
    tradeDate.value = r.date;
    ticker.value = r.ticker;
    side.value = r.side;
    qty.value = r.qty;
    price.value = r.price;
    fee.value = r.fee;
    note.value = r.note || '';
    addBtn.style.display = 'none';
    updateBtn.style.display = 'inline-block';
    cancelEdit.style.display = 'inline-block';
  }

  function applyEdit(){
    if(!editingId) return;
    if(!validateForm()) return;
    const idx = trades.findIndex(t=>t.id === editingId);
    if(idx === -1) return;
    trades[idx] = {
      id: editingId,
      date: tradeDate.value,
      ticker: ticker.value.trim(),
      side: side.value,
      qty: Number(qty.value),
      price: Number(price.value),
      fee: Number(fee.value) || 0,
      note: note.value || ''
    };
    save();
    clearForm();
  }

  function toCSV(){
    const hdr = ['id','date','ticker','side','qty','price','fee','note'];
    const rows = trades.map(r => hdr.map(h=> {
      const v = r[h];
      if(v === null || v === undefined) return '';
      return '"'+String(v).replace(/"/g,'""')+'"';
    }).join(','));
    return [hdr.join(','), ...rows].join('\\n');
  }

  function exportCSV(){
    const csv = toCSV();
    const blob = new Blob([csv], {type:'text/csv;charset=utf-8;'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'stocktracer_trades.csv';
    document.body.appendChild(a);
    a.click();
    a.remove();
    URL.revokeObjectURL(url);
  }

  function importCSVFile(file){
    const reader = new FileReader();
    reader.onload = function(e){
      const txt = e.target.result;
      parseCSV(txt);
    };
    reader.readAsText(file, 'utf-8');
  }

  function parseCSV(txt){
    const lines = txt.split(/\\r?\\n/).filter(l=>l.trim()!=='');
    if(lines.length < 2){ alert('CSV 內容不足'); return; }
    const headers = lines[0].split(',').map(h=>h.trim());
    const arr = [];
    for(let i=1;i<lines.length;i++){
      const cols = lines[i].split(',');
      if(cols.length < headers.length) continue;
      const obj = {};
      headers.forEach((h, idx)=> obj[h] = cols[idx] ? cols[idx].replace(/^"|"$/g,'').replace(/""/g,'"') : '');
      arr.push({
        id: obj.id || uid(),
        date: obj.date || nowDate(),
        ticker: (obj.ticker || '').padStart(4,'0'),
        side: (obj.side || 'BUY'),
        qty: Number(obj.qty) || 0,
        price: Number(obj.price) || 0,
        fee: Number(obj.fee) || 0,
        note: obj.note || ''
      });
    }
    if(!confirm('匯入將會加入現有紀錄。確定要繼續？')) return;
    trades = trades.concat(arr);
    save();
  }

  function setMarketPrice(){
    const tk = (marketPriceTicker.value || '').trim();
    const mp = marketPrice.value;
    if(!/^\d{4}$/.test(tk)){ alert('請輸入 4 位數字的股票代號'); return; }
    if(mp === '' || mp === null || isNaN(Number(mp))){ alert('請輸入有效價格'); return; }
    localStorage.setItem('MP_stocktracer_'+tk, String(Number(mp)));
    render();
    marketPriceTicker.value = '';
    marketPrice.value = '';
  }

  function viewRecordsOf(tk){
    const row = Array.from(tradesBody.querySelectorAll('tr')).find(tr => tr.children[1].textContent === tk);
    if(row){
      row.scrollIntoView({behavior:'smooth', block:'center'});
      row.style.transition = 'background 0.6s';
      const orig = row.style.background;
      row.style.background = 'linear-gradient(90deg, rgba(124,58,237,0.12), rgba(6,182,212,0.08))';
      setTimeout(()=> row.style.background = orig, 1200);
    } else {
      alert('找不到此代號的紀錄');
    }
  }

  function init(){
    load();
    render();
    tradeDate.value = nowDate();

    addBtn.addEventListener('click', addTrade);
    updateBtn.addEventListener('click', applyEdit);
    cancelEdit.addEventListener('click', clearForm);
    exportBtn.addEventListener('click', exportCSV);
    importFile.addEventListener('change', (e)=> {
      const f = e.target.files[0];
      if(f) importCSVFile(f);
      e.target.value = '';
    });
    clearAll.addEventListener('click', ()=> {
      if(!confirm('確定要清除所有本地資料？此動作無法復原')) return;
      trades = [];
      Object.keys(localStorage).forEach(k=> { if(k.startsWith('MP_stocktracer_')) localStorage.removeItem(k); });
      save();
    });
    setMarket.addEventListener('click', setMarketPrice);

    summaryList.addEventListener('click', (e)=>{
      const el = e.target;
      if(el.classList.contains('btn-set-price')){
        const tk = el.dataset.t;
        const mp = prompt('輸入 ' + tk + ' 的最新價格（手動）');
        if(mp !== null && mp !== ''){
          if(isNaN(Number(mp))){ alert('價格輸入錯誤'); return; }
          localStorage.setItem('MP_stocktracer_'+tk, String(Number(mp)));
          render();
        }
      } else if(el.classList.contains('btn-view')){
        viewRecordsOf(el.dataset.t);
      }
    });
  }

  init();
})();
</script>
</body>
</html>
