<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>منصة تداول العملات الرقمية</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }
        header {
            background-color: #333;
            color: #fff;
            padding: 10px 0;
            text-align: center;
        }
        main {
            padding: 20px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: center;
        }
        th {
            background-color: #f4f4f4;
        }
        button {
            margin: 5px;
            padding: 10px 20px;
            border: none;
            color: #fff;
            background-color: #007bff;
            cursor: pointer;
        }
        button:hover {
            background-color: #0056b3;
        }
        form {
            margin: 0;
        }
        label {
            display: block;
            margin: 10px 0 5px;
        }
        input, select {
            padding: 10px;
            width: 100%;
            box-sizing: border-box;
        }
        #refresh-button {
            background-color: #28a745;
        }
        #refresh-button:hover {
            background-color: #218838;
        }
        #chart-container {
            margin-top: 20px;
        }
        .user-form {
            margin-top: 20px;
            padding: 10px;
            border: 1px solid #ddd;
            background-color: #fff;
            border-radius: 5px;
        }
        .user-form input {
            margin-bottom: 10px;
        }
        #wallet-address {
            margin-top: 20px;
            padding: 10px;
            border: 1px solid #ddd;
            background-color: #fff;
            border-radius: 5px;
        }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@walletconnect/web3-provider@1.6.6/dist/umd/index.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/web3@1.3.6/dist/web3.min.js"></script>
</head>
<body>
    <header>
        <h1>منصة تداول العملات الرقمية</h1>
    </header>
    <main>
        <section id="market">
            <h2>أسواق العملات</h2>
            <button id="refresh-button">تحديث الأسعار</button>
            <table id="crypto-table">
                <thead>
                    <tr>
                        <th>العملة</th>
                        <th>السعر الحالي</th>
                        <th>التغير اليومي</th>
                        <th>التداول</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td>Bitcoin</td>
                        <td id="bitcoin-price">$--</td>
                        <td id="bitcoin-change">--%</td>
                        <td>
                            <button onclick="trade('bitcoin', 'buy')">شراء</button>
                            <button onclick="trade('bitcoin', 'sell')">بيع</button>
                        </td>
                    </tr>
                    <tr>
                        <td>Ethereum</td>
                        <td id="ethereum-price">$--</td>
                        <td id="ethereum-change">--%</td>
                        <td>
                            <button onclick="trade('ethereum', 'buy')">شراء</button>
                            <button onclick="trade('ethereum', 'sell')">بيع</button>
                        </td>
                    </tr>
                    <tr>
                        <td>Ripple</td>
                        <td id="ripple-price">$--</td>
                        <td id="ripple-change">--%</td>
                        <td>
                            <button onclick="trade('ripple', 'buy')">شراء</button>
                            <button onclick="trade('ripple', 'sell')">بيع</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </section>
        <section id="trade">
            <h2>تداول</h2>
            <form id="trade-form">
                <label for="currency">اختر العملة:</label>
                <select id="currency" name="currency">
                    <option value="bitcoin">Bitcoin</option>
                    <option value="ethereum">Ethereum</option>
                    <option value="ripple">Ripple</option>
                </select>
                <label for="amount">الكمية:</label>
                <input type="number" id="amount" name="amount" step="0.01" min="0" required>
                <button type="submit" id="buy-button">شراء</button>
                <button type="button" id="sell-button">بيع</button>
            </form>
        </section>
        <section id="chart-container">
            <h2>الرسم البياني للأسعار</h2>
            <canvas id="crypto-chart"></canvas>
        </section>
        <section class="user-form">
            <h2>ربط المحفظة</h2>
            <button id="connect-wallet">ربط المحفظة</button>
            <div id="wallet-address">لم يتم ربط المحفظة بعد</div>
        </section>
    </main>
    <script>
        async function fetchCryptoPrices() {
            try {
                const response = await fetch('https://api.coingecko.com/api/v3/simple/price?ids=bitcoin,ethereum,ripple&vs_currencies=usd&include_24hr_change=true');
                const data = await response.json();

                document.getElementById('bitcoin-price').textContent = `$${data.bitcoin.usd.toFixed(2)}`;
                document.getElementById('ethereum-price').textContent = `$${data.ethereum.usd.toFixed(2)}`;
                document.getElementById('ripple-price').textContent = `$${data.ripple.usd.toFixed(2)}`;

                document.getElementById('bitcoin-change').textContent = `${data.bitcoin.usd_24h_change.toFixed(2)}%`;
                document.getElementById('ethereum-change').textContent = `${data.ethereum.usd_24h_change.toFixed(2)}%`;
                document.getElementById('ripple-change').textContent = `${data.ripple.usd_24h_change.toFixed(2)}%`;

                updateChart(data);

            } catch (error) {
                console.error('Error fetching crypto prices:', error);
            }
        }

        function updateChart(data) {
            const ctx = document.getElementById('crypto-chart').getContext('2d');
            if (window.cryptoChart) {
                window.cryptoChart.destroy();
            }
            window.cryptoChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['Bitcoin', 'Ethereum', 'Ripple'],
                    datasets: [
                        {
                            label: 'سعر العملة',
                            data: [
                                data.bitcoin.usd,
                                data.ethereum.usd,
                                data.ripple.usd
                            ],
                            borderColor: '#007bff',
                            backgroundColor: 'rgba(0, 123, 255, 0.2)',
                            fill: true
                        }
                    ]
                },
                options: {
                    responsive: true,
                    scales: {
                        x: {
                            beginAtZero: true
                        },
                        y: {
                            beginAtZero: true
                        }
                    }
                }
            });
        }

        function trade(currency, action) {
            const amount = document.getElementById('amount').value;
            if (amount && !isNaN(amount) && amount > 0) {
                alert(`تم ${action} ${amount} ${currency}.`);
            } else {
                alert('يرجى إدخال كمية صحيحة.');
            }
        }

        document.getElementById('trade-form').addEventListener('submit', function(event) {
            event.preventDefault(); // Prevent form submission
            const currency = document.getElementById('currency').value;
            const amount = document.getElementById('amount').value;
            if (document.getElementById('buy-button').clicked) {
                trade(currency, 'شراء');
            } else if (document.getElementById('sell-button').clicked) {
                trade(currency, 'بيع');
            }
        });

        document.getElementById('refresh-button').addEventListener('click', fetchCryptoPrices);

        // Fetch prices on page load
        fetchCryptoPrices();

        // Refresh prices every 60 seconds
        setInterval(fetchCryptoPrices, 60000); // Refresh every 60 seconds

        // Add event listener for trade buttons
        document.getElementById('buy-button').addEventListener('click', function() {
            document.getElementById('trade-form').dispatchEvent(new Event('submit'));
        });
        document.getElementById('sell-button').addEventListener('click', function() {
            document.getElementById('trade-form').dispatchEvent(newEvent('submit'));
        });// Wallet connection
        async function connectWallet() {
            const provider = new WalletConnectProvider.default({
                infuraId: '307ac241abe648e2b493f2bc29fee512', // استخدم معرف Infura الخاص بك هنا
            });

            try {
                await provider.enable();
                const web3 = new Web3(provider);

                const accounts = await web3.eth.getAccounts();
                const walletAddress = accounts[0];

                document.getElementById('wallet-address').textContent = `محفظة مربوطة: ${walletAddress}`;
            } catch (error) {
                console.error('Error connecting to wallet:', error);
            }
        }

        document.getElementById('connect-wallet').addEventListener('click', connectWallet);
    </script>
</body>
</html>
