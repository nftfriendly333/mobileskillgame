<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Iron Arena — PvE Hub</title>
<link href="https://fonts.googleapis.com/css2?family=Cinzel+Decorative:wght@700;900&family=Cinzel:wght@400;600&family=IM+Fell+English:ital@0;1&display=swap" rel="stylesheet">
<!-- Firebase SDK — initialized via Cloudflare Worker -->
<script type="module">
  import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js';
  import { getDatabase, ref, set, get, child, query, orderByKey, limitToFirst, onValue, push, serverTimestamp }
    from 'https://www.gstatic.com/firebasejs/10.12.0/firebase-database.js';

  window._fbReady = false;
  window._fbDb    = null;

  const firebaseConfig = {
    apiKey:            "AIzaSyA76CnkQAK5vmYqv30nO8BL77e1kPj76IM",
    authDomain:        "the-best-iron-arena.firebaseapp.com",
    projectId:         "the-best-iron-arena",
    storageBucket:     "the-best-iron-arena.firebasestorage.app",
    messagingSenderId: "584814007725",
    appId:             "1:584814007725:web:acc65cddc620f130fdfd24",
    databaseURL:       "https://the-best-iron-arena-default-rtdb.firebaseio.com"
  };

  try {
    const { getApps } = await import('https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js');
    const app = getApps().length ? getApps()[0] : initializeApp(firebaseConfig);
    window._fbDb          = getDatabase(app);
    window._fbSet         = set;
    window._fbRef         = ref;
    window._fbGet         = get;
    window._fbChild       = child;
    window._fbQuery       = query;
    window._fbOrderByKey  = orderByKey;
    window._fbLimitToFirst = limitToFirst;
    window._fbOnValue      = onValue;
    window._fbPush         = push;
    window._fbServerTs     = serverTimestamp;
    window._fbReady       = true;
    window.WORKER_URL     = null; // Worker not in use — direct config active
    console.log('Firebase connected ✓');
  } catch(e) {
    console.warn('Firebase init failed:', e);
  }
</script>
<style>
  :root {
    --bg: #0a0806;
    --bg2: #110e0a;
    --bg3: #1a1410;
    --panel: #1e1812;
    --border: #3a2e20;
    --border2: #5a4530;
    --gold: #c9a84c;
    --gold2: #e8c96a;
    --gold3: #7a5c1e;
    --red: #c0392b;
    --red2: #e74c3c;
    --red3: #7a1a10;
    --blue: #2980b9;
    --blue2: #3498db;
    --green: #27ae60;
    --green2: #2ecc71;
    --orange: #e67e22;
    --purple: #8e44ad;
    --text: #d4b896;
    --text2: #a08060;
    --text3: #6a5040;
    --white: #f0e8d8;
    --shadow: rgba(0,0,0,0.8);
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'IM Fell English', serif;
    min-height: 100vh;
    overflow-x: hidden;
    background-image:
      radial-gradient(ellipse at 20% 80%, rgba(100,60,10,0.08) 0%, transparent 60%),
      radial-gradient(ellipse at 80% 20%, rgba(80,40,10,0.06) 0%, transparent 50%);
  }

  /* Scanline overlay */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background: repeating-linear-gradient(0deg, transparent, transparent 3px, rgba(0,0,0,0.03) 3px, rgba(0,0,0,0.03) 4px);
    pointer-events: none;
    z-index: 9999;
  }

  h1, h2, h3 { font-family: 'Cinzel Decorative', serif; }

  .game-header {
    text-align: center;
    padding: 2rem 1rem 1rem;
    border-bottom: 1px solid var(--border);
    position: relative;
  }

  .game-header h1 {
    font-size: clamp(1.4rem, 4vw, 2.8rem);
    color: var(--gold);
    letter-spacing: 0.15em;
    text-shadow: 0 0 30px rgba(201,168,76,0.4), 0 2px 4px black;
    animation: titlePulse 4s ease-in-out infinite;
  }

  @keyframes titlePulse {
    0%, 100% { text-shadow: 0 0 30px rgba(201,168,76,0.4), 0 2px 4px black; }
    50% { text-shadow: 0 0 60px rgba(201,168,76,0.7), 0 2px 4px black; }
  }

  .game-header p {
    color: var(--text2);
    font-style: italic;
    font-size: 1.15rem;
    margin-top: 0.3rem;
    letter-spacing: 0.1em;
  }

  .ornament {
    color: var(--gold3);
    font-size: 1.8rem;
    margin: 0 0.5rem;
  }

  /* ── LAYOUT: mobile-first single column ── */
  .game-container {
    max-width: 860px;
    margin: 0 auto;
    padding: 1rem;
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  /* Order: 1=arena, 2=shop, 3=skills */
  .arena-col   { order: 1; }
  .shop-panel  { order: 2; }
  .skills-panel{ order: 3; }

  /* Skills grid: 2 columns on mobile */
  .skills-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }

  /* Desktop: side-by-side for shop + skills, arena full width on top */
  @media (min-width: 700px) {
    .game-container {
      max-width: 1100px;
      display: grid;
      grid-template-columns: 1fr 1fr;
      grid-template-rows: auto auto;
      gap: 1.2rem;
    }
    .arena-col    { grid-column: 1 / -1; order: 1; }
    .shop-panel   { grid-column: 1; order: 2; }
    .skills-panel { grid-column: 2; order: 3; }
    .skills-grid  { grid-template-columns: 1fr 1fr; }
  }

  @media (min-width: 1000px) {
    .game-container { max-width: 1200px; gap: 1.5rem; }
  }

  /* PANEL styles */
  .panel {
    background: var(--panel);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 1.4rem;
    position: relative;
  }

  .panel::before {
    content: '';
    position: absolute;
    inset: 3px;
    border: 1px solid var(--border3, transparent);
    border-radius: 2px;
    pointer-events: none;
  }

  .panel-title {
    font-family: 'Cinzel', serif;
    font-size: 0.9rem;
    font-weight: 600;
    letter-spacing: 0.25em;
    color: var(--gold3);
    text-transform: uppercase;
    margin-bottom: 1rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    gap: 0.4rem;
  }

  /* SKILL SECTION */
  .skills-panel { grid-column: 1; }

  .skill-row {
    margin-bottom: 1rem;
  }

  .skill-label {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 0.3rem;
    font-size: 1.15rem;
  }

  .skill-name {
    display: flex;
    align-items: center;
    gap: 0.4rem;
    color: var(--text);
    font-style: italic;
  }

  .skill-icon { font-size: 1.5rem; }

  .skill-val {
    font-family: 'Cinzel', serif;
    font-size: 0.95rem;
    color: var(--gold);
    background: rgba(201,168,76,0.1);
    border: 1px solid var(--gold3);
    padding: 0.1rem 0.4rem;
    border-radius: 2px;
  }

  .skill-bar-bg {
    height: 8px;
    background: rgba(255,255,255,0.05);
    border-radius: 3px;
    border: 1px solid var(--border);
    overflow: hidden;
  }

  .skill-bar-fill {
    height: 100%;
    border-radius: 3px;
    transition: width 0.5s ease;
  }

  .bar-stamina { background: linear-gradient(90deg, #27ae60, #2ecc71); }
  .bar-attack  { background: linear-gradient(90deg, #c0392b, #e74c3c); }
  .bar-defense { background: linear-gradient(90deg, #2980b9, #3498db); }
  .bar-crit    { background: linear-gradient(90deg, #8e44ad, #9b59b6); }

  .skill-xp-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 0.2rem;
    font-size: 0.9rem;
    color: var(--text3);
  }

  .xp-progress-bg {
    height: 3px;
    background: rgba(255,255,255,0.04);
    border-radius: 2px;
    flex: 1;
    margin: 0 0.5rem;
    overflow: hidden;
  }

  .xp-progress-fill {
    height: 100%;
    background: var(--gold3);
    transition: width 0.4s ease;
    border-radius: 2px;
  }

  .btn-levelup {
    width: 100%;
    margin-top: 0.5rem;
    padding: 0.4rem;
    background: linear-gradient(135deg, #1e1812, #2a2016);
    border: 1px solid var(--gold3);
    color: var(--gold);
    font-family: 'Cinzel', serif;
    font-size: 0.85rem;
    letter-spacing: 0.15em;
    cursor: pointer;
    border-radius: 2px;
    transition: all 0.2s;
    position: relative;
    overflow: hidden;
  }

  .btn-levelup:hover:not(:disabled) {
    background: linear-gradient(135deg, #2a2016, #3a2e1a);
    border-color: var(--gold);
    color: var(--gold2);
    box-shadow: 0 0 8px rgba(201,168,76,0.2);
  }

  .btn-levelup:disabled {
    opacity: 0.4;
    cursor: not-allowed;
    border-color: var(--border);
    color: var(--text3);
  }

  .btn-levelup .progress-bar {
    position: absolute;
    left: 0;
    bottom: 0;
    height: 2px;
    background: var(--gold3);
    transition: width 0.1s linear;
  }

  .cooldown-text {
    font-size: 0.8rem;
    color: var(--text3);
    text-align: center;
    margin-top: 0.2rem;
    min-height: 0.8rem;
    font-family: 'Cinzel', serif;
    letter-spacing: 0.1em;
  }

  /* MAIN ARENA */
  .arena-col {
    grid-column: 2;
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  /* COMBATANT DISPLAY */
  .combatants {
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    gap: 1rem;
    align-items: start;
  }

  .vs-divider {
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: 'Cinzel Decorative', serif;
    font-size: 1.5rem;
    color: var(--gold3);
    padding-top: 2rem;
    text-shadow: 0 0 20px rgba(201,168,76,0.3);
  }

  .combatant-card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 1rem;
    text-align: center;
    position: relative;
    transition: border-color 0.3s;
  }

  .combatant-card.active-turn {
    border-color: var(--gold);
    box-shadow: 0 0 15px rgba(201,168,76,0.15);
  }

  /* ── COMBATANT BASE ── */
  .combatant-sprite {
    font-size: 4rem;
    line-height: 1;
    margin-bottom: 0.5rem;
    display: block;
    filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6));
    transform-origin: center bottom;
    will-change: transform;
  }

  /* ── HIT REACTION ── */
  .combatant-card.taking-hit { animation: hitRecoil 0.45s ease forwards; }
  @keyframes hitRecoil {
    0%   { transform: translateX(0)    rotate(0deg)  scale(1);    filter: brightness(1); }
    15%  { transform: translateX(12px) rotate(3deg)  scale(0.93); filter: brightness(2.5) saturate(0); }
    35%  { transform: translateX(-8px) rotate(-2deg) scale(0.96); filter: brightness(1.8); }
    60%  { transform: translateX(5px)  rotate(1deg)  scale(0.98); filter: brightness(1.3); }
    100% { transform: translateX(0)    rotate(0deg)  scale(1);    filter: brightness(1); }
  }

  /* ── LIGHT ATTACK: quick dart + snap ── */
  .combatant-card.attack-light .combatant-sprite {
    animation: lightStrike 0.55s cubic-bezier(0.25,0.46,0.45,0.94);
  }
  @keyframes lightStrike {
    0%   { transform: translateX(0)    scale(1)    rotate(0deg); filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)); }
    10%  { transform: translateX(-18px) scale(0.9) rotate(-8deg); }
    35%  { transform: translateX(70px)  scale(1.25) rotate(12deg); filter: drop-shadow(0 0 18px rgba(255,200,50,0.9)); }
    55%  { transform: translateX(50px)  scale(1.1) rotate(6deg); }
    75%  { transform: translateX(8px)   scale(1.02) rotate(1deg); }
    100% { transform: translateX(0)    scale(1)    rotate(0deg); filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)); }
  }

  /* ── HEAVY ATTACK: wind-up + ground-slam ── */
  .combatant-card.attack-heavy .combatant-sprite {
    animation: heavySlam 0.9s cubic-bezier(0.22,0.61,0.36,1);
  }
  @keyframes heavySlam {
    0%   { transform: translateX(0)      translateY(0)    scale(1)    rotate(0deg);
           filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)) brightness(1); opacity:1; }
    /* Wind-up: pull way back, grow huge */
    10%  { transform: translateX(-50px)  translateY(-15px) scale(1.5) rotate(-25deg);
           filter: drop-shadow(0 0 15px rgba(255,80,0,0.6)) brightness(1.3); }
    18%  { transform: translateX(-65px)  translateY(-30px) scale(1.8) rotate(-35deg);
           filter: drop-shadow(0 0 35px rgba(255,60,0,0.9)) brightness(1.8); }
    20%  { transform: translateX(-65px)  translateY(-35px) scale(2.0) rotate(-38deg);
           filter: drop-shadow(0 0 50px rgba(255,40,0,1))   brightness(2.2); }
    /* Blur streak — nearly invisible flash across */
    30%  { transform: translateX(60px)   translateY(-5px)  scale(0.7) rotate(15deg);
           filter: blur(4px) drop-shadow(0 0 60px rgba(255,20,0,1)) brightness(3);
           opacity: 0.5; }
    /* Crash into target */
    42%  { transform: translateX(110px)  translateY(12px)  scale(2.2) rotate(28deg);
           filter: drop-shadow(0 0 70px rgba(255,50,0,1))  brightness(2.5); opacity:1; }
    /* Bounce-back recoil */
    55%  { transform: translateX(80px)   translateY(0)     scale(1.5) rotate(12deg);
           filter: drop-shadow(0 0 30px rgba(255,80,30,0.7)) brightness(1.6); }
    70%  { transform: translateX(25px)   translateY(0)     scale(1.15) rotate(4deg);
           filter: drop-shadow(0 0 12px rgba(255,100,30,0.4)) brightness(1.2); }
    85%  { transform: translateX(5px)    translateY(0)     scale(1.03) rotate(1deg);
           filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)) brightness(1); }
    100% { transform: translateX(0)      translateY(0)     scale(1)    rotate(0deg);
           filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)) brightness(1); opacity:1; }
  }

  /* Screen shake on heavy hit landing */
  @keyframes screenShake {
    0%,100% { transform: translate(0,0); }
    10%  { transform: translate(-6px, 3px); }
    20%  { transform: translate(6px, -4px); }
    30%  { transform: translate(-5px, 5px); }
    40%  { transform: translate(7px, -3px); }
    50%  { transform: translate(-4px, 4px); }
    60%  { transform: translate(5px, -2px); }
    80%  { transform: translate(-2px, 2px); }
  }
  body.screen-shake { animation: screenShake 0.45s ease; }

  /* ── ONE STRIKE: spin + teleport ── */
  .combatant-card.attack-onestrike .combatant-sprite {
    animation: oneStrikeAnim 0.75s ease-in-out;
  }
  @keyframes oneStrikeAnim {
    0%   { transform: scale(1)    rotate(0deg)   translateX(0);    opacity: 1;   filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)); }
    20%  { transform: scale(1.5)  rotate(180deg) translateX(0);    opacity: 0.7; filter: drop-shadow(0 0 20px rgba(180,0,255,0.8)); }
    40%  { transform: scale(0.3)  rotate(540deg) translateX(120px);opacity: 0;   }
    41%  { transform: scale(0.3)  rotate(540deg) translateX(120px);opacity: 0;   }
    60%  { transform: scale(1.8)  rotate(720deg) translateX(80px); opacity: 1;   filter: drop-shadow(0 0 40px rgba(255,0,0,1)); }
    80%  { transform: scale(1.1)  rotate(730deg) translateX(5px);  }
    100% { transform: scale(1)    rotate(720deg) translateX(0);    opacity: 1;   filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)); }
  }

  /* ── MONSTER ATTACK: looming pounce ── */
  .enemy-card.attack-monster .combatant-sprite {
    animation: monsterPounce 0.7s cubic-bezier(0.22,1,0.36,1);
  }
  @keyframes monsterPounce {
    0%   { transform: translateX(0)     translateY(0)    scale(1)    rotate(0deg);  filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)); }
    10%  { transform: translateX(15px)  translateY(-30px) scale(1.4) rotate(5deg);  filter: drop-shadow(0 0 15px rgba(180,0,255,0.5)); }
    20%  { transform: translateX(15px)  translateY(-38px) scale(1.5) rotate(8deg);  filter: drop-shadow(0 0 25px rgba(180,0,255,0.8)); }
    50%  { transform: translateX(-90px) translateY(-10px) scale(1.7) rotate(-5deg); filter: drop-shadow(0 0 40px rgba(180,0,255,1)); }
    70%  { transform: translateX(-60px) translateY(0)    scale(1.2) rotate(-2deg); }
    85%  { transform: translateX(-8px)  translateY(0)    scale(1.02) rotate(0deg); }
    100% { transform: translateX(0)     translateY(0)    scale(1)    rotate(0deg);  filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)); }
  }

  /* ── MONSTER CRIT: grow + slam ── */
  .enemy-card.attack-monster-crit .combatant-sprite {
    animation: monsterCrit 0.75s cubic-bezier(0.22,1,0.36,1);
  }
  @keyframes monsterCrit {
    0%   { transform: scale(1)    rotate(0deg)  translateX(0);    filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)); }
    15%  { transform: scale(2.2)  rotate(-10deg) translateX(10px); filter: drop-shadow(0 0 30px rgba(255,0,0,0.8)) brightness(1.5); }
    25%  { transform: scale(2.4)  rotate(-12deg) translateX(10px); filter: drop-shadow(0 0 50px rgba(255,0,0,1)) brightness(2); }
    50%  { transform: scale(1.8)  rotate(5deg)  translateX(-110px);filter: drop-shadow(0 0 40px rgba(255,50,0,1)); }
    70%  { transform: scale(1.2)  rotate(1deg)  translateX(-20px); }
    85%  { transform: scale(1.02) rotate(0deg)  translateX(-3px); }
    100% { transform: scale(1)    rotate(0deg)  translateX(0);    filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)); }
  }

  /* ── BLOCK STANCE ── */
  .combatant-card.attack-block .combatant-sprite {
    animation: blockStance 0.5s ease;
  }
  @keyframes blockStance {
    0%   { transform: scale(1)    translateX(0)   rotate(0deg); filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)); }
    20%  { transform: scale(0.85) translateX(-15px) rotate(-5deg); filter: drop-shadow(0 0 20px rgba(52,152,219,0.9)); }
    50%  { transform: scale(1.1)  translateX(-10px) rotate(-3deg); filter: drop-shadow(0 0 30px rgba(52,152,219,1)); }
    75%  { transform: scale(1.05) translateX(-5px)  rotate(-1deg); }
    100% { transform: scale(1)    translateX(0)   rotate(0deg); filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6)); }
  }

  /* ── IMPACT SHOCKWAVE on enemy card ── */
  .combatant-card.impact::after {
    content: '';
    position: absolute;
    inset: 0;
    border-radius: 4px;
    animation: shockwave 0.5s ease-out forwards;
    pointer-events: none;
  }
  @keyframes shockwave {
    0%   { box-shadow: inset 0 0 0px 0px rgba(255,200,50,0); }
    20%  { box-shadow: inset 0 0 30px 8px rgba(255,200,50,0.7); }
    60%  { box-shadow: inset 0 0 15px 4px rgba(255,100,30,0.4); }
    100% { box-shadow: inset 0 0 0px 0px rgba(255,200,50,0); }
  }
  .combatant-card.impact-heavy::after {
    animation: shockwaveHeavy 0.6s ease-out forwards;
  }
  @keyframes shockwaveHeavy {
    0%   { box-shadow: inset 0 0 0px 0px rgba(255,50,0,0); background: transparent; }
    15%  { box-shadow: inset 0 0 50px 15px rgba(255,50,0,0.6); background: rgba(255,50,0,0.08); }
    40%  { box-shadow: inset 0 0 30px 8px rgba(255,100,30,0.4); }
    100% { box-shadow: inset 0 0 0px 0px rgba(255,50,0,0); background: transparent; }
  }
  .combatant-card.impact-os::after {
    animation: shockwaveOS 0.7s ease-out forwards;
  }
  @keyframes shockwaveOS {
    0%   { box-shadow: inset 0 0 0px 0px rgba(180,0,255,0); background: transparent; }
    10%  { box-shadow: inset 0 0 80px 20px rgba(180,0,255,0.7); background: rgba(180,0,255,0.12); }
    50%  { box-shadow: inset 0 0 40px 10px rgba(180,0,255,0.3); }
    100% { box-shadow: inset 0 0 0px 0px rgba(180,0,255,0); background: transparent; }
  }

  .combatant-name {
    font-family: 'Cinzel', serif;
    font-size: 1.05rem;
    font-weight: 600;
    letter-spacing: 0.1em;
    color: var(--white);
    margin-bottom: 0.2rem;
  }

  .combatant-title {
    font-size: 0.85rem;
    color: var(--text3);
    font-style: italic;
    margin-bottom: 0.8rem;
  }

  /* HP / Stamina bars */
  .stat-bar-row {
    margin-bottom: 0.5rem;
    text-align: left;
  }

  .stat-bar-label {
    display: flex;
    justify-content: space-between;
    font-size: 0.85rem;
    color: var(--text2);
    margin-bottom: 0.2rem;
    font-family: 'Cinzel', serif;
    letter-spacing: 0.05em;
  }

  .stat-bar-outer {
    height: 10px;
    background: rgba(255,255,255,0.05);
    border-radius: 4px;
    border: 1px solid var(--border);
    overflow: hidden;
  }

  .stat-bar-inner {
    height: 100%;
    border-radius: 4px;
    transition: width 0.4s ease;
  }

  .hp-bar { background: linear-gradient(90deg, #c0392b, #e74c3c); }
  .stamina-bar { background: linear-gradient(90deg, #27ae60, #2ecc71); }
  .enemy-hp-bar { background: linear-gradient(90deg, #8e44ad, #c0392b); }

  /* Floating damage numbers */
  .damage-float {
    position: absolute;
    top: 20%;
    left: 50%;
    transform: translateX(-50%);
    font-family: 'Cinzel Decorative', serif;
    font-size: 1.5rem;
    font-weight: 900;
    pointer-events: none;
    animation: floatUp 1.2s ease forwards;
    z-index: 100;
    text-shadow: 0 2px 4px black;
  }

  @keyframes floatUp {
    0% { opacity: 1; transform: translateX(-50%) translateY(0) scale(0.8); }
    30% { transform: translateX(-50%) translateY(-15px) scale(1.2); }
    100% { opacity: 0; transform: translateX(-50%) translateY(-50px) scale(0.9); }
  }

  .damage-normal { color: #e74c3c; }
  .damage-crit { color: #f39c12; }
  .damage-block { color: #3498db; }
  .damage-miss { color: #95a5a6; }
  .damage-enemy { color: #9b59b6; }
  .damage-heal { color: #2ecc71; }

  /* BATTLE LOG */
  .battle-log {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 4px;
    height: 200px;
    overflow-y: auto;
    padding: 1rem;
    font-size: 1.05rem;
    line-height: 1.7;
  }

  .battle-log::-webkit-scrollbar { width: 4px; }
  .battle-log::-webkit-scrollbar-track { background: var(--bg); }
  .battle-log::-webkit-scrollbar-thumb { background: var(--border2); border-radius: 2px; }

  .log-entry { padding: 0.1rem 0; border-bottom: 1px solid rgba(255,255,255,0.03); }
  .log-turn { color: var(--gold3); font-family: 'Cinzel', serif; font-size: 0.85rem; }
  .log-player { color: var(--text); }
  .log-enemy { color: #c084fc; }
  .log-crit { color: #f59e0b; font-weight: bold; }
  .log-block { color: #60a5fa; }
  .log-death { color: var(--red2); font-family: 'Cinzel', serif; letter-spacing: 0.05em; }
  .log-win { color: var(--green2); font-family: 'Cinzel', serif; }
  .log-system { color: var(--gold); font-style: italic; }

  /* ACTIONS */
  .action-area {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 1rem;
  }

  .action-title {
    font-family: 'Cinzel', serif;
    font-size: 0.85rem;
    letter-spacing: 0.2em;
    color: var(--text3);
    text-transform: uppercase;
    margin-bottom: 0.8rem;
    text-align: center;
  }

  .action-buttons {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 0.8rem;
  }

  .action-btn {
    padding: 0.8rem 0.5rem;
    border: 1px solid;
    border-radius: 4px;
    cursor: pointer;
    font-family: 'Cinzel', serif;
    font-size: 0.9rem;
    letter-spacing: 0.1em;
    font-weight: 600;
    text-align: center;
    transition: all 0.2s;
    position: relative;
    overflow: hidden;
  }

  .action-btn::before {
    content: '';
    position: absolute;
    inset: 0;
    opacity: 0;
    transition: opacity 0.2s;
  }

  .action-btn:hover:not(:disabled)::before { opacity: 1; }

  .action-btn:disabled {
    opacity: 0.3;
    cursor: not-allowed;
  }

  .btn-block {
    background: linear-gradient(135deg, #0d1f3c, #1a3a6b);
    border-color: var(--blue);
    color: var(--blue2);
  }
  .btn-block::before { background: radial-gradient(circle at center, rgba(52,152,219,0.15), transparent); }
  .btn-block:hover:not(:disabled) {
    background: linear-gradient(135deg, #1a3a6b, #2980b9);
    box-shadow: 0 0 15px rgba(52,152,219,0.3);
    transform: translateY(-2px);
  }

  .btn-light {
    background: linear-gradient(135deg, #1a120a, #3a2010);
    border-color: var(--orange);
    color: #f39c12;
  }
  .btn-light::before { background: radial-gradient(circle at center, rgba(230,126,34,0.15), transparent); }
  .btn-light:hover:not(:disabled) {
    background: linear-gradient(135deg, #3a2010, #e67e22);
    box-shadow: 0 0 15px rgba(230,126,34,0.3);
    transform: translateY(-2px);
  }

  .btn-heavy {
    background: linear-gradient(135deg, #200a0a, #5a1010);
    border-color: var(--red);
    color: var(--red2);
  }
  .btn-heavy::before { background: radial-gradient(circle at center, rgba(192,57,43,0.15), transparent); }
  .btn-heavy:hover:not(:disabled) {
    background: linear-gradient(135deg, #5a1010, #c0392b);
    box-shadow: 0 0 15px rgba(192,57,43,0.35);
    transform: translateY(-2px);
  }

  .action-btn .btn-icon { font-size: 1.5rem; display: block; margin-bottom: 0.3rem; }
  .action-btn .btn-name { display: block; }
  .action-btn .btn-cost {
    display: block;
    font-size: 0.8rem;
    color: inherit;
    opacity: 0.6;
    margin-top: 0.2rem;
    font-weight: 400;
  }

  /* Status area */
  .status-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.5rem 0;
    border-top: 1px solid var(--border);
    margin-top: 0.8rem;
  }

  .status-badge {
    font-family: 'Cinzel', serif;
    font-size: 0.85rem;
    letter-spacing: 0.1em;
    padding: 0.2rem 0.6rem;
    border-radius: 2px;
    border: 1px solid;
  }

  .badge-fight { background: rgba(192,57,43,0.15); border-color: var(--red3); color: var(--red2); }
  .badge-idle  { background: rgba(201,168,76,0.1);  border-color: var(--gold3); color: var(--gold); }
  .badge-won   { background: rgba(39,174,96,0.15); border-color: #1a5c30; color: var(--green2); }
  .badge-dead  { background: rgba(80,20,20,0.3); border-color: var(--red3); color: var(--red2); }

  .wave-info {
    font-family: 'Cinzel', serif;
    font-size: 0.85rem;
    color: var(--text3);
    letter-spacing: 0.1em;
  }

  .wave-info span { color: var(--gold); }

  /* FIGHT / NEXT buttons */
  .main-btn {
    width: 100%;
    padding: 0.8rem;
    font-family: 'Cinzel Decorative', serif;
    font-size: 1.15rem;
    letter-spacing: 0.15em;
    cursor: pointer;
    border-radius: 4px;
    transition: all 0.25s;
    border: 2px solid;
    margin-top: 0.8rem;
  }

  .btn-fight {
    background: linear-gradient(135deg, #3a0808, #7a1010);
    border-color: var(--red);
    color: var(--red2);
    text-shadow: 0 0 10px rgba(192,57,43,0.5);
  }

  .btn-fight:hover {
    background: linear-gradient(135deg, #7a1010, #c0392b);
    box-shadow: 0 0 25px rgba(192,57,43,0.4);
    transform: translateY(-2px);
  }

  .btn-next {
    background: linear-gradient(135deg, #1a1008, #3a2e10);
    border-color: var(--gold);
    color: var(--gold);
    text-shadow: 0 0 10px rgba(201,168,76,0.4);
  }

  .btn-next:hover {
    background: linear-gradient(135deg, #3a2e10, #6a4e10);
    box-shadow: 0 0 25px rgba(201,168,76,0.3);
    transform: translateY(-2px);
  }

  .btn-respawn {
    background: linear-gradient(135deg, #0a1a0a, #1a3a1a);
    border-color: var(--green);
    color: var(--green2);
  }

  .btn-respawn:hover {
    background: linear-gradient(135deg, #1a3a1a, #27ae60);
    box-shadow: 0 0 25px rgba(39,174,96,0.3);
    transform: translateY(-2px);
  }

  .hidden { display: none !important; }

  /* XP reward popup */
  .xp-popup {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%) scale(0.8);
    background: var(--bg2);
    border: 2px solid var(--gold);
    border-radius: 8px;
    padding: 2rem 3rem;
    text-align: center;
    z-index: 1000;
    animation: popupIn 0.4s ease forwards;
    box-shadow: 0 0 60px rgba(201,168,76,0.3), 0 20px 60px black;
  }

  @keyframes popupIn {
    to { transform: translate(-50%, -50%) scale(1); }
  }

  .xp-popup h2 {
    font-size: 1.5rem;
    color: var(--gold);
    margin-bottom: 0.5rem;
  }

  .xp-popup p {
    color: var(--text);
    margin: 0.3rem 0;
    font-style: italic;
  }

  .xp-breakdown {
    margin: 1rem 0;
    font-family: 'Cinzel', serif;
    font-size: 1.05rem;
  }

  .xp-item {
    display: flex;
    justify-content: space-between;
    padding: 0.3rem 0;
    border-bottom: 1px solid var(--border);
    color: var(--text2);
  }

  .xp-item span:last-child { color: var(--green2); }

  .overlay {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.7);
    z-index: 999;
  }

  /* Enemy tier badges */
  .enemy-tier {
    font-family: 'Cinzel', serif;
    font-size: 0.8rem;
    letter-spacing: 0.15em;
    padding: 0.15rem 0.5rem;
    border-radius: 2px;
    display: inline-block;
    margin-bottom: 0.5rem;
  }

  .tier-common   { background: rgba(100,100,100,0.2); border: 1px solid #666; color: #aaa; }
  .tier-uncommon { background: rgba(39,174,96,0.15);  border: 1px solid #27ae60; color: #2ecc71; }
  .tier-rare     { background: rgba(41,128,185,0.15); border: 1px solid #2980b9; color: #3498db; }
  .tier-epic     { background: rgba(142,68,173,0.15); border: 1px solid #8e44ad; color: #9b59b6; }
  .tier-legendary{ background: rgba(201,168,76,0.15); border: 1px solid var(--gold); color: var(--gold2); }
  .tier-mythic   { background: rgba(192,57,43,0.2);   border: 1px solid #e74c3c; color: #ff6b6b; }
  .tier-titan    {
    background: linear-gradient(135deg, rgba(255,215,0,0.15), rgba(255,100,255,0.1));
    border: 1px solid #ffd700;
    color: #ffd700;
    text-shadow: 0 0 8px rgba(255,215,0,0.6);
    animation: titanBadgePulse 2s ease-in-out infinite;
    font-weight: bold;
    letter-spacing: 0.2em;
  }
  @keyframes titanBadgePulse {
    0%, 100% { box-shadow: 0 0 4px rgba(255,215,0,0.3); }
    50%       { box-shadow: 0 0 12px rgba(255,215,0,0.7), 0 0 24px rgba(255,100,255,0.3); }
  }

  /* ── MOBILE POLISH ── */
  @media (max-width: 480px) {
    .game-header h1 { font-size: 1.4rem; }
    .panel { padding: 0.8rem; }

    /* Smaller combatant cards on mobile */
    .arena-panel { padding: 0.6rem !important; }
    .combatants { gap: 0.4rem; }
    .combatant-card { padding: 0.5rem 0.4rem; }
    .combatant-sprite { font-size: 1.8rem; margin-bottom: 0.2rem; }
    .combatant-name { font-size: 0.7rem; }
    .combatant-title { font-size: 0.6rem; margin-bottom: 0.4rem; }
    .stat-bar-row { margin-bottom: 0.3rem; }
    .stat-bar-label { font-size: 0.6rem; margin-bottom: 0.15rem; }
    .stat-bar-outer { height: 6px; }
    .enemy-tier { font-size: 0.55rem; padding: 0.1rem 0.3rem; margin-bottom: 0.3rem; }
    .vs-divider { font-size: 1rem; padding-top: 1rem; }
    .turn-indicator { font-size: 0.7rem; padding: 0.2rem; }

    /* Actions stay readable */
    .action-buttons { grid-template-columns: 1fr 1fr; }
    .action-buttons .btn-heavy { grid-column: 1 / -1; }

    /* Battle log compact */
    .battle-log { height: 110px; font-size: 0.78rem; }

    .skills-grid { grid-template-columns: 1fr; }
    .skin-grid { grid-template-columns: repeat(2, 1fr); }
    .xp-popup { width: 92vw; padding: 1rem; }
    .main-btn { font-size: 0.85rem; }
  }


  /* Particle effects */
  .particle {
    position: absolute;
    pointer-events: none;
    border-radius: 50%;
    animation: particleFly 1s ease forwards;
  }

  @keyframes particleFly {
    0% { opacity: 1; transform: translate(0, 0) scale(1); }
    100% { opacity: 0; transform: var(--particle-end, translate(20px, -40px)) scale(0); }
  }

  .turn-indicator {
    text-align: center;
    font-family: 'Cinzel', serif;
    font-size: 0.9rem;
    letter-spacing: 0.2em;
    color: var(--text3);
    padding: 0.3rem;
  }

  .turn-indicator.player-turn { color: var(--gold); }
  .turn-indicator.enemy-turn  { color: #c084fc; }

  #skill-clicks-info {
    font-size: 0.85rem;
    color: var(--text3);
    text-align: center;
    margin-top: 0.5rem;
    font-family: 'Cinzel', serif;
    letter-spacing: 0.05em;
  }

  /* ── SHOP ── */
  .shop-tabs { display:flex; gap:0.3rem; margin-bottom:1rem; }
  .shop-tab {
    flex:1; padding:0.35rem 0.2rem; font-family:'Cinzel',serif; font-size: 0.8rem;
    letter-spacing:0.1em; background:var(--bg); border:1px solid var(--border);
    color:var(--text3); cursor:pointer; border-radius:2px; transition:all 0.2s; text-align:center;
  }
  .shop-tab.active { background:rgba(201,168,76,0.1); border-color:var(--gold3); color:var(--gold); }
  .shop-tab:hover:not(.active) { border-color:var(--border2); color:var(--text2); }
  .shop-section { display:none; }
  .shop-section.active { display:block; }
  .shop-item {
    background:var(--bg); border:1px solid var(--border); border-radius:3px;
    padding:0.7rem; margin-bottom:0.6rem; transition:border-color 0.2s;
  }
  .shop-item:hover { border-color:var(--border2); }
  .shop-item-header { display:flex; justify-content:space-between; align-items:flex-start; margin-bottom:0.3rem; }
  .shop-item-name { font-family:'Cinzel',serif; font-size: 0.95rem; font-weight:600; color:var(--white); display:flex; align-items:center; gap:0.3rem; }
  .shop-item-desc { font-size: 0.9rem; color:var(--text3); font-style:italic; margin-bottom:0.5rem; line-height:1.4; }
  .shop-price { font-family:'Cinzel',serif; font-size: 0.85rem; color:var(--gold); background:rgba(201,168,76,0.1); border:1px solid var(--gold3); padding:0.15rem 0.4rem; border-radius:2px; white-space:nowrap; }
  .shop-price.unaffordable { color:var(--text3); background:rgba(0,0,0,0.2); border-color:var(--border); }
  .btn-buy { width:100%; padding:0.35rem; font-family:'Cinzel',serif; font-size: 0.85rem; letter-spacing:0.1em; cursor:pointer; border-radius:2px; transition:all 0.2s; border:1px solid; }
  .btn-buy-gold { background:linear-gradient(135deg,#1e1608,#3a2e10); border-color:var(--gold3); color:var(--gold); }
  .btn-buy-gold:hover:not(:disabled) { background:linear-gradient(135deg,#3a2e10,#6a4e10); border-color:var(--gold); box-shadow:0 0 10px rgba(201,168,76,0.2); }
  .btn-buy-red { background:linear-gradient(135deg,#200808,#5a1010); border-color:var(--red3); color:var(--red2); }
  .btn-buy-red:hover:not(:disabled) { background:linear-gradient(135deg,#5a1010,#a01818); border-color:var(--red); box-shadow:0 0 10px rgba(192,57,43,0.3); }
  .btn-buy-blue { background:linear-gradient(135deg,#080e20,#102040); border-color:#2980b9; color:#60a5fa; }
  .btn-buy-blue:hover:not(:disabled) { background:linear-gradient(135deg,#102040,#1a3a6b); border-color:#3498db; box-shadow:0 0 10px rgba(52,152,219,0.3); }
  .btn-buy:disabled { opacity:0.35; cursor:not-allowed; }
  .owned-badge { font-family:'Cinzel',serif; font-size: 0.8rem; color:var(--green2); background:rgba(39,174,96,0.1); border:1px solid #1a5c30; padding:0.15rem 0.4rem; border-radius:2px; }
  .skin-grid { display:grid; grid-template-columns:repeat(2,1fr); gap:0.5rem; margin-bottom:0.5rem; }
  .skin-card { background:var(--bg); border:1px solid var(--border); border-radius:3px; padding:0.6rem; text-align:center; cursor:pointer; transition:all 0.2s; position:relative; }
  .skin-card:hover { border-color:var(--border2); }
  .skin-card.equipped { border-color:var(--gold); background:rgba(201,168,76,0.06); }
  .skin-card.owned-skin { border-color:#1a5c30; }
  .skin-card.skin-locked { opacity:0.6; cursor: not-allowed; border-color: var(--border); }
  .skin-card.skin-locked:hover { border-color: var(--border); transform: none; }
  .skin-emoji { font-size:2rem; display:block; margin-bottom:0.3rem; }
  .skin-name { font-family:'Cinzel',serif; font-size: 0.8rem; color:var(--text2); display:block; margin-bottom:0.2rem; }
  .skin-price-tag { font-size: 0.8rem; color:var(--gold3); font-family:'Cinzel',serif; }
  .name-input-row { display:flex; gap:0.5rem; margin-top:0.5rem; }
  .name-input { flex:1; background:var(--bg); border:1px solid var(--border2); color:var(--white); font-family:'Cinzel',serif; font-size: 0.95rem; padding:0.4rem 0.6rem; border-radius:2px; outline:none; }
  .name-input:focus { border-color:var(--gold3); }
  .one-strike-banner { background:linear-gradient(135deg,rgba(192,57,43,0.15),rgba(120,20,10,0.2)); border:1px solid var(--red3); border-radius:3px; padding:0.8rem; text-align:center; margin-bottom:0.6rem; }
  .one-strike-banner .big-icon { font-size:3.2rem; display:block; margin-bottom:0.4rem; }
  .one-strike-banner h3 { font-size: 1.45rem; color:var(--red2); letter-spacing:0.15em; margin-bottom:0.4rem; text-shadow:0 0 15px rgba(192,57,43,0.5); }
  .one-strike-stats { font-size: 1.05rem; color:var(--text3); font-style:italic; line-height:1.8; margin-bottom:0.6rem; }
  .btn-one-strike {
    background:linear-gradient(135deg,#3a0505,#7a1010); border:2px solid #c0392b; border-radius:4px;
    color:#ff6b6b; font-family:'Cinzel Decorative',serif; font-size: 0.9rem; letter-spacing:0.1em;
    padding:0.7rem; cursor:pointer; width:100%; margin-top:0.6rem; text-align:center;
    transition:all 0.2s; position:relative; overflow:hidden;
  }
  .btn-one-strike:hover:not(:disabled) { box-shadow:0 0 20px rgba(192,57,43,0.5); transform:translateY(-1px); }
  .btn-one-strike:disabled, .btn-one-strike.used { opacity:0.3; cursor:not-allowed; border-color:var(--border); color:var(--text3); }
  .os-flash { position:fixed; inset:0; background:radial-gradient(circle at center,rgba(255,50,20,0.55),transparent 70%); pointer-events:none; z-index:9998; animation:osFlash 0.8s ease forwards; }
  @keyframes osFlash { 0%{opacity:0} 20%{opacity:1} 100%{opacity:0} }
  .xp-level-row { display:flex; justify-content:space-between; align-items:center; padding:0.4rem 0; border-bottom:1px solid var(--border); font-size: 0.95rem; }
  .xp-level-row:last-child { border-bottom:none; }
  .xp-level-name { display:flex; align-items:center; gap:0.4rem; color:var(--text); }
  .xp-level-cost { color:var(--gold3); font-family:'Cinzel',serif; font-size: 0.85rem; }

  /* ── HOW TO PLAY ── */
  .how-to-play {
    max-width: 860px;
    margin: 0 auto 0;
    padding: 0 1rem 1rem;
  }

  .htp-toggle {
    width: 100%;
    background: linear-gradient(135deg, #1a1208, #2a200e);
    border: 1px solid var(--gold3);
    color: var(--gold);
    font-family: 'Cinzel', serif;
    font-size: 1rem;
    letter-spacing: 0.2em;
    padding: 0.75rem 1rem;
    cursor: pointer;
    border-radius: 4px;
    text-align: left;
    display: flex;
    justify-content: space-between;
    align-items: center;
    transition: all 0.2s;
  }
  .htp-toggle:hover { background: linear-gradient(135deg, #2a200e, #3a2e14); border-color: var(--gold); }
  .htp-toggle .arrow { transition: transform 0.3s; font-size: 0.8rem; }
  .htp-toggle.open .arrow { transform: rotate(180deg); }

  .htp-body {
    display: none;
    background: var(--panel);
    border: 1px solid var(--border);
    border-top: none;
    border-radius: 0 0 4px 4px;
    padding: 1.4rem;
  }
  .htp-body.open { display: block; }

  .htp-sections {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 1.2rem;
  }

  .htp-card {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 1rem;
  }

  .htp-card-title {
    font-family: 'Cinzel', serif;
    font-size: 0.95rem;
    font-weight: 600;
    letter-spacing: 0.1em;
    margin-bottom: 0.8rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .htp-card ul {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .htp-card ul li {
    font-size: 0.9rem;
    color: var(--text2);
    line-height: 1.7;
    padding: 0.2rem 0;
    border-bottom: 1px solid rgba(255,255,255,0.03);
    display: flex;
    gap: 0.5rem;
    align-items: flex-start;
  }

  .htp-card ul li:last-child { border-bottom: none; }

  .htp-card ul li .li-icon {
    flex-shrink: 0;
    width: 1.4rem;
    text-align: center;
    margin-top: 0.05rem;
  }

  .htp-tip {
    margin-top: 1.2rem;
    background: linear-gradient(135deg, rgba(201,168,76,0.06), rgba(201,168,76,0.02));
    border: 1px solid var(--gold3);
    border-radius: 3px;
    padding: 0.8rem 1rem;
    font-size: 0.9rem;
    color: var(--text2);
    font-style: italic;
    line-height: 1.6;
    text-align: center;
  }

  .htp-tip strong { color: var(--gold); font-style: normal; }

  /* Save bar */
  .save-bar {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 0.6rem;
    padding: 0.5rem 1rem;
    background: rgba(0,0,0,0.3);
    border-bottom: 1px solid var(--border);
  }
  .btn-save {
    background: linear-gradient(135deg, #0e1a0e, #1a3a1a);
    border: 1px solid var(--green);
    color: var(--green2);
    font-family: 'Cinzel', serif;
    font-size: 0.8rem;
    letter-spacing: 0.1em;
    padding: 0.35rem 1rem;
    border-radius: 3px;
    cursor: pointer;
    transition: all 0.2s;
  }
  .btn-save:hover { background: linear-gradient(135deg, #1a3a1a, #27ae60); box-shadow: 0 0 10px rgba(39,174,96,0.3); }
  .btn-delete-save {
    background: transparent;
    border: 1px solid var(--border);
    color: var(--text3);
    font-family: 'Cinzel', serif;
    font-size: 0.75rem;
    letter-spacing: 0.08em;
    padding: 0.35rem 0.8rem;
    border-radius: 3px;
    cursor: pointer;
    transition: all 0.2s;
  }
  .btn-delete-save:hover { border-color: var(--red3); color: var(--red2); }
  .save-info { font-size: 0.75rem; color: var(--text3); font-family: 'Cinzel', serif; letter-spacing: 0.05em; font-style: italic; }

  /* ── SAVE BAR ── */
  .save-bar {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 0.8rem;
    padding: 0.5rem 1rem;
    background: rgba(0,0,0,0.35);
    border-bottom: 1px solid var(--border);
    flex-wrap: wrap;
  }
  .btn-save {
    background: linear-gradient(135deg, #0e1a0e, #1a3a1a);
    border: 1px solid var(--green);
    color: var(--green2);
    font-family: 'Cinzel', serif;
    font-size: 0.85rem;
    letter-spacing: 0.1em;
    padding: 0.4rem 1.1rem;
    border-radius: 3px;
    cursor: pointer;
    transition: all 0.2s;
  }
  .btn-save:hover { background: linear-gradient(135deg, #1a3a1a, #27ae60); box-shadow: 0 0 12px rgba(39,174,96,0.3); }
  .btn-delete-save {
    background: transparent;
    border: 1px solid var(--border);
    color: var(--text3);
    font-family: 'Cinzel', serif;
    font-size: 0.8rem;
    letter-spacing: 0.08em;
    padding: 0.4rem 0.9rem;
    border-radius: 3px;
    cursor: pointer;
    transition: all 0.2s;
  }
  .btn-delete-save:hover { border-color: var(--red3); color: var(--red2); }
  .save-info { font-size: 0.8rem; color: var(--text3); font-family: 'Cinzel', serif; letter-spacing: 0.05em; font-style: italic; }




  /* ══ LEADERBOARD ══ */
  .lb-popup {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%) scale(0.92);
    background: #0d0b08;
    border: 1px solid rgba(201,168,76,0.35);
    border-radius: 12px;
    padding: 1.6rem 1.4rem;
    z-index: 1001;
    width: 92vw;
    max-width: 640px;
    max-height: 88vh;
    overflow-y: auto;
    box-shadow: 0 0 80px rgba(201,168,76,0.12), 0 24px 80px rgba(0,0,0,0.9);
    animation: popupIn 0.3s ease forwards;
  }

  .lb-popup h2 {
    font-size: 1.1rem;
    color: var(--gold);
    text-align: center;
    letter-spacing: 0.2em;
    margin-bottom: 0.25rem;
    text-shadow: 0 0 24px rgba(201,168,76,0.5);
  }

  .lb-subtitle {
    text-align: center;
    font-size: 0.72rem;
    color: var(--text3);
    font-style: italic;
    margin-bottom: 1.2rem;
    letter-spacing: 0.06em;
  }

  .lb-tabs {
    display: flex;
    gap: 0.4rem;
    margin-bottom: 1.2rem;
    background: #070604;
    border-radius: 6px;
    padding: 4px;
  }

  .lb-tab {
    flex: 1;
    padding: 0.45rem 0.3rem;
    font-family: 'Cinzel', serif;
    font-size: 0.68rem;
    letter-spacing: 0.08em;
    background: transparent;
    border: none;
    color: #6a5040;
    cursor: pointer;
    border-radius: 4px;
    transition: all 0.2s;
    text-align: center;
  }
  .lb-tab.active { background: rgba(201,168,76,0.15); color: var(--gold); box-shadow: inset 0 0 0 1px rgba(201,168,76,0.3); }
  .lb-tab:hover:not(.active) { color: var(--text2); background: rgba(255,255,255,0.04); }

  .lb-table {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0 3px;
    font-size: 0.85rem;
  }

  .lb-table thead tr {
    background: transparent;
  }

  .lb-table th {
    font-family: 'Cinzel', serif;
    font-size: 0.62rem;
    letter-spacing: 0.14em;
    color: rgba(201,168,76,0.5);
    text-align: left;
    padding: 0.3rem 0.8rem;
    border-bottom: 1px solid rgba(201,168,76,0.12);
    text-transform: uppercase;
  }

  .lb-table tbody tr {
    background: #141008;
    transition: background 0.15s;
  }
  .lb-table tbody tr:nth-child(even) { background: #100e06; }
  .lb-table tbody tr:hover { background: #1e180a; }

  .lb-table td {
    padding: 0.65rem 0.8rem;
    color: #c0a880;
    vertical-align: middle;
    border-top: 1px solid rgba(255,255,255,0.03);
    border-bottom: 1px solid rgba(0,0,0,0.3);
  }
  .lb-table tbody tr td:first-child { border-radius: 6px 0 0 6px; }
  .lb-table tbody tr td:last-child  { border-radius: 0 6px 6px 0; }

  .lb-rank {
    font-family: 'Cinzel', serif;
    font-size: 0.82rem;
    color: #6a5040;
    width: 2.4rem;
    text-align: center;
  }
  .lb-rank-1 { color: #f59e0b; font-size: 1.05rem; text-shadow: 0 0 10px rgba(245,158,11,0.5); }
  .lb-rank-2 { color: #cbd5e1; font-size: 1rem; }
  .lb-rank-3 { color: #cd7f32; font-size: 0.95rem; }

  .lb-name {
    font-family: 'Cinzel', serif;
    font-size: 0.84rem;
    color: #e8d5b0;
    letter-spacing: 0.04em;
  }
  .lb-name.is-you { color: var(--gold); text-shadow: 0 0 8px rgba(201,168,76,0.4); }
  .lb-name.is-you::after { content: ' ✦ You'; font-size: 0.6rem; color: rgba(201,168,76,0.6); letter-spacing: 0.1em; }

  .lb-score {
    font-family: 'Cinzel', serif;
    font-size: 0.88rem;
    color: #4ade80;
    text-align: right;
    font-weight: 600;
    letter-spacing: 0.04em;
  }

  .lb-skin { font-size: 1.15rem; line-height: 1; }

  .lb-empty {
    text-align: center;
    color: var(--text3);
    font-style: italic;
    padding: 2rem;
    font-size: 0.82rem;
    letter-spacing: 0.05em;
  }

  .lb-you-row td {
    background: rgba(201,168,76,0.06) !important;
    border-top: 1px solid rgba(201,168,76,0.12) !important;
    border-bottom: 1px solid rgba(201,168,76,0.12) !important;
  }

  .lb-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 1.2rem;
    padding-top: 0.8rem;
    border-top: 1px solid rgba(201,168,76,0.1);
    flex-wrap: wrap;
    gap: 0.5rem;
  }

  .lb-refresh-info {
    font-size: 0.68rem;
    color: #6a5040;
    font-style: italic;
    letter-spacing: 0.04em;
  }

  .btn-lb-close {
    background: rgba(201,168,76,0.08);
    border: 1px solid rgba(201,168,76,0.3);
    color: var(--gold);
    font-family: 'Cinzel', serif;
    font-size: 0.72rem;
    letter-spacing: 0.12em;
    padding: 0.45rem 1.2rem;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.2s;
  }
  .btn-lb-close:hover { background: rgba(201,168,76,0.16); border-color: var(--gold); box-shadow: 0 0 12px rgba(201,168,76,0.2); }

  .btn-leaderboard {
    background: rgba(201,168,76,0.08);
    border: 1px solid rgba(201,168,76,0.3);
    color: var(--gold);
    font-family: 'Cinzel', serif;
    font-size: 0.82rem;
    letter-spacing: 0.1em;
    padding: 0.4rem 1rem;
    border-radius: 3px;
    cursor: pointer;
    transition: all 0.2s;
  }
  .btn-leaderboard:hover { background: rgba(201,168,76,0.16); border-color: var(--gold); box-shadow: 0 0 14px rgba(201,168,76,0.25); }

  .lb-loading {
    text-align: center;
    color: #6a5040;
    font-style: italic;
    padding: 2rem;
    font-size: 0.82rem;
    letter-spacing: 0.06em;
    animation: pulse 1.5s ease-in-out infinite;
  }


  /* ══ GUILD SYSTEM ══ */
  .guild-section { display: flex; flex-direction: column; gap: 0.8rem; }

  .guild-locked-msg {
    text-align: center;
    background: rgba(0,0,0,0.3);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 1.2rem;
  }
  .guild-locked-msg .lock-icon { font-size: 2rem; display: block; margin-bottom: 0.4rem; }
  .guild-locked-msg p { font-size: 0.85rem; color: var(--text3); font-style: italic; line-height: 1.6; }
  .guild-locked-msg strong { color: var(--gold); }

  .guild-progress-bar-bg {
    height: 8px;
    background: rgba(255,255,255,0.05);
    border: 1px solid var(--border);
    border-radius: 4px;
    overflow: hidden;
    margin: 0.6rem 0 0.3rem;
  }
  .guild-progress-bar-fill {
    height: 100%;
    background: linear-gradient(90deg, var(--gold3), var(--gold));
    border-radius: 4px;
    transition: width 0.4s ease;
  }
  .guild-progress-label {
    font-family: 'Cinzel', serif;
    font-size: 0.72rem;
    color: var(--text3);
    text-align: center;
    letter-spacing: 0.08em;
  }

  .guild-panel {
    background: var(--bg);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 0.9rem;
  }

  .guild-panel-title {
    font-family: 'Cinzel', serif;
    font-size: 0.78rem;
    letter-spacing: 0.15em;
    color: var(--gold3);
    text-transform: uppercase;
    margin-bottom: 0.7rem;
    padding-bottom: 0.4rem;
    border-bottom: 1px solid var(--border);
  }

  .guild-name-display {
    font-family: 'Cinzel Decorative', serif;
    font-size: 1.1rem;
    color: var(--gold);
    text-align: center;
    margin-bottom: 0.2rem;
    text-shadow: 0 0 15px rgba(201,168,76,0.35);
  }

  .guild-code-display {
    text-align: center;
    font-family: 'Cinzel', serif;
    font-size: 0.78rem;
    color: var(--text3);
    margin-bottom: 0.7rem;
  }

  .guild-code-badge {
    display: inline-block;
    background: rgba(201,168,76,0.1);
    border: 1px solid var(--gold3);
    border-radius: 3px;
    padding: 0.15rem 0.6rem;
    color: var(--gold);
    font-family: monospace;
    font-size: 1rem;
    letter-spacing: 0.15em;
    cursor: pointer;
    transition: all 0.2s;
  }
  .guild-code-badge:hover { background: rgba(201,168,76,0.2); }

  .guild-role-badge {
    display: inline-block;
    font-family: 'Cinzel', serif;
    font-size: 0.68rem;
    letter-spacing: 0.1em;
    padding: 0.15rem 0.5rem;
    border-radius: 2px;
    margin-bottom: 0.6rem;
  }
  .role-leader  { background: rgba(201,168,76,0.15); border: 1px solid var(--gold3); color: var(--gold); }
  .role-member  { background: rgba(41,128,185,0.12);  border: 1px solid #2980b9; color: #60a5fa; }

  .guild-members-list { margin-top: 0.5rem; }

  .guild-member-row {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.4rem 0;
    border-bottom: 1px solid rgba(255,255,255,0.04);
    font-size: 0.82rem;
  }
  .guild-member-row:last-child { border-bottom: none; }
  .guild-member-skin { font-size: 1rem; }
  .guild-member-name { flex: 1; color: var(--text); font-family: 'Cinzel', serif; font-size: 0.78rem; }
  .guild-member-name.is-you { color: var(--gold); }
  .guild-member-wave { color: var(--green2); font-family: 'Cinzel', serif; font-size: 0.75rem; }
  .guild-member-leader-crown { font-size: 0.8rem; }

  .guild-avg-wave {
    text-align: center;
    font-family: 'Cinzel', serif;
    font-size: 0.8rem;
    color: var(--text3);
    padding: 0.6rem;
    border-top: 1px solid var(--border);
    margin-top: 0.5rem;
  }
  .guild-avg-wave span { color: var(--gold); font-size: 1rem; }

  .guild-input-row { display: flex; gap: 0.5rem; margin-top: 0.4rem; }
  .guild-input {
    flex: 1;
    background: var(--bg);
    border: 1px solid var(--border2);
    color: var(--white);
    font-family: 'Cinzel', serif;
    font-size: 0.82rem;
    padding: 0.4rem 0.6rem;
    border-radius: 2px;
    outline: none;
    text-transform: uppercase;
    letter-spacing: 0.1em;
  }
  .guild-input:focus { border-color: var(--gold3); }
  .guild-input::placeholder { text-transform: none; letter-spacing: normal; color: var(--text3); }

  .btn-guild {
    padding: 0.4rem 0.9rem;
    font-family: 'Cinzel', serif;
    font-size: 0.72rem;
    letter-spacing: 0.1em;
    border-radius: 2px;
    cursor: pointer;
    transition: all 0.2s;
    border: 1px solid;
    white-space: nowrap;
  }
  .btn-guild-gold { background: linear-gradient(135deg,#1e1608,#3a2e10); border-color: var(--gold3); color: var(--gold); }
  .btn-guild-gold:hover { background: linear-gradient(135deg,#3a2e10,#6a4e10); border-color: var(--gold); }
  .btn-guild-red  { background: linear-gradient(135deg,#200808,#5a1010); border-color: var(--red3); color: var(--red2); }
  .btn-guild-red:hover  { background: linear-gradient(135deg,#5a1010,#a01818); border-color: var(--red); }
  .btn-guild-blue { background: linear-gradient(135deg,#0a1020,#102040); border-color: #2980b9; color: #60a5fa; }
  .btn-guild-blue:hover { background: linear-gradient(135deg,#102040,#1a4080); border-color: #3498db; }
  .btn-guild:disabled { opacity: 0.35; cursor: not-allowed; }

  .guild-error { font-size: 0.78rem; color: var(--red2); font-style: italic; margin-top: 0.3rem; min-height: 1rem; }
  .guild-success { font-size: 0.78rem; color: var(--green2); font-style: italic; margin-top: 0.3rem; }

  /* Leaderboard guild tab */
  .lb-guild-row {
    display: flex;
    align-items: center;
    gap: 0.6rem;
    padding: 0.55rem 0.6rem;
    border-bottom: 1px solid rgba(255,255,255,0.04);
  }
  .lb-guild-row:last-child { border-bottom: none; }
  .lb-guild-name { flex: 1; font-family: 'Cinzel', serif; font-size: 0.82rem; color: var(--white); }
  .lb-guild-name.is-my-guild { color: var(--gold); }
  .lb-guild-name.is-my-guild::after { content: ' (Yours)'; font-size: 0.65rem; color: var(--gold3); }
  .lb-guild-members { font-size: 0.75rem; color: var(--text3); }
  .lb-guild-avg { font-family: 'Cinzel', serif; font-size: 0.85rem; color: var(--green2); text-align: right; }


  /* ── FIREBASE SETUP BANNER ── */
  .firebase-banner {
    background: linear-gradient(135deg, rgba(255,160,0,0.1), rgba(255,100,0,0.08));
    border: 1px solid #f59e0b;
    border-radius: 4px;
    padding: 0.8rem 1.2rem;
    margin: 0 auto;
    max-width: 860px;
    display: flex;
    align-items: center;
    gap: 0.8rem;
    flex-wrap: wrap;
    font-size: 0.82rem;
    color: var(--text2);
  }
  .firebase-banner.connected {
    background: linear-gradient(135deg, rgba(39,174,96,0.08), rgba(39,174,96,0.04));
    border-color: #27ae60;
    color: var(--green2);
  }
  .firebase-banner .fb-icon { font-size: 1.2rem; flex-shrink: 0; }
  .firebase-banner strong { color: #f59e0b; }
  .firebase-banner.connected strong { color: var(--green2); }
  .firebase-banner a { color: #f59e0b; }
  .firebase-banner.connected a { display: none; }


  /* ── POTIONS ── */
  .potion-card {
    background: var(--bg);
    border: 1px solid var(--border);
    border-radius: 3px;
    padding: 0.8rem;
    margin-bottom: 0.6rem;
    transition: border-color 0.2s;
  }
  .potion-card:hover { border-color: var(--border2); }
  .potion-header { display:flex; justify-content:space-between; align-items:center; margin-bottom:0.3rem; }
  .potion-name { font-family:'Cinzel',serif; font-size:1.05rem; font-weight:600; color:var(--white); display:flex; align-items:center; gap:0.4rem; }
  .potion-desc { font-size:1.0rem; color:var(--text3); font-style:italic; margin-bottom:0.6rem; line-height:1.7; }
  .potion-active-badge {
    display:inline-block; font-family:'Cinzel',serif; font-size:0.65rem;
    padding:0.15rem 0.5rem; border-radius:2px; margin-bottom:0.4rem;
    background:rgba(39,174,96,0.12); border:1px solid #1a5c30; color:var(--green2);
  }


  /* ── ADMIN SNAPSHOT ── */
  .btn-admin-snapshot {
    background: linear-gradient(135deg, #0a0a1a, #1a1a3a);
    border: 1px solid #4a4a8a;
    color: #8a8aff;
    font-family: 'Cinzel', serif;
    font-size: 0.82rem;
    letter-spacing: 0.1em;
    padding: 0.4rem 1rem;
    border-radius: 3px;
    cursor: pointer;
    transition: all 0.2s;
    display: none;
  }
  .btn-admin-snapshot:hover { background: linear-gradient(135deg,#1a1a3a,#2a2a6a); border-color:#8a8aff; box-shadow:0 0 10px rgba(138,138,255,0.25); }
  .btn-admin-snapshot.visible { display: inline-block; }


  /* ── WALLET ── */
  .wallet-linked-banner {
    background: rgba(39,174,96,0.08);
    border: 1px solid #1a5c30;
    border-radius: 3px;
    padding: 0.5rem 0.9rem;
    font-size: 0.82rem;
    color: var(--green2);
    font-family: 'Cinzel', serif;
    margin-bottom: 0.6rem;
    word-break: break-all;
  }
  .wallet-linked-banner.hidden { display: none; }

  /* ═══════════════════════════════════════
     BOSS FIGHT SYSTEM STYLES
  ═══════════════════════════════════════ */

  /* Boss alert banner */
  #boss-alert-banner {
    display: none;
    background: linear-gradient(135deg, #1a0000 0%, #2d0000 50%, #1a0000 100%);
    border-bottom: 2px solid var(--red2);
    padding: 9px 16px;
    text-align: center;
    position: sticky;
    top: 48px;
    z-index: 8999;
    animation: bossAlertPulse 2s ease-in-out infinite;
  }
  #boss-alert-banner.active { display: block; }
  @keyframes bossAlertPulse {
    0%,100% { border-bottom-color:var(--red); box-shadow:0 2px 16px rgba(192,57,43,0.3); }
    50%     { border-bottom-color:var(--gold); box-shadow:0 2px 24px rgba(192,57,43,0.6); }
  }
  .boss-alert-inner {
    display: flex; align-items: center; justify-content: center;
    gap: 10px; flex-wrap: wrap;
  }
  .boss-alert-skull { font-size:1.2rem; animation: bossSkullSpin 3s linear infinite; }
  @keyframes bossSkullSpin { to { transform: rotate(360deg); } }
  .boss-alert-text { font-family:'Cinzel Decorative',serif; font-size:0.72rem; color:var(--red2); letter-spacing:2px; }
  .boss-alert-name { font-family:'Cinzel Decorative',serif; font-size:0.72rem; color:var(--gold); font-weight:900; }

  /* Arena locked overlay */
  #arena-locked-overlay {
    display: none;
    position: absolute;
    inset: 0;
    z-index: 500;
    background: rgba(10,8,6,0.92);
    backdrop-filter: blur(3px);
    border-radius: 4px;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    gap: 14px;
    text-align: center;
    padding: 20px;
  }
  #arena-locked-overlay.active { display: flex; }
  .arena-locked-icon { font-size: 3rem; }
  .arena-locked-title { font-family:'Cinzel Decorative',serif; font-size:1.1rem; color:var(--red2); letter-spacing:3px; }
  .arena-locked-sub { font-size:0.78rem; color:var(--text3); font-family:'IM Fell English',serif; font-style:italic; line-height:1.7; max-width:240px; }
  .arena-locked-btn {
    background: linear-gradient(135deg, #1a0000, #2d0a0a);
    border: 1px solid var(--red2); color: var(--red2);
    font-family:'Cinzel',serif; font-size:0.75rem; letter-spacing:2px;
    padding:10px 22px; cursor:pointer; text-transform:uppercase; transition:all 0.2s;
  }
  .arena-locked-btn:hover { background:linear-gradient(135deg,#2d0000,#450a0a); box-shadow:0 0 14px rgba(192,57,43,0.4); }

  /* Boss fight panel */
  #boss-fight-panel {
    display: none;
    background: linear-gradient(135deg, #14080a 0%, #0d0508 100%);
    border: 1px solid var(--red3);
    border-radius: 4px;
    padding: 16px;
    margin-bottom: 1rem;
    position: relative;
    overflow: hidden;
  }
  #boss-fight-panel.active { display: block; }
  #boss-fight-panel::before {
    content:''; position:absolute; inset:0;
    background: radial-gradient(ellipse at 50% 0%, rgba(139,0,0,0.18) 0%, transparent 65%);
    pointer-events:none;
  }
  .boss-panel-title {
    font-family:'Cinzel Decorative',serif; font-size:0.72rem; color:var(--gold);
    letter-spacing:3px; text-transform:uppercase; margin-bottom:12px;
    border-bottom:1px solid var(--border); padding-bottom:8px;
    display:flex; align-items:center; gap:8px;
  }
  .live-dot {
    display:inline-block; width:7px; height:7px; border-radius:50%;
    background:var(--green2); animation: liveBlink 1.5s ease-in-out infinite;
  }
  @keyframes liveBlink { 0%,100%{opacity:1} 50%{opacity:0.2} }

  .boss-name-display {
    font-family:'Cinzel Decorative',serif; font-size:1rem; color:var(--red2);
    letter-spacing:2px; margin-bottom:3px;
    text-shadow:0 0 12px rgba(192,57,43,0.5);
  }
  .boss-tier-display { font-size:0.62rem; color:var(--gold2); letter-spacing:3px; margin-bottom:12px; }
  .boss-phase-badge {
    display:none; font-size:0.6rem; letter-spacing:2px; color:#ff6b00;
    border:1px solid #ff6b00; padding:2px 8px; margin-bottom:10px;
    animation: phasePulse 0.8s ease-in-out infinite;
  }
  .boss-phase-badge.visible { display:inline-block; }
  @keyframes phasePulse { 0%,100%{opacity:0.7} 50%{opacity:1} }

  /* Boss HP */
  .boss-hp-row { display:flex; align-items:center; justify-content:space-between; margin-bottom:5px; }
  .boss-hp-label { font-size:0.68rem; color:var(--text2); letter-spacing:1px; }
  .boss-hp-value { font-family:'IM Fell English',serif; font-size:0.8rem; color:var(--white); }
  .boss-hp-track {
    height:13px; background:rgba(255,255,255,0.05); border:1px solid var(--border);
    border-radius:2px; overflow:hidden; margin-bottom:12px; position:relative;
  }
  .boss-hp-fill {
    height:100%;
    background: linear-gradient(90deg, var(--red3), var(--red));
    transition: width 0.7s cubic-bezier(0.4,0,0.2,1);
    position:relative;
  }
  .boss-hp-fill::after {
    content:''; position:absolute; top:0; right:0; width:3px; height:100%;
    background:rgba(255,255,255,0.5); animation:hpShimmer 1.8s ease-in-out infinite;
  }
  @keyframes hpShimmer { 0%,100%{opacity:0.2} 50%{opacity:0.9} }
  .boss-hp-fill.enraged {
    background: linear-gradient(90deg, #5c1a00, #ff6b00);
    animation: enragedPulse 0.5s ease-in-out infinite;
  }
  @keyframes enragedPulse { 0%,100%{filter:brightness(1)} 50%{filter:brightness(1.35)} }

  /* Roster */
  .roster-section-title { font-size:0.63rem; color:var(--text3); letter-spacing:2px; margin-bottom:7px; text-transform:uppercase; }
  .boss-roster { display:flex; flex-direction:column; gap:5px; margin-bottom:12px; }
  .roster-entry {
    display:flex; align-items:center; gap:7px;
    background:rgba(255,255,255,0.02); border:1px solid var(--border);
    border-radius:2px; padding:5px 9px; font-size:0.68rem;
  }
  .roster-entry.me { border-color:var(--gold3); background:rgba(201,168,76,0.06); }
  .roster-skin { font-size:1rem; }
  .roster-name { flex:1; color:var(--text); letter-spacing:0.5px; }
  .roster-dmg { font-family:'IM Fell English',serif; font-size:0.68rem; color:var(--red2); }
  .roster-pct { font-family:'IM Fell English',serif; font-size:0.63rem; color:var(--gold); min-width:30px; text-align:right; }
  .me-tag { font-size:0.52rem; color:var(--gold); border:1px solid var(--gold3); padding:1px 4px; letter-spacing:1px; }

  /* Boss log */
  .boss-log-wrap { max-height:100px; overflow-y:auto; display:flex; flex-direction:column-reverse; gap:3px; }
  .boss-log-wrap::-webkit-scrollbar { width:3px; }
  .boss-log-wrap::-webkit-scrollbar-thumb { background:var(--border); }
  .boss-log-entry {
    font-family:'IM Fell English',serif; font-size:0.68rem;
    padding:3px 7px; border-radius:2px; line-height:1.4;
  }
  .boss-log-entry.bdmg    { color:var(--red2);  background:rgba(192,57,43,0.06); }
  .boss-log-entry.bheal   { color:var(--green2); background:rgba(46,204,113,0.06); }
  .boss-log-entry.bsystem { color:var(--gold);   background:rgba(201,168,76,0.06); }
  .boss-log-entry.batk    { color:#ff9944;        background:rgba(255,100,0,0.06); }
  .boss-log-entry.bspecial{ color:var(--purple);  background:rgba(142,68,173,0.06); }

  /* Skill tap mode (during boss) */
  .skill-boss-info {
    font-size:0.68rem; color:var(--text3); font-style:italic;
    line-height:1.6; margin-bottom:10px; padding:8px 10px;
    background:rgba(0,0,0,0.2); border:1px solid var(--border);
    border-radius:3px; display:none;
  }
  .skill-boss-info.active { display:block; }
  .skill-boss-damage-badge {
    display:none; font-size:0.58rem; color:var(--red2); border:1px solid var(--red3);
    padding:2px 6px; letter-spacing:1px; margin-top:4px; text-align:center;
  }
  .skill-boss-damage-badge.active { display:block; }

  /* Boss attack button */
  #boss-attack-btn {
    display:none;
    width:100%; padding:14px;
    background:linear-gradient(135deg,#1a0000,#2d0000);
    border:1px solid var(--red2); color:var(--red2);
    font-family:'Cinzel Decorative',serif; font-size:0.85rem; letter-spacing:3px;
    cursor:pointer; text-transform:uppercase; position:relative; overflow:hidden;
    transition:all 0.2s; margin-bottom:10px;
  }
  #boss-attack-btn.active { display:block; }
  #boss-attack-btn:not(:disabled):hover {
    background:linear-gradient(135deg,#2d0000,#450000);
    box-shadow:0 0 18px rgba(192,57,43,0.4); border-color:var(--gold); color:var(--gold);
  }
  #boss-attack-btn:not(:disabled):active { transform:scale(0.98); }
  #boss-attack-btn:disabled { opacity:0.35; cursor:not-allowed; }
  .boss-btn-cd-bar {
    position:absolute; bottom:0; left:0; height:3px;
    background:var(--red2); transition:width 0.1s linear; width:0%;
  }

  #boss-heavy-btn {
    display:none;
    width:100%; padding:12px;
    background:linear-gradient(135deg,#0d0000,#1a0800);
    border:1px solid #7a1a10; color:#c0392b;
    font-family:'Cinzel Decorative',serif; font-size:0.8rem; letter-spacing:2px;
    cursor:pointer; text-transform:uppercase; position:relative; overflow:hidden;
    transition:all 0.2s; margin-bottom:10px;
  }
  #boss-heavy-btn.active { display:block; }
  #boss-heavy-btn:not(:disabled):hover {
    background:linear-gradient(135deg,#1a0000,#300500);
    box-shadow:0 0 18px rgba(139,0,0,0.5); border-color:var(--red2); color:var(--red2);
  }
  #boss-heavy-btn:not(:disabled):active { transform:scale(0.98); }
  #boss-heavy-btn:disabled { opacity:0.35; cursor:not-allowed; }

  /* ═══════════════════════════════════════
     BOSS ANIMATED GRAPHIC
  ═══════════════════════════════════════ */

  .boss-graphic-wrap {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 130px;
    margin-bottom: 14px;
    overflow: hidden;
  }

  /* Aura ring behind the boss */
  .boss-aura {
    position: absolute;
    width: 110px; height: 110px;
    border-radius: 50%;
    background: radial-gradient(circle, rgba(139,0,0,0.35) 0%, transparent 70%);
    animation: bossAuraPulse 2.5s ease-in-out infinite;
    pointer-events: none;
  }
  .boss-aura.rage {
    background: radial-gradient(circle, rgba(255,80,0,0.45) 0%, transparent 70%);
    animation: bossAuraPulse 0.8s ease-in-out infinite;
  }
  @keyframes bossAuraPulse {
    0%,100% { transform: scale(1);   opacity: 0.6; }
    50%     { transform: scale(1.25); opacity: 1;   }
  }

  /* Ground shadow */
  .boss-shadow {
    position: absolute;
    bottom: 8px;
    width: 80px; height: 14px;
    background: radial-gradient(ellipse, rgba(0,0,0,0.7) 0%, transparent 70%);
    border-radius: 50%;
    animation: bossShadowBreath 2.5s ease-in-out infinite;
  }
  @keyframes bossShadowBreath {
    0%,100% { transform: scaleX(1);   opacity: 0.7; }
    50%     { transform: scaleX(1.15); opacity: 0.4; }
  }

  /* The sprite itself */
  .boss-sprite-el {
    font-size: 5.5rem;
    line-height: 1;
    display: block;
    position: relative;
    z-index: 2;
    transform-origin: center bottom;
    filter: drop-shadow(0 6px 16px rgba(0,0,0,0.8)) drop-shadow(0 0 20px rgba(139,0,0,0.6));
    animation: bossIdle 3s ease-in-out infinite;
    will-change: transform, filter;
    cursor: default;
    user-select: none;
  }
  .boss-sprite-el.rage {
    filter: drop-shadow(0 6px 16px rgba(0,0,0,0.8)) drop-shadow(0 0 30px rgba(255,80,0,0.9));
    animation: bossIdleRage 1.2s ease-in-out infinite;
  }

  /* Idle float */
  @keyframes bossIdle {
    0%,100% { transform: translateY(0)   scale(1)    rotate(0deg); }
    30%     { transform: translateY(-8px) scale(1.02) rotate(-1deg); }
    60%     { transform: translateY(-5px) scale(1.01) rotate(0.5deg); }
  }
  /* Enraged idle — faster, more erratic */
  @keyframes bossIdleRage {
    0%,100% { transform: translateY(0)    scale(1)    rotate(0deg); }
    25%     { transform: translateY(-10px) scale(1.05) rotate(-2deg); }
    75%     { transform: translateY(-4px)  scale(1.03) rotate(2deg); }
  }

  /* Hit flash */
  .boss-sprite-el.boss-taking-hit {
    animation: bossTakingHit 0.5s ease forwards !important;
  }
  @keyframes bossTakingHit {
    0%   { transform: translateX(0)    scale(1)    rotate(0deg);  filter: drop-shadow(0 6px 16px rgba(0,0,0,0.8)) drop-shadow(0 0 20px rgba(139,0,0,0.6)); }
    15%  { transform: translateX(18px) scale(0.88) rotate(4deg);  filter: brightness(3) saturate(0) drop-shadow(0 0 30px white); }
    35%  { transform: translateX(-12px)scale(0.92) rotate(-3deg); filter: brightness(2) drop-shadow(0 0 20px rgba(255,80,0,1)); }
    60%  { transform: translateX(7px)  scale(0.97) rotate(1deg);  filter: brightness(1.4); }
    100% { transform: translateX(0)    scale(1)    rotate(0deg);  filter: drop-shadow(0 6px 16px rgba(0,0,0,0.8)) drop-shadow(0 0 20px rgba(139,0,0,0.6)); }
  }

  /* Boss attacks */
  .boss-sprite-el.boss-attacking {
    animation: bossAttack 0.7s cubic-bezier(0.22,1,0.36,1) forwards !important;
  }
  @keyframes bossAttack {
    0%   { transform: translateX(0)     translateY(0)     scale(1)   rotate(0deg);  filter: drop-shadow(0 6px 16px rgba(0,0,0,0.8)); }
    15%  { transform: translateX(20px)  translateY(-25px) scale(1.5) rotate(8deg);  filter: drop-shadow(0 0 20px rgba(255,60,0,0.7)) brightness(1.4); }
    25%  { transform: translateX(20px)  translateY(-38px) scale(1.7) rotate(12deg); filter: drop-shadow(0 0 40px rgba(255,20,0,1))   brightness(2); }
    50%  { transform: translateX(-80px) translateY(-8px)  scale(2.0) rotate(-5deg); filter: drop-shadow(0 0 60px rgba(255,0,0,1))   brightness(2.5); }
    65%  { transform: translateX(-60px) translateY(0)     scale(1.4) rotate(-2deg); filter: drop-shadow(0 0 25px rgba(255,60,0,0.6)); }
    82%  { transform: translateX(-8px)  translateY(0)     scale(1.05) rotate(0deg); }
    100% { transform: translateX(0)     translateY(0)     scale(1)    rotate(0deg); filter: drop-shadow(0 6px 16px rgba(0,0,0,0.8)) drop-shadow(0 0 20px rgba(139,0,0,0.6)); }
  }

  /* Particle burst on hit */
  .boss-hit-particle {
    position: absolute;
    width: 6px; height: 6px;
    border-radius: 50%;
    pointer-events: none;
    z-index: 10;
    animation: bossParticleBurst 0.8s ease-out forwards;
  }
  @keyframes bossParticleBurst {
    0%   { transform: translate(0,0) scale(1); opacity: 1; }
    100% { transform: var(--p-end) scale(0); opacity: 0; }
  }

  /* Boss name + health label under graphic */
  .boss-graphic-label {
    font-family: 'Cinzel', serif;
    font-size: 0.62rem;
    color: var(--text3);
    letter-spacing: 2px;
    text-align: center;
    margin-top: 4px;
    position: relative;
    z-index: 2;
  }


  .btn-boss-spawn {
    background: linear-gradient(135deg, #1a0000, #2d0000);
    border: 1px solid var(--red);
    color: var(--red2);
    font-family: 'Cinzel Decorative', serif;
    font-size: 0.7rem;
    letter-spacing: 2px;
    padding: 0.45rem 1rem;
    cursor: pointer;
    text-transform: uppercase;
    transition: all 0.2s;
    white-space: nowrap;
    animation: bossButtonGlow 3s ease-in-out infinite;
  }
  .btn-boss-spawn:hover {
    background: linear-gradient(135deg, #2d0000, #450000);
    border-color: var(--gold);
    color: var(--gold);
    box-shadow: 0 0 14px rgba(192,57,43,0.5);
    animation: none;
  }
  .btn-boss-spawn.boss-live {
    border-color: var(--gold);
    color: var(--gold);
    animation: bossLivePulse 1s ease-in-out infinite;
  }
  @keyframes bossButtonGlow {
    0%,100% { box-shadow: 0 0 4px rgba(192,57,43,0.2); }
    50%     { box-shadow: 0 0 10px rgba(192,57,43,0.5); }
  }
  @keyframes bossLivePulse {
    0%,100% { box-shadow: 0 0 8px rgba(201,168,76,0.4); border-color: var(--gold); }
    50%     { box-shadow: 0 0 18px rgba(201,168,76,0.7); border-color: var(--gold2); }
  }

  /* Modal inputs focus */
  #boss-spawn-modal input:focus,
  #boss-spawn-modal select:focus {
    border-color: var(--red3) !important;
    outline: none;
  }
  #boss-spawn-confirm-btn:hover { background: linear-gradient(135deg,#2d0000,#450000) !important; box-shadow: 0 0 14px rgba(192,57,43,0.4); }
  #boss-end-btn:hover { border-color: var(--text2) !important; color: var(--text2) !important; }


  .boss-float-dmg {
    position:fixed; font-family:'Cinzel Decorative',serif;
    font-size:1.2rem; font-weight:900; pointer-events:none; z-index:9998;
    animation: bossDmgFloat 1.2s ease-out forwards;
  }
  @keyframes bossDmgFloat {
    0%   { transform:translateY(0) scale(1); opacity:1; }
    100% { transform:translateY(-70px) scale(0.65); opacity:0; }
  }

  /* Enhancements disabled badge */
  .enhancements-disabled-note {
    display:none; font-size:0.65rem; color:var(--text3); text-align:center;
    padding:6px 10px; border:1px solid var(--border); border-radius:3px;
    font-style:italic; margin-bottom:8px;
  }
  .enhancements-disabled-note.active { display:block; }


<link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@300;400;500&display=swap" rel="stylesheet">
</head>
<script>
// Pre-stub: queue storage calls until PveStorage.install() is ready
(function() {
  if (window.storage) return;
  var _q = [];
  var _stub = function(method) {
    return function() {
      var args = arguments;
      return new Promise(function(resolve) {
        _q.push(function() { resolve(window.storage[method].apply(window.storage, args)); });
      });
    };
  };
  window._storageQueue = _q;
  window.storage = {
    get: _stub('get'), set: _stub('set'),
    delete: _stub('delete'), list: _stub('list'),
    _isStub: true
  };
})();
</script>
<body>
<!-- ── SITE NAV ── -->
<style>
#site-nav{position:sticky;top:0;z-index:9000;background:#0a0a0f;border-bottom:1px solid rgba(255,255,255,0.07);display:flex;align-items:center;justify-content:space-between;padding:0 18px;height:48px;font-family:'DM Mono',monospace;gap:12px;}
#site-nav .sn-logo{font-size:0.72rem;letter-spacing:0.18em;text-transform:uppercase;color:#e8e6f0;text-decoration:none;white-space:nowrap;flex-shrink:0;}
#site-nav .sn-logo span{color:#c8a96e;font-style:italic;}
#site-nav .sn-links{display:flex;align-items:center;gap:4px;overflow-x:auto;-webkit-overflow-scrolling:touch;scrollbar-width:none;flex:1;}
#site-nav .sn-links::-webkit-scrollbar{display:none;}
#site-nav .sn-links a{font-size:0.62rem;letter-spacing:0.1em;text-transform:uppercase;padding:5px 10px;border-radius:6px;text-decoration:none;color:#6b6880;border:1px solid transparent;white-space:nowrap;transition:all 0.18s;}
#site-nav .sn-links a:hover{color:#e8e6f0;border-color:rgba(255,255,255,0.1);background:rgba(255,255,255,0.04);}
#site-nav .sn-links a.sn-active{color:#e8e6f0;border-color:rgba(255,255,255,0.12);background:rgba(255,255,255,0.06);}
#site-nav .sn-links a.sn-arena.sn-active{color:#e06c6c;border-color:rgba(224,108,108,0.3);background:rgba(224,108,108,0.07);}
#site-nav .sn-links a.sn-trade.sn-active{color:#7faaff;border-color:rgba(127,170,255,0.3);background:rgba(127,170,255,0.07);}
#site-nav .sn-links a.sn-tourney.sn-active{color:#d4a8ff;border-color:rgba(212,168,255,0.3);background:rgba(212,168,255,0.07);}
</style>
<nav id="site-nav">
  <a class="sn-logo" href="index.html">PvE <span>Hub</span></a>
  <div class="sn-links">
    <a href="index.html" class="sn-home">Home</a>
    <a href="iron-arena.html" class="sn-arena sn-active">⚔ Iron Arena</a>
    <a href="trade-together.html" class="sn-trade">📊 Trade Together</a>
  </div>
</nav>

<!-- ══ BOSS ALERT BANNER ══ -->
<div id="boss-alert-banner">
  <div class="boss-alert-inner">
    <span class="boss-alert-skull">💀</span>
    <span class="boss-alert-text">BOSS FIGHT ACTIVE —</span>
    <span class="boss-alert-name" id="boss-alert-name-text">THE IRON COLOSSUS</span>
    <span class="boss-alert-text">— ALL COMBATANTS REPORT</span>
    <span class="boss-alert-skull">💀</span>
  </div>
</div>






  
<div class="game-header">
  <h1>⚔ IRON ARENA ⚔</h1>
  <p><span class="ornament">✦</span> Turn-Based Combat — Train, Fight, Survive <span class="ornament">✦</span></p>
</div>

<div class="save-bar">
  <button class="btn-leaderboard" onclick="openLeaderboard()">🏆 Leaderboard</button>
  <button class="btn-save" onclick="saveGame()">💾 Save Progress</button>
  <span class="save-info">Auto-saves on wins &amp; purchases</span>
  <button class="btn-admin-snapshot" id="btn-admin-snapshot" onclick="downloadSnapshotCSV()">📊 XP Snapshot</button>
  <button class="btn-admin-snapshot" id="btn-skin-snapshot" onclick="openSkinSnapshot()" style="display:none;">🎭 Skin Snapshot</button>
  <button class="btn-delete-save" onclick="deleteSave()">🗑 Reset Save</button>
  <button class="btn-boss-spawn" id="btn-boss-spawn" onclick="toggleBossSpawnModal()">☠ BOSS FIGHT</button>
</div>

<!-- ══ BOSS SPAWN MODAL ══ -->
<div id="boss-spawn-overlay" onclick="closeBossSpawnModal()" style="display:none;position:fixed;inset:0;z-index:9500;background:rgba(0,0,0,0.82);backdrop-filter:blur(4px);"></div>
<div id="boss-spawn-modal" style="display:none;position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);z-index:9501;width:min(94vw,420px);background:#110a0b;border:1px solid #8b0000;border-radius:4px;padding:22px 20px;font-family:'IM Fell English',serif;">
  <div style="font-family:'Cinzel Decorative',serif;font-size:1rem;color:#e74c3c;letter-spacing:3px;text-align:center;margin-bottom:4px;">☠ INITIATE BOSS FIGHT</div>
  <div style="font-size:0.7rem;color:#6a5040;letter-spacing:2px;text-align:center;margin-bottom:18px;font-family:'Cinzel',serif;">ADMIN · WORLD EVENT</div>

  <div style="display:flex;flex-direction:column;gap:10px;margin-bottom:16px;">
    <div style="display:flex;flex-direction:column;gap:4px;">
      <label style="font-size:0.7rem;color:#a08060;letter-spacing:1px;">BOSS NAME</label>
      <input id="boss-spawn-name" type="text" value="The Iron Colossus" maxlength="40" style="background:#0a0608;border:1px solid #3a2e20;color:#f0e8d8;font-family:'IM Fell English',serif;font-size:0.85rem;padding:8px 10px;border-radius:2px;outline:none;width:100%;">
    </div>
    <div style="display:flex;gap:10px;">
      <div style="display:flex;flex-direction:column;gap:4px;flex:1;">
        <label style="font-size:0.7rem;color:#a08060;letter-spacing:1px;">BOSS HP</label>
        <input id="boss-spawn-hp" type="number" value="1000000" min="10000" step="50000" style="background:#0a0608;border:1px solid #3a2e20;color:#f0e8d8;font-family:'IM Fell English',serif;font-size:0.85rem;padding:8px 10px;border-radius:2px;outline:none;width:100%;">
      </div>
      <div style="display:flex;flex-direction:column;gap:4px;flex:1;">
        <label style="font-size:0.7rem;color:#a08060;letter-spacing:1px;">LOOT POOL ($PvE)</label>
        <input id="boss-spawn-loot" type="number" value="25000" min="0" step="1000" style="background:#0a0608;border:1px solid #3a2e20;color:#f0e8d8;font-family:'IM Fell English',serif;font-size:0.85rem;padding:8px 10px;border-radius:2px;outline:none;width:100%;">
      </div>
    </div>
    <div style="display:flex;flex-direction:column;gap:4px;">
      <label style="font-size:0.7rem;color:#a08060;letter-spacing:1px;">TIER</label>
      <select id="boss-spawn-tier" style="background:#0a0608;border:1px solid #3a2e20;color:#f0e8d8;font-family:'IM Fell English',serif;font-size:0.85rem;padding:8px 10px;border-radius:2px;outline:none;width:100%;">
        <option value="⬛ TITAN TIER · WORLD BOSS">⬛ Titan — World Boss</option>
        <option value="🟣 MYTHIC TIER · GUILD BOSS">🟣 Mythic — Guild Boss</option>
        <option value="🔴 LEGENDARY TIER · RAID BOSS">🔴 Legendary — Raid Boss</option>
        <option value="🟠 EPIC TIER · EVENT BOSS">🟠 Epic — Event Boss</option>
      </select>
    </div>
  </div>

  <!-- Active boss warning -->
  <div id="boss-spawn-active-warning" style="display:none;font-size:0.72rem;color:#e67e22;border:1px solid #7a4510;padding:8px 10px;border-radius:3px;margin-bottom:12px;text-align:center;font-style:italic;">
    ⚠ A boss fight is already active. End it first before spawning a new one.
  </div>

  <div style="display:flex;gap:10px;">
    <button onclick="spawnBossFromModal()" id="boss-spawn-confirm-btn" style="flex:1;background:linear-gradient(135deg,#1a0000,#2d0000);border:1px solid #e74c3c;color:#e74c3c;font-family:'Cinzel Decorative',serif;font-size:0.75rem;letter-spacing:2px;padding:12px;cursor:pointer;transition:all 0.2s;text-transform:uppercase;">
      ⚔ SPAWN BOSS
    </button>
    <button onclick="endBossFromModal()" id="boss-end-btn" style="flex:1;background:linear-gradient(135deg,#0a0a00,#1a1a00);border:1px solid #6a5040;color:#6a5040;font-family:'Cinzel',serif;font-size:0.72rem;letter-spacing:2px;padding:12px;cursor:pointer;transition:all 0.2s;text-transform:uppercase;">
      ☠ END FIGHT
    </button>
    <button onclick="closeBossSpawnModal()" style="background:transparent;border:1px solid #3a2e20;color:#6a5040;font-family:'Cinzel',serif;font-size:0.72rem;letter-spacing:1px;padding:12px 16px;cursor:pointer;transition:all 0.2s;">
      ✕
    </button>
  </div>
</div>

<div class="firebase-banner" id="firebase-banner">
  <span class="fb-icon">🔥</span>
  <span><strong>Leaderboard &amp; Guilds need Firebase.</strong>
  Open the file in a text editor, find <code style="background:rgba(0,0,0,0.3);padding:0.1rem 0.3rem;border-radius:2px;">YOUR_PROJECT</code> and paste your
  <a href="https://console.firebase.google.com" target="_blank">Firebase config</a>.
  Takes 5 minutes — see instructions below.</span>
</div>

<div class="how-to-play">
  <button class="htp-toggle" onclick="toggleHTP(this)">
    <span>📖 HOW TO PLAY — Click to expand</span>
    <span class="arrow">▼</span>
  </button>
  <div class="htp-body" id="htp-body">
    <div class="htp-sections">

      <div class="htp-card">
        <div class="htp-card-title" style="color:var(--red2)">⚔️ Fighting</div>
        <ul>
          <li><span class="li-icon">1️⃣</span><span>Hit <strong style="color:var(--white)">ENTER THE ARENA</strong> to start a fight against the current wave's enemy.</span></li>
          <li><span class="li-icon">🛡️</span><span><strong style="color:#60a5fa">BLOCK</strong> costs 5 stamina and heavily reduces the next enemy hit.</span></li>
          <li><span class="li-icon">⚡</span><span><strong style="color:#f39c12">LIGHT ATTACK</strong> costs 10 stamina — reliable, safe damage every turn.</span></li>
          <li><span class="li-icon">💀</span><span><strong style="color:var(--red2)">HEAVY ATTACK</strong> costs 25 stamina — nearly 2× the damage of a Light Attack.</span></li>
          <li><span class="li-icon">💚</span><span>Stamina <strong style="color:var(--white)">regenerates</strong> a little each turn. Running out limits your moves.</span></li>
          <li><span class="li-icon">💥</span><span>Your <strong style="color:#c084fc">Critical</strong> skill gives a chance to deal bonus multiplied damage on any attack.</span></li>
          <li><span class="li-icon">♻️</span><span>If you die, you can <strong style="color:var(--white)">Respawn</strong> — but you'll lose 10% of your total XP.</span></li>
        </ul>
      </div>

      <div class="htp-card">
        <div class="htp-card-title" style="color:var(--green2)">⚡ Leveling Up Skills</div>
        <ul>
          <li><span class="li-icon">👆</span><span>Find the <strong style="color:var(--white)">Skill Training</strong> section at the bottom. Each skill has a <strong style="color:var(--white)">LEVEL UP</strong> button.</span></li>
          <li><span class="li-icon">🔢</span><span>Click <strong style="color:var(--gold)">25 times</strong> on a skill's button to earn a level-up.</span></li>
          <li><span class="li-icon">⏳</span><span>After leveling up, that skill enters a <strong style="color:var(--white)">1-minute cooldown</strong>. Plan ahead!</span></li>
          <li><span class="li-icon">💚</span><span><strong style="color:var(--green2)">Stamina</strong> — increases your max HP and max stamina pool.</span></li>
          <li><span class="li-icon">⚔️</span><span><strong style="color:var(--red2)">Attack</strong> — increases damage dealt on both Light and Heavy attacks.</span></li>
          <li><span class="li-icon">🛡️</span><span><strong style="color:#60a5fa">Defense</strong> — reduces incoming damage and improves your block.</span></li>
          <li><span class="li-icon">💥</span><span><strong style="color:#c084fc">Critical</strong> — raises your crit chance and crit damage multiplier.</span></li>
        </ul>
      </div>

      <div class="htp-card">
        <div class="htp-card-title" style="color:var(--gold)">🏪 Spending XP</div>
        <ul>
          <li><span class="li-icon">🏆</span><span>Winning fights earns <strong style="color:var(--gold)">XP</strong>. Tougher enemies on higher waves drop much more.</span></li>
          <li><span class="li-icon">🏷️</span><span><strong style="color:var(--white)">Name</strong> your hero for free the first time. Renaming after that costs 10 XP.</span></li>
          <li><span class="li-icon">🎭</span><span>Unlock <strong style="color:var(--white)">Cosmetic Skins</strong> (20–250 XP) to change your warrior's appearance on the battlefield.</span></li>
          <li><span class="li-icon">📈</span><span>Spend XP to instantly <strong style="color:var(--gold)">Boost a Skill</strong> by 1 level (15–20 XP each) — no clicking required!</span></li>
          <li><span class="li-icon">☠️</span><span>Unlock <strong style="color:var(--red2)">ONE STRIKE</strong> for 150 XP — a powerful gamble attack (see below).</span></li>
        </ul>
      </div>

      <div class="htp-card">
        <div class="htp-card-title" style="color:var(--red2)">☠ ONE STRIKE</div>
        <ul>
          <li><span class="li-icon">🔓</span><span>Purchase ONE STRIKE in the <strong style="color:var(--white)">Special</strong> shop tab for <strong style="color:var(--gold)">150 XP</strong>.</span></li>
          <li><span class="li-icon">🎯</span><span>Only a <strong style="color:var(--red2)">25% chance to hit</strong> — it's a high-risk gamble every time.</span></li>
          <li><span class="li-icon">💀</span><span>If it <strong style="color:var(--red2)">hits</strong>, the enemy is instantly killed — regardless of HP or level.</span></li>
          <li><span class="li-icon">💸</span><span>Whether it hits or misses, you lose <strong style="color:var(--white)">75% of your stamina</strong> and <strong style="color:var(--white)">50% of your max HP</strong>.</span></li>
          <li><span class="li-icon">⚠️</span><span>You can only use ONE STRIKE <strong style="color:var(--gold)">once per match</strong>. Use it wisely!</span></li>
        </ul>
      </div>

      <div class="htp-card">
        <div class="htp-card-title" style="color:#a78bfa">🧪 Venom of Spite &amp; 🌫️ Veil of Shadows</div>
        <ul>
          <li><span class="li-icon">🛒</span><span>Both potions are purchased in the <strong style="color:var(--white)">Special</strong> shop tab. Cost scales up with your current wave.</span></li>
          <li><span class="li-icon">⚠️</span><span><strong style="color:#fbbf24">Potions are active for the next round only</strong> — they are consumed the moment the fight begins and do not carry over.</span></li>
          <li><span class="li-icon">🧪</span><span><strong style="color:#a78bfa">Venom of Spite</strong> — poisons the enemy at fight start. They take <strong style="color:#a78bfa">10% of their max HP</strong> as damage every turn for the whole fight.</span></li>
          <li><span class="li-icon">🌫️</span><span><strong style="color:#60a5fa">Veil of Shadows</strong> — shrouds you in darkness. The enemy has a <strong style="color:#60a5fa">25% chance to miss</strong> every attack for the whole fight.</span></li>
          <li><span class="li-icon">💡</span><span>Stack both in the same round for a serious edge — buy before you hit <strong style="color:var(--white)">ENTER THE ARENA</strong>.</span></li>
        </ul>
      </div>

    </div>
    <div class="htp-tip">
      💡 <strong>Pro tip:</strong> Train skills between every wave. Enemies scale hard — a well-timed Heavy Attack with high Attack skill will carry you far. Save ONE STRIKE for Legendary or Mythic tier bosses!
    </div>
  </div>
</div>

<div class="game-container">

  <!-- ══ BOSS FIGHT PANEL (shown when a boss is active) ══ -->
  <div id="boss-fight-panel">
    <div class="boss-panel-title">
      <span class="live-dot"></span>
      BOSS FIGHT IN PROGRESS
    </div>
    <div class="boss-name-display" id="boss-name-display">THE IRON COLOSSUS</div>
    <div class="boss-tier-display" id="boss-tier-display">⬛ TITAN TIER · WORLD BOSS</div>
    <div class="boss-phase-badge" id="boss-phase-badge">⚠ RAGE PHASE ACTIVE</div>

    <!-- Boss animated graphic -->
    <div class="boss-graphic-wrap" id="boss-graphic-wrap">
      <div class="boss-aura" id="boss-aura"></div>
      <span class="boss-sprite-el" id="boss-sprite-el">👁️</span>
      <div class="boss-shadow"></div>
      <div class="boss-graphic-label" id="boss-graphic-label">THE IRON COLOSSUS</div>
    </div>

    <div class="boss-hp-row">
      <span class="boss-hp-label"><span class="live-dot"></span> BOSS HP — LIVE</span>
      <span class="boss-hp-value" id="boss-hp-value-text">1,000,000 / 1,000,000</span>
    </div>
    <div class="boss-hp-track">
      <div class="boss-hp-fill" id="boss-hp-fill" style="width:100%;"></div>
    </div>

    <div class="roster-section-title">⚔ ACTIVE COMBATANTS</div>
    <div class="boss-roster" id="boss-roster"></div>

    <button id="boss-attack-btn" onclick="bossDoAttack('light')">
      ⚔ STRIKE THE BOSS ⚔
      <div class="boss-btn-cd-bar" id="boss-btn-cd-bar"></div>
    </button>
    <button id="boss-heavy-btn" onclick="bossDoAttack('heavy')">
      💀 HEAVY STRIKE 💀
      <div style="font-size:0.68rem;opacity:0.65;margin-top:2px;font-family:'IM Fell English',serif;font-style:italic;">2× damage · 8s cooldown</div>
      <div class="boss-btn-cd-bar" id="boss-heavy-cd-bar"></div>
    </button>

    <div class="boss-panel-title" style="margin-top:8px;">📜 BATTLE FEED</div>
    <div class="boss-log-wrap" id="boss-log-wrap"></div>
  </div>

  <!-- ══ SECTION 1: FIGHT ARENA (top) ══ -->
  <div class="arena-col" style="position:relative;">

    <!-- Combatants -->
    <div class="panel arena-panel" style="position:relative;">
      <div class="combatants">
        <!-- Player -->
        <div class="combatant-card" id="player-card">
          <span class="combatant-sprite" id="player-sprite">🧙</span>
          <div class="combatant-name">THE WARRIOR</div>
          <div class="combatant-title" id="player-title">Novice Wanderer</div>
          <div class="stat-bar-row">
            <div class="stat-bar-label"><span>❤️ HP</span><span id="player-hp-txt">100 / 100</span></div>
            <div class="stat-bar-outer"><div class="stat-bar-inner hp-bar" id="player-hp-bar" style="width:100%"></div></div>
          </div>
          <div class="stat-bar-row">
            <div class="stat-bar-label"><span>💚 Stamina</span><span id="player-sta-txt">100 / 100</span></div>
            <div class="stat-bar-outer"><div class="stat-bar-inner stamina-bar" id="player-sta-bar" style="width:100%"></div></div>
          </div>
        </div>

        <div class="vs-divider">VS</div>

        <!-- Enemy -->
        <div class="combatant-card enemy-card" id="enemy-card">
          <div class="enemy-tier tier-common" id="enemy-tier-badge">COMMON</div>
          <span class="combatant-sprite" id="enemy-sprite">🐺</span>
          <div class="combatant-name" id="enemy-name">WILD WOLF</div>
          <div class="combatant-title" id="enemy-subtitle">A hungry beast of the wilds</div>
          <div class="stat-bar-row">
            <div class="stat-bar-label"><span>❤️ HP</span><span id="enemy-hp-txt">? / ?</span></div>
            <div class="stat-bar-outer"><div class="stat-bar-inner enemy-hp-bar" id="enemy-hp-bar" style="width:100%"></div></div>
          </div>
          <div class="stat-bar-row">
            <div class="stat-bar-label"><span>⚔️ ATK</span><span id="enemy-atk-txt">?</span></div>
            <div class="stat-bar-label" style="margin-top:0.2rem"><span>🛡️ DEF</span><span id="enemy-def-txt">?</span></div>
          </div>
        </div>
      </div>
      <div class="turn-indicator" id="turn-indicator">— Prepare for Battle —</div>
    </div>

    <!-- Actions -->
    <div class="action-area panel" id="action-area">
      <div class="action-title">— Choose Your Action —</div>
      <div class="action-buttons">
        <button class="action-btn btn-block" id="btn-action-block" onclick="playerAction('block')" disabled>
          <span class="btn-icon">🛡️</span>
          <span class="btn-name">BLOCK</span>
          <span class="btn-cost">-5 Stamina</span>
        </button>
        <button class="action-btn btn-light" id="btn-action-light" onclick="playerAction('light')" disabled>
          <span class="btn-icon">⚡</span>
          <span class="btn-name">LIGHT ATK</span>
          <span class="btn-cost">-10 Stamina</span>
        </button>
        <button class="action-btn btn-heavy" id="btn-action-heavy" onclick="playerAction('heavy')" disabled>
          <span class="btn-icon">💀</span>
          <span class="btn-name">HEAVY ATK</span>
          <span class="btn-cost">-25 Stamina</span>
        </button>
      </div>

      <button class="btn-one-strike hidden" id="btn-one-strike" onclick="playerAction('onestrike')">
        ☠ ONE STRIKE — KILL OR WOUND ☠
        <div style="font-size:0.75rem;opacity:0.7;margin-top:0.2rem;font-family:'IM Fell English',serif;font-style:italic;">25% hit · Kills instantly · −75% Stamina · −50% HP</div>
      </button>

      <div id="potion-active-display" class="hidden" style="
        margin-top:0.6rem;
        padding:0.55rem 0.8rem;
        border-radius:4px;
        background:rgba(0,0,0,0.25);
        border:1px solid rgba(255,255,255,0.07);
        font-family:'Cinzel',serif;
        font-size:0.78rem;
        letter-spacing:0.06em;
        display:flex;
        flex-direction:column;
        gap:0.3rem;
      "></div>

      <div class="status-row">
        <span class="status-badge badge-idle" id="status-badge">⚙ IDLE</span>
        <span class="wave-info">Wave <span id="wave-num">1</span> · <span id="enemy-count">0</span> Defeated</span>
        <span class="wave-info">XP: <span id="xp-total-display"><span>0</span></span></span>
      </div>

      <!-- Boss mode: enhancements disabled notice -->
      <div class="enhancements-disabled-note" id="enhancements-disabled-note">
        ⛔ One Strike &amp; special enhancements are disabled during Boss Fights
      </div>

      <button class="main-btn btn-fight" id="btn-fight" onclick="startFight()">⚔ ENTER THE ARENA</button>
      <button class="main-btn btn-next hidden" id="btn-next" onclick="nextEnemy()">▶ NEXT CHALLENGER</button>
      <button class="main-btn btn-respawn hidden" id="btn-respawn" onclick="respawn()">♻ RESPAWN (Lose 10% XP)</button>
    </div>



    <!-- Battle Log -->
    <div class="panel log-panel">
      <div class="panel-title">📜 Battle Chronicle</div>
      <div class="battle-log" id="battle-log">
        <div class="log-entry log-system">⚔ Welcome to the Iron Arena. Train your skills and face your enemies.</div>
        <div class="log-entry log-system">💡 Click LEVEL UP 25 times per skill to advance. Each level-up triggers a 1-minute cooldown.</div>
      </div>
    </div>


  </div><!-- /arena-col -->

  <!-- ══ SECTION 2: XP SHOP (middle) ══ -->
  <div class="panel shop-panel">
    <div class="panel-title">🏪 The Iron Bazaar</div>
    <div style="text-align:center; margin-bottom:0.8rem;">
      <span style="font-family:'Cinzel',serif; font-size:0.9rem; color:var(--text3);">Available XP: </span>
      <span style="font-family:'Cinzel',serif; font-size:1.1rem; color:var(--gold); font-weight:bold;" id="shop-xp-display">0</span>
    </div>
    <div class="shop-tabs">
      <button class="shop-tab active" onclick="switchShopTab('name')">🏷 Name</button>
      <button class="shop-tab" onclick="switchShopTab('skins')">🎭 Skins</button>
      <button class="shop-tab" onclick="switchShopTab('skills')">📈 Skills</button>
      <button class="shop-tab" onclick="switchShopTab('guild')">⚔ Guild</button>
      <button class="shop-tab" onclick="switchShopTab('special')">☠ Special</button>
    </div>

    <!-- NAME TAB -->
    <div class="shop-section active" id="tab-name">

      <!-- Hero Name -->
      <div class="shop-item">
        <div class="shop-item-name">🏷️ Hero Name</div>
        <div class="shop-item-desc">Name your warrior. First change is free — after that, 10 XP each.</div>
        <div class="name-input-row">
          <input class="name-input" id="hero-name-input" type="text" maxlength="16" placeholder="Enter name..." />
          <button class="btn-buy btn-buy-gold" id="btn-set-name" onclick="setHeroName()">SET</button>
        </div>
        <div style="font-size:0.8rem; color:var(--text3); margin-top:0.4rem; font-style:italic;" id="name-cost-label">First change is free</div>
      </div>

      <!-- Wallet -->
      <div class="shop-item">
        <div class="shop-item-name">👛 Wallet</div>
        <div class="shop-item-desc">Connect your Ethereum wallet to save progress and receive $PvE rewards. Works with MetaMask, Coinbase, Rabby, Rainbow, Trust, and any WalletConnect-compatible wallet.</div>
        <div class="wallet-linked-banner hidden" id="wallet-linked-banner"></div>
        <div id="wallet-connect-area">
          <button class="btn-buy btn-buy-gold" onclick="PveAuth._openModal()" style="width:100%;padding:0.6rem;font-size:0.9rem;letter-spacing:0.1em;">
            ⬡ CONNECT WALLET
          </button>
          <div style="font-size:0.72rem;color:var(--text3);margin-top:0.5rem;font-style:italic;text-align:center;">
            Signs a message to verify ownership — no transaction, no gas fee.
          </div>
        </div>
      </div>

    </div>

    <!-- SKINS TAB -->
    <div class="shop-section" id="tab-skins">
      <div style="font-size:0.85rem; color:var(--text3); font-style:italic; margin-bottom:0.7rem;">Tap an owned skin to equip it.</div>
      <div class="skin-grid" id="skin-grid"></div>
    </div>

    <!-- SKILLS XP TAB -->
    <div class="shop-section" id="tab-skills">
      <div style="font-size:0.85rem; color:var(--text3); font-style:italic; margin-bottom:0.7rem;">Spend XP to instantly boost a skill by 1 level.</div>
      <div id="xp-skill-list"></div>
    </div>

    <!-- GUILD TAB -->
    <div class="shop-section" id="tab-guild">
      <div id="guild-ui-container">
        <div class="lb-loading">Loading guild data...</div>
      </div>
    </div>

    <!-- SPECIAL TAB -->
    <div class="shop-section" id="tab-special">

      <!-- ONE STRIKE -->
      <div class="one-strike-banner">
        <span class="big-icon">☠</span>
        <h3>ONE STRIKE</h3>
        <div class="one-strike-stats">
          🎯 25% chance to hit<br>
          💀 Instantly kills any enemy<br>
          💚 Costs 75% of your stamina<br>
          ❤️ Costs 50% of your max HP<br>
          ⚠ One use per match
        </div>
        <div id="one-strike-shop-status"></div>
        <button class="btn-buy btn-buy-red" id="btn-buy-one-strike" onclick="buyOneStrike()">UNLOCK FOR 150 XP</button>
      </div>

      <!-- POISON POTION -->
      <div class="potion-card" id="potion-poison-card">
        <div class="potion-header">
          <span class="potion-name">🧪 Venom of Spite</span>
          <span class="shop-price">1,500 XP</span>
        </div>
        <div class="potion-desc">Poison the enemy at the start of combat — they take <strong style="color:#a78bfa">10% of their max HP</strong> as poison damage every turn. <strong style="color:var(--white)">⚠ Active for the next round only</strong> — consumed on use.</div>
        <div id="potion-poison-status"></div>
        <button class="btn-buy btn-buy-red" id="btn-buy-poison" onclick="buyPotion('poison')" style="border-color:#7c3aed;color:#a78bfa;">BUY FOR 1,500 XP</button>
      </div>

      <!-- MISS POTION -->
      <div class="potion-card" id="potion-miss-card">
        <div class="potion-header">
          <span class="potion-name">🌫️ Veil of Shadows</span>
          <span class="shop-price">5,000 XP</span>
        </div>
        <div class="potion-desc">Shroud yourself in darkness — the enemy <strong style="color:#60a5fa">misses 25% of all attacks</strong> for the duration of the fight. <strong style="color:var(--white)">⚠ Active for the next round only</strong> — consumed on use.</div>
        <div id="potion-miss-status"></div>
        <button class="btn-buy btn-buy-blue" id="btn-buy-miss" onclick="buyPotion('miss')">BUY FOR 5,000 XP</button>
      </div>

    </div>
  </div><!-- /shop-panel -->

  <!-- ══ SECTION 3: SKILL TRAINING (bottom) ══ -->
  <div class="panel skills-panel">
    <div class="panel-title">⚡ Skill Training</div>

    <!-- Boss mode skill note -->
    <div class="skill-boss-info" id="skill-boss-info">
      ⚡ <strong style="color:var(--gold);">Boss Mode Active</strong> — Each level-up fires a bonus damage strike against the boss.
      Tap 25 times to level up. One Strike &amp; potions are disabled during boss fights.
    </div>

    <div class="skills-grid">
      <div class="skill-row" id="skill-stamina">
        <div class="skill-label">
          <span class="skill-name"><span class="skill-icon">💚</span> Stamina</span>
          <span class="skill-val" id="lv-stamina">LV 1</span>
        </div>
        <div class="skill-bar-bg"><div class="skill-bar-fill bar-stamina" id="bar-stamina" style="width:10%"></div></div>
        <div class="skill-xp-row">
          <span id="xp-stamina-label">0/25</span>
          <div class="xp-progress-bg"><div class="xp-progress-fill" id="xp-stamina-bar" style="width:0%"></div></div>
          <span>Next LV</span>
        </div>
        <button class="btn-levelup" id="btn-stamina" onclick="clickSkill('stamina')">
          LEVEL UP <span id="clicks-stamina"></span>
          <div class="progress-bar" id="cd-bar-stamina" style="width:0%"></div>
        </button>
        <div class="cooldown-text" id="cd-text-stamina"></div>
      </div>

      <!-- Attack -->
      <div class="skill-row" id="skill-attack">
        <div class="skill-label">
          <span class="skill-name"><span class="skill-icon">⚔️</span> Attack</span>
          <span class="skill-val" id="lv-attack">LV 1</span>
        </div>
        <div class="skill-bar-bg"><div class="skill-bar-fill bar-attack" id="bar-attack" style="width:10%"></div></div>
        <div class="skill-xp-row">
          <span id="xp-attack-label">0/25</span>
          <div class="xp-progress-bg"><div class="xp-progress-fill" id="xp-attack-bar" style="width:0%"></div></div>
          <span>Next LV</span>
        </div>
        <button class="btn-levelup" id="btn-attack" onclick="clickSkill('attack')">
          LEVEL UP <span id="clicks-attack"></span>
          <div class="progress-bar" id="cd-bar-attack" style="width:0%"></div>
        </button>
        <div class="cooldown-text" id="cd-text-attack"></div>
      </div>

      <!-- Defense -->
      <div class="skill-row" id="skill-defense">
        <div class="skill-label">
          <span class="skill-name"><span class="skill-icon">🛡️</span> Defense</span>
          <span class="skill-val" id="lv-defense">LV 1</span>
        </div>
        <div class="skill-bar-bg"><div class="skill-bar-fill bar-defense" id="bar-defense" style="width:10%"></div></div>
        <div class="skill-xp-row">
          <span id="xp-defense-label">0/25</span>
          <div class="xp-progress-bg"><div class="xp-progress-fill" id="xp-defense-bar" style="width:0%"></div></div>
          <span>Next LV</span>
        </div>
        <button class="btn-levelup" id="btn-defense" onclick="clickSkill('defense')">
          LEVEL UP <span id="clicks-defense"></span>
          <div class="progress-bar" id="cd-bar-defense" style="width:0%"></div>
        </button>
        <div class="cooldown-text" id="cd-text-defense"></div>
      </div>

      <!-- Crit -->
      <div class="skill-row" id="skill-crit">
        <div class="skill-label">
          <span class="skill-name"><span class="skill-icon">💥</span> Critical</span>
          <span class="skill-val" id="lv-crit">LV 1</span>
        </div>
        <div class="skill-bar-bg"><div class="skill-bar-fill bar-crit" id="bar-crit" style="width:10%"></div></div>
        <div class="skill-xp-row">
          <span id="xp-crit-label">0/25</span>
          <div class="xp-progress-bg"><div class="xp-progress-fill" id="xp-crit-bar" style="width:0%"></div></div>
          <span>Next LV</span>
        </div>
        <button class="btn-levelup" id="btn-crit" onclick="clickSkill('crit')">
          LEVEL UP <span id="clicks-crit"></span>
          <div class="progress-bar" id="cd-bar-crit" style="width:0%"></div>
        </button>
        <div class="cooldown-text" id="cd-text-crit"></div>
      </div>
    </div><!-- /skills-grid -->

    <div id="skill-clicks-info">Click 25 times to level up — 1 min cooldown after each level</div>
  </div><!-- /skills-panel -->

</div><!-- /game-container -->

<!-- XP Reward Popup -->
<div class="overlay hidden" id="overlay" onclick="closePopup()"></div>
<div class="xp-popup hidden" id="xp-popup">
  <h2>⚔ VICTORY ⚔</h2>
  <p id="popup-enemy-name">Enemy Defeated!</p>
  <div class="xp-breakdown" id="xp-breakdown"></div>
  <button class="main-btn btn-next" onclick="closePopup()" style="margin-top:0.5rem; width:auto; padding:0.5rem 2rem;">Claim Rewards</button>
</div>


<script>
// ============================================================
// GAME STATE
// ============================================================
const SKILLS = ['stamina','attack','defense','crit'];
const CLICKS_PER_LEVEL = 25;
const COOLDOWN_MS = 60 * 1000;

const state = {
  skills: { stamina:1, attack:1, defense:1, crit:1 },
  skillClicks: { stamina:0, attack:0, defense:0, crit:0 },
  cdTimers: { stamina:null, attack:null, defense:null, crit:null },
  totalXP: 0,
  lifetimeXP: 0,    // never decremented — only goes up
  enemiesDefeated: 0,
  wave: 1,
  fight: {
    active: false,
    playerTurn: true,
    playerHP: 100,
    playerMaxHP: 100,
    playerStamina: 100,
    playerMaxStamina: 100,
    blocking: false,
    enemyMaxHP: 0,
    turn: 0,
    won: false,
  }
};

// ============================================================
// ENEMY DEFINITIONS
// ============================================================
const ENEMY_TIERS = [
  { name:'WILD WOLF',    sprite:'🐺', title:'A hungry beast of the wilds',            tier:'COMMON',    tierClass:'tier-common',    baseHP:30,  baseAtk:4,  baseDef:1,  xpBase:8,   wave:1  },
  { name:'GOBLIN SCOUT', sprite:'👺', title:'A sneaky creature from the shadows',      tier:'COMMON',    tierClass:'tier-common',    baseHP:40,  baseAtk:5,  baseDef:2,  xpBase:10,  wave:1  },
  { name:'ORC WARRIOR',  sprite:'👹', title:'A brutish orc with crude weapons',        tier:'UNCOMMON',  tierClass:'tier-uncommon',  baseHP:70,  baseAtk:9,  baseDef:4,  xpBase:20,  wave:4  },
  { name:'DARK KNIGHT',  sprite:'🗡️', title:'A fallen knight consumed by darkness',    tier:'RARE',      tierClass:'tier-rare',      baseHP:100, baseAtk:13, baseDef:7,  xpBase:35,  wave:6  },
  { name:'STONE GOLEM',  sprite:'🪨', title:'Ancient earth magic given form',          tier:'RARE',      tierClass:'tier-rare',      baseHP:140, baseAtk:11, baseDef:12, xpBase:45,  wave:8  },
  { name:'SHADOW DRAGON',sprite:'🐉', title:'A dragon born of void and nightmares',    tier:'EPIC',      tierClass:'tier-epic',      baseHP:180, baseAtk:18, baseDef:10, xpBase:70,  wave:10 },
  { name:'ARCANE LICH',  sprite:'💀', title:'An immortal sorcerer of death magic',     tier:'EPIC',      tierClass:'tier-epic',      baseHP:200, baseAtk:22, baseDef:14, xpBase:90,  wave:12 },
  { name:'DEMON LORD',   sprite:'😈', title:'A prince of the infernal planes',         tier:'LEGENDARY', tierClass:'tier-legendary', baseHP:280, baseAtk:28, baseDef:18, xpBase:130, wave:15 },
  { name:'CHAOS TITAN',  sprite:'🔥', title:'An entity of pure destructive force',     tier:'LEGENDARY', tierClass:'tier-legendary', baseHP:360, baseAtk:36, baseDef:22, xpBase:180, wave:18 },
  { name:'VOID EMPEROR',    sprite:'🌑', title:'The sovereign of nothingness',                   tier:'MYTHIC',    tierClass:'tier-mythic',    baseHP:500,  baseAtk:50,  baseDef:30,  xpBase:300, wave:22 },

  // ── 7 NEW LEGENDARY & MYTHIC ENEMIES ──
  { name:'SOUL REAPER',      sprite:'💀', title:'Death given hunger — it feeds on the fallen',     tier:'LEGENDARY', tierClass:'tier-legendary', baseHP:420,  baseAtk:42,  baseDef:24,  xpBase:220, wave:20 },
  { name:'ABYSSAL KRAKEN',   sprite:'🐙', title:'An ocean of darkness wrapped in tentacles',       tier:'LEGENDARY', tierClass:'tier-legendary', baseHP:480,  baseAtk:38,  baseDef:28,  xpBase:240, wave:22 },
  { name:'INFERNO PHOENIX',  sprite:'🔥', title:'Reborn from ash — it cannot truly die',           tier:'LEGENDARY', tierClass:'tier-legendary', baseHP:390,  baseAtk:48,  baseDef:20,  xpBase:230, wave:24 },
  { name:'STORM COLOSSUS',   sprite:'⛈️', title:'A titan born of lightning and ruin',              tier:'LEGENDARY', tierClass:'tier-legendary', baseHP:520,  baseAtk:44,  baseDef:26,  xpBase:250, wave:26 },
  { name:'ECLIPSE WRAITH',   sprite:'🌘', title:'A being between worlds — neither alive nor dead', tier:'MYTHIC',    tierClass:'tier-mythic',    baseHP:580,  baseAtk:58,  baseDef:32,  xpBase:340, wave:28 },
  { name:'BLOOD LEVIATHAN',  sprite:'🩸', title:'Ancient and vast — it bled the first age dry',   tier:'MYTHIC',    tierClass:'tier-mythic',    baseHP:680,  baseAtk:65,  baseDef:36,  xpBase:400, wave:32 },
  { name:'ETERNAL DEVOURER', sprite:'🕳️', title:'The end of all things, given form and appetite',  tier:'MYTHIC',    tierClass:'tier-mythic',    baseHP:800,  baseAtk:75,  baseDef:42,  xpBase:500, wave:36 },

  // ── TITAN TIER — Beyond Mythic ──
  { name:'WORLD BREAKER',    sprite:'🌍', title:'It shattered a world once — yours is next',         tier:'TITAN', tierClass:'tier-titan', baseHP:900,  baseAtk:70,  baseDef:55,  xpBase:700, wave:60 },
  { name:'ASTRAL EXECUTIONER',sprite:'⚡',title:'Summoned from the space between stars to destroy',   tier:'TITAN', tierClass:'tier-titan', baseHP:1000, baseAtk:78,  baseDef:60,  xpBase:850, wave:65 },
  { name:'UNDYING BEHEMOTH',  sprite:'🦣', title:'Killed ten thousand times — it keeps returning',    tier:'TITAN', tierClass:'tier-titan', baseHP:1200, baseAtk:75,  baseDef:80,  xpBase:1000,wave:70 },
  { name:'VOID ARCHITECT',    sprite:'🌀', title:'The intelligence that designed the abyss itself',   tier:'TITAN', tierClass:'tier-titan', baseHP:1100, baseAtk:90,  baseDef:65,  xpBase:1100,wave:75 },
  { name:'THE FIRST EVIL',    sprite:'👁️', title:'Older than gods — it was there before the light',  tier:'TITAN', tierClass:'tier-titan', baseHP:1500, baseAtk:100, baseDef:90,  xpBase:1500,wave:80 },
];

function getEnemyForWave(wave) {
  let eligible = ENEMY_TIERS.filter(e => e.wave <= wave);
  // After wave 35, only Legendary and Mythic enemies appear
  if (wave > 35) {
    const elites = eligible.filter(e => e.tier === 'LEGENDARY' || e.tier === 'MYTHIC' || e.tier === 'TITAN');
    if (elites.length) eligible = elites;
  }
  if (wave > 100) {
    const apex = eligible.filter(e => e.tier === 'MYTHIC' || e.tier === 'TITAN');
    if (apex.length) eligible = apex;
  }
  const weights = eligible.map((e, i) => Math.pow(i + 1, 1.5));
  const total = weights.reduce((a,b)=>a+b, 0);
  let r = Math.random() * total;
  for (let i = 0; i < eligible.length; i++) { r -= weights[i]; if (r <= 0) return eligible[i]; }
  return eligible[eligible.length - 1];
}

function scaleEnemy(base, wave) {
  const scale = 1 + (wave - 1) * 0.18;
  return { ...base,
    hp:       Math.round(base.baseHP  * scale),
    atk:      Math.round(base.baseAtk * scale * 0.875),  // -12.5% attack
    def:      Math.round(base.baseDef * scale * 0.90),   // -10% defense
    xpReward: Math.round(base.xpBase  * scale),
  };
}

// ============================================================
// PLAYER STATS
// ============================================================
function getPlayerMaxHP()      { return Math.round((60 + state.skills.defense * 8 + state.skills.stamina * 2) * 1.265); } // +26.5% total HP buff (15%+10%)
function getPlayerMaxStamina() { return 60 + state.skills.stamina * 5; }   // nerfed: was *10
function getPlayerAttack(type) {
  const base = Math.round((3 + state.skills.attack * 2) * 1.125); // +12.5%
  if (type === 'light') return Math.round(base * 1.25);
  if (type === 'heavy') return Math.round(base * 1.9);
  return 0;
}
function getPlayerDefense()    { return (1 + state.skills.defense * 1.5) * 1.125; } // +12.5%
function getCritChance()       { return Math.min(0.05 + (state.skills.crit - 1) * 0.035, 0.55); }
function getCritMultiplier()   { return 1.5 + (state.skills.crit - 1) * 0.05; }

// ============================================================
// SKILL SYSTEM
// ============================================================
function clickSkill(skill) {
  if (state.cdTimers[skill]) return;
  state.skillClicks[skill]++;
  if (state.skillClicks[skill] >= CLICKS_PER_LEVEL) {
    state.skills[skill]++;
    state.skillClicks[skill] = 0;
    addLog(`🌟 <span class="log-win">${skill.toUpperCase()} leveled up to ${state.skills[skill]}!</span>`, 'system');
    startCooldownTimer(skill);
    // ── Boss System: fire bonus damage on level-up ──
    if (typeof BossSystem !== 'undefined' && BossSystem.isActive()) {
      const skillNames = { stamina:'Stamina', attack:'Attack', defense:'Defense', crit:'Critical' };
      BossSystem.onSkillLevelUp(skillNames[skill] || skill, state.skills[skill]);
    }
    saveGame(); // auto-save on level-up
  }
  renderSkills();
}

function startCooldownTimer(skill) {
  if (state.cdTimers[skill]) clearInterval(state.cdTimers[skill]);
  const btn = document.getElementById('btn-' + skill);
  btn.disabled = true;
  const start = Date.now();
  const duration = COOLDOWN_MS;
  state.cdTimers[skill] = setInterval(() => {
    const remain = duration - (Date.now() - start);
    if (remain <= 0) {
      clearInterval(state.cdTimers[skill]);
      state.cdTimers[skill] = null;
      btn.disabled = false;
      document.getElementById('cd-bar-' + skill).style.width = '0%';
      document.getElementById('cd-text-' + skill).textContent = '';
      return;
    }
    document.getElementById('cd-bar-' + skill).style.width = ((duration - remain) / duration * 100).toFixed(1) + '%';
    document.getElementById('cd-text-' + skill).textContent = `⏳ ${Math.ceil(remain/1000)}s cooldown`;
  }, 100);
}

function renderSkills() {
  SKILLS.forEach(skill => {
    const lv = state.skills[skill];
    const clicks = state.skillClicks[skill];
    const pct = (clicks / CLICKS_PER_LEVEL * 100).toFixed(0);
    document.getElementById('lv-' + skill).textContent = `LV ${lv}`;
    document.getElementById('bar-' + skill).style.width = Math.min(lv * 9, 100) + '%';
    document.getElementById('xp-' + skill + '-label').textContent = `${clicks}/25`;
    document.getElementById('xp-' + skill + '-bar').style.width = pct + '%';
    document.getElementById('clicks-' + skill).textContent = clicks > 0 ? `(${clicks}/25)` : '';
  });
  const totalLv = SKILLS.reduce((s,k) => s + state.skills[k], 0);
  const titles = ['Novice Wanderer','Apprentice Fighter','Battle-Hardened Soldier','Veteran Warrior','Elite Champion','Master of Combat','Legendary Slayer','Godlike Destroyer'];
  document.getElementById('player-title').textContent = titles[Math.min(Math.floor(totalLv / 8), titles.length - 1)];
  // Only auto-change sprite if no skin is equipped (skin 0 is default Wanderer)
  if (!shopState || shopState.equippedSkin === 0) {
    const autoSprite = totalLv < 10 ? '🧙' : totalLv < 20 ? '⚔️' : totalLv < 35 ? '🥷' : '🦸';
    SKINS[0].emoji = autoSprite; // keep skin 0 in sync with level
    document.getElementById('player-sprite').textContent = autoSprite;
  } else {
    // Re-apply the equipped skin so renderSkills never clobbers it
    document.getElementById('player-sprite').textContent = SKINS[shopState.equippedSkin].emoji;
  }
}

// ============================================================
// FIGHT SYSTEM
// ============================================================
let currentEnemy = null;
let actionLocked = false;

function startFight() {
  const base = getEnemyForWave(state.wave);
  currentEnemy = scaleEnemy(base, state.wave);
  const maxHP = getPlayerMaxHP();
  const maxSta = getPlayerMaxStamina();
  const f = state.fight;
  f.playerHP = Math.floor(maxHP); f.playerMaxHP = Math.floor(maxHP);
  // Stormbringer always starts at full stamina
  f.playerStamina = maxSta; f.playerMaxStamina = maxSta;
  f.blocking = false;
  f.enemyMaxHP = currentEnemy.hp;
  f.active = true; f.playerTurn = true; f.turn = 1; f.won = false;
  actionLocked = false;
  shopState.oneStrikeUsedThisMatch = false;
  activatePotions();

  document.getElementById('enemy-name').textContent = currentEnemy.name;
  document.getElementById('enemy-subtitle').textContent = currentEnemy.title;
  document.getElementById('enemy-sprite').textContent = currentEnemy.sprite;
  document.getElementById('enemy-tier-badge').textContent = currentEnemy.tier;
  document.getElementById('enemy-tier-badge').className = 'enemy-tier ' + currentEnemy.tierClass;
  document.getElementById('enemy-atk-txt').textContent = currentEnemy.atk;
  document.getElementById('enemy-def-txt').textContent = currentEnemy.def;
  document.getElementById('btn-fight').classList.add('hidden');
  document.getElementById('btn-next').classList.add('hidden');
  document.getElementById('btn-respawn').classList.add('hidden');
  setActionButtons(true);
  setStatus('fight', '⚔ FIGHTING');
  updateBars();
  updateOneStrikeButton();
  addLog(`--- Turn ${f.turn} ---`, 'turn');
  addLog(`⚔ ${currentEnemy.name} appears! HP:${currentEnemy.hp} ATK:${currentEnemy.atk} DEF:${currentEnemy.def}`, 'system');
  setTurnIndicator('player');
}

function playerAction(type) {
  if (!state.fight.active || !state.fight.playerTurn || actionLocked) return;

  if (type === 'onestrike') {
    if (!shopState.oneStrikeUnlocked || shopState.oneStrikeUsedThisMatch) return;
    actionLocked = true;
    setActionButtons(false);
    state.fight.playerTurn = false;
    shopState.oneStrikeUsedThisMatch = true;
    const f = state.fight;
    f.playerStamina = Math.max(0, f.playerStamina - Math.floor(f.playerMaxStamina * 0.75));
    f.playerHP = Math.max(1, f.playerHP - Math.floor(f.playerMaxHP * 0.5));
    const flash = document.createElement('div'); flash.className = 'os-flash'; document.body.appendChild(flash);
    setTimeout(() => flash.remove(), 900);
    addLog(`☠ <span class="log-death">ONE STRIKE unleashed!</span>`, 'death');
    const hit = Math.random() < 0.25;
    setTimeout(() => {
      if (hit) {
        addLog(`💀 <span class="log-crit">ONE STRIKE CONNECTS! ${currentEnemy.name} is OBLITERATED!</span>`, 'crit');
        showDamage('enemy-card', '☠ INSTANT KILL', 'damage-crit');
        animateCard('player-card', 'attack-onestrike', 800);
        setTimeout(() => {
          animateCard('enemy-card', 'taking-hit', 500);
          animateCard('enemy-card', 'impact-os', 700);
        }, 500);
        currentEnemy.hp = 0; updateBars(); setTimeout(playerWins, 900);
      } else {
        addLog(`💨 ONE STRIKE MISSED... energy fades.`, 'miss');
        showDamage('enemy-card', 'MISS', 'damage-miss');
        updateBars();
        f.playerHP <= 1 ? setTimeout(playerDies, 700) : setTimeout(enemyTurn, 1000);
      }
    }, 700);
    return;
  }

  const f = state.fight;
  const stamCost = type === 'block' ? 5 : type === 'light' ? 10 : 25;
  if (f.playerStamina < stamCost) { addLog(`😤 Not enough stamina!`, 'player'); return; }
  actionLocked = true;
  setActionButtons(false);
  f.playerTurn = false;
  f.blocking = false;
  f.playerStamina = Math.max(0, f.playerStamina - stamCost);

  if (type === 'block') {
    f.blocking = true;
    addLog(`🛡️ You raise your shield!`, 'block');
    showDamage('player-card', 'BLOCK!', 'damage-block');
    animateCard('player-card', 'attack-block', 550);
    updateBars();
    setTimeout(enemyTurn, 700);
  } else {
    const rawAtk = getPlayerAttack(type);
    // Heavy attack only connects 75% of the time
    const heavyMiss = type === 'heavy' && Math.random() > 0.75;
    if (heavyMiss) {
      addLog(`💀 Heavy — <span class="log-miss">MISSED! The blow swings wide!</span>`, 'player');
      showDamage('enemy-card', 'MISS!', 'damage-miss');
      animateCard('player-card', 'attack-heavy', 950);
      updateBars();
      if (currentEnemy.hp <= 0) { setTimeout(playerWins, 800); return; }
      setTimeout(enemyTurn, 1000);
      return;
    }
    const isCrit = Math.random() < getCritChance();
    let dmg = Math.max(1, Math.round(rawAtk - currentEnemy.def * 0.4));
    if (isCrit) dmg = Math.round(dmg * getCritMultiplier());
    currentEnemy.hp = Math.max(0, currentEnemy.hp - dmg);
    const label = type === 'light' ? '⚡ Light' : '💀 Heavy';
    addLog(isCrit
      ? `${label} — <span class="log-crit">CRITICAL HIT! ${dmg} damage!</span>`
      : `${label} — ${dmg} damage to ${currentEnemy.name}.`, 'player');
    showDamage('enemy-card', isCrit ? `💥 ${dmg}!` : `-${dmg}`, isCrit ? 'damage-crit' : 'damage-normal');
    const atkAnim = type === 'light' ? 'attack-light' : 'attack-heavy';
    const impactAnim = type === 'light' ? 'impact' : 'impact-heavy';
    const atkDur = type === 'heavy' ? 950 : 650;
    animateCard('player-card', atkAnim, atkDur);
    const hitDelay = type === 'heavy' ? 420 : 280;
    setTimeout(() => {
      animateCard('enemy-card', 'taking-hit', 500);
      animateCard('enemy-card', impactAnim, 650);
      if (type === 'heavy') {
        document.body.classList.add('screen-shake');
        setTimeout(() => document.body.classList.remove('screen-shake'), 500);
      }
    }, hitDelay);
    updateBars();
    if (currentEnemy.hp <= 0) { setTimeout(playerWins, 800); return; }
    setTimeout(enemyTurn, 1000);
  }
}

function enemyTurn() {
  const f = state.fight;
  if (!f.active) return;
  setTurnIndicator('enemy');
  addLog(`--- Enemy Turn ---`, 'turn');
  // Veil of Shadows: 25% miss chance
  if (potionState.miss.activeThisFight && Math.random() < 0.25) {
    addLog(`🌫️ <span style="color:#60a5fa">The enemy's attack misses! (Veil of Shadows)</span>`, 'block');
    showDamage('player-card', 'MISS!', 'damage-miss');
    f.playerStamina = Math.min(f.playerMaxStamina, f.playerStamina + 9 + Math.floor(state.skills.stamina * 1.25));
    updateBars();
    f.turn++; f.playerTurn = true; actionLocked = false; f.blocking = false;
    setTimeout(() => { addLog(`--- Turn ${f.turn} ---`, 'turn'); setTurnIndicator('player'); setActionButtons(true); }, 600);
    return;
  }
  let dmg = Math.max(1, Math.round(currentEnemy.atk - getPlayerDefense() * 0.5));
  let enemyCrit = false;
  if (f.blocking) {
    const blockFactor = 0.15 + (state.skills.defense * 0.03);
    dmg = Math.max(1, Math.round(dmg * (1 - Math.min(blockFactor * 3, 0.75))));
    addLog(`🛡️ <span class="log-block">Blocked! ${currentEnemy.name} deals only ${dmg}.</span>`, 'block');
    showDamage('player-card', `-${dmg}`, 'damage-block');
  } else {
    enemyCrit = Math.random() < 0.12;
    if (enemyCrit) dmg = Math.round(dmg * 1.5);
    addLog(enemyCrit
      ? `💜 <span class="log-enemy">${currentEnemy.name} lands a CRUSHING BLOW! ${dmg} damage!</span>`
      : `💜 <span class="log-enemy">${currentEnemy.name} attacks for ${dmg}.</span>`, 'enemy');
    showDamage('player-card', `-${dmg}`, 'damage-enemy');
  }
  f.playerHP = Math.floor(Math.max(0, f.playerHP - dmg));
  const monsterAnim = enemyCrit ? 'attack-monster-crit' : 'attack-monster';
  animateCard('enemy-card', monsterAnim, 750);
  setTimeout(() => {
    animateCard('player-card', 'taking-hit', 500);
    if (enemyCrit) animateCard('player-card', 'impact-heavy', 600);
    else animateCard('player-card', 'impact', 550);
  }, 420);
  f.playerStamina = Math.min(f.playerMaxStamina, f.playerStamina + 9 + Math.floor(state.skills.stamina * 1.25));
  // Venom of Spite: 10% of enemy max HP as poison damage per turn
  if (potionState.poison.activeThisFight) {
    const poisonDmg = Math.max(1, Math.round(f.enemyMaxHP * 0.10));
    currentEnemy.hp = Math.max(0, currentEnemy.hp - poisonDmg);
    addLog(`🧪 <span style="color:#a78bfa">Poison deals ${poisonDmg} damage to ${currentEnemy.name}! (${currentEnemy.hp} HP left)</span>`, 'system');
    if (currentEnemy.hp <= 0) { updateBars(); setTimeout(playerWins, 500); return; }
  }
  updateBars();
  if (f.playerHP <= 0) { setTimeout(playerDies, 700); return; }
  f.turn++; f.playerTurn = true; actionLocked = false; f.blocking = false;
  setTimeout(() => {
    addLog(`--- Turn ${f.turn} ---`, 'turn');
    setTurnIndicator('player');
    setActionButtons(true);
  }, 600);
}

function playerWins() {
  const f = state.fight;
  f.active = false; f.won = true; actionLocked = false;
  state.enemiesDefeated++;
  // -10% penalty after wave 25; bonus multiplier kicks in after wave 40
  const xpPenalty = state.wave > 25;
  let xpMultiplier = xpPenalty ? 0.9 : 1.0;
  if      (state.wave > 80) xpMultiplier *= 2.0;
  else if (state.wave > 60) xpMultiplier *= 1.6;
  else if (state.wave > 40) xpMultiplier *= 1.3;
  const xpGain = Math.round(currentEnemy.xpReward * xpMultiplier);
  state.totalXP    += xpGain;
  state.lifetimeXP += xpGain;  // lifetime never decremented
  const penaltyTag = xpPenalty && state.wave <= 40 ? ' <span style="color:var(--text3);font-size:0.85em;">(-10%)</span>' : '';
  const bonusTag   = state.wave > 40 ? ` <span style="color:var(--green2);font-size:0.85em;">(x${xpMultiplier.toFixed(2)} bonus)</span>` : '';
  addLog(`☠ <span class="log-death">${currentEnemy.name} slain!</span>`, 'death');
  addLog(`<span class="log-win">🏆 VICTORY! +${xpGain} XP!${penaltyTag}${bonusTag}</span>`, 'win');
  setStatus('won', '✓ VICTORY');
  setActionButtons(false);
  setTurnIndicator('none');
  document.getElementById('btn-one-strike').classList.add('hidden');
  updatePotionFightButtons();
  const perSkill = Math.floor(xpGain / 4);
  showVictoryPopup({stamina:perSkill,attack:perSkill,defense:perSkill,crit:xpGain-perSkill*3}, xpGain);
  state.wave++;
  document.getElementById('btn-next').classList.remove('hidden');
  document.getElementById('wave-num').textContent = state.wave;
  document.getElementById('enemy-count').textContent = state.enemiesDefeated;
  updateXPDisplay();
  saveGame(); // auto-save on win
}

function playerDies() {
  const f = state.fight;
  f.active = false; actionLocked = false;
  addLog(`💀 <span class="log-death">YOU HAVE BEEN DEFEATED...</span>`, 'death');
  setStatus('dead', '💀 DEFEATED');
  setActionButtons(false);
  setTurnIndicator('none');
  document.getElementById('btn-one-strike').classList.add('hidden');
  updatePotionFightButtons();
  document.getElementById('btn-respawn').classList.remove('hidden');
}

function resetPlayerStats() {
  const f = state.fight;
  const maxHP  = getPlayerMaxHP();
  const maxSta = getPlayerMaxStamina();
  f.playerHP           = Math.floor(maxHP);
  f.playerMaxHP        = maxHP;
  f.playerStamina      = maxSta;
  f.playerMaxStamina   = maxSta;
  updateBars();
}

function nextEnemy() {
  document.getElementById('btn-next').classList.add('hidden');
  document.getElementById('btn-fight').classList.remove('hidden');
  document.getElementById('btn-fight').textContent = `⚔ CHALLENGE WAVE ${state.wave}`;
  document.getElementById('wave-num').textContent = state.wave;
  setStatus('idle', '⚙ IDLE');
  setTurnIndicator('none');
  resetPlayerStats();
}

function respawn() {
  const penalty = Math.floor(state.totalXP * 0.1);
  state.totalXP = Math.max(0, state.totalXP - penalty);
  addLog(`♻ Respawned. Lost ${penalty} XP.`, 'system');
  document.getElementById('btn-respawn').classList.add('hidden');
  document.getElementById('btn-fight').classList.remove('hidden');
  document.getElementById('btn-fight').textContent = `⚔ CHALLENGE WAVE ${state.wave} (REVENGE)`;
  setStatus('idle', '⚙ IDLE');
  resetPlayerStats();
  updateXPDisplay();
}

// ============================================================
// UI HELPERS
// ============================================================
function updateBars() {
  const f = state.fight;
  const hpPct = Math.max(0, f.playerHP / f.playerMaxHP * 100);
  document.getElementById('player-hp-bar').style.width = hpPct + '%';
  document.getElementById('player-hp-txt').textContent = `${Math.floor(Math.max(0,f.playerHP))} / ${Math.floor(f.playerMaxHP)}`;
  const staPct = Math.max(0, f.playerStamina / f.playerMaxStamina * 100);
  document.getElementById('player-sta-bar').style.width = staPct + '%';
  document.getElementById('player-sta-txt').textContent = `${Math.round(f.playerStamina)} / ${f.playerMaxStamina}`;
  if (currentEnemy) {
    const ePct = Math.max(0, currentEnemy.hp / f.enemyMaxHP * 100);
    document.getElementById('enemy-hp-bar').style.width = ePct + '%';
    document.getElementById('enemy-hp-txt').textContent = `${Math.max(0,currentEnemy.hp)} / ${f.enemyMaxHP}`;
  }
}

function setActionButtons(enabled) {
  ['block','light','heavy'].forEach(t => {
    document.getElementById('btn-action-' + t).disabled = !enabled;
  });
  if (typeof shopState !== 'undefined') updateOneStrikeButton();
  if (typeof potionState !== 'undefined') updatePotionFightButtons();
  if (!enabled) { const b = document.getElementById('btn-one-strike'); if(b) b.disabled = true; }
}

function setStatus(type, text) {
  const el = document.getElementById('status-badge');
  el.className = 'status-badge badge-' + type;
  el.textContent = text;
}

function setTurnIndicator(who) {
  const el = document.getElementById('turn-indicator');
  el.className = 'turn-indicator';
  if (who === 'player') { el.className += ' player-turn'; el.textContent = '⚔ YOUR TURN'; }
  else if (who === 'enemy') { el.className += ' enemy-turn'; el.textContent = `💜 ${currentEnemy ? currentEnemy.name : 'ENEMY'}'S TURN`; }
  else { el.textContent = '— Prepare for Battle —'; }
}

function addLog(html, type) {
  const log = document.getElementById('battle-log');
  const div = document.createElement('div');
  div.className = 'log-entry log-' + type;
  div.innerHTML = html;
  log.appendChild(div);
  log.scrollTop = log.scrollHeight;
}

function showDamage(cardId, text, cls) {
  const card = document.getElementById(cardId);
  const el = document.createElement('div');
  el.className = `damage-float ${cls}`;
  el.textContent = text;
  card.style.position = 'relative';
  card.appendChild(el);
  setTimeout(() => el.remove(), 1200);
}

function animateCard(cardId, animClass, duration) {
  const el = document.getElementById(cardId);
  // Remove any existing animation classes first
  const allAnim = ['attacking','attack-light','attack-heavy','attack-onestrike',
                   'attack-block','attack-monster','attack-monster-crit',
                   'taking-hit','impact','impact-heavy','impact-os'];
  allAnim.forEach(c => el.classList.remove(c));
  // Force reflow so re-adding the class re-triggers the animation
  void el.offsetWidth;
  el.classList.add(animClass);
  setTimeout(() => el.classList.remove(animClass), duration || 750);
}

function updateXPDisplay() {
  const el = document.getElementById('xp-total-display');
  if (el) el.textContent = state.totalXP;
  const shop = document.getElementById('shop-xp-display');
  if (shop) shop.textContent = state.totalXP;
  // Refresh skills tab buttons immediately so afford-ability updates live
  const skillsTab = document.getElementById('tab-skills');
  if (skillsTab && skillsTab.classList.contains('active')) renderXPSkillList();
  // Refresh special tab if active
  const specialTab = document.getElementById('tab-special');
  if (specialTab && specialTab.classList.contains('active')) renderOneStrikeShop();
}

function showVictoryPopup(xpDist, total) {
  document.getElementById('overlay').classList.remove('hidden');
  document.getElementById('xp-popup').classList.remove('hidden');
  document.getElementById('popup-enemy-name').textContent = `${currentEnemy.name} — Wave ${state.wave - 1}`;
  document.getElementById('xp-breakdown').innerHTML = `
    <div class="xp-item"><span>💚 Stamina XP</span><span>+${xpDist.stamina}</span></div>
    <div class="xp-item"><span>⚔️ Attack XP</span><span>+${xpDist.attack}</span></div>
    <div class="xp-item"><span>🛡️ Defense XP</span><span>+${xpDist.defense}</span></div>
    <div class="xp-item"><span>💥 Critical XP</span><span>+${xpDist.crit}</span></div>
    <div class="xp-item" style="border-top:1px solid var(--gold3);margin-top:0.3rem;color:var(--gold);">
      <span>Total XP</span><span style="color:var(--gold2)">+${total}</span>
    </div>`;
}

function closePopup() {
  document.getElementById('overlay').classList.add('hidden');
  document.getElementById('xp-popup').classList.add('hidden');
}

function spawnParticles(targetId, color) {
  const el = document.getElementById(targetId);
  if (!el) return;
  for (let i = 0; i < 6; i++) {
    const p = document.createElement('div');
    p.className = 'particle';
    p.style.cssText = `width:${4+Math.random()*5}px;height:${4+Math.random()*5}px;background:${color};left:${Math.random()*100}%;top:50%;`;
    p.style.setProperty('--particle-end', `translate(${(Math.random()-.5)*80}px,${-(20+Math.random()*60)}px)`);
    el.parentElement.style.position = 'relative';
    el.parentElement.appendChild(p);
    setTimeout(() => p.remove(), 1000);
  }
}

// ============================================================
// SHOP STATE
// ============================================================
const shopState = {
  heroName: '',
  nameChangeCount: 0,
  equippedSkin: 0,
  ownedSkins: [0],
  oneStrikeUnlocked: false,
  oneStrikeUsedThisMatch: false,
  walletAddress: '',
};

const SKINS = [
  { emoji:'🧙', name:'Wanderer',            price:0      },
  { emoji:'🗡️', name:'Iron Duelist',        price:500    },
  { emoji:'🥷',  name:'Void Assassin',       price:1200   },
  { emoji:'🧝', name:'Thornwood Sentinel',  price:2500   },
  { emoji:'🦅', name:'Skywarden',           price:5000   },
  { emoji:'🧟', name:'Risen Warlord',       price:8500   },
  { emoji:'🐉', name:'Dragonbound',         price:15000  },
  { emoji:'💀', name:'Death Incarnate',     price:25000  },
  { emoji:'⚡', name:'Storm Herald',        price:40000  },
  { emoji:'🌑', name:'Eclipse Sovereign',   price:60000  },
  { emoji:'🔱', name:'Abyssal Titan',       price:90000  },
  { emoji:'👁️', name:'The Undying One',     price:150000 },
];

// Base XP costs — scale up every 5 waves
const XP_SKILL_COST_BASE = { stamina:15, attack:15, defense:15, crit:20 };

function getSkillXPCost(skill) {
  const tier       = Math.floor((state.wave - 1) / 5);  // +50% every 5 waves
  const waveMult   = 1 + tier * 0.5;
  const skillLv    = state.skills[skill];
  // Gentler scaling for early levels (1-15), steeper after
  const lvMult     = skillLv <= 15
    ? 1 + (skillLv - 1) * 0.06    // +6% per level up to LV 15
    : 1 + 14 * 0.06 + (skillLv - 15) * 0.175; // steeper after LV 15
  return Math.round(XP_SKILL_COST_BASE[skill] * waveMult * lvMult);
}

function switchShopTab(tab) {
  document.querySelectorAll('.shop-tab').forEach((t,i) => {
    t.classList.toggle('active', ['name','skins','skills','guild','special'][i] === tab);
  });
  document.querySelectorAll('.shop-section').forEach(s => s.classList.remove('active'));
  document.getElementById('tab-' + tab).classList.add('active');
  if (tab === 'skins')   renderSkinGrid();
  if (tab === 'skills')  renderXPSkillList();
  if (tab === 'special') renderOneStrikeShop();
  if (tab === 'guild')   renderGuildUI();
  if (tab === 'name')    renderWalletDisplay();
}

function setHeroName() {
  const input = document.getElementById('hero-name-input');
  const name = input.value.trim();
  if (!name) return;
  const cost = shopState.nameChangeCount === 0 ? 0 : 10;
  if (state.totalXP < cost) { addLog(`💸 Need ${cost} XP to rename.`, 'system'); return; }
  state.totalXP -= cost;
  shopState.heroName = name;
  shopState.nameChangeCount++;
  document.querySelector('.combatant-name').textContent = name.toUpperCase();
  document.getElementById('name-cost-label').textContent = shopState.nameChangeCount >= 1 ? 'Rename costs 10 XP' : 'First change is free';
  addLog(`🏷️ <span class="log-win">Warrior renamed to "${name}"!</span>`, 'system');
  input.value = '';
  updateXPDisplay();
  saveGame();
}

// wallet connection handled by PveAuth — see PVE STORAGE + AUTH block below

function renderWalletDisplay() {
  const linked  = document.getElementById('wallet-linked-banner');
  const area    = document.getElementById('wallet-connect-area');
  if (!linked) return;

  const profile = (typeof PveAuth !== 'undefined') ? PveAuth.getProfile() : null;
  const addr    = profile?.address || shopState.walletAddress || '';

  if (addr) {
    const short = addr.slice(0, 8) + '...' + addr.slice(-4);
    linked.classList.remove('hidden');
    linked.innerHTML = '✔ Wallet connected: <span style="font-family:monospace;font-size:0.8rem;opacity:0.85;">' + short + '</span>'
      + ' <button onclick="PveAuth._openModal()" style="background:none;border:none;color:var(--text3);cursor:pointer;font-size:0.75rem;margin-left:0.5rem;text-decoration:underline;">switch</button>';
    if (area) area.style.display = 'none';
  } else {
    linked.classList.add('hidden');
    if (area) area.style.display = '';
  }
}

// Block the fight button until wallet is saved
function updateFightButtonWalletGate() {
  // Fight button always enabled (no wallet required)
  const fightBtn = document.getElementById('btn-fight');
  if (!fightBtn) return;
  fightBtn.disabled = false;
  fightBtn.title = '';
  fightBtn.style.opacity = '';
  fightBtn.style.cursor  = '';
}

function renderSkinGrid() {
  const grid = document.getElementById('skin-grid');
  grid.innerHTML = '';
  SKINS.forEach((skin, i) => {
    const owned    = shopState.ownedSkins.includes(i);
    const equipped = shopState.equippedSkin === i;
    // Locked unless previous skin is owned (skin 0 always available)
    const prevOwned = i === 0 || shopState.ownedSkins.includes(i - 1);
    const locked   = !owned && !prevOwned;

    const card = document.createElement('div');
    card.className = 'skin-card'
      + (equipped ? ' equipped' : '')
      + (owned && !equipped ? ' owned-skin' : '')
      + (locked ? ' skin-locked' : '');

    let priceTag;
    if (equipped)      priceTag = '✦ EQUIPPED';
    else if (owned)    priceTag = '✓ OWNED — tap to equip';
    else if (locked)   priceTag = '🔒 Buy previous skin first';
    else               priceTag = skin.price.toLocaleString() + ' XP';

    card.innerHTML = `
      <span class="skin-emoji" style="opacity:${locked?'0.3':'1'}">${skin.emoji}</span>
      <span class="skin-name" style="opacity:${locked?'0.4':'1'}">${skin.name}</span>
      <span class="skin-price-tag" style="${locked?'color:var(--text3)':''}">${priceTag}</span>
    `;

    if (!locked) card.onclick = () => owned ? equipSkin(i) : buySkin(i);
    grid.appendChild(card);
  });
}

function buySkin(i) {
  const skin = SKINS[i];
  // Enforce sequential unlock
  if (i > 0 && !shopState.ownedSkins.includes(i - 1)) {
    addLog(`🔒 You must own ${SKINS[i-1].name} first.`, 'system'); return;
  }
  if (state.totalXP < skin.price) {
    addLog(`💸 Need ${skin.price.toLocaleString()} XP for ${skin.name}.`, 'system'); return;
  }
  state.totalXP -= skin.price;
  shopState.ownedSkins.push(i);
  equipSkin(i);
  addLog(`🎭 <span class="log-win">Unlocked ${skin.name} skin!</span>`, 'system');
  updateXPDisplay();
  saveGame();
}

function equipSkin(i) {
  // Can only equip owned skins
  if (!shopState.ownedSkins.includes(i)) return;
  shopState.equippedSkin = i;
  document.getElementById('player-sprite').textContent = SKINS[i].emoji;
  // Sync skin to shared profile
  const _p = PveAuth.getProfile();
  if (_p) PveAuth.saveProfile({ skin: SKINS[i].emoji });
  renderSkinGrid();
}

function renderXPSkillList() {
  const icons = { stamina:'💚', attack:'⚔️', defense:'🛡️', crit:'💥' };
  const tier = Math.floor((state.wave - 1) / 5);
  const tierLabel = tier > 0 ? ` <span style="color:var(--red2);font-size:0.7rem;">(Wave ${Math.min(state.wave,99)} pricing)</span>` : '';
  document.getElementById('xp-skill-list').innerHTML =
    `<div style="font-size:0.78rem;color:var(--text3);font-style:italic;margin-bottom:0.5rem;">
       Costs rise +50% every 5 waves${tierLabel}
     </div>` +
    SKILLS.map(skill => {
      const cost = getSkillXPCost(skill);
      const canAfford = state.totalXP >= cost;
      return `<div class="xp-level-row">
        <span class="xp-level-name">${icons[skill]} ${skill.charAt(0).toUpperCase()+skill.slice(1)} LV ${state.skills[skill]}</span>
        <span class="xp-level-cost">${cost} XP</span>
        <button class="btn-buy btn-buy-gold" style="width:auto;padding:0.25rem 0.7rem;margin-left:0.4rem;"
          onclick="buySkillLevel('${skill}')" ${canAfford?'':'disabled'}>+1 LV</button>
      </div>`;
    }).join('');
}

function buySkillLevel(skill) {
  const cost = getSkillXPCost(skill);
  if (state.totalXP < cost) return;
  state.totalXP -= cost;
  state.skills[skill]++;
  addLog(`💰 <span class="log-win">Spent ${cost} XP — ${skill.toUpperCase()} is now LV ${state.skills[skill]}!</span>`, 'system');
  renderSkills();
  renderXPSkillList();
  updateXPDisplay();
  saveGame();
}

function renderOneStrikeShop() {
  const statusEl = document.getElementById('one-strike-shop-status');
  const buyBtn = document.getElementById('btn-buy-one-strike');
  if (shopState.oneStrikeUnlocked) {
    statusEl.innerHTML = `<span class="owned-badge" style="display:inline-block;margin-bottom:0.4rem;">✓ UNLOCKED</span>`;
    buyBtn.disabled = true; buyBtn.textContent = 'ALREADY UNLOCKED';
  } else {
    statusEl.innerHTML = '';
    const canAfford = state.totalXP >= 150;
    buyBtn.disabled = !canAfford;
    buyBtn.textContent = canAfford ? 'UNLOCK FOR 150 XP' : `NEED 150 XP (have ${state.totalXP})`;
  }
  renderPotionShop();
}

function buyOneStrike() {
  if (shopState.oneStrikeUnlocked || state.totalXP < 150) return;
  state.totalXP -= 150;
  shopState.oneStrikeUnlocked = true;
  addLog(`☠ <span class="log-death">ONE STRIKE unlocked!</span>`, 'system');
  renderOneStrikeShop(); updateXPDisplay();
  saveGame();
}

function updateOneStrikeButton() {
  const btn = document.getElementById('btn-one-strike');
  if (!shopState.oneStrikeUnlocked || !state.fight.active) { btn.classList.add('hidden'); return; }
  btn.classList.remove('hidden');
  btn.disabled = shopState.oneStrikeUsedThisMatch;
  btn.classList.toggle('used', shopState.oneStrikeUsedThisMatch);
}

function updatePotionFightButtons() {
  const el = document.getElementById('potion-active-display');
  if (!el) return;

  const lines = [];
  if (potionState.poison.activeThisFight) {
    lines.push('<span style="color:#a78bfa;">🧪 Venom of Spite is active — enemy takes 10% max HP poison each turn</span>');
  } else if (potionState.poison.owned && state.fight.active) {
    lines.push('<span style="color:#7c3aed;opacity:0.7;">🧪 Venom of Spite in inventory — activates next round only</span>');
  }
  if (potionState.miss.activeThisFight) {
    lines.push('<span style="color:#60a5fa;">🌫️ Veil of Shadows is active — enemy has 25% miss chance</span>');
  } else if (potionState.miss.owned && state.fight.active) {
    lines.push('<span style="color:#2980b9;opacity:0.7;">🌫️ Veil of Shadows in inventory — activates next round only</span>');
  }

  if (lines.length > 0) {
    el.innerHTML = lines.join('');
    el.classList.remove('hidden');
    el.style.display = 'flex';
  } else {
    el.classList.add('hidden');
    el.style.display = 'none';
  }
}







// ============================================================
// GUILD SYSTEM
// ============================================================
const GUILD_PREFIX      = 'guild:';
const GUILD_UNLOCK_WAVE = 30;   // must have beaten wave 30 to create
const GUILD_CREATE_COST = 1000; // XP cost to create a guild
const GUILD_JOIN_WAVE   = 10;   // must have beaten wave 10 to join
const GUILD_MAX_MEMBERS = 5;

const guildState = {
  myGuildCode:  null,
  myGuildName:  null,
  isLeader:     false,
};

function generateGuildCode() {
  const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
  return Array.from({length:6}, () => chars[Math.floor(Math.random()*chars.length)]).join('');
}

function getHighestWaveBeaten() {
  return Math.max(0, state.wave - 1);
}

// Guild membership is persisted via saveGame() to Firebase — no localStorage needed
function saveGuildLocally() {
  // no-op: guild state is saved as part of the full saveGame() call
}

// ── Fetch a single guild record from shared storage ──
async function fetchGuild(code) {
  if (!window._fbReady) return null;
  try {
    const snap = await window._fbGet(window._fbRef(window._fbDb, 'guilds/' + code));
    return snap.exists() ? snap.val() : null;
  } catch { return null; }
}

async function saveGuild(code, data) {
  if (!window._fbReady) return;
  try {
    await window._fbSet(window._fbRef(window._fbDb, 'guilds/' + code), data);
  } catch(e) { console.warn('saveGuild failed:', e); }
}

async function fetchAllGuilds() {
  if (!window._fbReady) return [];
  try {
    const snap = await window._fbGet(window._fbRef(window._fbDb, 'guilds'));
    if (!snap.exists()) return [];
    return Object.values(snap.val());
  } catch { return []; }
}

// ── Create a guild (leader only, wave 30+) ──
async function createGuild() {
  const nameEl = document.getElementById('guild-create-name');
  const errEl  = document.getElementById('guild-create-error');
  const name = nameEl?.value.trim();
  if (!name) { if(errEl) errEl.textContent = 'Enter a guild name.'; return; }
  if (name.length < 3 || name.length > 24) { if(errEl) errEl.textContent = 'Name must be 3–24 characters.'; return; }
  if (getHighestWaveBeaten() < GUILD_UNLOCK_WAVE) {
    if(errEl) errEl.textContent = `You must reach wave ${GUILD_UNLOCK_WAVE} first.`; return;
  }
  if (state.totalXP < GUILD_CREATE_COST) {
    if(errEl) errEl.textContent = `Need ${GUILD_CREATE_COST} XP to found a guild (have ${state.totalXP}).`; return;
  }

  state.totalXP -= GUILD_CREATE_COST;
  updateXPDisplay();
  const code = generateGuildCode();
  const myKey = getLbPlayerKey().replace('lb:','');
  const guild = {
    name,
    code,
    leaderKey:   myKey,
    leaderName:  shopState.heroName || 'Anonymous',
    members:     [myKey],
    created:     Date.now(),
  };

  await saveGuild(code, guild);
  guildState.myGuildCode = code;
  guildState.myGuildName = name;
  guildState.isLeader    = true;
  saveGuildLocally();
  await submitLeaderboardScore();
  showSaveToast(`⚔ Guild "${name}" created! Code: ${code}`, '#c9a84c');
  renderGuildUI();
}

// ── Join a guild (wave 10+) ──
async function joinGuild() {
  const codeEl = document.getElementById('guild-join-code');
  const errEl  = document.getElementById('guild-join-error');
  const code   = codeEl?.value.trim().toUpperCase();
  if (!code || code.length !== 6) { if(errEl) errEl.textContent = 'Enter a valid 6-character code.'; return; }
  if (getHighestWaveBeaten() < GUILD_JOIN_WAVE) {
    if(errEl) errEl.textContent = `You must reach wave ${GUILD_JOIN_WAVE} first.`; return;
  }
  if (guildState.myGuildCode) { if(errEl) errEl.textContent = 'Leave your current guild first.'; return; }

  const guild = await fetchGuild(code);
  if (!guild) { if(errEl) errEl.textContent = 'Guild not found. Check the code.'; return; }
  if (guild.members.length >= GUILD_MAX_MEMBERS) { if(errEl) errEl.textContent = 'Guild is full (5 members max).'; return; }

  const myKey = getLbPlayerKey().replace('lb:','');
  if (!guild.members.includes(myKey)) guild.members.push(myKey);
  await saveGuild(code, guild);

  guildState.myGuildCode = code;
  guildState.myGuildName = guild.name;
  guildState.isLeader    = false;
  saveGuildLocally();
  await submitLeaderboardScore();
  showSaveToast(`⚔ Joined "${guild.name}"!`, '#2ecc71');
  renderGuildUI();
}

// ── Leave a guild ──
async function leaveGuild() {
  if (!guildState.myGuildCode) return;
  if (!confirm('Leave your guild? You will lose your leader status if applicable.')) return;

  const guild = await fetchGuild(guildState.myGuildCode);
  if (guild) {
    const myKey = getLbPlayerKey().replace('lb:','');
    guild.members = guild.members.filter(k => k !== myKey);
    // If leader leaves and members remain, transfer to first member
    if (guildState.isLeader && guild.members.length > 0) {
      guild.leaderKey = guild.members[0];
      guild.leaderName = 'New Leader';
    }
    if (guild.members.length === 0) {
      // Disband
      try {
        if (window._fbReady) await window._fbSet(window._fbRef(window._fbDb, 'guilds/' + guildState.myGuildCode), null);
      } catch{}
    } else {
      await saveGuild(guildState.myGuildCode, guild);
    }
  }

  guildState.myGuildCode = null;
  guildState.myGuildName = null;
  guildState.isLeader    = false;
  saveGuildLocally();
  await submitLeaderboardScore();
  renderGuildUI();
}

// ── Render guild UI in shop tab ──
async function renderGuildUI() {
  const container = document.getElementById('guild-ui-container');
  if (!container) return;
  container.innerHTML = '<div class="lb-loading">⏳ Loading guild data...</div>';

  const waveBeaten = getHighestWaveBeaten();
  const myKey = getLbPlayerKey().replace('lb:','');

  // ── Not in a guild ──
  if (!guildState.myGuildCode) {
    const canCreate = waveBeaten >= GUILD_UNLOCK_WAVE;
    const canJoin   = waveBeaten >= GUILD_JOIN_WAVE;
    const waveProgress = Math.min(waveBeaten / GUILD_UNLOCK_WAVE * 100, 100);

    container.innerHTML = `
      <div class="guild-section">
        ${!canCreate ? `
        <div class="guild-locked-msg">
          <span class="lock-icon">🏰</span>
          <p>Reach <strong>Wave ${GUILD_UNLOCK_WAVE}</strong> to found a Guild and become a Guild Leader.</p>
          <div class="guild-progress-bar-bg">
            <div class="guild-progress-bar-fill" style="width:${waveProgress}%"></div>
          </div>
          <div class="guild-progress-label">Wave ${waveBeaten} / ${GUILD_UNLOCK_WAVE} to unlock</div>
        </div>` : ''}

        ${canCreate ? `
        <div class="guild-panel">
          <div class="guild-panel-title">👑 Found a Guild — 1,000 XP</div>
          <div class="guild-input-row">
            <input class="guild-input" id="guild-create-name" maxlength="24" placeholder="Enter guild name..." />
            <button class="btn-guild btn-guild-gold" onclick="createGuild()">CREATE</button>
          </div>
          <div style="font-size:0.75rem;color:var(--text3);font-style:italic;margin-top:0.3rem;">
            Costs 1,000 XP · Wave ${GUILD_UNLOCK_WAVE}+ required · You have <span style="color:var(--gold)">${state.totalXP}</span> XP
          </div>
          <div class="guild-error" id="guild-create-error"></div>
        </div>` : ''}

        <div class="guild-panel">
          <div class="guild-panel-title">⚔ Join a Guild ${!canJoin ? `(Wave ${GUILD_JOIN_WAVE}+ Required)` : ''}</div>
          <div class="guild-input-row">
            <input class="guild-input" id="guild-join-code" maxlength="6" placeholder="6-char code..."
              style="letter-spacing:0.2em;" ${!canJoin ? 'disabled' : ''} />
            <button class="btn-guild btn-guild-blue" onclick="joinGuild()" ${!canJoin ? 'disabled' : ''}>JOIN</button>
          </div>
          <div class="guild-error" id="guild-join-error"></div>
          ${!canJoin ? `<div style="font-size:0.75rem;color:var(--text3);font-style:italic;margin-top:0.3rem;">Reach Wave ${GUILD_JOIN_WAVE} to join a guild (Wave ${waveBeaten} now)</div>` : ''}
        </div>
      </div>`;
    return;
  }

  // ── In a guild — fetch full data ──
  const guild = await fetchGuild(guildState.myGuildCode);
  if (!guild) {
    // Guild was disbanded
    guildState.myGuildCode = null; guildState.myGuildName = null; guildState.isLeader = false;
    saveGuildLocally();
    renderGuildUI(); return;
  }

  // Fetch all member scores from leaderboard
  const allEntries = await fetchLeaderboardEntries();
  const memberEntries = allEntries.filter(e => e.guildCode === guildState.myGuildCode);
  const avgWave = memberEntries.length
    ? (memberEntries.reduce((s,e) => s + (e.wave||0), 0) / memberEntries.length).toFixed(1)
    : '—';

  const memberRows = guild.members.slice(0, 20).map(mKey => {
    const entry = allEntries.find(e => e._key === 'lb:' + mKey);
    const isMe  = mKey === myKey;
    const isLeader = mKey === guild.leaderKey;
    return `<div class="guild-member-row">
      <span class="guild-member-skin">${entry?.skin || '🧙'}</span>
      ${isLeader ? '<span class="guild-member-leader-crown">👑</span>' : ''}
      <span class="guild-member-name ${isMe?'is-you':''}">${entry?.name || 'Unknown Warrior'}</span>
      <span class="guild-member-wave">Wave ${entry?.wave || '?'}</span>
    </div>`;
  }).join('');

  container.innerHTML = `
    <div class="guild-section">
      <div class="guild-panel">
        <div class="guild-name-display">${guild.name}</div>
        <div class="guild-code-display">
          Code: <span class="guild-code-badge" onclick="navigator.clipboard?.writeText('${guild.code}');showSaveToast('Code copied!','#c9a84c')" title="Click to copy">${guild.code}</span>
        </div>
        <div style="text-align:center;margin-bottom:0.6rem;">
          <span class="guild-role-badge ${guildState.isLeader?'role-leader':'role-member'}">
            ${guildState.isLeader ? '👑 Guild Leader' : '⚔ Member'}
          </span>
        </div>
        <div class="guild-panel-title">Members (${guild.members.length}/${GUILD_MAX_MEMBERS})</div>
        <div class="guild-members-list">${memberRows || '<div style="color:var(--text3);font-style:italic;font-size:0.8rem;">No score data yet — save your game!</div>'}</div>
        <div class="guild-avg-wave">Avg Wave Completion: <span>${avgWave}</span></div>
        <div style="margin-top:0.7rem;display:flex;justify-content:space-between;gap:0.5rem;flex-wrap:wrap;">
          <div style="flex:1;">
            <div style="font-family:'Cinzel',serif;font-size:0.72rem;color:var(--text3);margin-bottom:0.3rem;">Send XP to all guild members — any member can distribute</div>
            <div style="display:flex;gap:0.4rem;">
              <input class="guild-input" id="guild-xp-amount" type="number" min="1" placeholder="Amount each..." style="max-width:120px;"/>
              <button class="btn-guild btn-guild-gold" onclick="distributeGuildXP()">SEND</button>
            </div>
            <div class="guild-error" id="guild-xp-error"></div>
          </div>
          <div style="display:flex;align-items:flex-end;">
            <button class="btn-guild btn-guild-red" onclick="leaveGuild()">Leave Guild</button>
          </div>
        </div>
      </div>
    </div>`;
}

// ── Guild leaderboard tab ──
async function renderGuildLeaderboard() {
  const guilds = await fetchAllGuilds();
  const allEntries = await fetchLeaderboardEntries();

  if (!guilds.length) return '<div class="lb-empty">No guilds have been founded yet.<br><span style="font-size:0.78rem;color:var(--text3);">Reach Wave 30 to found the first guild!</span></div>';

  // Compute avg wave per guild
  const guildStats = guilds.map(g => {
    const members = allEntries.filter(e => e.guildCode === g.code);
    const avgWave = members.length
      ? members.reduce((s,e) => s + (e.wave||0), 0) / members.length
      : 0;
    return { ...g, memberCount: members.length, avgWave };
  }).sort((a,b) => b.avgWave - a.avgWave);

  const rankIcon = i => i===0?'🥇':i===1?'🥈':i===2?'🥉':`${i+1}`;

  const rows = guildStats.slice(0,20).map((g,i) => {
    const isMyGuild = g.code === guildState.myGuildCode;
    return `<tr class="${isMyGuild?'lb-you-row':''}">
      <td class="lb-rank ${i<3?'lb-rank-'+(i+1):''}">${rankIcon(i)}</td>
      <td class="lb-guild-name ${isMyGuild?'is-my-guild':''}">${g.name}</td>
      <td class="lb-guild-members">${g.memberCount} member${g.memberCount!==1?'s':''}</td>
      <td class="lb-guild-avg">Avg Wave ${g.avgWave.toFixed(1)}</td>
    </tr>`;
  }).join('');

  return `<table class="lb-table">
    <thead><tr>
      <th>#</th><th>Guild</th><th>Members</th><th style="text-align:right">Avg Wave</th>
    </tr></thead>
    <tbody>${rows}</tbody>
  </table>`;
}



// ============================================================
// POTION SYSTEM
// ============================================================
const potionState = {
  poison: { owned: false, activeThisFight: false },
  miss:   { owned: false, activeThisFight: false },
};

const POTION_BASE_COST = { poison: 1500, miss: 5000 };
function getPotionCost(type) {
  const base = POTION_BASE_COST[type];
  const wave = state.wave || 1;
  // Every 10 waves adds 20% to the base cost, capped at 5× base
  const mult = Math.min(5, 1 + Math.floor(wave / 10) * 0.2);
  return Math.round(base * mult / 100) * 100;
}

function buyPotion(type) {
  const cost = getPotionCost(type);
  if (state.totalXP < cost) {
    addLog(`💸 Need ${cost.toLocaleString()} XP for that potion.`, 'system');
    return;
  }
  if (potionState[type].owned) {
    addLog(`🧪 You already have that potion.`, 'system');
    return;
  }
  state.totalXP -= cost;
  potionState[type].owned = true;
  const names = { poison: 'Venom of Spite', miss: 'Veil of Shadows' };
  addLog(`🧪 <span class="log-win">${names[type]} purchased!</span>`, 'system');
  updateXPDisplay();
  renderOneStrikeShop(); // refresh whole special tab
  saveGame();
}

function activatePotions() {
  // Called at fight start — reset active flags
  potionState.poison.activeThisFight = false;
  potionState.miss.activeThisFight   = false;
  // Auto-activate if owned
  if (potionState.poison.owned) {
    potionState.poison.activeThisFight = true;
    potionState.poison.owned = false; // consumed on use
    addLog(`🧪 <span style="color:#a78bfa">Venom of Spite applied! Enemy takes 10% of their max HP as poison each turn. (This round only)</span>`, 'system');
  }
  if (potionState.miss.owned) {
    potionState.miss.activeThisFight = true;
    potionState.miss.owned = false;
    addLog(`🌫️ <span style="color:#60a5fa">Veil of Shadows active! Enemy has 25% miss chance. (This round only)</span>`, 'system');
  }
}

function renderPotionShop() {
  ['poison','miss'].forEach(type => {
    const statusEl = document.getElementById('potion-' + type + '-status');
    const buyBtn   = document.getElementById('btn-buy-' + type);
    if (!statusEl || !buyBtn) return;
    const cost     = getPotionCost(type);
    const owned    = potionState[type].owned;
    const canAfford = state.totalXP >= cost;
    if (owned) {
      statusEl.innerHTML = '<span class="potion-active-badge">✓ IN INVENTORY — activates next round only</span>';
      buyBtn.disabled = true;
      buyBtn.textContent = 'PURCHASED';
    } else {
      statusEl.innerHTML = '';
      buyBtn.disabled = !canAfford;
      buyBtn.textContent = canAfford
        ? `BUY FOR ${cost.toLocaleString()} XP`
        : `NEED ${cost.toLocaleString()} XP (have ${state.totalXP.toLocaleString()})`;
    }
  });
}



// ── Distribute XP to all guild members (leader only) ──
async function distributeGuildXP() {
  const errEl = document.getElementById('guild-xp-error');
  const input = document.getElementById('guild-xp-amount');
  const amount = parseInt(input?.value);
  if (!amount || amount < 1) { if(errEl) errEl.textContent = 'Enter a valid XP amount.'; return; }
  const guild = await fetchGuild(guildState.myGuildCode);
  if (!guild)                { if(errEl) errEl.textContent = 'Could not load guild data.'; return; }
  const myUid       = getLbPlayerKey().replace('lb:','');
  const otherMembers = guild.members.filter(uid => uid !== myUid);
  const memberCount  = otherMembers.length;
  if (memberCount === 0) { if(errEl) errEl.textContent = 'No other members to send XP to.'; return; }
  const totalCost   = amount * memberCount;
  if (state.totalXP < totalCost) {
    if(errEl) errEl.textContent = `Need ${totalCost.toLocaleString()} XP total (${amount} × ${memberCount} other members). You have ${state.totalXP.toLocaleString()}.`;
    return;
  }
  state.totalXP -= totalCost;
  updateXPDisplay();
  // Write each OTHER member's bonus to Firebase
  if (window._fbReady) {
    await Promise.all(otherMembers.map(async uid => {
      try {
        const key = 'guildXP/' + guildState.myGuildCode + '/' + uid;
        const snap = await window._fbGet(window._fbRef(window._fbDb, key));
        const existing = snap.exists() ? (snap.val() || 0) : 0;
        await window._fbSet(window._fbRef(window._fbDb, key), existing + amount);
      } catch(e) { console.warn('xp dist failed for', uid, e); }
    }));
  }
  addLog(`💰 <span class="log-win">Sent ${amount.toLocaleString()} XP to ${memberCount} other guild member${memberCount!==1?'s':''}!</span>`, 'system');
  showSaveToast(`Sent ${amount.toLocaleString()} XP to ${memberCount} member${memberCount!==1?'s':''}!`, '#c9a84c');
  if(errEl) errEl.textContent = '';
  if(input) input.value = '';
  saveGame();
  renderGuildUI();
}

// ── Claim pending guild XP on load ──
async function claimGuildXP() {
  if (!window._fbReady || !guildState.myGuildCode) return;
  try {
    const uid = getLbPlayerKey().replace('lb:','');
    const key = 'guildXP/' + guildState.myGuildCode + '/' + uid;
    const snap = await window._fbGet(window._fbRef(window._fbDb, key));
    if (!snap.exists() || !snap.val()) return;
    const pending = snap.val();
    state.totalXP    += pending;
    state.lifetimeXP += pending;
    await window._fbSet(window._fbRef(window._fbDb, key), 0);
    updateXPDisplay();
    showSaveToast(`+${pending.toLocaleString()} XP from your Guild!`, '#c9a84c');
    addLog(`🏰 <span class="log-win">Received ${pending.toLocaleString()} XP distributed by your Guild Leader!</span>`, 'system');
    saveGame();
  } catch(e) { console.warn('claimGuildXP failed:', e); }
}


// ============================================================
// LEADERBOARD  (shared storage — visible to all players)
// ============================================================
const LB_PREFIX  = 'lb:';          // shared key prefix
const LB_MAX     = 100;            // max entries to store globally
let   lbCurrentTab = 'lifetime';
let   lbMyKey = null;              // this player's storage key

// Derive a stable player key — wallet address required
function getLbPlayerKey() {
  if (lbMyKey) return lbMyKey;
  const profile = (typeof PveAuth !== 'undefined') ? PveAuth.getProfile() : null;
  const walletAddr = profile?.address || shopState.walletAddress || '';
  if (walletAddr) {
    lbMyKey = LB_PREFIX + walletAddr.toLowerCase();
    return lbMyKey;
  }
  return null; // no wallet = no leaderboard key
}

async function submitLeaderboardScore() {
  const key = getLbPlayerKey();
  if (!key) return; // no wallet connected, skip leaderboard submit
  const name = shopState.heroName || 'Anonymous Warrior';
  const skin = SKINS[shopState.equippedSkin]?.emoji || '🧙';
  const totalSkillLv = SKILLS.reduce((s,k) => s + state.skills[k], 0);

  const entry = {
    name,
    skin,
    enemiesDefeated: state.enemiesDefeated,
    wave:            Math.max(1, state.wave - 1),
    totalXP:         state.totalXP,
    lifetimeXP:      state.lifetimeXP,
    lifetimeXP:      state.lifetimeXP,
    totalSkillLv,
    guildCode:       guildState.myGuildCode || null,
    wallet:          shopState.walletAddress || '',
    ts:              Date.now(),
  };

  if (!window._fbReady) return;
  try {
    const uid = key.replace('lb:', '');
    await window._fbSet(window._fbRef(window._fbDb, 'scores/' + uid), entry);
  } catch(e) {
    console.warn('Leaderboard submit failed:', e);
  }
}

async function fetchLeaderboardEntries() {
  if (!window._fbReady) return [];
  try {
    const myKey = getLbPlayerKey();
    const snap = await window._fbGet(window._fbRef(window._fbDb, 'scores'));
    if (!snap.exists()) return [];
    return Object.entries(snap.val()).slice(0, LB_MAX).map(([uid, d]) => {
      d._key  = 'lb:' + uid;
      d._isMe = ('lb:' + uid === myKey);
      return d;
    });
  } catch(e) {
    console.warn('Leaderboard fetch failed:', e);
    return [];
  }
}

function sortLeaderboard(entries, tab) {
  const field = { wave:'wave', lifetime:'lifetimeXP' }[tab];
  return [...entries].sort((a, b) => (b[field] || 0) - (a[field] || 0));
}

function renderLeaderboardTable(entries, tab) {
  const field  = { wave:'wave', lifetime:'lifetimeXP' }[tab];
  const label  = { wave:'Highest Wave Reached', lifetime:'Lifetime XP Earned' }[tab];
  const fmt    = { wave: v => `Wave ${v}`, lifetime: v => `${v.toLocaleString()} XP` }[tab];

  if (!entries.length) {
    return `<div class="lb-empty">No scores yet — be the first on the board!</div>`;
  }

  const sorted = sortLeaderboard(entries, tab);

  const rankIcon = i => i === 0 ? '🥇' : i === 1 ? '🥈' : i === 2 ? '🥉' : `${i+1}`;
  const rankCls  = i => i === 0 ? 'lb-rank lb-rank-1' : i === 1 ? 'lb-rank lb-rank-2' : i === 2 ? 'lb-rank lb-rank-3' : 'lb-rank';

  const rows = sorted.slice(0, 100).map((e, i) => `
    <tr class="${e._isMe ? 'lb-you-row' : ''}">
      <td class="${rankCls(i)}">${rankIcon(i)}</td>
      <td class="lb-skin">${e.skin || '🧙'}</td>
      <td><span class="lb-name${e._isMe ? ' is-you' : ''}">${e.name || 'Unknown'}</span></td>
      <td class="lb-score">${fmt(e[field] || 0)}</td>
    </tr>`).join('');

  return `
    <table class="lb-table">
      <thead>
        <tr>
          <th>#</th>
          <th></th>
          <th>Warrior</th>
          <th style="text-align:right">${label}</th>
        </tr>
      </thead>
      <tbody>${rows}</tbody>
    </table>`;
}

async function openLeaderboard() {
  document.getElementById('lb-overlay').classList.remove('hidden');
  document.getElementById('lb-popup').classList.remove('hidden');
  lbCurrentTab = 'lifetime';
  document.querySelectorAll('.lb-tab').forEach((t,i) => {
    t.classList.toggle('active', i === 1); // default to Lifetime XP tab
  });
  await refreshLeaderboard();
  // Pre-load guild data in background
  fetchAllGuilds().catch(()=>{});
}

async function refreshLeaderboard() {
  if (!window._fbReady) {
    document.getElementById('lb-content').innerHTML =
      `<div class="lb-empty" style="line-height:1.8">
        🔥 <strong style="color:#f59e0b">Firebase not configured</strong><br>
        <span style="font-size:0.8rem">Add your Firebase config to the HTML file to enable the shared leaderboard and guilds.<br>
        See the orange banner at the top of the page for instructions.</span>
      </div>`;
    document.getElementById('lb-refresh-info').textContent = 'Firebase required for leaderboard';
    return;
  }
  document.getElementById('lb-content').innerHTML = '<div class="lb-loading">⏳ Loading scores...</div>';
  const entries = await fetchLeaderboardEntries();
  document.getElementById('lb-content').innerHTML = renderLeaderboardTable(entries, lbCurrentTab);
  document.getElementById('lb-refresh-info').textContent =
    `${entries.length} warrior${entries.length !== 1 ? 's' : ''} on the board · Updates on every save`;
}

async function switchLbTab(tab) {
  lbCurrentTab = tab;
  const tabs = ['wave','lifetime','guilds'];
  document.querySelectorAll('.lb-tab').forEach((t,i) => t.classList.toggle('active', tabs[i] === tab));
  document.getElementById('lb-content').innerHTML = '<div class="lb-loading">⏳ Loading...</div>';
  if (tab === 'guilds') {
    const html = await renderGuildLeaderboard();
    document.getElementById('lb-content').innerHTML = html;
  } else {
    const entries = await fetchLeaderboardEntries();
    document.getElementById('lb-content').innerHTML = renderLeaderboardTable(entries, tab);
  }
}

function closeLeaderboard() {
  document.getElementById('lb-overlay').classList.add('hidden');
  document.getElementById('lb-popup').classList.add('hidden');
}


// ============================================================
// SAVE / LOAD — Firebase only, keyed to wallet address
// ============================================================

// ── Save key helpers (mirrors Trade Together's _ttLocalKey pattern) ──
function _arenaLocalKey() {
  return shopState.walletAddress
    ? 'arena:' + shopState.walletAddress.toLowerCase()
    : 'arena:guest';
}

function _saveLocal(data) {
  try { localStorage.setItem(_arenaLocalKey(), JSON.stringify(data)); } catch(e) {}
}

function _loadLocal() {
  try {
    const raw = localStorage.getItem(_arenaLocalKey());
    return raw ? JSON.parse(raw) : null;
  } catch(e) { return null; }
}

async function saveGame() {
  if (!shopState.walletAddress) {
    showSaveToast('⚠ Connect wallet to save', '#e74c3c');
    return;
  }
  const data = {
    skills:          state.skills,
    skillClicks:     state.skillClicks,
    totalXP:         state.totalXP,
    lifetimeXP:      state.lifetimeXP,
    enemiesDefeated: state.enemiesDefeated,
    wave:            state.wave,
    shop: {
      heroName:          shopState.heroName,
      nameChangeCount:   shopState.nameChangeCount,
      equippedSkin:      shopState.equippedSkin,
      ownedSkins:        shopState.ownedSkins,
      oneStrikeUnlocked: shopState.oneStrikeUnlocked,
      walletAddress:     shopState.walletAddress,
    },
    guild: {
      code:     guildState.myGuildCode,
      name:     guildState.myGuildName,
      isLeader: guildState.isLeader,
    },
    potions: {
      poisonOwned: potionState.poison.owned,
      missOwned:   potionState.miss.owned,
    },
    savedAt: Date.now(),
  };

  // Layer 1: localStorage — instant, survives refresh, no dependencies
  _saveLocal(data);

  // Layer 2: window.storage / PveStorage (wallet-keyed Firebase)
  if (window.storage && shopState.walletAddress && !window.storage._isStub) {
    const cloudKey = 'arena-save:' + shopState.walletAddress.toLowerCase();
    try { await window.storage.set(cloudKey, JSON.stringify(data), false); } catch(e) {}
  }

  // Layer 3: Firebase REST leaderboard
  submitLeaderboardScore();

  showSaveToast('✔ Progress saved!', '#27ae60');
}

async function loadGame() {
  try {
    const addr = shopState.walletAddress;

    // Layer 2: window.storage / PveStorage (cloud, most authoritative)
    if (window.storage && addr && !window.storage._isStub) {
      try {
        const cloudKey = 'arena-save:' + addr.toLowerCase();
        const r = await window.storage.get(cloudKey, false);
        if (r) {
          const data = JSON.parse(r.value);
          if (data && typeof data.wave === 'number') {
            _applyLoadedData(data);
            _saveLocal(data); // keep local in sync
            return true;
          }
        }
      } catch(e) {}
    }

    // Layer 1: localStorage fallback
    const ls = _loadLocal();
    if (ls && typeof ls.wave === 'number') {
      _applyLoadedData(ls);
      return true;
    }

    return false;
  } catch(e) {
    console.warn('loadGame error:', e);
    return false;
  }
}

function _applyLoadedData(data) {
  Object.assign(state.skills,      data.skills      || {});
  Object.assign(state.skillClicks, data.skillClicks || {});
  state.totalXP          = data.totalXP          || 0;
  state.lifetimeXP       = data.lifetimeXP       || state.totalXP;
  state.enemiesDefeated  = data.enemiesDefeated  || 0;
  state.wave             = data.wave             || 1;

  if (data.shop) {
    shopState.heroName          = data.shop.heroName          || '';
    shopState.nameChangeCount   = data.shop.nameChangeCount   || 0;
    shopState.equippedSkin      = data.shop.equippedSkin      || 0;
    shopState.ownedSkins        = data.shop.ownedSkins        || [0];
    shopState.oneStrikeUnlocked = data.shop.oneStrikeUnlocked || false;
    if (data.shop.walletAddress) shopState.walletAddress = data.shop.walletAddress;
  }
  if (shopState.heroName) {
    const nameEl = document.querySelector('.combatant-name');
    if (nameEl) nameEl.textContent = shopState.heroName.toUpperCase();
    const costEl = document.getElementById('name-cost-label');
    if (costEl) costEl.textContent = 'Rename costs 10 XP';
  }
  if (shopState.equippedSkin > 0) {
    document.getElementById('player-sprite').textContent = SKINS[shopState.equippedSkin].emoji;
  }
  if (data.guild?.code) {
    guildState.myGuildCode = data.guild.code;
    guildState.myGuildName = data.guild.name;
    guildState.isLeader    = data.guild.isLeader || false;
    saveGuildLocally();
  }
  if (data.potions) {
    potionState.poison.owned = data.potions.poisonOwned || false;
    potionState.miss.owned   = data.potions.missOwned   || false;
  }
  document.getElementById('wave-num').textContent    = state.wave;
  document.getElementById('enemy-count').textContent = state.enemiesDefeated;
  document.getElementById('btn-fight').textContent   = state.wave > 1
    ? `⚔ CHALLENGE WAVE ${state.wave}` : '⚔ ENTER THE ARENA';
}

function deleteSave() {
  const first = confirm(
    'RESET ALL PROGRESS?\n\n' +
    'This will permanently delete all skill levels, XP, hero name, skins, wave progress and guild membership.\n\n' +
    'This CANNOT be undone. Continue?'
  );
  if (!first) return;

  const second = confirm(
    'FINAL WARNING\n\n' +
    'ALL progress will be lost forever.\n\n' +
    'Press OK to reset everything and start from the beginning.\n' +
    'Press Cancel to keep your progress.'
  );
  if (!second) return;

  // Delete localStorage save
  try { localStorage.removeItem(_arenaLocalKey()); } catch(e) {}
  // Delete cloud save
  try {
    const addr = shopState.walletAddress;
    if (addr && window.storage && !window.storage._isStub) {
      window.storage.delete('arena-save:' + addr.toLowerCase(), false).catch(() => {});
    }
  } catch(e) {}

  SKILLS.forEach(s => {
    state.skills[s]      = 1;
    state.skillClicks[s] = 0;
    if (state.cdTimers[s]) { clearInterval(state.cdTimers[s]); state.cdTimers[s] = null; }
    const btn = document.getElementById('btn-' + s);
    if (btn) btn.disabled = false;
    const cdBar  = document.getElementById('cd-bar-'  + s);
    const cdText = document.getElementById('cd-text-' + s);
    if (cdBar)  cdBar.style.width  = '0%';
    if (cdText) cdText.textContent = '';
  });

  state.totalXP         = 0;
  state.lifetimeXP      = 0;
  state.enemiesDefeated = 0;
  state.wave            = 1;
  state.fight.active    = false;
  actionLocked          = false;

  shopState.heroName               = '';
  shopState.nameChangeCount        = 0;
  shopState.equippedSkin           = 0;
  shopState.ownedSkins             = [0];
  shopState.oneStrikeUnlocked      = false;
  shopState.oneStrikeUsedThisMatch = false;
  shopState.walletAddress          = '';

  guildState.myGuildCode = null;
  guildState.myGuildName = null;
  guildState.isLeader    = false;

  const nameEl = document.querySelector('.combatant-name');
  if (nameEl) nameEl.textContent = 'THE WARRIOR';
  document.getElementById('player-sprite').textContent  = '🧙';
  document.getElementById('player-title').textContent   = 'Novice Wanderer';
  const nameCostEl = document.getElementById('name-cost-label');
  if (nameCostEl) nameCostEl.textContent = 'First change is free';
  document.getElementById('btn-fight').classList.remove('hidden');
  document.getElementById('btn-fight').textContent = '⚔ ENTER THE ARENA';
  document.getElementById('btn-next').classList.add('hidden');
  document.getElementById('btn-respawn').classList.add('hidden');
  document.getElementById('btn-one-strike').classList.add('hidden');
  updatePotionFightButtons();
  document.getElementById('wave-num').textContent    = '1';
  document.getElementById('enemy-count').textContent = '0';

  setStatus('idle', '⚙ IDLE');
  setTurnIndicator('none');
  setActionButtons(false);
  renderSkills();
  resetPlayerStats();
  updateXPDisplay();
  renderSkinGrid();

  showSaveToast('Progress reset. Starting fresh!', '#e74c3c');
}
function showSaveToast(msg, color) {
  let toast = document.getElementById('save-toast');
  if (!toast) {
    toast = document.createElement('div');
    toast.id = 'save-toast';
    toast.style.cssText = `position:fixed;bottom:1.2rem;right:1.2rem;padding:0.6rem 1.1rem;
      border-radius:4px;font-family:'Cinzel',serif;font-size:0.85rem;letter-spacing:0.08em;
      z-index:9999;transition:opacity 0.4s;pointer-events:none;border:1px solid rgba(255,255,255,0.15);
      box-shadow:0 4px 20px rgba(0,0,0,0.6);`;
    document.body.appendChild(toast);
  }
  toast.textContent = msg;
  toast.style.background = color + 'dd';
  toast.style.color = '#fff';
  toast.style.opacity = '1';
  clearTimeout(toast._timer);
  toast._timer = setTimeout(() => { toast.style.opacity = '0'; }, 2200);
}

// Auto-save on key events
const _origPlayerWins = playerWins;
// Patch: save after win, after skill levelup, after shop purchases
const _patchSave = () => saveGame();



// ============================================================
// ADMIN SNAPSHOT  (secret key: Shift+Alt+A)
// ============================================================
let adminUnlocked = false;

document.addEventListener('keydown', function(e) {
  if (e.shiftKey && e.altKey && e.key === 'A') {
    adminUnlocked = !adminUnlocked;
    const btn = document.getElementById('btn-admin-snapshot');
    if (btn) btn.classList.toggle('visible', adminUnlocked);
    const btn2 = document.getElementById('btn-skin-snapshot');
    if (btn2) btn2.style.display = adminUnlocked ? 'inline-block' : 'none';
    showSaveToast(adminUnlocked ? '🔓 Admin mode ON' : '🔒 Admin mode OFF', adminUnlocked ? '#8a8aff' : '#666');
  }
});

async function downloadSnapshotCSV() {
  if (!adminUnlocked) return;
  if (!window._fbReady) {
    showSaveToast('Firebase not connected', '#e74c3c'); return;
  }

  showSaveToast('⏳ Fetching snapshot...', '#8a8aff');

  try {
    // Fetch all scores
    const scoresSnap = await window._fbGet(window._fbRef(window._fbDb, 'scores'));
    const scores = scoresSnap.exists() ? scoresSnap.val() : {};

    // Fetch all guilds for guild name lookup
    const guildsSnap = await window._fbGet(window._fbRef(window._fbDb, 'guilds'));
    const guilds = guildsSnap.exists() ? guildsSnap.val() : {};
    const guildNameMap = {};
    Object.values(guilds).forEach(g => { guildNameMap[g.code] = g.name; });

    // Build CSV
    const headers = ['Player Name','Skin','Wallet Address','Total XP','Highest Wave','Enemies Defeated','Skill Levels','Guild','Last Saved'];
    const rows = Object.entries(scores).map(([uid, d]) => {
      const guildName = d.guildCode ? (guildNameMap[d.guildCode] || d.guildCode) : '';
      const lastSaved = d.ts ? new Date(d.ts).toLocaleString() : '';
      return [
        `"${(d.name || 'Unknown').replace(/"/g,'""')}"`,
        d.skin || '',
        `"${(d.wallet || '').replace(/"/g,'""')}"`,
        d.lifetimeXP || d.totalXP || 0,
        d.totalXP || 0,
        d.wave || 0,
        d.enemiesDefeated || 0,
        d.totalSkillLv || 0,
        `"${guildName.replace(/"/g,'""')}"`,
        `"${lastSaved}"`,
      ].join(',');
    });

    // Sort by XP descending
    rows.sort((a, b) => {
      const xpA = parseInt(a.split(',')[2]) || 0;
      const xpB = parseInt(b.split(',')[2]) || 0;
      return xpB - xpA;
    });

    const csv = [headers.join(','), ...rows].join('\n');
    const ts  = new Date().toISOString().replace(/[:.]/g,'-').slice(0,19);
    const filename = `iron-arena-snapshot-${ts}.csv`;

    // Download
    const blob = new Blob([csv], { type: 'text/csv' });
    const url  = URL.createObjectURL(blob);
    const a    = document.createElement('a');
    a.href     = url;
    a.download = filename;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);

    showSaveToast(`✓ Snapshot downloaded — ${rows.length} players`, '#8a8aff');
  } catch(e) {
    console.error('Snapshot failed:', e);
    showSaveToast('Snapshot failed — check console', '#e74c3c');
  }
}

// ============================================================
// SKIN SNAPSHOT
// ============================================================
async function openSkinSnapshot() {
  if (!adminUnlocked) return;
  if (!window._fbReady) { showSaveToast('Firebase not connected', '#e74c3c'); return; }

  showSaveToast('⏳ Fetching skin data...', '#8a8aff');

  try {
    const scoresSnap = await window._fbGet(window._fbRef(window._fbDb, 'scores'));
    const scores = scoresSnap.exists() ? scoresSnap.val() : {};

    const usersSnap = await window._fbGet(window._fbRef(window._fbDb, 'users'));
    const users = usersSnap.exists() ? usersSnap.val() : {};

    const playerMap = {};

    Object.values(scores).forEach(d => {
      const addr = (d.wallet || '').toLowerCase();
      if (!addr) return;
      playerMap[addr] = {
        name:        d.name || 'Unknown',
        wallet:      d.wallet || '',
        wave:        d.wave || 0,
        lifetimeXP:  d.lifetimeXP || d.totalXP || 0,
        equippedSkin: d.skin || '🧙',
        ownedSkins:  [0],
      };
    });

    Object.entries(users).forEach(([addr, saveKeys]) => {
      const addrLow = addr.toLowerCase();
      const saveKey = Object.keys(saveKeys || {}).find(k => k.startsWith('arena-save:'));
      if (!saveKey) return;
      try {
        const saveData = JSON.parse(saveKeys[saveKey]);
        if (!saveData) return;
        const owned    = saveData?.shop?.ownedSkins || [0];
        const equipped = saveData?.shop?.equippedSkin || 0;
        const wallet   = saveData?.shop?.walletAddress || addr;
        const name     = saveData?.shop?.heroName || '';
        const addrKey  = wallet.toLowerCase() || addrLow;
        if (!playerMap[addrKey]) {
          playerMap[addrKey] = {
            name: name || 'Unknown', wallet,
            wave: saveData?.wave || 0,
            lifetimeXP: saveData?.lifetimeXP || saveData?.totalXP || 0,
            equippedSkin: SKINS[equipped]?.emoji || '🧙',
            ownedSkins: owned,
          };
        } else {
          playerMap[addrKey].ownedSkins = owned;
          if (name) playerMap[addrKey].name = name;
        }
      } catch(e) {}
    });

    const players = Object.values(playerMap).sort((a,b) => b.lifetimeXP - a.lifetimeXP);
    if (!players.length) { showSaveToast('No player data found', '#e74c3c'); return; }

    _showSkinModal(players);
    showSaveToast(`✓ Loaded ${players.length} players`, '#8a8aff');
  } catch(e) {
    console.error('Skin snapshot failed:', e);
    showSaveToast('Skin snapshot failed — check console', '#e74c3c');
  }
}

function _showSkinModal(players) {
  const old = document.getElementById('skin-snapshot-overlay');
  if (old) old.remove();

  const overlay = document.createElement('div');
  overlay.id = 'skin-snapshot-overlay';
  overlay.style.cssText = `
    position:fixed;inset:0;background:rgba(0,0,0,0.92);z-index:9999;
    display:flex;flex-direction:column;align-items:center;padding:16px;
    font-family:'Cinzel',serif;overflow-y:auto;
  `;

  const header = document.createElement('div');
  header.style.cssText = 'width:100%;max-width:1100px;display:flex;justify-content:space-between;align-items:center;margin-bottom:16px;flex-wrap:wrap;gap:10px;';
  header.innerHTML = `
    <div style="font-size:1.1rem;color:#c8a96e;letter-spacing:.15em;">🎭 SKIN SNAPSHOT — ${players.length} PLAYERS</div>
    <div style="display:flex;gap:10px;">
      <button onclick="_downloadSkinCSV()" style="font-family:'Cinzel',serif;font-size:.72rem;padding:8px 16px;background:rgba(200,169,110,.12);border:1px solid rgba(200,169,110,.4);color:#c8a96e;border-radius:4px;cursor:pointer;letter-spacing:.08em;">⬇ Download CSV</button>
      <button onclick="document.getElementById('skin-snapshot-overlay').remove()" style="font-family:'Cinzel',serif;font-size:.72rem;padding:8px 16px;background:rgba(255,255,255,.05);border:1px solid rgba(255,255,255,.15);color:rgba(255,255,255,.6);border-radius:4px;cursor:pointer;">✕ Close</button>
    </div>
  `;
  overlay.appendChild(header);

  const wrap = document.createElement('div');
  wrap.style.cssText = 'width:100%;max-width:1100px;overflow-x:auto;';

  const skinHeaders = SKINS.map(s =>
    `<th style="padding:8px 6px;font-size:.6rem;letter-spacing:.08em;white-space:nowrap;color:rgba(200,169,110,.7);border-bottom:1px solid rgba(200,169,110,.15);">${s.emoji}<br><span style="font-size:.5rem;opacity:.6;">${s.name}</span></th>`
  ).join('');

  let rows = '';
  players.forEach((p, idx) => {
    const walletShort = p.wallet ? p.wallet.slice(0,6) + '…' + p.wallet.slice(-4) : '—';
    const skinCells = SKINS.map((s, i) => {
      const owns     = p.ownedSkins.includes(i);
      const equipped = p.equippedSkin === s.emoji;
      const bg      = owns ? (equipped ? 'rgba(200,169,110,.18)' : 'rgba(39,174,96,.12)') : 'transparent';
      const border  = owns ? (equipped ? 'rgba(200,169,110,.5)'  : 'rgba(39,174,96,.3)')  : 'rgba(255,255,255,.05)';
      const content = owns ? (equipped ? `<span style="font-size:1.1rem;">${s.emoji}</span>` : '✓') : '·';
      const color   = owns ? (equipped ? '#c8a96e' : '#27ae60') : 'rgba(255,255,255,.15)';
      return `<td style="text-align:center;padding:6px 4px;background:${bg};border:1px solid ${border};border-radius:3px;color:${color};font-size:.75rem;">${content}</td>`;
    }).join('');

    const rowBg = idx % 2 === 0 ? 'rgba(255,255,255,.02)' : 'transparent';
    rows += `<tr style="background:${rowBg};">
      <td style="padding:8px 10px;font-size:.72rem;color:rgba(200,169,110,.6);text-align:center;">${idx+1}</td>
      <td style="padding:8px 10px;font-size:.78rem;color:#e8e6f0;white-space:nowrap;max-width:130px;overflow:hidden;text-overflow:ellipsis;">${(p.name||'Unknown').replace(/</g,'&lt;')}</td>
      <td style="padding:8px 10px;font-size:.65rem;color:rgba(255,255,255,.4);font-family:monospace;">${walletShort}</td>
      <td style="padding:8px 10px;font-size:.72rem;color:rgba(127,170,255,.8);text-align:center;">Wv ${p.wave}</td>
      <td style="padding:8px 10px;font-size:.72rem;color:rgba(200,169,110,.7);text-align:right;white-space:nowrap;">${(p.lifetimeXP||0).toLocaleString()} XP</td>
      <td style="padding:8px 10px;font-size:.72rem;color:#27ae60;text-align:center;">${p.ownedSkins.length} / ${SKINS.length}</td>
      ${skinCells}
    </tr>`;
  });

  wrap.innerHTML = `
    <table style="border-collapse:separate;border-spacing:2px;width:100%;min-width:900px;">
      <thead><tr>
        <th style="padding:8px 10px;font-size:.62rem;letter-spacing:.1em;color:rgba(200,169,110,.7);text-align:center;border-bottom:1px solid rgba(200,169,110,.15);">#</th>
        <th style="padding:8px 10px;font-size:.62rem;letter-spacing:.1em;color:rgba(200,169,110,.7);text-align:left;border-bottom:1px solid rgba(200,169,110,.15);">PLAYER</th>
        <th style="padding:8px 10px;font-size:.62rem;letter-spacing:.1em;color:rgba(200,169,110,.7);border-bottom:1px solid rgba(200,169,110,.15);">WALLET</th>
        <th style="padding:8px 10px;font-size:.62rem;letter-spacing:.1em;color:rgba(200,169,110,.7);text-align:center;border-bottom:1px solid rgba(200,169,110,.15);">WAVE</th>
        <th style="padding:8px 10px;font-size:.62rem;letter-spacing:.1em;color:rgba(200,169,110,.7);text-align:right;border-bottom:1px solid rgba(200,169,110,.15);">LIFETIME XP</th>
        <th style="padding:8px 10px;font-size:.62rem;letter-spacing:.1em;color:rgba(200,169,110,.7);text-align:center;border-bottom:1px solid rgba(200,169,110,.15);">OWNED</th>
        ${skinHeaders}
      </tr></thead>
      <tbody>${rows}</tbody>
    </table>
  `;

  const legend = document.createElement('div');
  legend.style.cssText = 'width:100%;max-width:1100px;display:flex;gap:20px;margin-top:12px;font-size:.65rem;color:rgba(255,255,255,.4);flex-wrap:wrap;';
  legend.innerHTML = `
    <span><span style="color:#c8a96e;font-size:.9rem;">🧙</span> = Currently equipped</span>
    <span><span style="color:#27ae60;">✓</span> = Owned, not equipped</span>
    <span style="color:rgba(255,255,255,.2);">· = Not owned</span>
    <span>Sorted by lifetime XP</span>
  `;
  overlay.appendChild(wrap);
  overlay.appendChild(legend);
  document.body.appendChild(overlay);

  window._skinSnapshotPlayers = players;
}

function _downloadSkinCSV() {
  const players = window._skinSnapshotPlayers;
  if (!players) return;

  const skinNames = SKINS.map(s => `${s.emoji} ${s.name}`);
  const headers = ['Rank','Player','Wallet','Wave','Lifetime XP','Skins Owned',...skinNames];

  const rows = players.map((p, idx) => {
    const skinCols = SKINS.map((s, i) => {
      if (!p.ownedSkins.includes(i)) return 'No';
      return p.equippedSkin === s.emoji ? 'Equipped' : 'Owned';
    });
    return [
      idx + 1,
      `"${(p.name||'Unknown').replace(/"/g,'""')}"`,
      `"${(p.wallet||'').replace(/"/g,'""')}"`,
      p.wave || 0,
      p.lifetimeXP || 0,
      p.ownedSkins.length,
      ...skinCols,
    ].join(',');
  });

  const csv  = [headers.join(','), ...rows].join('\n');
  const ts   = new Date().toISOString().replace(/[:.]/g,'-').slice(0,19);
  const blob = new Blob([csv], { type:'text/csv' });
  const url  = URL.createObjectURL(blob);
  const a    = document.createElement('a');
  a.href = url; a.download = `iron-arena-skin-snapshot-${ts}.csv`;
  document.body.appendChild(a); a.click();
  document.body.removeChild(a); URL.revokeObjectURL(url);
  showSaveToast('✓ Skin CSV downloaded', '#8a8aff');
}

// ============================================================

function toggleHTP(btn) {
  btn.classList.toggle('open');
  document.getElementById('htp-body').classList.toggle('open');
}

// ============================================================
// ══ BOSS FIGHT SYSTEM ══
// ════════════════════════════════════════════════════════════
// Firebase path: bosses/active  → live boss object or null
// bosses/active/currentHp       → live HP (all players see it)
// bosses/active/players/{uid}   → per-player damage
//
// In demo mode (no active Firebase boss), use the dev console:
//   window.BOSS_DEV_SPAWN()   — spawn a test boss
//   window.BOSS_DEV_END()     — end the test boss
// ============================================================

const BossSystem = (() => {
  // ── State ──────────────────────────────────────────────────
  let _boss       = null;   // current boss object
  let _active     = false;  // is a boss fight live?
  let _myUid      = null;   // set from wallet / device id
  let _myDamage   = 0;
  let _rosterData = {};     // uid → { name, skin, damage }
  let _attackCd      = false;
  let _heavyAttackCd = false;
  let _attackCdTimer = null;
  let _heavyCdTimer  = null;
  const ATTACK_CD_MS = 3000;
  const HEAVY_CD_MS  = 8000;

  // Dev / demo mock roster
  let _mockMode   = false;
  let _mockBoss   = null;

  // ── Init: wire up Firebase listener once FB is ready ───────
  function init() {
    _resolveMyUid();

    // If Firebase is already ready (module loaded before this runs), wire immediately
    if (window._fbReady && window._fbOnValue) {
      _wireFirebase();
      return;
    }

    // Otherwise poll
    let tries = 0;
    const poll = setInterval(() => {
      if (window._fbReady && window._fbOnValue) {
        clearInterval(poll);
        _wireFirebase();
      } else if (++tries > 80) {
        clearInterval(poll);
        console.warn('[BossSystem] Firebase not ready after 16s — live sync disabled.');
      }
    }, 200);
  }

  function _resolveMyUid() {
    // Try every storage key the game uses for player identity
    const raw =
      sessionStorage.getItem('arena_wallet') ||
      localStorage.getItem('arena_wallet')   ||
      localStorage.getItem('pve-wallet-v1')  ||
      localStorage.getItem('arena-device-id');
    if (raw) {
      _myUid = raw.toLowerCase().replace(/[.#$/[\]]/g, '_').slice(0, 60);
    } else {
      // Generate a stable device ID so the player can always participate
      let devId = localStorage.getItem('boss-device-id');
      if (!devId) {
        devId = 'dev_' + Math.random().toString(36).slice(2, 10);
        localStorage.setItem('boss-device-id', devId);
      }
      _myUid = devId;
    }
  }

  function _wireFirebase() {
    try {
      _resolveMyUid(); // wallet may have connected since init() ran
      const db = window._fbDb;
      const onValue = window._fbOnValue;
      const ref = (path) => window._fbRef(db, path);

      // Listen to bosses/active — fires immediately with current value, then on every change
      onValue(ref('bosses/active'), snap => {
        const data = snap.val();
        if (data) {
          _onBossActive(data);
        } else {
          _onBossEnded();
        }
      });
    } catch(e) {
      console.warn('[BossSystem] Firebase listener error:', e);
    }
  }

  // ── Boss becomes active ────────────────────────────────────
  function _onBossActive(data) {
    _boss   = data;
    _active = true;
    _rosterData = data.players || {};

    // Recalculate my damage from firebase
    if (_rosterData[_myUid]) {
      _myDamage = _rosterData[_myUid].damage || 0;
    }

    _setArenaLocked(true);
    _renderBossPanel(true);
    _renderBossSprite();
    _renderBossHp();
    _renderRoster();
    _showSkillBossMode(true);
    _updateEnhancementsUI(true);

    document.getElementById('boss-alert-banner').classList.add('active');
    document.getElementById('boss-alert-name-text').textContent = (data.name || 'THE BOSS').toUpperCase();
    document.getElementById('boss-attack-btn').classList.add('active');
    document.getElementById('boss-heavy-btn').classList.add('active');
  }

  // ── Boss ends ──────────────────────────────────────────────
  function _onBossEnded() {
    _active = false;
    _boss   = null;
    _myDamage = 0;
    _rosterData = {};

    _setArenaLocked(false);
    _renderBossPanel(false);
    _showSkillBossMode(false);
    _updateEnhancementsUI(false);

    document.getElementById('boss-alert-banner').classList.remove('active');
    document.getElementById('boss-attack-btn').classList.remove('active');
    document.getElementById('boss-heavy-btn').classList.remove('active');
  }

  // ── Arena lock ─────────────────────────────────────────────
  function _setArenaLocked(locked) {
    // Soft interrupt only — arena stays fully open during boss fights.
    // The boss panel + alert banner are sufficient signal.
    // Only show/hide the optional enhancements note.
    const note = document.getElementById('enhancements-disabled-note');
    if (note) note.classList.toggle('active', locked);
  }

  // ── Render boss panel ──────────────────────────────────────
  function _renderBossPanel(show) {
    const panel = document.getElementById('boss-fight-panel');
    if (!panel) return;
    panel.classList.toggle('active', show);
    if (show && _boss) {
      document.getElementById('boss-name-display').textContent = (_boss.name || 'THE BOSS').toUpperCase();
      document.getElementById('boss-tier-display').textContent = _boss.tier || '⬛ WORLD BOSS';
    }
  }

  // ── Render HP bar ─────────────────────────────────────────
  function _renderBossHp() {
    if (!_boss) return;
    const pct = Math.max(0, Math.min(100, (_boss.currentHp / _boss.maxHp) * 100));
    const fill = document.getElementById('boss-hp-fill');
    const text = document.getElementById('boss-hp-value-text');
    if (fill) {
      fill.style.width = pct + '%';
      fill.classList.toggle('enraged', pct <= 25);
    }
    if (text) text.textContent = `${Number(_boss.currentHp).toLocaleString()} / ${Number(_boss.maxHp).toLocaleString()}`;

    // Sync sprite rage state
    const spriteEl = document.getElementById('boss-sprite-el');
    const auraEl   = document.getElementById('boss-aura');
    if (spriteEl) spriteEl.classList.toggle('rage', pct <= 25);
    if (auraEl)   auraEl.classList.toggle('rage', pct <= 25);

    // Phase badge
    const badge = document.getElementById('boss-phase-badge');
    if (badge) {
      const phase = pct <= 25 ? 4 : pct <= 50 ? 3 : pct <= 75 ? 2 : 1;
      badge.classList.toggle('visible', phase >= 3);
      badge.textContent = phase === 4 ? '☠ ENRAGED — FINAL STAND' : '⚠ RAGE PHASE ACTIVE';
    }
  }

  // ── Render roster ──────────────────────────────────────────
  function _renderRoster() {
    const el = document.getElementById('boss-roster');
    if (!el) return;
    const players = Object.entries(_rosterData);
    const total = players.reduce((s, [, p]) => s + (p.damage || 0), 0);
    const sorted = players.sort((a, b) => (b[1].damage || 0) - (a[1].damage || 0));
    el.innerHTML = sorted.map(([uid, p]) => {
      const pct = total > 0 ? ((p.damage / total) * 100).toFixed(1) : '0.0';
      const me = uid === _myUid;
      return `<div class="roster-entry${me ? ' me' : ''}">
        <span class="roster-skin">${p.skin || '⚔️'}</span>
        <span class="roster-name">${p.name || uid.slice(0,8)}</span>
        ${me ? '<span class="me-tag">YOU</span>' : ''}
        <span class="roster-dmg">${Number(p.damage||0).toLocaleString()}</span>
        <span class="roster-pct">${pct}%</span>
      </div>`;
    }).join('');
  }

  // ── Skill boss mode (show boss note in skills panel) ──────
  function _showSkillBossMode(show) {
    const note = document.getElementById('skill-boss-info');
    if (note) note.classList.toggle('active', show);
  }

  // ── Disable One Strike + potions during boss ──────────────
  function _updateEnhancementsUI(bossActive) {
    // One Strike
    const osBtn  = document.getElementById('btn-one-strike');
    const osBuy  = document.getElementById('btn-buy-one-strike');
    if (osBtn)  { osBtn.disabled  = bossActive; if (bossActive) osBtn.classList.add('hidden'); }
    if (osBuy)  { osBuy.disabled  = bossActive; }

    // Potions
    ['btn-buy-poison', 'btn-buy-miss'].forEach(id => {
      const b = document.getElementById(id);
      if (b) b.disabled = bossActive;
    });

    // In-fight potion display
    const potDisplay = document.getElementById('potion-active-display');
    if (potDisplay && bossActive) potDisplay.classList.add('hidden');
  }

  // ── Player attacks the boss ────────────────────────────────
  function doAttack(type = 'light') {
    const isHeavy = type === 'heavy';
    if (!_active) return;
    if (isHeavy && _heavyAttackCd) return;
    if (!isHeavy && _attackCd) return;

    // Pull live stats from Iron Arena game state
    const atkLv      = (typeof state !== 'undefined') ? (state.skills?.attack || 1) : 1;
    const critLv     = (typeof state !== 'undefined') ? (state.skills?.crit   || 1) : 1;
    const playerName = (typeof shopState !== 'undefined' && shopState.heroName)
                       ? shopState.heroName : 'Warrior';

    // Crit chance scales with crit skill: 5% base + 2% per level, cap 80%
    const critChance = Math.min(0.80, 0.05 + critLv * 0.02);
    // Crit multiplier scales too: 1.5× base + 0.05 per level, cap 3×
    const critMult   = Math.min(3.0, 1.5 + critLv * 0.05);

    // Base damage — heavy is 2× base but same crit system
    const baseDmg    = isHeavy
      ? (160 + atkLv * 50) * (0.8 + Math.random() * 0.4)
      : (80  + atkLv * 25) * (0.8 + Math.random() * 0.4);
    const isCrit     = Math.random() < critChance;
    const dmg        = Math.floor(baseDmg * (isCrit ? critMult : 1));

    const hitWord    = isHeavy ? 'HEAVY STRIKE' : 'strikes';
    const logMsg     = isCrit
      ? `💥 ${playerName} CRITICAL ${isHeavy ? 'HEAVY ' : ''}HIT — ${dmg.toLocaleString()} dmg! (×${critMult.toFixed(1)})`
      : `${isHeavy ? '💀' : '⚔'} ${playerName} ${hitWord} for ${dmg.toLocaleString()} dmg`;
    const logType    = isCrit ? 'bspecial' : 'bdmg';

    _dealDamage(dmg, logMsg, logType);
    _floatDmg(
      isCrit ? `CRIT! ${dmg.toLocaleString()}` : dmg.toLocaleString(),
      isCrit ? '#e8c96a' : (isHeavy ? '#ff6b00' : '#e74c3c')
    );

    // Start cooldown for the button used
    const cdMs  = isHeavy ? HEAVY_CD_MS : ATTACK_CD_MS;
    const btnId = isHeavy ? 'boss-heavy-btn'  : 'boss-attack-btn';
    const barId = isHeavy ? 'boss-heavy-cd-bar' : 'boss-btn-cd-bar';
    const btn   = document.getElementById(btnId);
    const bar   = document.getElementById(barId);

    if (isHeavy) _heavyAttackCd = true;
    else         _attackCd      = true;
    if (btn) btn.disabled = true;

    let elapsed = 0;
    const timer = setInterval(() => {
      elapsed += 80;
      if (bar) bar.style.width = (elapsed / cdMs * 100) + '%';
      if (elapsed >= cdMs) {
        clearInterval(timer);
        if (isHeavy) _heavyAttackCd = false;
        else         _attackCd      = false;
        if (btn) btn.disabled = false;
        if (bar) bar.style.width = '0%';
      }
    }, 80);
  }

  // ── Skill level-up bonus hit ───────────────────────────────
  function onSkillLevelUp(skillName, newLevel) {
    if (!_active) return;
    const playerName = (typeof shopState !== 'undefined' && shopState.heroName)
                       ? shopState.heroName : 'Warrior';
    const bonusDmg = Math.floor(400 + newLevel * 180 + Math.random() * 250);
    _dealDamage(bonusDmg,
      `⚡ ${playerName} — ${skillName} Lv${newLevel} bonus strike: ${bonusDmg.toLocaleString()} dmg`,
      'bspecial');
    _floatDmg(`LV UP!`, '#c9a84c', true);
  }

  // ── Deal damage (local + Firebase write) ──────────────────
  function _dealDamage(dmg, logMsg, logType) {
    if (!_boss) return;
    _myDamage += dmg;
    _boss.currentHp = Math.max(0, _boss.currentHp - dmg);

    // Update roster data locally for instant render
    if (!_rosterData[_myUid]) {
      const skin = (typeof shopState !== 'undefined' && SKINS)
        ? (SKINS[shopState.equippedSkin]?.emoji || '⚔️') : '⚔️';
      const name = (typeof shopState !== 'undefined')
        ? (shopState.heroName || 'Warrior') : 'Warrior';
      _rosterData[_myUid] = { name, skin, damage: 0 };
    }
    _rosterData[_myUid].damage = _myDamage;

    _addLog(logMsg, logType);
    _renderBossHp();
    _renderRoster();
    _bossSpriteHit(); // animate boss taking the hit

    // Shake on big hits
    if (dmg > 3000) {
      document.body.classList.add('boss-shake');
      setTimeout(() => document.body.classList.remove('boss-shake'), 400);
    }

    // Write to Firebase (if available)
    if (window._fbReady && window._fbDb) {
      try {
        const db = window._fbDb;
        const refFn = (path) => window._fbRef(db, path);
        // Atomic: decrement HP and update my damage share
        window._fbSet(refFn(`bosses/active/currentHp`), _boss.currentHp);
        window._fbSet(refFn(`bosses/active/players/${_myUid}/damage`), _myDamage);
        // Set player info on first hit
        if (!_rosterData[_myUid]?.name) {
          const skin = SKINS?.[shopState?.equippedSkin]?.emoji || '⚔️';
          const name = shopState?.heroName || 'Warrior';
          window._fbSet(refFn(`bosses/active/players/${_myUid}`), { name, skin, damage: _myDamage });
        }
      } catch(e) { /* silent */ }
    }

    // Check if boss died
    if (_boss.currentHp <= 0) {
      _addLog(`☠ ${_boss.name} has been slain!`, 'bsystem');
      const total = Object.values(_rosterData).reduce((s,p) => s + (p.damage||0), 0);
      // Even split among all players who dealt at least 1 damage
      const participants = Object.values(_rosterData).filter(p => (p.damage || 0) > 0);
      const myParticipated = (_myDamage > 0);
      if (_boss.lootPool && myParticipated && participants.length > 0) {
        const evenShare = Math.floor(_boss.lootPool / participants.length);
        _addLog(`💰 Even split: ${_boss.lootPool.toLocaleString()} ÷ ${participants.length} players = ${evenShare.toLocaleString()} $PvE each`, 'bheal');
      } else if (_boss.lootPool && !myParticipated) {
        _addLog(`⚠ You dealt no damage — no loot earned`, 'batk');
      }
      setTimeout(() => {
        if (window._fbReady && window._fbDb) {
          window._fbSet(window._fbRef(window._fbDb, 'bosses/active'), null);
        } else {
          _onBossEnded();
        }
      }, 1200);
    }
  }

  // ── Boss battle log ────────────────────────────────────────
  function _addLog(msg, type) {
    const wrap = document.getElementById('boss-log-wrap');
    if (!wrap) return;
    const ts = new Date().toLocaleTimeString([], {hour:'2-digit',minute:'2-digit',second:'2-digit'});
    const el = document.createElement('div');
    el.className = `boss-log-entry ${type}`;
    el.textContent = `[${ts}] ${msg}`;
    wrap.prepend(el);
    while (wrap.children.length > 35) wrap.removeChild(wrap.lastChild);
    // Animate boss lunging when it attacks
    if (type === 'batk') _bossSpriteAttack();
  }

  // ── Floating damage numbers ────────────────────────────────
  function _floatDmg(text, color, isLevelUp = false) {
    const el = document.createElement('div');
    el.className = 'boss-float-dmg';
    el.textContent = text;
    el.style.color  = color;
    el.style.left   = (20 + Math.random() * 50) + '%';
    el.style.top    = isLevelUp ? '35%' : (15 + Math.random() * 25) + '%';
    el.style.fontSize = isLevelUp ? '1.4rem' : '1.1rem';
    document.body.appendChild(el);
    setTimeout(() => el.remove(), 1200);
  }

  // ── Boss sprite map ────────────────────────────────────────
  const BOSS_SPRITES = {
    'the iron colossus':    '🗿',
    'world breaker':        '🌍',
    'astral executioner':   '⚡',
    'undying behemoth':     '🦣',
    'void architect':       '🌀',
    'the first evil':       '👁️',
    'void emperor':         '🌑',
    'eclipse wraith':       '🌘',
    'blood leviathan':      '🩸',
    'eternal devourer':     '🕳️',
    'demon lord':           '😈',
    'chaos titan':          '🔥',
    'soul reaper':          '💀',
    'abyssal kraken':       '🐙',
    'inferno phoenix':      '🦅',
    'storm colossus':       '⛈️',
    'shadow dragon':        '🐉',
    'arcane lich':          '☠️',
  };
  const TIER_SPRITES = {
    'titan': '👁️', 'mythic': '🌑', 'legendary': '😈', 'epic': '🐉', 'rare': '🗡️',
  };

  function _getBossSprite(name, tier) {
    const key = (name || '').toLowerCase().trim();
    if (BOSS_SPRITES[key]) return BOSS_SPRITES[key];
    for (const [k, v] of Object.entries(BOSS_SPRITES)) {
      if (key.includes(k) || k.includes(key.split(' ')[0])) return v;
    }
    const t = (tier || '').toLowerCase();
    for (const [k, v] of Object.entries(TIER_SPRITES)) {
      if (t.includes(k)) return v;
    }
    return '💀';
  }

  function _renderBossSprite() {
    if (!_boss) return;
    const el    = document.getElementById('boss-sprite-el');
    const label = document.getElementById('boss-graphic-label');
    const aura  = document.getElementById('boss-aura');
    if (!el) return;
    el.textContent = _getBossSprite(_boss.name, _boss.tier);
    if (label) label.textContent = (_boss.name || 'BOSS').toUpperCase();
    const pct = _boss.maxHp > 0 ? (_boss.currentHp / _boss.maxHp) * 100 : 100;
    el.classList.toggle('rage', pct <= 25);
    if (aura) aura.classList.toggle('rage', pct <= 25);
  }

  function _bossSpriteHit() {
    const el = document.getElementById('boss-sprite-el');
    if (!el) return;
    el.classList.remove('boss-taking-hit', 'boss-attacking');
    void el.offsetWidth;
    el.classList.add('boss-taking-hit');
    setTimeout(() => el.classList.remove('boss-taking-hit'), 520);
    _spawnHitParticles();
  }

  function _bossSpriteAttack() {
    const el = document.getElementById('boss-sprite-el');
    if (!el) return;
    el.classList.remove('boss-taking-hit', 'boss-attacking');
    void el.offsetWidth;
    el.classList.add('boss-attacking');
    setTimeout(() => el.classList.remove('boss-attacking'), 720);
  }

  function _spawnHitParticles() {
    const wrap = document.getElementById('boss-graphic-wrap');
    if (!wrap) return;
    const colors = ['#e74c3c','#f39c12','#e8c96a','#ff6b00','#ffffff'];
    for (let i = 0; i < 8; i++) {
      const p = document.createElement('div');
      p.className = 'boss-hit-particle';
      p.style.background = colors[Math.floor(Math.random() * colors.length)];
      p.style.left = '50%'; p.style.top = '40%';
      const angle = Math.random() * Math.PI * 2;
      const dist  = 30 + Math.random() * 55;
      p.style.setProperty('--p-end', `translate(${Math.cos(angle)*dist}px, ${Math.sin(angle)*dist}px)`);
      p.style.animationDelay = (Math.random() * 80) + 'ms';
      wrap.appendChild(p);
      setTimeout(() => p.remove(), 900);
    }
  }

  // ── Dev helpers ────────────────────────────────────────────
  function devSpawn(opts = {}) {
    _resolveMyUid(); // make sure UID is fresh before spawning
    const boss = {
      name:      opts.name      || 'The Iron Colossus',
      tier:      opts.tier      || '⬛ TITAN TIER · WORLD BOSS',
      maxHp:     opts.maxHp     || 500000,
      currentHp: opts.currentHp || 500000,
      lootPool:  opts.lootPool  || 25000,
      players:   {},
    };
    // Always activate locally for instant UI response
    _onBossActive(boss);
    // Also write to Firebase so other players see it
    if (window._fbReady && window._fbDb) {
      try {
        window._fbSet(window._fbRef(window._fbDb, 'bosses/active'), boss);
      } catch(e) { console.warn('[BossSystem] Firebase write failed:', e); }
    }
  }

  function devEnd() {
    _onBossEnded();
    if (window._fbReady && window._fbDb) {
      try {
        window._fbSet(window._fbRef(window._fbDb, 'bosses/active'), null);
      } catch(e) { console.warn('[BossSystem] Firebase end failed:', e); }
    }
  }

  // ── Expose ─────────────────────────────────────────────────
  return { init, doAttack, onSkillLevelUp, devSpawn, devEnd,
    isActive: () => _active, addLog: _addLog,
    _resolveMyUid };
})();

// Expose dev helpers to console
window.BOSS_DEV_SPAWN = (opts) => BossSystem.devSpawn(opts);
window.BOSS_DEV_END   = () => BossSystem.devEnd();

// Global attack handler called by boss attack button
function bossDoAttack(type) { BossSystem.doAttack(type || 'light'); }

// ── Boss Spawn Modal ───────────────────────────────────────
function toggleBossSpawnModal() {
  const modal   = document.getElementById('boss-spawn-modal');
  const overlay = document.getElementById('boss-spawn-overlay');
  const isOpen  = modal.style.display === 'block';
  if (isOpen) { closeBossSpawnModal(); return; }

  // Update warning if boss already active
  const warn = document.getElementById('boss-spawn-active-warning');
  const confirmBtn = document.getElementById('boss-spawn-confirm-btn');
  const isLive = typeof BossSystem !== 'undefined' && BossSystem.isActive();
  warn.style.display = isLive ? 'block' : 'none';
  confirmBtn.disabled = isLive;
  confirmBtn.style.opacity = isLive ? '0.35' : '1';

  modal.style.display   = 'block';
  overlay.style.display = 'block';
}

function closeBossSpawnModal() {
  document.getElementById('boss-spawn-modal').style.display   = 'none';
  document.getElementById('boss-spawn-overlay').style.display = 'none';
}

function spawnBossFromModal() {
  if (typeof BossSystem !== 'undefined' && BossSystem.isActive()) return;
  const name    = document.getElementById('boss-spawn-name').value.trim() || 'The Iron Colossus';
  const maxHp   = parseInt(document.getElementById('boss-spawn-hp').value)   || 1000000;
  const loot    = parseInt(document.getElementById('boss-spawn-loot').value) || 25000;
  const tier    = document.getElementById('boss-spawn-tier').value;
  BossSystem.devSpawn({ name, maxHp, currentHp: maxHp, lootPool: loot, tier });
  document.getElementById('btn-boss-spawn').classList.add('boss-live');
  document.getElementById('btn-boss-spawn').textContent = '💀 BOSS LIVE';
  closeBossSpawnModal();
}

function endBossFromModal() {
  BossSystem.devEnd();
  document.getElementById('btn-boss-spawn').classList.remove('boss-live');
  document.getElementById('btn-boss-spawn').textContent = '☠ BOSS FIGHT';
  closeBossSpawnModal();
}

// Close modal on Escape
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') closeBossSpawnModal();
});

// Screen shake keyframe (boss system)
const _bossShakeStyle = document.createElement('style');
_bossShakeStyle.textContent = `
  @keyframes bossBodyShake {
    0%,100%{transform:translate(0,0) rotate(0deg)}
    20%{transform:translate(-3px,2px) rotate(-0.4deg)}
    40%{transform:translate(3px,-2px) rotate(0.4deg)}
    60%{transform:translate(-2px,3px) rotate(-0.3deg)}
    80%{transform:translate(2px,-1px) rotate(0.3deg)}
  }
  body.boss-shake { animation: bossBodyShake 0.4s ease-in-out; }
`;
document.head.appendChild(_bossShakeStyle);

// Start BossSystem when DOM is ready
document.addEventListener('DOMContentLoaded', () => BossSystem.init());
// Also init immediately in case DOM already loaded
if (document.readyState !== 'loading') BossSystem.init();

// Update boss UID when wallet connects
if (typeof PveAuth !== 'undefined') {
  PveAuth.onLogin(profile => {
    if (profile?.address) BossSystem._resolveMyUid?.();
  });
}


</script>


<!-- ══ LEADERBOARD POPUP ══ -->
<div class="overlay hidden" id="lb-overlay" onclick="closeLeaderboard()"></div>
<div class="lb-popup hidden" id="lb-popup">
  <h2>🏆 IRON ARENA — HALL OF GLORY 🏆</h2>
  <div class="lb-subtitle">Top warriors across all sessions</div>

  <div class="lb-tabs">
    <button class="lb-tab active" onclick="switchLbTab('wave')">🌊 Highest Wave</button>
    <button class="lb-tab" onclick="switchLbTab('lifetime')">⚡ Lifetime XP</button>
    <button class="lb-tab" onclick="switchLbTab('guilds')">⚔ Guilds</button>
  </div>

  <div id="lb-content">
    <div class="lb-loading">Loading scores...</div>
  </div>

  <div class="lb-footer">
    <span class="lb-refresh-info" id="lb-refresh-info">Scores update on every save</span>
    <button class="btn-lb-close" onclick="closeLeaderboard()">✕ Close</button>
  </div>
</div>









<!-- ── PVE STORAGE + AUTH ── -->
<script>
// ═══════════════════════════════════════════════════════════════
// PvE STORAGE — Firebase Realtime Database adapter
// Replaces window.storage across all pages
// ═══════════════════════════════════════════════════════════════
// Paths:
//   Private (per-user):  /users/{sanitizedAddress}/{key}
//   Shared  (global):    /shared/{key}
//
// API mirrors window.storage:
//   PveStorage.get(key, shared)   → { value: jsonString } | null
//   PveStorage.set(key, value, shared)
//   PveStorage.delete(key, shared)
//   PveStorage.list(prefix, shared) → { keys: [...] }
// ═══════════════════════════════════════════════════════════════

const PveStorage = (() => {
  let _db = null;
  let _fbGet, _fbSet, _fbRemove, _fbRef, _fbChild;
  let _userAddr = null;
  let _ready = false;
  let _readyCallbacks = [];

  // ── Firebase init — reuses the instance already loaded by the top <script type="module"> ──
  async function init() {
    if (_ready) return;
    try {
      // Poll until the top module script has finished setting window._fbReady
      let tries = 0;
      while (!window._fbReady && tries++ < 40) {
        await new Promise(r => setTimeout(r, 150));
      }
      if (!window._fbReady) throw new Error('window._fbReady never became true');

      _db       = window._fbDb;
      _fbGet    = window._fbGet;
      _fbSet    = window._fbSet;
      _fbRemove = (ref) => window._fbSet(ref, null); // Firebase v9 delete = set null
      _fbRef    = (path) => window._fbRef(window._fbDb, path);
      _fbChild  = window._fbChild;
      _ready    = true;
      _readyCallbacks.forEach(fn => fn());
      _readyCallbacks = [];
    } catch(e) {
      console.error('PveStorage: Firebase init failed', e);
    }
  }

  function onReady(fn) {
    if (_ready) fn();
    else _readyCallbacks.push(fn);
  }

  function setUser(address) {
    // Sanitize address for Firebase path (no dots, $, #, [, ], /)
    _userAddr = address ? address.toLowerCase().replace(/[.#$/[\]]/g, '_') : null;
  }

  // ── Path helpers ───────────────────────────────────────────
  function _sanitizeKey(key) {
    // Firebase keys can't contain . # $ [ ] /
    return key.replace(/[.#$[\]]/g, '_').replace(/\//g, '__');
  }

  function _path(key, shared) {
    const k = _sanitizeKey(key);
    if (shared) return `shared/${k}`;
    if (!_userAddr) throw new Error('PveStorage: user address not set — call setUser() first');
    return `users/${_userAddr}/${k}`;
  }

  // ── Public API ─────────────────────────────────────────────
  async function get(key, shared = false) {
    if (!_ready) await init();
    try {
      const snap = await _fbGet(_fbRef(_path(key, shared)));
      if (!snap.exists()) return null;
      return { value: snap.val() };
    } catch(e) {
      console.warn('PveStorage.get error', key, e);
      return null;
    }
  }

  async function set(key, value, shared = false) {
    if (!_ready) await init();
    try {
      await _fbSet(_fbRef(_path(key, shared)), value);
      return { key, value, shared };
    } catch(e) {
      console.warn('PveStorage.set error', key, e);
      return null;
    }
  }

  async function del(key, shared = false) {
    if (!_ready) await init();
    try {
      await _fbRemove(_fbRef(_path(key, shared)));
      return { key, deleted: true, shared };
    } catch(e) {
      console.warn('PveStorage.delete error', key, e);
      return null;
    }
  }

  async function list(prefix = '', shared = false) {
    if (!_ready) await init();
    try {
      const basePath = shared ? 'shared' : `users/${_userAddr}`;
      const snap = await _fbGet(_fbRef(basePath));
      if (!snap.exists()) return { keys: [] };
      const allKeys = Object.keys(snap.val() || {});
      const sanitizedPrefix = _sanitizeKey(prefix);
      const keys = allKeys
        .filter(k => !sanitizedPrefix || k.startsWith(sanitizedPrefix))
        .map(k => k); // return as-is (already sanitized)
      return { keys, shared };
    } catch(e) {
      console.warn('PveStorage.list error', e);
      return { keys: [] };
    }
  }

  // ── Install as window.storage replacement ─────────────────
  // Called once after wallet connects
  function install(address) {
    setUser(address);
    const realStorage = {
      get:    (k, s) => get(k, s),
      set:    (k, v, s) => set(k, v, s),
      delete: (k, s) => del(k, s),
      list:   (p, s) => list(p, s),
    };
    window.storage = realStorage;
    // Flush any queued calls that came in before install()
    if (window._storageQueue && window._storageQueue.length) {
      window._storageQueue.forEach(fn => fn());
      window._storageQueue = [];
    }
  }

  // Auto-init Firebase on load
  init();

  return { init, install, setUser, get, set, delete: del, list, onReady };
})();
window.PveStorage = PveStorage;
const PveStorage2 = PveStorage; // alias suppress re-declaration
(()=>{
})();


// ═══════════════════════════════════════════════════════════════
// PvE WALLET AUTH v3 — Zero dependency, direct window.ethereum
// No CDN imports. No ad-blocker issues. MetaMask / any EIP-1193
// wallet works. Falls back gracefully if no wallet installed.
//   pve-wallet-v1  →  { address, displayName, skin, uid }
// ═══════════════════════════════════════════════════════════════
const PveAuth = (() => {
  const WALLET_KEY      = 'pve-wallet-v1';
  let _profile          = null;
  let _onLoginCb        = null;
  let _onLoginListeners = [];

  function _shortAddr(a) { return a ? a.slice(0,6)+'…'+a.slice(-4) : 'Wallet'; }
  function _load()  { try { return JSON.parse(localStorage.getItem(WALLET_KEY)); } catch { return null; } }
  function _save(p) { p ? localStorage.setItem(WALLET_KEY, JSON.stringify(p)) : localStorage.removeItem(WALLET_KEY); }

  function _eth() { return window.ethereum || null; }

  async function _handleAddress(rawAddr) {
    if (!rawAddr) return;
    const addr     = rawAddr.toLowerCase();
    const existing = _load();
    const stableUid = 'w_' + addr.slice(2);
    if (existing && existing.address && existing.address.toLowerCase() === addr) {
      _profile = existing;
      _profile.uid = stableUid;
    } else {
      _profile = { uid: stableUid, address: rawAddr, displayName: _shortAddr(rawAddr), skin: '⚔' };
    }
    _save(_profile);
    const storageKey = addr;
    if (typeof PveStorage !== 'undefined') PveStorage.install(storageKey);
    _updateNavUser();
    _dismissModal();
    if (_onLoginCb) { _onLoginCb(_profile); _onLoginCb = null; }
    _onLoginListeners.forEach(fn => { try { fn(_profile); } catch(e){} });
  }

  // Listen for wallet account changes (user switches in MetaMask extension)
  function _listenAccountChanges() {
    const eth = _eth();
    if (!eth) return;
    eth.on('accountsChanged', async (accounts) => {
      if (accounts && accounts[0]) {
        await _handleAddress(accounts[0]);
      } else {
        // User disconnected all accounts
        _profile = null; _save(null); _updateNavUser();
        location.reload();
      }
    });
  }

  // ── Connect flow ──────────────────────────────────────────────
  async function _connect() {
    const errEl = document.getElementById('a-err');
    const btnEl = document.getElementById('a-btn');
    const eth = _eth();

    if (!eth) {
      // No wallet detected — show install link
      if (errEl) errEl.innerHTML = 'No wallet detected. <a href="https://metamask.io/download/" target="_blank" style="color:#c8a96e">Install MetaMask</a> or use a wallet browser.';
      return;
    }

    try {
      if (btnEl) { btnEl.disabled = true; btnEl.textContent = 'Connecting…'; }
      if (errEl) errEl.textContent = '';
      const accounts = await eth.request({ method: 'eth_requestAccounts' });
      if (accounts && accounts[0]) {
        await _handleAddress(accounts[0]);
      }
    } catch(e) {
      const msg = e.code === 4001 ? 'Connection cancelled.' : 'Connection failed. Try again.';
      if (errEl) errEl.textContent = msg;
      if (btnEl) { btnEl.disabled = false; btnEl.textContent = '⬡ Connect Wallet'; }
    }
  }

  // ── Modal UI ──────────────────────────────────────────────────
  function _injectModal() {
    if (document.getElementById('pve-auth-modal')) return;
    const el = document.createElement('div');
    el.id = 'pve-auth-modal';
    el.innerHTML = `
<style>
#pve-auth-overlay{position:fixed;inset:0;background:rgba(0,0,0,0.92);z-index:99998;display:flex;align-items:center;justify-content:center;padding:16px;backdrop-filter:blur(6px);}
#pve-auth-box{background:#0e0e18;border:1px solid rgba(200,169,110,0.25);border-radius:16px;width:100%;max-width:320px;font-family:'Cinzel',serif;overflow:hidden;}
#pve-auth-box .ah{padding:28px 24px 20px;border-bottom:1px solid rgba(255,255,255,0.06);text-align:center;}
#pve-auth-box .ah-logo{font-size:2.4rem;margin-bottom:10px;}
#pve-auth-box .ah-title{font-size:0.9rem;letter-spacing:0.2em;text-transform:uppercase;color:#e8c96a;margin-bottom:6px;}
#pve-auth-box .ah-sub{font-size:0.62rem;color:#6b6880;letter-spacing:0.05em;line-height:1.8;font-family:monospace;}
#pve-auth-box .abody{padding:24px 24px 28px;display:flex;flex-direction:column;gap:12px;}
#pve-auth-box .abtn{font-family:'Cinzel',serif;font-size:0.75rem;letter-spacing:0.12em;text-transform:uppercase;padding:14px;border-radius:8px;cursor:pointer;transition:all 0.2s;width:100%;display:flex;align-items:center;justify-content:center;gap:10px;border:1px solid rgba(200,169,110,0.5);background:linear-gradient(135deg,rgba(200,169,110,0.12),rgba(200,169,110,0.04));color:#c8a96e;}
#pve-auth-box .abtn:hover:not(:disabled){background:linear-gradient(135deg,rgba(200,169,110,0.24),rgba(200,169,110,0.12));border-color:#c8a96e;color:#e8c96a;}
#pve-auth-box .abtn:disabled{opacity:0.45;cursor:not-allowed;}
#pve-auth-box .aerr{font-size:0.62rem;color:#e06c6c;min-height:16px;text-align:center;font-family:monospace;line-height:1.5;}
#pve-auth-box .afoot{font-size:0.55rem;color:#3a3848;text-align:center;font-family:monospace;letter-spacing:0.05em;}
</style>
<div id="pve-auth-overlay">
  <div id="pve-auth-box">
    <div class="ah">
      <div class="ah-logo">⚔</div>
      <div class="ah-title">Iron Arena</div>
      <div class="ah-sub">Connect your wallet to save progress<br>and claim $PvE rewards</div>
    </div>
    <div class="abody">
      <button class="abtn" id="a-btn" onclick="PveAuth._openModal()">
        <span>⬡</span> Connect Wallet
      </button>
      <div class="aerr" id="a-err"></div>
      <div class="afoot">🔒 Non-custodial — we never access your funds</div>
    </div>
  </div>
</div>`;
    document.body.appendChild(el);
  }

  function _dismissModal() {
    const el = document.getElementById('pve-auth-modal');
    if (el) el.remove();
  }

  // ── Nav chip ──────────────────────────────────────────────────
  function _updateNavUser() {
    const nav = document.getElementById('site-nav');
    if (!nav) return;
    let chip = nav.querySelector('.sn-user');
    if (!chip) {
      chip = document.createElement('span');
      chip.className = 'sn-user';
      chip.style.cssText = 'font-size:0.6rem;letter-spacing:0.06em;color:#c8a96e;white-space:nowrap;flex-shrink:0;cursor:pointer;padding:4px 8px;border:1px solid rgba(200,169,110,0.2);border-radius:6px;transition:all 0.18s;';
      chip.onmouseenter = () => { chip.style.borderColor='rgba(200,169,110,0.5)';chip.style.color='#e8e6f0'; };
      chip.onmouseleave = () => { chip.style.borderColor='rgba(200,169,110,0.2)';chip.style.color='#c8a96e'; };
      nav.appendChild(chip);
    }
    if (_profile) {
      chip.textContent = (_profile.skin||'⚔') + ' ' + (_profile.displayName || _shortAddr(_profile.address));
      chip.title = 'Connected: ' + (_profile.address||'') + '\nSwitch wallets in MetaMask, or click to disconnect';
      chip.onclick = () => {
        if (confirm('Disconnect wallet?\n\nYour progress is saved to Firebase and will restore when you reconnect.')) {
          _profile = null; _save(null); _updateNavUser();
          location.reload();
        }
      };
    } else {
      chip.textContent = '⬡ Connect';
      chip.onclick = () => _openModal();
    }
  }

  async function _openModal() { _injectModal(); await _connect(); }

  // ── Public API ────────────────────────────────────────────────
  async function init() {
    _profile = _load();
    // Listen for account switches in MetaMask immediately
    _listenAccountChanges();
    // If we have a cached profile, try to silently verify wallet is still connected
    if (_profile && _profile.address) {
      const storageKey = _profile.address.toLowerCase();
      if (typeof PveStorage !== 'undefined') PveStorage.install(storageKey);
      const eth = _eth();
      if (eth) {
        try {
          const accounts = await eth.request({ method: 'eth_accounts' }); // non-prompting check
          if (accounts && accounts[0] && accounts[0].toLowerCase() === _profile.address.toLowerCase()) {
            console.log('[PveAuth] Wallet silently reconnected:', _shortAddr(accounts[0]));
          } else if (accounts && accounts[0]) {
            // Different account active in MetaMask — update silently
            await _handleAddress(accounts[0]);
          }
        } catch(e) { /* silent fail */ }
      }
    }
    _updateNavUser();
    return _profile;
  }

  function getProfile()       { return _profile; }
  async function saveProfile(p) { _profile = { ..._profile, ...p }; _save(_profile); _updateNavUser(); }
  function logout()           { _profile = null; _save(null); _updateNavUser(); location.reload(); }
  function onLogin(fn)        { _onLoginListeners.push(fn); }

  async function requireLogin() {
    if (_profile && _profile.address) return _profile;
    _injectModal();
    return new Promise(resolve => {
      _onLoginCb = (p) => { resolve(p); };
    });
  }

  return { init, getProfile, saveProfile, logout, requireLogin, onLogin, _openModal, _shortAddr };
})();
window.PveAuth = PveAuth;

window.PveStorage = PveStorage;



// Phase 1: Render game immediately — playable before any wallet check
(function() {
  renderSkills();
  resetPlayerStats();
  updateXPDisplay();
  renderSkinGrid();
  updateFightButtonWalletGate();
})();

// Boot: mirrors Trade Together's window.addEventListener('load') pattern.
// Check pve-wallet-v1 first (shared ecosystem key), then sessionStorage fallback.
// Never blocks gameplay — save loads silently in background.
window.addEventListener('load', function() {
  var _sharedWallet = null;
  try { _sharedWallet = JSON.parse(localStorage.getItem('pve-wallet-v1') || 'null'); } catch(e) {}

  if (_sharedWallet && _sharedWallet.address) {
    // Already connected via another PvE app — boot straight in
    shopState.walletAddress = _sharedWallet.address;
    sessionStorage.setItem('arena_wallet', _sharedWallet.address);
    const storageKey = _sharedWallet.address.toLowerCase();
    if (typeof PveStorage !== 'undefined') PveStorage.install(storageKey);
    renderWalletDisplay();
    updateFightButtonWalletGate();
    _bootLoadSave(_sharedWallet.address);
    return;
  }

  // Fall back to sessionStorage (same-tab reload)
  var savedWallet = sessionStorage.getItem('arena_wallet');
  if (savedWallet) {
    shopState.walletAddress = savedWallet;
    const storageKey = savedWallet.toLowerCase();
    if (typeof PveStorage !== 'undefined') PveStorage.install(storageKey);
    renderWalletDisplay();
    updateFightButtonWalletGate();
    _bootLoadSave(savedWallet);
    return;
  }

  // No cached wallet — init PveAuth silently (won't prompt, just checks MetaMask)
  (async () => {
    try {
      await PveAuth.init('arena');
      const profile = PveAuth.getProfile();
      if (profile && profile.address) {
        shopState.walletAddress = profile.address;
        sessionStorage.setItem('arena_wallet', profile.address);
        if (typeof PveStorage !== 'undefined') PveStorage.install(profile.address.toLowerCase());
        renderWalletDisplay();
        updateFightButtonWalletGate();
        _bootLoadSave(profile.address);
      }
    } catch(e) {}
  })();
});

async function _bootLoadSave(addr) {
  // Wait for Firebase (up to 5s), then load save
  if (!window._fbReady) {
    await new Promise(resolve => {
      const check = setInterval(() => {
        if (window._fbReady) { clearInterval(check); resolve(); }
      }, 100);
      setTimeout(() => { clearInterval(check); resolve(); }, 5000);
    });
  }
  const loaded = await loadGame().catch(() => false);
  if (loaded) {
    renderSkills();
    resetPlayerStats();
    updateXPDisplay();
    renderSkinGrid();
    renderWalletDisplay();
    updateFightButtonWalletGate();
    showSaveToast('⚔ Progress restored!', '#7a5c1e');
    claimGuildXP();
  }
  const _skinEmoji = SKINS[shopState.equippedSkin]?.emoji || '🧙';
  PveAuth.saveProfile({ skin: _skinEmoji }).catch(() => {});
  setTimeout(() => {
    const banner = document.getElementById('firebase-banner');
    if (banner && window._fbReady) {
      banner.classList.add('connected');
      banner.innerHTML = '<span class="fb-icon">✅</span><strong>Firebase connected</strong> — Leaderboard & Guilds are live!';
    }
  }, 1500);
}

// ── Wallet switch handler ──────────────────────────────────────────
PveAuth.onLogin(async (profile) => {
  const newAddr = (profile.address || '').toLowerCase();
  if (!newAddr) return;

  const currentAddr = (shopState.walletAddress || '').toLowerCase();
  // Use localStorage to detect same wallet on fresh load (shopState may be empty)
  const savedAddr = (() => {
    try {
      const raw = localStorage.getItem('arena:' + currentAddr);
      const d = raw ? JSON.parse(raw) : null;
      return (d?.shop?.walletAddress || '').toLowerCase();
    } catch(e) { return ''; }
  })();
  const isSameWallet = newAddr === currentAddr || newAddr === savedAddr;

  if (isSameWallet) {
    // Same wallet reconnecting — wire up storage without resetting state
    shopState.walletAddress = profile.address;
    sessionStorage.setItem('arena_wallet', profile.address);
    if (typeof PveStorage !== 'undefined') PveStorage.install(newAddr);
    renderWalletDisplay();
    updateFightButtonWalletGate();
    return;
  }

  // Different wallet — save current progress, then switch
  showSaveToast('🔄 Wallet switched — saving...', '#c9a84c');
  await saveGame().catch(() => {});

  lbMyKey = null;
  shopState.walletAddress = profile.address;
  sessionStorage.setItem('arena_wallet', profile.address);

  Object.assign(state.skills,      { stamina:1, attack:1, defense:1, crit:1 });
  Object.assign(state.skillClicks, { stamina:0, attack:0, defense:0, crit:0 });
  state.totalXP = 0; state.lifetimeXP = 0;
  state.enemiesDefeated = 0; state.wave = 1;
  shopState.heroName = ''; shopState.nameChangeCount = 0;
  shopState.equippedSkin = 0; shopState.ownedSkins = [0];
  shopState.oneStrikeUnlocked = false; shopState.oneStrikeUsedThisMatch = false;
  potionState.poison.owned = false; potionState.poison.activeThisFight = false;
  potionState.miss.owned   = false; potionState.miss.activeThisFight   = false;
  guildState.myGuildCode = null; guildState.myGuildName = null; guildState.isLeader = false;

  const combatantNameEl = document.querySelector('.combatant-name');
  if (combatantNameEl) combatantNameEl.textContent = 'THE WARRIOR';
  const heroNameInput = document.getElementById('hero-name-input');
  if (heroNameInput) heroNameInput.value = '';
  const nameCostLabel = document.getElementById('name-cost-label');
  if (nameCostLabel) nameCostLabel.textContent = 'Free first name';
  document.getElementById('wave-num').textContent    = '1';
  document.getElementById('enemy-count').textContent = '0';
  document.getElementById('btn-fight').textContent   = '⚔ ENTER THE ARENA';
  document.getElementById('player-sprite').textContent = '🧙';

  if (typeof PveStorage !== 'undefined') PveStorage.install(newAddr);

  const loaded = await loadGame().catch(() => false);
  renderSkills();
  resetPlayerStats();
  updateXPDisplay();
  renderSkinGrid();
  renderWalletDisplay();
  updateFightButtonWalletGate();

  showSaveToast(loaded ? '✔ Wallet switched — save loaded!' : '✔ New wallet — fresh start!', '#27ae60');
  if (loaded) claimGuildXP();

  const _skinEmoji = SKINS[shopState.equippedSkin]?.emoji || '🧙';
  await PveAuth.saveProfile({ skin: _skinEmoji }).catch(() => {});
});
</script>
</body>
</html>
