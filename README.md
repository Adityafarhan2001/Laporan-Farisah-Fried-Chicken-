<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aplikasi Kasir FFC - Kelola Harga & Stok</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700;800&family=Anton&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Poppins', sans-serif; background-color: #FFFDE7; }
        .font-anton { font-family: 'Anton', sans-serif; }
        .card { background-color: white; border-radius: 0.75rem; box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1); }
        .awning { height: 40px; background-image: repeating-linear-gradient(-45deg, #ffc107, #ffc107 30px, #ef4444 30px, #ef4444 60px); border-bottom: 5px solid #b91c1c; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.2); }
        .menu-button { background-color: #FFD60A; color: #B91C1C; border: 2px solid #B91C1C; padding: 0.5rem 0.75rem; border-radius: 0.5rem; text-align: center; transition: all 0.2s ease-in-out; box-shadow: 0 2px 4px 0 rgb(0 0 0 / 0.1); font-weight: 700; }
        .menu-button:hover:not(:disabled) { background-color: #D32F2F; color: #FFD60A; transform: scale(1.05); }
        .menu-button:disabled { background-color: #e5e7eb; color: #9ca3af; border-color: #d1d5db; cursor: not-allowed; }
        .qty-button { width: 28px; height: 28px; border-radius: 9999px; display: flex; align-items: center; justify-content: center; font-weight: bold; transition: background-color 0.2s; }
        /* Modal Styling */
        #menu-modal-overlay { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background-color: rgba(0, 0, 0, 0.5); z-index: 50; display: none; align-items: center; justify-content: center; }
        /* Template yang tersembunyi */
        .printable-area { display: none; position: absolute; left: -9999px; background: white; color: black; }
        #receipt-template { width: 302px; padding: 15px; font-family: 'Courier New', Courier, monospace; font-size: 12px; }
        #shift-report-template { width: 794px; padding: 40px; font-family: 'Arial', sans-serif; font-size: 12px; }
    </style>
</head>
<body class="p-0 sm:p-4">
    <div id="main-app" class="max-w-screen-2xl mx-auto bg-white sm:rounded-xl shadow-2xl overflow-hidden">
        <!-- HEADER -->
        <header class="relative">
            <div class="awning"></div>
            <div class="bg-yellow-400 p-4 flex justify-between items-center">
                <div class="text-left">
                     <h1 class="font-anton text-3xl sm:text-5xl text-red-700" style="text-shadow: 2px 2px 0 #fff;">FARISAH FRIED CHICKEN</h1>
                     <p class="text-xs sm:text-sm font-semibold text-gray-700 italic">klo berani...cobain aja !</p>
                </div>
                <div class="text-right space-y-2">
                    <input type="date" id="reportDate" class="p-2 border rounded-lg bg-white shadow-sm w-full sm:w-auto">
                    <div>
                        <label for="cashier-name-input" class="text-xs font-semibold text-gray-700">Nama Kasir:</label>
                        <input type="text" id="cashier-name-input" class="p-1 border rounded-md shadow-sm text-sm w-28">
                    </div>
                </div>
            </div>
        </header>

        <main class="grid grid-cols-1 xl:grid-cols-3 gap-6 p-6 bg-yellow-100">
            <!-- KOLOM KIRI: MENU -->
            <section class="xl:col-span-1 flex flex-col gap-6">
                <div class="card p-4">
                    <div class="flex justify-between items-center mb-4">
                        <h2 class="text-2xl font-bold text-red-700">Pilih Menu</h2>
                        <button onclick="openMenuModal()" title="Kelola Menu, Harga & Stok" class="text-gray-600 hover:text-red-700 transition-colors">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.096 2.572-1.065z" /><path stroke-linecap="round" stroke-linejoin="round" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" /></svg>
                        </button>
                    </div>
                    <div id="menu-container" class="flex flex-col gap-6"></div>
                </div>    
            </section>

            <!-- KOLOM TENGAH: TRANSAKSI SAAT INI -->
            <section class="xl:col-span-1 flex flex-col gap-6">
                <div class="card p-4"><h2 class="text-2xl font-bold text-red-700 mb-4">Transaksi Saat Ini</h2><div class="overflow-y-auto min-h-[200px] max-h-[400px] pr-2"><table class="w-full text-left"><tbody id="cart-table-body" class="divide-y divide-gray-200"></tbody></table></div><div class="border-t-2 border-dashed mt-4 pt-4"><div class="flex justify-between items-center font-bold text-xl mb-4"><span class="text-gray-800">Subtotal</span><span id="cart-subtotal">Rp 0</span></div><button onclick="saveTransaction()" class="w-full bg-red-600 hover:bg-red-700 text-white font-bold py-3 rounded-lg text-lg transition-all duration-300 shadow-lg hover:shadow-xl transform hover:scale-105 flex items-center justify-center gap-2"><svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg>Simpan Transaksi</button></div></div>
            </section>

            <!-- KOLOM KANAN: LAPORAN -->
            <section class="xl:col-span-1 flex flex-col gap-6">
                <div class="card p-4"><h2 class="text-2xl font-bold text-red-700 mb-4">Produk Terlaris Hari Ini</h2><div id="best-sellers-container" class="space-y-2"></div></div>
                <div class="card p-4"><h2 class="text-2xl font-bold text-red-700 mb-4">Riwayat Transaksi</h2><div id="transaction-history" class="overflow-y-auto max-h-[250px] pr-2 space-y-2"></div></div>
                <div class="card p-4"><div class="flex justify-between items-center mb-4"><h2 class="text-2xl font-bold text-red-700">Pengeluaran</h2><button onclick="addExpenseRow()" class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-semibold py-1 px-3 rounded-md text-sm transition-colors">+ Tambah</button></div><div class="overflow-y-auto max-h-[150px] pr-2"><table class="w-full"><tbody id="expense-table-body" class="divide-y divide-gray-200"></tbody></table></div></div>
                <div class="card p-6 bg-gray-800 text-white sticky top-6">
                    <div class="flex justify-between items-start">
                        <h2 class="text-xl font-bold mb-4 text-yellow-300">Ringkasan Hari Ini</h2>
                        <button onclick="resetDailyData()" title="Reset Data Hari Ini" class="text-gray-400 hover:text-white"><svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg></button>
                    </div>
                    <div class="space-y-3 text-lg">
                        <div class="flex justify-between items-center"><span class="text-gray-300">Pendapatan Kotor</span><span id="finance-pendapatan-kotor" class="font-semibold text-white">Rp 0</span></div>
                        <div class="flex justify-between items-center"><span class="text-gray-300">Total Pengeluaran</span><span id="finance-total-pengeluaran" class="font-semibold text-red-400">(Rp 0)</span></div>
                        <hr class="my-2 border-gray-600 border-dashed">
                        <div class="flex justify-between items-center text-2xl"><span class="font-bold text-yellow-300">LABA BERSIH</span><span id="finance-laba-bersih" class="font-extrabold text-green-400">Rp 0</span></div>
                    </div>
                    <button onclick="generateShiftReport()" class="w-full mt-4 bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 rounded-lg text-md transition-colors flex items-center justify-center gap-2"><svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 20 20" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" /></svg>Buat Laporan Shift (PDF)</button>
                </div>
            </section>
        </main>
    </div>

    <!-- Modal Kelola Menu -->
    <div id="menu-modal-overlay">
        <div class="bg-white rounded-lg shadow-xl w-full max-w-lg m-4">
            <div class="p-4 border-b flex justify-between items-center">
                <h3 class="text-xl font-bold text-gray-800">Kelola Menu, Harga & Stok</h3>
                <button onclick="closeMenuModal()" class="text-gray-500 hover:text-gray-800 text-2xl">&times;</button>
            </div>
            <div class="p-4 max-h-96 overflow-y-auto">
                <div id="menu-management-list" class="space-y-4"></div>
            </div>
            <div class="p-4 border-t bg-gray-50 flex justify-end gap-3">
                <button onclick="closeMenuModal()" class="bg-gray-200 text-gray-700 px-4 py-2 rounded-lg hover:bg-gray-300 font-semibold">Batal</button>
                <button onclick="saveMenuChanges()" class="bg-red-600 text-white px-4 py-2 rounded-lg hover:bg-red-700 font-semibold">Simpan Perubahan</button>
            </div>
        </div>
    </div>

    <!-- Template Struk (tersembunyi) -->
    <div id="receipt-template" class="printable-area">
        <div id="receipt-content" class="text-center">
            <h3 class="font-bold text-lg">FARISAH FRIED CHICKEN</h3><p>Jl. Contoh Usaha No. 123, Depok</p><p>------------------------------</p><div class="text-left text-xs"><p>No: <span id="receipt-id"></span></p><p>Tgl: <span id="receipt-date"></span></p><p>Kasir: <span id="receipt-cashier"></span></p></div><p>------------------------------</p><table id="receipt-items" class="w-full text-xs"></table><p>------------------------------</p><div class="text-right font-bold">TOTAL: <span id="receipt-total"></span></div><p>------------------------------</p><p class="font-semibold mt-2">Terima Kasih!</p><p class="text-xs">klo berani...cobain aja !</p>
        </div>
    </div>
    
    <!-- Template Laporan Shift (tersembunyi) -->
    <div id="shift-report-template" class="printable-area">
        <div id="shift-report-content" style="padding: 20px; border: 1px solid #ccc;">
            <h1 style="text-align: center; font-size: 24px; font-weight: bold; margin-bottom: 20px;">Laporan Serah Terima Shift</h1>
            <table style="width: 100%; margin-bottom: 20px; font-size: 14px;">
                <tr><td style="font-weight: bold;">Tanggal:</td><td id="report-date-val"></td><td style="font-weight: bold;">Nama Kasir:</td><td id="report-cashier-val"></td></tr>
            </table>
            <hr style="margin-bottom: 20px;"><h2 style="font-size: 18px; font-weight: bold; margin-bottom: 10px;">Ringkasan Keuangan</h2>
            <table style="width: 50%; font-size: 14px; margin-bottom: 20px;">
                <tr><td>Pendapatan Kotor:</td><td id="report-gross-val" style="text-align: right;"></td></tr>
                <tr><td>Total Pengeluaran:</td><td id="report-expense-val" style="text-align: right;"></td></tr>
                <tr style="font-weight: bold; border-top: 1px solid #000;"><td>LABA BERSIH:</td><td id="report-net-val" style="text-align: right;"></td></tr>
            </table>
            <hr style="margin-bottom: 20px;"><h2 style="font-size: 18px; font-weight: bold; margin-bottom: 10px;">Produk Terlaris</h2>
            <div id="report-bestsellers-val"></div>
            <hr style="margin: 20px 0;"><h2 style="font-size: 18px; font-weight: bold; margin-bottom: 10px;">Rincian Transaksi</h2>
            <table id="report-transactions-table" style="width: 100%; border-collapse: collapse; font-size: 12px;"></table>
            <hr style="margin: 20px 0;"><h2 style="font-size: 18px; font-weight: bold; margin-bottom: 10px;">Rincian Pengeluaran</h2>
            <table id="report-expenses-table" style="width: 100%; border-collapse: collapse; font-size: 12px;"></table>
            <hr style="margin: 20px 0;"><h2 style="font-size: 18px; font-weight: bold; margin-bottom: 10px;">Laporan Sisa Stok</h2>
            <table id="report-stock-table" style="width: 100%; border-collapse: collapse; font-size: 12px;"></table>
        </div>
    </div>

    <script>
        // --- GLOBAL STATE ---
        let currentCart = [];
        let completedTransactions = [];
        let expenses = [];
        let menuState = {};
        let cashierName = 'Lita';
        let transactionCounter = 0;

        // --- DATA ---
        const initialMenuData = {
            "Ayam": [ { name: "Paha Atas", price: 9000, stock: 50 }, { name: "Paha Bawah", price: 8000, stock: 50 }, { name: "Dada", price: 9000, stock: 50 }, { name: "Dada Tulang", price: 9000, stock: 40 }, { name: "Sayap", price: 8000, stock: 40 } ],
            "Saus": [ { name: "Saus Barbeque", price: 3000, stock: 100 }, { name: "Saus Geprek", price: 3000, stock: 100 } ],
            "Lainnya": [ { name: "Nasi Putih", price: 5000, stock: 200 }, { name: "Kentang Goreng", price: 10000, stock: 30 }, { name: "Air Mineral", price: 4000, stock: 48 }, { name: "Es Teh Manis", price: 5000, stock: 100 } ]
        };
        
        // --- UTILITY FUNCTIONS ---
        const formatCurrency = (value) => new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(value);
        const getTodayString = () => new Date().toISOString().split('T')[0];

        // --- LOCAL STORAGE FUNCTIONS ---
        function saveStateToLocalStorage() {
            const date = document.getElementById('reportDate').value;
            localStorage.setItem(`ffc_transactions_${date}`, JSON.stringify(completedTransactions));
            localStorage.setItem(`ffc_expenses_${date}`, JSON.stringify(expenses));
            localStorage.setItem(`ffc_menuState_${date}`, JSON.stringify(menuState));
            localStorage.setItem(`ffc_cashierName`, cashierName);
            localStorage.setItem(`ffc_transactionCounter_${date}`, transactionCounter);
        }

        function loadStateFromLocalStorage() {
            const date = document.getElementById('reportDate').value;
            try {
                const savedTransactions = JSON.parse(localStorage.getItem(`ffc_transactions_${date}`) || '[]');
                const savedExpenses = JSON.parse(localStorage.getItem(`ffc_expenses_${date}`) || '[]');
                const savedCashier = localStorage.getItem('ffc_cashierName');
                const savedCounter = parseInt(localStorage.getItem(`ffc_transactionCounter_${date}`) || '0', 10);
                const savedMenuState = JSON.parse(localStorage.getItem(`ffc_menuState_${date}`));

                completedTransactions = savedTransactions;
                expenses = savedExpenses;
                transactionCounter = savedCounter;
                if (savedCashier) {
                    cashierName = savedCashier;
                    document.getElementById('cashier-name-input').value = cashierName;
                }
                if (!savedMenuState) {
                    menuState = JSON.parse(JSON.stringify(initialMenuData));
                } else {
                    menuState = savedMenuState;
                }
            } catch (e) {
                console.error("Gagal memuat data:", e);
                completedTransactions = [];
                expenses = [];
                transactionCounter = 0;
                menuState = JSON.parse(JSON.stringify(initialMenuData));
            }
        }

        // --- RENDER FUNCTIONS (UI Updates) ---
        function renderAll() {
            renderMenu();
            renderCart();
            renderExpenses();
            renderTransactionHistory();
            renderBestSellers();
            calculateFinancialSummary();
        }

        function renderMenu() {
            const menuContainer = document.getElementById('menu-container');
            menuContainer.innerHTML = '';
            for (const category in menuState) {
                const categoryDiv = document.createElement('div');
                categoryDiv.innerHTML = `<h3 class="text-lg font-bold text-red-700 mb-3">${category}</h3>`;
                const gridDiv = document.createElement('div');
                gridDiv.className = 'grid grid-cols-2 lg:grid-cols-2 gap-3';
                menuState[category].forEach(item => {
                    const button = document.createElement('button');
                    button.className = 'menu-button';
                    button.onclick = () => addItemToCart(item.name, item.price);
                    const stock = item.stock;
                    const stockText = stock > 0 ? `Sisa: ${stock}` : 'Habis';
                    button.disabled = stock <= 0;
                    button.innerHTML = `<span>${item.name}</span><span class="block text-sm font-semibold">${stockText}</span>`;
                    gridDiv.appendChild(button);
                });
                categoryDiv.appendChild(gridDiv);
                menuContainer.appendChild(categoryDiv);
            }
        }

        function renderCart() {
            const tableBody = document.getElementById('cart-table-body');
            tableBody.innerHTML = '';
            if (currentCart.length === 0) {
                tableBody.innerHTML = `<tr id="empty-cart-row"><td colspan="3" class="text-center py-12 text-gray-400 font-semibold">Pilih menu di sebelah kiri</td></tr>`;
                document.getElementById('cart-subtotal').textContent = formatCurrency(0);
                return;
            }
            currentCart.forEach(item => {
                const row = tableBody.insertRow();
                row.innerHTML = `
                    <td class="p-2 font-semibold text-gray-700">${item.name}</td>
                    <td class="p-2"><div class="flex items-center justify-center gap-2"><button onclick="updateQuantity('${item.name}', -1)" class="qty-button bg-red-200 text-red-700 hover:bg-red-300">-</button><span class="font-bold w-6 text-center">${item.quantity}</span><button onclick="updateQuantity('${item.name}', 1)" class="qty-button bg-green-200 text-green-700 hover:bg-green-300">+</button></div></td>
                    <td class="p-2 text-right font-bold text-gray-800">${formatCurrency(item.price * item.quantity)}</td>
                `;
            });
            const subtotal = currentCart.reduce((sum, item) => sum + (item.price * item.quantity), 0);
            document.getElementById('cart-subtotal').textContent = formatCurrency(subtotal);
        }

        function renderExpenses() {
            const tableBody = document.getElementById('expense-table-body');
            tableBody.innerHTML = '';
            expenses.forEach((expense) => {
                const row = tableBody.insertRow();
                row.innerHTML = `
                    <td class="py-1 pr-1"><input type="text" value="${expense.name}" onchange="updateExpense('${expense.id}', 'name', this.value)" class="w-full p-2 border rounded-md" placeholder="Keterangan Biaya"></td>
                    <td class="py-1 pl-1 w-2/5"><input type="number" value="${expense.amount}" oninput="updateExpense('${expense.id}', 'amount', this.value)" class="w-full p-2 border rounded-md text-right" placeholder="0"></td>
                    <td class="py-1 pl-2 text-center"><button onclick="removeExpense('${expense.id}')" class="text-gray-400 hover:text-red-500 transition-colors"><svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" /></svg></button></td>
                `;
            });
            calculateFinancialSummary();
        }

        function renderTransactionHistory() {
            const historyContainer = document.getElementById('transaction-history');
            historyContainer.innerHTML = '';
            if (completedTransactions.length === 0) {
                 historyContainer.innerHTML = `<p class="text-center py-8 text-gray-400 font-semibold">Belum ada transaksi</p>`;
                 return;
            }
            completedTransactions.slice().reverse().forEach(trx => {
                const trxDiv = document.createElement('div');
                trxDiv.className = 'bg-white p-2 rounded-md border text-sm flex justify-between items-center';
                const timestamp = new Date(trx.timestamp);
                trxDiv.innerHTML = `
                    <div><span class="font-bold text-gray-700">Transaksi #${trx.id}</span><span class="block text-xs text-gray-500">${timestamp.toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' })} - Kasir: <strong>${trx.cashier || 'N/A'}</strong></span></div>
                    <div class="text-right"><span class="font-bold text-green-700">${formatCurrency(trx.total)}</span><button onclick="generateReceipt('${trx.id}')" class="block text-xs bg-blue-500 text-white px-2 py-0.5 rounded-full mt-1 hover:bg-blue-600">Cetak Struk</button></div>
                `;
                historyContainer.appendChild(trxDiv);
            });
        }
        
        function renderBestSellers() {
            const container = document.getElementById('best-sellers-container');
            container.innerHTML = '';
            const salesCount = {};
            completedTransactions.forEach(trx => {
                trx.items.forEach(item => {
                    salesCount[item.name] = (salesCount[item.name] || 0) + item.quantity;
                });
            });

            const sortedItems = Object.entries(salesCount).sort((a, b) => b[1] - a[1]).slice(0, 3);

            if (sortedItems.length === 0) {
                container.innerHTML = `<p class="text-center py-4 text-gray-400 font-semibold">Belum ada data penjualan</p>`;
                return;
            }

            sortedItems.forEach(([name, count], index) => {
                const itemDiv = document.createElement('div');
                itemDiv.className = 'flex items-center gap-3';
                const medalColors = ['text-yellow-400', 'text-gray-400', 'text-yellow-600'];
                itemDiv.innerHTML = `
                    <div class="font-bold text-lg ${medalColors[index] || 'text-gray-500'}">${index + 1}.</div>
                    <div class="flex-grow font-semibold text-gray-700">${name}</div>
                    <div class="font-bold text-gray-800">${count} terjual</div>
                `;
                container.appendChild(itemDiv);
            });
        }

        function calculateFinancialSummary() {
            const totalPendapatan = completedTransactions.reduce((sum, trx) => sum + trx.total, 0);
            const totalPengeluaran = expenses.reduce((sum, exp) => sum + (parseFloat(exp.amount) || 0), 0);
            const labaBersih = totalPendapatan - totalPengeluaran;
            document.getElementById('finance-pendapatan-kotor').textContent = formatCurrency(totalPendapatan);
            document.getElementById('finance-total-pengeluaran').textContent = `(${formatCurrency(totalPengeluaran)})`;
            const labaElement = document.getElementById('finance-laba-bersih');
            labaElement.textContent = formatCurrency(labaBersih);
            labaElement.classList.toggle('text-red-400', labaBersih < 0);
            labaElement.classList.toggle('text-green-400', labaBersih >= 0);
        }

        // --- DATA HANDLING LOGIC (OFFLINE) ---
        function saveTransaction() {
            if (currentCart.length === 0) return;
            transactionCounter++;
            const newTransaction = {
                id: transactionCounter,
                items: [...currentCart],
                total: currentCart.reduce((sum, item) => sum + (item.price * item.quantity), 0),
                timestamp: new Date().toISOString(),
                cashier: cashierName || 'Unknown'
            };
            completedTransactions.push(newTransaction);
            saveStateToLocalStorage();
            renderAll();
            currentCart = [];
            renderCart();
        }
        
        function addExpenseRow() {
            const newExpense = { id: `exp_${Date.now()}`, name: 'Biaya Baru', amount: 0 };
            expenses.push(newExpense);
            saveStateToLocalStorage();
            renderExpenses();
        }

        function updateExpense(docId, field, value) {
            const expense = expenses.find(e => e.id === docId);
            if (expense) {
                expense[field] = (field === 'amount') ? parseFloat(value) || 0 : value;
                saveStateToLocalStorage();
                calculateFinancialSummary();
            }
        }

        function removeExpense(docId) {
            expenses = expenses.filter(e => e.id !== docId);
            saveStateToLocalStorage();
            renderExpenses();
        }

        // --- CART & STOCK LOGIC (Local) ---
        function findMenuItem(name) {
            for (const category in menuState) {
                const item = menuState[category].find(i => i.name === name);
                if (item) return item;
            }
            return null;
        }

        window.addItemToCart = (name, price) => {
            const menuItem = findMenuItem(name);
            if (!menuItem || menuItem.stock <= 0) return;
            menuItem.stock--;
            const existingItemInCart = currentCart.find(item => item.name === name);
            if (existingItemInCart) { existingItemInCart.quantity++; } 
            else { currentCart.push({ name, price, quantity: 1 }); }
            renderCart();
            renderMenu();
        }

        window.updateQuantity = (name, change) => {
            const itemIndex = currentCart.findIndex(item => item.name === name);
            if (itemIndex === -1) return;
            const menuItem = findMenuItem(name);
            if (!menuItem) return;
            if (change > 0 && menuItem.stock <= 0) return;
            currentCart[itemIndex].quantity += change;
            menuItem.stock -= change;
            if (currentCart[itemIndex].quantity <= 0) {
                currentCart.splice(itemIndex, 1);
            }
            renderCart();
            renderMenu();
        }

        // --- MODAL & PDF & RESET ---
        window.openMenuModal = () => {
            const listContainer = document.getElementById('menu-management-list');
            listContainer.innerHTML = '';
            for (const category in menuState) {
                const categoryTitle = document.createElement('h4');
                categoryTitle.className = 'text-md font-bold text-red-700 border-b pb-1';
                categoryTitle.textContent = category;
                listContainer.appendChild(categoryTitle);
                menuState[category].forEach(item => {
                    const itemDiv = document.createElement('div');
                    itemDiv.className = 'grid grid-cols-3 gap-4 items-center';
                    itemDiv.innerHTML = `
                        <label class="font-semibold text-gray-700 col-span-1">${item.name}</label>
                        <div class="col-span-1"><label class="text-sm">Harga:</label><input type="number" data-name="${item.name}" data-type="price" value="${item.price}" class="w-full p-1 border rounded-md text-right"></div>
                        <div class="col-span-1"><label class="text-sm">Stok:</label><input type="number" data-name="${item.name}" data-type="stock" value="${item.stock}" class="w-full p-1 border rounded-md text-right"></div>
                    `;
                    listContainer.appendChild(itemDiv);
                });
            }
            document.getElementById('menu-modal-overlay').style.display = 'flex';
        }

        window.closeMenuModal = () => {
            document.getElementById('menu-modal-overlay').style.display = 'none';
        }

        window.saveMenuChanges = () => {
            const inputs = document.querySelectorAll('#menu-management-list input');
            inputs.forEach(input => {
                const itemName = input.dataset.name;
                const inputType = input.dataset.type;
                const newValue = parseInt(input.value, 10);
                
                const menuItem = findMenuItem(itemName);
                if (menuItem) {
                    menuItem[inputType] = isNaN(newValue) ? 0 : newValue;
                }
            });
            saveStateToLocalStorage();
            renderMenu();
            closeMenuModal();
        }

        window.resetDailyData = () => {
            const date = document.getElementById('reportDate').value;
            if (confirm(`Anda yakin ingin menghapus semua data untuk tanggal ${date}? Aksi ini tidak bisa dibatalkan.`)) {
                localStorage.removeItem(`ffc_transactions_${date}`);
                localStorage.removeItem(`ffc_expenses_${date}`);
                localStorage.removeItem(`ffc_menuState_${date}`);
                localStorage.removeItem(`ffc_transactionCounter_${date}`);
                loadStateFromLocalStorage();
                renderAll();
            }
        }

        window.generateReceipt = (transactionId) => {
            const { jsPDF } = window.jspdf;
            const trx = completedTransactions.find(t => t.id == transactionId);
            if (!trx) return;
            const timestamp = new Date(trx.timestamp);
            document.getElementById('receipt-id').textContent = trx.id;
            document.getElementById('receipt-date').textContent = timestamp.toLocaleString('id-ID');
            document.getElementById('receipt-cashier').textContent = trx.cashier || 'N/A';
            const itemsTable = document.getElementById('receipt-items');
            itemsTable.innerHTML = '';
            trx.items.forEach(item => {
                const row = itemsTable.insertRow();
                row.innerHTML = `<td class="text-left" colspan="2">${item.name}</td><tr></tr><td class="text-left">${item.quantity}x ${formatCurrency(item.price)}</td><td class="text-right">${formatCurrency(item.quantity * item.price)}</td>`;
            });
            document.getElementById('receipt-total').textContent = formatCurrency(trx.total);
            const receiptElement = document.getElementById('receipt-template');
            receiptElement.style.display = 'block';
            html2canvas(receiptElement, { scale: 2 }).then(canvas => {
                const imgData = canvas.toDataURL('image/png');
                const pdf = new jsPDF({ orientation: 'portrait', unit: 'mm', format: [80, canvas.height * 80 / canvas.width] });
                pdf.addImage(imgData, 'PNG', 0, 0, 80, canvas.height * 80 / canvas.width);
                pdf.save(`struk-ffc-${trx.id}.pdf`);
                receiptElement.style.display = 'none';
            });
        }

        window.generateShiftReport = () => {
            const { jsPDF } = window.jspdf;
            const reportElement = document.getElementById('shift-report-template');
            
            document.getElementById('report-date-val').textContent = new Date(document.getElementById('reportDate').value).toLocaleDateString('id-ID', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
            document.getElementById('report-cashier-val').textContent = cashierName;
            document.getElementById('report-gross-val').textContent = document.getElementById('finance-pendapatan-kotor').textContent;
            document.getElementById('report-expense-val').textContent = document.getElementById('finance-total-pengeluaran').textContent;
            document.getElementById('report-net-val').textContent = document.getElementById('finance-laba-bersih').textContent;
            document.getElementById('report-bestsellers-val').innerHTML = document.getElementById('best-sellers-container').innerHTML;

            const trxTable = document.getElementById('report-transactions-table');
            trxTable.innerHTML = `<thead style="background-color: #eee; font-weight: bold;"><tr><td style="padding: 5px;">Waktu</td><td style="padding: 5px;">Item</td><td style="padding: 5px; text-align: right;">Total</td></tr></thead><tbody></tbody>`;
            const trxBody = trxTable.querySelector('tbody');
            completedTransactions.forEach(trx => {
                const itemsString = trx.items.map(i => `${i.quantity}x ${i.name}`).join(', ');
                const row = trxBody.insertRow();
                row.innerHTML = `<td style="padding: 5px; border-bottom: 1px solid #ddd;">${new Date(trx.timestamp).toLocaleTimeString('id-ID')}</td><td style="padding: 5px; border-bottom: 1px solid #ddd;">${itemsString}</td><td style="padding: 5px; border-bottom: 1px solid #ddd; text-align: right;">${formatCurrency(trx.total)}</td>`;
            });

            const expTable = document.getElementById('report-expenses-table');
            expTable.innerHTML = `<thead style="background-color: #eee; font-weight: bold;"><tr><td style="padding: 5px;">Keterangan</td><td style="padding: 5px; text-align: right;">Nominal</td></tr></thead><tbody></tbody>`;
            const expBody = expTable.querySelector('tbody');
            expenses.forEach(exp => {
                const row = expBody.insertRow();
                row.innerHTML = `<td style="padding: 5px; border-bottom: 1px solid #ddd;">${exp.name}</td><td style="padding: 5px; border-bottom: 1px solid #ddd; text-align: right;">${formatCurrency(exp.amount)}</td>`;
            });

            const stockTable = document.getElementById('report-stock-table');
            stockTable.innerHTML = `<thead style="background-color: #eee; font-weight: bold;"><tr><td style="padding: 5px;">Kategori</td><td style="padding: 5px;">Nama Produk</td><td style="padding: 5px; text-align: right;">Sisa Stok</td></tr></thead><tbody></tbody>`;
            const stockBody = stockTable.querySelector('tbody');
            for (const category in menuState) {
                menuState[category].forEach(item => {
                    const row = stockBody.insertRow();
                    row.innerHTML = `<td style="padding: 5px; border-bottom: 1px solid #ddd;">${category}</td><td style="padding: 5px; border-bottom: 1px solid #ddd;">${item.name}</td><td style="padding: 5px; border-bottom: 1px solid #ddd; text-align: right;">${item.stock}</td>`;
                });
            }

            reportElement.style.display = 'block';
            html2canvas(reportElement, { scale: 2 }).then(canvas => {
                const imgData = canvas.toDataURL('image/png');
                const pdf = new jsPDF({ orientation: 'portrait', unit: 'mm', format: 'a4' });
                const imgProps= pdf.getImageProperties(imgData);
                const pdfWidth = pdf.internal.pageSize.getWidth();
                const pdfHeight = (imgProps.height * pdfWidth) / imgProps.width;
                pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
                pdf.save(`laporan-shift-${cashierName}-${getTodayString()}.pdf`);
                reportElement.style.display = 'none';
            });
        };

        // --- INITIALIZATION ---
        function initApp() {
            const reportDateInput = document.getElementById('reportDate');
            reportDateInput.value = getTodayString();
            reportDateInput.addEventListener('change', () => {
                loadStateFromLocalStorage();
                renderAll();
            });
            
            const cashierInput = document.getElementById('cashier-name-input');
            cashierInput.addEventListener('input', (e) => {
                cashierName = e.target.value;
                localStorage.setItem('ffc_cashierName', cashierName);
            });

            loadStateFromLocalStorage();
            renderAll();
        }

        window.onload = () => {
            initApp();
        };
    </script>
</body>
</html>

