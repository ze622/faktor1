<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>حسابداری هوشمند</title>
    <style>
        body { font-family: Tahoma, sans-serif; background: #f0f2f5; padding: 10px; }
        .card { background: white; padding: 15px; border-radius: 10px; margin-bottom: 10px; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
        .tab-btn { background: #ddd; border: none; padding: 10px; cursor: pointer; border-radius: 5px; margin-left: 5px; }
        .active-tab { background: #34495e; color: white; }
        .page { display: none; } .active-page { display: block; }
        input, textarea, select { width: 100%; padding: 8px; margin: 5px 0; border: 1px solid #ccc; border-radius: 5px; }
        button { background: #27ae60; color: white; border: none; padding: 10px; width: 100%; cursor: pointer; border-radius: 5px; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; font-size: 13px; }
        th, td { border: 1px solid #eee; padding: 8px; text-align: center; }
    </style>
</head>
<body>

<div style="margin-bottom: 15px;">
    <button class="tab-btn active-tab" onclick="switchPage('prod')">مدیریت کالاها</button>
    <button class="tab-btn" onclick="switchPage('inv')">فاکتورها</button>
</div>

<!-- صفحه کالاها -->
<div id="page-prod" class="page active-page">
    <div class="card">
        <h3>تعریف/ویرایش کالا</h3>
        <input type="text" id="pName" placeholder="نام کالا">
        <input type="number" id="pPrice" placeholder="قیمت">
        <button onclick="saveProduct()">ذخیره کالا</button>
    </div>
    <div class="card">
        <table id="prodTable"><thead><tr><th>کالا</th><th>قیمت</th><th>تاریخچه</th></tr></thead><tbody id="prodBody"></tbody></table>
    </div>
</div>

<!-- صفحه فاکتورها -->
<div id="page-inv" class="page">
    <div class="card">
        <h3>ثبت فاکتور جدید</h3>
        <select id="sStore"><option value="1">فروشگاه ۱</option><option value="2">فروشگاه ۲</option></select>
        <textarea id="invoiceText" rows="3" placeholder="مثال:&#10;شلوار 2&#10;پیراهن 1"></textarea>
        <button onclick="processInvoice()">محاسبه و ثبت</button>
    </div>
    <div class="card">
        <h3>لیست فاکتورها</h3>
        <table id="invTable"><thead><tr><th>فاکتور</th><th>جمع</th><th>سود</th><th>مشاهده</th></tr></thead><tbody id="invBody"></tbody></table>
    </div>
</div>

<script>
    let products = JSON.parse(localStorage.getItem('products')) || {};
    let invoices = JSON.parse(localStorage.getItem('invoices')) || [];

    // صفحه بندی
    function switchPage(page) {
        document.querySelectorAll('.page').forEach(p => p.classList.remove('active-page'));
        document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active-tab'));
        document.getElementById('page-' + page).classList.add('active-page');
        event.target.classList.add('active-tab');
    }

    // مدیریت کالا
    function saveProduct() {
        let name = document.getElementById('pName').value.trim();
        let price = parseFloat(document.getElementById('pPrice').value);
        if(!name || !price) return;
        
        let history = products[name] ? products[name].history : [];
        history.push({ price, date: new Date().toLocaleDateString('fa-IR') });
        products[name] = { price, history };
        localStorage.setItem('products', JSON.stringify(products));
        render();
    }

    function showHistory(name) {
        let h = products[name].history.map(x => `${x.date}: ${x.price}`).join('\n');
        alert("تاریخچه قیمت " + name + ":\n" + h);
    }

    // جستجوی هوشمند
    function findProductPrice(searchName) {
        let keys = Object.keys(products);
        // پیدا کردن اولین کلیدی که بخشی از نامش با ورودی مشابه است
        let match = keys.find(k => k.includes(searchName) || searchName.includes(k));
        return match ? products[match].price : 0;
    }

    // مدیریت فاکتور
    function processInvoice() {
        let lines = document.getElementById('invoiceText').value.trim().split('\n');
        let store = document.getElementById('sStore').value;
        let totalSale = 0;
        let items = [];

        lines.forEach(line => {
            let parts = line.split(' ');
            let search = parts[0];
            let qty = parseInt(parts[1]) || 1;
            let price = findProductPrice(search);
            if(price > 0) {
                totalSale += (price * qty);
                items.push({name: search, price, qty});
            }
        });

        if(totalSale === 0) return alert("کالایی یافت نشد!");
        let sumS1 = invoices.filter(i => i.store == '1').reduce((a, b) => a + b.sale, 0);
        let profit = (store == '1') ? (totalSale * 0.70) : (totalSale - (sumS1 * 0.30) - (totalSale * 0.12));

        invoices.push({ id: Date.now(), store, sale: totalSale, profit, items });
        localStorage.setItem('invoices', JSON.stringify(invoices));
        render();
        alert("ثبت شد.");
    }

    function viewInvoice(id) {
        let inv = invoices.find(x => x.id == id);
        let itemsText = inv.items.map(i => `${i.name} (${i.qty})`).join('\n');
        let choice = prompt("مشاهده و کپی فاکتور:\n\n" + itemsText + "\n\nبرای کپی، متن بالا را کپی کنید.\nبرای حذف، 'حذف' را تایپ کنید.");
        if(choice === 'حذف') {
            invoices = invoices.filter(x => x.id != id);
            localStorage.setItem('invoices', JSON.stringify(invoices));
            render();
        }
    }

    function render() {
        // رندر کالاها
        let pBody = document.getElementById('prodBody');
        pBody.innerHTML = Object.entries(products).map(([name, data]) => 
            `<tr><td>${name}</td><td>${data.price}</td><td onclick="showHistory('${name}')" style="cursor:pointer">📜</td></tr>`
        ).join('');
        
        // رندر فاکتورها
        let iBody = document.getElementById('invBody');
        iBody.innerHTML = invoices.map(inv => 
            `<tr><td>فـ${inv.store}</td><td>${inv.sale}</td><td>${inv.profit.toFixed(0)}</td><td onclick="viewInvoice(${inv.id})" style="cursor:pointer">🔍</td></tr>`
        ).join('');
    }
    render();
</script>
</body>
</html>
