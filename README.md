<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Coruja Play — Painel IPTV</title>
  <style>
    :root{font-family:Inter,system-ui,Arial,sans-serif}
    body{margin:0;padding:20px;background:#0f172a;color:#e6eef8}
    .card{background:#0b1220;border-radius:10px;padding:18px;box-shadow:0 6px 18px rgba(2,6,23,.6)}
    h1{margin:0 0 10px;font-size:20px}
    label{display:block;margin-top:10px;font-size:13px;color:#bcd0ea}
    input,select{width:100%;padding:8px;margin-top:6px;border-radius:6px;border:1px solid #16324a;background:#071224;color:#e6eef8}
    button{margin-top:12px;padding:10px 12px;border-radius:8px;border:0;background:#1f8cff;color:white;cursor:pointer}
    table{width:100%;border-collapse:collapse;margin-top:18px}
    th,td{padding:8px;border-bottom:1px solid rgba(255,255,255,.04);text-align:left;font-size:14px}
    .tag{display:inline-block;padding:4px 8px;border-radius:999px;background:#142d3f;color:#bfe0ff;font-size:12px}
    .expired{background:#4d1f1f;color:#ffd6d6}
    .soon{background:#4a3b1f;color:#fff6d6}
    .actions button{margin-right:6px}
    .small{font-size:12px;color:#9fb6d4}
    .topbar{display:flex;gap:10px;align-items:center;justify-content:space-between}
    .search{width:200px}
    .note{margin-top:12px;font-size:13px;color:#9fb6d4}
    .app-icon{font-size:18px;margin-right:5px}
  </style>
</head>
<body>
  <div class="card">
    <div class="topbar">
      <div>
        <h1>Coruja Play — Painel de Clientes IPTV</h1>
        <div class="small">Adicione clientes, veja vencimentos e envie mensagem no WhatsApp com o PIX.</div>
      </div>
      <div>
        <input id="search" class="search" placeholder="Pesquisar cliente..." />
      </div>
    </div>

    <form id="form">
      <label>Cliente</label>
      <input id="cliente" required placeholder="Nome do cliente" />

      <label>Aplicativo</label>
      <select id="app">
        <option value="Coruja Play">🦉 Coruja Play</option>
        <option value="IPTV Smarters">📺 IPTV Smarters</option>
        <option value="DuplexPlay">🎬 DuplexPlay</option>
        <option value="Smart STB">💡 Smart STB</option>
        <option value="XCIPTV">⚙️ XCIPTV</option>
        <option value="Outro">🔘 Outro</option>
      </select>

      <label>Preço</label>
      <select id="preco">
        <option value="30">R$ 30,00</option>
        <option value="50">R$ 50,00</option>
        <option value="60">R$ 60,00</option>
      </select>

      <label>MAC (opcional)</label>
      <input id="mac" placeholder="AA:BB:CC:11:22:33" />

      <label>Chave (key) (opcional)</label>
      <input id="chave" placeholder="Chave do cliente / key" />

      <label>Telefone (incluir código do país sem '+' — ex: 5511999998888)</label>
      <input id="telefone" placeholder="5511999998888" />

      <label>Data de expiração</label>
      <input id="vencimento" type="date" required />

      <button type="submit">Adicionar / Atualizar cliente</button>
    </form>

    <div class="note">PIX: <strong>Elcid Batista</strong> — Banco: <strong>Nubank</strong> — Chave PIX: <strong id="pix-key">81985117522</strong></div>

    <table id="tabela">
      <thead>
        <tr><th>Cliente</th><th>App</th><th>Preço</th><th>MAC</th><th>Chave</th><th>Telefone</th><th>Vencimento</th><th>Status</th><th>Ações</th></tr>
      </thead>
      <tbody></tbody>
    </table>

  </div>

  <script>
    const PIX_NAME = 'Elcid Batista';
    const PIX_BANK = 'Nubank';
    const PIX_KEY = '81985117522';

    const form = document.getElementById('form');
    const clienteInput = document.getElementById('cliente');
    const appInput = document.getElementById('app');
    const precoInput = document.getElementById('preco');
    const macInput = document.getElementById('mac');
    const chaveInput = document.getElementById('chave');
    const telInput = document.getElementById('telefone');
    const vencInput = document.getElementById('vencimento');
    const tabelaBody = document.querySelector('#tabela tbody');
    const search = document.getElementById('search');

    let clientes = JSON.parse(localStorage.getItem('coruja_clientes') || '[]');

    function save(){ localStorage.setItem('coruja_clientes', JSON.stringify(clientes)); }
    function formatBRDate(dateStr){ if(!dateStr) return ''; const d = new Date(dateStr); return d.toLocaleDateString('pt-BR'); }
    function daysUntil(dateStr){ const today = new Date(); const d = new Date(dateStr + 'T00:00:00'); const diff = Math.ceil((d - new Date(today.getFullYear(), today.getMonth(), today.getDate()))/(1000*60*60*24)); return diff; }

    function getAppIcon(app){
      switch(app){
        case 'Coruja Play': return '🦉';
        case 'IPTV Smarters': return '📺';
        case 'DuplexPlay': return '🎬';
        case 'Smart STB': return '💡';
        case 'XCIPTV': return '⚙️';
        default: return '🔘';
      }
    }

    function render(filter=''){
      tabelaBody.innerHTML='';
      clientes.forEach((c, idx)=>{
        if(filter && !c.cliente.toLowerCase().includes(filter.toLowerCase())) return;
        const tr = document.createElement('tr');
        const days = daysUntil(c.vencimento);
        let statusLabel = '';
        if(days < 0) statusLabel = `<span class="tag expired">Vencido (${Math.abs(days)}d)</span>`;
        else if(days <= 3) statusLabel = `<span class="tag soon">Vence em ${days} dia(s)</span>`;
        else statusLabel = `<span class="tag">OK</span>`;

        tr.innerHTML = `
          <td>${c.cliente}</td>
          <td><span class="app-icon">${getAppIcon(c.app)}</span>${c.app}</td>
          <td>R$ ${Number(c.preco).toFixed(2)}</td>
          <td>${c.mac||''}</td>
          <td>${c.chave||''}</td>
          <td>${c.telefone||''}</td>
          <td>${formatBRDate(c.vencimento)}</td>
          <td>${statusLabel}</td>
          <td class="actions">
            <button onclick="openWhatsApp(${idx})">Enviar WhatsApp</button>
            <button onclick="copiarPIX()">Copiar PIX</button>
            <button onclick="editar(${idx})">Editar</button>
            <button onclick="remover(${idx})" style="background:#e24b4b">Remover</button>
          </td>
        `;
        tabelaBody.appendChild(tr);
      });
    }

    function gerarMensagem(c){
      const venc = formatBRDate(c.vencimento);
      return `Olá ${c.cliente},\\n\\nInformamos que a sua assinatura *${c.app}* com vencimento em ${venc} está próxima / vencida.\\nValor: R$ ${Number(c.preco).toFixed(2)}.\\n\\nPara regularizar, envie o pagamento via PIX:\\nNome: ${PIX_NAME}\\nBanco: ${PIX_BANK}\\nChave PIX: ${PIX_KEY}\\n\\nApós o pagamento, por favor envie o comprovante aqui.\\n\\nObrigado!`;
    }

    function sanitizePhone(phone){ if(!phone) return ''; return phone.replace(/\\D/g, ''); }

    window.openWhatsApp = function(idx){
      const c = clientes[idx];
      if(!c || !c.telefone){ alert('Cliente não tem telefone cadastrado'); return; }
      const tel = sanitizePhone(c.telefone);
      const text = encodeURIComponent(gerarMensagem(c));
      const url = `https://wa.me/${tel}?text=${text}`;
      window.open(url,'_blank');
    }

    window.copiarPIX = function(){
      navigator.clipboard.writeText(`${PIX_NAME} - ${PIX_BANK} - PIX: ${PIX_KEY}`).then(()=>{
        alert('Chave PIX copiada para a área de transferência');
      });
    }

    window.editar = function(idx){
      const c = clientes[idx];
      clienteInput.value = c.cliente;
      appInput.value = c.app;
      precoInput.value = c.preco;
      macInput.value = c.mac || '';
      chaveInput.value = c.chave || '';
      telInput.value = c.telefone || '';
      vencInput.value = c.vencimento;
      clientes.splice(idx,1);
      save(); render(search.value);
    }

    window.remover = function(idx){
      if(confirm('Remover este cliente?')){
        clientes.splice(idx,1);
        save(); render(search.value);
      }
    }

    form.addEventListener('submit', e=>{
      e.preventDefault();
      const novo = {
        cliente: clienteInput.value.trim(),
        app: appInput.value,
        preco: precoInput.value,
        mac: macInput.value.trim(),
        chave: chaveInput.value.trim(),
        telefone: telInput.value.trim(),
        vencimento: vencInput.value
      };
      clientes.push(novo); save(); render(search.value); form.reset();
    });

    search.addEventListener('input', ()=> render(search.value));

    function checkVencimentos(){
      const aviso = clientes.filter(c => daysUntil(c.vencimento) <= 3);
      if(aviso.length){
        const list = aviso.map(a=>`${a.cliente} — vence em ${daysUntil(a.vencimento)} dia(s)`).join('\\n');
        if(confirm('Há clientes com vencimento em até 3 dias:\\n\\n'+list+'\\n\\nDeseja abrir a lista para enviar WhatsApp agora?')){
          aviso.forEach(c => {
            const tel = sanitizePhone(c.telefone);
            if(!tel) return;
            const text = encodeURIComponent(gerarMensagem(c));
            const url = `https://wa.me/${tel}?text=${text}`;
            window.open(url, '_blank');
          });
        }
      }
    }

    render();
    setTimeout(checkVencimentos, 500);
  </script>
</body>
</html>
