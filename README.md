<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>AREXMADO - Minecraft 통합 도구</title>
  <meta name="description" content="마인크래프트 모드팩 분석 도구 (GitHub Pages용 정적 페이지)" />
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--muted:#9aa4b2;--accent:#60a5fa}
    [data-theme="light"]{--bg:#f8fafc;--card:#ffffff;--muted:#55606a;--accent:#2563eb}
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter,Segoe UI,Apple SD Gothic Neo,'Noto Sans KR',sans-serif;background:var(--bg);color:#e6eef8}
    .wrap{max-width:1100px;margin:28px auto;padding:20px}
    header{display:flex;justify-content:space-between;align-items:center;margin-bottom:20px}
    h1{margin:0;font-size:20px}
    main{display:grid;grid-template-columns:1fr;gap:18px}
    .card{background:var(--card);padding:16px;border-radius:12px;box-shadow:0 6px 18px rgba(2,6,23,0.6)}
    .tool-title{font-weight:700;margin-bottom:8px;font-size:18px}
    input[type=file]{display:block;margin-top:8px}
    button{margin-top:10px;padding:8px 14px;border:none;border-radius:8px;background:var(--accent);color:#fff;font-weight:600;cursor:pointer}
    pre{background:#111827;padding:10px;border-radius:8px;max-height:300px;overflow:auto;white-space:pre-wrap;word-wrap:break-word}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <h1>AREXMADO - Minecraft 모드팩 분석 도구</h1>
    </header>
    <main>
      <div class="card">
        <div class="tool-title">모드팩 분석</div>
        <input type="file" id="mpFile" accept=".zip,.mrpack" />
        <button id="analyzeMp">분석하기</button>
        <pre id="log"></pre>
      </div>
    </main>
  </div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
  <script>
  document.getElementById('analyzeMp').addEventListener('click', async () => {
    const fileInput = document.getElementById('mpFile');
    const file = fileInput.files[0];
    if (!file) {
      log("모드팩 파일을 선택해주세요.");
      return;
    }

    log("모드팩 분석 중: " + file.name);
    const zip = await JSZip.loadAsync(file);
    let result = [];

    // manifest.json 검사
    if (zip.files["manifest.json"]) {
      const manifestText = await zip.files["manifest.json"].async("string");
      try {
        const manifest = JSON.parse(manifestText);
        result.push("모드팩 이름: " + (manifest.name || "알 수 없음"));
        result.push("마인크래프트 버전: " + (manifest.minecraft?.version || "알 수 없음"));
        if (manifest.files) {
          result.push("포함된 모드 수: " + manifest.files.length);
        }
      } catch(e) {
        result.push("manifest.json을 해석할 수 없습니다.");
      }
    }

    // mods 폴더 스캔
    const modFiles = Object.keys(zip.files).filter(f => f.startsWith("mods/") && f.endsWith(".jar"));
    if (modFiles.length > 0) {
      result.push("mods 폴더에 포함된 모드 수: " + modFiles.length);
      result.push("모드 목록:\n- " + modFiles.join("\n- "));
    }

    if (result.length === 0) {
      result.push("분석할 수 있는 정보가 없습니다.");
    }

    log(result.join("\n"));
  });

  function log(msg) {
    const logDiv = document.getElementById('log');
    logDiv.textContent = msg + "\n";
  }
  </script>
</body>
</html>
