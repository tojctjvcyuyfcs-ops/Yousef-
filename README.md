<!doctype html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>مسابقة — اربح الجائزة</title>
  <style>
    :root{--bg:#f5f7fb;--card:#fff;--accent:#1a73e8;--muted:#6b7280}
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter, Tahoma, Arial, sans-serif;background:linear-gradient(180deg,var(--bg),#eef3fb);color:#111}
    .container{max-width:920px;margin:36px auto;padding:24px}

    /* card */
    .card{background:var(--card);border-radius:14px;box-shadow:0 14px 40px rgba(20,30,60,0.08);overflow:hidden}

    /* hero */
    .hero{display:flex;gap:24px;align-items:center;padding:28px}
    .gift{flex:0 0 180px;padding:10px}
    .gift img{width:100%;border-radius:10px}
    .lead{flex:1}
    h1{margin:0;font-size:20px}
    p.lead-desc{margin:8px 0 0;color:var(--muted)}

    /* form area */
    .form-area{padding:20px;border-top:1px solid #f1f5f9}
    label{display:block;margin-bottom:8px;font-weight:600}
    input[type=email]{width:100%;padding:12px 14px;border-radius:10px;border:1px solid #e6eefc;font-size:15px}
    .row{display:flex;gap:12px;margin-top:12px}
    button{background:var(--accent);color:#fff;border:0;padding:12px 16px;border-radius:10px;font-weight:600;cursor:pointer}
    button.ghost{background:transparent;color:var(--accent);border:1px solid rgba(26,115,232,0.12)}

    /* after signup */
    .after{display:none;padding:20px;border-top:1px solid #f1f5f9}
    .share-box{display:flex;gap:12px;align-items:center}
    .share-link{flex:1;padding:10px;border-radius:8px;border:1px dashed #dbeafe;background:#fbfeff}
    .share-actions{display:flex;gap:8px}
    .stat{margin-top:12px;color:var(--muted)}

    /* footer small */
    .small{font-size:13px;color:var(--muted);padding:16px;text-align:center}

    /* responsive */
    @media (max-width:720px){
      .hero{flex-direction:column;align-items:stretch}
      .gift{flex:unset}
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="card">
      <div class="hero">
        <div class="gift">
          <img src="data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='600' height='400'><rect fill='%231a73e8' width='600' height='400'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-family='Tahoma' font-size='40' fill='white'>هدية!</text></svg>" alt="gift">
        </div>
        <div class="lead">
          <h1>شارك واربح — فرصة للفوز بجائزة قيمة</h1>
          <p class="lead-desc">أدخل بريدك الإلكتروني للمشاركة. بعد التسجيل، شارك رابطك مع 5 مجموعات لزيادة فرصك.</p>
        </div>
      </div>

      <div class="form-area">
        <label for="email">بريدك الإلكتروني</label>
        <input id="email" type="email" placeholder="name@example.com" aria-label="email">
        <div class="row">
          <button id="join">سجّل دخولي</button>
          <button class="ghost" id="terms">سياسة الخصوصية</button>
        </div>
        <p id="msg" style="color:crimson;margin-top:10px;display:none"></p>
      </div>

      <div class="after" id="after">
        <p style="margin:0;font-weight:700">ممتاز! أنت مسجل.</p>
        <p class="stat">شارك هذا الرابط مع 5 مجموعات أو أصدقاء لزيادة فرصك:</p>
        <div class="share-box">
          <input class="share-link" id="shareLink" readonly>
          <div class="share-actions">
            <button id="copy">نسخ</button>
            <button id="wa">واتساب</button>
          </div>
        </div>
        <p class="stat" id="countInfo">عدد الزيارات من رابطك: 0</p>
      </div>

      <div class="small">لن نطلب أو نخزن كلمة المرور أبداً. سيتم استخدام البريد فقط للتواصل والإعلان عن الفائز.</div>

    </div>
  </div>

  <script>
    // SAFE contest script
    const joinBtn = document.getElementById('join');
    const emailInput = document.getElementById('email');
    const msg = document.getElementById('msg');
    const after = document.getElementById('after');
    const shareLinkEl = document.getElementById('shareLink');
    const countInfo = document.getElementById('countInfo');

    // generate simple ref
    function genRef(){
      return Math.random().toString(36).slice(2,9).toUpperCase();
    }

    function saveToServer(payload){
      // placeholder: try POST to /api/signup (you can set up a server later)
      fetch('/api/signup', {method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(payload)}).catch(()=>{
        // fallback: save locally (development)
        const pending = JSON.parse(localStorage.getItem('pending')||'[]');
        pending.push(payload);
        localStorage.setItem('pending', JSON.stringify(pending));
      });
    }

    joinBtn.addEventListener('click', ()=>{
      const email = emailInput.value.trim();
      if(!email || !email.includes('@')){ msg.style.display='block'; msg.textContent='ادخل بريد إلكتروني صالح'; return; }
      msg.style.display='none';

      const userRef = genRef();
      const sourceRef = new URLSearchParams(location.search).get('ref') || '';
      const payload = {email, userRef, sourceRef, ts: new Date().toISOString()};
      saveToServer(payload);

      // show after area
      after.style.display='block';
      const myLink = location.origin + location.pathname + '?ref=' + userRef;
      shareLinkEl.value = myLink;
      localStorage.setItem('myRef', userRef);
      localStorage.setItem('shareCount', '0');
      updateCount();

      // friendly UX
      joinBtn.textContent = 'تم التسجيل'; joinBtn.disabled = true;
    });

    // share actions
    document.getElementById('copy').addEventListener('click', ()=>{
      shareLinkEl.select(); document.execCommand('copy'); alert('تم نسخ الرابط');
    });
    document.getElementById('wa').addEventListener('click', ()=>{
      const link = encodeURIComponent(shareLinkEl.value);
      window.open('https://wa.me/?text=' + link, '_blank');
    });

    // track visits (demo-only): when someone opens with ?ref=XYZ increment local counter
    (function trackVisit(){
      const vRef = new URLSearchParams(location.search).get('ref');
      if(!vRef) return;
      const myRef = localStorage.getItem('myRef');
      if(myRef && vRef === myRef) return; // owner visiting own link
      let count = parseInt(localStorage.getItem('shareCount')||'0',10);
      localStorage.setItem('shareCount', String(count+1));
      updateCount();
    })();

    function updateCount(){
      const c = localStorage.getItem('shareCount')||'0';
      countInfo.textContent = 'عدد الزيارات من رابطك (تجريبي): ' + c;
    }

  </script>
</body>
</html>
