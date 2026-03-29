<!SCI THE SHADY GROVE>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCI Dashboard - Fix Présentation</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        /* CONFIGURATION POUR ÉVITER LE DÉCALAGE */
        * { box-sizing: border-box; }
        
        body, html { 
            margin: 0; padding: 0; 
            width: 100%; height: 100%;
            font-family: 'Segoe UI', sans-serif;
            background-color: #f0f2f5;
            /* Permet le scroll si le zoom est trop grand */
            overflow: auto; 
            display: flex;
            justify-content: center; /* Centre horizontalement */
            align-items: flex-start; /* Aligne en haut */
        }
        
        #main-container {
            width: 95%;
            max-width: 1300px;
            margin: 20px auto;
            padding: 20px;
            background: white;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            display: flex;
            flex-direction: column;
            gap: 15px;
            /* Utilisation de zoom pour PowerPoint */
            zoom: 1; 
        }

        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #1a365d;
            color: white;
            padding: 15px 25px;
            border-radius: 10px;
        }

        /* Boutons de Zoom fixes en haut à droite de l'écran */
        .zoom-controls { 
            position: fixed; top: 10px; right: 10px; z-index: 9999; 
            background: white; padding: 5px; border-radius: 8px; 
            box-shadow: 0 4px 12px rgba(0,0,0,0.2);
            display: flex; gap: 5px;
        }
        .zoom-controls button { 
            padding: 8px 15px; cursor: pointer; border: 1px solid #cbd5e0; 
            background: white; border-radius: 5px; font-weight: bold; font-size: 14px;
        }
        .zoom-controls button:hover { background: #f7fafc; }

        /* KPI */
        .kpi-row {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 15px;
        }
        .card {
            background: #fff; padding: 15px; border-radius: 10px; text-align: center;
            border: 1px solid #edf2f7; border-bottom: 4px solid #3182ce;
        }
        .card span { font-size: 0.8em; color: #718096; text-transform: uppercase; font-weight: bold; }
        .card b { display: block; font-size: 1.6em; color: #2d3748; margin-top: 5px; }

        /* Contenu */
        .main-content {
            display: grid;
            grid-template-columns: 2fr 1fr;
            gap: 15px;
        }

        .chart-box {
            background: #fff; padding: 20px; border-radius: 12px;
            border: 1px solid #edf2f7; min-height: 400px;
        }

        .details-box { display: flex; flex-direction: column; gap: 15px; }
        .sub-card {
            background: #f8fafc; padding: 18px; border-radius: 12px;
            border: 1px solid #edf2f7; flex: 1;
        }

        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { text-align: left; padding: 10px; border-bottom: 1px solid #e2e8f0; font-size: 0.9em; }
        .renta-badge { background: #3182ce; color: white; padding: 3px 8px; border-radius: 5px; font-weight: bold; }
    </style>
</head>
<body>

<div class="zoom-controls">
  <button onclick="changerZoom(0.05)">+</button>
  <button onclick="changerZoom(-0.05)">-</button>
  <button onclick="resetZoom()">100%</button>
</div>

<div id="main-container">
    <header>
        <h1 style="margin:0; font-size: 1.5em;">Analyse SCI : Biens A & B</h1>
        <div style="text-align: right;">
            <div style="font-weight: bold;">Patrimoine : 121 000 €</div>
            <div style="font-size: 0.8em; opacity: 0.8;">Simulation sur 10 ans</div>
        </div>
    </header>

    <div class="kpi-row">
        <div class="card">
            <span>Investissement</span>
            <b>121 000 €</b>
        </div>
        <div class="card" style="border-color: #38a169;">
            <span>Loyer Mensuel Net</span>
            <b>808 €</b>
        </div>
        <div class="card" style="border-color: #d69e2e;">
            <span>Rentabilité Nette</span>
            <b>8.01 %</b>
        </div>
        <div class="card" style="border-color: #805ad5;">
            <span>Cumul (10 ans)</span>
            <b id="totalCumule">0 €</b>
        </div>
    </div>

    <div class="main-content">
        <div class="chart-box">
            <canvas id="sciChart"></canvas>
        </div>

        <div class="details-box">
            <div class="sub-card">
                <span style="font-weight: bold; color: #1a365d;">Composition</span>
                <table>
                    <tr><th>Bien</th><th>Prix</th><th>Renta.</th></tr>
                    <tr><td>Bien A</td><td>73k €</td><td><span class="renta-badge">9.0%</span></td></tr>
                    <tr><td>Bien B</td><td>48k €</td><td><span class="renta-badge">6.5%</span></td></tr>
                </table>
            </div>
            <div class="sub-card" style="background: #1a365d; color: white;">
                <span style="font-weight: bold; color: #63b3ed;">Focus Année 10</span>
                <p style="margin: 15px 0 5px 0;">Loyer Annuel : <b>11 588 €</b></p>
                <p style="margin: 5px 0;">Revenus totaux : <b style="color: #63b3ed;">+106 169 €</b></p>
            </div>
        </div>
    </div>
</div>

<script>
    let niveauZoom = 1.0;
    const conteneur = document.getElementById('main-container');

    function changerZoom(delta) {
        niveauZoom += delta;
        conteneur.style.zoom = niveauZoom;
    }

    function resetZoom() {
        niveauZoom = 1.0;
        conteneur.style.zoom = 1;
    }

    // Calculs (Identiques à vos données)
    const totalPatrimoine = 121000;
    let loyerAnnuel = 9696;
    let cumul = 0;
    const labels = [], dataRenta = [], dataCumule = [];

    for (let i = 1; i <= 10; i++) {
        labels.push("An " + i);
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
                    label: 'Cumul des Loyers (€)',
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
            plugins: { legend: { position: 'bottom' } },
            scales: {
                y: { position: 'left', title: { display: true, text: 'Rentabilité (%)' } },
                y1: { position: 'right', grid: { drawOnChartArea: false }, title: { display: true, text: 'Cumul (€)' } }
            }
        }
    });
</script>

</body>
</html>
