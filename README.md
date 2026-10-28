/**
 * READING QUEST — backend for the GitHub Pages website
 * ----------------------------------------------------
 * Lives in the Reading Quest Google Sheet (Extensions > Apps Script).
 * Deploy > New deployment > Web app (Execute as: Me, Who has access: Anyone),
 * then paste the Web app URL into index.html (WEB_APP_URL).
 *
 * Students sign in with Student ID + first name (checked against the Roster tab).
 * Each book log saves Title, Author, Minutes Read, Stars (1-5), and a 3-sentence review.
 * Every GOAL_EVERY books (default 15) for a student, TEACHER_EMAIL gets an email.
 * The website shows only first name + last initial, never IDs.
 */

const SH = { roster: 'Roster', log: 'Book Log', settings: 'Settings' };
// Book Log columns
const L = { time: 1, id: 2, first: 3, last: 4, cls: 5, title: 6, author: 7, minutes: 8, stars: 9, review: 10, num: 11, goal: 12 };

/* ---------------- Menu (inside the sheet) ---------------- */

function onOpen() {
  SpreadsheetApp.getUi().createMenu('Reading Quest')
    .addItem('Send a test goal email to me', 'testGoalEmail')
    .addItem('Show top 10 readers', 'showTop')
    .addToUi();
}

function testGoalEmail() {
  const s = settings_();
  sendGoalEmail_(s, { id: '00000000', first: 'Sample', last: 'Student', cls: 'TIH' }, Number(s.GOAL_EVERY) || 15, 420,
    [{ title: 'Charlotte\'s Web', author: 'E. B. White', minutes: 45, stars: 5 }]);
  SpreadsheetApp.getUi().alert('Test email sent to ' + s.TEACHER_EMAIL);
}

function showTop() {
  const board = leaderboard_(null).slice(0, 10);
  SpreadsheetApp.getUi().alert(board.map(r => r.rank + '. ' + r.name + ' (' + r.cls + ') - ' + r.books + ' books, ' + r.minutes + ' min').join('\n') || 'No books yet.');
}

/* ---------------- Web app ---------------- */

function doGet() {
  return out_({ ok: true, message: 'Reading Quest is running.' });
}

function doPost(e) {
  let d;
  try { d = JSON.parse(e.postData.contents || '{}'); } catch (err) { return out_({ ok: false, message: 'Something went wrong. Please try again.' }); }
  if (d.website) return out_({ ok: false, message: 'Something went wrong. Please try again.' }); // spam trap
  const st = auth_(d.id, d.first);
  if (!st) return out_({ ok: false, code: 'auth', message: "We couldn't find that Student ID and first name. Check them and try again, or ask your teacher." });
  try {
    if (d.action === 'log') return out_(logBook_(st, d));
    return out_(Object.assign({ ok: true }, state_(st)));
  } catch (err) {
    return out_({ ok: false, message: 'Something went wrong. Please try again in a minute.' });
  }
}

/* ---------------- Logging a book ---------------- */

function logBook_(st, d) {
  const s = settings_();
  const title = clean_(d.title, 150), author = clean_(d.author, 100), review = clean_(d.review, 2000);
  const minutes = Math.round(Number(d.minutes)), stars = Math.round(Number(d.stars));
  const need = Number(s.MIN_SENTENCES) || 3;

  if (!title || !author) return { ok: false, message: 'Please add the book title and the author.' };
  if (!(minutes >= 1 && minutes <= 1000)) return { ok: false, message: 'Please enter how many minutes you read (1 to 1000).' };
  if (!(stars >= 1 && stars <= 5)) return { ok: false, message: 'Please choose 1 to 5 stars.' };
  if (sentences_(review) < need || !looksReal_(review)) return { ok: false, message: 'Your book review needs at least ' + need + ' complete sentences.' };

  const lock = LockService.getScriptLock();
  lock.waitLock(30000);
  try {
    const log = sheet_(SH.log);
    const mine = booksFor_(st.id);
    if (mine.some(b => norm_(b.title) === norm_(title))) return { ok: false, message: 'You already logged "' + title + '". Pick a new book to keep climbing!' };

    const num = mine.length + 1;
    const goal = Number(s.GOAL_EVERY) || 15;
    const hitGoal = num % goal === 0;
    log.appendRow([new Date(), Number(st.id), st.first, st.last, st.cls, title, author, minutes, stars, review, num, '']);

    if (hitGoal) {
      const all = booksFor_(st.id);
      const totalMin = all.reduce((t, b) => t + b.minutes, 0);
      try {
        sendGoalEmail_(s, st, num, totalMin, all.slice(-goal));
        log.getRange(log.getLastRow(), L.goal).setValue('Emailed ' + new Date().toLocaleDateString());
      } catch (err) {
        log.getRange(log.getLastRow(), L.goal).setValue('Email failed: ' + err);
      }
    }
    const left = goal - (num % goal);
    const message = hitGoal
      ? 'LEVEL UP! You reached ' + num + ' books! Your teacher has been told about your awesome reading.'
      : 'Book added! That is book #' + num + '. Only ' + left + (left === 1 ? ' more book' : ' more books') + ' to your next goal.';
    return Object.assign({ ok: true, message: message, levelUp: hitGoal }, state_(st));
  } finally {
    lock.releaseLock();
  }
}

function sendGoalEmail_(s, st, num, totalMin, recent) {
  const to = s.TEACHER_EMAIL;
  if (!to) return;
  const rows = recent.map(b => '<tr><td>' + esc_(b.title) + '</td><td>' + esc_(b.author) + '</td><td>' + b.minutes + '</td><td>' + '★'.repeat(b.stars) + '</td></tr>').join('');
  const html =
    '<h2 style="font-family:Arial">🎉 ' + esc_(st.first + ' ' + st.last) + ' reached ' + num + ' books!</h2>' +
    '<p style="font-family:Arial"><b>Student ID:</b> ' + esc_(st.id) + '<br><b>Class:</b> ' + esc_(st.cls) +
    '<br><b>Total books:</b> ' + num + '<br><b>Total minutes read:</b> ' + totalMin + '</p>' +
    '<p style="font-family:Arial"><b>Their most recent books:</b></p>' +
    '<table border="1" cellpadding="6" style="border-collapse:collapse;font-family:Arial;font-size:14px"><tr><th>Title</th><th>Author</th><th>Minutes</th><th>Stars</th></tr>' + rows + '</table>' +
    '<p style="font-family:Arial;color:#555">Sent automatically by ' + esc_(s.CLUB_NAME || 'Reading Quest') + '.</p>';
  MailApp.sendEmail(to, (s.CLUB_NAME || 'Reading Quest') + ': ' + st.first + ' ' + st.last + ' reached ' + num + ' books!',
    st.first + ' ' + st.last + ' (ID ' + st.id + ', ' + st.cls + ') reached ' + num + ' books and ' + totalMin + ' minutes.', { htmlBody: html, name: s.CLUB_NAME || 'Reading Quest' });
}

/* ---------------- What the website shows ---------------- */

function state_(st) {
  const s = settings_();
  const mine = booksFor_(st.id).map(b => ({ title: b.title, author: b.author, minutes: b.minutes, stars: b.stars, date: b.date }));
  return {
    club: s.CLUB_NAME || 'Reading Quest',
    goalEvery: Number(s.GOAL_EVERY) || 15,
    minSentences: Number(s.MIN_SENTENCES) || 3,
    me: { name: display_(st.first, st.last), cls: st.cls, books: mine.length, minutes: mine.reduce((t, b) => t + b.minutes, 0) },
    myBooks: mine.reverse(),
    leaderboard: leaderboard_(st.id),
    reviews: reviews_(Number(s.REVIEWS_SHOWN) || 100)
  };
}

/** Every student on the roster, most books at the top, fewest at the bottom. */
function leaderboard_(meId) {
  const roster = roster_();
  const tally = {};
  logRows_().forEach(r => {
    const id = String(r[L.id - 1]);
    tally[id] = tally[id] || { books: 0, minutes: 0 };
    tally[id].books++; tally[id].minutes += Number(r[L.minutes - 1]) || 0;
  });
  const board = roster.map(p => ({
    name: display_(p.first, p.last), cls: p.cls,
    books: (tally[p.id] || {}).books || 0, minutes: (tally[p.id] || {}).minutes || 0,
    me: meId != null && p.id === String(meId)
  }));
  board.sort((a, b) => b.books - a.books || b.minutes - a.minutes || a.name.localeCompare(b.name));
  let rank = 0, prev = null;
  board.forEach((r, i) => { const key = r.books + '|' + r.minutes; if (key !== prev) rank = i + 1; prev = key; r.rank = rank; });
  return board;
}

function reviews_(n) {
  return logRows_().slice(-n).reverse().map(r => ({
    name: display_(r[L.first - 1], r[L.last - 1]), cls: String(r[L.cls - 1]),
    title: String(r[L.title - 1]), author: String(r[L.author - 1]),
    stars: Number(r[L.stars - 1]) || 0, minutes: Number(r[L.minutes - 1]) || 0,
    review: String(r[L.review - 1]),
    date: fmtDate_(r[L.time - 1])
  }));
}

/* ---------------- Helpers ---------------- */

function sheet_(n) { return SpreadsheetApp.getActive().getSheetByName(n); }

function settings_() {
  const sh = sheet_(SH.settings);
  const out = {};
  sh.getRange(2, 1, Math.max(sh.getLastRow() - 1, 1), 2).getValues().forEach(([k, v]) => { if (k) out[String(k).trim()] = String(v).trim(); });
  return out;
}

function roster_() {
  const sh = sheet_(SH.roster);
  return sh.getRange(2, 1, Math.max(sh.getLastRow() - 1, 1), 4).getValues()
    .filter(r => r[0] !== '')
    .map(r => ({ id: String(r[0]).replace(/\D/g, ''), first: String(r[1]).trim(), last: String(r[2]).trim(), cls: String(r[3]).trim() }));
}

function logRows_() {
  const sh = sheet_(SH.log);
  const last = sh.getLastRow();
  return last < 2 ? [] : sh.getRange(2, 1, last - 1, L.goal).getValues().filter(r => r[L.id - 1] !== '');
}

function booksFor_(id) {
  return logRows_().filter(r => String(r[L.id - 1]) === String(id)).map(r => ({
    title: String(r[L.title - 1]), author: String(r[L.author - 1]),
    minutes: Number(r[L.minutes - 1]) || 0, stars: Number(r[L.stars - 1]) || 0, date: fmtDate_(r[L.time - 1])
  }));
}

/** Student ID must be on the Roster, and the first name must match (nicknames in quotes also work). */
function auth_(id, first) {
  const cleanId = String(id || '').replace(/\D/g, '');
  const typed = norm_(first);
  if (!cleanId || !typed) return null;
  const p = roster_().find(r => r.id === cleanId);
  if (!p) return null;
  const options = [norm_(p.first)].concat(String(p.first).split(/[\s"“”()]+/).map(norm_)).filter(Boolean);
  return options.indexOf(typed) > -1 ? p : null;
}

function display_(first, last) {
  const f = String(first).replace(/\s+/g, ' ').trim();
  const initial = String(last).trim().charAt(0).toUpperCase();
  return f + (initial ? ' ' + initial + '.' : '');
}

function sentences_(t) {
  return String(t || '').split(/[.!?]+/).map(x => x.trim()).filter(x => x.split(/\s+/).filter(Boolean).length >= 3).length;
}

function looksReal_(t) {
  const s = String(t || '').trim();
  if (s.length < 40 || /(.)\1{5,}/.test(s)) return false;
  const words = s.toLowerCase().split(/\s+/).filter(Boolean);
  return new Set(words).size >= 6;
}

function clean_(t, max) { return String(t || '').replace(/[\u0000-\u001F]/g, ' ').trim().slice(0, max); }
function norm_(t) { return String(t || '').toLowerCase().replace(/[^a-z]/g, ''); }
function esc_(t) { return String(t).replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;'); }
function fmtDate_(d) { return d instanceof Date ? Utilities.formatDate(d, Session.getScriptTimeZone(), 'MMM d, yyyy') : String(d || ''); }
function out_(o) { return ContentService.createTextOutput(JSON.stringify(o)).setMimeType(ContentService.MimeType.JSON); }
