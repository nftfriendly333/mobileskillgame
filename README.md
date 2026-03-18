<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>IRON ARENA — Turn-Based Combat</title>
<link href="https://fonts.googleapis.com/css2?family=Cinzel+Decorative:wght@700;900&family=Cinzel:wght@400;600&family=IM+Fell+English:ital@0;1&display=swap" rel="stylesheet">
<!-- Firebase SDK -->
<script type="module">
  import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js';
  import { getDatabase, ref, set, get, child, query, orderByKey, limitToFirst }
    from 'https://www.gstatic.com/firebasejs/10.12.0/firebase-database.js';
  window._fbReady = false;
  window._fbDb    = null;

// For Firebase JS SDK v7.20.0 and later, measurementId is optional
const firebaseConfig = {
  apiKey: "AIzaSyAI3Pd07qv_w_YRfhhnsbSu-CXh4X-ZT9Q",
  authDomain: "live-leaderboard-aee95.firebaseapp.com",
  projectId: "live-leaderboard-aee95",
  storageBucket: "live-leaderboard-aee95.firebasestorage.app",
  messagingSenderId: "316888776436",
  appId: "1:316888776436:web:6be767dfa0a80909cd68a1",
  measurementId: "G-KRF6XBDTGH"
};

  try {
    if (firebaseConfig.databaseURL.includes('YOUR_PROJECT')) {
      console.warn('Firebase not configured — leaderboard/guilds disabled.');
    } else {
      const app = initializeApp(firebaseConfig);
      window._fbDb    = getDatabase(app);
      window._fbSet   = set;
      window._fbRef   = ref;
      window._fbGet   = get;
      window._fbChild = child;
      window._fbReady = true;
      console.log('Firebase connected ✓');
    }
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
  .btn-buy:disabled { opacity:0.35; cursor:not-allowed; }
  .owned-badge { font-family:'Cinzel',serif; font-size: 0.8rem; color:var(--green2); background:rgba(39,174,96,0.1); border:1px solid #1a5c30; padding:0.15rem 0.4rem; border-radius:2px; }
  .skin-grid { display:grid; grid-template-columns:repeat(2,1fr); gap:0.5rem; margin-bottom:0.5rem; }
  .skin-card { background:var(--bg); border:1px solid var(--border); border-radius:3px; padding:0.6rem; text-align:center; cursor:pointer; transition:all 0.2s; position:relative; }
  .skin-card:hover { border-color:var(--border2); }
  .skin-card.equipped { border-color:var(--gold); background:rgba(201,168,76,0.06); }
  .skin-card.owned-skin { border-color:#1a5c30; }
  .skin-emoji { font-size:2rem; display:block; margin-bottom:0.3rem; }
  .skin-name { font-family:'Cinzel',serif; font-size: 0.8rem; color:var(--text2); display:block; margin-bottom:0.2rem; }
  .skin-price-tag { font-size: 0.8rem; color:var(--gold3); font-family:'Cinzel',serif; }
  .name-input-row { display:flex; gap:0.5rem; margin-top:0.5rem; }
  .name-input { flex:1; background:var(--bg); border:1px solid var(--border2); color:var(--white); font-family:'Cinzel',serif; font-size: 0.95rem; padding:0.4rem 0.6rem; border-radius:2px; outline:none; }
  .name-input:focus { border-color:var(--gold3); }
  .one-strike-banner { background:linear-gradient(135deg,rgba(192,57,43,0.15),rgba(120,20,10,0.2)); border:1px solid var(--red3); border-radius:3px; padding:0.8rem; text-align:center; margin-bottom:0.6rem; }
  .one-strike-banner .big-icon { font-size:2.5rem; display:block; margin-bottom:0.3rem; }
  .one-strike-banner h3 { font-size: 1.15rem; color:var(--red2); letter-spacing:0.15em; margin-bottom:0.3rem; text-shadow:0 0 15px rgba(192,57,43,0.5); }
  .one-strike-stats { font-size: 0.85rem; color:var(--text3); font-style:italic; line-height:1.6; margin-bottom:0.5rem; }
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
    background: var(--bg2);
    border: 2px solid var(--gold3);
    border-radius: 8px;
    padding: 1.6rem 1.8rem;
    z-index: 1001;
    width: 92vw;
    max-width: 620px;
    max-height: 88vh;
    overflow-y: auto;
    box-shadow: 0 0 60px rgba(201,168,76,0.2), 0 20px 60px black;
    animation: popupIn 0.3s ease forwards;
  }

  .lb-popup h2 {
    font-size: 1.2rem;
    color: var(--gold);
    text-align: center;
    letter-spacing: 0.15em;
    margin-bottom: 0.3rem;
    text-shadow: 0 0 20px rgba(201,168,76,0.4);
  }

  .lb-subtitle {
    text-align: center;
    font-size: 0.8rem;
    color: var(--text3);
    font-style: italic;
    margin-bottom: 1.2rem;
  }

  .lb-tabs {
    display: flex;
    gap: 0.4rem;
    margin-bottom: 1rem;
  }

  .lb-tab {
    flex: 1;
    padding: 0.4rem 0.3rem;
    font-family: 'Cinzel', serif;
    font-size: 0.7rem;
    letter-spacing: 0.08em;
    background: var(--bg);
    border: 1px solid var(--border);
    color: var(--text3);
    cursor: pointer;
    border-radius: 2px;
    transition: all 0.2s;
    text-align: center;
  }
  .lb-tab.active { background: rgba(201,168,76,0.1); border-color: var(--gold3); color: var(--gold); }
  .lb-tab:hover:not(.active) { border-color: var(--border2); color: var(--text2); }

  .lb-table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.85rem;
  }

  .lb-table th {
    font-family: 'Cinzel', serif;
    font-size: 0.7rem;
    letter-spacing: 0.1em;
    color: var(--gold3);
    text-align: left;
    padding: 0.4rem 0.6rem;
    border-bottom: 1px solid var(--border2);
    text-transform: uppercase;
  }

  .lb-table td {
    padding: 0.55rem 0.6rem;
    border-bottom: 1px solid rgba(255,255,255,0.04);
    color: var(--text2);
    vertical-align: middle;
  }

  .lb-table tr:last-child td { border-bottom: none; }

  .lb-table tr:hover td { background: rgba(255,255,255,0.02); }

  .lb-rank {
    font-family: 'Cinzel', serif;
    font-size: 0.8rem;
    color: var(--text3);
    width: 2rem;
    text-align: center;
  }

  .lb-rank-1 { color: #f59e0b; font-size: 1rem; }
  .lb-rank-2 { color: #94a3b8; font-size: 0.95rem; }
  .lb-rank-3 { color: #b45309; font-size: 0.9rem; }

  .lb-name {
    font-family: 'Cinzel', serif;
    font-size: 0.82rem;
    color: var(--white);
  }
  .lb-name.is-you { color: var(--gold); }
  .lb-name.is-you::after { content: ' (You)'; font-size: 0.65rem; color: var(--gold3); }

  .lb-score {
    font-family: 'Cinzel', serif;
    font-size: 0.85rem;
    color: var(--green2);
    text-align: right;
  }

  .lb-skin { font-size: 1.1rem; }

  .lb-empty {
    text-align: center;
    color: var(--text3);
    font-style: italic;
    padding: 1.5rem;
    font-size: 0.85rem;
  }

  .lb-you-row td { background: rgba(201,168,76,0.04); }

  .lb-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 1rem;
    padding-top: 0.8rem;
    border-top: 1px solid var(--border);
    flex-wrap: wrap;
    gap: 0.5rem;
  }

  .lb-refresh-info {
    font-size: 0.72rem;
    color: var(--text3);
    font-style: italic;
  }

  .btn-lb-close {
    background: linear-gradient(135deg, #1e1812, #2a2016);
    border: 1px solid var(--gold3);
    color: var(--gold);
    font-family: 'Cinzel', serif;
    font-size: 0.75rem;
    letter-spacing: 0.1em;
    padding: 0.4rem 1rem;
    border-radius: 3px;
    cursor: pointer;
    transition: all 0.2s;
  }
  .btn-lb-close:hover { border-color: var(--gold); box-shadow: 0 0 10px rgba(201,168,76,0.2); }

  .btn-leaderboard {
    background: linear-gradient(135deg, #1a1408, #2e2010);
    border: 1px solid var(--gold3);
    color: var(--gold);
    font-family: 'Cinzel', serif;
    font-size: 0.82rem;
    letter-spacing: 0.1em;
    padding: 0.4rem 1rem;
    border-radius: 3px;
    cursor: pointer;
    transition: all 0.2s;
  }
  .btn-leaderboard:hover { background: linear-gradient(135deg,#2e2010,#4a3418); border-color:var(--gold); box-shadow:0 0 12px rgba(201,168,76,0.2); }

  .lb-loading {
    text-align: center;
    color: var(--text3);
    font-style: italic;
    padding: 1.5rem;
    font-size: 0.85rem;
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

</style>
</head>
<body>


  
<div class="game-header">
  <h1>⚔ IRON ARENA ⚔</h1>
  <p><span class="ornament">✦</span> Turn-Based Combat — Train, Fight, Survive <span class="ornament">✦</span></p>
</div>

<div class="save-bar">
  <button class="btn-leaderboard" onclick="openLeaderboard()">🏆 Leaderboard</button>
  <button class="btn-save" onclick="saveGame()">💾 Save Progress</button>
  <span class="save-info">Auto-saves on wins &amp; purchases</span>
  <button class="btn-delete-save" onclick="deleteSave()">🗑 Reset Save</button>
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

    </div>
    <div class="htp-tip">
      💡 <strong>Pro tip:</strong> Train skills between every wave. Enemies scale hard — a well-timed Heavy Attack with high Attack skill will carry you far. Save ONE STRIKE for Legendary or Mythic tier bosses!
    </div>
  </div>
</div>

<div class="game-container">

  <!-- ══ SECTION 1: FIGHT ARENA (top) ══ -->
  <div class="arena-col">

    <!-- Combatants -->
    <div class="panel arena-panel">
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

      <div class="status-row">
        <span class="status-badge badge-idle" id="status-badge">⚙ IDLE</span>
        <span class="wave-info">Wave <span id="wave-num">1</span> · <span id="enemy-count">0</span> Defeated</span>
        <span class="wave-info">XP: <span id="xp-total-display"><span>0</span></span></span>
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
      <div class="shop-item">
        <div class="shop-item-name">🏷️ Hero Name</div>
        <div class="shop-item-desc">Name your warrior. First change is free — after that, 10 XP each.</div>
        <div class="name-input-row">
          <input class="name-input" id="hero-name-input" type="text" maxlength="16" placeholder="Enter name..." />
          <button class="btn-buy btn-buy-gold" id="btn-set-name" onclick="setHeroName()">SET</button>
        </div>
        <div style="font-size:0.8rem; color:var(--text3); margin-top:0.4rem; font-style:italic;" id="name-cost-label">First change is free</div>
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
    </div>
  </div><!-- /shop-panel -->

  <!-- ══ SECTION 3: SKILL TRAINING (bottom) ══ -->
  <div class="panel skills-panel">
    <div class="panel-title">⚡ Skill Training</div>

    <div class="skills-grid">
      <!-- Stamina -->
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
];

function getEnemyForWave(wave) {
  let eligible = ENEMY_TIERS.filter(e => e.wave <= wave);
  // After wave 35, only Legendary and Mythic enemies appear
  if (wave > 35) {
    const elites = eligible.filter(e => e.tier === 'LEGENDARY' || e.tier === 'MYTHIC');
    if (elites.length) eligible = elites;
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
    hp: Math.round(base.baseHP * scale),
    atk: Math.round(base.baseAtk * scale),
    def: Math.round(base.baseDef * scale),
    xpReward: Math.round(base.xpBase * scale),
  };
}

// ============================================================
// PLAYER STATS
// ============================================================
function getPlayerMaxHP()      { return 60 + state.skills.defense * 8 + state.skills.stamina * 2; }  // nerfed: was *5
function getPlayerMaxStamina() { return 60 + state.skills.stamina * 5; }   // nerfed: was *10
function getPlayerAttack(type) {
  const base = 3 + state.skills.attack * 2;
  if (type === 'light') return base;
  if (type === 'heavy') return Math.round(base * 1.9);
  return 0;
}
function getPlayerDefense()    { return 1 + state.skills.defense * 1.5; }
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
  f.playerHP = maxHP; f.playerMaxHP = maxHP;
  // Stormbringer always starts at full stamina
  f.playerStamina = maxSta; f.playerMaxStamina = maxSta;
  f.blocking = false;
  f.enemyMaxHP = currentEnemy.hp;
  f.active = true; f.playerTurn = true; f.turn = 1; f.won = false;
  actionLocked = false;
  shopState.oneStrikeUsedThisMatch = false;

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
  let dmg = Math.max(1, currentEnemy.atk - getPlayerDefense() * 0.5);
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
  f.playerHP = Math.max(0, f.playerHP - dmg);
  const monsterAnim = enemyCrit ? 'attack-monster-crit' : 'attack-monster';
  animateCard('enemy-card', monsterAnim, 750);
  setTimeout(() => {
    animateCard('player-card', 'taking-hit', 500);
    if (enemyCrit) animateCard('player-card', 'impact-heavy', 600);
    else animateCard('player-card', 'impact', 550);
  }, 420);
  f.playerStamina = Math.min(f.playerMaxStamina, f.playerStamina + 8 + Math.floor(state.skills.stamina * 1.5));
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
  const xpPenalty = state.wave > 25;
  const xpGain = xpPenalty ? Math.round(currentEnemy.xpReward * 0.9) : currentEnemy.xpReward;
  state.totalXP += xpGain;
  addLog(`☠ <span class="log-death">${currentEnemy.name} slain!</span>`, 'death');
  addLog(`<span class="log-win">🏆 VICTORY! +${xpGain} XP!${xpPenalty ? ' <span style="color:var(--text3);font-size:0.85em;">(-10% late game)</span>' : ''}</span>`, 'win');
  setStatus('won', '✓ VICTORY');
  setActionButtons(false);
  setTurnIndicator('none');
  document.getElementById('btn-one-strike').classList.add('hidden');
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
  document.getElementById('btn-respawn').classList.remove('hidden');
}

function resetPlayerStats() {
  const f = state.fight;
  const maxHP  = getPlayerMaxHP();
  const maxSta = getPlayerMaxStamina();
  f.playerHP           = maxHP;
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
  document.getElementById('player-hp-txt').textContent = `${Math.max(0,f.playerHP)} / ${f.playerMaxHP}`;
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
};

const SKINS = [
  { emoji:'🧙', name:'Wanderer',     price:0   },
  { emoji:'⚔️',  name:'Swordsman',   price:20  },
  { emoji:'🥷',  name:'Shadow Blade',price:40  },
  { emoji:'🦸',  name:'Champion',    price:70  },
  { emoji:'🧝',  name:'Elven Archer',price:90  },
  { emoji:'🧟',  name:'Undead King', price:120 },
  { emoji:'🐲',  name:'Dragon Knight',price:180},
  { emoji:'👑',  name:'Immortal',    price:250 },
];

// Base XP costs — scale up every 5 waves
const XP_SKILL_COST_BASE = { stamina:15, attack:15, defense:15, crit:20 };

function getSkillXPCost(skill) {
  const tier = Math.floor((state.wave - 1) / 5); // increases every 5 waves
  const multiplier = 1 + tier * 0.5;              // +50% per tier (wave 1-5: 1x, 6-10: 1.5x, 11-15: 2x ...)
  return Math.round(XP_SKILL_COST_BASE[skill] * multiplier);
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

function renderSkinGrid() {
  const grid = document.getElementById('skin-grid');
  grid.innerHTML = '';
  SKINS.forEach((skin, i) => {
    const owned = shopState.ownedSkins.includes(i);
    const equipped = shopState.equippedSkin === i;
    const card = document.createElement('div');
    card.className = 'skin-card' + (equipped?' equipped':'') + (owned&&!equipped?' owned-skin':'');
    card.innerHTML = `<span class="skin-emoji">${skin.emoji}</span><span class="skin-name">${skin.name}</span><span class="skin-price-tag">${owned?(equipped?'✦ EQUIPPED':'✓ OWNED'):skin.price+' XP'}</span>`;
    card.onclick = () => owned ? equipSkin(i) : buySkin(i);
    grid.appendChild(card);
  });
}

function buySkin(i) {
  const skin = SKINS[i];
  if (state.totalXP < skin.price) { addLog(`💸 Need ${skin.price} XP for ${skin.name}.`, 'system'); return; }
  state.totalXP -= skin.price;
  shopState.ownedSkins.push(i);
  equipSkin(i);
  addLog(`🎭 <span class="log-win">Unlocked ${skin.name} skin!</span>`, 'system');
  updateXPDisplay();
  saveGame();
}

function equipSkin(i) {
  shopState.equippedSkin = i;
  document.getElementById('player-sprite').textContent = SKINS[i].emoji;
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







// ============================================================
// GUILD SYSTEM
// ============================================================
const GUILD_PREFIX      = 'guild:';
const GUILD_UNLOCK_WAVE = 30;   // must have beaten wave 30 to create
const GUILD_CREATE_COST = 1000; // XP cost to create a guild
const GUILD_JOIN_WAVE   = 10;   // must have beaten wave 10 to join
const GUILD_MAX_MEMBERS = 20;

const guildState = {
  myGuildCode:  localStorage.getItem('ironArena_guildCode')  || null,
  myGuildName:  localStorage.getItem('ironArena_guildName')  || null,
  isLeader:     localStorage.getItem('ironArena_guildLeader') === '1',
};

function generateGuildCode() {
  const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
  return Array.from({length:6}, () => chars[Math.floor(Math.random()*chars.length)]).join('');
}

function getHighestWaveBeaten() {
  return Math.max(0, state.wave - 1);
}

// ── Persist guild membership locally ──
function saveGuildLocally() {
  if (guildState.myGuildCode) {
    localStorage.setItem('ironArena_guildCode',   guildState.myGuildCode);
    localStorage.setItem('ironArena_guildName',   guildState.myGuildName || '');
    localStorage.setItem('ironArena_guildLeader', guildState.isLeader ? '1' : '0');
  } else {
    localStorage.removeItem('ironArena_guildCode');
    localStorage.removeItem('ironArena_guildName');
    localStorage.removeItem('ironArena_guildLeader');
  }
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
  if (guild.members.length >= GUILD_MAX_MEMBERS) { if(errEl) errEl.textContent = 'Guild is full (20 members max).'; return; }

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
        <div style="margin-top:0.7rem;text-align:right;">
          <button class="btn-guild btn-guild-red" onclick="leaveGuild()">Leave Guild</button>
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
// LEADERBOARD  (shared storage — visible to all players)
// ============================================================
const LB_PREFIX  = 'lb:';          // shared key prefix
const LB_MAX     = 50;             // max entries to store globally
let   lbCurrentTab = 'enemies';
let   lbMyKey = null;              // this player's storage key

// Derive a stable player key from hero name + browser fingerprint
function getLbPlayerKey() {
  if (lbMyKey) return lbMyKey;
  // Use hero name + a stored local UID
  let uid = localStorage.getItem('ironArena_uid');
  if (!uid) {
    uid = 'p_' + Math.random().toString(36).slice(2, 10);
    localStorage.setItem('ironArena_uid', uid);
  }
  lbMyKey = LB_PREFIX + uid;
  return lbMyKey;
}

async function submitLeaderboardScore() {
  const key = getLbPlayerKey();
  const name = shopState.heroName || 'Anonymous Warrior';
  const skin = SKINS[shopState.equippedSkin]?.emoji || '🧙';
  const totalSkillLv = SKILLS.reduce((s,k) => s + state.skills[k], 0);

  const entry = {
    name,
    skin,
    enemiesDefeated: state.enemiesDefeated,
    wave:            Math.max(1, state.wave - 1),
    totalXP:         state.totalXP,
    totalSkillLv,
    guildCode:       guildState.myGuildCode || null,
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
  const field = { wave:'wave', xp:'totalXP' }[tab];
  return [...entries].sort((a, b) => (b[field] || 0) - (a[field] || 0));
}

function renderLeaderboardTable(entries, tab) {
  const field  = { wave:'wave', xp:'totalXP' }[tab];
  const label  = { wave:'Highest Wave Reached', xp:'Total XP Earned' }[tab];
  const fmt    = { wave: v => `Wave ${v}`, xp: v => `${v.toLocaleString()} XP` }[tab];

  if (!entries.length) {
    return `<div class="lb-empty">No scores yet — be the first on the board!</div>`;
  }

  const sorted = sortLeaderboard(entries, tab);

  const rankIcon = i => i === 0 ? '🥇' : i === 1 ? '🥈' : i === 2 ? '🥉' : `${i+1}`;
  const rankCls  = i => i === 0 ? 'lb-rank lb-rank-1' : i === 1 ? 'lb-rank lb-rank-2' : i === 2 ? 'lb-rank lb-rank-3' : 'lb-rank';

  const rows = sorted.slice(0, 20).map((e, i) => `
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
  lbCurrentTab = 'wave';
  document.querySelectorAll('.lb-tab').forEach((t,i) => {
    t.classList.toggle('active', i === 0);
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
  const tabs = ['wave','xp','guilds'];
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
// SAVE / LOAD  (localStorage)
// ============================================================
const SAVE_KEY = 'ironArena_v1';

function saveGame() {
  const data = {
    skills:       state.skills,
    skillClicks:  state.skillClicks,
    totalXP:      state.totalXP,
    enemiesDefeated: state.enemiesDefeated,
    wave:         state.wave,
    shop: {
      heroName:          shopState.heroName,
      nameChangeCount:   shopState.nameChangeCount,
      equippedSkin:      shopState.equippedSkin,
      ownedSkins:        shopState.ownedSkins,
      oneStrikeUnlocked: shopState.oneStrikeUnlocked,
    },
    guild: {
      code:     guildState.myGuildCode,
      name:     guildState.myGuildName,
      isLeader: guildState.isLeader,
    }
  };
  try {
    localStorage.setItem(SAVE_KEY, JSON.stringify(data));
    showSaveToast('✔ Progress saved!', '#27ae60');
    submitLeaderboardScore(); // push to shared leaderboard
  } catch(e) {
    showSaveToast('⚠ Save failed', '#e74c3c');
  }
}

function loadGame() {
  try {
    const raw = localStorage.getItem(SAVE_KEY);
    if (!raw) return false;
    const data = JSON.parse(raw);

    // Restore state
    Object.assign(state.skills,      data.skills      || {});
    Object.assign(state.skillClicks, data.skillClicks || {});
    state.totalXP          = data.totalXP          || 0;
    state.enemiesDefeated  = data.enemiesDefeated  || 0;
    state.wave             = data.wave             || 1;

    // Restore shop
    if (data.shop) {
      shopState.heroName          = data.shop.heroName          || '';
      shopState.nameChangeCount   = data.shop.nameChangeCount   || 0;
      shopState.equippedSkin      = data.shop.equippedSkin      || 0;
      shopState.ownedSkins        = data.shop.ownedSkins        || [0];
      shopState.oneStrikeUnlocked = data.shop.oneStrikeUnlocked || false;
    }

    // Apply hero name
    if (shopState.heroName) {
      const nameEl = document.querySelector('.combatant-name');
      if (nameEl) nameEl.textContent = shopState.heroName.toUpperCase();
      document.getElementById('name-cost-label').textContent = 'Rename costs 10 XP';
    }

    // Apply equipped skin
    if (shopState.equippedSkin > 0) {
      document.getElementById('player-sprite').textContent = SKINS[shopState.equippedSkin].emoji;
    }

    // Restore guild membership
    if (data.guild?.code) {
      guildState.myGuildCode = data.guild.code;
      guildState.myGuildName = data.guild.name;
      guildState.isLeader    = data.guild.isLeader || false;
      saveGuildLocally();
    }

    // Update wave / enemy count display
    document.getElementById('wave-num').textContent = state.wave;
    document.getElementById('enemy-count').textContent = state.enemiesDefeated;
    document.getElementById('btn-fight').textContent = state.wave > 1
      ? `⚔ CHALLENGE WAVE ${state.wave}` : '⚔ ENTER THE ARENA';

    return true;
  } catch(e) {
    console.warn('Load failed:', e);
    return false;
  }
}

function deleteSave() {
  if (!confirm('Delete all saved progress? This cannot be undone.')) return;
  localStorage.removeItem(SAVE_KEY);
  showSaveToast('🗑 Save deleted — refresh to restart', '#e74c3c');
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
// INIT
// ============================================================
(function() {
  // Load save first, then render everything fresh
  const loaded = loadGame();
  renderSkills();
  resetPlayerStats();
  updateXPDisplay();
  renderSkinGrid();
  if (loaded) showSaveToast('⚔ Progress restored!', '#7a5c1e');
  // Update Firebase banner status after SDK loads
  setTimeout(() => {
    const banner = document.getElementById('firebase-banner');
    if (banner && window._fbReady) {
      banner.classList.add('connected');
      banner.innerHTML = '<span class="fb-icon">✅</span><strong>Firebase connected</strong> — Leaderboard &amp; Guilds are live!';
    }
  }, 1500);
})();

function toggleHTP(btn) {
  btn.classList.toggle('open');
  document.getElementById('htp-body').classList.toggle('open');
}

</script>


<!-- ══ LEADERBOARD POPUP ══ -->
<div class="overlay hidden" id="lb-overlay" onclick="closeLeaderboard()"></div>
<div class="lb-popup hidden" id="lb-popup">
  <h2>🏆 IRON ARENA — HALL OF GLORY 🏆</h2>
  <div class="lb-subtitle">Top warriors across all sessions</div>

  <div class="lb-tabs">
    <button class="lb-tab active" onclick="switchLbTab('wave')">🌊 Highest Wave</button>
    <button class="lb-tab" onclick="switchLbTab('xp')">⚡ Total XP</button>
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

</body>
</html>
