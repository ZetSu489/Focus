# Focus
A new app
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>FocusFlow — Productivity & Pomodoro</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          colors: {
            brand: { 500: '#6366f1', 600: '#4f46e5', 700: '#4338ca' }
          }
        }
      }
    }
  </script>
  <style>
    .timer-ring-circle {
      transition: stroke-dashoffset 0.5s ease-in-out;
      transform: rotate(-90deg);
      transform-origin: 50% 50%;
    }
  </style>
</head>
<body class="bg-slate-50 dark:bg-slate-900 text-slate-800 dark:text-slate-100 min-h-screen transition-colors duration-200">
  <div class="max-w-4xl mx-auto px-4 py-8">
    <!-- Header -->
    <header class="flex justify-between items-center mb-8 pb-4 border-b border-slate-200 dark:border-slate-800">
      <div>
        <h1 class="text-2xl font-bold tracking-tight text-slate-900 dark:text-white flex items-center gap-2">
          ⚡ FocusFlow
        </h1>
        <p class="text-xs text-slate-500 dark:text-slate-400">Pomodoro Timer & Sprint Board</p>
      </div>
      <button id="themeToggle" class="px-3 py-1.5 rounded-lg border border-slate-300 dark:border-slate-700 text-xs font-medium hover:bg-slate-100 dark:hover:bg-slate-800 transition">
        Toggle Theme
      </button>
    </header>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
      <!-- Pomodoro Section -->
      <section class="md:col-span-1 bg-white dark:bg-slate-800 p-6 rounded-2xl shadow-sm border border-slate-200 dark:border-slate-700/60 flex flex-col items-center justify-center text-center">
        <div class="flex gap-2 mb-6">
          <button id="modePomodoro" class="px-3 py-1 text-xs font-semibold rounded-full bg-brand-500 text-white">Focus</button>
          <button id="modeBreak" class="px-3 py-1 text-xs font-medium rounded-full bg-slate-100 dark:bg-slate-700 text-slate-600 dark:text-slate-300">Break</button>
        </div>

        <div class="relative w-48 h-48 flex items-center justify-center my-2">
          <svg class="w-full h-full" viewBox="0 0 100 100">
            <circle class="text-slate-200 dark:text-slate-700" stroke-width="6" stroke="currentColor" fill="transparent" r="42" cx="50" cy="50"/>
            <circle id="timerRing" class="text-brand-500 timer-ring-circle" stroke-width="6" stroke-dasharray="264" stroke-dashoffset="0" stroke-linecap="round" stroke="currentColor" fill="transparent" r="42" cx="50" cy="50"/>
          </svg>
          <div class="absolute flex flex-col items-center">
            <span id="timerDisplay" class="text-4xl font-extrabold tracking-tight">25:00</span>
            <span id="timerState" class="text-[10px] uppercase font-bold tracking-widest text-slate-400 mt-1">Ready</span>
          </div>
        </div>

        <div class="flex gap-3 mt-6">
          <button id="startBtn" class="px-5 py-2 rounded-xl bg-brand-500 hover:bg-brand-600 text-white font-semibold text-sm transition shadow-sm">Start</button>
          <button id="resetBtn" class="px-4 py-2 rounded-xl border border-slate-200 dark:border-slate-700 hover:bg-slate-100 dark:hover:bg-slate-700 text-xs font-medium transition">Reset</button>
        </div>
      </section>

      <!-- Task Kanban / List -->
      <section class="md:col-span-2 bg-white dark:bg-slate-800 p-6 rounded-2xl shadow-sm border border-slate-200 dark:border-slate-700/60 flex flex-col">
        <div class="flex items-center justify-between mb-4">
          <h2 class="font-bold text-slate-900 dark:text-white text-base">Active Tasks</h2>
          <span id="taskCount" class="text-xs bg-slate-100 dark:bg-slate-700 text-slate-600 dark:text-slate-300 px-2 py-0.5 rounded-full font-medium">0 items</span>
        </div>

        <form id="taskForm" class="flex gap-2 mb-4">
          <input id="taskInput" type="text" placeholder="Add a new task..." required class="flex-1 px-3 py-2 text-sm bg-slate-50 dark:bg-slate-900 border border-slate-200 dark:border-slate-700 rounded-xl focus:outline-none focus:ring-2 focus:ring-brand-500">
          <button type="submit" class="px-4 py-2 bg-brand-500 hover:bg-brand-600 text-white font-medium text-sm rounded-xl transition">Add</button>
        </form>

        <ul id="taskList" class="flex-1 space-y-2 overflow-y-auto max-h-[300px]">
          <!-- Dynamic Task items inserted here -->
        </ul>
      </section>
    </div>
  </div>

  <script>
    // Timer State
    let totalSeconds = 25 * 60;
    let remainingSeconds = totalSeconds;
    let timerInterval = null;
    let isBreak = false;

    const timerDisplay = document.getElementById('timerDisplay');
    const timerRing = document.getElementById('timerRing');
    const timerState = document.getElementById('timerState');
    const startBtn = document.getElementById('startBtn');
    const resetBtn = document.getElementById('resetBtn');
    const modePomodoro = document.getElementById('modePomodoro');
    const modeBreak = document.getElementById('modeBreak');
    const perimeter = 2 * Math.PI * 42; // ~263.89

    timerRing.style.strokeDasharray = `${perimeter}`;

    function updateDisplay() {
      const minutes = Math.floor(remainingSeconds / 60);
      const seconds = remainingSeconds % 60;
      timerDisplay.textContent = `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
      
      const fraction = remainingSeconds / totalSeconds;
      const offset = perimeter - (fraction * perimeter);
      timerRing.style.strokeDashoffset = offset;
      document.title = `${timerDisplay.textContent} - FocusFlow`;
    }

    function toggleTimer() {
      if (timerInterval) {
        clearInterval(timerInterval);
        timerInterval = null;
        startBtn.textContent = 'Start';
        timerState.textContent = 'Paused';
      } else {
        timerInterval = setInterval(() => {
          if (remainingSeconds > 0) {
            remainingSeconds--;
            updateDisplay();
          } else {
            clearInterval(timerInterval);
            timerInterval = null;
            startBtn.textContent = 'Start';
            timerState.textContent = 'Done!';
            playChime();
          }
        }, 1000);
        startBtn.textContent = 'Pause';
        timerState.textContent = isBreak ? 'Resting' : 'Focusing';
      }
    }

    function resetTimer() {
      if (timerInterval) clearInterval(timerInterval);
      timerInterval = null;
      remainingSeconds = totalSeconds;
      startBtn.textContent = 'Start';
      timerState.textContent = 'Ready';
      updateDisplay();
    }

    function setMode(breakMode) {
      isBreak = breakMode;
      totalSeconds = isBreak ? 5 * 60 : 25 * 60;
      if (isBreak) {
        modeBreak.className = 'px-3 py-1 text-xs font-semibold rounded-full bg-brand-500 text-white';
        modePomodoro.className = 'px-3 py-1 text-xs font-medium rounded-full bg-slate-100 dark:bg-slate-700 text-slate-600 dark:text-slate-300';
      } else {
        modePomodoro.className = 'px-3 py-1 text-xs font-semibold rounded-full bg-brand-500 text-white';
        modeBreak.className = 'px-3 py-1 text-xs font-medium rounded-full bg-slate-100 dark:bg-slate-700 text-slate-600 dark:text-slate-300';
      }
      resetTimer();
    }

    // Audio chime using Web Audio API (no external file needed)
    function playChime() {
      try {
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.frequency.setValueAtTime(587.33, audioCtx.currentTime); // D5
        gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 1.2);
        osc.start();
        osc.stop(audioCtx.currentTime + 1.2);
      } catch (e) {}
    }

    startBtn.addEventListener('click', toggleTimer);
    resetBtn.addEventListener('click', resetTimer);
    modePomodoro.addEventListener('click', () => setMode(false));
    modeBreak.addEventListener('click', () => setMode(true));

    // Task Management
    const taskForm = document.getElementById('taskForm');
    const taskInput = document.getElementById('taskInput');
    const taskList = document.getElementById('taskList');
    const taskCount = document.getElementById('taskCount');
    let tasks = JSON.parse(localStorage.getItem('focus_tasks') || '[]');

    function renderTasks() {
      taskList.innerHTML = '';
      tasks.forEach((task, index) => {
        const li = document.createElement('li');
        li.className = `flex items-center justify-between p-3 rounded-xl border text-sm transition ${
          task.completed 
            ? 'bg-slate-50/50 dark:bg-slate-900/50 border-slate-200 dark:border-slate-800 text-slate-400 line-through' 
            : 'bg-white dark:bg-slate-900 border-slate-200 dark:border-slate-700'
        }`;
        
        li.innerHTML = `
          <div class="flex items-center gap-3">
            <input type="checkbox" ${task.completed ? 'checked' : ''} onchange="toggleTask(${index})" class="rounded border-slate-300 text-brand-500 focus:ring-brand-500 h-4 w-4">
            <span>${escapeHTML(task.text)}</span>
          </div>
          <button onclick="deleteTask(${index})" class="text-slate-400 hover:text-red-500 text-xs">Delete</button>
        `;
        taskList.appendChild(li);
      });
      taskCount.textContent = `${tasks.length} items`;
      localStorage.setItem('focus_tasks', JSON.stringify(tasks));
    }

    function escapeHTML(str) {
      return str.replace(/[&<>'"]/g, tag => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', "'": '&#39;', '"': '&quot;' }[tag] || tag));
    }

    window.toggleTask = function(index) {
      tasks[index].completed = !tasks[index].completed;
      renderTasks();
    };

    window.deleteTask = function(index) {
      tasks.splice(index, 1);
      renderTasks();
    };

    taskForm.addEventListener('submit', (e) => {
      e.preventDefault();
      const text = taskInput.value.trim();
      if (text) {
        tasks.push({ text, completed: false });
        taskInput.value = '';
        renderTasks();
      }
    });

    // Theme Toggle
    const themeToggle = document.getElementById('themeToggle');
    themeToggle.addEventListener('click', () => {
      document.documentElement.classList.toggle('dark');
    });

    // Init
    updateDisplay();
    renderTasks();
  </script>
</body>
</html>
