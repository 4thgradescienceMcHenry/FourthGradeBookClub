<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="robots" content="noindex, nofollow">
<title>Reading Quest</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Nunito:wght@600;700;800;900&display=swap" rel="stylesheet">
<style>
  :root {
    --grass: #5DA130; --grass-dark: #3F7A1F; --dirt: #8B5A2B; --stone: #7D7D7D; --stone-dark: #4A4A4A;
    --wood: #A0703A; --wood-dark: #6B4A23; --gold: #F7C531; --gold-dark: #B8860B; --diamond: #4DE1E8;
    --emerald: #17C964; --redstone: #E03C31; --sky: #7EC8F5; --ink: #1E1E1E; --paper: #F4E9D4; --white: #FFFFFF;
    --px: "Press Start 2P", "Courier New", monospace;
  }
  * { box-sizing: border-box; }
  html { scroll-behavior: smooth; }
  body { margin: 0; font-family: "Nunito", Arial, sans-serif; font-weight: 700; color: var(--ink); background: var(--sky); font-size: 18px; line-height: 1.5; min-height: 100vh; }
  #world { position: fixed; inset: 0; width: 100%; height: 100%; z-index: 0; image-rendering: pixelated; }
  main { position: relative; z-index: 1; max-width: 1080px; margin: 0 auto; padding: 24px 16px 260px; }

  /* Pixel panels */
  .panel { background: var(--paper); border: 4px solid var(--ink); box-shadow: inset -6px -6px 0 rgba(0,0,0,.18), inset 6px 6px 0 rgba(255,255,255,.55), 6px 6px 0 rgba(0,0,0,.35); padding: 24px; margin-bottom: 28px; }
  .panel.stone { background: #9A9A9A; color: var(--white); box-shadow: inset -6px -6px 0 #5E5E5E, inset 6px 6px 0 #C4C4C4, 6px 6px 0 rgba(0,0,0,.35); }
  .panel.wood { background: var(--wood); color: var(--white); box-shadow: inset -6px -6px 0 var(--wood-dark), inset 6px 6px 0 #C48E52, 6px 6px 0 rgba(0,0,0,.35); }
  .panel.dark { background: #2B2B2B; color: var(--white); box-shadow: inset -6px -6px 0 #151515, inset 6px 6px 0 #4A4A4A, 6px 6px 0 rgba(0,0,0,.35); }
  h1, h2, h3, .px { font-family: var(--px); font-weight: 400; line-height: 1.4; margin: 0 0 16px; }
  h2 { font-size: 20px; }
  h3 { font-size: 14px; }
  .title { text-align: center; color: var(--white); font-size: clamp(26px, 6vw, 52px); text-shadow: 4px 4px 0 var(--ink), -2px -2px 0 var(--ink), 2px -2px 0 var(--ink), -2px 2px 0 var(--ink); margin: 28px 0 8px; letter-spacing: 2px; }
  .title .y { color: var(--gold); }
  .tagline { text-align: center; font-family: var(--px); color: var(--white); font-size: clamp(10px, 2vw, 14px); text-shadow: 2px 2px 0 var(--ink); margin: 0 0 30px; }
  .tagline .blink { animation: blink 1.2s steps(2) infinite; color: var(--gold); }
  @keyframes blink { 50% { opacity: 0; } }

  /* Buttons */
  .btn { font-family: var(--px); font-size: 13px; color: var(--white); background: var(--grass); border: 4px solid var(--ink); padding: 14px 20px; cursor: pointer; box-shadow: inset -4px -4px 0 var(--grass-dark), inset 4px 4px 0 #8BD150; text-shadow: 2px 2px 0 rgba(0,0,0,.4); }
  .btn:hover { filter: brightness(1.08); }
  .btn:active { box-shadow: inset 4px 4px 0 var(--grass-dark); }
  .btn:disabled { filter: grayscale(.6); cursor: wait; }
  .btn.small { font-size: 10px; padding: 8px 12px; }
  .btn.gray { background: #8A8A8A; box-shadow: inset -4px -4px 0 #555, inset 4px 4px 0 #BBB; }

  /* Forms */
  label { display: block; font-weight: 900; margin-bottom: 6px; }
  .hint { font-weight: 700; color: #6B5B45; font-size: 15px; }
  input[type=text], input[type=number], textarea { width: 100%; font: inherit; font-size: 18px; padding: 12px; border: 4px solid var(--ink); background: var(--white); color: var(--ink); box-shadow: inset 4px 4px 0 rgba(0,0,0,.12); border-radius: 0; }
  input:focus, textarea:focus { outline: 4px solid var(--gold); outline-offset: 0; }
  textarea { min-height: 140px; resize: vertical; }
  .grid2 { display: grid; grid-template-columns: 1fr 1fr; gap: 18px; }
  .field { margin-bottom: 18px; }
  .stars { display: flex; gap: 8px; }
  .stars button { width: 52px; height: 52px; border: 4px solid var(--ink); background: #CFCFCF; cursor: pointer; font-size: 28px; line-height: 1; color: #8A8A8A; box-shadow: inset -3px -3px 0 rgba(0,0,0,.2); }
  .stars button.on { background: var(--gold); color: #7A4F00; box-shadow: inset -3px -3px 0 var(--gold-dark), inset 3px 3px 0 #FFE58A; }
  .counter { font-family: var(--px); font-size: 10px; margin-top: 8px; color: var(--redstone); }
  .counter.ok { color: var(--grass-dark); }
  .msg { display: none; margin-top: 18px; padding: 14px 16px; border: 4px solid var(--ink); font-weight: 900; }
  .msg.show { display: block; }
  .msg.good { background: #C9F2A9; }
  .msg.bad { background: #FFC9C4; }
  .trap { position: absolute; left: -9999px; }
  .notice { background: var(--gold); border: 4px solid var(--ink); padding: 12px 16px; font-weight: 900; margin-bottom: 22px; }

  /* Login */
  .login { max-width: 560px; margin: 0 auto 28px; }
  .how { display: grid; grid-template-columns: repeat(3, 1fr); gap: 16px; max-width: 900px; margin: 0 auto; }
  .how .panel { text-align: center; margin: 0; }
  .block-icon { width: 64px; height: 64px; margin: 0 auto 12px; border: 4px solid var(--ink); image-rendering: pixelated; }

  /* Top readers podium */
  .podium { display: grid; grid-template-columns: 1fr 1.15fr 1fr; gap: 14px; align-items: end; margin-top: 6px; }
  .pod { text-align: center; padding: 16px 10px; border: 4px solid var(--ink); color: var(--ink); }
  .pod .crown { font-family: var(--px); font-size: 26px; }
  .pod .nm { font-family: var(--px); font-size: 13px; margin: 10px 0 6px; word-break: break-word; }
  .pod .meta { font-weight: 900; font-size: 15px; }
  .pod.p1 { background: var(--gold); min-height: 210px; box-shadow: inset -6px -6px 0 var(--gold-dark), inset 6px 6px 0 #FFE58A; }
  .pod.p2 { background: var(--diamond); min-height: 175px; box-shadow: inset -6px -6px 0 #22A9B0, inset 6px 6px 0 #A6F6F9; }
  .pod.p3 { background: var(--emerald); min-height: 150px; box-shadow: inset -6px -6px 0 #0E8C45, inset 6px 6px 0 #7CF0AE; }
  .pod.empty { opacity: .55; }
  .hall { display: flex; flex-wrap: wrap; gap: 10px; margin-top: 18px; }
  .chip { background: #3A3A3A; border: 3px solid var(--ink); padding: 6px 12px; font-weight: 900; font-size: 15px; color: var(--white); }
  .chip b { color: var(--gold); font-family: var(--px); font-size: 11px; margin-right: 6px; }
  .me-tag { background: var(--redstone); color: var(--white); font-family: var(--px); font-size: 9px; padding: 3px 6px; margin-left: 6px; vertical-align: middle; }

  /* Player stats */
  .player-top { display: flex; justify-content: space-between; align-items: flex-start; gap: 12px; flex-wrap: wrap; }
  .stats { display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; margin: 14px 0 18px; }
  .stat { background: rgba(0,0,0,.25); border: 4px solid var(--ink); padding: 14px; text-align: center; }
  .stat .n { font-family: var(--px); font-size: 24px; color: var(--gold); display: block; margin-bottom: 8px; }
  .xp { border: 4px solid var(--ink); background: #222; height: 30px; position: relative; }
  .xp .fill { height: 100%; width: 0; background: repeating-linear-gradient(90deg, #7CFC00 0 18px, #5FD000 18px 20px); transition: width 1s steps(15); }
  .xp .lbl { position: absolute; inset: 0; display: grid; place-items: center; font-family: var(--px); font-size: 10px; color: var(--white); text-shadow: 2px 2px 0 #000; }

  /* Leaderboard */
  .tabs { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 14px; }
  .tabs button { font-family: var(--px); font-size: 10px; padding: 8px 12px; border: 3px solid var(--ink); background: #777; color: var(--white); cursor: pointer; }
  .tabs button.on { background: var(--gold); color: var(--ink); }
  .board { width: 100%; border-collapse: collapse; background: var(--white); color: var(--ink); }
  .board th { font-family: var(--px); font-size: 10px; text-align: left; padding: 10px; background: var(--stone-dark); color: var(--white); position: sticky; top: 0; }
  .board td { padding: 9px 10px; border-bottom: 3px solid #DDD; font-weight: 800; }
  .board tr.r1 td { background: #FFF1B8; } .board tr.r2 td { background: #D7FAFB; } .board tr.r3 td { background: #CFF6DD; }
  .board tr.me td { background: #FFD9D4; border-bottom-color: var(--redstone); }
  .board .rk { font-family: var(--px); font-size: 11px; width: 60px; }
  .board-wrap { max-height: 620px; overflow-y: auto; border: 4px solid var(--ink); }
  .num { text-align: right; }

  /* Bookshelf + reviews */
  .shelf { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 14px; }
  .book { background: var(--white); color: var(--ink); border: 4px solid var(--ink); padding: 14px; box-shadow: inset -4px -4px 0 rgba(0,0,0,.12); }
  .book .t { font-weight: 900; font-size: 17px; }
  .book .a { color: #555; font-size: 15px; }
  .gold-stars { color: var(--gold-dark); letter-spacing: 2px; font-size: 18px; }
  .reviews { display: grid; gap: 14px; }
  .review { background: var(--white); color: var(--ink); border: 4px solid var(--ink); padding: 16px; box-shadow: inset -4px -4px 0 rgba(0,0,0,.12); }
  .review .head { display: flex; justify-content: space-between; gap: 10px; flex-wrap: wrap; margin-bottom: 6px; }
  .review .who { font-family: var(--px); font-size: 11px; color: var(--grass-dark); }
  .review .when { color: #777; font-size: 14px; }
  .review p { margin: 8px 0 0; font-weight: 700; }
  .empty-note { font-style: italic; opacity: .85; }

  .confetti { position: fixed; top: -20px; width: 14px; height: 14px; z-index: 60; border: 2px solid rgba(0,0,0,.4); animation: fall 2.8s linear forwards; }
  @keyframes fall { to { transform: translateY(110vh) rotate(360deg); } }
  .levelup { position: fixed; inset: 0; display: none; place-items: center; z-index: 55; pointer-events: none; }
  .levelup.show { display: grid; animation: pop 2.6s ease forwards; }
  .levelup span { font-family: var(--px); font-size: clamp(28px, 7vw, 64px); color: #7CFC00; text-shadow: 5px 5px 0 #000, -3px -3px 0 #000; }
  @keyframes pop { 0% { transform: scale(.3); opacity: 0; } 15% { transform: scale(1.1); opacity: 1; } 80% { opacity: 1; } 100% { opacity: 0; } }

  .hidden { display: none !important; }
  footer { text-align: center; font-family: var(--px); font-size: 10px; color: var(--white); text-shadow: 2px 2px 0 #000; }

  @media (max-width: 760px) {
    .grid2, .how, .stats { grid-template-columns: 1fr; }
    .podium { grid-template-columns: 1fr; }
    .pod { min-height: 0 !important; }
    .pod.p1 { order: -1; }
    .panel { padding: 18px 14px; }
    .board .hide-sm { display: none; }
  }
  @media (prefers-reduced-motion: reduce) { * { animation: none !important; transition: none !important; } }
</style>
</head>
<body>
<canvas id="world" aria-hidden="true"></canvas>

<main>
  <h1 class="title">READING <span class="y">QUEST</span></h1>
  <p class="tagline">Read books. Earn XP. Climb the leaderboard! <span class="blink">▮</span></p>
  <div class="notice hidden" id="preview">Preview mode: this site is not connected to the Reading Quest sheet yet. Sign in with any ID and name to try it with sample readers. (Teacher: add your Web App URL in index.html.)</div>

  <!-- ===== Sign in ===== -->
  <section id="loginView">
    <div class="panel login">
      <h2>Sign in</h2>
      <form id="loginForm" novalidate>
        <div class="field"><label for="sid">Student ID number</label><input type="text" id="sid" inputmode="numeric" autocomplete="off" required></div>
        <div class="field"><label for="sfirst">First name</label><input type="text" id="sfirst" autocomplete="given-name" required></div>
        <button class="btn" type="submit" id="loginBtn">▶ START QUEST</button>
        <div class="msg" id="loginMsg" role="status" aria-live="polite"></div>
      </form>
    </div>
    <div class="how">
      <div class="panel wood"><canvas class="block-icon" data-block="grass" width="64" height="64"></canvas><h3>1. READ</h3><p>Read any book you like, all the way to the end.</p></div>
      <div class="panel stone"><canvas class="block-icon" data-block="ore" width="64" height="64"></canvas><h3>2. LOG IT</h3><p>Add the title, author, minutes, stars, and a 3-sentence review.</p></div>
      <div class="panel dark"><canvas class="block-icon" data-block="gold" width="64" height="64"></canvas><h3>3. LEVEL UP</h3><p>Every 15 books is a big goal. Climb to the top of the leaderboard!</p></div>
    </div>
  </section>

  <!-- ===== Signed in ===== -->
  <section id="appView" class="hidden">
    <div class="panel dark" id="topReaders">
      <h2>🏆 TOP READERS</h2>
      <div class="podium" id="podium"></div>
      <div class="hall" id="hall"></div>
    </div>

    <div class="panel wood">
      <div class="player-top">
        <div>
          <h2 id="hello">Welcome!</h2>
          <div id="myClass"></div>
        </div>
        <button class="btn small gray" id="logout" type="button">SIGN OUT</button>
      </div>
      <div class="stats">
        <div class="stat"><span class="n" id="stBooks">0</span>Books read</div>
        <div class="stat"><span class="n" id="stMin">0</span>Minutes read</div>
        <div class="stat"><span class="n" id="stRank">-</span>Your rank</div>
      </div>
      <div class="xp" aria-label="Progress to next goal"><div class="fill" id="xpFill"></div><div class="lbl" id="xpLbl">0 / 15 BOOKS TO NEXT GOAL</div></div>
    </div>

    <div class="panel" id="logPanel">
      <h2>📖 LOG A BOOK</h2>
      <form id="bookForm" novalidate>
        <div class="grid2">
          <div class="field"><label for="title">Book title</label><input type="text" id="title" maxlength="150" required></div>
          <div class="field"><label for="author">Author</label><input type="text" id="author" maxlength="100" required></div>
        </div>
        <div class="grid2">
          <div class="field"><label for="minutes">Number of minutes read</label><input type="number" id="minutes" min="1" max="1000" inputmode="numeric" required></div>
          <div class="field"><span style="font-weight:900;display:block;margin-bottom:6px" id="starsLbl">Stars (1 to 5)</span><div class="stars" id="stars" role="radiogroup" aria-labelledby="starsLbl"></div></div>
        </div>
        <div class="field">
          <label for="review">Book review <span class="hint">Write at least 3 complete sentences. What was it about? What did you like? Who should read it?</span></label>
          <textarea id="review" maxlength="2000" required></textarea>
          <div class="counter" id="sentCount">0 / 3 SENTENCES</div>
        </div>
        <div class="trap" aria-hidden="true"><label for="website">Leave empty</label><input type="text" id="website" tabindex="-1" autocomplete="off"></div>
        <button class="btn" type="submit" id="addBtn">+ ADD TO MY BOOKSHELF</button>
        <div class="msg" id="bookMsg" role="status" aria-live="polite"></div>
      </form>
    </div>

    <div class="panel stone">
      <h2>📚 MY BOOKSHELF</h2>
      <div class="shelf" id="shelf"></div>
    </div>

    <div class="panel dark">
      <h2>⛏ LEADERBOARD</h2>
      <p style="margin-top:-6px">Every reader, from the most books at the top to the fewest at the bottom.</p>
      <div class="tabs" id="tabs"></div>
      <div class="board-wrap">
        <table class="board"><thead><tr><th>Rank</th><th>Reader</th><th class="hide-sm">Class</th><th class="num">Books</th><th class="num hide-sm">Minutes</th></tr></thead><tbody id="boardBody"></tbody></table>
      </div>
    </div>

    <div class="panel wood">
      <h2>💬 BOOK REVIEWS</h2>
      <p style="margin-top:-6px">See what other readers are saying. Newest reviews first.</p>
      <div class="reviews" id="reviews"></div>
    </div>
  </section>

  <footer>Only first names and last initials are shown. Have fun reading!</footer>
</main>

<div class="levelup" id="levelup"><span>LEVEL UP!</span></div>

<script>
  // ===== TEACHER SETUP: paste your Apps Script Web App URL between the quotes =====
  const WEB_APP_URL = "";
  // ================================================================================

  const $ = (id) => document.getElementById(id);
  const esc = (t) => String(t == null ? "" : t).replace(/[&<>"']/g, c => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" }[c]));
  const starStr = (n) => "★".repeat(n) + "☆".repeat(5 - n);
  let session = null, data = null, stars = 0, tab = "All", MIN_SENT = 3, GOAL = 15;
  const preview = !WEB_APP_URL;
  if (preview) $("preview").classList.remove("hidden");

  /* ---------- Pixel world background ---------- */
  const PAL = {
    grass: ["#5DA130", "#6DB43A", "#4E8F27", "#7BC043"], grassSide: ["#5DA130", "#4E8F27"],
    dirt: ["#8B5A2B", "#7A4E24", "#9C6A36", "#6E4520"], stone: ["#7D7D7D", "#8E8E8E", "#6B6B6B", "#9A9A9A"],
    wood: ["#6B4A23", "#7D5A2E", "#5A3D1C"], leaf: ["#3D8C2A", "#2F7320", "#4CA035", "#285F1B"],
    gold: ["#F7C531", "#E6B020", "#FFE066"], diamond: ["#4DE1E8", "#2BC4CC", "#A6F6F9"], emerald: ["#17C964", "#0FA04E"], red: ["#E03C31", "#B32A21"]
  };
  function rnd(seed) { let s = seed % 2147483647; if (s <= 0) s += 2147483646; return () => (s = s * 16807 % 2147483647) / 2147483647; }
  function tile(type, size, seed) {
    const c = document.createElement("canvas"); c.width = c.height = size;
    const g = c.getContext("2d"), r = rnd(seed), p = size / 8;
    for (let y = 0; y < 8; y++) for (let x = 0; x < 8; x++) {
      let pal = PAL[type] || PAL.stone;
      if (type === "grassBlock") pal = y < 2 || (y === 2 && r() < .5) ? PAL.grass : PAL.dirt;
      if (type === "ore") pal = r() < .18 ? (seed % 3 === 0 ? PAL.diamond : seed % 3 === 1 ? PAL.gold : PAL.emerald) : PAL.stone;
      if (type === "log") pal = (x === 0 || x === 7) ? ["#5A3D1C"] : PAL.wood;
      g.fillStyle = pal[Math.floor(r() * pal.length)];
      g.fillRect(x * p, y * p, p, p);
    }
    return c;
  }
  // icons on sign-in page
  document.querySelectorAll(".block-icon").forEach((cv, i) => {
    const map = { grass: "grassBlock", ore: "ore", gold: "gold" };
    cv.getContext("2d").drawImage(tile(map[cv.dataset.block], 64, 99 + i * 7), 0, 0);
  });

  const world = $("world"), wg = world.getContext("2d");
  let terrain = null, clouds = [], B = 40;
  function buildWorld() {
    const W = window.innerWidth, H = window.innerHeight;
    world.width = W; world.height = H;
    B = W < 600 ? 28 : 40;
    const cols = Math.ceil(W / B) + 1, rows = Math.ceil(H / B) + 1;
    terrain = document.createElement("canvas"); terrain.width = W; terrain.height = H;
    const t = terrain.getContext("2d"); t.imageSmoothingEnabled = false;
    const r = rnd(12345);
    const tiles = {};
    const get = (k, n) => (tiles[k + n] ||= tile(k, B, 1000 + n * 31 + k.length * 7));
    let h = 4;
    const heights = [];
    for (let x = 0; x < cols; x++) { h += Math.floor(r() * 3) - 1; h = Math.max(3, Math.min(6, h)); heights.push(h); }
    for (let x = 0; x < cols; x++) {
      const top = rows - heights[x];
      for (let y = top; y < rows; y++) {
        let k = y === top ? "grassBlock" : y < top + 2 ? "dirt" : (r() < .1 ? "ore" : "stone");
        t.drawImage(get(k, (x * 7 + y * 3) % 5), x * B, y * B);
      }
      // trees
      if (x > 1 && x < cols - 2 && r() < .09 && heights[x - 1] === heights[x]) {
        const trunkH = 3 + Math.floor(r() * 2);
        for (let i = 1; i <= trunkH; i++) t.drawImage(get("log", i % 3), x * B, (top - i) * B);
        for (let ly = -2; ly <= 0; ly++) for (let lx = -2; lx <= 2; lx++) {
          if (Math.abs(lx) === 2 && ly === -2) continue;
          t.drawImage(get("leaf", (lx + ly + 9) % 4), (x + lx) * B, (top - trunkH - 1 + ly) * B);
        }
        t.drawImage(get("leaf", 1), x * B, (top - trunkH - 4) * B);
      }
    }
    clouds = [];
    const n = Math.max(4, Math.round(W / 260));
    for (let i = 0; i < n; i++) clouds.push({ x: r() * W, y: 20 + r() * H * 0.35, w: 3 + Math.floor(r() * 4), h: 1 + Math.floor(r() * 2), s: 0.08 + r() * 0.15 });
    drawWorld();
  }
  function drawWorld() {
    const W = world.width, H = world.height;
    const grd = wg.createLinearGradient(0, 0, 0, H);
    grd.addColorStop(0, "#5FB4F0"); grd.addColorStop(1, "#BDE6FF");
    wg.fillStyle = grd; wg.fillRect(0, 0, W, H);
    // blocky sun
    wg.fillStyle = "#FFF6A8"; wg.fillRect(W - B * 4, B * 1.2, B * 2, B * 2);
    wg.fillStyle = "#FFE24D"; wg.fillRect(W - B * 3.75, B * 1.45, B * 1.5, B * 1.5);
    // clouds
    clouds.forEach(c => {
      const bw = B * 1.2;
      wg.fillStyle = "rgba(255,255,255,.92)";
      wg.fillRect(c.x, c.y, c.w * bw, c.h * bw * 0.6);
      wg.fillRect(c.x + bw, c.y - bw * 0.5, (c.w - 2) * bw, bw * 0.6);
      wg.fillStyle = "rgba(220,230,240,.9)";
      wg.fillRect(c.x, c.y + c.h * bw * 0.6 - 6, c.w * bw, 6);
    });
    wg.drawImage(terrain, 0, 0);
  }
  const reduce = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
  function animate() {
    clouds.forEach(c => { c.x += c.s; if (c.x > world.width + 50) c.x = -c.w * B * 1.2; });
    drawWorld();
    requestAnimationFrame(animate);
  }
  let rt; window.addEventListener("resize", () => { clearTimeout(rt); rt = setTimeout(buildWorld, 150); });
  buildWorld(); if (!reduce) requestAnimationFrame(animate);

  /* ---------- Talking to the Reading Quest sheet ---------- */
  async function api(body) {
    if (preview) return demoApi(body);
    const res = await fetch(WEB_APP_URL, { method: "POST", body: JSON.stringify(body) }); // text/plain: no preflight
    return res.json();
  }

  /* ---------- Sign in ---------- */
  function saveSession(s) { try { s ? sessionStorage.setItem("rq", JSON.stringify(s)) : sessionStorage.removeItem("rq"); } catch (e) {} }
  function loadSession() { try { return JSON.parse(sessionStorage.getItem("rq") || "null"); } catch (e) { return null; } }

  $("loginForm").addEventListener("submit", async (e) => {
    e.preventDefault();
    const id = $("sid").value.replace(/\D/g, ""), first = $("sfirst").value.trim();
    if (!id || !first) return show("loginMsg", "Type your Student ID and first name.", false);
    $("loginBtn").disabled = true; $("loginBtn").textContent = "LOADING...";
    try {
      const out = await api({ action: "login", id, first });
      if (!out.ok) { show("loginMsg", out.message, false); return; }
      session = { id, first }; saveSession(session); enter(out);
    } catch (err) {
      show("loginMsg", "We couldn't reach Reading Quest. Please try again in a minute.", false);
    } finally { $("loginBtn").disabled = false; $("loginBtn").textContent = "▶ START QUEST"; }
  });
  $("logout").addEventListener("click", () => { session = null; saveSession(null); $("appView").classList.add("hidden"); $("loginView").classList.remove("hidden"); window.scrollTo(0, 0); });

  function enter(d) {
    data = d; GOAL = d.goalEvery || 15; MIN_SENT = d.minSentences || 3;
    $("loginView").classList.add("hidden"); $("appView").classList.remove("hidden");
    renderAll(); window.scrollTo(0, 0);
  }

  /* ---------- Rendering ---------- */
  function renderAll() {
    const me = data.me, board = data.leaderboard || [];
    $("hello").textContent = "Welcome, " + me.name + "!";
    $("myClass").textContent = "Class " + me.cls;
    $("stBooks").textContent = me.books; $("stMin").textContent = me.minutes;
    const mine = board.find(r => r.me);
    $("stRank").textContent = mine && me.books > 0 ? "#" + mine.rank : "-";
    const into = me.books % GOAL, goals = Math.floor(me.books / GOAL);
    $("xpFill").style.width = (100 * into / GOAL) + "%";
    $("xpLbl").textContent = into + " / " + GOAL + " BOOKS TO NEXT GOAL" + (goals ? "  ·  GOALS: " + goals : "");
    updateSentences();

    // podium (top 3 readers with at least 1 book)
    const readers = board.filter(r => r.books > 0);
    const pods = [readers[1], readers[0], readers[2]], cls = ["p2", "p1", "p3"], icons = ["🥈", "👑", "🥉"];
    $("podium").innerHTML = pods.map((r, i) => r
      ? '<div class="pod ' + cls[i] + '"><div class="crown">' + icons[i] + '</div><div class="nm">' + esc(r.name) + (r.me ? '<span class="me-tag">YOU</span>' : '') + '</div><div class="meta">' + r.books + ' books · ' + r.minutes + ' min</div><div class="meta">Class ' + esc(r.cls) + '</div></div>'
      : '<div class="pod ' + cls[i] + ' empty"><div class="crown">' + icons[i] + '</div><div class="nm">???</div><div class="meta">Log a book to take this spot!</div></div>').join("");
    $("hall").innerHTML = readers.slice(3, 10).map(r => '<span class="chip"><b>#' + r.rank + '</b>' + esc(r.name) + ' · ' + r.books + (r.me ? '<span class="me-tag">YOU</span>' : '') + '</span>').join("");

    // bookshelf
    $("shelf").innerHTML = (data.myBooks || []).length
      ? data.myBooks.map(b => '<div class="book"><div class="t">' + esc(b.title) + '</div><div class="a">by ' + esc(b.author) + '</div><div class="gold-stars" aria-label="' + b.stars + ' stars">' + starStr(b.stars) + '</div><div class="a">' + b.minutes + ' minutes · ' + esc(b.date) + '</div></div>').join("")
      : '<p class="empty-note">Your bookshelf is empty. Log your first book above!</p>';

    // tabs + leaderboard
    const classes = ["All"].concat([...new Set(board.map(r => r.cls))].filter(Boolean).sort());
    $("tabs").innerHTML = classes.map(c => '<button type="button" class="' + (c === tab ? "on" : "") + '" data-c="' + esc(c) + '">' + esc(c === "All" ? "ALL CLASSES" : c) + '</button>').join("");
    $("tabs").querySelectorAll("button").forEach(b => b.addEventListener("click", () => { tab = b.dataset.c; renderAll(); }));
    const rows = tab === "All" ? board : board.filter(r => r.cls === tab);
    $("boardBody").innerHTML = rows.map(r => {
      const top = r.books > 0 && r.rank <= 3 ? " r" + r.rank : "";
      return '<tr class="' + top + (r.me ? " me" : "") + '"><td class="rk">#' + r.rank + '</td><td>' + esc(r.name) + (r.me ? '<span class="me-tag">YOU</span>' : '') + '</td><td class="hide-sm">' + esc(r.cls) + '</td><td class="num">' + r.books + '</td><td class="num hide-sm">' + r.minutes + '</td></tr>';
    }).join("");

    // reviews
    const rv = data.reviews || [];
    $("reviews").innerHTML = rv.length
      ? rv.map(r => '<article class="review"><div class="head"><span class="who">' + esc(r.name) + ' · ' + esc(r.cls) + '</span><span class="when">' + esc(r.date) + '</span></div><div class="t" style="font-weight:900;font-size:18px">' + esc(r.title) + ' <span style="font-weight:700;color:#555;font-size:15px">by ' + esc(r.author) + '</span></div><div class="gold-stars" aria-label="' + r.stars + ' stars">' + starStr(r.stars) + '</div><p>' + esc(r.review) + '</p></article>').join("")
      : '<p class="empty-note">No reviews yet. Be the first!</p>';
  }

  function show(id, text, good) { const m = $(id); m.textContent = text; m.className = "msg show " + (good ? "good" : "bad"); }

  /* ---------- Stars + sentence counter ---------- */
  for (let i = 1; i <= 5; i++) {
    const b = document.createElement("button");
    b.type = "button"; b.textContent = "★"; b.setAttribute("role", "radio"); b.setAttribute("aria-label", i + (i === 1 ? " star" : " stars")); b.setAttribute("aria-checked", "false");
    b.addEventListener("click", () => setStars(i));
    $("stars").appendChild(b);
  }
  function setStars(n) { stars = n; [...$("stars").children].forEach((b, i) => { b.classList.toggle("on", i < n); b.setAttribute("aria-checked", String(i + 1 === n)); }); }
  const countSentences = (t) => t.split(/[.!?]+/).map(x => x.trim()).filter(x => x.split(/\s+/).filter(Boolean).length >= 3).length;
  function updateSentences() {
    const n = countSentences($("review").value), el = $("sentCount");
    el.textContent = Math.min(n, 99) + " / " + MIN_SENT + " SENTENCES" + (n >= MIN_SENT ? "  ✓" : "");
    el.classList.toggle("ok", n >= MIN_SENT);
  }
  $("review").addEventListener("input", updateSentences);

  /* ---------- Log a book ---------- */
  $("bookForm").addEventListener("submit", async (e) => {
    e.preventDefault();
    const b = { action: "log", id: session.id, first: session.first, title: $("title").value.trim(), author: $("author").value.trim(), minutes: $("minutes").value, stars, review: $("review").value.trim(), website: $("website").value };
    if (!b.title || !b.author) return show("bookMsg", "Please add the book title and the author.", false);
    if (!(Number(b.minutes) >= 1 && Number(b.minutes) <= 1000)) return show("bookMsg", "Please enter how many minutes you read.", false);
    if (!stars) return show("bookMsg", "Please choose 1 to 5 stars.", false);
    if (countSentences(b.review) < MIN_SENT) return show("bookMsg", "Your book review needs at least " + MIN_SENT + " complete sentences.", false);
    $("addBtn").disabled = true; $("addBtn").textContent = "SAVING...";
    try {
      const out = await api(b);
      if (!out.ok) { show("bookMsg", out.message, false); return; }
      data = out; $("bookForm").reset(); setStars(0); renderAll();
      show("bookMsg", out.message, true);
      celebrate(out.levelUp);
    } catch (err) {
      show("bookMsg", "We couldn't reach Reading Quest. Please try again in a minute.", false);
    } finally { $("addBtn").disabled = false; $("addBtn").textContent = "+ ADD TO MY BOOKSHELF"; }
  });

  function celebrate(big) {
    if (reduce) return;
    const colors = ["#5DA130", "#8B5A2B", "#F7C531", "#4DE1E8", "#17C964", "#E03C31", "#7D7D7D"];
    for (let i = 0; i < (big ? 140 : 60); i++) {
      const c = document.createElement("div"); c.className = "confetti";
      c.style.left = Math.random() * 100 + "vw"; c.style.background = colors[i % colors.length];
      c.style.animationDelay = Math.random() * 0.8 + "s"; document.body.appendChild(c);
      setTimeout(() => c.remove(), 3800);
    }
    if (big) { const l = $("levelup"); l.classList.remove("show"); void l.offsetWidth; l.classList.add("show"); }
  }

  /* ---------- Preview mode (no sheet connected yet) ---------- */
  const DEMO = { goal: 15, readers: [
    ["Ava", "R", "TIH", 17, 980], ["Marcus", "T", "TIN", 14, 760], ["Zoe", "K", "TIU", 12, 690], ["Liam", "B", "TIR", 9, 455], ["Maya", "S", "TIH", 8, 400],
    ["Noah", "P", "TIU", 6, 310], ["Ella", "W", "TIN", 5, 260], ["Jaden", "M", "TIR", 3, 150], ["Ruby", "C", "TIH", 2, 90], ["Owen", "D", "TIU", 0, 0], ["Isla", "F", "TIN", 0, 0]
  ], reviews: [
    { name: "Ava R.", cls: "TIH", title: "Charlotte's Web", author: "E. B. White", stars: 5, review: "Wilbur is a pig who is scared of being sent away. Charlotte the spider writes words in her web to save him. I loved how loyal she was, and anyone who likes animals should read it.", date: "Oct 27, 2026" },
    { name: "Marcus T.", cls: "TIN", title: "The Magic School Bus: Inside the Earth", author: "Joanna Cole", stars: 4, review: "Ms. Frizzle takes her class deep underground. They learn about rocks, lava, and layers of the Earth. It was funny and full of science facts." , date: "Oct 26, 2026" }
  ], mine: [] };
  function demoApi(b) {
    return new Promise(res => setTimeout(() => {
      if (!b.id || !b.first) return res({ ok: false, message: "Type your Student ID and first name." });
      const myName = b.first.charAt(0).toUpperCase() + b.first.slice(1).toLowerCase() + " (you)";
      let levelUp = false, message = "";
      if (b.action === "log") {
        if (DEMO.mine.some(x => x.title.toLowerCase() === b.title.toLowerCase())) return res({ ok: false, message: 'You already logged "' + b.title + '". Pick a new book!' });
        DEMO.mine.push({ title: b.title, author: b.author, minutes: Number(b.minutes), stars: b.stars, date: "Today" });
        DEMO.reviews.unshift({ name: myName, cls: "TIH", title: b.title, author: b.author, stars: b.stars, review: b.review, date: "Today" });
        const n = DEMO.mine.length; levelUp = n % DEMO.goal === 0; const left = DEMO.goal - (n % DEMO.goal);
        message = levelUp ? "LEVEL UP! You reached " + n + " books!" : "Book added! That is book #" + n + ". Only " + left + " more to your next goal.";
      }
      const mineMin = DEMO.mine.reduce((t, x) => t + x.minutes, 0);
      const board = DEMO.readers.map(r => ({ name: r[0] + " " + r[1] + ".", cls: r[2], books: r[3], minutes: r[4], me: false }))
        .concat([{ name: myName, cls: "TIH", books: DEMO.mine.length, minutes: mineMin, me: true }])
        .sort((a, c) => c.books - a.books || c.minutes - a.minutes);
      board.forEach((r, i) => r.rank = i + 1);
      res({ ok: true, message, levelUp, goalEvery: DEMO.goal, minSentences: 3, me: { name: myName, cls: "TIH", books: DEMO.mine.length, minutes: mineMin }, myBooks: DEMO.mine.slice().reverse(), leaderboard: board, reviews: DEMO.reviews });
    }, 300));
  }

  /* ---------- Resume a session after refresh ---------- */
  (async () => {
    const s = loadSession();
    if (!s) return;
    try { const out = await api({ action: "login", id: s.id, first: s.first }); if (out.ok) { session = s; enter(out); } } catch (e) {}
  })();
</script>
</body>
</html>
