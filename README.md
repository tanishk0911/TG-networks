<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>💘 Proposal</title>
  <style>
    :root{
      --pink1:#ffd6e7; /* soft pink */
      --pink2:#ffb6d1; /* deeper pink */
      --ink:#2a2a2a;
      --white:#ffffff;
      --accent:#ff4d8d;
    }
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,Inter,Arial}
    body{
      background: radial-gradient(120% 120% at 50% 10%, var(--pink1), var(--pink2));
      overflow:hidden;
      color:var(--ink);
    }

    /* Floating hearts */
    .hearts{position:fixed;inset:0;pointer-events:none;opacity:.85}
    .heart{position:absolute;left:var(--x);bottom:-10vh; font-size:var(--sz); filter:drop-shadow(0 6px 10px rgba(0,0,0,.15));
      animation:float var(--dur) linear var(--delay) infinite;}
    @keyframes float{to{transform:translateY(-120vh) rotate(25deg);opacity:.2}}

    /* Center card */
    .wrap{position:fixed;inset:0;display:grid;place-items:center;padding:24px}
    .card{width:min(680px,92vw);background:rgba(255,255,255,.55);backdrop-filter: blur(10px);
      border:1px solid rgba(255,255,255,.7); border-radius:24px; padding:28px 24px; box-shadow:0 20px 60px rgba(0,0,0,.12)}
    .title{font-size:clamp(24px,5vw,40px);line-height:1.15;margin:6px 0 8px;color:#111}
    .sub{font-size:clamp(14px,3.2vw,18px);opacity:.75;margin:0 0 16px}
    .big{font-size:clamp(26px,6vw,48px);font-weight:800;color:var(--accent);letter-spacing:.5px;margin:10px 0 6px}

    /* Typewriter line */
    .tw{--caret: var(--ink); display:inline-block; border-right: 2px solid var(--caret); white-space:nowrap; overflow:hidden}

    /* Buttons */
    .cta{display:flex;gap:12px;flex-wrap:wrap;margin-top:14px}
    button{appearance:none;border:0;border-radius:16px;padding:12px 18px;font-size:16px;cursor:pointer;transition:transform .08s ease, box-shadow .2s ease}
    .yes{background:var(--accent);color:var(--white);box-shadow:0 10px 24px rgba(255,77,141,.35)}
    .no{background:#222;color:#fff}
    button:active{transform:translateY(1px)}

    /* Tap to start overlay (for autoplay policies) */
    .gate{position:fixed;inset:0;background:linear-gradient(180deg,rgba(0,0,0,.15),rgba(0,0,0,.45));display:grid;place-items:center;z-index:30}
    .gate .inner{background:#fff;border-radius:18px;padding:18px 20px;text-align:center;box-shadow:0 16px 50px rgba(0,0,0,.2);max-width:90vw}
    .gate h2{margin:.2rem 0 .4rem}

    /* Dark transition @ 6s */
    .cut{position:fixed;inset:0;background:#0a0a0a;opacity:0;pointer-events:none;transition:opacity .6s ease;z-index:20}
    .cut.on{opacity:1}

    /* Subtle footer */
    .ft{position:fixed;left:12px;bottom:10px;font-size:12px;opacity:.7}
  </style>
</head>
<body>
  <!-- Music (replace src with your mp3 if you want) -->
  <audio id="bgm" preload="auto">
    <source src="bensound-romantic.mp3" type="audio/mpeg">
    <!-- <source src="proposal.mp3" type="audio/mpeg"> -->
  </audio>

  <!-- Dark cut for 6s mark -->
  <div class="cut" id="cut"></div>

  <!-- Floating hearts -->
  <div class="hearts" id="hearts"></div>

  <div class="wrap">
    <div class="card">
      <div class="sub">From me to you…</div>
      <div class="title">I have something to say 💗</div>
      <div class="big"><span id="tw1" class="tw"></span></div>
      <p class="sub" id="tw2" style="border-right:2px solid transparent"></p>

      <div class="cta">
        <button class="yes" id="yes">Yes! 💖</button>
        <button class="no" id="no">Umm… No 😅</button>
      </div>
    </div>
  </div>

  <div class="ft">Made with ❤ — customize text, timing & music.</div>
  <div class="gate" id="gate">
    <div class="inner">
      <h2>Tap to start the proposal</h2>
      <button id="start">Start 💌</button>
    </div>
<script>
    // —— CONFIG you can tweak ——
    const TOTAL_DURATION = 30000; // 30s like your clip
    const DARK_AT_MS = 6000;      // ~6s black cut
    const DARK_LEN_MS = 900;      // ~0.9s cut

    const LINE1 = "Will you be my Day-Maker?" // main proposal
    const LINE2 = "I promise pizza, memes, and forever respawns 🍕❤️"; // sub line

    // —— Typewriter ——
    function typeText(el, text, speed=65){
      return new Promise(resolve =>{
        el.textContent = "";
        let i=0;
        const t = setInterval(()=>{
          el.textContent += text[i++];
          if(i>=text.length){clearInterval(t); resolve();}
        }, speed);
      });
    }

    // —— Floating hearts generator ——
    function spawnHearts(){
      const box = document.getElementById('hearts');
      const make = ()=>{
        const e = document.createElement('div');
        e.className = 'heart';
        e.style.setProperty('--x', Math.random()*100+"vw");
        e.style.setProperty('--sz', (Math.random()*3+1)+"rem");
        e.style.setProperty('--dur', (Math.random()*6+10)+"s");
        e.style.setProperty('--delay', (Math.random()*5)+"s");
        e.textContent = Math.random()>.2 ? '❤️' : '💗';
        box.appendChild(e);
        setTimeout(()=> e.remove(), 18000);
      };
      for(let i=0;i<24;i++) make();
      setInterval(make, 700);
    }

    // —— Dark cut timing ——
    function scheduleCut(){
      const cut = document.getElementById('cut');
      setTimeout(()=>{cut.classList.add('on'); setTimeout(()=>cut.classList.remove('on'), DARK_LEN_MS);}, DARK_AT_MS);
    }

    // —— Buttons behavior ——
    const yesBtn = document.getElementById('yes');
    const noBtn  = document.getElementById('no');
    yesBtn.addEventListener('click', ()=>{
      yesBtn.textContent = 'Yaaay! 💍';

    });
    
    noBtn.addEventListener('mouseenter', ()=>{
      // run away button
      const x = Math.random()*60 - 30; // -30 to 30vw
      const y = Math.random()*40 - 20; // -20 to 20vh
      noBtn.style.transform = `translate(${x}vw, ${y}vh)`;
    });

    // —— Start gate ——
    const gate = document.getElementById('gate');
    document.getElementById('start').addEventListener('click', async ()=>{
      gate.remove();
      spawnHearts();
      scheduleCut();

      // Typewriter sequence
      const t1 = document.getElementById('tw1');
      const t2 = document.getElementById('tw2');
      await typeText(t1, LINE1, 70);
      await new Promise(r=>setTimeout(r, 600));
      await typeText(t2, LINE2, 24);

      // Music (if you added a source)
      const audio = document.getElementById('bgm');
      try{ await audio.play(); }catch(e){}

      // Auto end after TOTAL_DURATION (optional)
      setTimeout(()=>{
        // You can redirect or show a final message
        // alert('The End 💖');
      }, TOTAL_DURATION);
    });
  </script>
</body>
</html>
