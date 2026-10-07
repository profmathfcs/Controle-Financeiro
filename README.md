# Pedidos 2026
Plataforma de pedidos EEEM JACOB HOFF
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Controle de Gastos</title>
<style>
:root{--bg:#f6f7f9;--card:#fff;--tx:#1b1f24;--mut:#6b7280;--bd:#e5e7eb;--ac:#2f6f5e;--bad:#c0392b;--ok:#2f6f5e;
box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#15181c;--card:#1e2328;--tx:#eceff3;--mut:#9aa3ad;--bd:#2d343b;--ac:#4fb59a;--bad:#e5766a;--ok:#4fb59a}}
:root[data-theme="dark"]{--bg:#15181c;--card:#1e2328;--tx:#eceff3;--mut:#9aa3ad;--bd:#2d343b;--ac:#4fb59a;--bad:#e5766a;--ok:#4fb59a}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--tx);font:15px/1.4 system-ui,-apple-system,"Segoe UI",Roboto,sans-serif}
main{max-width:860px;margin:0 auto;padding:16px}
header{display:flex;justify-content:space-between;align-items:center;gap:12px;flex-wrap:wrap;margin-bottom:12px}
h1{font-size:20px;margin:0}
h2{font-size:15px;margin:0 0 10px}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:12px;margin-bottom:12px}
.card{background:var(--card);border:1px solid var(--bd);border-radius:12px;padding:14px;margin-bottom:12px}
.grid .card{margin:0}
.lbl{color:var(--mut);font-size:12px}
.val{font-size:22px;font-weight:600;margin-top:2px}
input,select,button{font:inherit;color:var(--tx);background:var(--bg);border:1px solid var(--bd);border-radius:8px;padding:8px 10px;min-width:0}
button{cursor:pointer}
button.p{background:var(--ac);color:#fff;border-color:var(--ac)}
form{display:grid;grid-template-columns:repeat(auto-fit,minmax(130px,1fr));gap:8px}
.bar{display:grid;grid-template-columns:100px 1fr 90px;gap:8px;align-items:center;margin:6px 0;font-size:13px}
.trk{background:var(--bg);border-radius:6px;height:14px;overflow:hidden}
.fill{height:100%;background:var(--ac);border-radius:6px}
.row{display:flex;justify-content:space-between;align-items:center;gap:8px;padding:8px 0;border-top:1px solid var(--bd)}
.row:first-of-type{border-top:0}
.row small{color:var(--mut)}
.x{border:0;background:none;color:var(--mut);padding:4px 8px}
.prog{height:8px;background:var(--bg);border-radius:6px;overflow:hidden;margin-top:8px}
.prog>div{height:100%;background:var(--ok)}
.empty{color:var(--mut);text-align:center;padding:12px}
</style>
</head>
<body>
<main>
<header>
  <h1>💰 Controle de Gastos</h1>
  <input type="month" id="mes" aria-label="Mês">
</header>

<div class="grid">
  <div class="card"><div class="lbl">Saldo anterior (remanescente)</div><div class="val" id="ant">R$ 0,00</div></div>
  <div class="card"><div class="lbl">Receitas do mês</div><div class="val" id="rec" style="color:var(--ok)">R$ 0,00</div></div>
  <div class="card"><div class="lbl">Despesas do mês</div><div class="val" id="total" style="color:var(--bad)">R$ 0,00</div></div>
  <div class="card"><div class="lbl">Saldo do mês</div><div class="val" id="mesS">R$ 0,00</div></div>
  <div class="card"><div class="lbl">Saldo final (remanescente)</div><div class="val" id="saldo">R$ 0,00</div>
    <div class="prog"><div id="pg" style="width:0"></div></div></div>
</div>

<div class="card">
  <h2>Novo lançamento</h2>
  <form id="f">
    <select id="t"><option value="d">Despesa</option><option value="r">Receita</option></select>
    <input id="d" placeholder="Descrição" required>
    <input id="v" type="number" min="0.01" step="0.01" placeholder="Valor (R$)" required>
    <select id="c"></select>
    <input id="dt" type="date" required>
    <button class="p">Adicionar</button>
  </form>
</div>

<div class="card"><h2>Despesas por categoria</h2><div id="cats"></div></div>
<div class="card"><h2>Lançamentos</h2><div id="list"></div></div>
</main>
<script>
const CATS={d:["Moradia","Alimentação","Transporte","Saúde","Lazer","Educação","Outros"],r:["Salário","Extra","Investimentos","Outros"]};
const KEY="gastos-v1";
let S={items:[]};
try{const r=localStorage.getItem(KEY);if(r)S=Object.assign(S,JSON.parse(r))}catch(e){}
function save(){try{localStorage.setItem(KEY,JSON.stringify({items:S.items}))}catch(e){}}
const $=id=>document.getElementById(id);
const brl=n=>n.toLocaleString("pt-BR",{style:"currency",currency:"BRL"});
const sum=a=>a.reduce((s,i)=>s+i.v,0);
const tp=i=>i.t==="r"?"r":"d";
const today=new Date().toISOString().slice(0,10);
$("mes").value=today.slice(0,7);$("dt").value=today;
function fillCats(){$("c").textContent="";CATS[$("t").value].forEach(c=>{const o=document.createElement("option");o.textContent=c;$("c").appendChild(o)})}
fillCats();$("t").addEventListener("change",fillCats);

$("f").addEventListener("submit",e=>{
  e.preventDefault();
  S.items.push({id:Date.now(),t:$("t").value,d:$("d").value.trim(),v:parseFloat($("v").value),c:$("c").value,dt:$("dt").value});
  save();$("d").value="";$("v").value="";
  $("mes").value=$("dt").value.slice(0,7);render();
});
$("mes").addEventListener("change",render);

function setMoney(id,n,color){const e=$(id);e.textContent=brl(n);e.style.color=color||(n<0?"var(--bad)":"")}

function render(){
  const m=$("mes").value;
  const prev=S.items.filter(i=>i.dt.slice(0,7)<m);
  const its=S.items.filter(i=>i.dt.slice(0,7)===m).sort((a,b)=>b.dt.localeCompare(a.dt)||b.id-a.id);
  const ant=sum(prev.filter(i=>tp(i)==="r"))-sum(prev.filter(i=>tp(i)==="d"));
  const rec=sum(its.filter(i=>tp(i)==="r")), des=sum(its.filter(i=>tp(i)==="d"));
  const mes=rec-des, fim=ant+mes;
  setMoney("ant",ant);setMoney("rec",rec,"var(--ok)");setMoney("total",des,"var(--bad)");setMoney("mesS",mes);setMoney("saldo",fim);
  const disp=ant+rec;
  const pct=disp>0?Math.min(100,des/disp*100):(des>0?100:0);
  $("pg").style.width=pct+"%";$("pg").style.background=fim<0?"var(--bad)":"var(--ok)";

  const by={};its.filter(i=>tp(i)==="d").forEach(i=>by[i.c]=(by[i.c]||0)+i.v);
  const cats=$("cats");cats.textContent="";
  const ent=Object.entries(by).sort((a,b)=>b[1]-a[1]);
  if(!ent.length)cats.innerHTML='<div class="empty">Sem despesas neste mês.</div>';
  ent.forEach(([k,v])=>{
    const r=document.createElement("div");r.className="bar";
    const a=document.createElement("span");a.textContent=k;
    const t=document.createElement("div");t.className="trk";
    const f=document.createElement("div");f.className="fill";f.style.width=(v/des*100)+"%";t.appendChild(f);
    const n=document.createElement("span");n.style.textAlign="right";n.textContent=brl(v);
    r.append(a,t,n);cats.appendChild(r);
  });

  const L=$("list");L.textContent="";
  if(!its.length)L.innerHTML='<div class="empty">Nenhum lançamento.</div>';
  its.forEach(i=>{
    const r=document.createElement("div");r.className="row";
    const l=document.createElement("div");
    const s=document.createElement("div");s.textContent=i.d;
    const sm=document.createElement("small");
    sm.textContent=(tp(i)==="r"?"Receita":"Despesa")+" · "+i.c+" · "+i.dt.split("-").reverse().join("/");
    l.append(s,sm);
    const rt=document.createElement("div");
    const vv=document.createElement("strong");
    vv.textContent=(tp(i)==="r"?"+ ":"− ")+brl(i.v);
    vv.style.color=tp(i)==="r"?"var(--ok)":"var(--bad)";
    const x=document.createElement("button");x.className="x";x.textContent="✕";x.title="Excluir";
    x.onclick=()=>{S.items=S.items.filter(z=>z.id!==i.id);save();render()};
    rt.append(vv,x);r.append(l,rt);L.appendChild(r);
  });
}
render();
</script>
</body>
</html>
