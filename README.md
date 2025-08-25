<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>AREXMADO - Minecraft 통합 도구</title>
  <meta name="description" content="마인크래프트 모드/리소스팩/번역 등 통합 도구 웹 인터페이스 (GitHub Pages용 정적 페이지)" />
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--muted:#9aa4b2;--accent:#60a5fa}
    [data-theme="light"]{--bg:#f8fafc;--card:#ffffff;--muted:#55606a;--accent:#2563eb}
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter,Segoe UI,Apple SD Gothic Neo,'Noto Sans KR',sans-serif;background:var(--bg);color:#e6eef8}
    .wrap{max-width:1100px;margin:28px auto;padding:20px}
    header{display:flex;justify-content:space-between;align-items:center;margin-bottom:20px}
    h1{margin:0;font-size:20px}
    nav{display:flex;gap:10px;align-items:center}
    a.btn{background:var(--card);padding:8px 12px;border-radius:10px;text-decoration:none;color:inherit;font-weight:600}
    main{display:grid;grid-template-columns:1fr 360px;gap:18px}
    .card{background:var(--card);padding:16px;border-radius:12px;box-shadow:0 6px 18px rgba(2,6,23,0.6)}
    .tools{display:grid;gap:12px}
    .tool-title{font-weight:700;margin-bottom:8px}
    input[type=file]{display:block;margin-top:8px}
    .log{height:180px;overflow:auto;background:rgba(255,255,255,0.02);padding:8px;border-radius:8px;font-family:monospace;font-size:13px;color:var(--muted)}
    footer{margin-top:18px;color:var(--muted);font-size:13px;text-align:center}
    /* simple responsive */
    @media(max-width:900px){main{grid-template-columns:1fr}}
    .small{font-size:13px;color:var(--muted)}
    .link{color:var(--accent);cursor:pointer}
    .pill{display:inline-block;padding:6px 10px;border-radius:999px;background:rgba(255,255,255,0.03);font-size:13px}
    .input-row{display:flex;gap:8px;align-items:center}
    button.primary{background:linear-gradient(90deg,rgba(96,165,250,0.12),rgba(96,165,250,0.06));border:1px solid rgba(96,165,250,0.18);padding:8px 12px;border-radius:8px;color:var(--accent);cursor:pointer}
  </style>
</head>
<body data-theme="dark">
  <div class="wrap">
    <header>
      <div>
        <h1>AREXMADO - Minecraft 통합 도구</h1>
        <div class="small">모드 번역 · 리소스팩 빌더 · 모드팩 분석 · 다운로드 관리</div>
      </div>
      <nav>
        <a class="btn" href="#tools">도구</a>
        <a class="btn" href="#about">설명</a>
        <a class="btn" href="#community">커뮤니티</a>
        <div style="width:8px"></div>
        <button id="themeToggle" class="btn">라이트 모드</button>
      </nav>
    </header>

    <main>
      <section>
        <div id="tools" class="card tools">
          <div class="card">
            <div class="tool-title">1) 모드 번역기 (.zip/.jar 업로드)</div>
            <div class="small">여러 모드 ZIP/JAR 파일을 업로드하면 내부의 .lang/.json 파일을 탐색하여 번역을 시뮬레이트합니다. (클라이언트-side 데모)</div>
            <div style="margin-top:8px">
              <input id="modUpload" type="file" accept=".zip,.jar" multiple />
              <div class="small">업로드 후 '분석 시작'을 눌러 시뮬레이션합니다. 실제 번역 API 연동은 서버 또는 키가 필요합니다.</div>
              <div style="margin-top:8px"><button id="analyzeMods" class="primary">분석 시작</button></div>
            </div>
          </div>

          <div class="card">
            <div class="tool-title">2) 리소스팩 빌더 (이미지/언어 파일 업로드)</div>
            <div class="small">업로드한 파일로 리소스팩 구조를 만들어 ZIP으로 다운로드합니다. (브라우저에서 생성)</div>
            <div style="margin-top:8px">
              <input id="rpFiles" type="file" webkitdirectory directory multiple />
              <div style="margin-top:8px" class="input-row">
                <input id="packName" placeholder="리소스팩 이름" />
                <button id="buildRp" class="primary">리소스팩 생성</button>
              </div>
            </div>
          </div>

          <div class="card">
            <div class="tool-title">3) 모드팩 분석기</div>
            <div class="small">모드 목록, 충돌 의심 파일, 번역 포함 여부 등을 검사합니다.</div>
            <div style="margin-top:8px">
              <input id="packUpload" type="file" accept=".zip" />
              <div style="margin-top:8px"><button id="analyzePack" class="primary">모드팩 분석</button></div>
            </div>
          </div>

          <div class="card">
            <div class="tool-title">4) 서버/다운로드 매니저 (데모)</div>
            <div class="small">GitHub Releases 또는 개인 서버와 연동하여 다운로드 링크를 보여줍니다. (백엔드 필요)</div>
            <div style="margin-top:8px">
              <div class="pill">GitHub Releases 연동 권장</div>
            </div>
          </div>

        </div>

        <div style="margin-top:12px" class="card">
          <div class="tool-title">실행 로그</div>
          <div id="logArea" class="log">작동 로그가 여기에 표시됩니다.</div>
        </div>
      </section>

      <aside>
        <div class="card">
          <div class="tool-title">설정</div>
          <div class="small">기본 동작을 변경하세요.</div>
          <div style="margin-top:8px">
            <label class="small">번역 엔진: <select id="engine"><option>데모 (로컬 시뮬레이션)</option><option>DeepL (서버 필요)</option><option>Papago (서버 필요)</option></select></label>
          </div>
          <div style="margin-top:8px">
            <label class="small">기본 언어: <select id="langSel"><option>ko_kr</option><option>en_us</option></select></label>
          </div>
        </div>

        <div style="margin-top:12px" class="card">
          <div class="tool-title">빠른 링크</div>
          <div style="margin-top:8px"><a id="githubLink" class="link">GitHub 저장소 만들기 & 배포 방법</a></div>
          <div style="margin-top:8px"><a id="youtubeLink" class="link">AREXMADO 채널</a></div>
          <div style="margin-top:8px"><a id="communityLink" class="link">팬카페 / 커뮤니티 추가</a></div>
        </div>

        <div style="margin-top:12px" class="card">
          <div class="tool-title">배포 힌트</div>
          <div class="small">1) 새 GitHub 리포지토리 생성<br/>2) index.html을 루트에 업로드<br/>3) Settings → Pages → Branch: main, Root 선택 → 저장<br/>4) CNAME 사용 시 루트에 CNAME 파일 추가</div>
        </div>
      </aside>
    </main>

    <footer>
      <div class="small">이 페이지는 정적 사이트(프론트엔드) 예시입니다. 실제 모드 번역, 서버 저장, 자동 업데이트 등은 서버/백엔드 연동이 필요합니다.</div>
    </footer>
  </div>

  <script>
    // 간단한 클라이언트 시뮬레이션 로직 (서버 없이 동작하는 데모)
    const log = (t)=>{const a=document.getElementById('logArea');a.innerText = new Date().toLocaleTimeString() + ' — ' + t + '\n' + a.innerText}
    document.getElementById('themeToggle').addEventListener('click',()=>{
      const b=document.body;const is=b.getAttribute('data-theme')==='dark';b.setAttribute('data-theme', is? 'light':'dark');document.getElementById('themeToggle').innerText = is? '라이트 모드':'다크 모드';
    })

    document.getElementById('analyzeMods').addEventListener('click',async()=>{
      const files = document.getElementById('modUpload').files; if(!files.length){log('업로드 파일 없음');return}
      log('모드 분석 시작: ' + files.length + '개 파일');
      for(const f of files){
        log('파일 읽는중: ' + f.name);
        // 데모: 이름으로 간단 분류
        if(/\.lang|_ko_kr|ko_kr/i.test(f.name)) log(f.name + ' → 이미 한국어 리소스 포함됨으로 건너뜀');
        else if(/\.jar|\.zip/i.test(f.name)) log(f.name + ' → 내부 분석(시뮬레이션)');
        else log(f.name + ' → 번역 필요 가능성 있음');
        await new Promise(r=>setTimeout(r,400));
      }
      log('모드 분석 완료');
    })

    document.getElementById('analyzePack').addEventListener('click',()=>{
      const f = document.getElementById('packUpload').files[0]; if(!f){log('모드팩 파일 없음');return}
      log('모드팩 분석 시작: ' + f.name);
      setTimeout(()=>{log('모드 수: 예시 12개 / 번역 누락: 3개 / 권장: ko_kr 추가')},800);
    })

    document.getElementById('buildRp').addEventListener('click',()=>{
      const name = document.getElementById('packName').value || 'AREXMADO_ResourcePack';
      log('리소스팩 생성 시도: ' + name + '.zip (브라우저 시뮬레이션)');
      setTimeout(()=>{log('리소스팩 생성 완료 (시뮬레이션) — 다운로드 링크 생성 필요 (서버 업로드 또는 client-zip 사용)')},700);
    })

    // 링크들
    document.getElementById('githubLink').addEventListener('click',()=>{
      log('GitHub에 새 리포지토리를 만들고 index.html을 푸시한 뒤 Settings → Pages에서 사이트를 활성화하세요. (루트에 index.html을 둡니다)')
      alert('배포 힌트:\n1) 새 리포지토리 생성\n2) index.html 업로드\n3) Settings → Pages → main branch / root 선택\n4) 배포 완료')
    })
    document.getElementById('youtubeLink').addEventListener('click',()=>{window.open('https://www.youtube.com/@arexmado','_blank')})
    document.getElementById('communityLink').addEventListener('click',()=>{log('팬카페/커뮤니티 링크를 추가해 주세요 (예: Naver Cafe, Discord, Forum).')})
  </script>
</body>
</html>
