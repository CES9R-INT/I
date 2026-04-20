function exportToWhatsApp() {
    let message = "*MERCURIAL - CESAR INTERNATIONAL*\n";
    message += "📅 " + document.getElementById('current-date').innerText + "\n\n";

    const rows = document.querySelectorAll('tbody tr');
    rows.forEach(row => {
        const produit = row.cells[0].innerText;
        const calibre = row.cells[1].innerText;
        const prix = row.querySelector('.prix-fr').innerText; // On prend le prix FR par défaut

        if (prix && prix !== "0.00") {
            message += `• *${produit}* (${calibre}) : ${prix}€\n`;
        }
    });

    message += "\n🚀 _Prix départ Perpignan_";

    // Copie dans le presse-papier et petite alerte
    navigator.clipboard.writeText(message).then(() => {
        alert("Texte copié ! Vous pouvez maintenant le coller dans WhatsApp.");
    });
}
