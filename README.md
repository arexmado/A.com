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
    input[type=file], select, input[type=text]{display:block;margin-top:8px;padding:6px;border-radius:6px;border:1px solid rgba(255,255,255,0.1);background:rgba(255,255,255,0.05);color:inherit}
    .log{height:180px;overflow:auto;background:rgba(255,255,255,0.02);padding:8px;border-radius:8px;font-family:monospace;font-size:13px;color:var(--muted)}
    footer{margin-top:18px;color:var(--muted);font-size:13px;text-align:center}
    @media(max-width:900px){main{grid-template-columns:1fr}}
    .small{font-size:13px;color:var(--muted)}
    .link{color:var(--accent);cursor:pointer}
    .pill{display:inline-block;padding:6px 10px;border-radius:999px;background:rgba(255,255,255,0.03);font-size:13px}
    .input-row{display:flex;gap:8px;align-items:center}
    button.primary{background:linear-gradient(90deg,rgba(96,165,250,0.12),rgba(96,165,250,0.06));border:1px solid rgba(96,165,250,0.18);padding:8px 12px;border-radius:8px;color:var(--accent);cursor:pointer}
    ul.tree{list-style:none;padding-left:16px}
    ul.tree li{margin:4px 0}
    ul.tree li span{color:var(--accent);cursor:pointer}
    .desc{font-size:12px;color:var(--muted);margin-left:4px}
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
              <div class="small">업로드 후 '분석 시작'을 눌러 시뮬레이션합니다.</div>
              <div style="margin-top:8px"><button id="analyzeMods" class="primary">분석 시작</button></div>
            </div>
          </div>

          <div class="card">
            <div class="tool-title">2) 리소스팩 빌더 (트리 구조 + 버전)</div>
            <div class="small">업로드한 다중 파일/폴더를 트리 구조로 표시하고, 설명과 마인크래프트 버전을 지정하여 리소스팩 ZIP을 생성합니다.</div>
            <div style="margin-top:8px">
              <input id="rpFiles" type="file" webkitdirectory directory multiple />
              <div id="rpTree" style="margin-top:12px"></div>
              <div style="margin-top:8px" class="input-row">
                <input id="packName" placeholder="리소스팩 이름" />
                <select id="packVersion">
                  <option value="1.20">1.20</option>
                  <option value="1.19">1.19</option>
                  <option value="1.18">1.18</option>
                  <option value="1.17">1.17</option>
                  <option value="1.16">1.16</option>
                  <option value="custom">직접 입력</option>
                </select>
                <input id="customVersion" placeholder="버전 입력" style="display:none" />
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
            <div class="small">GitHub Releases 또는 개인 서버와 연동하여 다운로드 링크를 보여줍니다.</div>
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
      <div class="small">이 페이지는 정적 사이트 예시입니다. 실제 번역, 서버 저장 등은 서버/백엔드 연동이 필요합니다.</div>
    </footer>
  </div>

  <script>
    const log = (t)=>{const a=document.getElementById('logArea');a.innerText = new Date().toLocaleTimeString() + ' — ' + t + "\n" + a.innerText}
    document.getElementById('themeToggle').addEventListener('click',()=>{
      const b=document.body;const is=b.getAttribute('data-theme')==='dark';b.setAttribute('data-theme', is? 'light':'dark');document.getElementById('themeToggle').innerText = is? '라이트 모드':'다크 모드';
    })

    document.getElementById('analyzeMods').addEventListener('click',async()=>{
      const files = document.getElementById('modUpload').files; if(!files.length){log('업로드 파일 없음');return}
      log('모드 분석 시작: ' + files.length + '개 파일');
      for(const f of files){
        log('파일 읽는중: ' + f.name);
        if(/\.lang|_ko_kr|ko_kr/i.test(f.name)) log(f.name + ' → 이미 한국어 리소스 포함됨');
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

    // 리소스팩 트리 구조 표시
    document.getElementById('rpFiles').addEventListener('change',(e)=>{
      const files = e.target.files;
      if(!files.length){document.getElementById('rpTree').innerHTML='';return}
      const tree = {};
      for(const f of files){
        const parts = f.webkitRelativePath.split('/');
        let cur = tree;
        for(const p of parts){cur[p] = cur[p] || {};cur = cur[p];}
      }
      const render=(obj)=>{
        const ul=document.createElement('ul');ul.classList.add('tree');
        for(const k in obj){
          const li=document.createElement('li');
          if(Object.keys(obj[k]).length){
            li.innerHTML='<span>'+k+'</span> <span class="desc">(폴더 설명: '+descForFolder(k)+')</span>';
            li.appendChild(render(obj[k]));
          } else {
            li.innerHTML=k;
          }
          ul.appendChild(li);
        }
        return ul;
      }
      const descForFolder=(name)=>{
        if(/lang/i.test(name)) return '이 폴더는 언어 폴더입니다';
        if(/textures?/i.test(name)) return '이 폴더는 텍스처/이미지 폴더입니다';
        if(/sounds?/i.test(name)) return '이 폴더는 사운드 리소스 폴더입니다';
        return '일반 폴더';
      }
      document.getElementById('rpTree').innerHTML='';
      document.getElementById('rpTree').appendChild(render(tree));
      log('리소스팩 파일 구조 표시 완료');
    })

    // 버전 선택 커스텀 입력 처리
    document.getElementById('packVersion').addEventListener('change',(e)=>{
      document.getElementById('customVersion').style.display = e.target.value==='custom' ? 'block':'none';
    })

    document.getElementById('buildRp').addEventListener('click',()=>{
      const name = document.getElementById('packName').value || 'AREXMADO_ResourcePack';
      let version = document.getElementById('packVersion').value;
      if(version==='custom') version=document.getElementById('customVersion').value||'알 수 없음';
      log('리소스팩 생성: '+name+'.zip (버전: '+version+', 트리 구조 포함, 시뮬레이션)');
      setTimeout(()=>{log('리소스팩 생성 완료 (시뮬레이션)')},700);
    })

    document.getElementById('githubLink').addEventListener('click',()=>{
      log('GitHub에 index.html을 푸시 후 Settings → Pages에서 활성화');
      alert('배포 힌트: 리포지토리 생성 → index.html 업로드 → Pages 설정');
    })
    document.getElementById('youtubeLink').addEventListener('click',()=>{window.open('https://www.youtube.com/@arexmado','_blank')})
    document.getElementById('communityLink').addEventListener('click',()=>{log('팬카페/커뮤니티 링크를 추가해 주세요')})
  </script>
</body>
</html>
