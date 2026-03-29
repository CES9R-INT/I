THE SHADY GROVE SCI
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Analyse SCI - Reporting Complet</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { 
            font-family: 'Segoe UI', sans-serif; 
            background: #f4f7f9; 
            margin: 0; padding: 20px; color: #2d3748;
            transform-origin: top center; transition: transform 0.2s;
        }
        .container { 
            max-width: 950px; margin: auto; background: white; 
            padding: 25px; border-radius: 12px; box-shadow: 0 8px 20px rgba(0,0,0,0.06); 
        }
        header { text-align: center; margin-bottom: 20px; border-bottom: 1px solid #eee; padding-bottom: 10px; }
        h1 { font-size: 1.3rem; color: #1a365d; margin: 0; }
        
        .kpi-grid { 
            display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); 
            gap: 12px; margin-bottom: 20px; 
        }
        .card { 
            background: #fff; padding: 12px; border-radius: 8px; text-align: center; 
            border: 1px solid #e2e8f0; border-top: 4px solid #3182ce;
        }
        .card span { font-size: 0.7rem; color: #718096; font-weight: bold; text-transform: uppercase; }
        .card b { display: block; font-size: 1.3rem; color: #2d3748; margin-top: 5px; }

        .chart-main { height: 350px; margin-bottom: 25px; }
        .chart-sub { height: 220px; margin-bottom: 25px; padding: 15px; background: #fcfcfc; border-radius: 8px; border: 1px solid #eee; }
        
        h2 { font-size: 0.95rem; color: #1a365d; margin-bottom: 10px; border-left: 3px solid #3182ce; padding-left: 10px; }

        .details-section { display: grid; grid-template-columns: 1.2fr 0.8fr; gap: 20px; }
        .sub-card { background: #fdfdfd; padding: 15px; border-radius: 8px; border: 1px solid #eee; }
        table { width: 100%; border-collapse: collapse; font-size: 0.85rem; }
        th, td { text-align: left; padding: 10px; border-bottom: 1px solid #f0f0f0; }
        th { background: #f8fafc; color: #4a5568; }
        
        /* Style spécifique pour la ligne Total */
        .row-total { background: #edf2f7; font-weight: bold; border-top: 2px solid #cbd5e0; }
        .renta-tag { background: #ebf8ff; color: #3182ce; padding: 2px 6px; border-radius: 4px; font-weight: bold; }

        .zoom-bar {
            position: fixed; top: 10px; right: 10px; z-index: 100;
            background: white; padding: 5px; border-radius: 5px; border: 1px solid #ccc;
        }
        .zoom-bar button { cursor: pointer; padding: 5px 10px; font-weight: bold; border: none; background: #eee; margin: 0 2px; }
    </style>
</head>
<body>

<div class="zoom-bar">
    <button onclick="zoom(0.1)">+</button>
    <button onclick="zoom(-0.1)">-</button>
</div>

<div class="container">
    <header>
        <h1>Analyse GESTION D'ACTIFS </h1>
    </header>

    <div class="kpi-grid">
        <div class="card"><span>Investissement</span><b>121 000 €</b></div>
        <div class="card" style="border-top-color: #38a169;"><span>Loyer Net / mois</span><b>808 €</b></div>
        <div class="card" style="border-top-color: #d69e2e;"><span>Cumul 10 ans</span><b id="txtCumul">0 €</b></div>
        <div class="card" style="border-top-color: #e53e3e;"><span>ROI Global</span><b id="txtRoi">0 %</b></div>
    </div>

    <h2>Projection de Richesse (Euros & ROI %)</h2>
    <div class="chart-main">
        <canvas id="chartCumul"></canvas>
    </div>

    <h2>Progression de la Rentabilité Annuelle (%)</h2>
    <div class="chart-sub">
        <canvas id="chartRentaLine"></canvas>
    </div>

    <div class="details-section">
        <div class="sub-card">
            <h3 style="margin-top:0; font-size: 0.9rem;">Performance par Actif</h3>
            <table>
                <thead>
                    <tr><th>Bien</th><th>Prix Achat</th><th>Loyer Net</th><th>Renta.</th></tr>
                </thead>
                <tbody>
                    <tr><td>Bien A</td><td>73 000 €</td><td>550 €</td><td><span class="renta-tag">9,04 %</span></td></tr>
                    <tr><td>Bien B</td><td>48 000 €</td><td>258 €</td><td><span class="renta-tag">6,45 %</span></td></tr>
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
            <h3 style="margin-top:0; font-size: 0.9rem;">Résumé à 10 ans</h3>
            <p style="font-size: 0.8rem; line-height: 1.5; color: #4a5568;">
                • Indexation : <b>2% / an</b><br>
                • Revenu mensuel final : <b id="rentaAn10">0 €</b><br>
                • Capital récupéré : <b id="pctRecup">0 %</b>
            </p>
        </div>
    </div>
</div>

<script>
    let currentZoom = 1.0;
    function zoom(v) { currentZoom += v; document.body.style.transform = `scale(${currentZoom})`; }

    const totalInvesti = 121000;
    let loyerAnnuel = 9696;
    let cumulCash = 0;

    const labels = [];
    const dataRentaAnnuelle = []; 
    const dataRoiCumule = [];     
    const dataCumulEuros = [];    

    for (let i = 1; i <= 10; i++) {
        labels.push("An " + i);
        let rentaAn = (loyerAnnuel / totalInvesti) * 100;
        cumulCash += loyerAnnuel;
        let roiCumule = (cumulCash / totalInvesti) * 100;

        dataRentaAnnuelle.push(rentaAn.toFixed(2));
        dataRoiCumule.push(roiCumule.toFixed(2));
        dataCumulEuros.push(Math.round(cumulCash));

        if(i === 10) {
            document.getElementById('txtCumul').innerText = Math.round(cumulCash).toLocaleString() + " €";
            document.getElementById('txtRoi').innerText = roiCumule.toFixed(2) + " %";
            document.getElementById('rentaAn10').innerText = Math.round(loyerAnnuel / 12) + " €/m";
            document.getElementById('pctRecup').innerText = roiCumule.toFixed(1) + " %";
        }
        loyerAnnuel *= 1.02;
    }

    // Graphique 1 : Cumul
    new Chart(document.getElementById('chartCumul'), {
        type: 'line',
        data: {
            labels: labels,
            datasets: [
                { label: 'ROI Cumulé (%)', data: dataRoiCumule, borderColor: '#e53e3e', backgroundColor: 'rgba(229, 62, 62, 0.1)', fill: true, yAxisID: 'y' },
                { label: 'Cash-flow cumulé (€)', data: dataCumulEuros, borderColor: '#d69e2e', yAxisID: 'y1', pointRadius: 0 }
            ]
        },
        options: {
            responsive: true, maintainAspectRatio: false,
            scales: {
                y: { position: 'left', title: { display: true, text: 'ROI %' } },
                y1: { position: 'right', grid: { display: false }, title: { display: true, text: 'Cumul €' } }
            }
        }
    });

    // Graphique 2 : Rentabilité Annuelle (Ligne)
    new Chart(document.getElementById('chartRentaLine'), {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                label: 'Rentabilité annuelle (%)',
                data: dataRentaAnnuelle,
                borderColor: '#3182ce',
                backgroundColor: 'rgba(49, 130, 206, 0.1)',
                borderWidth: 3,
                tension: 0.4,
                fill: true,
                pointRadius: 4,
                pointBackgroundColor: '#fff'
            }]
        },
        options: {
            responsive: true, maintainAspectRatio: false,
            scales: {
                y: { 
                    min: 8, 
                    max: 10,
                    ticks: { stepSize: 0.5 },
                    title: { display: true, text: 'Renta %' } 
                }
            },
            plugins: {
                legend: { display: false }
            }
        }
    });
</script>

</body>
</html>
