<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>AREXMADO 온라인 IDE</title>
  <style>
    body { margin:0; display:flex; height:100vh; overflow:hidden; font-family:sans-serif; }
    #sidebar { width:200px; background:#1e1e1e; color:#fff; padding:10px; box-sizing:border-box; }
    #editor { flex:1; }
    #right { width:40%; display:flex; flex-direction:column; border-left:1px solid #444; }
    #preview { flex:1; border:0; background:#fff; }
    #console { height:150px; background:#111; color:#0f0; overflow:auto; padding:5px; font-family:monospace; }
    button { margin-bottom:10px; width:100%; }
  </style>
  <!-- Monaco Editor -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/monaco-editor/0.44.0/min/vs/loader.min.js"></script>
  <!-- Pyodide for Python -->
  <script src="https://cdn.jsdelivr.net/pyodide/v0.23.4/full/pyodide.js"></script>
</head>
<body>
  <div id="sidebar">
    <button id="runBtn">▶ 실행</button>
    <ul id="fileList"></ul>
  </div>
  <div id="editor"></div>
  <div id="right">
    <iframe id="preview"></iframe>
    <div id="console"></div>
  </div>

  <script>
    document.addEventListener("DOMContentLoaded", async () => {
      // 파일 저장소
      let files = {
        "index.html": "<h1>Hello World</h1>",
        "style.css": "body { background: lightblue; }",
        "main.js": "console.log('Hello JS');",
        "main.py": "print('Hello Python')"
      };
      let currentFile = "index.html";
      const fileList = document.getElementById("fileList");
      const preview = document.getElementById("preview");
      const consoleDiv = document.getElementById("console");

      // 파일 목록 출력
      function renderFileList() {
        fileList.innerHTML = "";
        Object.keys(files).forEach(name => {
          const li = document.createElement("li");
          li.textContent = name;
          li.style.cursor = "pointer";
          li.onclick = () => loadFile(name);
          fileList.appendChild(li);
        });
      }

      // Monaco Editor 로드
      require.config({ paths: { vs: "https://cdnjs.cloudflare.com/ajax/libs/monaco-editor/0.44.0/min/vs" } });
      require(["vs/editor/editor.main"], function () {
        window.editor = monaco.editor.create(document.getElementById("editor"), {
          value: files[currentFile],
          language: "html",
          theme: "vs-dark"
        });
      });

      function loadFile(name) {
        currentFile = name;
        const lang = name.endsWith(".js") ? "javascript" :
                      name.endsWith(".css") ? "css" :
                      name.endsWith(".py") ? "python" : "html";
        window.editor.setValue(files[name]);
        monaco.editor.setModelLanguage(window.editor.getModel(), lang);
      }

      function saveFile() {
        files[currentFile] = window.editor.getValue();
      }

      function log(msg) {
        consoleDiv.textContent += msg + "\n";
        consoleDiv.scrollTop = consoleDiv.scrollHeight;
      }

      async function run() {
        saveFile();
        consoleDiv.textContent = "";
        // Python 실행
        if (currentFile.endsWith(".py")) {
          const pyodide = await loadPyodide();
          try {
            let result = await pyodide.runPythonAsync(files[currentFile]);
            log(result ?? "");
          } catch (e) {
            log("[Error] " + e);
          }
        } else {
          // HTML+CSS+JS 실행
          const doc = `<!DOCTYPE html><html><head><style>${files["style.css"]}</style></head><body>${files["index.html"]}<script>${files["main.js"]}<\/script></body></html>`;
          preview.srcdoc = doc;
        }
      }

      document.getElementById("runBtn").onclick = run;
      renderFileList();
    });
  </script>
</body>
</html>
