<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Quiz Game v2.5</title>

<style>
* {
  box-sizing: border-box;
  font-family: Arial, sans-serif;
}

body {
  margin: 0;
  background: linear-gradient(180deg,#4facfe,#00f2fe);
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.container {
  background: #fff;
  width: 95%;
  max-width: 420px;
  padding: 20px;
  border-radius: 16px;
  text-align: center;
}

h2,h3 {
  margin: 10px 0;
}

input {
  width: 100%;
  padding: 12px;
  margin: 10px 0;
  font-size: 16px;
}

button {
  width: 100%;
  padding: 14px;
  margin: 6px 0;
  font-size: 16px;
  border: none;
  border-radius: 10px;
  background: #4facfe;
  color: white;
}

button:active {
  background: #2f9cff;
}

textarea {
  width: 100%;
  height: 90px;
  padding: 10px;
  font-size: 15px;
}

.feedback {
  font-weight: bold;
  margin: 10px 0;
}

.small {
  font-size: 13px;
  color: gray;
}
</style>
</head>

<body>

<!-- LOGIN -->
<div class="container" id="login">
  <h2>Quiz Game</h2>
  <p class="small">Version 2.5</p>
  <p class="small">Create by @manzz</p>
  <input id="nama" placeholder="Masukkan Nama Siswa">
  <button onclick="mulai()">Mulai Game</button>
</div>

<!-- QUIZ -->
<div class="container" id="quiz" style="display:none;">
  <h3 id="soal"></h3>
  <div id="opsi"></div>
  <textarea id="essay" style="display:none;" placeholder="Tulis jawabanmu..."></textarea>
  <div class="feedback" id="feedback"></div>
  <button id="lanjut" onclick="next()" style="display:none;">Lanjut Soal</button>
</div>

<audio id="benar" src="https://assets.mixkit.co/sfx/preview/mixkit-correct-answer-tone-2870.mp3"></audio>
<audio id="salah" src="https://assets.mixkit.co/sfx/preview/mixkit-wrong-answer-fail-notification-946.mp3"></audio>

<script>
// ================= DATABASE SOAL =================

// 15 PILIHAN GANDA
const pg = [
 {t:"pg",q:"1. Ibu kota Indonesia adalah …",a:["Jakarta","Bandung","Surabaya","Medan"],c:0},
 {t:"pg",q:"2. 8 + 7 = …",a:["13","14","15","16"],c:2},
 {t:"pg",q:"3. Warna bendera Indonesia adalah …",a:["Merah Putih","Putih Biru","Merah Biru","Hijau Putih"],c:0},
 {t:"pg",q:"4. Hasil dari 6 × 5 adalah …",a:["25","30","35","40"],c:1},
 {t:"pg",q:"5. Planet terdekat dengan matahari adalah …",a:["Venus","Mars","Merkurius","Bumi"],c:2},
 {t:"pg",q:"6. Alat untuk menulis adalah …",a:["Penggaris","Pensil","Penghapus","Buku"],c:1},
 {t:"pg",q:"7. Lambang negara Indonesia adalah …",a:["Harimau","Garuda","Elang","Rajawali"],c:1},
 {t:"pg",q:"8. 10 ÷ 2 = …",a:["3","4","5","6"],c:2},
 {t:"pg",q:"9. Contoh hewan herbivora adalah …",a:["Singa","Harimau","Sapi","Serigala"],c:2},
 {t:"pg",q:"10. Matahari terbit dari arah …",a:["Utara","Selatan","Barat","Timur"],c:3},
 {t:"pg",q:"11. Jumlah hari dalam satu minggu adalah …",a:["5","6","7","8"],c:2},
 {t:"pg",q:"12. Alat pernapasan manusia adalah …",a:["Jantung","Paru-paru","Hati","Lambung"],c:1},
 {t:"pg",q:"13. Contoh sikap jujur adalah …",a:["Mencontek","Mengakui kesalahan","Berbohong","Menyalahkan orang"],c:1},
 {t:"pg",q:"14. 20 – 8 = …",a:["10","11","12","13"],c:2},
 {t:"pg",q:"15. Benda yang dapat mengalir adalah …",a:["Batu","Air","Kayu","Besi"],c:1}
];

// 5 ESSAY
const essay = [
 {t:"essay",q:"16. Sebutkan sila pertama Pancasila!"},
 {t:"essay",q:"17. Apa fungsi jantung bagi manusia?"},
 {t:"essay",q:"18. Jelaskan arti sikap jujur!"},
 {t:"essay",q:"19. Apa yang dimaksud dengan energi?"},
 {t:"essay",q:"20. Sebutkan contoh sikap disiplin di sekolah!"}
];

// GABUNG & ACAK
let soal = [...pg, ...essay].sort(()=>Math.random()-0.5);

// ================= LOGIC =================

let i = 0;
let skor = 0;
let nama = "";

function mulai(){
  nama = document.getElementById("nama").value;
  if(!nama) return alert("Nama harus diisi!");
  document.getElementById("login").style.display="none";
  document.getElementById("quiz").style.display="block";
  tampil();
}

function tampil(){
  document.getElementById("feedback").innerText="";
  document.getElementById("lanjut").style.display="none";

  let s = soal[i];
  document.getElementById("soal").innerText = s.q;
  let opsi = document.getElementById("opsi");
  let es = document.getElementById("essay");
  opsi.innerHTML="";
  es.style.display="none";

  if(s.t==="pg"){
    s.a.forEach((x,n)=>{
      let b=document.createElement("button");
      b.innerText=x;
      b.onclick=()=>cek(n);
      opsi.appendChild(b);
    });
  } else {
    es.style.display="block";
    let b=document.createElement("button");
    b.innerText="Kirim Jawaban";
    b.onclick=kirimEssay;
    opsi.appendChild(b);
  }
}

function cek(j){
  let s=soal[i];
  if(j===s.c){
    skor+=5;
    document.getElementById("benar").play();
    document.getElementById("feedback").innerText="✔ Jawaban Benar";
  } else {
    document.getElementById("salah").play();
    document.getElementById("feedback").innerText="✘ Jawaban Salah";
  }
  document.getElementById("lanjut").style.display="block";
}

function kirimEssay(){
  skor+=5;
  document.getElementById("benar").play();
  document.getElementById("feedback").innerText="✔ Jawaban Tersimpan";
  document.getElementById("lanjut").style.display="block";
}

function next(){
  i++;
  if(i<soal.length) tampil();
  else hasil();
}

function hasil(){
  let nilai = skor>=85?"A":skor>=70?"B":skor>=55?"C":skor>=40?"D":"E";
  document.body.innerHTML=`
  <div class="container">
    <h2>Quiz Game</h2>
    <p>Nama: ${nama}</p>
    <p>Skor: ${skor}</p>
    <h3>Nilai: ${nilai}</h3>
    <p class="small">Version 2.5</p>
    <p class="small">Create by @manzz</p>
  </div>`;
}
</script>

</body>
</html>
