<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>حسابداری کامل</title>
    <style>
        body { font-family: Tahoma, sans-serif; background: #f4f7f6; padding: 10px; margin: 0; }
        .card { background: white; padding: 15px; border-radius: 12px; margin-bottom: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        textarea, input, select { width: 100%; padding: 12px; margin: 5px 0; border: 1px solid #ccc; border-radius: 8px; box-sizing: border-box; }
        button { background: #3498db; color: white; border: none; padding: 12px; border-radius: 8px; width: 100%; margin-top: 5px; cursor: pointer; }
        .btn-danger { background: #e74c3c; }
        .btn-success { background: #27ae60; }
        .row { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #eee; }
    </style>
</head>
<body>

<div class="card">
    <h3>مدیریت کالاها (ورود دسته‌جمعی)</h3>
    <textarea id="bulkProducts" rows="4" placeholder="نام کالا:قیمت&#10;شلوار:500000&#10;پیراهن:300000"></textarea>
    <button class="btn-success" onclick="saveBulk()">ذخیره همه</button>
    <button onclick="exportCSV()">خروجی اکسل (CSV)</button>
    <div id="prodList" style="margin-top:10px; font-size:12px;"></div>
</div>

<div class="card">
    <h3>ثبت فاکتور</h3>
    <input type="text" id="invNum" placeholder="شماره فاکتور (دستی)">
    <select id="sStore"><option value="1">فروشگاه ۱</option><option value="2">فروشگاه ۲</option></select>
    <textarea id="invItems" rows="3" placeholder="نام کالا تعداد"></textarea>
    <button class="btn-success" onclick="saveInvoice()">ثبت نهایی</button>
</div>

<div class="card">
    <h3>لیست فاکتورها</h3>
    <div id="invList"></div>
</div>

<script>
    let products = JSON.parse(localStorage.getItem('products')) || {};
    let invoices = JSON.parse(localStorage.getItem('invoices')) || [];

    function saveBulk() {
        let text = document.getElementById('bulkProducts').value;
        let lines = text.split('\n');
        lines.forEach(line => {
            let [name, price] = line.split(':');
            if(name && price) products[name.trim()] = { price: parseFloat(price), history: [] };
        });
        localStorage.setItem('products', JSON.stringify(products));
        render();
        alert("ذخیره شد");
    }

    function exportCSV() {
        let csv = "نام کالا,قیمت\n" + Object.entries(products).map(([n, p]) => `${n},${p.price}`).join('\n');
        let blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
        let link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = "products.csv";
        link.click();
    }

    function saveInvoice() {
        let id = document.getElementById('invNum').value || Date.now();
        let store = document.getElementById('sStore').value;
        let lines = document.getElementById('invItems').value.split('\n');
        let total = 0;
        
        lines.forEach(line => {
            let [name, qty] = line.split(' ');
            if(products[name]) total += (products[name].price * (qty || 1));
        });

        let profit = (store == 1) ? (total * 0.7) : (total * 0.5); // فرمول سود نمونه
        invoices.push({ id, store, total, profit });
        localStorage.setItem('invoices', JSON.stringify(invoices));
        render();
    }

    function render() {
        document.getElementById('prodList').innerHTML = Object.keys(products).map(n => `<div>${n}: ${products[n].price}</div>`).join('');
        document.getElementById('invList').innerHTML = invoices.map(i => 
            `<div class="row"><div>فاکتور ${i.id} | سود: ${i.profit}</div><button class="btn-danger" style="width:auto" onclick="deleteInv('${i.id}')">حذف</button></div>`
        ).join('');
    }

    function deleteInv(id) {
        invoices = invoices.filter(i => i.id != id);
        localStorage.setItem('invoices', JSON.stringify(invoices));
        render();
    }

    render();
</script>
</body>
</html>
