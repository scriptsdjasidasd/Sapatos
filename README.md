<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no" />
  <title>Controle Financeiro — Degrade Preto</title>
  <style>
    :root{
      --glass: rgba(255,255,255,0.06);
      --accent: rgba(255,255,255,0.12);
      --ok: #4ade80;
      --bad: #fb7185;
      --text: #e6eef8;
    }

    /* Reset básico */
    *{box-sizing:border-box;}
    html,body{
      height:100%;
      margin:0;
      padding:0;
      font-family:Inter,ui-sans-serif,system-ui,-apple-system,"Segoe UI",Roboto,"Helvetica Neue",Arial;
      overflow-x:hidden; /* bloqueia scroll horizontal */
      background: linear-gradient(135deg, #000000 0%, #0b1226 45%, #081129 100%);
      color: var(--text);
      -webkit-font-smoothing:antialiased;
    }

    body{
      display:flex;
      align-items:center;
      justify-content:center;
      padding:16px;
    }

    .container{
      width:100%;
      max-width:980px;
      background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
      border-radius:18px;
      padding:22px;
      box-shadow:0 8px 30px rgba(2,6,23,0.7);
      backdrop-filter: blur(6px);
      border:1px solid rgba(255,255,255,0.03);
      display:grid;
      grid-template-columns:1fr 360px;
      gap:20px;
    }

    header.app-header{display:flex;align-items:center;gap:14px;}
    .logo{
      width:56px;
      height:56px;
      border-radius:12px;
      background:linear-gradient(135deg,#111 0%, #000 100%);
      display:flex;
      align-items:center;
      justify-content:center;
      border:1px solid rgba(255,255,255,0.04);
    }
    .logo strong{font-size:18px;}
    h1{margin:0;font-size:20px;}
    p.lead{margin:0;color:rgba(230,238,248,0.7);font-size:13px;}

    .main-card{
      padding:18px;
      border-radius:12px;
      background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005));
      border:1px solid rgba(255,255,255,0.02);
    }

    .balance{display:flex;flex-direction:column;gap:8px;}
    .balance .value{font-size:28px;font-weight:700;}
    .totals{display:flex;gap:8px;}
    .total-chip{
      flex:1;
      padding:10px;
      border-radius:10px;
      background:var(--glass);
      text-align:center;
      border:1px solid rgba(255,255,255,0.02);
    }
    .total-chip small{display:block;color:rgba(230,238,248,0.7);}

    form.add-form{display:flex;flex-wrap:wrap;gap:8px;margin-top:12px;}
    form input, form select{
      padding:10px;
      border-radius:8px;
      border:1px solid rgba(255,255,255,0.04);
      background:transparent;
      color:var(--text);
      min-width:0;
    }
    form input[type="number"]{width:150px;}
    .btn{
      padding:10px 14px;
      border-radius:10px;
      border:0;
      background:linear-gradient(90deg,#111 0%, #222 100%);
      cursor:pointer;
      color:var(--text);
      box-shadow:inset 0 -2px 0 rgba(255,255,255,0.02);
    }

    .transactions{
      margin-top:14px;
      display:flex;
      flex-direction:column;
      gap:8px;
      max-height:360px;
      overflow:auto;
      padding-right:6px;
    }
    .tx{display:flex;align-items:center;justify-content:space-between;padding:10px;border-radius:10px;background:linear-gradient(90deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005));border:1px solid rgba(255,255,255,0.02);}
    .tx .meta{display:flex;gap:12px;align-items:center;}
    .dot{width:10px;height:10px;border-radius:50%;}
    .tx .desc{font-size:14px;}
    .tx .amount{font-weight:700;}

    .sidebar{display:flex;flex-direction:column;gap:12px;}
    .card{background:var(--glass);padding:14px;border-radius:12px;border:1px solid rgba(255,255,255,0.02);}
    .chart-canvas{width:100%;height:120px;}

    /* Responsivo para celular */
    @media (max-width:880px){
      .container{grid-template-columns:1fr;}
      .sidebar{order:2;}
    }
  </style>
</head>
<body>
  <div class="container" role="application">
    <div>
      <header class="app-header">
        <div class="logo"><strong>€</strong></div>
        <div>
          <h1>Meu Cofrinho</h1>
          <p class="lead">Controle rápido de entradas e saídas — simples e com estilo.</p>
        </div>
      </header>

      <section class="main-card" aria-labelledby="balLabel">
        <div class="balance">
          <small id="balLabel">Saldo atual</small>
          <div class="value" id="balance">R$ 0,00</div>
          <div class="totals">
            <div class="total-chip">
              <small>Entradas</small>
              <div id="income">R$ 0,00</div>
            </div>
            <div class="total-chip">
              <small>Saídas</small>
              <div id="expense">R$ 0,00</div>
            </div>
          </div>
        </div>

        <form class="add-form" id="form">
          <input type="text" id="desc" placeholder="Descrição (ex: Freelance)" required />
          <select id="type">
            <option value="income">Entrada</option>
            <option value="expense">Saída</option>
          </select>
          <input type="number" id="amount" placeholder="Valor (R$)" step="0.01" required />
          <button class="btn" type="submit">Adicionar</button>
        </form>

        <div class="transactions" id="list"></div>
      </section>
    </div>

    <aside class="sidebar">
      <div class="card">
        <h3 style="margin:0 0 8px 0">Resumo rápido</h3>
        <p style="margin:0 0 12px 0;color:rgba(230,238,248,0.75);font-size:13px">Veja sua tendência dos últimos lançamentos.</p>
        <canvas id="spark" class="chart-canvas"></canvas>
      </div>

      <div class="card" style="display:flex;justify-content:space-between;align-items:center">
        <div>
          <small>Dados salvos localmente</small>
          <div style="font-weight:700">LocalStorage</div>
        </div>
        <button class="btn" id="reset">Limpar</button>
      </div>
    </aside>
  </div>

  <script>
    const form = document.getElementById('form');
    const desc = document.getElementById('desc');
    const amount = document.getElementById('amount');
    const type = document.getElementById('type');
    const list = document.getElementById('list');
    const balanceEl = document.getElementById('balance');
    const incomeEl = document.getElementById('income');
    const expenseEl = document.getElementById('expense');
    const spark = document.getElementById('spark');
    const resetBtn = document.getElementById('reset');

    let tx = JSON.parse(localStorage.getItem('tx')) || [];

    function formatBRL(v){
      const n = Number(v) || 0;
      return n.toLocaleString('pt-BR',{style:'currency',currency:'BRL'});
    }

    function save(){ localStorage.setItem('tx', JSON.stringify(tx)); }

    function update(){
      const income = tx.filter(t=>t.type==='income').reduce((s,t)=>s+Number(t.amount),0);
      const expense = tx.filter(t=>t.type==='expense').reduce((s,t)=>s+Number(t.amount),0);
      const bal = income - expense;
      balanceEl.textContent = formatBRL(bal);
      incomeEl.textContent = formatBRL(income);
      expenseEl.textContent = formatBRL(expense);

      list.innerHTML = '';
      tx.slice().reverse().forEach((t,i)=>{
        const el = document.createElement('div');
        el.className='tx';
        const dot = document.createElement('div');
        dot.className='dot';
        dot.style.background = t.type==='income'? 'linear-gradient(90deg,#0f766e,#34d399)':'linear-gradient(90deg,#991b1b,#fb7185)';
        const meta = document.createElement('div');
        meta.className='meta';
        meta.appendChild(dot);
        const descEl = document.createElement('div');
        descEl.className='desc';
        descEl.textContent = t.desc;
        meta.appendChild(descEl);

        const right = document.createElement('div');
        right.style.display='flex';
        right.style.gap='8px';
        right.style.alignItems='center';
        const amt = document.createElement('div');
        amt.className='amount';
        amt.textContent = (t.type==='expense'?'- ':'')+formatBRL(t.amount);
        const del = document.createElement('button');
        del.className='btn';
        del.textContent='x';
        del.style.padding='6px 8px';
        del.onclick=()=>{ remove(tx.length-1-i) };
        right.appendChild(amt);
        right.appendChild(del);

        el.appendChild(meta);
        el.appendChild(right);
        list.appendChild(el);
      });

      drawSpark();
    }

    function addTransaction(o){ tx.push(o); save(); update(); }
    function remove(index){ tx.splice(index,1); save(); update(); }

    form.addEventListener('submit', e=>{
      e.preventDefault();
      const a = parseFloat(amount.value);
      if(!desc.value || !a){ alert('Preencha descrição e valor'); return; }
      addTransaction({desc:desc.value,amount:Math.abs(a),type:type.value,at:Date.now()});
      desc.value=''; amount.value=''; type.value='income';
    });

    resetBtn.addEventListener('click', ()=>{
      if(confirm('Limpar todos os dados?')){ tx=[]; save(); update(); }
    });

    function drawSpark(){
      const ctx = spark.getContext('2d');
      const w = spark.width = spark.clientWidth*devicePixelRatio;
      const h = spark.height = spark.clientHeight*devicePixelRatio;
      ctx.clearRect(0,0,w,h);
      if(tx.length===0) return;
      const data = tx.map(t=>t.type==='income'? Number(t.amount) : -Number(t.amount));
      const slice = data.slice(-20);
      const min = Math.min(...slice);
      const max = Math.max(...slice);
      const range = (max-min)||1;
      ctx.lineWidth = 2*devicePixelRatio;
      ctx.beginPath();
      slice.forEach((v,i)=>{
        const x = (i/(slice.length-1||1))*(w-8)+4;
        const y = h-((v-min)/range)*(h-8)-4;
        if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
      });
      const grad = ctx.createLinearGradient(0,0,w,0);
      grad.addColorStop(0,'#06b6d4'); grad.addColorStop(1,'#7c3aed');
      ctx.strokeStyle = grad; ctx.stroke();
      ctx.lineTo(w-4,h); ctx.lineTo(4,h); ctx.closePath();
      ctx.fillStyle='rgba(124,58,237,0.06)'; ctx.fill();
    }

    update();
    window.addEventListener('resize',()=>requestAnimationFrame(drawSpark));
  </script>
</body>
</html>
