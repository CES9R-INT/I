<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard SCI - Présentation 16:9</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        /* Optimisation pour le format 16:9 sans scroll */
        * { box-sizing: border-box; }
        body, html { 
            height: 100%; margin: 0; padding: 0; overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f2f5;
        }
        
        .presentation-container {
            display: grid;
            grid-template-rows: auto 1fr;
            height: 100vh;
            padding: 15px;
            gap: 15px;
        }

        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #1a365d;
            color: white;
            padding: 10px 25px;
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        h1 { margin: 0; font-size: 1.6em; }

        /* KPI Cards en haut */
        .kpi-row {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 15px;
        }

        .card {
            background: white;
            padding: 12px;
            border-radius: 8px;
            text-align: center;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
            border-bottom: 4px solid #3182ce;
        }

        .card span { font-size: 0.8em; color: #718096; text-transform: uppercase; font-weight: bold; }
        .card b { display: block; font-size: 1.5em; color: #2d3748; margin-top: 5px; }

        /* Zone principale : Graphique + Détails */
        .main-content {
            display: grid;
            grid-template-columns: 2fr 1fr;
            gap: 15px;
            min-height: 0; /* Important pour le scroll interne si besoin */
        }

        .chart-box {
            background: white;
            padding: 15px;
            border-radius: 10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
            display: flex;
            flex-direction: column;
        }

        .details-box {
            display: grid;
            grid-template-rows: 1fr 1fr;
            gap: 15px;
        }

        .sub-card {
            background: white;
            padding: 15px;
            border-radius: 10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
        }

        table { width: 100%; border-collapse: collapse; margin-top: 10px; font-size: 0.85em; }
        th, td { text-align: left; padding: 8px; border-bottom: 1px solid #edf2f7; }
        th { color: #718096; }

        .renta-badge {
            background: #ebf8ff;
            color: #3182ce;
            padding: 4px 8px;
            border-radius: 5px;
            font-weight: bold;
        }
    </style>
</head>
<body>

<div class="presentation-container">
    <header>
        <h1>Analyse Patrimoniale SCI : Biens A & B</h1>
        <div style="font-size: 0.9em; opacity: 0.8;">Simulation sur 10 ans (Indexation 2%/an)</div>
    </header>

    <div class="kpi-row">
        <div class="card">
            <span>Investissement Total</span>
            <b>121 000 €</b>
        </div>
        <div class="card" style="border-color: #38a169;">
            <span>Revenu Net Mensuel</span>
            <b>808 €</b>
        </div>
        <div class="card" style="border-color: #d69e2e;">
            <span>Rentabilité Nette</span>
            <b>8.01 %</b>
        </div>
        <div class="card" style="border-color: #805ad5;">
            <span>Cash-flow 10 ans</span>
            <b id="totalCumule">106 169 €</b>
        </div>
    </div>

    <div class="main-content">
        <div class="chart-box">
            <canvas id="sciChart"></canvas>
        </div>

        <div class="details-box">
            <div class="sub-card">
                <span style="font-weight: bold; color: #1a365d;">Composition du Patrimoine</span>
                <table>
                    <tr><th>Bien</th><th>Valeur</th><th>Loyer Net</th><th>Renta.</th></tr>
                    <tr><td>Bien A</td><td>73 000 €</td><td>550 €</td><td><span class="renta-badge">9.04%</span></td></tr>
                    <tr><td>Bien B</td><td>48 000 €</td><td>258 €</td><td><span class="renta-badge">6.45%</span></td></tr>
                </table>
            </div>
            <div class="sub-card">
                <span style="font-weight: bold; color: #1a365d;">Projection à l'année 10</span>
                <table>
                    <tr><td>Revenu annuel An 10</td><td style="text-align: right; font-weight: bold;">11 588 €</td></tr>
                    <tr><td>Rentabilité sur coût An 10</td><td style="text-align: right; font-weight: bold;">9.58 %</td></tr>
                    <tr><td>Plus-value estimée (2%)</td><td style="text-align: right; font-weight: bold;">+26 500 €</td></tr>
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
    const dataAnnuel = [];
    const dataCumule = [];

    for (let i = 1; i <= 10; i++) {
        labels.push("Année " + i);
        let renta = (loyerAnnuel / totalPatrimoine) * 100;
        cumul += loyerAnnuel;
        
        dataRenta.push(renta.toFixed(2));
        dataAnnuel.push(Math.round(loyerAnnuel));
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
                    tension: 0.3
                },
                {
                    label: 'Rentabilité (%)',
                    data: dataRenta,
                    borderColor: '#3182ce',
                    yAxisID: 'y',
                    tension: 0.3
                }
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: {
                legend: { position: 'bottom' }
            },
            scales: {
                y: { position: 'left', title: { display: true, text: 'Rentabilité (%)' } },
                y1: { position: 'right', grid: { drawOnChartArea: false }, title: { display: true, text: 'Cumul (€)' } }
            }
        }
    });
</script>

</body>
</html>
