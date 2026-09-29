<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>إدارة صالة الألعاب V2</title>
<style>
body{margin:0;background:#0f131a;color:#fff;font-family:Arial}
.wrap{max-width:1150px;margin:auto;padding:20px}
header,.panel,.card{background:#171d27;border-radius:15px;padding:18px;margin-bottom:18px}
h1{color:#00d9ff} input,select,button{padding:11px;border:0;border-radius:9px;margin:4px}
input,select{background:#252d3a;color:#fff}button{background:#00a8cc;color:#fff;font-weight:bold;cursor:pointer}
button:hover{opacity:.85}.cards{display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:14px}
.card{margin:0;border:1px solid #344054}.busy{border-color:#ffb020}.time{font-size:29px;color:#00d9ff;margin:12px 0}
.stats{display:grid;grid-template-columns:repeat(4,1fr);gap:10px}.stat{background:#202735;padding:14px;border-radius:10px;text-align:center}.stat b{display:block;font-size:23px;color:#00d9ff}
table{width:100%;border-collapse:collapse;margin-top:10px}td,th{padding:10px;border-bottom:1px solid #344054;text-align:center}
th{color:#00d9ff}.danger{background:#d94b59}.muted{color:#aaa}
@media(max-width:650px){.stats{grid-template-columns:1fr 1fr}}
</style>
</head>
<body>
<div class="wrap">
<header><h1>🎮 إدارة صالة الألعاب</h1><p>PS4 • PS5 • Xbox • PC</p></header>

<div class="panel">
<h2>➕ إضافة جهاز</h2>
<input id="name" placeholder="اسم الجهاز مثل Xbox-01">
<select id="type"><option>PS5</option><option>PS4</option><option>Xbox</option><option>PC</option></select>
<input id="rate" type="number" min="0" value="200" placeholder="السعر بالساعة">
<button onclick="addDevice()">إضافة الجهاز</button>
</div>

<div class="panel">
<h2>📊 إحصائيات اليوم</h2>
<div class="stats">
<div class="stat">الأجهزة<b id="deviceCount">0</b></div>
<div class="stat">المشغولة<b id="busyCount">0</b></div>
<div class="stat">الجلسات<b id="sessionCount">0</b></div>
<div class="stat">مدخول اليوم<b id="todayRevenue">0 دج</b></div>
</div>
</div>

<div class="panel">
<h2>🎮 الأجهزة</h2>
<div class="cards" id="devices"></div>
</div>

<div class="panel">
<h2>💰 مدخول الأجهزة</h2>
<div id="deviceRevenue"></div>
</div>

<div class="panel">
<h2>📅 سجل الأيام</h2>
<p class="muted">السجل محفوظ داخل المتصفح.</p>
<input id="searchDate" placeholder="ابحث بالتاريخ">
<table>
<thead><tr><th>التاريخ</th><th>الجلسات</th><th>المدخول</th><th>وقت اللعب</th></tr></thead>
<tbody id="history"></tbody>
</table>
<br>
<button class="danger" onclick="clearHistory()">🗑 مسح سجل الأيام</button>
</div>
</div>

<script>
let devices=JSON.parse(localStorage.getItem("hall_devices")||"[]");
let history=JSON.parse(localStorage.getItem("hall_history")||"[]");

function save(){
 localStorage.setItem("hall_devices",JSON.stringify(devices));
 localStorage.setItem("hall_history",JSON.stringify(history));
}
function today(){return new Date().toLocaleDateString("ar-DZ")}
function addDevice(){
 const n=document.getElementById("name").value.trim();
 const t=document.getElementById("type").value;
 const r=Number(document.getElementById("rate").value);
 if(!n||r<0){alert("أدخل اسم الجهاز والسعر");return}
 devices.push({id:Date.now(),name:n,type:t,rate:r,running:false,start:0});
 document.getElementById("name").value="";
 save();render();
}
function startDevice(id){
 const d=devices.find(x=>x.id===id);
 if(d){d.running=true;d.start=Date.now();save();render()}
}
function stopDevice(id){
 const d=devices.find(x=>x.id===id);
 if(!d||!d.running)return;
 const ms=Date.now()-d.start;
 const price=ms/3600000*d.rate;
 let day=history.find(x=>x.date===today());
 if(!day){day={date:today(),sessions:0,revenue:0,minutes:0,byType:{}};history.push(day)}
 day.sessions++;
 day.revenue+=price;
 day.minutes+=Math.round(ms/60000);
 day.byType[d.type]=(day.byType[d.type]||0)+price;
 d.running=false;d.start=0;
 save();render();
}
function deleteDevice(id){
 if(confirm("هل تريد حذف الجهاز؟")){
  devices=devices.filter(x=>x.id!==id);save();render();
 }
}
function clearHistory(){
 if(confirm("هل تريد مسح سجل الأيام بالكامل؟")){
  history=[];save();render();
 }
}
function time(ms){
 let s=Math.floor(ms/1000),h=Math.floor(s/3600),m=Math.floor(s%3600/60),sec=s%60;
 return String(h).padStart(2,"0")+":"+String(m).padStart(2,"0")+":"+String(sec).padStart(2,"0");
}
function elapsed(d){return d.running?Date.now()-d.start:0}
function esc(x){
 const e=document.createElement("div");e.textContent=x;return e.innerHTML;
}
function render(){
 const box=document.getElementById("devices");box.innerHTML="";
 devices.forEach(d=>{
  const ms=elapsed(d),price=ms/3600000*d.rate;
  box.innerHTML+=`
  <div class="card ${d.running?"busy":""}">
   <div>${d.running?"🟠 مشغول":"🟢 متاح"}</div>
   <h2>${esc(d.name)}</h2>
   <div>🎮 ${esc(d.type)} — ${d.rate} دج/ساعة</div>
   <div class="time">${time(ms)}</div>
   <div>💰 السعر الحالي: ${price.toFixed(0)} دج</div>
   <button onclick="${d.running?"stopDevice":"startDevice"}(${d.id})">${d.running?"⏹ إنهاء الجلسة":"▶ بدء الجلسة"}</button>
   <button onclick="deleteDevice(${d.id})">🗑 حذف</button>
  </div>`;
 });
 const day=history.find(x=>x.date===today())||{sessions:0,revenue:0,byType:{}};
 document.getElementById("deviceCount").textContent=devices.length;
 document.getElementById("busyCount").textContent=devices.filter(x=>x.running).length;
 document.getElementById("sessionCount").textContent=day.sessions;
 document.getElementById("todayRevenue").textContent=day.revenue.toFixed(0)+" دج";

 let types=["PS5","PS4","Xbox","PC"];
 document.getElementById("deviceRevenue").innerHTML=types.map(t=>
 `<p>🎮 <b>${t}</b>: ${(day.byType[t]||0).toFixed(0)} دج</p>`).join("");

 const q=document.getElementById("searchDate").value.trim();
 document.getElementById("history").innerHTML=[...history].reverse()
 .filter(x=>!q||x.date.includes(q))
 .map(x=>{
  let h=Math.floor(x.minutes/60),m=x.minutes%60;
  return `<tr><td>${x.date}</td><td>${x.sessions}</td><td>${x.revenue.toFixed(0)} دج</td><td>${h}س ${m}د</td></tr>`;
 }).join("");
}
document.getElementById("searchDate").addEventListener("input",render);
setInterval(render,1000);
render();
</script>
</body>
</html>
gaming_hall_manager_final.html
HTML
