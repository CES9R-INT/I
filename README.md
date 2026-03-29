<!SCI THE SHADY GROVE>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCI Dashboard - Navigation Latérale</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        * { box-sizing: border-box; }
        body, html { 
            height: 100%; margin: 0; padding: 0; overflow: hidden;
            font-family: 'Segoe UI', sans-serif; background-color: #f0f2f5;
        }
        
        /* Conteneur pour le balayage latéral */
        .viewport {
            display: flex;
            width: 200vw; /* 2 vues de large */
            height: calc(100vh - 60px); /* Moins la barre de nav */
            transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
        }

        .slide {
            width: 100vw;
            height: 100%;
            padding: 20px;
            display: grid;
            gap: 15px;
        }

        /* --- STYLE DES COMPOSANTS --- */
        header {
            background: #1a365d; color: white; padding: 10px 25px;
            border-radius: 10px; display: flex; justify-content: space-between; align-items: center;
        }

        .kpi-row { display: grid; grid-template-columns: repeat(4, 1fr); gap: 15px; }
        .card { 
            background: white; padding: 15px; border-radius: 8px; text-align: center;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05); border-bottom: 4px solid #3182ce;
        }
        .card b { display: block; font-size: 1.8em; color: #2d3748; }

        .main-layout { display: grid; grid-template-columns: 2fr 1fr; gap: 15px; min-height: 0; }
        .chart-box, .sub-card { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }

        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { text-align: left; padding: 12px; border-bottom: 1px solid #edf2f7; }

        /* --- BARRE DE NAVIGATION BAS --- */
        .nav-bar {
            height: 60px; background: #1a365d; display: flex;
            justify-content: center; align-items: center; gap: 20px;
            position: fixed; bottom: 0; width: 100%; z-index: 1000;
        }
        .nav-btn {
            padding: 8px 20px; background: rgba(255,255,255,0.1); color: white;
            border: 1px solid rgba(255,255,255,0.3); border-radius: 20px; cursor: pointer;
            transition: 0.3s; font-weight: bold;
        }
        .nav-btn.active { background: #3182ce; border-color: #3182ce; }
        .nav-btn:hover { background: rgba(255,255,255,0.2); }
    </style>
</head>
<body>

<div class="viewport" id="viewport">
    
    <section class="slide">
        <header>
            <h1>Dashboard SCI : Performance Globale</h1>
            <b>Investissement : 121 000 €</b>
        </header>
        
        <div class="kpi-row">
            <div class="card"><span>Loyer Net Mensuel</span><b>808 €</b></div>
            <div class="card" style="border-color: #38a169;"><span>Renta. Nette</span><b>8.01 %</b></div>
            <div class="card" style="border-color: #d69e2e;"><span>Projection 10 ans</span><b id="cumul1">0 €</b></div>
            <div class="card" style="border-color: #e53e3e;"><span>Bien A / Bien B</span><b>60% / 40%</b></div>
        </div>

        <div class="main-layout">
            <div class="chart-box"><canvas id="sciChart"></canvas></div>
            <div class="sub-card">
                <h3 style="margin-top:0">Répartition Initiale</h3>
                <table>
                    <tr><td>Bien A (73k€)</td><td style="color:#3182ce; font-weight:bold;">550 €/m</td></tr>
                    <tr><td>Bien B (48k€)</td><td style="color:#3182ce; font-weight:bold;">258 €/m</td></tr>
                    <tr style="border-top: 2px solid #eee;"><td>Total SCI</td><td style="font-weight:bold;">808 €/m</td></tr>
                </table>
                <p style="font-size:0.8em; color:#718096; margin-top:20px;">* Simulation basée sur un investissement comptant ou avec apport couvrant les frais.</p>
            </div>
        </div>
    </section>

    <section class="slide">
        <header><h1>Projection des Flux de Trésorerie</h1></header>
        <div class="sub-card" style="height: 100%; overflow-y: auto;">
            <table id="tableFlux">
                <thead>
                    <tr style="background:#f8fafc">
                        <th>Année</th>
                        <th>Loyer Mensuel</th>
                        <th>Loyer Annuel</th>
                        <th>Renta / Coût</th>
                        <th>Revenu Cumulé</th>
                    </tr>
                </thead>
                <tbody id="fluxBody"></tbody>
            </table>
        </div>
    </section>

</div>

<nav class="nav-bar">
    <button class="nav-btn active" onclick="goToSlide(0, this)">1. Vue d'ensemble</button>
    <button class="nav-btn" onclick="goToSlide(1, this)">2. Tableau des flux</button>
</nav>

<script>
    // Logique de navigation
    function goToSlide(index, btn) {
        document.getElementById('viewport').style.transform = `translateX(-${index * 100}vw)`;
        document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
    }

    // Calculs et Graphique
    const totalPatrimoine = 121000;
    let loyerMensuel = 808;
    let cumul = 0;

    const labels = [];
    const dataRenta = [];
    const dataCumule = [];
    const fluxBody = document.getElementById('fluxBody');

    for (let i = 1; i <= 10; i++) {
        let loyerAnnuel = loyerMensuel * 12;
        let renta = (loyerAnnuel / totalPatrimoine) * 100;
        cumul += loyerAnnuel;

        labels.push("An " + i);
        dataRenta.push(renta.toFixed(2));
        dataCumule.push(Math.round(cumul));

        // Remplissage tableau Slide 2
        fluxBody.innerHTML += `
            <tr>
                <td><b>Année ${i}</b></td>
                <td>${Math.round(loyerMensuel)} €</td>
                <td>${Math.round(loyerAnnuel).toLocaleString()} €</td>
                <td><span style="color:#3182ce; font-weight:bold;">${renta.toFixed(2)} %</span></td>
                <td style="color:#d69e2e; font-weight:bold;">${Math.round(cumul).toLocaleString()} €</td>
            </tr>
        `;

        loyerMensuel *= 1.02; // Indexation
    }

    document.getElementById('cumul1').innerText = Math.round(cumul).toLocaleString() + " €";

    // Chart.js
    const ctx = document.getElementById('sciChart').getContext('2d');
    new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [
                { label: 'Cumul (€)', data: dataCumule, borderColor: '#d69e2e', backgroundColor: 'rgba(214,158,46,0.1)', fill: true, yAxisID: 'y1' },
                { label: 'Rentabilité (%)', data: dataRenta, borderColor: '#3182ce', tension: 0.3, yAxisID: 'y' }
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
                y: { position: 'left', title: { display: true, text: 'Renta %' } },
                y1: { position: 'right', grid: { drawOnChartArea: false }, title: { display: true, text: 'Cumul Cash' } }
            }
        }
    });
</script>

</body>
</html>
