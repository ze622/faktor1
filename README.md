<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>مدیریت کالا (عیب‌یابی)</title>
    <style>
        body { font-family: Tahoma; padding: 20px; background: #f4f4f4; }
        .card { background: white; padding: 15px; border-radius: 10px; margin-bottom: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        textarea { width: 100%; height: 80px; margin: 10px 0; }
        button { padding: 10px; cursor: pointer; }
        pre { background: #eee; padding: 10px; font-size: 12px; overflow-x: auto; }
    </style>
</head>
<body>

<div class="card">
    <h3>پنل مدیریت محصولات</h3>
    <textarea id="input" placeholder="نام کالا:قیمت (مثال: برنج:50000)"></textarea>
    <button onclick="saveProduct()">ذخیره کالا</button>
    <button onclick="clearProducts()" style="background:red; color:white;">پاکسازی کامل محصولات</button>
</div>

<div class="card">
    <h4>لیست ذخیره شده در حافظه:</h4>
    <div id="display"></div>
</div>

<script>
    // تابع ذخیره امن
    function saveProduct() {
        let input = document.getElementById('input').value;
        if(!input) return;
        
        let [name, price] = input.split(':');
        
        // خواندن دیتای قبلی
        let data = localStorage.getItem('my_products') || "{}";
        let products = {};
        
        try {
            products = JSON.parse(data);
        } catch(e) {
            products = {}; // اگر خراب بود، ریست کن
        }
        
        // افزودن محصول جدید
        products[name.trim()] = price.trim();
        
        // ذخیره
        localStorage.setItem('my_products', JSON.stringify(products));
        alert("ذخیره شد!");
        showProducts();
    }

    // پاکسازی لیست
    function clearProducts() {
        localStorage.removeItem('my_products');
        alert("لیست محصولات پاک شد. حالا تست کنید.");
        showProducts();
    }

    // نمایش وضعیت
    function showProducts() {
        let data = localStorage.getItem('my_products');
        document.getElementById('display').innerHTML = data ? `<pre>${data}</pre>` : "حافظه خالی است.";
    }

    showProducts();
</script>
</body>
</html>
