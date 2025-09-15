<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Monyet Lucu — Klik buat aksi</title>
  <style>
    :root{
      --bg:#f7f3e9;
      --card:#fff;
      --accent:#ffca28;
      --brown:#7b4f30;
      --light:#f0d7b0;
    }
    *{box-sizing:border-box}
    body{
      margin:0;
      min-height:100vh;
      display:flex;
      align-items:center;
      justify-content:center;
      background:
        radial-gradient(circle at 20% 20%, #fff8e0, transparent 20%),
        linear-gradient(180deg,var(--bg),#e8e4da 100%);
      font-family:system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial;
      color:#222;
    }

    .wrap{
      width:320px;
      background:var(--card);
      border-radius:20px;
      box-shadow:0 18px 40px rgba(20,20,20,0.12);
      padding:22px;
      text-align:center;
    }

    h1{font-size:20px;margin:0 0 8px}
    p{font-size:13px;margin:0 0 14px;color:#555}

    /* scene */
    .scene{
      width:220px;
      height:220px;
      margin:0 auto 12px;
      position:relative;
      overflow:visible;
      transform-origin:center;
      animation: swing 3s ease-in-out infinite;
    }

    @keyframes swing{
      0%{transform:rotate(-6deg)}
      50%{transform:rotate(6deg)}
      100%{transform:rotate(-6deg)}
    }

    /* simple banana drop */
    .banana{
      position:absolute;
      left:calc(50% + 64px);
      top:-40px;
      width:36px;
      height:36px;
      transform-origin:center;
      cursor:pointer;
      transition:transform .18s;
    }
    .banana:active{transform:scale(.9) rotate(-10deg)}

    /* caption button */
    .btn{
      display:inline-block;
      margin-top:8px;
      background:var(--accent);
      color:#3b2b15;
      padding:8px 12px;
      border-radius:12px;
      font-weight:600;
      font-size:13px;
      text-decoration:none;
      border:none;
      cursor:pointer;
      box-shadow:0 6px 14px rgba(0,0,0,.12);
    }

    small{display:block;margin-top:8px;color:#888}

    /* optional: reduce motion for users */
    @media (prefers-reduced-motion: reduce){
      .scene, @keyframes swing { animation: none !important; transform:none !important; }
    }
  </style>
</head>
<body>
  <div class="wrap">
    <h1>Monyet Lucu</h1>
    <p>Klik pisang untuk turun — monyetnya bakal kaget dan berkedip!</p>

    <div class="scene" id="scene" aria-hidden="true">
      <!-- monkey SVG (inline, so nggak perlu file) -->
      <svg viewBox="0 0 220 220" width="220" height="220" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Monyet lucu">
        <!-- ears -->
        <g id="ears">
          <circle cx="46" cy="80" r="28" fill="#6f4b2e"/>
          <circle cx="174" cy="80" r="28" fill="#6f4b2e"/>
          <circle cx="46" cy="80" r="18" fill="#f1d6b1"/>
          <circle cx="174" cy="80" r="18" fill="#f1d6b1"/>
        </g>

        <!-- head -->
        <g id="head">
          <ellipse cx="110" cy="104" rx="78" ry="78" fill="#7b4f30"/>
          <ellipse cx="110" cy="120" rx="50" ry="44" fill="#f1d6b1"/>
        </g>

        <!-- eyes group for blinking -->
        <g id="eyes">
          <g id="left-eye" transform="translate(-18,0)">
            <circle cx="88" cy="106" r="10" fill="#222"/>
            <circle cx="86" cy="104" r="3" fill="#fff"/>
          </g>
          <g id="right-eye" transform="translate(18,0)">
            <circle cx="132" cy="106" r="10" fill="#222"/>
            <circle cx="130" cy="104" r="3" fill="#fff"/>
          </g>
          <!-- eyelids (we'll animate opacity to simulate blink) -->
          <rect id="l-closed" x="74" y="98" width="28" height="16" rx="8" fill="#7b4f30" opacity="0"></rect>
          <rect id="r-closed" x="124" y="98" width="28" height="16" rx="8" fill="#7b4f30" opacity="0"></rect>
        </g>

        <!-- nose & mouth -->
        <g>
          <ellipse cx="110" cy="136" rx="12" ry="8" fill="#8a6243"/>
          <path d="M94 146 Q110 156 126 146" stroke="#8a6243" stroke-width="4" fill="transparent" stroke-linecap="round"/>
        </g>

        <!-- cheeks -->
        <circle cx="82" cy="132" r="6" fill="#ffb0a6" opacity="0.9"/>
        <circle cx="138" cy="132" r="6" fill="#ffb0a6" opacity="0.9"/>

        <!-- little tuft -->
        <path d="M110 58 q8 8 16 2" stroke="#4b2f1b" stroke-width="4" stroke-linecap="round" fill="none"/>
        <path d="M110 58 q-8 8 -16 2" stroke="#4b2f1b" stroke-width="4" stroke-linecap="round" fill="none"/>
      </svg>

      <!-- banana icon (inline SVG) -->
      <svg class="banana" id="banana" viewBox="0 0 64 64" width="44" height="44" title="Pisah">
        <g>
          <path d="M4 44c10-12 28-28 48-32" fill="none" stroke="#f7d26b" stroke-width="10" stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M6 40c7-8 20-20 38-26" fill="none" stroke="#ffeb99" stroke-width="6" stroke-linecap="round"/>
          <circle cx="54" cy="12" r="3" fill="#6b4b2b"/>
        </g>
      </svg>
    </div>

    <button class="btn" id="surpriseBtn">Bikin Monyet Kaget</button>
    <small>Atau klik pisang.</small>
  </div>

  <script>
    const banana = document.getElementById('banana');
    const lClosed = document.getElementById('l-closed');
    const rClosed = document.getElementById('r-closed');
    const scene = document.getElementById('scene');
    const btn = document.getElementById('surpriseBtn');

    // blink function: animate eyelids quickly
    function blink(duration = 220){
      lClosed.style.transition = `opacity ${duration/3}ms ease`;
      rClosed.style.transition = `opacity ${duration/3}ms ease`;
      lClosed.style.opacity = 1;
      rClosed.style.opacity = 1;
      setTimeout(()=> {
        lClosed.style.opacity = 0;
        rClosed.style.opacity = 0;
      }, duration/2);
    }

    // small head bounce / surprise
    function surprise(){
      // quick scale + rotate
      scene.animate([
        { transform: 'scale(1) rotate(0deg)' },
        { transform: 'scale(1.06) rotate(-8deg)' },
        { transform: 'scale(1) rotate(0deg)' }
      ], { duration: 420, easing: 'cubic-bezier(.2,.8,.2,1)' });
      blink(240);
    }

    // banana drop animation (banana falls then bounces)
    function dropBanana(){
      const topBefore = banana.getBoundingClientRect().top;
      banana.animate([
        { transform: 'translateY(0) rotate(0deg)' },
        { transform: 'translateY(120px) rotate(30deg)' },
        { transform: 'translateY(100px) rotate(-10deg)' }
      ], { duration: 700, easing:'cubic-bezier(.2,.8,.2,1)' })
      .onfinish = () => {
        surprise();
        // return banana to original spot with little delay
        setTimeout(()=> banana.style.transform = 'translateY(0)', 380);
      };
    }

    // events
    banana.addEventListener('click', dropBanana);
    btn.addEventListener('click', surprise);

    // random blink every 3-6 seconds
    setInterval(()=> {
      if (Math.random() > 0.4) blink(160);
    }, 3000 + Math.random()*3000);
  </script>
</body>
</html>
