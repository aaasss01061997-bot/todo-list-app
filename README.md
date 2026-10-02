const STORAGE_KEY = 'todo-list-items';

let tasks = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');
let currentFilter = 'all';

const todoForm = document.getElementById('todo-form');
const todoInput = document.getElementById('todo-input');
const todoList = document.getElementById('todo-list');
const taskSummary = document.getElementById('task-summary');
const clearCompletedBtn = document.getElementById('clear-completed');
const filterButtons = document.querySelectorAll('.filter-btn');

function saveTasks() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks));
}

function getFilteredTasks() {
  if (currentFilter === 'active') {
    return tasks.filter((task) => !task.completed);
  }

  if (currentFilter === 'completed') {
    return tasks.filter((task) => task.completed);
  }

  return tasks;
}

function updateSummary() {
  const remainingTasks = tasks.filter((task) => !task.completed).length;
  taskSummary.textContent =
    remainingTasks === 1 ? '1 task left' : `${remainingTasks} tasks left`;
}

function renderTasks() {
  const filteredTasks = getFilteredTasks();
  todoList.innerHTML = '';

  if (filteredTasks.length === 0) {
    const emptyState = document.createElement('li');
    emptyState.className = 'empty-state';
    emptyState.textContent = 'No tasks here yet. Add one above!';
    todoList.appendChild(emptyState);
    updateSummary();
    return;
  }

  filteredTasks.forEach((task) => {
    const listItem = document.createElement('li');
    listItem.className = `todo-item ${task.completed ? 'completed' : ''}`;

    const mainContent = document.createElement('div');
    mainContent.className = 'todo-main';

    const checkbox = document.createElement('input');
    checkbox.type = 'checkbox';
    checkbox.checked = task.completed;
    checkbox.className = 'todo-checkbox';
    checkbox.setAttribute('aria-label', `Mark task as complete: ${task.text}`);
    checkbox.addEventListener('change', () => {
      task.completed = checkbox.checked;
      saveTasks();
      renderTasks();
    });

    const text = document.createElement('span');
    text.className = 'todo-text';
    text.textContent = task.text;

    const deleteBtn = document.createElement('button');
    deleteBtn.type = 'button';
    deleteBtn.className = 'delete-btn';
    deleteBtn.textContent = 'Delete';
    deleteBtn.setAttribute('aria-label', `Delete task: ${task.text}`);
    deleteBtn.addEventListener('click', () => {
      tasks = tasks.filter((item) => item.id !== task.id);
      saveTasks();
      renderTasks();
    });

    mainContent.appendChild(checkbox);
    mainContent.appendChild(text);
    listItem.appendChild(mainContent);
    listItem.appendChild(deleteBtn);
    todoList.appendChild(listItem);
  });

  updateSummary();
}

todoForm.addEventListener('submit', (event) => {
  event.preventDefault();

  const text = todoInput.value.trim();
  if (!text) {
    todoInput.focus();
    return;
  }

  tasks.unshift({
    id: Date.now(),
    text,
    completed: false,
  });

  todoInput.value = '';
  saveTasks();
  renderTasks();
  todoInput.focus();
});

filterButtons.forEach((button) => {
  button.addEventListener('click', () => {
    currentFilter = button.dataset.filter;

    filterButtons.forEach((btn) => {
      btn.classList.toggle('active', btn === button);
    });

    renderTasks();
  });
});

clearCompletedBtn.addEventListener('click', () => {
  tasks = tasks.filter((task) => !task.completed);
  saveTasks();
  renderTasks();
});

renderTasks();
