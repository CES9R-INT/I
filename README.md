<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulateur Immo - 90k€ avec Zoom</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { 
            font-family: 'Segoe UI', sans-serif; 
            background: #f4f7f9; 
            padding: 20px; 
            color: #2d3748; 
            transform-origin: top center; /* Pour que le zoom reste centré en haut */
            transition: transform 0.2s ease-in-out;
        }
        .container { max-width: 1000px; margin: auto; background: white; padding: 30px; border-radius: 15px; box-shadow: 0 10px 25px rgba(0,0,0,0.08); }
        h1 { text-align: center; color: #1a365d; margin-bottom: 30px; }
        .grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; margin-bottom: 25px; }
        .input-group { background: #fff; padding: 12px; border: 1px solid #e2e8f0; border-radius: 8px; }
        .input-group.highlight { border-left: 4px solid #3182ce; }
        .input-group.highlight-alt { border-left: 4px solid #ed8936; }
        label { display: block; font-size: 0.8em; font-weight: bold; margin-bottom: 5px; color: #718096; }
        input[type="number"] { width: 100%; border: 1px solid #cbd5e0; padding: 8px; border-radius: 4px; outline: none; }
        
        .sliders-container { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 25px; }
        .slider-section { background: #ebf8ff; padding: 20px; border-radius: 12px; text-align: center; border: 1px solid #bee3f8; }
        input[type="range"] { width: 100%; margin-top: 15px; cursor: pointer; }
        
        .mensualite-card { 
            background: #2d3748; color: white; padding: 20px; border-radius: 10px; 
            margin-bottom: 25px; text-align: center; box-shadow: inset 0 2px 4px rgba(0,0,0,0.3);
        }
        .mensualite-card span { font-size: 0.9em; text-transform: uppercase; letter-spacing: 1px; color: #a0aec0; }
        .mensualite-card b { display: block; font-size: 2.2em; color: #63b3ed; margin-top: 5px; }

        .results { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 25px; }
        .card { padding: 15px; border-radius: 10px; text-align: center; color: white; }
        .bg-blue { background: #3182ce; }
        .bg-orange { background: #ed8936; }
        .val { font-size: 1.6em; font-weight: bold; display: block; }
        canvas { background: white; border-radius: 10px; padding: 10px; border: 1px solid #e2e8f0; }

        /* Style des boutons de zoom */
        .zoom-controls button {
            padding: 8px 12px;
            margin-left: 5px;
            cursor: pointer;
            border: 1px solid #cbd5e0;
            background: white;
            border-radius: 5px;
            font-weight: bold;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        .zoom-controls button:hover { background: #edf2f7; }
    </style>
</head>
<body>

<div class="zoom-controls" style="position: fixed; top: 10px; right: 10px; z-index: 9999;">
  <button onclick="ajusterZoom(0.1)">Zoom +</button>
  <button onclick="ajusterZoom(-0.1)">Zoom -</button>
</div>

<div class="container">
    <h1>Simulation Projet 90 000 €</h1>
    
    <div class="grid">
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
    </div>

    <div class="sliders-container">
        <div class="slider-section">
            <label>VOTRE APPORT : <span id="apportVal" style="color: #2b6cb0;">16000</span> €</label>
            <input type="range" id="apport" min="5000" max="90000" step="500" value="16000" oninput="update()">
            <div style="margin-top: 10px; font-size: 0.8em; color: #4a5568;">
                Emprunt : <span id="empruntVal">74000</span> €
            </div>
        </div>

        <div class="slider-section" style="background: #faf5ff; border-color: #e9d8fd;">
            <label>TAUX D'INTÉRÊT : <span id="tauxVal" style="color: #6b46c1;">4.5</span> %</label>
            <input type="range" id="taux" min="0" max="10" step="0.1" value="4.5" oninput="update()">
            <div style="margin-top: 10px; font-size: 0.8em; color: #4a5568;">
                Durée fixe : 20 ans
            </div>
        </div>
    </div>

    <div class="mensualite-card">
        <span>Mensualité du crédit</span>
        <b id="mensDisplay">0 €</b>
    </div>

    <div class="results">
        <div class="card bg-blue">
            <span>Rentabilité Scénario A</span>
            <span class="val" id="resA">0 %</span>
        </div>
        <div class="card bg-orange">
            <span>Rentabilité Scénario B</span>
            <span class="val" id="resB">0 %</span>
        </div>
    </div>

    <canvas id="compChart"></canvas>
</div>

<script>
    let chart;
    let currentZoom = 1.0;

    // Fonction de Zoom
    function ajusterZoom(delta) {
        currentZoom += delta;
        document.body.style.transform = `scale(${currentZoom})`;
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

        // Graphique
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
            scales: {
                y: { title: { display: true, text: 'Rentabilité (%)' } },
                x: { title: { display: true, text: 'Apport (€)' } }
            }
        }
    });

    update();
</script>

</body>
</html>
