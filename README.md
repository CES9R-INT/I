<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCI - Évolution 10 ans</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: 'Segoe UI', sans-serif; background: #f4f7f9; margin: 0; padding: 20px; color: #2d3748; transition: transform 0.2s; transform-origin: top center; }
        .main-wrapper { max-width: 1200px; margin: auto; background: white; padding: 25px; border-radius: 15px; box-shadow: 0 10px 25px rgba(0,0,0,0.08); }
        h1, h2 { text-align: center; color: #1a365d; }
        .sci-summary { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 15px; margin-bottom: 25px; }
        .stat-card { background: #edf2f7; padding: 15px; border-radius: 10px; text-align: center; border: 1px solid #e2e8f0; }
        .stat-card b { display: block; font-size: 1.4em; color: #2b6cb0; }
        .chart-container { position: relative; height: 400px; width: 100%; margin-top: 20px; }
        .zoom-controls { position: fixed; top: 10px; right: 10px; z-index: 9999; }
        .zoom-controls button { padding: 5px 10px; cursor: pointer; border: 1px solid #cbd5e0; background: white; border-radius: 5px; font-weight: bold; }
    </style>
</head>
<body>

<div class="zoom-controls">
  <button onclick="ajusterZoom(0.1)">Zoom +</button>
  <button onclick="ajusterZoom(-0.1)">Zoom -</button>
</div>

<div class="main-wrapper">
    <h1>Analyse de la SCI (Biens A + B)</h1>
    
    <div class="sci-summary">
        <div class="stat-card">
            <label>Valeur Totale Patrimoine</label>
            <b>121 000 €</b>
        </div>
        <div class="stat-card">
            <label>Revenu Net Mensuel (An 1)</label>
            <b>808 €</b>
        </div>
        <div class="stat-card">
            <label>Rentabilité Nette Départ</label>
            <b id="rentaInitiale">8.01 %</b>
        </div>
    </div>

    <h2>Évolution de la Rentabilité sur 10 ans</h2>
    <p style="text-align:center; font-size: 0.9em; color: #718096;">Hypothèse : Indexation des loyers de +2% par an</p>
    
    <div class="chart-container">
        <canvas id="sciChart"></canvas>
    </div>
</div>

<script>
    let currentZoom = 1.0;
    function ajusterZoom(delta) {
        currentZoom += delta;
        document.body.style.transform = `scale(${currentZoom})`;
    }

    // Données de base
    const valeurA = 73000;
    const valeurB = 48000;
    const totalPatrimoine = valeurA + valeurB; // 121 000 €

    const loyerNetMensuelA = 550;
    const loyerNetMensuelB = 258;
    let loyerAnnuelTotal = (loyerNetMensuelA + loyerNetMensuelB) * 12; // 9696 €

    const labels = [];
    const dataRenta = [];
    const dataCashflow = [];

    // Calcul sur 10 ans
    for (let annee = 1; annee <= 10; annee++) {
        labels.push("Année " + annee);
        
        // Calcul de la rentabilité pour l'année N
        let rentaAnnuelle = (loyerAnnuelTotal / totalPatrimoine) * 100;
        dataRenta.push(rentaAnnuelle.toFixed(2));
        dataCashflow.push(Math.round(loyerAnnuelTotal));

        // On augmente le loyer de 2% pour l'année suivante (indexation)
        loyerAnnuelTotal *= 1.02;
    }

    const ctx = document.getElementById('sciChart').getContext('2d');
    new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                label: 'Rentabilité Nette (%)',
                data: dataRenta,
                borderColor: '#3182ce',
                backgroundColor: 'rgba(49, 130, 206, 0.1)',
                fill: true,
                tension: 0.4,
                yAxisID: 'y',
            },
            {
                label: 'Cash-flow Net Annuel (€)',
                data: dataCashflow,
                borderColor: '#38a169',
                borderDash: [5, 5],
                tension: 0.4,
                yAxisID: 'y1',
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
                y: {
                    type: 'linear',
                    display: true,
                    position: 'left',
                    title: { display: true, text: 'Rentabilité (%)' }
                },
                y1: {
                    type: 'linear',
                    display: true,
                    position: 'right',
                    grid: { drawOnChartArea: false },
                    title: { display: true, text: 'Revenus (€/an)' }
                }
            },
            plugins: {
                tooltip: {
                    callbacks: {
                        label: function(context) {
                            if (context.datasetIndex === 0) return context.parsed.y + ' %';
                            return context.parsed.y.toLocaleString() + ' € / an';
                        }
                    }
                }
            }
        }
    });
</script>

</body>
</html>
