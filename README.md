<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Polfood Rampe Leergut</title>

<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

<style>
body { font-family: Arial; background:#f4f4f4; padding:15px; margin:0; }
.box { background:white; padding:15px; max-width:900px; margin:auto; border-radius:10px; box-shadow:0 0 10px rgba(0,0,0,0.1);}
h2 { text-align:center; }

label { font-weight:bold; display:block; margin-top:10px; }

input {
  width:100%;
  padding:10px;
  margin:6px 0;
  font-size:16px;
}

button {
  width:100%;
  padding:12px;
  margin-top:10px;
  background:black;
  color:white;
  border:none;
  border-radius:6px;
}

table {
  width:100%;
  margin-top:20px;
  border-collapse:collapse;
  font-size:12px;
}

th, td {
  border:1px solid #ddd;
  padding:6px;
  text-align:center;
}

.dropdown { position:relative; }
.dropdown-list {
  position:absolute;
  width:100%;
  background:white;
  border:1px solid #ccc;
  max-height:200px;
  overflow:auto;
  display:none;
  z-index:1000;
}

.dropdown-item { padding:10px; cursor:pointer; }
.dropdown-item:hover { background:#eee; }

.action-btn {
  background:#c62828;
  color:white;
  border:none;
  padding:6px 10px;
  border-radius:4px;
  cursor:pointer;
}
</style>
</head>

<body>

<div class="box">
<h2>🚛 Polfood Rampe Leergut</h2>

<label>Datum</label>
<input type="date" id="datum">

<label>Spedition</label>
<div class="dropdown">
  <input id="spedition">
  <div id="list" class="dropdown-list"></div>
</div>

<label>E2 IN</label>
<input type="number" inputmode="numeric" id="e2_in">

<label>E2 OUT</label>
<input type="number" inputmode="numeric" id="e2_out">

<label>H1 IN</label>
<input type="number" inputmode="numeric" id="h1_in">

<label>H1 OUT</label>
<input type="number" inputmode="numeric" id="h1_out">

<label>EPAL IN</label>
<input type="number" inputmode="numeric" id="epal_in">

<label>EPAL OUT</label>
<input type="number" inputmode="numeric" id="epal_out">

<label>Einweg</label>
<input type="number" inputmode="numeric" id="einweg">

<label>Bemerkung</label>
<input id="bemerkung">

<label>Foto</label>
<input type="file" id="foto" accept="image/*" capture="environment">

<button onclick="addEntry()">➕ Speichern</button>
<button onclick="exportExcel()">📦 Excel</button>

<table>
<thead>
<tr>
<th>Datum</th>
<th>Spedition</th>
<th>E2 IN</th>
<th>E2 OUT</th>
<th>H1 IN</th>
<th>H1 OUT</th>
<th>EPAL IN</th>
<th>EPAL OUT</th>
<th>Einweg</th>
<th>Bemerkung</th>
<th>Foto</th>
<th>Aktion</th>
</tr>
</thead>
<tbody id="table"></tbody>
</table>
</div>

<script>

const standard = ["Masan","Nordfrost","Dachser","Podolski","Garex","Nagel"];

let data = JSON.parse(localStorage.getItem("leergut") || "[]");
let listData = JSON.parse(localStorage.getItem("sped")) || [...standard];
let currentPhoto=null;

const input=document.getElementById("spedition");
const list=document.getElementById("list");

document.getElementById("datum").value=new Date().toISOString().split("T")[0];

// Dropdown
function renderList(f=""){
list.innerHTML="";
listData.filter(x=>x.toLowerCase().includes(f.toLowerCase()))
.forEach(x=>{
const d=document.createElement("div");
d.textContent=x;
d.className="dropdown-item";
d.onclick=()=>{input.value=x; list.style.display="none";}
list.appendChild(d);
});
list.style.display="block";
}

input.addEventListener("input",()=>renderList(input.value));
input.addEventListener("focus",()=>renderList(input.value));

document.addEventListener("click",(e)=>{
if(!e.target.closest(".dropdown")) list.style.display="none";
});

function addSped(v){
if(v && !listData.includes(v)){
listData.push(v);
localStorage.setItem("sped",JSON.stringify(listData));
}
}

// Eintrag speichern
function addEntry(){
const s=input.value;
if(!s)return alert("Spedition fehlt");

addSped(s);

data.push({
datum:datum.value,
spedition:s,
e2_in:+e2_in.value||0,
e2_out:+e2_out.value||0,
h1_in:+h1_in.value||0,
h1_out:+h1_out.value||0,
epal_in:+epal_in.value||0,
epal_out:+epal_out.value||0,
einweg:+einweg.value||0,
bemerkung:bemerkung.value
});

localStorage.setItem("leergut",JSON.stringify(data));
render();

document.querySelectorAll("input").forEach(i=>{if(i.type!=="date")i.value=""});
input.focus();
}

// 🔴 NEU: Löschen Funktion
function del(index){
if(confirm("Eintrag löschen?")){
data.splice(index,1);
localStorage.setItem("leergut",JSON.stringify(data));
render();
}
}

// Render Tabelle
function render(){
const t=document.getElementById("table");
t.innerHTML="";
data.forEach((r,i)=>{
t.innerHTML+=`
<tr>
<td>${r.datum}</td>
<td>${r.spedition}</td>
<td>${r.e2_in}</td>
<td>${r.e2_out}</td>
<td>${r.h1_in}</td>
<td>${r.h1_out}</td>
<td>${r.epal_in}</td>
<td>${r.epal_out}</td>
<td>${r.einweg}</td>
<td>${r.bemerkung}</td>
<td>${r.foto||""}</td>
<td><button class="action-btn" onclick="del(${i})">X</button></td>
</tr>`;
});
}

function exportExcel(){
const ws=XLSX.utils.json_to_sheet(data);
const wb=XLSX.utils.book_new();
XLSX.utils.book_append_sheet(wb,ws,"Leergut");
XLSX.writeFile(wb,"Leergut.xlsx");
}

render();

</script>

</body>
</html>
