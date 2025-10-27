# real-time-checking-teddy
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>TeddySense — Interactive Demo</title>
    <style>
      :root{--bg:#FFF7FB;--card:#FFFFFF;--muted:#6b6b6b;--accent:#FF92C2}
      html,body{height:100%;margin:0;font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial}
      body{display:flex;align-items:center;justify-content:center;background:linear-gradient(135deg,#FFF6F9,#F0FDFF);padding:28px}
      .card{width:940px;max-width:100%;background:var(--card);border-radius:18px;padding:18px;box-shadow:0 20px 60px rgba(60,40,70,0.06);display:grid;grid-template-columns:360px 1fr;gap:18px}
      .teddy{background:linear-gradient(180deg,#FFDAB8,#F4A876);border-radius:16px;padding:18px;display:flex;flex-direction:column;align-items:center;gap:12px;position:relative}
      .face{width:260px;height:260px;border-radius:50%;background:linear-gradient(180deg,#F7CBA1,#D98C5A);position:relative;box-shadow:0 6px 24px rgba(0,0,0,0.08)}
      .eye{position:absolute;top:90px;width:34px;height:34px;background:#222;border-radius:50%;box-shadow:0 4px 10px rgba(0,0,0,0.12)}
      .eye.left{left:72px}
      .eye.right{right:72px}
      .eye::after{content:'';position:absolute;left:6px;top:6px;width:10px;height:10px;background:rgba(255,255,255,0.95);border-radius:50%}
      .nose{position:absolute;top:140px;left:50%;transform:translateX(-50%);width:28px;height:20px;background:#241818;border-radius:50%}
      .mouth{position:absolute;top:166px;left:50%;transform:translateX(-50%);width:76px;height:36px;border-radius:0 0 36px 36px;border:3px solid rgba(0,0,0,0.9);background:transparent}
      .tongue{position:absolute;top:186px;left:50%;transform:translateX(-50%);width:28px;height:18px;background:#FF9ACB;border-radius:12px}
      .controls{display:flex;flex-direction:column;gap:8px;width:100%}
      .btn{padding:10px 12px;border-radius:12px;border:none;cursor:pointer;font-weight:700}
      .btn.primary{background:linear-gradient(90deg,#FFD8E6,#FFEFD6);color:#2b2b2b}
      .btn.ghost{background:transparent;border:2px dashed rgba(0,0,0,0.06)}
      .panel{padding:12px}
      .speech{background:#FFF8FB;border-radius:12px;padding:12px;margin-top:8px;font-weight:700}
      .stats{display:flex;gap:8px;flex-wrap:wrap;margin-top:10px}
      .stat{background:#fff;border-radius:10px;padding:8px;border:1px solid #EFEFEF;font-weight:700;color:var(--muted)}
      .hearts{position:absolute;top:-12px;left:50%;transform:translateX(-50%);pointer-events:none}
      .heart{width:20px;height:20px;background:linear-gradient(90deg,#FF92C2,#FF5C8A);clip-path:polygon(50% 0, 61% 12%, 75% 20%, 92% 35%, 92% 60%, 75% 78%, 50% 98%, 25% 78%, 8% 60%, 8% 35%, 25% 20%, 39% 12%);opacity:0;position:absolute}
      @keyframes floatUp{0%{transform:translateY(10px) scale(.7);opacity:0}30%{opacity:1}100%{transform:translateY(-140px) scale(1);opacity:0}}
      .heart.show{animation:floatUp 1400ms ease-out forwards}
      .confetti{position:absolute;right:8px;top:8px;width:140px;height:100px;pointer-events:none}
      /* responsive */
      @media(max-width:980px){.card{grid-template-columns:1fr} .teddy{order:0}}
    </style>
  </head>
  <body>
    <div class="card">
      <div class="teddy" id="teddyCol">
        <div class="hearts" id="hearts"></div>
        <div class="face" id="face">
          <div class="eye left"></div>
          <div class="eye right"></div>
          <div class="nose"></div>
          <div class="mouth"></div>
          <div class="tongue"></div>
        </div>
        <div class="controls">
          <button class="btn primary" id="moodHappy">😊 Make Teddy Happy</button>
          <button class="btn" id="moodCalm">🌬️ Breathe With Teddy</button>
          <div style="display:flex;gap:8px">
            <button class="btn ghost" id="toggleVoice">🔊 Voice: On</button>
            <button class="btn ghost" id="confettiBtn">🎉 Confetti</button>
          </div>
        </div>
      </div>
      <div class="panel">
        <h2 style="margin-top:0">TeddySense — Interactive</h2>
        <div class="speech" id="speech">Hi! Click a button and I'll respond with voice, chime, and magic ✨</div>
        <div class="stats" id="stats">
          <div class="stat">Interactions: <span id="count">0</span></div>
          <div class="stat">Last: <span id="last">—</span></div>
        </div>
        <canvas class="confetti" id="confettiCanvas"></canvas>
      </div>
    </div>

    <script>
      // tiny interactive behaviors
      const speechEl = document.getElementById('speech');
      const countEl = document.getElementById('count');
      const lastEl = document.getElementById('last');
      const hearts = document.getElementById('hearts');
      const face = document.getElementById('face');
      let interactions = 0;
      let voiceOn = true;

      function speak(text){ if(!voiceOn) return; if('speechSynthesis' in window){ const u=new SpeechSynthesisUtterance(text); u.rate=1; u.pitch=1.05; window.speechSynthesis.cancel(); window.speechSynthesis.speak(u);} }

      function playChime(){ const a = new (window.AudioContext || window.webkitAudioContext)(); const o=a.createOscillator(); const g=a.createGain(); o.type='sine'; o.frequency.value=660; g.gain.value=0.08; o.connect(g); g.connect(a.destination); o.start(); setTimeout(()=>{ o.frequency.value=780; },140); setTimeout(()=>{ o.stop(); a.close(); },420); }

      function addHeart(){ const h=document.createElement('div'); h.className='heart'; h.style.left = (20 + Math.random()*200) + 'px'; hearts.appendChild(h); setTimeout(()=>h.classList.add('show'),20); setTimeout(()=>h.remove(),1800); }

      function spawnConfetti(){ const c=document.getElementById('confettiCanvas'); const ctx=c.getContext('2d'); c.width=c.clientWidth; c.height=c.clientHeight; const particles=[]; for(let i=0;i<40;i++){ particles.push({x:Math.random()*c.width,y:Math.random()*c.height*0.2,vx:(Math.random()-0.5)*2,vy:Math.random()*3+1,w:4+Math.random()*8,h:6+Math.random()*6,c:['#FFB7D5','#FFF1A8','#B8F7D2','#BFE6FF','#E8C9FF'][Math.floor(Math.random()*5)],r:Math.random()*360}); }
        let t = setInterval(()=>{ ctx.clearRect(0,0,c.width,c.height); particles.forEach(p=>{ p.x+=p.vx; p.y+=p.vy; p.vy+=0.03; ctx.save(); ctx.translate(p.x,p.y); ctx.rotate(p.r*Math.PI/180); ctx.fillStyle=p.c; ctx.fillRect(-p.w/2,-p.h/2,p.w,p.h); ctx.restore(); }); if(particles.every(p=>p.y>c.height+30)){ clearInterval(t); } },16);
      }

      document.getElementById('moodHappy').addEventListener('click', ()=>{
        interactions++; countEl.textContent=interactions; lastEl.textContent='happy'; speechEl.textContent='Yay! Teddy is joyful — giggles incoming!'; speak('Yay! Teddy is joyful. Let us dance!'); playChime(); for(let i=0;i<6;i++) setTimeout(addHeart, i*120); spawnConfetti(); face.animate([{transform:'translateY(0)'},{transform:'translateY(-8px)'},{transform:'translateY(0)'}], {duration:700, iterations:1});
      });

      document.getElementById('moodCalm').addEventListener('click', ()=>{
        interactions++; countEl.textContent=interactions; lastEl.textContent='calm'; speechEl.textContent='Breathe with me — in for 4, hold 4, out 4.'; speak('Breathe in for four seconds, hold, and breathe out.'); const el=face; el.animate([{transform:'scale(1)'},{transform:'scale(1.06)'},{transform:'scale(1)'}], {duration:4000, iterations:3});
      });

      document.getElementById('toggleVoice').addEventListener('click', (e)=>{ voiceOn = !voiceOn; e.currentTarget.textContent = voiceOn? '🔊 Voice: On':'🔇 Voice: Off'; });
      document.getElementById('confettiBtn').addEventListener('click', ()=>{ spawnConfetti(); for(let i=0;i<4;i++) setTimeout(addHeart, i*200); speak('Party time!'); });

      // playful blink
      setInterval(()=>{ const eyes = document.querySelectorAll('.eye'); eyes.forEach(e=>e.animate([{height:'34px'},{height:'6px'},{height:'34px'}],{duration:480,iterations:1})); }, 4200);

      // init message
      speak('Hello! Open this file in a browser to interact with TeddySense.');
    </script>
  </body>
</html>
