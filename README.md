# PulseEditor1
body {
    background: #1a1a1a url('assets/background.jpg') no-repeat center/cover;
    color: #fff;
    font-family: 'Roboto', sans-serif;
    margin: 0;
    padding: 0;
    min-height: 100vh;
    transition: background 0.3s, color 0.3s;
}

#container {
    display: flex;
    flex-direction: column;
    height: 100vh;
}

#toolbar {
    background: rgba(0, 0, 0, 0.8);
    padding: 10px;
    display: flex;
    flex-wrap: wrap;
    gap: 5px;
}

button {
    border: 1px solid white;
    background: transparent;
    color: white;
    padding: 5px 10px;
    cursor: pointer;
    transition: background 0.3s;
}

button:hover {
    background: rgba(255, 255, 255, 0.2);
}

#editor {
    flex: 1;
    padding: 20px;
    background: rgba(255, 255, 255, 0.1);
    outline: none;
    overflow-y: auto;
}

#budgetPanel {
    width: 350px;
    position: fixed;
    right: -350px; /* Escondido por padrão */
    top: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.9);
    padding: 20px;
    transition: right 0.3s;
    overflow-y: auto;
}

#budgetPanel.active {
    right: 0; /* Visível quando ativo */
}

#budgetPanel input, #budgetPanel textarea {
    width: 100%;
    margin: 5px 0;
    padding: 5px;
    background: #333;
    border: 1px solid #fff;
    color: #fff;
    box-sizing: border-box;
}

#itemList {
    margin: 10px 0;
}

#pomodoroTimer {
    position: fixed;
    bottom: 10px;
    left: 10px;
    font-size: 24px;
    display: none;
}

/* Temas */
.light { background: #fff; color: #000; }
.blue { background: #1a2a44; color: #fff; }
.green { background: #1a3c34; color: #fff; }

/* Modo Zen */
.zen #toolbar, .zen #budgetPanel, .zen #pomodoroTimer { display: none; }
.zen #editor { background: transparent; }
// Editor
function changeFont() {
    const font = document.getElementById('fontSelect').value;
    document.getElementById('editor').style.fontFamily = font;
}

function saveEditor() {
    localStorage.setItem('pulseEditorContent', document.getElementById('editor').innerHTML);
}

function loadEditor() {
    document.getElementById('editor').innerHTML = localStorage.getItem('pulseEditorContent') || '';
}

// Música
function playMusic() {
    document.getElementById('ambientMusic').play();
}

function pauseMusic() {
    document.getElementById('ambientMusic').pause();
}

// Imagem de Fundo
function changeBackground(event) {
    const file = event.target.files[0];
    if (file) {
        const url = URL.createObjectURL(file);
        document.body.style.backgroundImage = `url(${url})`;
    }
}

// Tabela
function insertTable(rows, cols) {
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
    saveEditor();
}

// Exportação
const { jsPDF } = window.jspdf;
function generatePDF() {
    const doc = new jsPDF();
    doc.text(document.getElementById('editor').innerText, 10, 10);
    doc.save('PulseEditor.pdf');
}

function saveAsTxt() {
    const text = document.getElementById('editor').innerText;
    const blob = new Blob([text], { type: 'text/plain' });
    saveAs(blob, 'PulseEditor.txt');
}

function saveAsDocx() {
    const text = document.getElementById('editor').innerHTML;
    const blob = new Blob(['<!DOCTYPE html><html><body>' + text + '</body></html>'], { type: 'application/msword' });
    saveAs(blob, 'PulseEditor.docx');
}

// Orçamento
function toggleBudgetPanel() {
    document.getElementById('budgetPanel').classList.toggle('active');
}

function uploadLogo(event) {
    const file = event.target.files[0];
    if (file) {
        const url = URL.createObjectURL(file);
        const logo = document.getElementById('logoPreview');
        logo.src = url;
        logo.style.display = 'block';
        localStorage.setItem('pulseEditorLogo', url);
    }
}

let items = JSON.parse(localStorage.getItem('pulseEditorItems')) || [];
function addItem() {
    const item = {
        name: document.getElementById('itemName').value,
        description: document.getElementById('itemDescription').value,
        unitValue: parseFloat(document.getElementById('itemUnitValue').value) || 0,
        quantity: parseInt(document.getElementById('itemQuantity').value) || 1
    };
    item.total = item.unitValue * item.quantity;
    items.push(item);
    localStorage.setItem('pulseEditorItems', JSON.stringify(items));
    updateItemList();
    updateTotal();
}

function updateItemList() {
    const list = document.getElementById('itemList');
    list.innerHTML = items.map((item, index) => `
        <div>
            <span>${item.name} - ${item.description}: ${item.quantity}x R$${item.unitValue.toFixed(2)} = R$${item.total.toFixed(2)}</span>
            <button onclick="removeItem(${index})">Remover</button>
        </div>
    `).join('');
}

function removeItem(index) {
    items.splice(index, 1);
    localStorage.setItem('pulseEditorItems', JSON.stringify(items));
    updateItemList();
    updateTotal();
}

function updateTotal() {
    const total = items.reduce((sum, item) => sum + item.total, 0);
    document.getElementById('totalValue').innerText = total.toFixed(2);
}

function saveBudget() {
    const budget = {
        company: {
            name: document.getElementById('companyName').value,
            address: document.getElementById('companyAddress').value,
            date: document.getElementById('companyDate').value,
            logo: document.getElementById('logoPreview').src
        },
        client: {
            name: document.getElementById('clientName').value,
            address: document.getElementById('clientAddress').value,
            phone: document.getElementById('clientPhone').value,
            email: document.getElementById('clientEmail').value
        },
        supplier: {
            name: document.getElementById('supplierName').value,
            address: document.getElementById('supplierAddress').value
        },
        items: items,
        footer: document.getElementById('footerInfo').value
    };
    localStorage.setItem('pulseEditorBudget', JSON.stringify(budget));
}

function sendBudget(method) {
    const budget = JSON.parse(localStorage.getItem('pulseEditorBudget') || '{}');
    const text = `
Orçamento - PulseEditor
Empresa: ${budget.company?.name || ''} | ${budget.company?.address || ''} | ${budget.company?.date || ''}
Cliente: ${budget.client?.name || ''} | ${budget.client?.address || ''} | ${budget.client?.phone || ''} | ${budget.client?.email || ''}
Fornecedor: ${budget.supplier?.name || ''} | ${budget.supplier?.address || ''}
Itens:
${items.map(i => `${i.name} - ${i.description}: ${i.quantity}x R$${i.unitValue.toFixed(2)} = R$${i.total.toFixed(2)}`).join('\n')}
Total: R$${items.reduce((sum, i) => sum + i.total, 0).toFixed(2)}
Rodapé: ${budget.footer || ''}
    `;
    if (method === 'email') {
        const email = document.getElementById('sendToEmail').value;
        window.open(`mailto:${email}?subject=Orçamento PulseEditor&body=${encodeURIComponent(text)}`);
    } else if (method === 'whatsapp') {
        const phone = document.getElementById('sendToWhatsApp').value;
        window.open(`https://wa.me/${phone}?text=${encodeURIComponent(text)}`);
    }
}

// Modo Zen
function toggleZenMode() {
    document.body.classList.toggle('zen');
}

// Temas
function changeTheme() {
    const theme = document.getElementById('themeSelect').value;
    document.body.className = theme;
}

// Pomodoro
let timeLeft = 25 * 60;
let timerId = null;
function startPomodoro() {
    document.getElementById('pomodoroTimer').style.display = 'block';
    if (timerId) clearInterval(timerId);
    timerId = setInterval(() => {
        timeLeft--;
        const minutes = Math.floor(timeLeft / 60);
        const seconds = timeLeft % 60;
        document.getElementById('pomodoroTimer').innerText = `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
        if (timeLeft <= 0) {
            clearInterval(timerId);
            alert('Pomodoro concluído!');
            timeLeft = 25 * 60;
            document.getElementById('pomodoroTimer').style.display = 'none';
        }
    }, 1000);
}

// Carregar ao iniciar
window.onload = function() {
    loadEditor();
    const savedLogo = localStorage.getItem('pulseEditorLogo');
    if (savedLogo) {
        document.getElementById('logoPreview').src = savedLogo;
        document.getElementById('logoPreview').style.display = 'block';
    }
    updateItemList();
    updateTotal();
};
