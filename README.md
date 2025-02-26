# PulseEditor1
function toggleTheme() {
    document.body.classList.toggle('light-theme');
}body {
    background: #1a1a1a;
    color: #fff;
}
.light-theme {
    background: #fff;
    color: #000;
}<link href="https://fonts.googleapis.com/css2?family=Roboto|Open+Sans|...&display=swap" rel="stylesheet"><audio id="ambientMusic" loop>
    <source src="assets/ambient.mp3" type="audio/mp3">
</audio>
<button onclick="document.getElementById('ambientMusic').play()">Tocar Música</button>
<button onclick="document.getElementById('ambientMusic').pause()">Pausar</button>body {
    background-image: url('assets/background.jpg');
    background-size: cover;
}function changeBackground(event) {
    const file = event.target.files[0];
    const url = URL.createObjectURL(file);
    document.body.style.backgroundImage = `url(${url})`;
}const { jsPDF } = window.jspdf;
function generatePDF() {
    const doc = new jsPDF();
    doc.text(document.getElementById('editor').innerText, 10, 10);
    doc.save('documento.pdf');
}function insertTable(rows, cols) {
    let table = '<table border="1">';
    for (let i = 0; i < rows; i++) {
        table += '<tr>';
        for (let j = 0; j < cols; j++) {
            table += '<td contenteditable="true"></td>';
        }
        table += '</tr>';
    }
    table += '</table>';
    document.getElementById('editor').innerHTML += table;
}function saveProduct() {
    const product = document.getElementById('productName').value;
    let products = JSON.parse(localStorage.getItem('products')) || [];
    products.push(product);
    localStorage.setItem('products', JSON.stringify(products));
}****function saveEditor() {
    localStorage.setItem('editorContent', document.getElementById('editor').innerHTML);
}
function loadEditor() {
    document.getElementById('editor').innerHTML = localStorage.getItem('editorContent') || '';
}
