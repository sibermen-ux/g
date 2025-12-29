<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Boncuk Patlatmaca</title>

<script src="https://cdn.onesignal.com/sdks/OneSignalSDK.js" async=""></script>
<script>
  window.OneSignal = window.OneSignal || [];
  OneSignal.push(function() {
    OneSignal.init({
      appId: "BURAYA_ONESIGNAL_APP_ID_GELECEK", 
    });
  });
</script>

<style>
  * { box-sizing: border-box; user-select: none; -webkit-user-select: none; }
  body { margin: 0; font-family: sans-serif; background: #020617; height: 100vh; overflow: hidden; touch-action: none; }
  .hidden { display: none !important; }

  /* --- OYUN EKRANI --- */
  #gamePage { width: 100%; height: 100%; position: relative; background: radial-gradient(circle at bottom, #1e3a8a, #020617); }
  #score { position: absolute; top: 15px; left: 15px; color: white; font-size: 18px; z-index: 10; }
  .bubble { width: 38px; height: 38px; border-radius: 50%; position: absolute; box-shadow: inset -4px -4px 8px rgba(0,0,0,0.4), inset 4px 4px 6px rgba(255,255,255,0.3); }
  .c0 { background: #ef4444; } .c1 { background: #22c55e; } .c2 { background: #3b82f6; } .c3 { background: #eab308; } .c4 { background: #a855f7; }
  .dot { width: 6px; height: 6px; background: white; border-radius: 50%; position: absolute; opacity: 0.5; pointer-events: none; }
  #secretBar { position: absolute; bottom: 0; left: 0; height: 4px; background: #ec4899; width: 0%; opacity: 0.8; z-index: 100; transition: width 0.1s linear; }

  /* --- SOHBET EKRANI --- */
  #chatPage { width: 100%; height: 100%; background: #0f172a; display: flex; flex-direction: column; }
  #chatHeader { padding: 15px; background: #1e293b; color: white; display: flex; align-items: center; border-bottom: 1px solid #334155; }
  #backBtn { font-size: 24px; margin-right: 15px; cursor: pointer; color: #9ca3af; }
  #messages { flex: 1; padding: 15px; overflow-y: auto; display: flex; flex-direction: column; gap: 10px; }
  .msg { background: #ec4899; color: white; padding: 12px; border-radius: 15px; align-self: flex-end; max-width: 80%; font-size: 15px; }
  #btnContainer { padding: 20px; display: flex; flex-direction: column; gap: 10px; background: #1e293b; border-top: 1px solid #334155; }
  .send-btn { border: none; padding: 15px; border-radius: 12px; background: #f472b6; color: white; font-size: 16px; font-weight: bold; }
  .send-btn:active { background: #ec4899; transform: scale(0.98); }
</style>
</head>
<body>

<div id="gamePage">
  <div id="score">🎮 Skor: <span id="sVal">0</span></div>
  <div id="bubbles"></div>
  <div id="secretBar"></div>
</div>

<div id="chatPage" class="hidden">
  <div id="chatHeader"><span id="backBtn">←</span> <span>Oyun İstatistikleri</span></div>
  <div id="messages"></div>
  <div id="btnContainer">
    <button class="send-btn" onclick="triggerNotify('💔 Özledim')">💔 Özledim</button>
    <button class="send-btn" onclick="triggerNotify('❤️ Seni Seviyorum')">❤️ Seni Seviyorum</button>
    <button class="send-btn" onclick="triggerNotify('🌙 Aklımdasın')">🌙 Aklımdasın</button>
  </div>
</div>

<script>
/* --- KONFİGÜRASYON (OneSignal'dan Al) --- */
const APP_ID = "BURAYA_ONESIGNAL_APP_ID_GELECEK";
const REST_API_KEY = "BURAYA_REST_API_KEY_GELECEK";
const SITE_URL = "https://kullaniciadi.github.io/boncuk-game/"; // Kendi GitHub linkin

/* --- OYUN MOTORU --- */
const bubblesCont = document.getElementById("bubbles");
const colors = ["c0", "c1", "c2", "c3", "c4"];
let score = 0;

function initBubbles() {
  bubblesCont.innerHTML = "";
  for(let r=0; r<5; r++) {
    for(let c=0; c<8; c++) {
      const b = document.createElement("div");
      b.className = "bubble " + colors[Math.floor(Math.random()*5)];
      b.style.top = (r * 42 + 60) + "px";
      b.style.left = (c * 42 + (window.innerWidth - 8*42)/2) + "px";
      bubblesCont.appendChild(b);
    }
  }
}
initBubbles();

// Atış ve Puanlama (Görsel temsil)
gamePage.ontouchend = () => {
  score += 120;
  document.getElementById("sVal").innerText = score;
};

/* --- GİZLİ GEÇİŞ (2 Parmak - 3 Saniye) --- */
let sTimer, sStart;
document.addEventListener('touchstart', (e) => {
  if(e.touches.length === 2) {
    sStart = Date.now();
    sTimer = setInterval(() => {
      let diff = Date.now() - sStart;
      document.getElementById("secretBar").style.width = Math.min((diff/3000)*100, 100) + "%";
      if(diff >= 3000) {
        clearInterval(sTimer);
        document.getElementById("gamePage").classList.add("hidden");
        document.getElementById("chatPage").classList.remove("hidden");
        if(navigator.vibrate) navigator.vibrate(100);
      }
    }, 50);
  }
});
document.addEventListener('touchend', () => {
  clearInterval(sTimer);
  document.getElementById("secretBar").style.width = "0%";
});

/* --- BİLDİRİM GÖNDERME --- */
async function triggerNotify(txt) {
  // 1. Kendi ekranına mesajı ekle
  const m = document.createElement("div");
  m.className = "msg"; m.textContent = txt;
  document.getElementById("messages").appendChild(m);
  document.getElementById("messages").scrollTop = document.getElementById("messages").scrollHeight;

  // 2. OneSignal üzerinden karşı tarafa bildirim at
  try {
    await fetch("https://onesignal.com/api/v1/notifications", {
      method: "POST",
      headers: {
        "Content-Type": "application/json; charset=utf-8",
        "Authorization": "Basic " + REST_API_KEY
      },
      body: JSON.stringify({
        app_id: APP_ID,
        included_segments: ["Subscribed Users"],
        headings: {"en": "🎁 Günlük Bonus Kazandın!"},
        contents: {"en": txt},
        url: SITE_URL
      })
    });
  } catch (e) { console.error("Hata:", e); }
}

document.getElementById("backBtn").onclick = () => {
  document.getElementById("chatPage").classList.add("hidden");
  document.getElementById("gamePage").classList.remove("hidden");
};
</script>
</body>
</html>
