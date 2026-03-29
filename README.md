<!THE SHADY GROVE>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Analyse SCI - Performance Globale</title>
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
            max-width: 950px; 
            margin: auto; 
            background: white; 
            padding: 25px; 
            border-radius: 12px; 
            box-shadow: 0 8px 20px rgba(0,0,0,0.06); 
        }
        header { text-align: center; margin-bottom: 20px; border-bottom: 1px solid #eee; padding-bottom: 10px; }
        h1 { font-size: 1.3rem; color: #1a365d; margin: 0; }
        
        .kpi-grid { 
            display: grid; 
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); 
            gap: 12px; 
            margin-bottom: 25px; 
        }
        .card { 
            background: #fff; 
            padding: 12px; 
            border-radius: 8px; 
            text-align: center; 
            border: 1px solid #e2e8f0;
            border-top: 4px solid #3182ce;
        }
        .card span { font-size: 0.7rem; color: #718096; font-weight: bold; text-transform: uppercase; }
        .card b { display: block; font-size: 1.3rem; color: #2d3748; margin-top: 5px; }

        .chart-container { 
            position: relative; 
            height: 400px; 
            margin-bottom: 25px; 
            padding: 10px;
            border: 1px solid #f0f0f0;
            border-radius: 8px;
        }

        .details-section { display: grid; grid-template-columns: 1.2fr 0.8fr; gap: 20px; }
        .sub-card { background: #fdfdfd; padding: 15px; border-radius: 8px; border: 1px solid #eee; }
        table { width: 100%; border-collapse: collapse; font-size: 0.85rem; }
        th, td { text-align: left; padding: 10px; border-bottom: 1px solid #f0f0f0; }
        th { background: #f8fafc; color: #4a5568; }
        
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
        <h1>Analyse SCI : Rentabilité et Flux de Trésorerie</h1>
    </header>

    <div class="kpi-grid">
        <div class="card"><span>Investissement</span><b>121 000 €</b></div>
        <div class="card" style="border-top-color: #38a169;"><span>Loyer Mensuel Net</span><b>808 €</b></div>
        <div class="card" style="border-top-color: #d69e2e;"><span>Cumul Cash 10 ans</span><b id="txtCumul">0 €</b></div>
        <div class="card" style="border-top-color: #e53e3e;"><span>ROI Total (10 ans)</span><b id="txtRoi">0 %</b></div>
    </div>

    <div class="chart-container">
        <canvas id="mainChart"></canvas>
    </div>

    <div class="details-section">
        <div class="sub-card">
            <h3 style="margin-top:0; font-size: 0.95rem; color: #1a365d;">Détail des Actifs</h3>
            <table>
                <thead>
                    <tr><th>Bien</th><th>Prix Achat</th><th>Loyer Net</th><th>Rentabilité</th></tr>
                </thead>
                <tbody>
                    <tr><td>Appartement A</td><td>73 000 €</td><td>550 €</td><td><span class="renta-tag">9,04 %</span></td></tr>
                    <tr><td>Appartement B</td><td>48 000 €</td><td>258 €</td><td><span class="renta-tag">6,45 %</span></td></tr>
                    <tr style="background: #f8fafc; font-weight: bold;">
                        <td>Total SCI</td><td>121 000 €</td><td>808 €</td><td>8,01 %</td>
                    </tr>
                </tbody>
            </table>
        </div>
        
        <div class="sub-card">
            <h3 style="margin-top:0; font-size: 0.95rem; color: #1a365d;">Performance Long Terme</h3>
            <p style="font-size: 0.85rem; line-height: 1.5; color: #4a5568;">
                • Indexation des loyers : <b>2% / an</b><br>
                • Rentabilité an 10 : <b id="rentaAn10">0 %</b><br>
                • Plus-value estimée (2%/an) : <b>+26 500 €</b><br><br>
                <span style="font-style: italic; font-size: 0.75rem;">Le ROI Total calcule le cumul des loyers perçus divisé par le montant investi au départ.</span>
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
    const dataRentaAnnuelle = []; // Renta de l'année N
    const dataRoiCumule = [];     // ROI (Cumul / Investissement)
    const dataCumulEuros = [];    // Montant en €

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
            document.getElementById('rentaAn10').innerText = rentaAn.toFixed(2) + " %";
        }
        
        loyerAnnuel *= 1.02; // Indexation
    }

    new Chart(document.getElementById('mainChart'), {
        type: 'line',
        data: {
            labels: labels,
            datasets: [
                { 
                    label: 'ROI Cumulé (% du capital)', 
                    data: dataRoiCumule, 
                    borderColor: '#e53e3e', 
                    backgroundColor: 'rgba(229, 62, 62, 0.1)', 
                    fill: true, 
                    yAxisID: 'y' 
                },
                { 
                    label: 'Rentabilité annuelle (%)', 
                    data: dataRentaAnnuelle, 
                    borderColor: '#3182ce', 
                    borderDash: [5, 5],
                    yAxisID: 'y' 
                },
                { 
                    label: 'Cash-flow cumulé (€)', 
                    data: dataCumulEuros, 
                    borderColor: '#d69e2e', 
                    yAxisID: 'y1',
                    pointRadius: 0
                }
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            interaction: { mode: 'index', intersect: false },
            scales: {
                y: { 
                    position: 'left', 
                    title: { display: true, text: 'Performance (%)' },
                    min: 0
                },
                y1: { 
                    position: 'right', 
                    grid: { display: false }, 
                    title: { display: true, text: 'Cumul (€)' } 
                }
            }
        }
    });
</script>

</body>
</html>
