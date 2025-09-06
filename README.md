# FQedu-test
<html lang="zh-Hant">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>小考（五市場 ±｜滿分 20 分）</title>
<style>
  :root{
    --bg:#f6f7fb; --card:#ffffff; --muted:#6b7280; --text:#111827;
    --primary:#2563eb; --primary-600:#1d4ed8; --border:#e5e7eb; --chip:#f9fafb;
    --ok:#16a34a; --ok-bg:#ecfdf5; --err:#dc2626; --err-bg:#fef2f2;
  }
  *{box-sizing:border-box}
  body{margin:0; font-family:system-ui,-apple-system,"Segoe UI",Roboto,Noto Sans TC,Arial,"PingFang TC","Microsoft JhengHei",sans-serif; background:var(--bg); color:var(--text);}  
  .wrap{max-width:920px; margin:36px auto; padding:0 16px}
  .card{background:var(--card); border:1px solid var(--border); border-radius:16px; box-shadow:0 8px 24px rgba(0,0,0,.05)}
  .header{display:flex; align-items:center; justify-content:space-between; padding:18px 20px; gap:16px}
  .title{font-size:18px; font-weight:700}
  .crumb{font-size:13px; color:var(--muted)}
  .content{padding:20px}
  .row{display:flex; gap:12px; flex-wrap:wrap}
  label.lbl{font-size:14px; color:#374151; display:block; margin-bottom:6px}
  input[type="text"]{width:100%; padding:10px 12px; border:1px solid var(--border); border-radius:10px; font-size:16px; outline:none; background:#fff}
  .qcard{padding:14px 16px; border:1px solid var(--border); border-radius:12px; background:#f8fafc; line-height:1.8}
  .sec{margin-top:16px}
  .sec h4{margin:8px 0; font-size:14px; color:#374151}

  /* 膠囊按鈕 */
  .pill{display:inline-flex; align-items:center; gap:8px; border:1px solid var(--border); background:var(--chip); border-radius:999px; padding:10px 14px; cursor:pointer; user-select:none; box-shadow:0 2px 0 rgba(0,0,0,.03); margin-right:10px; margin-top:8px}
  .pill .sym{opacity:.7}
  .pill.active{border-color:var(--primary); box-shadow:0 0 0 3px rgba(37,99,235,.15); background:#eef2ff}
  .pill.ok{border-color:var(--ok); background:var(--ok-bg)}
  .pill.wrong{border-color:var(--err); background:var(--err-bg)}
  .pill.key{outline:2px dashed var(--primary)}
  .market{display:flex; align-items:center; justify-content:flex-start; gap:8px; flex-wrap:wrap}

  .btns{display:flex; gap:12px; margin-top:16px}
  .btn{padding:10px 16px; border:0; border-radius:10px; font-size:16px; cursor:pointer}
  .btn.primary{background:var(--primary); color:#fff}
  .btn.primary:hover{background:var(--primary-600)}
  .btn.ghost{background:#fff; border:1px solid var(--border); color:#374151}

  .msg{margin-top:14px; padding:12px; border-radius:12px; display:none}
  .msg.ok{display:block; background:#ebf7ee; color:#0b6b2e; border:1px solid #cde9d6}
  .msg.err{display:block; background:#fdecea; color:#b00020; border:1px solid #f5c2c7}
  .scorebox{margin-top:12px; padding:12px; border-radius:12px; border:1px dashed var(--border); background:#fff}
  .muted{color:var(--muted); font-size:13px}
</style>
</head>
<body>
  <div class="wrap">
    <!-- 身分區 -->
    <div class="card" style="margin-bottom:16px;">
      <div class="header">
        <div class="title">小考（五市場 ±｜滿分 20 分）</div>
        <div class="crumb" id="crumb">尚未載入題目</div>
      </div>
      <div class="content">
        <div class="row">
          <div style="flex:1 1 240px; min-width:200px;">
            <label class="lbl">姓名</label>
            <input id="name" type="text" placeholder="請輸入姓名" />
          </div>
          <div style="flex:1 1 240px; min-width:200px;">
            <label class="lbl">學號</label>
            <input id="sid" type="text" placeholder="請輸入學號" />
          </div>
          <div style="flex:0 0 auto; align-self:flex-end; margin-bottom:2px; display:flex; gap:8px;">
            <button class="btn primary" id="btnFetch">抽題</button>
            <button class="btn ghost" id="btnReset">清空</button>
          </div>
        </div>
      </div>
    </div>

    <!-- 題目卡 -->
    <div class="card" id="qwrap" style="display:none;">
      <div class="content">
        <div class="sec">
          <div class="muted" style="margin-bottom:6px">題目內容</div>
          <div class="qcard" id="qtext"></div>
        </div>

        <div class="sec">
          <h4>股市</h4>
          <div class="market" data-k="STOCK"></div>
        </div>
        <div class="sec">
          <h4>債市</h4>
          <div class="market" data-k="BOND"></div>
        </div>
        <div class="sec">
          <h4>外匯</h4>
          <div class="market" data-k="FX"></div>
        </div>
        <div class="sec">
          <h4>商品</h4>
          <div class="market" data-k="COM"></div>
        </div>
        <div class="sec">
          <h4>房地產</h4>
          <div class="market" data-k="RE"></div>
        </div>

        <div class="btns">
          <button class="btn primary" id="btnSubmit">送出作答</button>
          <button class="btn ghost" id="btnBack">返回上一步</button>
        </div>

        <div id="msg" class="msg"></div>
        <div id="scorebox" class="scorebox" style="display:none"></div>
        <p class="muted" style="margin-top:10px">抽題與作答皆由後端 API 處理；送出後會立即回寫到試算表（Responses / Scores）。</p>
      </div>
    </div>
  </div>

<script>
// 你的 Apps Script Web App URL（務必用 /exec）
const API_URL = 'https://script.google.com/macros/s/AKfycbwGtBVaDhEI6sB1beM13ib93F4kMmPFRjn5Tvv65GNSOWW_aO0Kq_ZJESeuczXw4B4aQA/exec';
const $ = id => document.getElementById(id);
const state = { qno:'', ans:{ STOCK:'',BOND:'',FX:'',COM:'',RE:'' } };

function show(el,b){ el.style.display = b? '':'none'; }
function toast(ok,text){ const m=$('msg'); m.className='msg '+(ok?'ok':'err'); m.textContent=text||''; }
function setCrumb(){
  const nm = $('name').value.trim();
  const sid = $('sid').value.trim();
  const q = state.qno ? `第 ${state.qno} 題` : '尚未載入題目';
  $('crumb').textContent = (nm||sid) ? `${nm||'未填姓名'}｜${sid||'未填學號'}｜${q}` : q;
}
function pill(label, sym){
  const div = document.createElement('div');
  div.className = 'pill';
  div.innerHTML = `<span>${label}</span><span class=\"sym\">（${sym}）</span>`;
  return div;
}
function clearMarks(box){ box.querySelectorAll('.pill').forEach(p=>p.classList.remove('ok','wrong','key','active')); }
function mountMarkets(){
  document.querySelectorAll('.market').forEach(box=>{
    const k = box.dataset.k; // STOCK/BOND/FX/COM/RE
    box.innerHTML = '';
    const p = pill('正面','＋'), n = pill('負面','－');
    p.addEventListener('click', ()=>{ select(k,'+'); clearMarks(box); p.classList.add('active'); });
    n.addEventListener('click', ()=>{ select(k,'-'); clearMarks(box); n.classList.add('active'); });
    box.appendChild(p); box.appendChild(n);
  });
}
function select(k, v){ state.ans[k] = v; }

async function callApi(action,payload){
  const res = await fetch(API_URL,{ method:'POST', body: JSON.stringify({action, payload}) });
  const j = await res.json();
  if(!j.ok) throw new Error(j.message || 'API error');
  return j;
}

// 僅呼叫後端的 requestRandom（取消 911 範例題 fallback）
$('btnFetch').addEventListener('click', async ()=>{
  toast(true,''); show($('qwrap'), false); $('scorebox').style.display='none';
  const name=$('name').value.trim(), sid=$('sid').value.trim();
  if(!name || !sid){ toast(false,'請先輸入「姓名、學號」'); return; }
  try{
    const r = await callApi('requestRandom', { studentId: sid });
    // 後端需回 question: { qno, text }
    if(!r.question || !r.question.qno){ throw new Error('後端未回傳題目'); }
    document.getElementById('qtext').textContent = r.question.text ? `Q${r.question.qno}｜${r.question.text}` : `Q${r.question.qno}`;
    state.qno = String(r.question.qno);
    mountMarkets();
    show(document.getElementById('qwrap'), true); setCrumb();
  }catch(err){ toast(false, String(err.message||err)); }
});

// 送出作答：必填五市場；後端寫入試算表並回傳 score / correctCount /（可選）ans
$('btnSubmit').addEventListener('click', async ()=>{
  toast(true,'');
  const name=$('name').value.trim(); const sid=$('sid').value.trim();
  const qno=state.qno;
  const {STOCK,BOND,FX,COM,RE} = state.ans;
  if(!name || !sid){ toast(false,'請先輸入「姓名、學號」'); return; }
  if(!qno){ toast(false,'請先抽題'); return; }
  if(!(STOCK && BOND && FX && COM && RE)){ toast(false,'請完成五個市場的 ± 選擇'); return; }
  try{
    const r = await callApi('submitAnswer', { name, studentId:sid, qno, stock:STOCK, bond:BOND, fx:FX, com:COM, re:RE });
    if(r.score == null){
      toast(true, '已送出作答（後端未回傳分數）');
    }else{
      toast(true, '作答完成');
      // 顯示分數
      const box = document.getElementById('scorebox');
      box.innerHTML = `<strong>本題得分：</strong>${Number(r.score ?? 0)}/20　｜　<strong>答對數：</strong>${Number(r.correctCount ?? 0)}/5`;
      box.style.display = '';
      // 若後端回傳標準答案 ans:{STOCK:'+',...}，就標記每市場對/錯與正解
      if(r.ans){
        document.querySelectorAll('.market').forEach(box=>{
          const k = box.dataset.k; const correct = r.ans[k];
          const [p,n] = box.querySelectorAll('.pill'); // p=正面, n=負面
          clearMarks(box);
          if(correct === '+') p.classList.add('key');
          if(correct === '-') n.classList.add('key');
          const user = state.ans[k];
          if(user === '+') p.classList.add(correct==='+'?'ok':'wrong');
          if(user === '-') n.classList.add(correct==='-'?'ok':'wrong');
        });
      }
    }
  }catch(err){ toast(false, String(err.message||err)); }
});

$('btnBack').addEventListener('click', ()=>{ show(document.getElementById('qwrap'), false); toast(true,''); });
$('btnReset').addEventListener('click', ()=>{
  state.qno=''; state.ans={STOCK:'',BOND:'',FX:'',COM:'',RE:''};
  show(document.getElementById('qwrap'), false); toast(true,''); setCrumb(); document.getElementById('scorebox').style.display='none';
});

document.getElementById('name').addEventListener('input', setCrumb);
document.getElementById('sid').addEventListener('input', setCrumb);
</script>
</body>
</html>
