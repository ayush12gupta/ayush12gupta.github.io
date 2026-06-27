---
layout: page
title: our list
permalink: /c4ts-n-plans/
nav: false
sitemap: false
description:
---

<style>
  .together-wrap {
    display: flex;
    gap: 2rem;
    align-items: flex-start;
    flex-wrap: wrap;
  }
  .tracker-card, .cats-card {
    background: linear-gradient(135deg, #fff0f6 0%, #f3e8ff 100%);
    border-radius: 1.2rem;
    padding: 1.6rem;
    box-shadow: 0 4px 18px rgba(200, 100, 200, 0.12);
    flex: 1 1 300px;
  }
  .card-title {
    font-size: 1.25rem;
    font-weight: 700;
    color: #9b59b6;
    margin-bottom: 1rem;
  }
  .add-row {
    display: flex;
    gap: 0.5rem;
    margin-bottom: 1rem;
  }
  .add-row input {
    flex: 1;
    border: 2px solid #d8aaff;
    border-radius: 0.7rem;
    padding: 0.45rem 0.8rem;
    font-size: 0.95rem;
    outline: none;
    background: #fff;
    color: #333;
    transition: border-color 0.2s;
  }
  .add-row input:focus { border-color: #9b59b6; }
  .add-row button, .cat-btn {
    background: linear-gradient(135deg, #c471ed, #9b59b6);
    color: #fff;
    border: none;
    border-radius: 0.7rem;
    padding: 0.45rem 1rem;
    font-size: 0.9rem;
    cursor: pointer;
    font-weight: 600;
    transition: opacity 0.2s;
  }
  .add-row button:hover, .cat-btn:hover { opacity: 0.85; }
  #activity-list { list-style: none; padding: 0; margin: 0; }
  #activity-list li {
    display: flex;
    align-items: center;
    gap: 0.6rem;
    padding: 0.5rem 0.3rem;
    border-bottom: 1px dashed #e0c8f8;
    font-size: 0.97rem;
    color: #444;
  }
  #activity-list li:last-child { border-bottom: none; }
  #activity-list li input[type="checkbox"] {
    accent-color: #9b59b6;
    width: 1.1rem;
    height: 1.1rem;
    cursor: pointer;
    flex-shrink: 0;
  }
  #activity-list li span {
    flex: 1;
    transition: all 0.2s;
  }
  #activity-list li.done span {
    text-decoration: line-through;
    color: #bbb;
  }
  .del-btn {
    background: none;
    border: none;
    color: #d9a0e0;
    font-size: 1rem;
    cursor: pointer;
    padding: 0 0.2rem;
    line-height: 1;
    transition: color 0.2s;
  }
  .del-btn:hover { color: #9b59b6; }
  .empty-msg {
    color: #c8a0dc;
    font-size: 0.9rem;
    font-style: italic;
    text-align: center;
    padding: 1rem 0;
  }
  .gif-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 0.7rem;
    margin-bottom: 1rem;
  }
  .gif-grid img {
    width: 100%;
    border-radius: 0.8rem;
    object-fit: cover;
    aspect-ratio: 1;
    display: block;
  }
  .random-cat-wrap {
    text-align: center;
    margin-top: 0.8rem;
  }
  #random-cat {
    width: 100%;
    border-radius: 0.8rem;
    margin-bottom: 0.8rem;
    display: none;
  }
  .cat-btn { width: 100%; padding: 0.6rem; font-size: 1rem; }
  @media (max-width: 600px) {
    .together-wrap { flex-direction: column; }
  }
</style>

<div class="together-wrap">

  <!-- Activity Tracker -->
  <div class="tracker-card">
    <div class="card-title">✨ things we wanna do</div>
    <div class="add-row">
      <input type="text" id="activity-input" placeholder="add something fun..." maxlength="120" />
      <button onclick="addActivity()">add</button>
    </div>
    <ul id="activity-list"></ul>
    <p class="empty-msg" id="empty-msg">no plans yet — add something! 🌸</p>
  </div>

  <!-- Cat GIFs -->
  <div class="cats-card">
    <div class="card-title">🐱 cat corner</div>
    <div class="gif-grid">
      <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" alt="cat" loading="lazy" />
      <img src="https://media.giphy.com/media/mlvseq9yvZhba/giphy.gif" alt="cat" loading="lazy" />
      <img src="https://media.giphy.com/media/vFKqnCdLPNOKc/giphy.gif" alt="cat" loading="lazy" />
      <img src="https://media.giphy.com/media/8vQSQ3cNXuDGo/giphy.gif" alt="cat" loading="lazy" />
    </div>
    <div class="random-cat-wrap">
      <img id="random-cat" alt="random cat" />
      <button class="cat-btn" onclick="fetchCat()">show me another cat 🐾</button>
    </div>
  </div>

</div>

<script>
  const STORAGE_KEY = 'together-activities';

  function load() {
    try { return JSON.parse(localStorage.getItem(STORAGE_KEY)) || []; }
    catch (e) { return []; }
  }

  function save(items) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(items));
  }

  function escHtml(s) {
    return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
  }

  function render() {
    const items = load();
    const ul = document.getElementById('activity-list');
    const empty = document.getElementById('empty-msg');
    ul.innerHTML = '';
    empty.style.display = items.length ? 'none' : 'block';
    items.forEach(function(item, i) {
      const li = document.createElement('li');
      if (item.done) li.classList.add('done');
      li.innerHTML =
        '<input type="checkbox"' + (item.done ? ' checked' : '') + ' onchange="toggle(' + i + ')" />' +
        '<span>' + escHtml(item.text) + '</span>' +
        '<button class="del-btn" onclick="remove(' + i + ')" title="delete">✕</button>';
      ul.appendChild(li);
    });
  }

  function addActivity() {
    const input = document.getElementById('activity-input');
    const text = input.value.trim();
    if (!text) return;
    const items = load();
    items.push({ text: text, done: false });
    save(items);
    input.value = '';
    render();
  }

  function toggle(i) {
    const items = load();
    items[i].done = !items[i].done;
    save(items);
    render();
  }

  function remove(i) {
    const items = load();
    items.splice(i, 1);
    save(items);
    render();
  }

  function fetchCat() {
    const img = document.getElementById('random-cat');
    img.style.display = 'block';
    img.src = 'https://cataas.com/cat/gif?t=' + Date.now();
  }

  document.addEventListener('DOMContentLoaded', function() {
    document.getElementById('activity-input').addEventListener('keydown', function(e) {
      if (e.key === 'Enter') addActivity();
    });
    render();
  });
</script>
