SCI-THE SHADY GROVE
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCI - Stratégie Boule de Neige</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: 'Segoe UI', sans-serif; background: #f4f7f9; margin: 0; padding: 20px; color: #2d3748; transform-origin: top center; transition: transform 0.2; }
        .container { max-width: 1100px; margin: auto; background: white; padding: 25px; border-radius: 12px; box-shadow: 0 8px 20px rgba(0,0,0,0.06); }
        header { text-align: center; margin-bottom: 20px; border-bottom: 1px solid #eee; padding-bottom: 10px; }
        h1 { font-size: 1.3rem; color: #1a365d; margin: 0; }
        h2 { font-size: 1rem; color: #2b6cb0; margin-top: 25px; border-left: 4px solid #2b6cb0; padding-left: 10px; }
        
        .kpi-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 12px; margin-bottom: 20px; }
        .card { background: #fff; padding: 12px; border-radius: 8px; text-align: center; border: 1px solid #e2e8f0; border-top: 4px solid #3182ce; }
        .card span { font-size: 0.7rem; color: #718096; font-weight: bold; text-transform: uppercase; }
        .card b { display: block; font-size: 1.2rem; color: #2d3748; margin-top: 5px; }

        .charts-wrapper { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 20px; }
        .chart-box { height: 250px; background: #fcfcfc; border-radius: 8px; border: 1px solid #eee; padding: 10px; }

        table { width: 100%; border-collapse: collapse; font-size: 0.85rem; margin-top: 10px; }
        th, td { text-align: left; padding: 8px; border-bottom: 1px solid #f0f0f0; }
        th { background: #f8fafc; color: #4a5568; }
        .row-total { background: #edf2f7; font-weight: bold; }
        .buy-event { background: #f0fff4; color: #2f855a; font-weight: bold; border-radius: 4px; padding: 2px 5px; }

        .zoom-bar { position: fixed; top: 10px; right: 10px; z-index: 100; background: white; border: 1px solid #ccc; border-radius: 5px; padding: 5px; }
        .zoom-bar button { cursor: pointer; padding: 5px 10px; font-weight: bold; border: none; background: #eee; margin: 0 2px; }
    </style>
</head>
<body>

<div class="zoom-bar">
    <button onclick="zoom(0.1)">+</button>
    <button onclick="zoom(-0.1)">-</button>
</div>

<div class="container">
    <header><h1>Stratégie SCI : Réinvestissement du Cash-flow</h1></header>

    <div class="kpi-grid">
        <div class="card"><span>Investissement Initial</span><b>121 000 €</b></div>
        <div class="card" style="border-top-color: #38a169;"><span>Loyer Départ</span><b>808 €/m</b></div>
        <div class="card" style="border-top-color: #d69e2e;"><span>Cumul 10 ans (Standard)</span><b>106 169 €</b></div>
        <div class="card" style="border-top-color: #805ad5;"><span>Parc Final (Scénario)</span><b id="parcFinal">2 biens</b></div>
    </div>

    <div class="charts-wrapper">
        <div class="chart-box">
            <canvas id="chartCumul"></canvas>
        </div>
        <div class="chart-box">
            <canvas id="chartRenta"></canvas>
        </div>
    </div>

    <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px;">
        <div class="sub-card">
            <h2>1. Performance par Actif Initial</h2>
            <table>
                <thead><tr><th>Bien</th><th>Prix</th><th>Loyer</th><th>Renta.</th></tr></thead>
                <tbody>
                    <tr><td>Bien A</td><td>73 000 €</td><td>550 €</td><td>9,04 %</td></tr>
                    <tr><td>Bien B</td><td>48 000 €</td><td>258 €</td><td>6,45 %</td></tr>
                </tbody>
                <tfoot><tr class="row-total"><td>TOTAL</td><td>121 000 €</td><td>808 €</td><td>8,01 %</td></tr></tfoot>
            </table>
        </div>

        <div class="sub-card">
            <h2>2. Scénario Réinvestissement (Seuil 50k€)</h2>
            <table>
                <thead><tr><th>Année</th><th>Loyer/m</th><th>Cash Cumulé</th><th>Action</th></tr></thead>
                <tbody id="bouleDeNeigeBody"></tbody>
            </table>
        </div>
    </div>
</div>

<script>
    let currentZoom = 1.0;
    function zoom(v) { currentZoom += v; document.body.style.transform = `scale(${currentZoom})`; }

    // Paramètres
    const investInitial = 121000;
    const loyerBienB = 258;
    const prixBienB = 50000; // 48k + 2k frais
    
    let cashCumule = 0;
    let loyerMensuelActuel = 808;
    let nombreDeBiensB_Achetes = 0;

    const labels = [];
    const dataCumul = [];
    const dataLoyer = [];

    const tableBody = document.getElementById('bouleDeNeigeBody');

    for (let an = 1; an <= 10; an++) {
        labels.push("An " + an);
        let cashAnnuel = loyerMensuelActuel * 12;
        cashCumule += cashAnnuel;

        let action = "-";
        // Si le cash atteint 50k, on achète un bien B
        if (cashCumule >= prixBienB) {
            nombreDeBiensB_Achetes++;
            cashCumule -= prixBienB; // On dépense le cash
            loyerMensuelActuel += loyerBienB; // Le nouveau bien génère du loyer
            action = `<span class="buy-event">+1 Bien B</span>`;
        }

        dataCumul.push(Math.round(cashCumule));
        dataLoyer.push(Math.round(loyerMensuelActuel));

        // Remplissage tableau
        tableBody.innerHTML += `
            <tr>
                <td>An ${an}</td>
                <td>${Math.round(loyerMensuelActuel)} €</td>
                <td>${Math.round(cashCumule).toLocaleString()} €</td>
                <td>${action}</td>
            </tr>
        `;

        // Indexation annuelle 2%
        loyerMensuelActuel *= 1.02;
    }

    document.getElementById('parcFinal').innerText = (2 + nombreDeBiensB_Achetes) + " biens";

    // Graphique 1 : Cash-flow résiduel après achats
    new Chart(document.getElementById('chartCumul'), {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{ label: 'Trésorerie disponible (€)', data: dataCumul, borderColor: '#d69e2e', backgroundColor: 'rgba(214,158,46,0.1)', fill: true }]
        },
        options: { responsive: true, maintainAspectRatio: false, plugins: { title: { display: true, text: 'Trésorerie après réinvestissement' } } }
    });

    // Graphique 2 : Explosion des revenus mensuels
    new Chart(document.getElementById('chartRenta'), {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{ label: 'Loyer mensuel global (€)', data: dataLoyer, borderColor: '#3182ce', tension: 0.3 }]
        },
        options: { responsive: true, maintainAspectRatio: false, plugins: { title: { display: true, text: 'Croissance du Loyer Mensuel' } } }
    });
</script>

</body>
</html>
