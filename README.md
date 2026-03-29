<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard SCI - Format Présentation</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        * { box-sizing: border-box; }
        body, html { 
            margin: 0; padding: 0; 
            font-family: 'Segoe UI', sans-serif;
            background-color: #f0f2f5;
            overflow-x: hidden; /* Empêche le défilement horizontal lors du zoom */
        }
        
        /* Conteneur principal adaptable */
        #zoom-target {
            display: flex;
            flex-direction: column;
            min-height: 100vh;
            padding: 10px 20px;
            gap: 10px;
            transform-origin: top center;
            transition: transform 0.2s ease-in-out;
        }

        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #1a365d;
            color: white;
            padding: 8px 20px;
            border-radius: 8px;
        }

        h1 { margin: 0; font-size: 1.4em; }

        /* Boutons de Zoom */
        .zoom-controls { 
            position: fixed; top: 15px; right: 20px; z-index: 9999; 
            background: rgba(255,255,255,0.9); padding: 5px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .zoom-controls button { 
            padding: 5px 12px; margin-left: 2px; cursor: pointer; 
            border: 1px solid #cbd5e0; background: white; border-radius: 4px; 
            font-weight: bold; font-size: 12px;
        }
        .zoom-controls button:hover { background: #edf2f7; }

        /* KPI Cards */
        .kpi-row {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 10px;
        }
        .card {
            background: white; padding: 10px; border-radius: 8px; text-align: center;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05); border-bottom: 3px solid #3182ce;
        }
        .card span { font-size: 0.7em; color: #718096; text-transform: uppercase; }
        .card b { display: block; font-size: 1.3em; color: #2d3748; margin-top: 2px; }

        /* Main Content */
        .main-content {
            display: grid;
            grid-template-columns: 1.8fr 1fr;
            gap: 10px;
            flex: 1; /* Prend l'espace restant */
        }

        .chart-box {
            background: white; padding: 10px; border-radius: 10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
            display: flex; flex-direction: column;
            min-height: 350px;
        }

        .details-box {
            display: flex; flex-direction: column; gap: 10px;
        }

        .sub-card {
            background: white; padding: 12px; border-radius: 10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05); flex: 1;
        }

        table { width: 100%; border-collapse: collapse; font-size: 0.8em; }
        th, td { text-align: left; padding: 6px; border-bottom: 1px solid #edf2f7; }
        .renta-badge { background: #ebf8ff; color: #3182ce; padding: 2px 6px; border-radius: 4px; font-weight: bold; }
    </style>
</head>
<body>

<div class="zoom-controls">
  <button onclick="ajusterZoom(0.05)">Zoom +</button>
  <button onclick="ajusterZoom(-0.05)">Zoom -</button>
  <button onclick="resetZoom()">100%</button>
</div>

<div id="zoom-target">
    <header>
        <h1>Tableau de Bord SCI : Performance 10 ans</h1>
        <div style="font-size: 0.8em; opacity: 0.9;">Simulation Biens A & B (Valeur 121k€)</div>
    </header>

    <div class="kpi-row">
        <div class="card">
            <span>Capital Engagé</span>
            <b>121 000 €</b>
        </div>
        <div class="card" style="border-color: #38a169;">
            <span>Loyer Net Mensuel</span>
            <b>808 €</b>
        </div>
        <div class="card" style="border-color: #d69e2e;">
            <span>Rentabilité Nette</span>
            <b>8.01 %</b>
        </div>
        <div class="card" style="border-color: #805ad5;">
            <span>Revenus Cumulés (10 ans)</span>
            <b id="totalCumule">0 €</b>
        </div>
    </div>

    <div class="main-content">
        <div class="chart-box">
            <canvas id="sciChart"></canvas>
        </div>

        <div class="details-box">
            <div class="sub-card">
                <span style="font-weight: bold; font-size: 0.9em; color: #1a365d;">Répartition du Patrimoine</span>
                <table>
                    <tr><th>Bien</th><th>Prix</th><th>Loyer</th><th>Renta.</th></tr>
                    <tr><td>Bien A</td><td>73 000 €</td><td>550 €</td><td><span class="renta-badge">9.0%</span></td></tr>
                    <tr><td>Bien B</td><td>48 000 €</td><td>258 €</td><td><span class="renta-badge">6.5%</span></td></tr>
                </table>
            </div>
            <div class="sub-card" style="background: #f8fafc;">
                <span style="font-weight: bold; font-size: 0.9em; color: #1a365d;">Perspectives An 10</span>
                <table>
                    <tr><td>Loyer Annuel projeté</td><td style="text-align: right;"><b>11 588 €</b></td></tr>
                    <tr><td>Renta. sur coût</td><td style="text-align: right;"><b>9.58 %</b></td></tr>
                    <tr><td>Cumul des loyers</td><td style="text-align: right; color: #805ad5;"><b>+106 169 €</b></td></tr>
                </table>
                <p style="font-size: 0.7em; color: #718096; margin-top: 10px;">* Basé sur une indexation annuelle de 2%.</p>
            </div>
        </div>
    </div>
</div>

<script>
    let currentZoom = 1.0;
    const zoomTarget = document.getElementById('zoom-target');

    function ajusterZoom(delta) {
        currentZoom += delta;
        zoomTarget.style.transform = `scale(${currentZoom})`;
        zoomTarget.style.width = (100 / currentZoom) + "%"; // Ajuste la largeur pour éviter le blanc
    }

    function resetZoom() {
        currentZoom = 1.0;
        zoomTarget.style.transform = `scale(1)`;
        zoomTarget.style.width = "100%";
    }

    // Données de calcul
    const totalPatrimoine = 121000;
    let loyerAnnuel = 9696;
    let cumul = 0;
    const labels = [];
    const dataRenta = [];
    const dataCumule = [];

    for (let i = 1; i <= 10; i++) {
        labels.push("Année " + i);
        let renta = (loyerAnnuel / totalPatrimoine) * 100;
        cumul += loyerAnnuel;
        dataRenta.push(renta.toFixed(2));
        dataCumule.push(Math.round(cumul));
        loyerAnnuel *= 1.02;
    }

    document.getElementById('totalCumule').innerText = Math.round(cumul).toLocaleString() + " €";

    const ctx = document.getElementById('sciChart').getContext('2d');
    new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [
                {
                    label: 'Revenus Cumulés (€)',
                    data: dataCumule,
                    borderColor: '#d69e2e',
                    backgroundColor: 'rgba(214, 158, 46, 0.1)',
                    fill: true,
                    yAxisID: 'y1',
                    tension: 0.3
                },
                {
                    label: 'Rentabilité (%)',
                    data: dataRenta,
                    borderColor: '#3182ce',
                    borderWidth: 3,
                    yAxisID: 'y',
                    tension: 0.3
                }
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { position: 'bottom', labels: { boxWidth: 12, font: { size: 11 } } } },
            scales: {
                y: { position: 'left', title: { display: true, text: 'Rentabilité (%)', font: { size: 10 } } },
                y1: { position: 'right', grid: { drawOnChartArea: false }, title: { display: true, text: 'Cumul (€)', font: { size: 10 } } }
            }
        }
    });
</script>

</body>
</html>
