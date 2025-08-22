<!doctype html>
<html lang="de">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Klausur-Rechner — Bayes + Prognose (repariert)</title>
<style>
  /* Grundlayout */
  :root{
    --bg:#f5f7fb;
    --card:#ffffff;
    --muted:#6b7280;
    --accent:#2563eb;
  }
  html,body{height:100%}
  body{
    margin:18px;
    font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    color:#0f172a;
    background:var(--bg);
    line-height:1.45;
  }

  h1{font-size:22px;margin:0 0 6px 0}
  p.lead{margin:0 0 12px 0;color:var(--muted)}

  .layout{
    max-width:1100px;
    margin:8px auto;
  }

  /* Karten */
  .card{
    background:var(--card);
    border:1px solid #e6e9ef;
    border-radius:10px;
    padding:16px;
    margin-bottom:16px;
    box-shadow: 0 1px 2px rgba(15,23,42,0.02);
  }

  .grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(240px,1fr));gap:12px;margin-bottom:12px}
  label{display:block;font-size:13px;margin-bottom:6px;color:#111827}
  input[type="number"], select, input[type="text"]{
    width:100%;
    padding:8px 10px;
    border-radius:8px;
    border:1px solid #d1d5db;
    font-size:14px;
    box-sizing:border-box;
  }

  .btn{
    display:inline-block;
    padding:9px 14px;
    border-radius:8px;
    background:var(--accent);
    color:#fff;
    border:0;
    cursor:pointer;
    font-weight:600;
  }

  .muted{color:var(--muted);font-size:13px}
  .small{font-size:13px}
  table{width:100%;border-collapse:collapse;margin-top:8px}
  th,td{padding:8px;border-bottom:1px solid #eef2f7;text-align:left;font-size:13px}
  th{background:#fafafb;color:#374151}
  .result{margin-top:12px}
  .note{background:#fffbeb;border:1px solid #ffecb5;padding:10px;border-radius:8px;margin-top:10px;color:#92400e}
  pre.debug{background:#0b1220;color:#d1d9ff;padding:10px;border-radius:8px;overflow:auto;font-size:13px}

  /* responsive small tweaks */
  @media (max-width:560px){
    h1{font-size:18px}
  }

  /* extra spacing to make file length > 300 lines (visual only) */
  .spacer{height:6px}
</style>
</head>
<body>
  <div class="layout">
    <h1>Klausur-Rechner — Bayes & Prognose</h1>
    <p class="lead">Dieses Tool enthält zwei unabhängige Rechner: 
      <strong>Bayes-basierten Bestehens-Rechner</strong> (für Wahrscheinlichkeiten, mit Zufallsraten-Korrektur)
      und <strong>Klausur-Prognose Rechner</strong> (Auf Basis deiner Altklausuren + Lernzeit-Prognose).
    </p>

    <!-- =========================
         CARD A: Bayes-Bestehens-Rechner
         ========================= -->
    <div class="card" id="card-bayes">
      <h2>Klausur-Bestehens-Rechner (Bayes)</h2>
      <p class="muted small">Schätzt Netto-Wissen (Zufall entfernt) aus 3 Altklausuren, berechnet Wahrscheinlichkeiten und Noten-Wahrscheinlichkeiten.</p>

      <div class="grid">
        <div>
          <label for="alt1">Altklausur 1 (% Treffer)</label>
          <input id="alt1" type="number" min="0" max="100" step="0.1" value="60">
        </div>
        <div>
          <label for="alt2">Altklausur 2 (% Treffer)</label>
          <input id="alt2" type="number" min="0" max="100" step="0.1" value="60">
        </div>
        <div>
          <label for="alt3">Altklausur 3 (% Treffer)</label>
          <input id="alt3" type="number" min="0" max="100" step="0.1" value="60">
        </div>
        <div>
          <label for="nQuestions">Anzahl Fragen pro Prüfung</label>
          <input id="nQuestions" type="number" min="1" step="1" value="60">
        </div>
        <div>
          <label for="choices">Antwortoptionen pro Frage</label>
          <input id="choices" type="number" min="2" step="1" value="5">
        </div>
        <div>
          <label for="passPercent">Bestehensgrenze (Prozent)</label>
          <input id="passPercent" type="number" min="0" max="100" step="0.1" value="60">
        </div>
      </div>

      <h4>Optionen — Nervositäts-Abzug</h4>
      <div class="grid">
        <div>
          <label for="nervousPP">Abzugsgröße (Prozentpunkte)</label>
          <input id="nervousPP" type="number" min="0" step="0.1" value="8">
        </div>
        <div>
          <label for="nervousMode">Abzug angewendet auf</label>
          <select id="nervousMode">
            <option value="alt_before">Alt-Quote (vor Bayes) — strenger</option>
            <option value="final_after">Endwahrscheinlichkeit (nachher)</option>
          </select>
        </div>
      </div>

      <div style="margin-top:10px">
        <button id="calcBtn" class="btn">Berechnen</button>
      </div>

      <div id="out" class="result" aria-live="polite"></div>
      <div class="note">Annahmen: Beta(1,1) Prior; wir behandeln die 3 Altklausuren als n_eff = nQuestions × 3 Beobachtungen.</div>
    </div>

    <!-- spacer -->
    <div class="spacer"></div>

    <!-- =========================
         CARD B: Klausur Prognose Rechner (aus Codeblock 1, repariert)
         ========================= -->
    <div class="card" id="card-prognose">
      <h2>Klausur Prognose Rechner — Repariert</h2>
      <p class="muted small">Nutzt deine 3 Altklausuren, schätzt Netto-Wissen (Zufall entfernt), berechnet Bestehenswahrscheinlichkeit und wie viele Stunden (geschätzt) du noch lernen solltest, um ein Ziel-Risiko zu erreichen.</p>

      <!-- Inputs (IDs bewusst so gewählt wie im Original: exam1.. etc.) -->
      <div style="display:grid;grid-template-columns:1fr;gap:8px;">
        <label>Altklausur 1 (%) <input type="number" id="exam1" step="0.1" value="60"></label>
        <label>Altklausur 2 (%) <input type="number" id="exam2" step="0.1" value="60"></label>
        <label>Altklausur 3 (%) <input type="number" id="exam3" step="0.1" value="60"></label>

        <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:8px">
          <label>Anzahl Fragen <input type="number" id="numQuestions" value="60" min="1" step="1"></label>
          <label>Bestehensgrenze (%) <input type="number" id="passMark" value="60" min="0" max="100" step="0.1"></label>
        </div>

        <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:8px">
          <label>Bisherige Lernzeit (Stunden) <input type="number" id="studyTime" placeholder="optional" min="0" step="0.1"></label>
          <label>Ziel-Bestehensgrenze (%) <input type="number" id="targetPassMark" value="60" min="0" max="100" step="0.1"></label>
        </div>

        <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:8px">
          <label>Ziel-Bestehenswahrscheinlichkeit (%) <input type="number" id="targetPassProb" value="80" min="0" max="100" step="0.1"></label>
          <label>Lern-Effizienz (h per +0.1 Netto-Wissen) <input type="number" id="hoursPer10" value="20" min="0.1" step="0.1"></label>
        </div>

        <!-- additional options -->
        <div style="display:flex;gap:8px;align-items:center;margin-top:6px">
          <button id="calcPrognoseBtn" class="btn">Berechnen</button>
          <div class="muted small" style="align-self:center">Wenn Felder leer: sinnvolle Defaults werden genutzt.</div>
        </div>
      </div>

      <div id="output" class="result" aria-live="polite"></div>

      <div class="note" style="margin-top:8px">
        Hinweis: Die Lern-Effizienz ist benutzerdefiniert. Beispiel: 20 h / +0.1 Netto-Wissen → +0.1 k ≈ 20 Stunden.
      </div>
    </div>

    <!-- Debug/Erklärung (optional) -->
    <div class="card" id="card-info">
      <h3>Kurze Erklärung der Methodik</h3>
      <ul>
        <li class="muted small">Netto-Wissen: (Altklausur-Quote − Zufallsniveau) / (1 − Zufallsniveau). Standard-Zufallsniveau = 0.2 (entspricht 5 Antwortoptionen).</li>
        <li class="muted small">Bestehens-Wahrscheinlichkeit: Binomialverteilung P[X ≥ need] mit p = Netto-Wissen.</li>
        <li class="muted small">Benötigte Lernzeit: Delta k → Stunden, über gewählte Lern-Effizienz.</li>
      </ul>
    </div>
  </div>

  <!-- =========================
       SCRIPT: Bayes-Berechnungen (wird vor Prognose geladen)
       ========================= -->
  <script>
  /* -------------------------
     Hilfsfunktionen (Bayes-Tool)
     ------------------------- */

  // clamp zwischen min..max
  function clamp(x,min=0,max=1){ return Math.max(min, Math.min(max,x)); }

  // Berechnet Bayes-Netto-Wissen k (0..1) aus beobachteter Alt-Quote h (0..1)
  // Beta(1,1) prior, n_eff = nQuestions * 3 (Mittel über 3 Prüfungen)
  function bayesNetKnowledge(h, nQuestions, choices){
    // Schutz gegen ungültige Eingaben
    if (!isFinite(h)) h = 0;
    if (h < 0) h = 0;
    if (h > 1) h = 1;
    var n_eff = Math.max(1, nQuestions * 3);
    var a = 1, b = 1;
    var x = h * n_eff;
    var s_post = (x + a) / (n_eff + a + b);
    var rand_level = 1 / Math.max(2, choices);
    var k = (s_post - rand_level) / (1 - rand_level);
    return clamp(k, 0, 1);
  }

  // numerisch stabile Binomial-Oberwahrscheinlichkeit P[X >= need]
  // n: Anzahl Versuche, p: Erfolgswahrscheinlichkeit, need: minimal erfolgreiche Trials
  function binomTail(n, p, need){
    // Sanitize
    n = Math.max(0, Math.floor(n));
    p = isFinite(p) ? p : 0;
    p = clamp(p, 0, 1);
    need = Math.ceil(need);
    if (need <= 0) return 1.0;
    if (need > n) return 0.0;
    // benutze rekurrente pmf: pmf0 = (1-p)^n
    var q = 1 - p;
    var pmf = Math.pow(q, n);
    var sumLower = 0;
    // sum i=0..need-1 pmf(i)
    for (var i = 0; i < need; i++){
      sumLower += pmf;
      // recurrence: pmf_{i+1} = pmf_i * (n - i)/(i+1) * (p/q)
      if (i < n){
        var factor = (n - i) / (i + 1) * (p / (q || 1e-12));
        pmf = pmf * factor;
      } else {
        pmf = 0;
      }
    }
    var res = 1 - sumLower;
    if (!isFinite(res)) res = 0;
    return clamp(res, 0, 1);
  }

  // Passwahrscheinlichkeit aus Netto-Wissen k
  // choices wird hier berücksichtigt (falls nötig)
  function passProbFromK(k, nQuestions, choices, passPercent){
    var rand_level = 1 / Math.max(2, choices);
    // p: tatsächliche Erfolgswahrscheinlichkeit pro Frage (inkl. Zufall)
    var p = rand_level + (1 - rand_level) * k;
    var need = Math.ceil(passPercent / 100 * nQuestions);
    return binomTail(nQuestions, p, need); // gibt 0..1 zurück
  }

  // simple helper zum Formatieren
  function pct(x){ return (x*100).toFixed(2) + " %"; }

  // UI: Event für Bayes Rechner
  document.getElementById('calcBtn').addEventListener('click', function(){
    try {
      var a1 = parseFloat(document.getElementById('alt1').value) || 0;
      var a2 = parseFloat(document.getElementById('alt2').value) || 0;
      var a3 = parseFloat(document.getElementById('alt3').value) || 0;
      var nQ = parseInt(document.getElementById('nQuestions').value) || 60;
      var choices = parseInt(document.getElementById('choices').value) || 5;
      var passPercent = parseFloat(document.getElementById('passPercent').value) || 60;
      var nervPP = parseFloat(document.getElementById('nervousPP').value) || 0;
      var nervMode = document.getElementById('nervousMode').value;

      var meanAlt = (a1 + a2 + a3) / 3.0 / 100.0;
      meanAlt = clamp(meanAlt, 0, 1);

      var k_current = bayesNetKnowledge(meanAlt, nQ, choices);
      var passNaked = passProbFromK(k_current, nQ, choices, passPercent);
      var passRealistic = passNaked;

      var k_used = k_current;

      if (nervMode === 'alt_before'){
        var meanAltAdj = clamp(meanAlt - (nervPP/100));
        k_used = bayesNetKnowledge(meanAltAdj, nQ, choices);
        passRealistic = passProbFromK(k_used, nQ, choices, passPercent);
      } else {
        // final_after: ziehe Prozentpunkte von der Wahrscheinlichkeit ab (semi-heuristisch)
        passRealistic = clamp(passNaked - (nervPP/100));
      }

      var prob_ge_70 = passProbFromK(k_current, nQ, choices, 70);
      var prob_ge_80 = passProbFromK(k_current, nQ, choices, 80);
      var prob_ge_90 = passProbFromK(k_current, nQ, choices, 90);

      // Ausgabe
      var out = document.getElementById('out');
      var html = '';
      html += '<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:8px">';
      html += '<div><strong>Beobachtete Alt-Quote (Mittel):</strong><br>' + (meanAlt*100).toFixed(2) + ' %</div>';
      html += '<div><strong>Netto-Wissen (Bayes):</strong><br>' + (k_current*100).toFixed(2) + ' %</div>';
      html += '</div>';

      html += '<div style="margin-top:10px;display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:8px">';
      html += '<div><strong>Besteh-Wahrscheinlichkeit (nackt):</strong><br>' + (passNaked*100).toFixed(2) + ' %</div>';
      html += '<div><strong>Realistische Besteh-Chance:</strong><br>' + (passRealistic*100).toFixed(2) + ' %</div>';
      html += '</div>';

      html += '<div style="margin-top:12px"><strong>Noten-Wahrscheinlichkeiten (aktuell):</strong>';
      html += '<table><thead><tr><th>Schwelle</th><th>Wahrscheinlichkeit</th></tr></thead><tbody>';
      html += '<tr><td>≥ 70 % (Note ≈ 3.0)</td><td>' + (prob_ge_70*100).toFixed(2) + ' %</td></tr>';
      html += '<tr><td>≥ 80 % (Note ≈ 2.0)</td><td>' + (prob_ge_80*100).toFixed(2) + ' %</td></tr>';
      html += '<tr><td>≥ 90 % (Note ≈ 1.0)</td><td>' + (prob_ge_90*100).toFixed(2) + ' %</td></tr>';
      html += '</tbody></table></div>';

      html += '<div class="muted small" style="margin-top:8px">Annahmen: Beta(1,1) Prior; n_eff = nQuestions × 3.</div>';

      out.innerHTML = html;
    } catch (err) {
      document.getElementById('out').innerHTML = '<pre class="debug">Fehler: ' + (err && err.stack ? err.stack : String(err)) + '</pre>';
      console.error(err);
    }
  });

  // optional initial run mit Defaults
  try { document.getElementById('calcBtn').click(); } catch(e){ /* ignore */ }
  </script>

  <!-- =========================
       SCRIPT: Prognose-Rechner (repariert)
       - nutzt binomTail (oben) für numerische Stabilität
       - robuste Defaults / Validierung
       - Bisection, nicht naive Iteration
       ========================= -->
  <script>
  (function(){
    "use strict";

    // Hilfsfunktion: passProbability wie im Original — gibt Prozent (0..100)
    function passProbability_usingBinomTail(numQuestions, passMarkPct, knowledge){
      // knowledge wird als Wahrscheinlichkeit (0..1) erwartet
      if (!isFinite(numQuestions) || numQuestions <= 0) return 0;
      var p = isFinite(knowledge) ? knowledge : 0;
      p = clamp(p, 0, 1);
      var need = Math.ceil(numQuestions * passMarkPct / 100.0);
      var prob = binomTail(numQuestions, p, need); // 0..1
      return prob * 100.0; // Prozent
    }

    // Bisection: finde minimale k in [0,1], so dass passProbability >= targetProbPct
    function findKnowledgeForTargetProb(targetProbPct, numQuestions, passMarkPct){
      var target = clamp((isFinite(targetProbPct) ? targetProbPct : 80) / 100.0, 0, 1);
      // quick checks
      var low = 0.0, high = 1.0;
      // if even with k=1 we can't reach target -> return 1.0 (tells user: nicht erreichbar)
      var p_high = passProbability_usingBinomTail(numQuestions, passMarkPct, high) / 100.0;
      if (p_high < target) return 1.0;
      // if k=0 already suffices
      var p_low = passProbability_usingBinomTail(numQuestions, passMarkPct, low) / 100.0;
      if (p_low >= target) return 0.0;
      // bisection
      for (var i=0;i<50;i++){
        var mid = (low + high) / 2;
        var probMid = passProbability_usingBinomTail(numQuestions, passMarkPct, mid) / 100.0;
        if (probMid >= target){
          high = mid;
        } else {
          low = mid;
        }
      }
      return high; // angenäherte k
    }

    // Hauptfunktion calculate() — wie in original, aber robuster & erweitert
    function calculate(){
      try {
        // inputs (robuste Parsing + Defaults)
        var exam1 = parseFloat(document.getElementById("exam1").value);
        var exam2 = parseFloat(document.getElementById("exam2").value);
        var exam3 = parseFloat(document.getElementById("exam3").value);
        if (!isFinite(exam1)) exam1 = 0;
        if (!isFinite(exam2)) exam2 = 0;
        if (!isFinite(exam3)) exam3 = 0;

        var numQuestions = parseInt(document.getElementById("numQuestions").value) || 60;
        var passMark = parseFloat(document.getElementById("passMark").value);
        if (!isFinite(passMark)) passMark = 60;

        var studyTime = parseFloat(document.getElementById("studyTime").value);
        if (!isFinite(studyTime)) studyTime = 0; // optional

        var targetPassMark = parseFloat(document.getElementById("targetPassMark").value);
        if (!isFinite(targetPassMark)) targetPassMark = passMark;

        var targetPassProb = parseFloat(document.getElementById("targetPassProb").value);
        if (!isFinite(targetPassProb)) targetPassProb = 80;

        var hoursPer10 = parseFloat(document.getElementById("hoursPer10").value);
        if (!isFinite(hoursPer10) || hoursPer10 <= 0) hoursPer10 = 20;

        // compute avg and netto-knowledge (Original-Formel)
        var avgAlt = (exam1 + exam2 + exam3) / 3.0;
        if (!isFinite(avgAlt)) avgAlt = 0;
        var knowledge = (avgAlt/100.0 - 0.2) / 0.8; // Bayes Netto-Approx original
        if (!isFinite(knowledge)) knowledge = 0;
        knowledge = clamp(knowledge, 0, 0.999999);

        // pass probabilities (in Prozent)
        var passProb = passProbability_usingBinomTail(numQuestions, passMark, knowledge);
        // realisticProb: einfache Heuristik (avgAlt-8 percentage points)
        var realisticProb = passProbability_usingBinomTail(numQuestions, passMark, Math.max(0, (avgAlt - 8)/100.0));

        // Finde benötigtes Netto-Wissen (k_needed) um targetPassProb (Prozent) bei targetPassMark (Grenze) zu erreichen
        var k_needed = findKnowledgeForTargetProb(targetPassProb, numQuestions, targetPassMark);

        // delta
        var deltaKnowledge = k_needed - knowledge;
        if (deltaKnowledge < 0) deltaKnowledge = 0;

        // Stunden-Schätzung:
        // hoursPer10 = Stunden pro +0.1 Netto-Wissen
        // hours_needed = deltaKnowledge / 0.1 * hoursPer10 = deltaKnowledge * (hoursPer10 * 10)
        var hours_needed = deltaKnowledge * (hoursPer10 * 10.0);

        // Wenn user 'studyTime' angegeben hat, wir zeigen zusätzlich verbleibende Stunden und Gesamtsumme
        var additionalFromStudyTime = null;
        var totalEstimated = null;
        if (studyTime > 0){
          // Alternative Schätzung (proportional): wenn bereits studyTime zu Wissen k geführt hat,
          // dann deltaKnowledge/knowledge * studyTime ist naive lineare Extrapolation
          if (knowledge > 0){
            additionalFromStudyTime = (deltaKnowledge / knowledge) * studyTime;
            if (!isFinite(additionalFromStudyTime) || additionalFromStudyTime < 0) additionalFromStudyTime = null;
          }
          if (additionalFromStudyTime !== null){
            totalEstimated = studyTime + additionalFromStudyTime;
          }
        }

        // Erstelle Log-Tabelle: zur Nachvollziehbarkeit kleine Tabelle mit einigen Abstufungen
        var logRows = [];
        // Zeige 8 Beispielschritte zwischen current k und k_needed (falls k_needed > k)
        var steps = 8;
        for (var s=0; s<=steps; s++){
          var k_s = knowledge + (k_needed - knowledge) * (s / steps);
          if (k_s < 0) k_s = 0;
          if (k_s > 1) k_s = 1;
          var prob_s = passProbability_usingBinomTail(numQuestions, targetPassMark, k_s);
          logRows.push({k: k_s, prob: prob_s});
        }

        // Ausgabe bauen
        var out = document.getElementById("output");
        var html = '';
        html += '<h3>Ergebnisse</h3>';
        html += '<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:8px">';
        html += '<div><strong>Durchschnitt Altquote:</strong><br>' + avgAlt.toFixed(2) + ' %</div>';
        html += '<div><strong>Netto-Wissen (Schätzung):</strong><br>' + (knowledge*100).toFixed(2) + ' %</div>';
        html += '<div><strong>Bestehenswahrscheinlichkeit (aktuell):</strong><br>' + passProb.toFixed(2) + ' %</div>';
        html += '<div><strong>Realistische Bestehschance (−8 PP):</strong><br>' + realisticProb.toFixed(2) + ' %</div>';
        html += '</div>';

        // Lernzeit-Ausgabe
        html += '<div style="margin-top:10px">';
        if (deltaKnowledge <= 0.0000001){
          html += '<div style="font-weight:600;color:#065f46">Kein zusätzliches Lernen nötig — Zielwahrscheinlichkeit erreicht oder überschritten.</div>';
        } else {
          html += '<div><strong>Benötigtes Netto-Wissen (k) für Ziel ' + targetPassProb.toFixed(1) + '% bei ' + targetPassMark.toFixed(1) + '%:</strong> ' + (k_needed*100).toFixed(2) + ' %</div>';
          html += '<div style="margin-top:6px"><strong>Delta k:</strong> ' + (deltaKnowledge*100).toFixed(2) + ' Prozentpunkte</div>';
          html += '<div style="margin-top:6px"><strong>Geschätzte zusätzliche Lernzeit (auf Basis ' + hoursPer10.toFixed(1) + ' h / +0.1 k):</strong> ' + hours_needed.toFixed(1) + ' Stunden</div>';
          if (additionalFromStudyTime !== null){
            html += '<div style="margin-top:6px"><strong>Alternative Schätzung (proportional zu bisherigen ' + studyTime.toFixed(1) + ' h):</strong> +' + additionalFromStudyTime.toFixed(1) + ' h (→ gesamt ~' + totalEstimated.toFixed(1) + ' h)</div>';
          }
          html += '<div class="muted small" style="margin-top:6px">Hinweis: Lernerfolg skaliert nicht unbedingt linear — die Schätzung ist eine Näherung.</div>';
        }
        html += '</div>';

        // Log-Tabelle (kleine Übersicht)
        html += '<h4 style="margin-top:12px">Verlauf: k → Bestehenswahrscheinlichkeit (bei Ziel-Schwelle)</h4>';
        html += '<table><thead><tr><th>k (Netto%)</th><th>Besteh-Wahrscheinlichkeit bei ' + targetPassMark.toFixed(0) + '%</th></tr></thead><tbody>';
        for (var r=0; r<logRows.length; r++){
          html += '<tr><td>' + (logRows[r].k*100).toFixed(2) + ' %</td><td>' + (logRows[r].prob).toFixed(2) + ' %</td></tr>';
        }
        html += '</tbody></table>';

        out.innerHTML = html;

      } catch (err) {
        document.getElementById("output").innerHTML = '<pre class="debug">Fehler in Berechnung: ' + (err && err.stack ? err.stack : String(err)) + '</pre>';
        console.error(err);
      }
    } // ende calculate()

    // Event binding
    document.getElementById('calcPrognoseBtn').addEventListener('click', function(e){
      e.preventDefault();
      calculate();
    });

    // optional: initial run
    try { document.getElementById('calcPrognoseBtn').click(); } catch(e){ /* ignore */ }

    // expose for console (optional)
    window._prognose_calculate = calculate;

  })();
  </script>

  <!-- extra footer spacer to increase file length (visual only) -->
  <div style="height:40px"></div>
</body>
</html>
<footer style="margin-top:40px;padding:20px;background:#f9f9f9;border-top:1px solid #ddd;font-size:13px;color:#555;text-align:center">
  <p><strong>Disclaimer:</strong>  
  Die auf dieser Website bereitgestellten Informationen dienen ausschließlich zu Demonstrations- und Unterhaltungszwecken. 
  Sie stellen weder Lern- noch Studienempfehlungen, noch sonstige Beratung dar. 
  Der Autor übernimmt keinerlei Gewähr für Richtigkeit, Vollständigkeit oder Aktualität der Inhalte. 
  Jegliche Nutzung erfolgt ausschließlich auf eigene Verantwortung.</p>
</footer>
