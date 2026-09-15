<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KhataBook - Expense & Ledger Manager</title>
    <!-- jsPDF Library for PDF Export -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        :root {
            --primary: #1a73e8;
            --danger: #d93025;
            --success: #1e8e3e;
            --bg: #f4f6f9;
            --card-bg: #ffffff;
            --text: #202124;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background-color: var(--bg);
            color: var(--text);
            padding: 20px;
            max-width: 1000px;
            margin: 0 auto;
        }

        header {
            text-align: center;
            margin-bottom: 20px;
            padding: 15px;
            background: var(--primary);
            color: white;
            border-radius: 8px;
        }

        .dashboard {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            margin-bottom: 20px;
        }

        .card {
            background: var(--card-bg);
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            text-align: center;
        }

        .card h3 {
            font-size: 14px;
            color: #666;
            margin-bottom: 5px;
        }

        .card .amount {
            font-size: 22px;
            font-weight: bold;
        }

        .text-red { color: var(--danger); }
        .text-green { color: var(--success); }

        .main-content {
            display: grid;
            grid-template-columns: 1fr 2fr;
            gap: 20px;
        }

        @media (max-width: 768px) {
            .main-content {
                grid-template-columns: 1fr;
            }
        }

        .panel {
            background: var(--card-bg);
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }

        .panel h2 {
            margin-bottom: 15px;
            font-size: 18px;
            border-bottom: 2px solid var(--bg);
            padding-bottom: 8px;
        }

        .form-group {
            margin-bottom: 12px;
        }

        .form-group label {
            display: block;
            margin-bottom: 5px;
            font-size: 13px;
            color: #555;
            font-weight: 600;
        }

        .form-group input, .form-group select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 5px;
            font-size: 14px;
        }

        .btn-group {
            display: flex;
            gap: 10px;
        }

        .btn {
            flex: 1;
            padding: 10px;
            border: none;
            border-radius: 5px;
            color: white;
            font-weight: bold;
            cursor: pointer;
            transition: 0.2s;
        }

        .btn-gave { background-color: var(--danger); }
        .btn-got { background-color: var(--success); }
        .btn-primary { background-color: var(--primary); }
        .btn-export { background-color: #5f6368; margin-top: 10px; width: 100%; }

        .btn:hover { opacity: 0.9; }

        .customer-list {
            list-style: none;
            max-height: 200px;
            overflow-y: auto;
            margin-bottom: 15px;
        }

        .customer-item {
            padding: 10px;
            border-bottom: 1px solid #eee;
            cursor: pointer;
            display: flex;
            justify-content: space-between;
        }

        .customer-item:hover, .customer-item.active {
            background-color: #e8f0fe;
            font-weight: bold;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 10px;
        }

        th, td {
            padding: 10px;
            text-align: left;
            border-bottom: 1px solid #eee;
            font-size: 14px;
        }

        th { background-color: #f8f9fa; }

        .delete-btn {
            color: var(--danger);
            cursor: pointer;
            border: none;
            background: none;
            font-weight: bold;
        }

        .header-action {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
            border-bottom: 2px solid var(--bg);
            padding-bottom: 8px;
        }

        .header-action h2 {
            border-bottom: none;
            padding-bottom: 0;
            margin-bottom: 0;
        }

        .btn-pdf {
            background-color: #d93025;
            color: white;
            border: none;
            padding: 8px 14px;
            border-radius: 5px;
            font-weight: bold;
            cursor: pointer;
            font-size: 13px;
        }

        .btn-pdf:hover {
            opacity: 0.9;
        }
    </style>
</head>
<body>

    <header>
        <h1>📖 Digital KhataBook</h1>
    </header>

    <!-- Top Dashboard Summary -->
    <div class="dashboard">
        <div class="card">
            <h3>TOTAL YOU WILL GET</h3>
            <div id="total-get" class="amount text-green">₹0</div>
        </div>
        <div class="card">
            <h3>TOTAL YOU WILL GIVE</h3>
            <div id="total-give" class="amount text-red">₹0</div>
        </div>
        <div class="card">
            <h3>NET BALANCE</h3>
            <div id="net-balance" class="amount">₹0</div>
        </div>
    </div>

    <div class="main-content">
        
        <!-- Left Side: Add Customer & Customer List -->
        <div class="panel">
            <h2>Add / Select Customer</h2>
            <div class="form-group">
                <input type="text" id="cust-name" placeholder="Enter Customer Name">
            </div>
            <button class="btn btn-primary" style="width: 100%; margin-bottom: 15px;" onclick="addCustomer()">+ Add Customer</button>

            <h3>Customers</h3>
            <ul class="customer-list" id="customer-list">
                <!-- Dynamic Customer List -->
            </ul>

            <button class="btn btn-export" onclick="exportFullSummaryPDF()">📄 Download Overall Summary PDF</button>
        </div>

        <!-- Right Side: Transactions -->
        <div class="panel">
            <div class="header-action">
                <h2 id="current-cust-title">Select a Customer</h2>
                <button id="export-cust-pdf-btn" class="btn-pdf" style="display: none;" onclick="exportCustomerPDF()">📥 Download PDF</button>
            </div>
            
            <div id="transaction-form" style="display: none;">
                <!-- Calendar Date Input (2000 to 2030) -->
                <div class="form-group">
                    <label for="tx-date">Select Date (2000 - 2030):</label>
                    <input type="date" id="tx-date" min="2000-01-01" max="2030-12-31">
                </div>

                <div class="form-group">
                    <input type="number" id="tx-amount" placeholder="Amount (₹)">
                </div>
                <div class="form-group">
                    <input type="text" id="tx-note" placeholder="Details/Note (e.g., Grocery, Rent)">
                </div>
                <div class="btn-group">
                    <button class="btn btn-gave" onclick="addTransaction('gave')">You Gave ₹ (Credit)</button>
                    <button class="btn btn-got" onclick="addTransaction('got')">You Got ₹ (Debit)</button>
                </div>
                <hr style="margin: 20px 0;">
            </div>

            <!-- Transaction History -->
            <h3>Transaction History</h3>
            <table>
                <thead>
                    <tr>
                        <th>Date</th>
                        <th>Details</th>
                        <th>You Gave</th>
                        <th>You Got</th>
                        <th>Action</th>
                    </tr>
                </thead>
                <tbody id="tx-history">
                    <!-- Dynamic Rows -->
                </tbody>
            </table>
        </div>

    </div>

    <script>
        // Store data locally
        let customers = JSON.parse(localStorage.getItem('khata_customers')) || [];
        let transactions = JSON.parse(localStorage.getItem('khata_transactions')) || [];
        let selectedCustomerId = null;

        // Set default calendar date to today
        function setDefaultDate() {
            const today = new Date().toISOString().split('T')[0];
            const dateInput = document.getElementById('tx-date');
            
            if (today >= '2000-01-01' && today <= '2030-12-31') {
                dateInput.value = today;
            } else {
                dateInput.value = '2026-01-01';
            }
        }

        // Initialize App
        function init() {
            setDefaultDate();
            renderCustomers();
            updateDashboard();
        }

        // Save Data to Local Storage
        function saveData() {
            localStorage.setItem('khata_customers', JSON.stringify(customers));
            localStorage.setItem('khata_transactions', JSON.stringify(transactions));
            updateDashboard();
        }

        // Add New Customer
        function addCustomer() {
            const nameInput = document.getElementById('cust-name');
            const name = nameInput.value.trim();

            if (!name) {
                alert('Please enter a customer name!');
                return;
            }

            const newCust = {
                id: Date.now(),
                name: name
            };

            customers.push(newCust);
            saveData();
            renderCustomers();
            nameInput.value = '';
            selectCustomer(newCust.id);
        }

        // Render Customer List
        function renderCustomers() {
            const list = document.getElementById('customer-list');
            list.innerHTML = '';

            customers.forEach(cust => {
                const li = document.createElement('li');
                li.className = `customer-item ${cust.id === selectedCustomerId ? 'active' : ''}`;
                li.onclick = () => selectCustomer(cust.id);
                
                const custBalance = getCustomerBalance(cust.id);
                let balClass = custBalance >= 0 ? 'text-green' : 'text-red';

                li.innerHTML = `
                    <span>${cust.name}</span>
                    <span class="${balClass}">₹${Math.abs(custBalance)}</span>
                `;
                list.appendChild(li);
            });
        }

        // Select Customer
        function selectCustomer(id) {
            selectedCustomerId = id;
            const cust = customers.find(c => c.id === id);
            
            document.getElementById('current-cust-title').innerText = `Transactions for: ${cust.name}`;
            document.getElementById('transaction-form').style.display = 'block';
            document.getElementById('export-cust-pdf-btn').style.display = 'inline-block';

            renderCustomers();
            renderTransactions();
        }

        // Add Transaction (Gave / Got)
        function addTransaction(type) {
            const amountInput = document.getElementById('tx-amount');
            const noteInput = document.getElementById('tx-note');
            const dateInput = document.getElementById('tx-date');

            const amount = parseFloat(amountInput.value);
            const note = noteInput.value.trim() || (type === 'gave' ? 'You Gave' : 'You Got');
            const selectedDate = dateInput.value;

            if (!selectedDate) {
                alert('Please select a valid date!');
                return;
            }

            if (selectedDate < '2000-01-01' || selectedDate > '2030-12-31') {
                alert('Please select a date between the year 2000 and 2030.');
                return;
            }

            if (!amount || amount <= 0) {
                alert('Please enter a valid amount!');
                return;
            }

            const newTx = {
                id: Date.now(),
                customerId: selectedCustomerId,
                type: type, // 'gave' or 'got'
                amount: amount,
                note: note,
                date: selectedDate
            };

            transactions.push(newTx);
            saveData();
            renderTransactions();
            renderCustomers();

            amountInput.value = '';
            noteInput.value = '';
        }

        // Render Transactions for selected customer
        function renderTransactions() {
            const tbody = document.getElementById('tx-history');
            tbody.innerHTML = '';

            const custTx = transactions.filter(t => t.customerId === selectedCustomerId);

            if (custTx.length === 0) {
                tbody.innerHTML = '<tr><td colspan="5" style="text-align:center;">No transactions found.</td></tr>';
                return;
            }

            // Sort transactions by date descending
            custTx.sort((a, b) => new Date(b.date) - new Date(a.date));

            custTx.forEach(tx => {
                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td>${tx.date}</td>
                    <td>${tx.note}</td>
                    <td class="text-red">${tx.type === 'gave' ? '₹' + tx.amount : '-'}</td>
                    <td class="text-green">${tx.type === 'got' ? '₹' + tx.amount : '-'}</td>
                    <td><button class="delete-btn" onclick="deleteTransaction(${tx.id})">✕ Delete</button></td>
                `;
                tbody.appendChild(tr);
            });
        }

        // Delete Single Transaction
        function deleteTransaction(txId) {
            transactions = transactions.filter(t => t.id !== txId);
            saveData();
            renderTransactions();
            renderCustomers();
        }

        // Get single customer net balance
        function getCustomerBalance(custValId) {
            const custTx = transactions.filter(t => t.customerId === custValId);
            return custTx.reduce((acc, tx) => {
                return tx.type === 'gave' ? acc + tx.amount : acc - tx.amount;
            }, 0);
        }

        // Update Overall Dashboard Totals
        function updateDashboard() {
            let totalGet = 0;
            let totalGive = 0;

            customers.forEach(cust => {
                const bal = getCustomerBalance(cust.id);
                if (bal > 0) {
                    totalGet += bal;
                } else {
                    totalGive += Math.abs(bal);
                }
            });

            document.getElementById('total-get').innerText = `₹${totalGet}`;
            document.getElementById('total-give').innerText = `₹${totalGive}`;

            const net = totalGet - totalGive;
            const netElem = document.getElementById('net-balance');
            netElem.innerText = `₹${Math.abs(net)} ${net >= 0 ? '(You Get)' : '(You Pay)'}`;
            netElem.className = `amount ${net >= 0 ? 'text-green' : 'text-red'}`;
        }

        // --- Export Single Customer PDF ---
        function exportCustomerPDF() {
            if (!selectedCustomerId) return;
            const cust = customers.find(c => c.id === selectedCustomerId);
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();

            const balance = getCustomerBalance(cust.id);
            const custTx = transactions.filter(t => t.customerId === selectedCustomerId);
            custTx.sort((a, b) => new Date(a.date) - new Date(b.date));

            // Header
            doc.setFontSize(20);
            doc.setTextColor(26, 115, 232);
            doc.text("KhataBook Transaction Statement", 14, 20);

            doc.setFontSize(12);
            doc.setTextColor(50, 50, 50);
            doc.text(`Customer Name: ${cust.name}`, 14, 30);
            doc.text(`Date Generated: ${new Date().toLocaleDateString()}`, 14, 37);

            let balText = balance >= 0 ? `Net Balance: You Get Rs. ${balance}` : `Net Balance: You Give Rs. ${Math.abs(balance)}`;
            doc.setFontSize(13);
            doc.setTextColor(balance >= 0 ? 30 : 217, balance >= 0 ? 142 : 48, balance >= 0 ? 62 : 37);
            doc.text(balText, 14, 46);

            // Table Header
            let startY = 56;
            doc.setFillColor(240, 240, 240);
            doc.rect(14, startY, 182, 8, 'F');
            doc.setFontSize(10);
            doc.setTextColor(0, 0, 0);
            doc.text("Date", 16, startY + 6);
            doc.text("Note / Details", 50, startY + 6);
            doc.text("You Gave (Rs)", 115, startY + 6);
            doc.text("You Got (Rs)", 155, startY + 6);

            startY += 12;

            custTx.forEach(tx => {
                if (startY > 280) {
                    doc.addPage();
                    startY = 20;
                }
                doc.text(tx.date, 16, startY);
                doc.text(tx.note, 50, startY);
                doc.text(tx.type === 'gave' ? `${tx.amount}` : '-', 115, startY);
                doc.text(tx.type === 'got' ? `${tx.amount}` : '-', 155, startY);
                startY += 8;
            });

            doc.save(`${cust.name}_KhataBook_Statement.pdf`);
        }

        // --- Export Full Overall Summary PDF ---
        function exportFullSummaryPDF() {
            if (customers.length === 0) {
                alert("No customer data available to export!");
                return;
            }

            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();

            doc.setFontSize(20);
            doc.setTextColor(26, 115, 232);
            doc.text("KhataBook Overall Account Summary", 14, 20);

            doc.setFontSize(11);
            doc.setTextColor(80, 80, 80);
            doc.text(`Generated on: ${new Date().toLocaleDateString()}`, 14, 28);

            let totalGet = 0;
            let totalGive = 0;

            customers.forEach(cust => {
                const bal = getCustomerBalance(cust.id);
                if (bal > 0) totalGet += bal;
                else totalGive += Math.abs(bal);
            });

            doc.setFontSize(12);
            doc.setTextColor(30, 142, 62);
            doc.text(`Total You Will Get: Rs. ${totalGet}`, 14, 38);
            doc.setTextColor(217, 48, 37);
            doc.text(`Total You Will Give: Rs. ${totalGive}`, 14, 46);

            // Customer Summary Table
            let startY = 56;
            doc.setFillColor(240, 240, 240);
            doc.rect(14, startY, 182, 8, 'F');
            doc.setFontSize(10);
            doc.setTextColor(0, 0, 0);
            doc.text("Customer Name", 16, startY + 6);
            doc.text("Status", 100, startY + 6);
            doc.text("Amount (Rs)", 150, startY + 6);

            startY += 12;

            customers.forEach(cust => {
                if (startY > 280) {
                    doc.addPage();
                    startY = 20;
                }
                const bal = getCustomerBalance(cust.id);
                const status = bal >= 0 ? "You Get" : "You Give";

                doc.text(cust.name, 16, startY);
                doc.text(status, 100, startY);
                doc.text(`${Math.abs(bal)}`, 150, startY);
                startY += 8;
            });

            doc.save(`KhataBook_Overall_Summary.pdf`);
        }

        // Run on Startup
        init();
    </script>
</body>
</html>
