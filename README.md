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

.photo-preview img {
  max-width:100%;
  max-height:150px;
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
  <input id="spedition" placeholder="Spedition wählen oder eingeben">
  <div id="list" class="dropdown-list"></div>
</div>

<label>E2 IN</label>
<input type="number" inputmode="numeric" pattern="[0-9]*" id="e2_in">

<label>E2 OUT</label>
<input type="number" inputmode="numeric" pattern="[0-9]*" id="e2_out">

<label>H1 IN</label>
<input type="number" inputmode="numeric" pattern="[0-9]*" id="h1_in">

<label>H1 OUT</label>
<input type="number" inputmode="numeric" pattern="[0-9]*" id="h1_out">

<label>EPAL IN</label>
<input type="number" inputmode="numeric" pattern="[0-9]*" id="epal_in">

<label>EPAL OUT</label>
<input type="number" inputmode="numeric" pattern="[0-9]*" id="epal_out">

<label>Einweg</label>
<input type="number" inputmode="numeric" pattern="[0-9]*" id="einweg">

<label>Bemerkung</label>
<input id="bemerkung">

<label>Foto</label>
<input type="file" id="foto" accept="image/*" capture="environment">

<div class="photo-preview" id="previewBox" style="display:none;">
<img id="preview">
</div>

<button onclick="addEntry()">➕ Eintrag speichern</button>
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

function setHeute(){
document.getElementById("datum").value=new Date().toISOString().split("T")[0];
}
setHeute();

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

document.getElementById("foto").addEventListener("change",(e)=>{
const f=e.target.files[0];
if(!f)return;
currentPhoto=f;

const r=new FileReader();
r.onload=(ev)=>{
document.getElementById("preview").src=ev.target.result;
document.getElementById("previewBox").style.display="block";
};
r.readAsDataURL(f);
});

const felder=["datum","spedition","e2_in","e2_out","h1_in","h1_out","epal_in","epal_out","einweg","bemerkung"];

felder.forEach((id,i)=>{
document.getElementById(id).addEventListener("keydown",(e)=>{
if(e.key==="Enter"){
e.preventDefault();
if(id==="spedition") addSped(input.value);

if(felder[i+1]){
document.getElementById(felder[i+1]).focus();
}else{
document.getElementById("foto").focus();
}
}
});
});

function addEntry(){
const d=document.getElementById("datum").value;
const s=input.value;

if(!s)return alert("Spedition fehlt");

addSped(s);

let foto="";
if(currentPhoto){
foto=Date.now()+".jpg";
const u=URL.createObjectURL(currentPhoto);
const a=document.createElement("a");
a.href=u;
a.download=foto;
a.click();
}

data.push({
datum:d,spedition:s,
e2_in:+e2_in.value||0,
e2_out:+e2_out.value||0,
h1_in:+h1_in.value||0,
h1_out:+h1_out.value||0,
epal_in:+epal_in.value||0,
epal_out:+epal_out.value||0,
einweg:+einweg.value||0,
bemerkung:bemerkung.value,
foto
});

localStorage.setItem("leergut",JSON.stringify(data));
render();

document.querySelectorAll("input").forEach(i=>{if(i.type!=="date")i.value=""});
currentPhoto=null;
previewBox.style.display="none";
setHeute();
input.focus();
}

function render(){
const t=document.getElementById("table");
t.innerHTML="";
data.forEach(r=>{
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
<td>${r.foto}</td>
</tr>`;
});
}

function exportExcel(){
const sums={};
data.forEach(r=>{
if(!sums[r.spedition]) sums[r.spedition]={e2:0,h1:0,epal:0};
sums[r.spedition].e2+=r.e2_in;
sums[r.spedition].h1+=r.h1_in;
sums[r.spedition].epal+=r.epal_in;
});

const ws=XLSX.utils.json_to_sheet(data);
const wb=XLSX.utils.book_new();
XLSX.utils.book_append_sheet(wb,ws,"Leergut");

XLSX.writeFile(wb,"Leergut.xlsx");
}

render();

</script>

</body>
</html>
