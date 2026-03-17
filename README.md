<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>IRON ARENA — Turn-Based Combat</title>
<link href="https://fonts.googleapis.com/css2?family=Cinzel+Decorative:wght@700;900&family=Cinzel:wght@400;600&family=IM+Fell+English:ital@0;1&display=swap" rel="stylesheet">
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

  .combatant-card.taking-hit {
    animation: shake 0.3s ease;
  }

  @keyframes shake {
    0%, 100% { transform: translateX(0); }
    25% { transform: translateX(-6px); }
    75% { transform: translateX(6px); }
  }

  .combatant-sprite {
    font-size: 4rem;
    line-height: 1;
    margin-bottom: 0.5rem;
    display: block;
    filter: drop-shadow(0 4px 8px rgba(0,0,0,0.6));
    transition: transform 0.3s;
  }

  .combatant-card.attacking .combatant-sprite {
    animation: attackAnim 0.4s ease;
  }

  @keyframes attackAnim {
    0%, 100% { transform: translateX(0) scale(1); }
    50% { transform: translateX(20px) scale(1.1); }
  }

  .enemy-card.attacking .combatant-sprite {
    animation: enemyAttackAnim 0.4s ease;
  }

  @keyframes enemyAttackAnim {
    0%, 100% { transform: translateX(0) scale(1); }
    50% { transform: translateX(-20px) scale(1.1); }
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
    .combatants { grid-template-columns: 1fr; }
    .vs-divider { display: none; }
    .combatant-sprite { font-size: 2.5rem; }
    .action-buttons { grid-template-columns: 1fr 1fr; }
    .action-buttons .btn-heavy { grid-column: 1 / -1; }
    .battle-log { height: 130px; }
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

</style>
</head>
<body>

<div class="game-header">
  <h1>⚔ IRON ARENA ⚔</h1>
  <p><span class="ornament">✦</span> Turn-Based Combat — Train, Fight, Survive <span class="ornament">✦</span></p>
</div>

<div class="save-bar">
  <button class="btn-save" onclick="saveGame()">💾 Save Progress</button>
  <span class="save-info">Auto-saves on wins &amp; purchases</span>
  <button class="btn-delete-save" onclick="deleteSave()">🗑 Reset Save</button>
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

    <!-- Battle Log -->
    <div class="panel log-panel">
      <div class="panel-title">📜 Battle Chronicle</div>
      <div class="battle-log" id="battle-log">
        <div class="log-entry log-system">⚔ Welcome to the Iron Arena. Train your skills and face your enemies.</div>
        <div class="log-entry log-system">💡 Click LEVEL UP 25 times per skill to advance. Each level-up triggers a 1-minute cooldown.</div>
      </div>
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
  { name:'VOID EMPEROR', sprite:'🌑', title:'The sovereign of nothingness',            tier:'MYTHIC',    tierClass:'tier-mythic',    baseHP:500, baseAtk:50, baseDef:30, xpBase:300, wave:22 },
];

function getEnemyForWave(wave) {
  const eligible = ENEMY_TIERS.filter(e => e.wave <= wave);
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
function getPlayerMaxHP()      { return 60 + state.skills.defense * 8 + state.skills.stamina * 5; }
function getPlayerMaxStamina() { return 60 + state.skills.stamina * 10; }
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
        animateCard('enemy-card', 'taking-hit');
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
    updateBars();
    setTimeout(enemyTurn, 700);
  } else {
    const rawAtk = getPlayerAttack(type);
    const isCrit = Math.random() < getCritChance();
    let dmg = Math.max(1, Math.round(rawAtk - currentEnemy.def * 0.4));
    if (isCrit) dmg = Math.round(dmg * getCritMultiplier());
    currentEnemy.hp = Math.max(0, currentEnemy.hp - dmg);
    const label = type === 'light' ? '⚡ Light' : '💀 Heavy';
    addLog(isCrit
      ? `${label} — <span class="log-crit">CRITICAL HIT! ${dmg} damage!</span>`
      : `${label} — ${dmg} damage to ${currentEnemy.name}.`, 'player');
    showDamage('enemy-card', isCrit ? `💥 ${dmg}!` : `-${dmg}`, isCrit ? 'damage-crit' : 'damage-normal');
    animateCard('player-card', 'attacking');
    setTimeout(() => animateCard('enemy-card', 'taking-hit'), 200);
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
  if (f.blocking) {
    const blockFactor = 0.15 + (state.skills.defense * 0.03);
    dmg = Math.max(1, Math.round(dmg * (1 - Math.min(blockFactor * 3, 0.75))));
    addLog(`🛡️ <span class="log-block">Blocked! ${currentEnemy.name} deals only ${dmg}.</span>`, 'block');
    showDamage('player-card', `-${dmg}`, 'damage-block');
  } else {
    const enemyCrit = Math.random() < 0.12;
    if (enemyCrit) dmg = Math.round(dmg * 1.5);
    addLog(enemyCrit
      ? `💜 <span class="log-enemy">${currentEnemy.name} lands a CRUSHING BLOW! ${dmg} damage!</span>`
      : `💜 <span class="log-enemy">${currentEnemy.name} attacks for ${dmg}.</span>`, 'enemy');
    showDamage('player-card', `-${dmg}`, 'damage-enemy');
  }
  f.playerHP = Math.max(0, f.playerHP - dmg);
  animateCard('enemy-card', 'attacking');
  setTimeout(() => animateCard('player-card', 'taking-hit'), 200);
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
  const xpGain = currentEnemy.xpReward;
  state.totalXP += xpGain;
  addLog(`☠ <span class="log-death">${currentEnemy.name} slain!</span>`, 'death');
  addLog(`<span class="log-win">🏆 VICTORY! +${xpGain} XP!</span>`, 'win');
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

function animateCard(cardId, animClass) {
  const el = document.getElementById(cardId);
  el.classList.add(animClass);
  setTimeout(() => el.classList.remove(animClass), 500);
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

const XP_SKILL_COST = { stamina:15, attack:15, defense:15, crit:20 };

function switchShopTab(tab) {
  document.querySelectorAll('.shop-tab').forEach((t,i) => {
    t.classList.toggle('active', ['name','skins','skills','special'][i] === tab);
  });
  document.querySelectorAll('.shop-section').forEach(s => s.classList.remove('active'));
  document.getElementById('tab-' + tab).classList.add('active');
  if (tab === 'skins') renderSkinGrid();
  if (tab === 'skills') renderXPSkillList();
  if (tab === 'special') renderOneStrikeShop();
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
  document.getElementById('xp-skill-list').innerHTML = SKILLS.map(skill => {
    const cost = XP_SKILL_COST[skill];
    return `<div class="xp-level-row">
      <span class="xp-level-name">${icons[skill]} ${skill.charAt(0).toUpperCase()+skill.slice(1)} LV ${state.skills[skill]}</span>
      <span class="xp-level-cost">${cost} XP</span>
      <button class="btn-buy btn-buy-gold" style="width:auto;padding:0.25rem 0.7rem;margin-left:0.4rem;"
        onclick="buySkillLevel('${skill}')" ${state.totalXP>=cost?'':'disabled'}>+1 LV</button>
    </div>`;
  }).join('');
}

function buySkillLevel(skill) {
  const cost = XP_SKILL_COST[skill];
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
    }
  };
  try {
    localStorage.setItem(SAVE_KEY, JSON.stringify(data));
    showSaveToast('✔ Progress saved!', '#27ae60');
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
})();

function toggleHTP(btn) {
  btn.classList.toggle('open');
  document.getElementById('htp-body').classList.toggle('open');
}

</script>

</body>
</html>
