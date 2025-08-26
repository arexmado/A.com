"""
Flask 설문조사 앱 (단일 파일, 템플릿 inline)
사용법:
1) 가상환경 생성 및 Flask 설치: pip install flask
2) 실행: python flask_survey_app.py
3) 브라우저로 접속: http://127.0.0.1:5000
관리자 페이지: /admin (관리자 비밀번호: admin 기본 — 환경변수 ADMIN_PW로 변경 가능)

기능 (MVP):
- 설문 생성(관리자)
- 설문 목록 보기
- 설문 응답 제출 (객관식/주관식 혼합)
- 설문 결과 조회(관리자)
- SQLite로 데이터 저장 (survey.db)

주의: 배포시에는 DEBUG=False, 실제 인증 추가, CSRF 보호 등을 적용하세요.
"""

from flask import Flask, g, request, redirect, url_for, render_template_string, abort
import sqlite3
import os
from datetime import datetime

DATABASE = 'survey.db'
ADMIN_PW = os.environ.get('ADMIN_PW', 'admin')  # 변경 권장

app = Flask(__name__)
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY', 'devsecret')

# ----------------- DB helpers -----------------

def get_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = sqlite3.connect(DATABASE)
        db.row_factory = sqlite3.Row
    return db

@app.teardown_appcontext
def close_connection(exception):
    db = getattr(g, '_database', None)
    if db is not None:
        db.close()

def init_db():
    db = get_db()
    cur = db.cursor()
    cur.executescript('''
    CREATE TABLE IF NOT EXISTS surveys (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        description TEXT,
        created_at TEXT NOT NULL
    );
    CREATE TABLE IF NOT EXISTS questions (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        survey_id INTEGER NOT NULL,
        qtext TEXT NOT NULL,
        qtype TEXT NOT NULL, -- 'text' or 'choice'
        choices TEXT, -- 콤마로 구분된 선택지 (choice 타입일 때)
        FOREIGN KEY(survey_id) REFERENCES surveys(id)
    );
    CREATE TABLE IF NOT EXISTS responses (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        survey_id INTEGER NOT NULL,
        submitted_at TEXT NOT NULL,
        FOREIGN KEY(survey_id) REFERENCES surveys(id)
    );
    CREATE TABLE IF NOT EXISTS answers (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        response_id INTEGER NOT NULL,
        question_id INTEGER NOT NULL,
        answer TEXT,
        FOREIGN KEY(response_id) REFERENCES responses(id),
        FOREIGN KEY(question_id) REFERENCES questions(id)
    );
    ''')
    db.commit()

# ----------------- Templates (inline) -----------------
BASE = """
<!doctype html>
<html lang="ko">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>{{ title }}</title>
<style>
 body{font-family: -apple-system, BlinkMacSystemFont, 'Malgun Gothic', '맑은 고딕', Arial;margin:20px;background:#f7f7f9}
 .container{max-width:900px;margin:0 auto;background:white;padding:20px;border-radius:8px;box-shadow:0 6px 18px rgba(0,0,0,0.06)}
 h1,h2{color:#333}
 label{display:block;margin:8px 0 4px}
 input[type=text], textarea, select{width:100%;padding:8px;border:1px solid #ddd;border-radius:6px}
 .btn{display:inline-block;padding:8px 12px;border-radius:6px;background:#2563eb;color:white;text-decoration:none}
 .muted{color:#666;font-size:0.9em}
 .question{padding:12px;border:1px solid #eee;border-radius:6px;margin-bottom:8px}
 .small{font-size:0.9em}
 nav a{margin-right:12px}
</style>
</head>
<body>
<div class="container">
<nav><a href="/">홈</a> <a href="/admin">관리자</a></nav>
<hr>
{% block body %}{% endblock %}
</div>
</body>
</html>
"""

INDEX_TMPL = """
{% extends base %}
{% block body %}
<h1>설문 목록</h1>
<p class="muted">아래 설문을 클릭해 답변하세요.</p>
<ul>
{% for s in surveys %}
  <li><a href="/survey/{{ s.id }}">{{ s.title }}</a> — <span class="muted">{{ s.created_at }}</span></li>
{% else %}
  <li>등록된 설문이 없습니다.</li>
{% endfor %}
</ul>
{% endblock %}
"""

SURVEY_TMPL = """
{% extends base %}
{% block body %}
<h1>{{ survey.title }}</h1>
<p class="muted">{{ survey.description }}</p>
<form method="post">
  {% for q in questions %}
    <div class="question">
      <label><strong>{{ loop.index }}. {{ q.qtext }}</strong></label>
      {% if q.qtype == 'text' %}
        <textarea name="q_{{ q.id }}" rows="3"></textarea>
      {% else %}
        {% for c in q.choices.split(',') %}
          <div><label><input type="radio" name="q_{{ q.id }}" value="{{ c.strip() }}"> {{ c.strip() }}</label></div>
        {% endfor %}
      {% endif %}
    </div>
  {% endfor %}
  <button class="btn" type="submit">제출</button>
</form>
{% endblock %}
"""

ADMIN_TMPL = """
{% extends base %}
{% block body %}
<h1>관리자</h1>
<p class="muted">설문 생성 및 결과 조회 (관리자 비밀번호 필요)</p>
<form method="post" action="/admin/login">
  <label>비밀번호</label>
  <input type="password" name="pw">
  <button class="btn" type="submit">로그인</button>
</form>
{% endblock %}
"""

ADMIN_DASH = """
{% extends base %}
{% block body %}
<h1>관리자 대시보드</h1>
<p><a class="btn" href="/admin/new">새 설문 만들기</a></p>
<h2>설문 목록</h2>
<ul>
{% for s in surveys %}
  <li>{{ s.title }} — <a href="/admin/survey/{{ s.id }}/results">결과 보기</a> | <a href="/admin/survey/{{ s.id }}/edit">수정</a></li>
{% else %}
  <li>설문이 없습니다.</li>
{% endfor %}
</ul>
{% endblock %}
"""

NEW_TMPL = """
{% extends base %}
{% block body %}
<h1>새 설문 만들기</h1>
<form method="post">
  <label>제목</label>
  <input type="text" name="title" required>
  <label>설명</label>
  <input type="text" name="description">
  <hr>
  <h3>질문 추가 (아래처럼 입력)</h3>
  <p class="muted">질문 형식: text 또는 choice<br>선택지는 콤마(,)로 구분합니다 (choice일 때)</p>
  <div id="qs">
    <div>
      <label>질문 텍스트</label>
      <input type="text" name="qtext_1">
      <label>형식 (text / choice)</label>
      <input type="text" name="qtype_1" value="text">
      <label>선택지 (choice일 때 콤마로 구분)</label>
      <input type="text" name="choices_1">
    </div>
  </div>
  <p class="muted">필요한 질문 개수만큼 qtext_N, qtype_N, choices_N 형태로 전송하면 됩니다. (1부터 시작)</p>
  <button class="btn" type="submit">생성</button>
</form>
{% endblock %}
"""

RESULTS_TMPL = """
{% extends base %}
{% block body %}
<h1>결과: {{ survey.title }}</h1>
<p class="muted">생성: {{ survey.created_at }}</p>
<h2>응답 수: {{ responses_count }}</h2>
{% for q in questions %}
  <div class="question">
    <strong>{{ loop.index }}. {{ q.qtext }}</strong>
    {% if q.qtype == 'text' %}
      <ul>
      {% for a in text_answers[q.id] %}
        <li>{{ a }}</li>
      {% else %}
        <li class="muted">응답 없음</li>
      {% endfor %}
      </ul>
    {% else %}
      <ul>
      {% for choice, cnt in choice_stats[q.id].items() %}
        <li>{{ choice }} — {{ cnt }}표</li>
      {% endfor %}
      </ul>
    {% endif %}
  </div>
{% endfor %}
{% endblock %}
"""

# ----------------- Routes -----------------

@app.route('/')
def index():
    db = get_db()
    cur = db.execute('SELECT * FROM surveys ORDER BY id DESC')
    surveys = cur.fetchall()
    return render_template_string(INDEX_TMPL, base=BASE, title='설문 목록', surveys=surveys)

@app.route('/survey/<int:sid>', methods=['GET', 'POST'])
def survey_view(sid):
    db = get_db()
    cur = db.execute('SELECT * FROM surveys WHERE id=?', (sid,))
    survey = cur.fetchone()
    if not survey:
        abort(404)
    qcur = db.execute('SELECT * FROM questions WHERE survey_id=? ORDER BY id', (sid,))
    questions = qcur.fetchall()
    if request.method == 'POST':
        # 응답 저장
        now = datetime.utcnow().isoformat()
        cur = db.execute('INSERT INTO responses (survey_id, submitted_at) VALUES (?, ?)', (sid, now))
        resp_id = cur.lastrowid
        for q in questions:
            key = f'q_{q["id"]}'
            val = request.form.get(key, '').strip()
            db.execute('INSERT INTO answers (response_id, question_id, answer) VALUES (?, ?, ?)', (resp_id, q['id'], val))
        db.commit()
        return render_template_string(BASE + "<h2>감사합니다! 응답이 저장되었습니다.</h2><p><a href='/'>목록으로</a></p>", title='감사')
    return render_template_string(SURVEY_TMPL, base=BASE, title=survey['title'], survey=survey, questions=questions)

# ----- Admin login (simple) -----
@app.route('/admin', methods=['GET'])
def admin():
    return render_template_string(ADMIN_TMPL, base=BASE, title='관리자')

@app.route('/admin/login', methods=['POST'])
def admin_login():
    pw = request.form.get('pw','')
    if pw != ADMIN_PW:
        return render_template_string(BASE + "<h2>비밀번호가 틀렸습니다.</h2><p><a href='/admin'>돌아가기</a></p>", title='비밀번호 오류')
    # 단순 세션 관리 대신 쿼리파라미터로 토큰(간단한 방식)
    token = 'admintoken'
    return redirect(url_for('admin_dash') + f'?t={token}')

@app.route('/admin/dashboard')
def admin_dash():
    t = request.args.get('t','')
    if t != 'admintoken':
        return redirect(url_for('admin'))
    db = get_db()
    cur = db.execute('SELECT * FROM surveys ORDER BY id DESC')
    surveys = cur.fetchall()
    return render_template_string(ADMIN_DASH, base=BASE, title='관리자 대시보드', surveys=surveys)

@app.route('/admin/new', methods=['GET','POST'])
def admin_new():
    t = request.args.get('t','')
    if t != 'admintoken':
        return redirect(url_for('admin'))
    if request.method == 'POST':
        title = request.form.get('title','').strip()
        desc = request.form.get('description','').strip()
        if not title:
            return '제목은 필수입니다.'
        now = datetime.utcnow().isoformat()
        db = get_db()
        cur = db.execute('INSERT INTO surveys (title, description, created_at) VALUES (?, ?, ?)', (title, desc, now))
        sid = cur.lastrowid
        # 질문 파싱: qtext_1, qtype_1, choices_1 ... 형식
        i = 1
        while True:
            qtext = request.form.get(f'qtext_{i}')
            if not qtext:
                break
            qtype = request.form.get(f'qtype_{i}','text').strip()
            choices = request.form.get(f'choices_{i}','').strip()
            db.execute('INSERT INTO questions (survey_id, qtext, qtype, choices) VALUES (?, ?, ?, ?)', (sid, qtext, qtype, choices))
            i += 1
        db.commit()
        return redirect(url_for('admin_dash') + '?t=admintoken')
    return render_template_string(NEW_TMPL, base=BASE, title='새 설문 만들기')

@app.route('/admin/survey/<int:sid>/results')
def admin_results(sid):
    t = request.args.get('t','')
    if t != 'admintoken':
        return redirect(url_for('admin'))
    db = get_db()
    cur = db.execute('SELECT * FROM surveys WHERE id=?', (sid,))
    survey = cur.fetchone()
    if not survey:
        abort(404)
    qcur = db.execute('SELECT * FROM questions WHERE survey_id=? ORDER BY id', (sid,))
    questions = qcur.fetchall()
    rcur = db.execute('SELECT COUNT(*) as cnt FROM responses WHERE survey_id=?', (sid,))
    responses_count = rcur.fetchone()['cnt']
    # 통계 생성
    choice_stats = {q['id']: {} for q in questions if q['qtype']=='choice'}
    text_answers = {q['id']: [] for q in questions if q['qtype']=='text'}
    acur = db.execute('SELECT a.question_id, a.answer FROM answers a JOIN responses r ON a.response_id=r.id WHERE r.survey_id=?', (sid,))
    for row in acur.fetchall():
        qid = row['question_id']
        ans = row['answer']
        # 선택형
        if qid in choice_stats:
            choice_stats[qid][ans] = choice_stats[qid].get(ans, 0) + 1
        if qid in text_answers:
            if ans:
                text_answers[qid].append(ans)
    return render_template_string(RESULTS_TMPL, base=BASE, title='결과', survey=survey, questions=questions, responses_count=responses_count, choice_stats=choice_stats, text_answers=text_answers)

# 편집 등 추가 라우트는 필요 시 확장

if __name__ == '__main__':
    # DB 초기화
    with app.app_context():
        init_db()
    app.run(debug=True)
