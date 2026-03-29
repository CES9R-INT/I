<!!THE SHADY GROVE-SCI>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Analyse SCI - Format Standard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { 
            font-family: 'Segoe UI', sans-serif; 
            background: #f4f7f9; 
            margin: 0; 
            padding: 20px; 
            color: #2d3748;
            transform-origin: top center;
            transition: transform 0.2s;
        }
        .container { 
            max-width: 900px; 
            margin: auto; 
            background: white; 
            padding: 25px; 
            border-radius: 12px; 
            box-shadow: 0 8px 20px rgba(0,0,0,0.06); 
        }
        header { text-align: center; margin-bottom: 20px; border-bottom: 1px solid #eee; padding-bottom: 10px; }
        h1 { font-size: 1.4rem; color: #1a365d; margin: 0; }
        
        /* Grille des chiffres clés */
        .kpi-grid { 
            display: grid; 
            grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); 
            gap: 15px; 
            margin-bottom: 25px; 
        }
        .card { 
            background: #fff; 
            padding: 15px; 
            border-radius: 8px; 
            text-align: center; 
            border: 1px solid #e2e8f0;
            border-top: 4px solid #3182ce;
        }
        .card span { font-size: 0.75rem; color: #718096; font-weight: bold; text-transform: uppercase; }
        .card b { display: block; font-size: 1.4rem; color: #2d3748; margin-top: 5px; }

        /* Zone Graphique */
        .chart-container { 
            position: relative; 
            height: 400px; 
            margin-bottom: 25px; 
            padding: 10px;
            border: 1px solid #f0f0f0;
            border-radius: 8px;
        }

        /* Tableaux */
        .details-section { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        .sub-card { background: #fdfdfd; padding: 15px; border-radius: 8px; border: 1px solid #eee; }
        table { width: 100%; border-collapse: collapse; font-size: 0.85rem; }
        th, td { text-align: left; padding: 8px; border-bottom: 1px solid #f0f0f0; }
        
        /* Zoom fixe */
        .zoom-bar {
            position: fixed; top: 10px; right: 10px; z-index: 100;
            background: white; padding: 5px; border-radius: 5px; border: 1px solid #ccc;
        }
        .zoom-bar button { cursor: pointer; padding: 5px 10px; font-weight: bold; }

        @media (max-width: 600px) { .details-section { grid-template-columns: 1fr; } }
    </style>
</head>
<body>

<div class="zoom-bar">
    <button onclick="zoom(0.1)">+</button>
    <button onclick="zoom(-0.1)">-</button>
</div>

<div class="container">
    <header>
        <h1>Performance Patrimoniale SCI (Biens A & B)</h1>
    </header>

    <div class="kpi-grid">
        <div class="card"><span>Patrimoine Total</span><b>121 000 €</b></div>
        <div class="card" style="border-top-color: #38a169;"><span>Revenu Mensuel Net</span><b>808 €</b></div>
        <div class="card" style="border-top-color: #d69e2e;"><span>Rentabilité Nette</span><b>8.01 %</b></div>
        <div class="card" style="border-top-color: #805ad5;"><span>Cumul 10 ans</span><b id="txtCumul">0 €</b></div>
    </div>

    <div class="chart-container">
        <canvas id="mainChart"></canvas>
    </div>

    <div class="details-section">
        <div class="sub-card">
            <h3 style="margin-top:0; font-size: 0.9rem;">Détail Actifs</h3>
            <table>
                <tr><th>Bien</th><th>Prix</th><th>Loyer Net</th></tr>
                <tr><td>Appart. A</td><td>73 000 €</td><td>550 €</td></tr>
                <tr><td>Appart. B</td><td>48 000 €</td><td>258 €</td></tr>
            </table>
        </div>
        <div class="sub-card">
            <h3 style="margin-top:0; font-size: 0.9rem;">Projections</h3>
            <p style="font-size: 0.8rem; line-height: 1.4; color: #4a5568;">
                • Indexation : <b>2% / an</b><br>
                • Revenu Année 10 : <b>~965 €/mois</b><br>
                • Cash-flow total : <b>> 100k€ sur 10 ans</b>
            </p>
        </div>
    </div>
</div>

<script>
    let currentZoom = 1.0;
    function zoom(v) { currentZoom += v; document.body.style.transform = `scale(${currentZoom})`; }

    const totalVal = 121000;
    let loyerAn = 9696;
    let cumul = 0;
    const labels = [], dRenta = [], dCumul = [];

    for (let i = 1; i <= 10; i++) {
        labels.push("An " + i);
        let renta = (loyerAn / totalVal) * 100;
        cumul += loyerAn;
        dRenta.push(renta.toFixed(2));
        dCumul.push(Math.round(cumul));
        loyerAn *= 1.02;
    }

    document.getElementById('txtCumul').innerText = Math.round(cumul).toLocaleString() + " €";

    new Chart(document.getElementById('mainChart'), {
        type: 'line',
        data: {
            labels: labels,
            datasets: [
                { label: 'Cumul Cash (€)', data: dCumul, borderColor: '#d69e2e', backgroundColor: 'rgba(214,158,46,0.1)', fill: true, yAxisID: 'y1' },
                { label: 'Renta (%)', data: dRenta, borderColor: '#3182ce', borderWidth: 3, yAxisID: 'y' }
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
                y: { position: 'left', title: { display: true, text: '%' } },
                y1: { position: 'right', grid: { display: false }, title: { display: true, text: 'Euros' } }
            }
        }
    });
</script>

</body>
</html>
