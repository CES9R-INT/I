SCI-THE SHADY GROVE
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Analyse SCI Complète</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { 
            font-family: 'Segoe UI', sans-serif; background: #f4f7f9; 
            margin: 0; padding: 20px; color: #2d3748;
            transform-origin: top center; transition: transform 0.2s;
        }
        .container { 
            max-width: 1000px; margin: auto; background: white; 
            padding: 25px; border-radius: 12px; box-shadow: 0 8px 20px rgba(0,0,0,0.06); 
        }
        header { text-align: center; margin-bottom: 20px; border-bottom: 1px solid #eee; padding-bottom: 10px; }
        h1 { font-size: 1.3rem; color: #1a365d; margin: 0; }
        h2 { font-size: 1rem; color: #1a365d; margin: 25px 0 10px 0; border-left: 4px solid #3182ce; padding-left: 10px; }
        
        .kpi-grid { 
            display: grid; grid-template-columns: repeat(4, 1fr); 
            gap: 12px; margin-bottom: 20px; 
        }
        .card { 
            background: #fff; padding: 12px; border-radius: 8px; text-align: center; 
            border: 1px solid #e2e8f0; border-top: 4px solid #3182ce;
        }
        .card span { font-size: 0.7rem; color: #718096; font-weight: bold; text-transform: uppercase; }
        .card b { display: block; font-size: 1.2rem; color: #2d3748; margin-top: 5px; }

        .chart-main { height: 350px; margin-bottom: 20px; }
        .chart-sub { height: 220px; margin-bottom: 25px; padding: 15px; background: #fcfcfc; border-radius: 8px; border: 1px solid #eee; }

        .details-section { display: grid; grid-template-columns: 1.2fr 0.8fr; gap: 20px; margin-bottom: 30px; }
        .sub-card { background: #fdfdfd; padding: 15px; border-radius: 8px; border: 1px solid #eee; }
        
        table { width: 100%; border-collapse: collapse; font-size: 0.85rem; }
        th, td { text-align: left; padding: 10px; border-bottom: 1px solid #f0f0f0; }
        th { background: #f8fafc; }
        .row-total { background: #edf2f7; font-weight: bold; border-top: 2px solid #cbd5e0; }
        .renta-tag { background: #ebf8ff; color: #3182ce; padding: 2px 6px; border-radius: 4px; font-weight: bold; }
        
        /* Section Scénario */
        .scenario-box { background: #f0f9ff; padding: 20px; border-radius: 12px; border: 1px solid #bee3f8; margin-top: 20px; }
        .buy-event { background: #f0fff4; color: #2f855a; font-weight: bold; border-radius: 4px; padding: 2px 6px; font-size: 0.75rem; border: 1px solid #c6f6d5; }

        .zoom-bar { position: fixed; top: 10px; right: 10px; z-index: 100; background: white; border: 1px solid #ccc; padding: 5px; border-radius: 5px; }
        .zoom-bar button { cursor: pointer; padding: 5px 10px; font-weight: bold; border: none; background: #eee; margin: 0 2px; }
    </style>
</head>
<body>

<div class="zoom-bar">
    <button onclick="zoom(0.1)">+</button>
    <button onclick="zoom(-0.1)">-</button>
</div>

<div class="container">
    <header><h1>Wealth Management Reporting: SCI Analysis & Strategies</h1></header>

    <div class="kpi-grid">
        <div class="card"><span>Investissement</span><b>121 000 €</b></div>
        <div class="card" style="border-top-color: #38a169;"><span>Loyer Mensuel</span><b>808 €</b></div>
        <div class="card" style="border-top-color: #d69e2e;"><span>Cumul Cash (10 ans)</span><b>106 169 €</b></div>
        <div class="card" style="border-top-color: #e53e3e;"><span>ROI Global (10 ans)</span><b>87.74 %</b></div>
    </div>

    <h2>1. Return On Investment (CASH FLOW & ROI %)</h2>
    <div class="chart-main">
        <canvas id="chartCumul"></canvas>
    </div>

    <h2>2. Progression de la Rentabilité Annuelle (%)</h2>
    <div class="chart-sub">
        <canvas id="chartRentaLine"></canvas>
    </div>

    <div class="details-section">
        <div class="sub-card">
            <h3>Détail de l'Actif et Totaux</h3>
            <table>
                <thead>
                    <tr><th>Bien</th><th>Prix Achat</th><th>Loyer Net</th><th>Renta.</th></tr>
                </thead>
                <tbody>
                    <tr><td>Appartement A</td><td>73 000 €</td><td>550 €</td><td><span class="renta-tag">9,04 %</span></td></tr>
                    <tr><td>Appartement B</td><td>48 000 €</td><td>258 €</td><td><span class="renta-tag">6,45 %</span></td></tr>
                </tbody>
                <tfoot>
                    <tr class="row-total">
                        <td>TOTAL SCI</td>
                        <td>121 000 €</td>
                        <td>808 €</td>
                        <td>8,01 %</td>
                    </tr>
                </tfoot>
            </table>
        </div>
        <div class="sub-card">
            <h3>PROJECTION</h3>
            <p style="font-size: 0.8rem; color: #4a5568;">
                • Indexation : <b>2% / an</b><br>
                • Calcul sur capital investi (121k€)<br>
                • Projection linéaire .
            </p>
        </div>
    </div>

    <div class="scenario-box">
        <h2 style="margin-top:0; border-left-color: #2f855a;">3. Scénario Dynamique : Réinvestissement (Seuil 50k€)</h2>
        <p style="font-size: 0.85rem; margin-bottom: 15px;">
            Dès que le cash-flow cumulé atteint <b>50 000 €</b>, un nouveau bien (type Bien B) est acquis.
        </p>
        <table>
            <thead>
                <tr><th>Année</th><th>Nombre de Biens</th><th>Loyer Mensuel</th><th>Trésorerie Dispo.</th><th>Événement</th></tr>
            </thead>
            <tbody id="scenarioBody"></tbody>
        </table>
    </div>
</div>

<script>
    let currentZoom = 1.0;
    function zoom(v) { currentZoom += v; document.body.style.transform = `scale(${currentZoom})`; }

    // DONNÉES POUR LES GRAPHS STANDARDS
    const totalInvesti = 121000;
    let loyerAnnuelStd = 9696;
    let cumulStd = 0;
    const labels = [];
    const dataRentaStd = [];
    const dataRoiStd = [];
    const dataCumulStd = [];

    // CALCUL SCÉNARIO RÉINVESTISSEMENT
    let cashScenario = 0;
    let loyerMensuelScen = 808;
    let nbBiens = 2;
    const tableBody = document.getElementById('scenarioBody');

    for (let i = 1; i <= 10; i++) {
        labels.push("An " + i);
        
        // Stats Standard
        let rentaAn = (loyerAnnuelStd / totalInvesti) * 100;
        cumulStd += loyerAnnuelStd;
        dataRentaStd.push(rentaAn.toFixed(2));
        dataRoiStd.push(((cumulStd / totalInvesti) * 100).toFixed(2));
        dataCumulStd.push(Math.round(cumulStd));
        loyerAnnuelStd *= 1.02;

        // Logique Réinvestissement
        cashScenario += (loyerMensuelScen * 12);
        let event = "-";
        if (cashScenario >= 50000) {
            nbBiens++;
            cashScenario -= 50000;
            loyerMensuelScen += 258;
            event = `<span class="buy-event">+1 BIEN ACQUIS</span>`;
        }

        tableBody.innerHTML += `
            <tr>
                <td>Année ${i}</td>
                <td>${nbBiens}</td>
                <td><b>${Math.round(loyerMensuelScen)} €</b></td>
                <td>${Math.round(cashScenario).toLocaleString()} €</td>
                <td>${event}</td>
            </tr>
        `;
        loyerMensuelScen *= 1.02;
    }

    // GRAPH 1
    new Chart(document.getElementById('chartCumul'), {
        type: 'line',
        data: {
            labels: labels,
            datasets: [
                { label: 'ROI Cumulé (%)', data: dataRoiStd, borderColor: '#e53e3e', backgroundColor: 'rgba(229, 62, 62, 0.1)', fill: true, yAxisID: 'y' },
                { label: 'Cash-flow (€)', data: dataCumulStd, borderColor: '#d69e2e', yAxisID: 'y1' }
            ]
        },
        options: { responsive: true, maintainAspectRatio: false }
    });

    // GRAPH 2
    new Chart(document.getElementById('chartRentaLine'), {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                label: 'Rentabilité annuelle (%)',
                data: dataRentaStd,
                borderColor: '#3182ce',
                borderWidth: 3, tension: 0.4, fill: false
            }]
        },
        options: { 
            responsive: true, maintainAspectRatio: false,
            scales: { y: { min: 8, max: 10 } }
        }
    });
</script>

</body>
</html>
