| Level                                           | Jenis Kesalahan                                         | Contoh                                                                                           | Denda                              | Dampak / Alasan                                   |
| :---------------------------------------------- | :------------------------------------------------------ | :----------------------------------------------------------------------------------------------- | :--------------------------------- | :------------------------------------------------ |
| **L1 – Basic Logic / Syntax Error**             | Kesalahan dasar yang seharusnya ketangkep lint/test     | Typo variabel, salah operator (`==` vs `===`), lupa `return`                                     | **Rp 2.000**                       | Kesalahan dasar, tanda kurang cek sebelum commit  |
| **L2 – Code Style / Konsistensi**               | Ganggu readability / konsistensi tim                    | Nama variabel ambigu, indentasi acak, comment lama gak dibersihin                                | **Rp 4.000**                       | Bikin codebase susah dibaca & di-maintain         |
| **L3 – Logic / Flow Error (Menengah)**          | Salah di alur logika / kondisi                          | Salah kondisi `if/else`, async gak `await`, infinite loop                                        | **Rp 8.000**                       | Efek langsung ke hasil akhir, bisa bug runtime    |
| **L3.5 – Error Handling / Exception**           | Gak handle error dengan benar                           | Try-catch kosong, error gak di-log, promise rejection gak di-handle                              | **Rp 10.000**                      | Silent failure, susah debugging production        |
| **L4 – Data Handling / Validation Error**       | Gagal handle data / validasi input                      | Input user gak di-sanitize, null/undefined gak di-handle, parsing asal                           | **Rp 12.000**                      | Potensi error runtime & celah keamanan            |
| **L4.5 – Redundancy / Duplicate Logic**         | Mengulang potongan kode tanpa refactor / abstraction    | Copy-paste logic validasi di banyak file, fungsi mirip tapi beda dikit, gak pakai helper/service | **Rp 14.000**                      | Nambah technical debt, rawan bug saat maintenance |
| **L5 – Performance Issue**                      | Kode gak efisien / boros resource                       | Query di loop (N+1), gak pakai caching, data structure gak optimal                               | **Rp 16.000**                      | Boros resource, ganggu performa sistem            |
| **L6 – Security / Access Control Error**        | Gagal jaga autentikasi / data sensitif                  | Token gak diverifikasi, credential di code, endpoint gak aman                                    | **Rp 20.000**                      | Risiko tinggi, bisa bocor data / di-hack          |
| **L7 – Architectural / Design Flaw (Advanced)** | Logic bisnis nyampur di controller / desain layer kacau | Logic transfer saldo di controller, circular dependency, spaghetti structure                     | **Rp 25.000**                      | Sulit maintenance, rawan duplikasi logic          |
| **L8 – Regression / Repeat Offense**            | Ulang kesalahan yang sama padahal udah di-review        | Commit ulang bug sama, copy-paste code salah                                                     | **Denda ×2 dari level sebelumnya** | Menunjukkan careless / gak belajar dari review    |
| **L9 – Testing / Documentation Gap**            | Gak ada test / dokumentasi untuk fitur penting          | Fitur baru tanpa unit test, API endpoint tanpa docs, complex logic tanpa comment                 | **Rp 6.000**                       | Susah maintain & onboarding, rawan regression     |


---

# 📑 Table of Contents

1. [L1 – Basic Logic / Syntax Error](#-l1--basic-logic--syntax-error)
2. [L2 – Code Style / Konsistensi](#️-l2--code-style--konsistensi)
3. [L3 – Logic / Flow Error](#️-l3--logic--flow-error)
4. [L3.5 – Error Handling / Exception](#-l35--error-handling--exception)
5. [L4 – Data Handling / Validation Error](#-l4--data-handling--validation-error)
6. [L4.5 – Redundancy / Duplicate Logic](#-l45--redundancy--duplicate-logic)
7. [L5 – Performance Issue](#-l5--performance-issue)
8. [L6 – Security / Access Control Error](#-l6--security--access-control-error)
9. [L7 – Architectural / Design Flaw](#️-l7--architectural--design-flaw)
10. [L8 – Regression / Repeat Offense](#-l8--regression--repeat-offense)
11. [L9 – Testing / Documentation Gap](#-l9--testing--documentation-gap)
12. [Tabel Ringkasan Denda](#-tabel-ringkasan-denda)
13. [Tips & Best Practices](#-tips--best-practices)
14. [Tools untuk Mencegah Error](#️-tools-untuk-mencegah-error)
15. [Contoh Perhitungan Denda](#-contoh-perhitungan-denda)
16. [FAQ](#-faq-frequently-asked-questions)

---

# 💻 Panduan Denda Code Review – Lengkap dengan Contoh & Solusi

## 🧩 L1 – Basic Logic / Syntax Error

### **Case 1: Assignment vs Comparison**
**Contoh Salah:**
```js
function isAdult(age) {
  if (age = 18) { // salah, ini assignment
    return true;
  }
}
console.log(isAdult(17)); // output: true ❌
```
**Solusi Benar:**
```js
function isAdult(age) {
  return age >= 18;
}
```

### **Case 2: Lupa Return Statement**
**Real-World Case:**
```js
function calculateDiscount(price, discountPercent) {
  const discount = price * (discountPercent / 100);
  price - discount; // ❌ lupa return
}
const finalPrice = calculateDiscount(100000, 10); // undefined
```
**Solusi Benar:**
```js
function calculateDiscount(price, discountPercent) {
  const discount = price * (discountPercent / 100);
  return price - discount;
}
```

### **Case 3: Typo Variabel**
**Real-World Case:**
```js
const userNmae = "John"; // typo
console.log(`Hello ${userName}`); // ReferenceError ❌
```
**Solusi Benar:**
```js
const userName = "John";
console.log(`Hello ${userName}`);
```

---

## ✏️ L2 – Code Style / Konsistensi

### **Case 1: Nama Variabel Ambigu & Formatting**
**Contoh Salah:**
```js
let a = getUser()
if(a.isactive){
console.log("OK")
}
```
**Solusi Benar:**
```js
const user = getUser();
if (user.isActive) {
  console.log("OK");
}
```

### **Case 2: Magic Numbers & Dead Code**
**Real-World Case:**
```js
function calculateShipping(weight) {
  // const oldRate = 5000; // dead code
  if (weight > 5) {
    return weight * 15000; // magic number ❌
  }
  return 10000; // magic number ❌
}
```
**Solusi Benar:**
```js
const SHIPPING_RATE = {
  BASE: 10000,
  PER_KG_HEAVY: 15000,
  HEAVY_THRESHOLD: 5
};

function calculateShipping(weight) {
  if (weight > SHIPPING_RATE.HEAVY_THRESHOLD) {
    return weight * SHIPPING_RATE.PER_KG_HEAVY;
  }
  return SHIPPING_RATE.BASE;
}
```

### **Case 3: Inconsistent Naming Convention**
**Real-World Case:**
```js
const user_name = "John";  // snake_case
const UserAge = 25;        // PascalCase
const useraddress = "..."; // no separator ❌
```
**Solusi Benar:**
```js
const userName = "John";
const userAge = 25;
const userAddress = "...";
```

---

## ⚙️ L3 – Logic / Flow Error

### **Case 1: Async/Await di forEach**
**Contoh Salah:**
```js
async function getTotal() {
  let total = 0;
  const items = await getItems();
  items.forEach(async (item) => {
    total += await getPrice(item);
  });
  return total; // hasilnya 0 ❌
}
```
**Solusi Benar:**
```js
async function getTotal() {
  const items = await getItems();
  const prices = await Promise.all(items.map(i => getPrice(i)));
  return prices.reduce((a, b) => a + b, 0);
}
```

### **Case 2: Kondisi If/Else Salah**
**Real-World Case:**
```js
function getShippingCost(city, weight) {
  if (city === "Jakarta") {
    return 10000;
  } else if (weight > 5) {
    return 20000;
  } else if (city === "Surabaya") { // ❌ gak akan pernah tercapai jika weight > 5
    return 15000;
  }
  return 25000;
}
// getShippingCost("Surabaya", 6) => 20000, harusnya 15000
```
**Solusi Benar:**
```js
function getShippingCost(city, weight) {
  const baseCost = {
    "Jakarta": 10000,
    "Surabaya": 15000,
    "default": 25000
  };
  
  const cost = baseCost[city] || baseCost.default;
  return weight > 5 ? cost * 2 : cost;
}
```

### **Case 3: Off-by-One Error**
**Real-World Case:**
```js
function getLastThreeOrders(orders) {
  return orders.slice(orders.length - 4); // ❌ ambil 4 item, bukan 3
}
```
**Solusi Benar:**
```js
function getLastThreeOrders(orders) {
  return orders.slice(-3); // atau orders.slice(orders.length - 3)
}
```

---

## 🚨 L3.5 – Error Handling / Exception

### **Case 1: Try-Catch Kosong (Silent Failure)**
**Real-World Case:**
```js
async function sendEmail(to, subject, body) {
  try {
    await emailService.send({ to, subject, body });
  } catch (error) {
    // ❌ error diabaikan, gak ada log, gak ada notifikasi
  }
}
```
**Solusi Benar:**
```js
async function sendEmail(to, subject, body) {
  try {
    await emailService.send({ to, subject, body });
    logger.info(`Email sent to ${to}`);
  } catch (error) {
    logger.error(`Failed to send email to ${to}:`, error);
    throw new Error(`Email delivery failed: ${error.message}`);
  }
}
```

### **Case 2: Promise Rejection Tidak Di-Handle**
**Real-World Case:**
```js
app.post('/checkout', async (req, res) => {
  const order = await createOrder(req.body); // ❌ jika error, server crash
  res.json({ orderId: order.id });
});
```
**Solusi Benar:**
```js
app.post('/checkout', async (req, res) => {
  try {
    const order = await createOrder(req.body);
    res.json({ orderId: order.id });
  } catch (error) {
    logger.error('Checkout failed:', error);
    res.status(500).json({ 
      error: 'Checkout failed', 
      message: error.message 
    });
  }
});
```

### **Case 3: Error Handling Terlalu Generic**
**Real-World Case:**
```js
try {
  const user = await db.users.findByPk(userId);
  const payment = await processPayment(user, amount);
  await sendReceipt(user.email);
} catch (error) {
  return res.status(500).send("Something went wrong"); // ❌ terlalu generic
}
```
**Solusi Benar:**
```js
try {
  const user = await db.users.findByPk(userId);
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  const payment = await processPayment(user, amount);
  await sendReceipt(user.email);
  
  res.json({ success: true, paymentId: payment.id });
} catch (error) {
  if (error.name === 'PaymentError') {
    return res.status(402).json({ error: 'Payment failed', details: error.message });
  }
  if (error.name === 'EmailError') {
    logger.error('Receipt email failed:', error);
    return res.json({ success: true, warning: 'Receipt email failed' });
  }
  logger.error('Unexpected error:', error);
  return res.status(500).json({ error: 'Internal server error' });
}
```

---

## 🧠 L4 – Data Handling / Validation Error

### **Case 1: Input Tidak Di-Validasi**
**Contoh Salah:**
```js
const email = req.body.email;
const user = await db.users.findOne({ where: { email } });
```
**Solusi Benar:**
```js
const { email } = req.body;
if (!email || !email.includes('@')) {
  return res.status(400).send("Email tidak valid");
}
const user = await db.users.findOne({ where: { email } });
```

### **Case 2: Null/Undefined Tidak Di-Handle**
**Real-World Case:**
```js
function getUserFullName(user) {
  return user.firstName + " " + user.lastName; // ❌ crash jika user null
}

const orders = await getOrders();
const total = orders.reduce((sum, o) => sum + o.amount, 0); // ❌ crash jika orders null
```
**Solusi Benar:**
```js
function getUserFullName(user) {
  if (!user) return 'Unknown User';
  return `${user.firstName || ''} ${user.lastName || ''}`.trim();
}

const orders = await getOrders();
const total = (orders || []).reduce((sum, o) => sum + (o.amount || 0), 0);
```

### **Case 3: Type Coercion & Parsing Asal**
**Real-World Case:**
```js
app.get('/products', async (req, res) => {
  const limit = req.query.limit; // ❌ string "10", bukan number
  const products = await db.products.findAll({ limit }); // bisa ambil semua data
  res.json(products);
});

const price = "100.50abc";
const total = price * 2; // ❌ NaN
```
**Solusi Benar:**
```js
app.get('/products', async (req, res) => {
  const limit = parseInt(req.query.limit, 10) || 10;
  if (limit < 1 || limit > 100) {
    return res.status(400).json({ error: 'Limit must be between 1-100' });
  }
  const products = await db.products.findAll({ limit });
  res.json(products);
});

const priceStr = "100.50abc";
const price = parseFloat(priceStr);
if (isNaN(price)) {
  throw new Error('Invalid price format');
}
const total = price * 2;
```

### **Case 4: SQL Injection via String Interpolation**
**Real-World Case:**
```js
const username = req.body.username;
const query = `SELECT * FROM users WHERE username = '${username}'`; // ❌ SQL injection
const user = await db.query(query);
```
**Solusi Benar:**
```js
const username = req.body.username;
const user = await db.users.findOne({ 
  where: { username } // parameterized query
});
// atau dengan raw query:
const [user] = await db.query(
  'SELECT * FROM users WHERE username = ?', 
  [username]
);
```

---

## 🔁 L4.5 – Redundancy / Duplicate Logic

### **Case 1: Kondisi Berulang Tanpa Helper**
**Contoh Salah:**
```js
// Di file A
if (user.status === 'active' && user.role === 'admin') { ... }

// Di file B
if (user.status === 'active' && user.role === 'admin') { ... }

// Di file C
if (user.status === 'active' && user.role === 'admin') { ... }
```
**Solusi Benar:**
```js
// utils/userHelpers.js
export function canAccessAdmin(user) {
  return user.status === 'active' && user.role === 'admin';
}

// Di semua file
import { canAccessAdmin } from './utils/userHelpers';
if (canAccessAdmin(user)) { ... }
```

### **Case 2: Copy-Paste Validation Logic**
**Real-World Case:**
```js
// userController.js
app.post('/register', (req, res) => {
  const { email, password } = req.body;
  if (!email || !email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
    return res.status(400).send('Invalid email');
  }
  if (!password || password.length < 8) {
    return res.status(400).send('Password too short');
  }
  // ... register logic
});

// authController.js
app.post('/update-email', (req, res) => {
  const { email } = req.body;
  if (!email || !email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) { // ❌ duplikasi
    return res.status(400).send('Invalid email');
  }
  // ... update logic
});
```
**Solusi Benar:**
```js
// validators/userValidator.js
export function validateEmail(email) {
  if (!email || !email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
    throw new Error('Invalid email format');
  }
  return true;
}

export function validatePassword(password) {
  if (!password || password.length < 8) {
    throw new Error('Password must be at least 8 characters');
  }
  return true;
}

// userController.js
import { validateEmail, validatePassword } from './validators/userValidator';

app.post('/register', (req, res) => {
  try {
    validateEmail(req.body.email);
    validatePassword(req.body.password);
    // ... register logic
  } catch (error) {
    return res.status(400).send(error.message);
  }
});
```

### **Case 3: Fungsi Mirip Tapi Beda Dikit**
**Real-World Case:**
```js
function calculateUserDiscount(user) {
  if (user.isPremium) return 0.2;
  if (user.orderCount > 10) return 0.1;
  return 0;
}

function calculateProductDiscount(product) { // ❌ hampir sama
  if (product.isFeatured) return 0.2;
  if (product.stock > 100) return 0.1;
  return 0;
}
```
**Solusi Benar:**
```js
function calculateDiscount(entity, rules) {
  for (const rule of rules) {
    if (rule.condition(entity)) {
      return rule.discount;
    }
  }
  return 0;
}

const userDiscountRules = [
  { condition: u => u.isPremium, discount: 0.2 },
  { condition: u => u.orderCount > 10, discount: 0.1 }
];

const productDiscountRules = [
  { condition: p => p.isFeatured, discount: 0.2 },
  { condition: p => p.stock > 100, discount: 0.1 }
];

const userDiscount = calculateDiscount(user, userDiscountRules);
const productDiscount = calculateDiscount(product, productDiscountRules);
```

---

## 🚀 L5 – Performance Issue

### **Case 1: N+1 Query Problem**
**Contoh Salah:**
```js
const users = await db.users.findAll();
for (const user of users) {
  user.orders = await db.orders.findAll({ where: { userId: user.id } });
}
// 1 query untuk users + N query untuk orders = N+1 queries ❌
```
**Solusi Benar:**
```js
const users = await db.users.findAll({ 
  include: [{ model: db.orders }] 
});
// Hanya 1-2 queries dengan JOIN
```

### **Case 2: Tidak Pakai Caching untuk Data Statis**
**Real-World Case:**
```js
app.get('/products/:id', async (req, res) => {
  const product = await db.products.findByPk(req.params.id); // ❌ query DB setiap request
  const categories = await db.categories.findAll(); // ❌ data jarang berubah
  res.json({ product, categories });
});
```
**Solusi Benar:**
```js
const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 600 }); // 10 menit

app.get('/products/:id', async (req, res) => {
  const product = await db.products.findByPk(req.params.id);
  
  let categories = cache.get('categories');
  if (!categories) {
    categories = await db.categories.findAll();
    cache.set('categories', categories);
  }
  
  res.json({ product, categories });
});
```

### **Case 3: Data Structure Tidak Optimal**
**Real-World Case:**
```js
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' },
  // ... 10000 users
];

function findUserById(id) {
  return users.find(u => u.id === id); // ❌ O(n) setiap kali
}

// Dipanggil 1000x
for (let i = 0; i < 1000; i++) {
  const user = findUserById(someId);
}
```
**Solusi Benar:**
```js
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' },
];

// Buat Map untuk O(1) lookup
const userMap = new Map(users.map(u => [u.id, u]));

function findUserById(id) {
  return userMap.get(id); // O(1)
}
```

### **Case 4: Load Semua Data Sekaligus**
**Real-World Case:**
```js
app.get('/orders', async (req, res) => {
  const orders = await db.orders.findAll(); // ❌ bisa jutaan records
  res.json(orders);
});
```
**Solusi Benar:**
```js
app.get('/orders', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = 20;
  const offset = (page - 1) * limit;
  
  const { count, rows } = await db.orders.findAndCountAll({
    limit,
    offset,
    order: [['createdAt', 'DESC']]
  });
  
  res.json({
    orders: rows,
    pagination: {
      total: count,
      page,
      totalPages: Math.ceil(count / limit)
    }
  });
});
```

---

## 🔐 L6 – Security / Access Control Error

### **Case 1: Endpoint Tanpa Authentication**
**Contoh Salah:**
```js
app.post('/deleteUser', async (req, res) => {
  await db.users.destroy({ where: { id: req.body.id } });
  res.send("User deleted");
});
```
**Solusi Benar:**
```js
app.post('/deleteUser', authenticate, async (req, res) => {
  if (!req.user.isAdmin) return res.status(403).send("Forbidden");
  await db.users.destroy({ where: { id: req.body.id } });
  res.send("User deleted");
});
```

### **Case 2: Credential Hardcoded di Code**
**Real-World Case:**
```js
const dbConfig = {
  host: 'localhost',
  user: 'root',
  password: 'admin123', // ❌ password di code
  database: 'production_db'
};

const apiKey = 'sk_live_abc123xyz'; // ❌ API key exposed
```
**Solusi Benar:**
```js
// .env file (jangan commit ke git!)
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=admin123
DB_NAME=production_db
API_KEY=sk_live_abc123xyz

// config.js
require('dotenv').config();

const dbConfig = {
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME
};

const apiKey = process.env.API_KEY;
```

### **Case 3: JWT Token Tidak Di-Verify**
**Real-World Case:**
```js
app.get('/profile', (req, res) => {
  const token = req.headers.authorization?.split(' ')[1];
  const decoded = jwt.decode(token); // ❌ decode tanpa verify
  const user = await db.users.findByPk(decoded.userId);
  res.json(user);
});
```
**Solusi Benar:**
```js
app.get('/profile', async (req, res) => {
  try {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) {
      return res.status(401).json({ error: 'No token provided' });
    }
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET); // verify signature
    const user = await db.users.findByPk(decoded.userId);
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json(user);
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
});
```

### **Case 4: IDOR (Insecure Direct Object Reference)**
**Real-World Case:**
```js
app.get('/orders/:orderId', authenticate, async (req, res) => {
  const order = await db.orders.findByPk(req.params.orderId); // ❌ user bisa akses order orang lain
  res.json(order);
});
```
**Solusi Benar:**
```js
app.get('/orders/:orderId', authenticate, async (req, res) => {
  const order = await db.orders.findOne({
    where: { 
      id: req.params.orderId,
      userId: req.user.id // pastikan order milik user yang login
    }
  });
  
  if (!order) {
    return res.status(404).json({ error: 'Order not found' });
  }
  
  res.json(order);
});
```

### **Case 5: XSS (Cross-Site Scripting)**
**Real-World Case:**
```js
app.get('/search', async (req, res) => {
  const query = req.query.q;
  const results = await searchProducts(query);
  res.send(`
    <h1>Search results for: ${query}</h1>
    <!-- ❌ jika query = <script>alert('XSS')</script> -->
  `);
});
```
**Solusi Benar:**
```js
const escapeHtml = (str) => {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
};

app.get('/search', async (req, res) => {
  const query = escapeHtml(req.query.q);
  const results = await searchProducts(req.query.q);
  res.send(`<h1>Search results for: ${query}</h1>`);
});

// Atau lebih baik: gunakan template engine dengan auto-escaping
```

---

## 🏗️ L7 – Architectural / Design Flaw

### **Case 1: Logic Bisnis di Controller**
**Contoh Salah:**
```js
app.post('/transfer', async (req, res) => {
  const { fromId, toId, amount } = req.body;
  const fromUser = await db.users.findByPk(fromId);
  const toUser = await db.users.findByPk(toId);
  if (fromUser.balance < amount) return res.status(400).send("Saldo tidak cukup");
  fromUser.balance -= amount;
  toUser.balance += amount;
  await fromUser.save();
  await toUser.save();
  res.send("Transfer berhasil");
});
```
**Solusi Benar:**
```js
// services/transferService.js
export async function transferFunds(fromId, toId, amount) {
  return await db.transaction(async (t) => {
    const fromUser = await db.users.findByPk(fromId, { transaction: t, lock: true });
    const toUser = await db.users.findByPk(toId, { transaction: t, lock: true });
    
    if (!fromUser || !toUser) {
      throw new Error("User not found");
    }
    if (fromUser.balance < amount) {
      throw new Error("Insufficient balance");
    }
    
    fromUser.balance -= amount;
    toUser.balance += amount;
    
    await Promise.all([
      fromUser.save({ transaction: t }),
      toUser.save({ transaction: t })
    ]);
    
    return { fromUser, toUser };
  });
}

// controllers/transferController.js
app.post('/transfer', async (req, res) => {
  try {
    const { fromId, toId, amount } = req.body;
    const result = await transferFunds(fromId, toId, amount);
    res.json({ success: true, result });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
```

### **Case 2: Circular Dependency**
**Real-World Case:**
```js
// userService.js
import { getOrdersByUser } from './orderService';

export function getUserWithOrders(userId) {
  const user = getUser(userId);
  user.orders = getOrdersByUser(userId);
  return user;
}

// orderService.js
import { getUserWithOrders } from './userService'; // ❌ circular dependency

export function getOrderWithUser(orderId) {
  const order = getOrder(orderId);
  order.user = getUserWithOrders(order.userId); // infinite loop
  return order;
}
```
**Solusi Benar:**
```js
// userService.js
export function getUser(userId) {
  return db.users.findByPk(userId);
}

// orderService.js
export function getOrdersByUser(userId) {
  return db.orders.findAll({ where: { userId } });
}

export function getOrder(orderId) {
  return db.orders.findByPk(orderId);
}

// aggregationService.js (layer terpisah)
import { getUser } from './userService';
import { getOrdersByUser, getOrder } from './orderService';

export async function getUserWithOrders(userId) {
  const [user, orders] = await Promise.all([
    getUser(userId),
    getOrdersByUser(userId)
  ]);
  return { ...user, orders };
}

export async function getOrderWithUser(orderId) {
  const order = await getOrder(orderId);
  const user = await getUser(order.userId);
  return { ...order, user };
}
```

### **Case 3: God Object / Fat Controller**
**Real-World Case:**
```js
class UserController {
  async register(req, res) { /* 50 lines */ }
  async login(req, res) { /* 40 lines */ }
  async updateProfile(req, res) { /* 60 lines */ }
  async deleteAccount(req, res) { /* 30 lines */ }
  async sendVerificationEmail(req, res) { /* 45 lines */ }
  async resetPassword(req, res) { /* 55 lines */ }
  async uploadAvatar(req, res) { /* 70 lines */ }
  // ... 20 methods lainnya, total 1000+ lines ❌
}
```
**Solusi Benar:**
```js
// Pisah berdasarkan domain/concern

// controllers/authController.js
class AuthController {
  async register(req, res) { /* delegate ke authService */ }
  async login(req, res) { /* delegate ke authService */ }
  async resetPassword(req, res) { /* delegate ke authService */ }
}

// controllers/profileController.js
class ProfileController {
  async updateProfile(req, res) { /* delegate ke profileService */ }
  async uploadAvatar(req, res) { /* delegate ke uploadService */ }
}

// controllers/accountController.js
class AccountController {
  async deleteAccount(req, res) { /* delegate ke accountService */ }
  async sendVerificationEmail(req, res) { /* delegate ke emailService */ }
}

// services/authService.js
export class AuthService {
  async register(userData) { /* business logic */ }
  async login(credentials) { /* business logic */ }
}
```

### **Case 4: Tight Coupling**
**Real-World Case:**
```js
class OrderService {
  async createOrder(orderData) {
    const order = await db.orders.create(orderData);
    
    // ❌ tightly coupled dengan implementasi spesifik
    const nodemailer = require('nodemailer');
    const transporter = nodemailer.createTransport({...});
    await transporter.sendMail({...});
    
    const stripe = require('stripe')('sk_test_...');
    await stripe.charges.create({...});
    
    return order;
  }
}
```
**Solusi Benar:**
```js
// Dependency Injection
class OrderService {
  constructor(emailService, paymentService) {
    this.emailService = emailService;
    this.paymentService = paymentService;
  }
  
  async createOrder(orderData) {
    const order = await db.orders.create(orderData);
    
    // Loose coupling via interfaces
    await this.emailService.send({
      to: orderData.email,
      template: 'order-confirmation',
      data: order
    });
    
    await this.paymentService.charge({
      amount: order.total,
      customerId: orderData.customerId
    });
    
    return order;
  }
}

// Mudah di-test dan di-swap implementasinya
const orderService = new OrderService(
  new EmailService(),
  new StripePaymentService()
);
```

---

## 🔁 L8 – Regression / Repeat Offense

### **Penjelasan:**
Kesalahan yang sama diulang setelah sudah di-review sebelumnya. Ini menunjukkan kurang perhatian atau tidak belajar dari feedback.

### **Case 1: Ulang Error yang Sama**
**Review Pertama:**
```js
// Commit #1 - Di-review: "Gunakan === bukan =="
if (user.role == 'admin') { ... }
```

**Commit Berikutnya (Repeat Offense):**
```js
// Commit #5 - Masih pakai == lagi ❌
if (order.status == 'completed') { ... }
```

**Solusi:**
- Setup ESLint dengan rule `eqeqeq: ["error", "always"]`
- Buat checklist sebelum commit
- Belajar dari feedback review sebelumnya

### **Case 2: Copy-Paste Code yang Sudah Di-Review Salah**
**Review Pertama:**
```js
// File A - Di-review: "Handle null check"
const userName = user.name.toUpperCase(); // ❌ crash jika user null
```

**Commit Berikutnya:**
```js
// File B - Copy-paste code yang sama ❌
const productName = product.name.toUpperCase(); // masih gak ada null check
```

**Solusi:**
```js
const userName = user?.name?.toUpperCase() || 'Unknown';
const productName = product?.name?.toUpperCase() || 'Unknown';
```

---

## 📝 L9 – Testing / Documentation Gap

### **Case 1: Fitur Baru Tanpa Unit Test**
**Real-World Case:**
```js
// services/discountService.js
export function calculateFinalPrice(price, discountCode) {
  if (discountCode === 'PROMO10') return price * 0.9;
  if (discountCode === 'PROMO20') return price * 0.8;
  if (discountCode === 'FREESHIP') return price;
  return price;
}
// ❌ Tidak ada test file
```

**Solusi Benar:**
```js
// services/discountService.js
export function calculateFinalPrice(price, discountCode) {
  const discounts = {
    'PROMO10': 0.9,
    'PROMO20': 0.8,
    'FREESHIP': 1.0
  };
  const multiplier = discounts[discountCode] || 1.0;
  return price * multiplier;
}

// __tests__/discountService.test.js
import { calculateFinalPrice } from '../services/discountService';

describe('calculateFinalPrice', () => {
  test('should apply 10% discount for PROMO10', () => {
    expect(calculateFinalPrice(100000, 'PROMO10')).toBe(90000);
  });
  
  test('should apply 20% discount for PROMO20', () => {
    expect(calculateFinalPrice(100000, 'PROMO20')).toBe(80000);
  });
  
  test('should not apply discount for FREESHIP', () => {
    expect(calculateFinalPrice(100000, 'FREESHIP')).toBe(100000);
  });
  
  test('should return original price for invalid code', () => {
    expect(calculateFinalPrice(100000, 'INVALID')).toBe(100000);
  });
});
```

### **Case 2: API Endpoint Tanpa Dokumentasi**
**Real-World Case:**
```js
// ❌ Tidak ada dokumentasi
app.post('/api/orders', async (req, res) => {
  const order = await createOrder(req.body);
  res.json(order);
});
```

**Solusi Benar:**
```js
/**
 * Create a new order
 * 
 * @route POST /api/orders
 * @access Private (requires authentication)
 * 
 * @param {Object} req.body - Order data
 * @param {string} req.body.userId - User ID
 * @param {Array<Object>} req.body.items - Order items
 * @param {string} req.body.items[].productId - Product ID
 * @param {number} req.body.items[].quantity - Item quantity
 * @param {string} req.body.shippingAddress - Shipping address
 * 
 * @returns {Object} 201 - Created order object
 * @returns {Object} 400 - Validation error
 * @returns {Object} 401 - Unauthorized
 * 
 * @example
 * // Request body:
 * {
 *   "userId": "123",
 *   "items": [
 *     { "productId": "prod_1", "quantity": 2 },
 *     { "productId": "prod_2", "quantity": 1 }
 *   ],
 *   "shippingAddress": "Jl. Sudirman No. 1, Jakarta"
 * }
 * 
 * // Response:
 * {
 *   "id": "order_123",
 *   "userId": "123",
 *   "total": 150000,
 *   "status": "pending",
 *   "createdAt": "2024-01-01T00:00:00Z"
 * }
 */
app.post('/api/orders', authenticate, async (req, res) => {
  try {
    const order = await createOrder(req.body);
    res.status(201).json(order);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
```

### **Case 3: Complex Logic Tanpa Comment**
**Real-World Case:**
```js
function calculateShipping(weight, distance, city) {
  let cost = 10000;
  if (weight > 5) cost += (weight - 5) * 2000;
  if (distance > 100) cost += (distance - 100) * 500;
  if (city === 'Jakarta' || city === 'Surabaya') cost *= 0.9;
  if (weight > 20 && distance > 200) cost *= 1.2;
  return cost;
  // ❌ Logic kompleks tanpa penjelasan
}
```

**Solusi Benar:**
```js
/**
 * Calculate shipping cost based on weight, distance, and city
 * 
 * Pricing rules:
 * - Base cost: Rp 10,000
 * - Additional weight (>5kg): Rp 2,000 per kg
 * - Long distance (>100km): Rp 500 per km
 * - Major cities discount: 10% off for Jakarta & Surabaya
 * - Heavy & far surcharge: 20% extra if weight >20kg AND distance >200km
 * 
 * @param {number} weight - Package weight in kg
 * @param {number} distance - Shipping distance in km
 * @param {string} city - Destination city
 * @returns {number} Total shipping cost in Rupiah
 */
function calculateShipping(weight, distance, city) {
  const BASE_COST = 10000;
  const WEIGHT_THRESHOLD = 5;
  const WEIGHT_RATE = 2000;
  const DISTANCE_THRESHOLD = 100;
  const DISTANCE_RATE = 500;
  const MAJOR_CITIES = ['Jakarta', 'Surabaya'];
  const MAJOR_CITY_DISCOUNT = 0.9;
  const HEAVY_FAR_SURCHARGE = 1.2;
  
  let cost = BASE_COST;
  
  // Additional cost for heavy packages
  if (weight > WEIGHT_THRESHOLD) {
    cost += (weight - WEIGHT_THRESHOLD) * WEIGHT_RATE;
  }
  
  // Additional cost for long distance
  if (distance > DISTANCE_THRESHOLD) {
    cost += (distance - DISTANCE_THRESHOLD) * DISTANCE_RATE;
  }
  
  // Discount for major cities
  if (MAJOR_CITIES.includes(city)) {
    cost *= MAJOR_CITY_DISCOUNT;
  }
  
  // Surcharge for heavy packages going far
  if (weight > 20 && distance > 200) {
    cost *= HEAVY_FAR_SURCHARGE;
  }
  
  return Math.round(cost);
}
```

### **Case 4: Tidak Ada Integration Test**
**Real-World Case:**
```js
// ❌ Hanya ada unit test untuk service, tidak ada integration test untuk API
describe('OrderService', () => {
  test('createOrder should create order', () => {
    // unit test
  });
});
```

**Solusi Benar:**
```js
// __tests__/integration/orders.test.js
import request from 'supertest';
import app from '../../app';
import { setupTestDB, cleanupTestDB } from '../helpers/testDb';

describe('Orders API Integration Tests', () => {
  beforeAll(async () => {
    await setupTestDB();
  });
  
  afterAll(async () => {
    await cleanupTestDB();
  });
  
  describe('POST /api/orders', () => {
    test('should create order with valid data', async () => {
      const orderData = {
        userId: 'user_123',
        items: [
          { productId: 'prod_1', quantity: 2 }
        ],
        shippingAddress: 'Test Address'
      };
      
      const response = await request(app)
        .post('/api/orders')
        .set('Authorization', 'Bearer valid_token')
        .send(orderData)
        .expect(201);
      
      expect(response.body).toHaveProperty('id');
      expect(response.body.userId).toBe('user_123');
      expect(response.body.status).toBe('pending');
    });
    
    test('should return 401 without authentication', async () => {
      await request(app)
        .post('/api/orders')
        .send({})
        .expect(401);
    });
    
    test('should return 400 with invalid data', async () => {
      const response = await request(app)
        .post('/api/orders')
        .set('Authorization', 'Bearer valid_token')
        .send({ invalid: 'data' })
        .expect(400);
      
      expect(response.body).toHaveProperty('error');
    });
  });
});
```

---

# 💰 Tabel Ringkasan Denda
| Level | Jenis Kesalahan | Denda |
|:------|:----------------|:-------|
| L1 | Syntax / Basic Logic | Rp 2.000 |
| L2 | Code Style / Konsistensi | Rp 4.000 |
| L3 | Logic / Flow Error | Rp 8.000 |
| L3.5 | Error Handling / Exception | Rp 10.000 |
| L4 | Data Handling / Validation | Rp 12.000 |
| L4.5 | Redundancy / Duplicate | Rp 14.000 |
| L5 | Performance Issue | Rp 16.000 |
| L6 | Security / Access Control | Rp 20.000 |
| L7 | Architecture / Design Flaw | Rp 25.000 |
| L8 | Regression / Repeat Offense | ×2 dari level sebelumnya |
| L9 | Testing / Documentation Gap | Rp 6.000 |

---

# 🎯 Tips & Best Practices

## ✅ Checklist Sebelum Commit

### **1. Code Quality**
- [ ] Tidak ada syntax error atau typo
- [ ] Gunakan `===` dan `!==` untuk comparison
- [ ] Semua variabel menggunakan naming convention yang konsisten (camelCase)
- [ ] Tidak ada magic numbers (gunakan konstanta)
- [ ] Tidak ada dead code atau comment yang tidak perlu

### **2. Error Handling**
- [ ] Semua async function di-wrap dengan try-catch
- [ ] Error di-log dengan informasi yang cukup
- [ ] Promise rejection di-handle dengan benar
- [ ] Error message informatif (bukan generic "Something went wrong")

### **3. Data Validation**
- [ ] Input user di-validasi sebelum diproses
- [ ] Null/undefined di-handle dengan benar
- [ ] Type checking dilakukan untuk data penting
- [ ] SQL injection prevention (gunakan parameterized query)

### **4. Security**
- [ ] Tidak ada credential hardcoded di code
- [ ] Endpoint penting ada authentication & authorization
- [ ] JWT token di-verify (bukan hanya decode)
- [ ] User input di-sanitize untuk prevent XSS
- [ ] IDOR vulnerability di-check (user hanya bisa akses data miliknya)

### **5. Performance**
- [ ] Tidak ada N+1 query problem
- [ ] Data statis menggunakan caching
- [ ] Pagination untuk data besar
- [ ] Data structure optimal (gunakan Map/Set jika perlu)

### **6. Architecture**
- [ ] Logic bisnis tidak di controller (pindah ke service layer)
- [ ] Tidak ada circular dependency
- [ ] Fungsi tidak terlalu panjang (max 50 lines)
- [ ] Single Responsibility Principle (satu fungsi satu tugas)

### **7. Testing & Documentation**
- [ ] Fitur baru ada unit test
- [ ] API endpoint ada dokumentasi (JSDoc atau Swagger)
- [ ] Complex logic ada comment penjelasan
- [ ] Integration test untuk critical flow

### **8. Code Reusability**
- [ ] Tidak ada duplikasi logic (refactor ke helper/utility)
- [ ] Validation logic di-extract ke validator
- [ ] Common pattern di-abstract ke reusable function

---

## 🛠️ Tools untuk Mencegah Error

### **1. ESLint Configuration**

**Untuk JavaScript/TypeScript General:**
```javascript
// eslint.config.js (ESLint Flat Config)
export default [
  {
    ignores: [
      'node_modules/**',
      'dist/**',
      'build/**',
      'coverage/**',
      '*.min.js',
    ],
  },
  {
    languageOptions: {
      ecmaVersion: 'latest',
      sourceType: 'module',
      globals: {
        console: 'readonly',
        process: 'readonly',
        __dirname: 'readonly',
        __filename: 'readonly',
      },
    },
    rules: {
      // Code Quality & Logic (Mencegah L1 Errors)
      indent: ['error', 2],
      'linebreak-style': ['error', 'unix'],
      quotes: ['error', 'single'],
      semi: ['error', 'always'],
      'no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      'no-console': ['warn'],
      'no-debugger': ['error'],
      'no-var': ['error'],
      'prefer-const': ['error'],
      eqeqeq: ['error', 'always'], // ⭐ Mencegah L1 error (== vs ===)
      curly: ['error', 'all'],
      
      // Code Style & Formatting (Mencegah L2 Errors)
      'brace-style': ['error', '1tbs', { allowSingleLine: true }],
      'comma-dangle': ['error', 'always-multiline'],
      'comma-spacing': ['error', { before: false, after: true }],
      'comma-style': ['error', 'last'],
      'computed-property-spacing': ['error', 'never'],
      'func-call-spacing': ['error', 'never'],
      'key-spacing': ['error', { beforeColon: false, afterColon: true }],
      'keyword-spacing': ['error', { before: true, after: true }],
      'object-curly-spacing': ['error', 'always'],
      'semi-spacing': ['error', { before: false, after: true }],
      'space-before-blocks': ['error', 'always'],
      'space-before-function-paren': ['error', {
        anonymous: 'never',
        named: 'never',
        asyncArrow: 'always',
      }],
      'space-in-parens': ['error', 'never'],
      'space-infix-ops': ['error'],
      'space-unary-ops': ['error', { words: true, nonwords: false }],
      'spaced-comment': ['error', 'always'],
      'arrow-spacing': ['error', { before: true, after: true }],
      'block-spacing': ['error', 'always'],
      
      // Naming Conventions (Mencegah L2 Errors)
      camelcase: ['error', { properties: 'always' }],
      'capitalized-comments': ['error', 'always', { ignoreConsecutiveComments: true }],
      'consistent-this': ['error', 'self'],
      
      // Best Practices
      'eol-last': ['error', 'always'],
      'func-name-matching': ['error'],
      'func-style': ['error', 'declaration', { allowArrowFunctions: true }],
      'max-depth': ['error', 4], // ⭐ Limit complexity
      'max-len': ['error', { code: 255, ignoreUrls: true, ignoreStrings: true }],
      'max-nested-callbacks': ['error', 3],
      'max-params': ['error', 4],
      'max-statements-per-line': ['error', { max: 1 }],
      'new-cap': ['error', { newIsCap: true, capIsNew: false }],
      'new-parens': ['error'],
      'no-array-constructor': ['error'],
      'no-bitwise': ['error'],
      'no-continue': ['error'],
      'no-inline-comments': ['error'],
      'no-lonely-if': ['error'],
      'no-mixed-operators': ['error'],
      'no-multiple-empty-lines': ['error', { max: 2, maxEOF: 1 }],
      'no-new-object': ['error'],
      'no-tabs': ['error'],
      'no-trailing-spaces': ['error'],
      'no-underscore-dangle': ['error', { allowAfterThis: true }],
      'no-unneeded-ternary': ['error'],
      'no-whitespace-before-property': ['error'],
      'object-curly-newline': ['error', { consistent: true }],
      'one-var': ['error', 'never'],
      'operator-assignment': ['error', 'always'],
      'operator-linebreak': ['error', 'after'],
      'padded-blocks': ['error', 'never'],
      'quote-props': ['error', 'as-needed'],
      'sort-vars': ['error'],
      'unicode-bom': ['error', 'never'],
    },
  },
];
```

**Atau untuk Legacy .eslintrc.json:**
```json
{
  "env": {
    "browser": true,
    "node": true,
    "es2021": true
  },
  "extends": ["eslint:recommended"],
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "rules": {
    "indent": ["error", 2],
    "linebreak-style": ["error", "unix"],
    "quotes": ["error", "single"],
    "semi": ["error", "always"],
    "no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
    "no-console": ["warn"],
    "no-debugger": ["error"],
    "no-var": ["error"],
    "prefer-const": ["error"],
    "eqeqeq": ["error", "always"],
    "curly": ["error", "all"],
    "brace-style": ["error", "1tbs", { "allowSingleLine": true }],
    "comma-dangle": ["error", "always-multiline"],
    "camelcase": ["error", { "properties": "always" }],
    "max-depth": ["error", 4],
    "max-params": ["error", 4],
    "max-len": ["error", { "code": 255, "ignoreUrls": true }]
  }
}
```

**Benefit dari config ini:**
- ✅ Auto-detect L1 errors (syntax, `==` vs `===`)
- ✅ Auto-detect L2 errors (naming, formatting)
- ✅ Enforce consistent code style
- ✅ Limit complexity (max depth, max params, max nested callbacks)
- ✅ Prevent common mistakes
- ✅ **Works untuk semua project JavaScript/TypeScript** (Node.js, React, Vue, Express, dll)

### **2. Pre-commit Hook (Husky)**
```json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint && npm run test"
    }
  }
}
```

### **3. TypeScript untuk Type Safety**
```typescript
// Mencegah banyak error di runtime
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

function getUser(id: string): User | null {
  // TypeScript akan error jika return type salah
}
```

---

## 📊 Contoh Perhitungan Denda

### **Scenario 1: Multiple Errors dalam 1 PR**
```
PR #123 - Feature: User Registration
- L1: Typo variabel `userNmae` → Rp 2.000
- L2: Magic number untuk password length → Rp 4.000
- L4: Email tidak di-validasi → Rp 12.000
- L6: Password tidak di-hash → Rp 20.000

Total Denda: Rp 38.000
```

### **Scenario 2: Repeat Offense**
```
PR #100 - Review: "Gunakan === bukan =="
Denda: Rp 2.000 (L1)

PR #105 - Masih pakai == lagi (Repeat Offense)
Denda: Rp 2.000 × 2 = Rp 4.000 (L8)

PR #110 - Masih pakai == lagi (Repeat Offense ke-2)
Denda: Rp 4.000 × 2 = Rp 8.000 (L8)
```

### **Scenario 3: Complex Bug**
```
PR #200 - Bug Fix: Transfer Saldo Error
- L3: Async/await salah di forEach → Rp 8.000
- L5: N+1 query problem → Rp 16.000
- L7: Logic bisnis di controller → Rp 25.000
- L9: Tidak ada unit test → Rp 6.000

Total Denda: Rp 55.000
```

---

## 🎓 Learning Resources

### **Untuk Mencegah Error Level 1-3:**
- ESLint Documentation
- JavaScript Best Practices
- Async/Await Tutorial

### **Untuk Mencegah Error Level 4-6:**
- OWASP Top 10 Security Risks
- Input Validation Best Practices
- JWT Authentication Guide

### **Untuk Mencegah Error Level 7:**
- Clean Architecture
- SOLID Principles
- Design Patterns

### **Untuk Level 9:**
- Jest Testing Framework
- Integration Testing with Supertest
- JSDoc Documentation Guide

---

## 💡 Pro Tips

1. **Setup ESLint + Prettier** - Auto-fix banyak error L1 & L2
2. **Code Review Checklist** - Buat template checklist untuk reviewer
3. **Pair Programming** - Kurangi error dengan review real-time
4. **Test-Driven Development (TDD)** - Tulis test dulu, code kemudian
5. **Regular Refactoring** - Jangan biarkan technical debt menumpuk
6. **Learn from Mistakes** - Catat error yang sering terjadi dan cara mencegahnya
7. **Use TypeScript** - Type safety mencegah banyak runtime error
8. **CI/CD Pipeline** - Automated testing sebelum merge ke main branch

---

## ❓ FAQ (Frequently Asked Questions)

### **Q: Apakah denda berlaku untuk semua jenis project?**
A: Ya, tapi bisa disesuaikan dengan kompleksitas project. Untuk project kecil/personal, bisa dikurangi 50%. Untuk project production/enterprise, bisa ditambah 50%.

### **Q: Bagaimana jika 1 error termasuk multiple level?**
A: Pilih level yang paling tinggi. Contoh: SQL injection (L4 + L6) → pakai L6 (Rp 20.000).

### **Q: Apakah denda bisa di-waive?**
A: Bisa, jika:
- Error terjadi karena requirement berubah mendadak
- Error di legacy code yang baru di-refactor
- Junior developer yang masih learning (warning dulu, denda setelah 3x)

### **Q: Bagaimana cara menghitung denda untuk 1 PR dengan banyak error?**
A: Akumulasi semua error. Contoh:
```
PR #123:
- 2x L1 = Rp 4.000
- 1x L3 = Rp 8.000
- 1x L6 = Rp 20.000
Total = Rp 32.000
```

### **Q: Apakah ada grace period untuk developer baru?**
A: Ya, 2 minggu pertama hanya warning. Minggu ke-3 denda 50%, minggu ke-4 dst denda full.

### **Q: Bagaimana dengan error yang ditemukan setelah merge?**
A: Denda ×1.5 karena sudah masuk production. Plus tanggung jawab hotfix.

### **Q: Apakah reviewer juga kena denda jika miss error?**
A: Ya, reviewer kena 30% dari denda developer jika miss error critical (L6-L7).

### **Q: Bagaimana cara dispute denda?**
A: Diskusi dengan tech lead. Bawa evidence (screenshot, documentation, dll). Keputusan tech lead final.

---

## 🎯 Kesimpulan

Sistem denda code review ini bertujuan untuk:

1. ✅ **Meningkatkan Code Quality** - Developer lebih hati-hati sebelum commit
2. ✅ **Reduce Technical Debt** - Mencegah bug masuk ke production
3. ✅ **Learning Culture** - Developer belajar dari mistake
4. ✅ **Consistency** - Semua developer follow best practices yang sama
5. ✅ **Accountability** - Setiap developer bertanggung jawab atas code-nya

### **Remember:**
> "The best code is code that doesn't need to be reviewed because it's already perfect. But since that's impossible, let's at least make it as good as possible before review."

### **Goal:**
Bukan untuk "menghukum" developer, tapi untuk **membangun habit menulis code berkualitas tinggi**.


**Happy Coding! 🚀**
