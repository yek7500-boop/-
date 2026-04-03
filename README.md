<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>할일 목록</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      background: #f5f5f0;
      min-height: 100vh;
      display: flex;
      justify-content: center;
      padding: 48px 16px;
    }
    .app {
      background: white;
      border-radius: 16px;
      padding: 32px;
      width: 100%;
      max-width: 480px;
      height: fit-content;
      box-shadow: 0 2px 12px rgba(0,0,0,0.08);
    }
    h1 { font-size: 22px; font-weight: 600; color: #1a1a1a; margin-bottom: 6px; }
    .subtitle { font-size: 13px; color: #999; margin-bottom: 24px; }
    .input-row { display: flex; gap: 8px; margin-bottom: 8px; }
    input[type="text"] {
      flex: 1; padding: 10px 14px;
      border: 1.5px solid #e5e5e5; border-radius: 10px;
      font-size: 14px; color: #1a1a1a; outline: none; transition: border-color .15s;
    }
    input[type="text"]:focus { border-color: #4f8ef7; }
    input[type="text"]::placeholder { color: #bbb; }
    .time-row { display: flex; align-items: center; gap: 8px; margin-bottom: 16px; }
    input[type="time"] {
      padding: 7px 12px; border: 1.5px solid #e5e5e5; border-radius: 10px;
      font-size: 13px; color: #555; outline: none;
    }
    .time-label { font-size: 12px; color: #aaa; }
    button.add-btn {
      padding: 10px 18px; background: #4f8ef7; color: white;
      border: none; border-radius: 10px; font-size: 14px; font-weight: 500;
      cursor: pointer; transition: background .15s; white-space: nowrap;
    }
    button.add-btn:hover { background: #3a7de8; }
    .filters { display: flex; gap: 6px; margin-bottom: 16px; }
    .filter-btn {
      padding: 5px 12px; border-radius: 999px; border: 1.5px solid #e5e5e5;
      background: transparent; font-size: 12px; color: #888; cursor: pointer; transition: all .15s;
    }
    .filter-btn.active { background: #1a1a1a; border-color: #1a1a1a; color: white; }
    .todo-list { display: flex; flex-direction: column; gap: 8px; }
    .todo-item {
      display: flex; align-items: center; gap: 12px;
      padding: 12px 14px; border: 1.5px solid #f0f0f0;
      border-radius: 10px; background: #fafafa; transition: all .15s;
    }
    .todo-item:hover { border-color: #e0e0e0; }
    .todo-item.done { background: #f8f8f8; }
    .todo-check {
      width: 20px; height: 20px; border-radius: 50%; border: 2px solid #ddd;
      cursor: pointer; flex-shrink: 0; display: flex; align-items: center;
      justify-content: center; transition: all .15s;
    }
    .todo-item.done .todo-check { background: #4caf82; border-color: #4caf82; }
    .todo-check svg { display: none; }
    .todo-item.done .todo-check svg { display: block; }
    .todo-body { flex: 1; min-width: 0; }
    .todo-text { font-size: 14px; color: #1a1a1a; line-height: 1.4; word-break: break-all; }
    .todo-item.done .todo-text { text-decoration: line-through; color: #bbb; }
    .todo-time { font-size: 11px; color: #bbb; margin-top: 2px; }
    .todo-item.done .todo-time { color: #ddd; }
    .del-btn {
      background: none; border: none; color: #ccc; cursor: pointer;
      font-size: 18px; line-height: 1; padding: 2px 4px;
      border-radius: 4px; transition: color .15s;
    }
    .del-btn:hover { color: #e57373; }
    .empty { text-align: center; padding: 32px 0; color: #ccc; font-size: 14px; }
    .footer {
      margin-top: 20px; padding-top: 16px; border-top: 1px solid #f0f0f0;
      display: flex; justify-content: space-between; align-items: center;
    }
    .count { font-size: 12px; color: #aaa; }
    .clear-btn { font-size: 12px; color: #ccc; background: none; border: none; cursor: pointer; }
    .clear-btn:hover { color: #e57373; }
  </style>
</head>
<body>
  <div class="app">
    <h1>할일 목록</h1>
    <p class="subtitle" id="date-label"></p>
    <div class="input-row">
      <input type="text" id="todo-input" placeholder="새로운 할일을 입력하세요..." />
      <button class="add-btn" onclick="addTodo()">추가</button>
    </div>
    <div class="time-row">
      <span class="time-label">시간 (선택)</span>
      <input type="time" id="todo-time" />
    </div>
    <div class="filters">
      <button class="filter-btn active" onclick="setFilter('all', this)">전체</button>
      <button class="filter-btn" onclick="setFilter('active', this)">진행중</button>
      <button class="filter-btn" onclick="setFilter('done', this)">완료</button>
    </div>
    <div class="todo-list" id="todo-list"></div>
    <div class="footer">
      <span class="count" id="count-label"></span>
      <button class="clear-btn" onclick="clearDone()">완료 항목 삭제</button>
    </div>
  </div>
  <script>
    let todos = JSON.parse(localStorage.getItem('todos') || '[]');
    let filter = 'all';
    if (!todos.find(t => t.id === 'preset-1')) {
      todos.unshift({ id: 'preset-1', text: '증시 분석하기', time: '09:30', done: false });
      save();
    }
    function save() { localStorage.setItem('todos', JSON.stringify(todos)); }
    function render() {
      const list = document.getElementById('todo-list');
      const filtered = todos
        .filter(t => filter === 'all' ? true : filter === 'done' ? t.done : !t.done)
        .sort((a, b) => {
          if (a.time && b.time) return a.time.localeCompare(b.time);
          if (a.time) return -1;
          if (b.time) return 1;
          return 0;
        });
      if (filtered.length === 0) {
        list.innerHTML = '<div class="empty">할일이 없어요 ✓</div>';
      } else {
        list.innerHTML = filtered.map(t => `
          <div class="todo-item ${t.done ? 'done' : ''}">
            <div class="todo-check" onclick="toggleTodo('${t.id}')">
              <svg width="10" height="10" viewBox="0 0 10 10" fill="none">
                <polyline points="1.5,5 4,7.5 8.5,2" stroke="white" stroke-width="1.8"
                  stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </div>
            <div class="todo-body">
              <div class="todo-text">${t.text}</div>
              ${t.time ? `<div class="todo-time">⏰ ${t.time}</div>` : ''}
            </div>
            <button class="del-btn" onclick="deleteTodo('${t.id}')">×</button>
          </div>
        `).join('');
      }
      const active = todos.filter(t => !t.done).length;
      document.getElementById('count-label').textContent =
        `남은 할일 ${active}개 / 전체 ${todos.length}개`;
    }
    function addTodo() {
      const input = document.getElementById('todo-input');
      const timeInput = document.getElementById('todo-time');
      const text = input.value.trim();
      if (!text) return;
      todos.unshift({ id: String(Date.now()), text, time: timeInput.value || '', done: false });
      input.value = ''; timeInput.value = '';
      save(); render();
    }
    function toggleTodo(id) {
      todos = todos.map(t => t.id === id ? { ...t, done: !t.done } : t);
      save(); render();
    }
    function deleteTodo(id) {
      todos = todos.filter(t => t.id !== id);
      save(); render();
    }
    function setFilter(f, btn) {
      filter = f;
      document.querySelectorAll('.filter-btn').forEach(b => b.classList.remove('active'));
      btn.classList.add('active'); render();
    }
    function clearDone() {
      todos = todos.filter(t => !t.done);
      save(); render();
    }
    document.getElementById('todo-input').addEventListener('keydown', e => {
      if (e.key === 'Enter') addTodo();
    });
    const today = new Date();
    document.getElementById('date-label').textContent =
      today.toLocaleDateString('ko-KR', { year: 'numeric', month: 'long', day: 'numeric', weekday: 'long' });
    render();
  </script>
</body>
</html>
