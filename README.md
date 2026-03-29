<!SCI THE SHADY GROVE>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard SCI - Correction Alignement</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        * { box-sizing: border-box; }
        
        body, html { 
            margin: 0; padding: 0; 
            font-family: 'Segoe UI', sans-serif;
            background-color: #f0f2f5;
            /* On enlève l'overflow hidden pour permettre de voir le bas si besoin */
        }
        
        #main-container {
            display: flex;
            flex-direction: column;
            width: 100%;
            max-width: 1400px; /* Largeur max pour garder l'aspect présentation */
            margin: 0 auto;
            padding: 15px;
            gap: 15px;
            /* La fonction zoom sera appliquée ici par le JS */
        }

        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #1a365d;
            color: white;
            padding: 12px 25px;
            border-radius: 10px;
        }

        h1 { margin: 0; font-size: 1.6em; }

        /* Boutons de Zoom corrigés */
        .zoom-controls { 
            position: fixed; top: 20px; right: 30px; z-index: 9999; 
            background: white; padding: 5px; border-radius: 8px; 
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }
        .zoom-controls button { 
            padding: 6px 15px; cursor: pointer; border: 1px solid #cbd5e0; 
            background: white; border-radius: 5px; font-weight: bold;
        }
        .zoom-controls button:hover { background: #f7fafc; }

        /* KPI Cards */
        .kpi-row {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 15px;
        }
        .card {
            background: white; padding: 15px; border-radius: 10px; text-align: center;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05); border-bottom: 4px solid #3182ce;
        }
        .card span { font-size: 0.8em; color: #718096; text-transform: uppercase; font-weight: bold; }
        .card b { display: block; font-size: 1.6em; color: #2d3748; margin-top: 5px; }

        /* Main Content */
        .main-content {
            display: grid;
            grid-template-columns: 2fr 1fr;
            gap: 15px;
        }

        .chart-box {
            background: white; padding: 20px; border-radius: 12px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
            min-height: 450px; /* Assure une belle taille au graph */
        }

        .details-box { display: flex; flex-direction: column; gap: 15px; }

        .sub-card {
            background: white; padding: 18px; border-radius: 12px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05); flex: 1;
        }

        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { text-align: left; padding: 10px; border-bottom: 1px solid #edf2f7; font-size: 0.9em; }
        .renta-badge { background: #ebf8ff; color: #3182ce; padding: 3px 8px; border-radius: 5px; font-weight: bold; }
    </style>
</head>
<body>

<div class="zoom-controls">
  <button onclick="changerZoom(0.1)">Zoom +</button>
  <button onclick="changerZoom(-0.1)">Zoom -</button>
  <button onclick="resetZoom()">Réinitialiser</button>
</div>

<div id="main-container">
    <header>
        <h1>Analyse Patrimoniale SCI : Biens A & B</h1>
        <div style="font-size: 1em; opacity: 0.8;">Format Présentation 16:9</div>
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
            <span>Cumul sur 10 ans</span>
            <b id="totalCumule">0 €</b>
        </div>
    </div>

    <div class="main-content">
        <div class="chart-box">
            <canvas id="sciChart"></canvas>
        </div>

        <div class="details-box">
            <div class="sub-card">
                <span style="font-weight: bold; color: #1a365d;">Répartition Biens</span>
                <table>
                    <tr><th>Bien</th><th>Valeur</th><th>Renta.</th></tr>
                    <tr><td>Bien A</td><td>73 000 €</td><td><span class="renta-badge">9.04%</span></td></tr>
                    <tr><td>Bien B</td><td>48 000 €</td><td><span class="renta-badge">6.45%</span></td></tr>
                </table>
            </div>
            <div class="sub-card" style="background: #f8fafc;">
                <span style="font-weight: bold; color: #1a365d;">Projection Finale</span>
                <p style="margin: 10px 0 5px 0; font-size: 0.9em;">Revenu Annuel (An 10) : <b>11 588 €</b></p>
                <p style="margin: 5px 0; font-size: 0.9em;">Plus-value estimée : <b>+26 500 €</b></p>
                <hr style="border: 0; border-top: 1px solid #e2e8f0; margin: 10px 0;">
                <p style="font-size: 0.8em; color: #718096;">Simulation basée sur une indexation annuelle de 2%.</p>
            </div>
        </div>
    </div>
</div>

<script>
    // GESTION DU ZOOM CORRIGÉE
    let niveauZoom = 1.0;
    const conteneur = document.getElementById('main-container');

    function changerZoom(delta) {
        niveauZoom += delta;
        // On utilise la propriété zoom qui gère mieux l'alignement que le scale
        conteneur.style.zoom = niveauZoom;
    }

    function resetZoom() {
        niveauZoom = 1.0;
        conteneur.style.zoom = 1.0;
    }

    // CALCULS ET GRAPHIQUE
    const totalPatrimoine = 121000;
    let loyerAnnuel = 9696;
    let cumul = 0;
    const labels = [], dataRenta = [], dataCumule = [];

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
                    label: 'Cumul des Loyers (€)',
                    data: dataCumule,
                    borderColor: '#d69e2e',
                    backgroundColor: 'rgba(214, 158, 46, 0.1)',
                    fill: true,
                    yAxisID: 'y1'
                },
                {
                    label: 'Rentabilité (%)',
                    data: dataRenta,
                    borderColor: '#3182ce',
                    borderWidth: 3,
                    yAxisID: 'y'
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
