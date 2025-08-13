<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Link Helper — Paste URL, get safe options</title>
  <meta name="description" content="A safe, original landing page inspired by link-helper sites. It validates a URL and offers compliant options (open, copy with fallback, fetch metadata via oEmbed if you wire an API).">
  <style>
    :root{--bg:#0b1220;--ink:#0f1723;--muted:#6b7280;--card:#ffffff;--accent:#10b981;--accent2:#3b82f6;--warn:#f59e0b;--danger:#ef4444}
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter,ui-sans-serif,system-ui,-apple-system,"Segoe UI",Roboto;line-height:1.45;color:var(--ink);background:radial-gradient(1200px 600px at 10% -10%,#1b2a52 0%,#0b1220 40%,#0b1220 100%)}
    .wrap{max-width:980px;margin:0 auto;padding:28px}
    .nav{display:flex;align-items:center;justify-content:space-between;color:#e8f1ff}
    .brand{display:flex;align-items:center;gap:10px;font-weight:800}
    .brand .mark{width:28px;height:28px;border-radius:8px;background:linear-gradient(135deg,var(--accent),var(--accent2))}

    .hero{margin-top:24px;display:grid;grid-template-columns:1fr 360px;gap:18px;align-items:start}
    .card{background:var(--card);border-radius:14px;box-shadow:0 10px 30px rgba(0,0,0,0.25)}
    .panel{padding:18px}
    .title{font-size:28px;margin:0;color:#eaf2ff}
    .subtitle{color:#b6c5ff;margin-top:6px}
    .input{margin-top:14px;display:flex;gap:10px}
    input[type="url"],input[type="text"],input.readonly{flex:1;padding:12px 14px;border-radius:10px;border:1px solid #e6e9ef;font-size:15px}
    input.readonly{background:#f8fafc}
    button{border:0;border-radius:10px;padding:12px 14px;font-weight:700;cursor:pointer}
    .go{background:var(--accent);color:white}
    .clear{background:#0f1723;color:white}

    .result{display:none;margin-top:12px}
    .row{display:flex;gap:10px;flex-wrap:wrap}
    .pill{display:inline-flex;align-items:center;gap:6px;padding:6px 10px;border-radius:999px;background:#eef7ff;color:#1e293b;font-size:13px}

    .copy-fallback{display:none;gap:8px;align-items:center;background:#fff7ed;border:1px solid #fed7aa;padding:10px;border-radius:10px}
    .copy-fallback .note{font-size:13px;color:#7c2d12}

    .side{padding:0}
    .side .panel{border-bottom:1px solid #eef2f6}
    .side h3{margin:0 0 8px}
    .muted{color:var(--muted)}
    code{background:#0b1220;color:#e6f2ff;padding:2px 6px;border-radius:6px}

    footer{margin-top:18px;color:#91a3ff;text-align:center;font-size:13px}

    .toast{position:fixed;left:16px;bottom:16px;background:#111827;color:#f9fafb;padding:10px 12px;border-radius:10px;box-shadow:0 10px 30px rgba(0,0,0,0.35);opacity:0;transform:translateY(8px);transition:.2s ease}
    .toast.show{opacity:1;transform:translateY(0)}

    details{background:rgba(255,255,255,0.06);border-radius:12px}
    summary{cursor:pointer;padding:10px 12px;color:#b6c5ff}
    .tests{padding:0 12px 12px}
    .test-pass{color:#10b981;font-weight:700}
    .test-info{color:#f59e0b}

    @media (max-width:900px){.hero{grid-template-columns:1fr}.wrap{padding:20px}}
  </style>
</head>
<body>
  <div class="wrap">
    <div class="nav">
      <div class="brand"><span class="mark" aria-hidden></span><span>LinkHelper</span></div>
      <div class="muted">Original demo • for learning</div>
    </div>

    <div class="hero">
      <section class="card panel" aria-label="URL tools">
        <h1 class="title">Paste a link. Get safe options.</h1>
        <div class="subtitle">Validates a URL and shows compliant actions (open, copy with fallback, attempt oEmbed via your server). No scraping. No DRM bypass.</div>

        <div class="input">
          <input id="url" type="url" placeholder="https://example.com/your-video-or-article" />
          <button class="go" id="analyze">Analyze</button>
          <button class="clear" id="reset">Clear</button>
        </div>

        <div id="result" class="result" aria-live="polite">
          <h3 style="margin:12px 0 8px">Detected</h3>
          <div class="row" id="badges"></div>

          <div class="row" style="margin-top:10px">
            <button id="open" class="go">Open link</button>
            <button id="copy">Copy link</button>
            <button id="meta" style="background:#3b82f6;color:white">Get metadata (via API)</button>
          </div>

          <div id="copyFallback" class="copy-fallback" role="region" aria-label="Copy fallback">
            <span class="note">Copy blocked by browser policy. Press Ctrl+C / Cmd+C after selecting.</span>
            <input id="copyInput" class="readonly" type="text" readonly value="" aria-label="Copy this text manually" />
            <button id="selectText" type="button" title="Select text">Select</button>
          </div>

          <div id="note" class="muted" style="margin-top:10px;font-size:13px"></div>

          <div id="metaOut" style="margin-top:12px;display:none">
            <label><strong>Metadata</strong> (demo JSON)</label>
            <pre id="metaPre" style="background:#0b1220;color:#e6f2ff;padding:12px;border-radius:10px;overflow:auto">{}</pre>
          </div>
        </div>
      </section>

      <aside class="card side" aria-label="Guidelines">
        <div class="panel">
          <h3>What this page does</h3>
          <ul class="muted">
            <li>Checks if the link is a valid <code>http(s)</code> URL</li>
            <li>Shows the domain and basic parsing</li>
            <li>Provides compliant actions: open, copy (with fallback), optional server-side metadata fetch</li>
          </ul>
        </div>
        <div class="panel">
          <h3>What it <em>won't</em> do</h3>
          <ul class="muted">
            <li>No downloading from sites that prohibit it in their Terms of Service</li>
            <li>No DRM or paywall bypass, no scraping of protected content</li>
          </ul>
        </div>
        <div class="panel">
          <h3>Optional backend (if you add one)</h3>
          <p class="muted">You can add a tiny server endpoint that uses official platform APIs (e.g., oEmbed for supported providers) to return public metadata like title/thumbnail. Example contract:</p>
          <pre style="white-space:pre-wrap">POST /api/oembed\n{ url: "https://..." } → returns { title, author, thumbnail, provider, width, height }</pre>
        </div>
        <div class="panel">
          <h3>Direct-download rule of thumb</h3>
          <p class="muted">This front-end only attempts downloads for links that are already public files (e.g., <code>.mp4</code>, <code>.zip</code>) and allow cross-origin access. Streaming pages (e.g., YouTube/TikTok/etc.) are <strong>not</strong> direct files and usually disallow this.</p>
        </div>
        <div class="panel">
          <details>
            <summary>Self‑tests (click to expand)</summary>
            <div class="tests">
              <button id="runTests" type="button">Run tests</button>
              <ul id="testList" class="muted"></ul>
            </div>
          </details>
        </div>
      </aside>
    </div>

    <footer>
      © <span id="year"></span> LinkHelper — Educational UI. Please respect each site's Terms of Service.
    </footer>
  </div>

  <div id="toast" class="toast" role="status" aria-live="polite"></div>

  <script>
  'use strict';
    const urlEl = document.getElementById('url');
    const analyzeBtn = document.getElementById('analyze');
    const resetBtn = document.getElementById('reset');
    const result = document.getElementById('result');
    const badges = document.getElementById('badges');
    const openBtn = document.getElementById('open');
    const copyBtn = document.getElementById('copy');
    const metaBtn = document.getElementById('meta');
    const note = document.getElementById('note');
    const metaOut = document.getElementById('metaOut');
    const metaPre = document.getElementById('metaPre');
    const copyFallback = document.getElementById('copyFallback');
    const copyInput = document.getElementById('copyInput');
    const selectTextBtn = document.getElementById('selectText');
    const toast = document.getElementById('toast');

    document.getElementById('year').textContent = new Date().getFullYear();

    function showToast(msg){
      toast.textContent = msg;
      toast.classList.add('show');
      setTimeout(()=> toast.classList.remove('show'), 2000);
    }

    function parse(u){
      try{ return new URL(u); }catch{ return null; }
    }

    function makePill(label){
      const span = document.createElement('span');
      span.className = 'pill';
      span.textContent = label;
      return span;
    }

    // Robust copy helper with fallbacks that won't throw in sandboxed environments
    async function safeCopy(text){
      // Attempt Permissions API hint (optional)
      try{
        if (navigator.permissions && navigator.permissions.query) {
          navigator.permissions.query({name:'clipboard-write'}).catch(()=>{});
        }
      }catch(_){}

      // Primary: modern Clipboard API (may be blocked by permissions policy)
      try{
        if (navigator.clipboard && navigator.clipboard.writeText) {
          await navigator.clipboard.writeText(text);
          return {ok:true, method:'clipboard'};
        }
      } catch(err){
        // Swallow NotAllowedError and fall through to legacy path
      }

      // Fallback: execCommand('copy') using a temporary textarea
      try{
        const ta = document.createElement('textarea');
        ta.value = text;
        ta.setAttribute('readonly','');
        ta.style.position = 'fixed';
        ta.style.top = '-1000px';
        document.body.appendChild(ta);
        ta.select();
        const ok = document.execCommand && document.execCommand('copy');
        document.body.removeChild(ta);
        if (ok) return {ok:true, method:'execCommand'};
      }catch(_){ }

      // Final fallback: show manual copy UI
      return {ok:false, method:'manual'};
    }

    function showManualCopyUI(text){
      copyInput.value = text;
      copyFallback.style.display = 'flex';
      // auto-select to help user
      setTimeout(()=>{ copyInput.focus(); copyInput.select(); }, 0);
    }

    function hideManualCopyUI(){
      copyFallback.style.display = 'none';
      copyInput.value = '';
    }

    function analyze(){
      hideManualCopyUI();
      const raw = urlEl.value.trim();
      const u = parse(raw);
      badges.innerHTML = '';
      metaOut.style.display = 'none';
      metaPre.textContent = '{}';

      if(!u){
        result.style.display = 'none';
        alert('Please enter a valid http(s) URL');
        return;
      }

      result.style.display = 'block';
      badges.append(
        makePill('Protocol: '+u.protocol.replace(':','')),
        makePill('Domain: '+u.hostname),
        makePill('Path: '+(u.pathname||'/')),
        makePill('Query: '+(u.search ? 'yes' : 'no')),
      );

      const ext = u.pathname.split('.').pop().toLowerCase();
      const directFile = ['mp4','mov','mp3','wav','zip','pdf','png','jpg','jpeg','webp','gif'].includes(ext);
      const isBigPlatform = /(youtube|youtu\.be|tiktok|facebook|instagram|twitter|x\.com)/i.test(u.hostname);

      let guidance = '';
      if(isBigPlatform){
        guidance = 'This looks like a streaming/social platform URL. Use the platform\'s share or embed tools. Downloading may violate Terms of Service.';
      } else if(directFile){
        guidance = 'Direct file detected. Your browser may download it if the server allows cross-origin requests.';
      } else {
        guidance = 'Standard webpage detected. You can open it or use your own backend to fetch public metadata.';
      }
      note.textContent = guidance;

      openBtn.onclick = ()=> window.open(u.toString(),'_blank');
      copyBtn.onclick = async ()=>{
        const text = u.toString();
        const res = await safeCopy(text);
        if(res.ok){
          hideManualCopyUI();
          showToast('Link copied to clipboard ('+res.method+')');
        } else {
          showManualCopyUI(text);
          showToast('Copy blocked by browser policy — use the manual fallback');
        }
      };

      metaBtn.onclick = async ()=>{
        // Demo-only: show mock JSON. In production, call your own compliant endpoint.
        const mock = {
          provider: u.hostname,
          title: 'Example title (mock)',
          author: 'Unknown',
          thumbnail: null,
          url: u.toString()
        };
        metaPre.textContent = JSON.stringify(mock,null,2);
        metaOut.style.display = 'block';
      };
    }

    analyzeBtn.addEventListener('click', analyze);
    urlEl.addEventListener('keydown', e=>{ if(e.key==='Enter'){ e.preventDefault(); analyze(); }});
    resetBtn.addEventListener('click', ()=>{ urlEl.value=''; result.style.display='none'; hideManualCopyUI(); });

    // Manual copy helper
    selectTextBtn.addEventListener('click', ()=>{ copyInput.focus(); copyInput.select(); showToast('Text selected — press Ctrl+C / Cmd+C'); });

    // ---- Self tests ----
    const runTestsBtn = document.getElementById('runTests');
    const testList = document.getElementById('testList');
    function addTestResult(text, ok){
      const li = document.createElement('li');
      li.innerHTML = ok ? `<span class="test-pass">PASS</span> — ${text}` : `<span style="color:var(--danger);font-weight:700">FAIL</span> — ${text}`;
      testList.appendChild(li);
    }
    function clearTests(){ testList.innerHTML=''; }

    async function runTests(){
      clearTests();
      // Test 1: parse valid URL
      addTestResult('parse("https://example.com/a?b=c") returns URL', !!parse('https://example.com/a?b=c'));
      // Test 2: parse invalid URL
      addTestResult('parse("not a url") returns null', parse('not a url')===null);
      // Test 3: pill maker
      const pill = makePill('Hello');
      addTestResult('makePill returns span.pill', pill && pill.tagName==='SPAN' && pill.classList.contains('pill'));
      // Test 4: safeCopy does not throw and returns result object
      let copyOk=false; let res=null;
      try{ res = await safeCopy('test'); copyOk = typeof res=== 'object' && 'ok' in res; }catch(e){ copyOk=false; }
      addTestResult('safeCopy returns {ok, method} and no uncaught error', copyOk);
      // Test 5: direct-file detection logic
      const u = new URL('https://cdn.example.com/file.mp4');
      const ext = u.pathname.split('.').pop().toLowerCase();
      const directFile = ['mp4','mov','mp3','wav','zip','pdf','png','jpg','jpeg','webp','gif'].includes(ext);
      addTestResult('direct-file detection works for .mp4', directFile===true);
      showToast('Self-tests finished');
    }
    if(runTestsBtn){ runTestsBtn.addEventListener('click', runTests); }
  </script>
</body>
</html>
