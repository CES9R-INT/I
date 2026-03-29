<!DOC>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCI Dashboard - Full Width 16:9</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        /* Reset complet pour occuper tout l'écran */
        * { box-sizing: border-box; }
        html, body { 
            height: 100%; width: 100%; margin: 0; padding: 0; 
            overflow: hidden; /* Empêche le scroll pour rester en mode présentation */
            font-family: 'Segoe UI', sans-serif; background-color: #f0f2f5;
        }

        /* Conteneur principal qui réagit au zoom */
        #page-wrapper {
            display: flex;
            flex-direction: column;
            height: 100vh;
            width: 100vw;
            padding: 15px;
            gap: 10px;
            transition: zoom 0.2s ease; /* Transition fluide */
        }

        header {
            background: #1a365d; color: white; padding: 10px 20px;
            border-radius: 8px; display: flex; justify-content: space-between; align-items: center;
        }

        .kpi-row { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
        .card { 
            background: white; padding: 10px; border-radius: 8px; text-align: center;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05); border-bottom: 4px solid #3182ce;
        }
        .card span { font-size: 0.75em; color: #718096; text-transform: uppercase; font-weight: bold; }
        .card b { display: block; font-size: 1.5em; color: #2d3748; }

        /* Disposition 16:9 */
        .main-layout { display: grid; grid-template-columns: 1.5fr 1fr; gap: 10px; flex: 1; min-height: 0; }
        .chart-box { background: white; padding: 15px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); display: flex; flex-direction: column; }
        .details-box { display: grid; grid-template-rows: 1fr 1.2fr; gap: 10px; }
        .sub-card { background: white; padding: 15px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); overflow: hidden; }

        table { width: 100%; border-collapse: collapse; font-size: 0.85em; }
        th, td { text-align: left; padding: 6px; border-bottom: 1px solid #edf2f7; }

        /* Contrôles de Zoom fixés en bas à droite pour ne pas gêner */
        .zoom-controls {
            position: fixed; bottom: 20px; right: 20px; z-index: 9999;
            background: white; padding: 5px; border-radius: 30px; box-shadow: 0 4px 10px rgba(0,0,0,0.2);
        }
        .zoom-controls button {
            padding: 8px 15px; border: none; background: #1a365d; color: white;
            border-radius: 20px; cursor: pointer; font-weight: bold; margin: 2px;
        }
        .zoom-controls button:hover { background: #3182ce; }
    </style>
</head>
<body>

<div class="zoom-controls">
  <button onclick="changeZoom(0.1)">Zoom +</button>
  <button onclick="changeZoom(-0.1)">Zoom -</button>
  <button onclick="resetZoom()">100%</button>
</div>

<div id="page-wrapper">
    <header>
        <h1 style="margin:0; font-size: 1.4em;">SCI Patrimoine - Présentation 16:9</h1>
        <div style="font-size: 0.9em; background: rgba(255,255,255,0.1); padding: 5px 15px; border-radius: 20px;">Investissement : 121 000 €</div>
    </header>

    <div class="kpi-row">
        <div class="card"><span>Revenu Mensuel</span><b>808 €</b></div>
        <div class="card" style="border-color: #38a169;"><span>Renta. Nette</span><b>8.01 %</b></div>
        <div class="card" style="border-color: #d69e2e;"><span>Cumul 10 ans</span><b id="kpiCumul">0 €</b></div>
        <div class="card" style="border-color: #805ad5;"><span>Performance</span><b>+22% sur 10 ans</b></div>
    </div>

    <div class="main-layout">
        <div class="chart-box">
            <h3 style="margin:0 0 10px 0; font-size: 1em; color: #1a365d;">Évolution : Rentabilité & Cumul Cash-flow</h3>
            <div style="flex:1; min-height: 0;">
                <canvas id="sciChart"></canvas>
            </div>
        </div>

        <div class="details-box">
            <div class="sub-card">
                <h3 style="margin:0 0 10px 0; font-size: 0.9em;">Composition</h3>
                <table>
                    <tr style="background: #f8fafc"><th>Bien</th><th>Achat</th><th>Loyer</th><th>Renta.</th></tr>
                    <tr><td>Appartement A</td><td>73 000 €</td><td>550 €</td><td style="color: #3182ce; font-weight: bold;">9.04%</td></tr>
                    <tr><td>Appartement B</td><td>48 000 €</td><td>258 €</td><td style="color: #3182ce; font-weight: bold;">6.45%</td></tr>
                </table>
            </div>
            <div class="sub-card">
                <h3 style="margin:0 0 10px 0; font-size: 0.9em;">Flux de Trésorerie (Projection 10 ans)</h3>
                <table id="fluxTable">
                    <thead><tr style="background: #f8fafc"><th>Année</th><th>Annuel</th><th>Cumulé</th></tr></thead>
                    <tbody id="fluxBody"></tbody>
                </table>
            </div>
        </div>
    </div>
</div>

<script>
    let currentZoom = 1.0;
    const wrapper = document.getElementById('page-wrapper');

    function changeZoom(delta) {
        currentZoom += delta;
        wrapper.style.zoom = currentZoom;
    }

    function resetZoom() {
        currentZoom = 1.0;
        wrapper.style.zoom = 1;
    }

    // Données SCI
    const totalPatrimoine = 121000;
    let loyerMensuel = 808;
    let cumul = 0;
    const labels = [];
    const dataRenta = [];
    const dataCumule = [];
    const fluxBody = document.getElementById('fluxBody');

    for (let i = 1; i <= 10; i++) {
        let loyerAnnuel = loyerMensuel * 12;
        let renta = (loyerAnnuel / totalPatrimoine) * 100;
        cumul += loyerAnnuel;

        labels.push("An " + i);
        dataRenta.push(renta.toFixed(2));
        dataCumule.push(Math.round(cumul));

        // Remplissage tableau (Seulement quelques lignes pour garder la place)
        if (i === 1 || i === 5 || i === 10) {
            fluxBody.innerHTML += `<tr><td>An ${i}</td><td>${Math.round(loyerAnnuel).toLocaleString()} €</td><td style="font-weight:bold;">${Math.round(cumul).toLocaleString()} €</td></tr>`;
        }

        loyerMensuel *= 1.02; // Indexation
    }

    document.getElementById('kpiCumul').innerText = Math.round(cumul).toLocaleString() + " €";

    // Chart.js
    const ctx = document.getElementById('sciChart').getContext('2d');
    new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [
                { label: 'Cumul (€)', data: dataCumule, borderColor: '#d69e2e', backgroundColor: 'rgba(214,158,46,0.1)', fill: true, yAxisID: 'y1', pointRadius: 4 },
                { label: 'Renta (%)', data: dataRenta, borderColor: '#3182ce', tension: 0.3, yAxisID: 'y', pointRadius: 4 }
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { position: 'top', labels: { boxWidth: 10, font: { size: 11 } } } },
            scales: {
                y: { position: 'left', title: { display: true, text: 'Renta %' } },
                y1: { position: 'right', grid: { drawOnChartArea: false }, title: { display: true, text: 'Cumul Cash' } }
            }
        }
    });
</script>

</body>
</html>
