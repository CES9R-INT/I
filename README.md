<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulateur Immo 16:9 - 90k€</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        /* Configuration de base pour le 16:9 */
        body { 
            font-family: 'Segoe UI', sans-serif; 
            background: #f4f7f9; 
            margin: 0;
            padding: 20px;
            color: #2d3748; 
            display: flex;
            justify-content: center;
            align-items: flex-start;
            min-height: 100vh;
            transform-origin: top center;
            transition: transform 0.2s ease-in-out;
        }

        .main-wrapper {
            display: grid;
            grid-template-columns: 350px 1fr; /* Colonne gauche fixe, droite flexible */
            gap: 20px;
            max-width: 1400px;
            width: 100%;
            background: white;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.08);
        }

        h1 { grid-column: 1 / -1; text-align: center; color: #1a365d; margin: 0 0 10px 0; font-size: 1.5em; }

        /* Colonne de Gauche : Inputs & Sliders */
        .sidebar { display: flex; flex-direction: column; gap: 15px; }

        .input-group { background: #fff; padding: 10px; border: 1px solid #e2e8f0; border-radius: 8px; }
        .input-group.highlight { border-left: 4px solid #3182ce; }
        .input-group.highlight-alt { border-left: 4px solid #ed8936; }
        label { display: block; font-size: 0.75em; font-weight: bold; margin-bottom: 5px; color: #718096; }
        input[type="number"] { width: 90%; border: 1px solid #cbd5e0; padding: 6px; border-radius: 4px; outline: none; }
        
        .slider-section { background: #ebf8ff; padding: 15px; border-radius: 10px; text-align: center; border: 1px solid #bee3f8; }
        .slider-section.alt { background: #faf5ff; border-color: #e9d8fd; }
        input[type="range"] { width: 100%; margin-top: 10px; cursor: pointer; }

        /* Colonne de Droite : Résultats & Graphique */
        .content { display: flex; flex-direction: column; gap: 15px; }

        .mensualite-card { 
            background: #2d3748; color: white; padding: 15px; border-radius: 10px; 
            text-align: center; box-shadow: inset 0 2px 4px rgba(0,0,0,0.3);
        }
        .mensualite-card b { display: block; font-size: 2em; color: #63b3ed; }

        .results-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .card { padding: 10px; border-radius: 10px; text-align: center; color: white; font-size: 0.9em; }
        .bg-blue { background: #3182ce; }
        .bg-orange { background: #ed8936; }
        .val { font-size: 1.4em; font-weight: bold; display: block; }

        canvas { background: white; border-radius: 10px; padding: 10px; border: 1px solid #e2e8f0; max-height: 350px; }

        /* Zoom controls */
        .zoom-controls { position: fixed; top: 10px; right: 10px; z-index: 9999; }
        .zoom-controls button { padding: 5px 10px; margin-left: 5px; cursor: pointer; border: 1px solid #cbd5e0; background: white; border-radius: 5px; font-weight: bold; }
    </style>
</head>
<body>

<div class="zoom-controls">
  <button onclick="ajusterZoom(0.1)">Zoom +</button>
  <button onclick="ajusterZoom(-0.1)">Zoom -</button>
  <button onclick="resetZoom()">100%</button>
</div>

<div class="main-wrapper">
    <h1>Simulation Immobilière - Projet 90 000 €</h1>

    <div class="sidebar">
        <div class="input-group">
            <label>Prix Projet (€)</label>
            <input type="number" id="prix" value="90000" oninput="update()">
        </div>
        <div class="input-group highlight">
            <label>Loyer A (Annuel €)</label>
            <input type="number" id="loyerA" value="7800" oninput="update()">
        </div>
        <div class="input-group highlight-alt">
            <label>Loyer B (Annuel €)</label>
            <input type="number" id="loyerB" value="9600" oninput="update()">
        </div>
        <div class="input-group">
            <label>Charges/Taxes (€)</label>
            <input type="number" id="charges" value="1988" oninput="update()">
        </div>

        <div class="slider-section">
            <label>APPORT : <span id="apportVal" style="color: #2b6cb0;">16000</span> €</label>
            <input type="range" id="apport" min="5000" max="90000" step="500" value="16000" oninput="update()">
            <div style="font-size: 0.75em; margin-top:5px;">Emprunt : <span id="empruntVal">74000</span> €</div>
        </div>

        <div class="slider-section alt">
            <label>TAUX : <span id="tauxVal" style="color: #6b46c1;">4.5</span> %</label>
            <input type="range" id="taux" min="0" max="10" step="0.1" value="4.5" oninput="update()">
            <div style="font-size: 0.75em; margin-top:5px;">Durée : 20 ans</div>
        </div>
    </div>

    <div class="content">
        <div class="mensualite-card">
            <span>Mensualité du crédit</span>
            <b id="mensDisplay">0 €</b>
        </div>

        <div class="results-grid">
            <div class="card bg-blue">
                <span>Renta Scénario A</span>
                <span class="val" id="resA">0 %</span>
            </div>
            <div class="card bg-orange">
                <span>Renta Scénario B</span>
                <span class="val" id="resB">0 %</span>
            </div>
        </div>

        <canvas id="compChart"></canvas>
    </div>
</div>

<script>
    let chart;
    let currentZoom = 1.0;

    function ajusterZoom(delta) {
        currentZoom += delta;
        document.body.style.transform = `scale(${currentZoom})`;
    }
    function resetZoom() {
        currentZoom = 1.0;
        document.body.style.transform = `scale(1)`;
    }

    function calculateMensualite(P, A, T) {
        const borrowed = P - A;
        if (borrowed <= 0) return 0;
        if (T === 0) return borrowed / (20 * 12);
        const i = (T / 100) / 12; //
        const m = 20 * 12;        //
        return borrowed * (i / (1 - Math.pow(1 + i, -m)));
    }

    function calculateRA(P, L, C, A, T) {
        if (A <= 0) return 0; 
        const mens = calculateMensualite(P, A, T);
        const cf_annuel = (L - C) - (mens * 12);
        return (cf_annuel / A) * 100;
    }

    function update() {
        const P = parseFloat(document.getElementById('prix').value) || 0;
        const LA = parseFloat(document.getElementById('loyerA').value) || 0;
        const LB = parseFloat(document.getElementById('loyerB').value) || 0;
        const C = parseFloat(document.getElementById('charges').value) || 0;
        const A = parseFloat(document.getElementById('apport').value) || 0;
        const T = parseFloat(document.getElementById('taux').value) || 0;

        document.getElementById('apportVal').innerText = A.toLocaleString();
        document.getElementById('tauxVal').innerText = T.toFixed(1);
        document.getElementById('empruntVal').innerText = (P - A > 0 ? P - A : 0).toLocaleString();

        const mens = calculateMensualite(P, A, T);
        const raA = calculateRA(P, LA, C, A, T);
        const raB = calculateRA(P, LB, C, A, T);

        document.getElementById('mensDisplay').innerText = Math.round(mens).toLocaleString() + " €";
        document.getElementById('resA').innerText = (A > 0 ? raA.toFixed(2) : "0.00") + " %";
        document.getElementById('resB').innerText = (A > 0 ? raB.toFixed(2) : "0.00") + " %";

        const labels = [];
        const dataA = [];
        const dataB = [];
        for (let step = 5000; step <= P; step += 2500) {
            labels.push(step);
            dataA.push(calculateRA(P, LA, C, step, T));
            dataB.push(calculateRA(P, LB, C, step, T));
        }
        chart.data.labels = labels;
        chart.data.datasets[0].data = dataA;
        chart.data.datasets[1].data = dataB;
        chart.update();
    }

    const ctx = document.getElementById('compChart').getContext('2d');
    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: [],
            datasets: [
                { label: 'Loyer A', data: [], borderColor: '#3182ce', tension: 0.3, pointRadius: 0 },
                { label: 'Loyer B', data: [], borderColor: '#ed8936', tension: 0.3, pointRadius: 0 }
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
                y: { title: { display: true, text: 'Rentabilité (%)', font: { size: 10 } } },
                x: { title: { display: true, text: 'Apport (€)', font: { size: 10 } } }
            }
        }
    });

    update();
</script>

</body>
</html>
