<!DOCTY>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCI Dashboard - Format Feuille PPT</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        /* Optimisation pour Format Feuille 16:9 sans scroll */
        * { box-sizing: border-box; }
        body, html { 
            height: 100vh; width: 100vw; margin: 0; padding: 0; 
            overflow: hidden; /* Important pour l'intégration PPT sans barre de défilement */
            font-family: 'Segoe UI', sans-serif; background-color: #f0f2f5;
            transform-origin: top center;
            transition: transform 0.2s ease-in-out;
        }
        
        /* Conteneur principal simulant une "feuille A4 paysage" */
        .presentation-container {
            display: grid;
            grid-template-rows: auto auto 1fr;
            height: 100vh;
            width: 100vw;
            padding: 1vw; /* Padding proportionnel à la largeur de l'écran */
            gap: 1vh; /* Gap proportionnel à la hauteur de l'écran */
        }

        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #1a365d;
            color: white;
            padding: 0.8vh 2vw; /* Condensé */
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        /* Titre réduit */
        h1 { margin: 0; font-size: 1.2rem; } /* Taille de police réduite */

        /* KPI Cards */
        .kpi-row {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 1vw;
            margin-bottom: 0.5vh;
        }

        .card {
            background: white;
            padding: 0.8vh 1vw; /* Condensé */
            border-radius: 6px;
            text-align: center;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
            border-bottom: 3px solid #3182ce; /* Bordure affinée */
        }

        .card span { font-size: 0.7rem; color: #718096; text-transform: uppercase; font-weight: bold; }
        .card b { display: block; font-size: 1.3rem; color: #2d3748; margin-top: 2px; }

        /* Zone principale : Graphique + Détails */
        .main-content {
            display: grid;
            grid-template-columns: 2fr 1fr;
            gap: 1vw;
            min-height: 0;
            flex: 1; /* Pour occuper tout l'espace restant */
        }

        .chart-box {
            background: white;
            padding: 1vw;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
            display: flex;
            flex-direction: column;
            height: 100%; /* Important pour maintainAspectRatio */
        }

        .details-box {
            display: grid;
            grid-template-rows: 1fr 1.2fr;
            gap: 1vh;
            height: 100%;
        }

        .sub-card {
            background: white;
            padding: 1vw;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
            display: flex;
            flex-direction: column;
            justify-content: space-around;
        }

        table { width: 100%; border-collapse: collapse; margin-top: 5px; font-size: 0.8rem; }
        th, td { text-align: left; padding: 6px; border-bottom: 1px solid #edf2f7; }
        th { color: #718096; }

        .renta-badge {
            background: #ebf8ff;
            color: #3182ce;
            padding: 3px 6px;
            border-radius: 4px;
            font-weight: bold;
        }
    </style>
</head>
<body>

<div class="presentation-container">
    <header>
        <h1>Performance SCI Biens A+B</h1>
        <div style="font-size: 0.8rem; opacity: 0.8;">Simulation sur 10 ans (Indexation 2%/an)</div>
    </header>

    <div class="kpi-row">
        <div class="card">
            <span>Investissement</span>
            <b>121 000 €</b>
        </div>
        <div class="card" style="border-color: #38a169;">
            <span>Revenu Net Mensuel</span>
            <b>808 €</b>
        </div>
        <div class="card" style="border-color: #d69e2e;">
            <span>Renta. Nette Initiale</span>
            <b>8.01 %</b>
        </div>
        <div class="card" style="border-color: #805ad5;">
            <span>Cash-flow 10 ans</span>
            <b id="totalCumule">0 €</b>
        </div>
    </div>

    <div class="main-content">
        <div class="chart-box">
            <h3 style="margin:0 0 5px 0; font-size: 0.9rem; color: #1a365d;">Évolution : Rentabilité & Cumul Cash-flow</h3>
            <div style="flex:1; min-height: 0;">
                <canvas id="sciChart"></canvas>
            </div>
        </div>

        <div class="details-box">
            <div class="sub-card">
                <span style="font-weight: bold; color: #1a365d; font-size: 0.85rem;">Composition Patrimoine Initial</span>
                <table>
                    <tr><th>Bien</th><th>Valeur</th><th>Loyer Net</th><th>Renta.</th></tr>
                    <tr><td>Bien A</td><td>73 000 €</td><td>550 €</td><td><span class="renta-badge">9.04%</span></td></tr>
                    <tr><td>Bien B</td><td>48 000 €</td><td>258 €</td><td><span class="renta-badge">6.45%</span></td></tr>
                </table>
            </div>
            <div class="sub-card">
                <span style="font-weight: bold; color: #1a365d; font-size: 0.85rem;">Projection Clé (Année 10)</span>
                <table>
                    <tr><td>Revenu annuel An 10</td><td style="text-align: right; font-weight: bold;">11 588 €</td></tr>
                    <tr><td>Rentabilité / coût An 10</td><td style="text-align: right; font-weight: bold;">9.58 %</td></tr>
                    <tr><td>Plus-value latente (+2%/an)</td><td style="text-align: right; font-weight: bold;">+26 500 €</td></tr>
                </table>
            </div>
        </div>
    </div>
</div>

<script>
    const totalPatrimoine = 121000;
    let loyerAnnuel = 9696;
    let cumul = 0;

    const labels = [];
    const dataRenta = [];
    const dataCumule = [];

    for (let i = 1; i <= 10; i++) {
        labels.push("An " + i);
        let renta = (loyerAnnuel / totalPatrimoine) * 100;
        cumul += loyerAnnuel;
        
        dataRenta.push(renta.toFixed(2));
        dataCumule.push(Math.round(cumul));
        
        loyerAnnuel *= 1.02; // Indexation 2%
    }

    document.getElementById('totalCumule').innerText = Math.round(cumul).toLocaleString() + " €";

    const ctx = document.getElementById('sciChart').getContext('2d');
    new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [
                {
                    label: 'Revenu Cumulé (€)',
                    data: dataCumule,
                    borderColor: '#d69e2e',
                    backgroundColor: 'rgba(214, 158, 46, 0.1)',
                    fill: true,
                    yAxisID: 'y1',
                    order: 1,
                    tension: 0.3
                },
                {
                    label: 'Rentabilité (%)',
                    data: dataRenta,
                    borderColor: '#3182ce',
                    borderWidth: 3,
                    yAxisID: 'y',
                    order: 2,
                    tension: 0.3
                }
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: {
                legend: { position: 'bottom', labels: { boxWidth: 10, font: { size: 10 } } }
            },
            scales: {
                y: { position: 'left', title: { display: true, text: 'Renta %', font: { size: 10 } } },
                y1: { position: 'right', grid: { drawOnChartArea: false }, title: { display: true, text: 'Cumul Cash (€)', font: { size: 10 } } }
            }
        }
    });
</script>

</body>
</html>
