# faktor2
اپلیکیشن مدیریت و صدور فاکتور هوشمند 
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>سیستم فاکتور کرشمه</title>
    <script src="https://cdn.sheetjs.com/xlsx-0.20.2/package/dist/xlsx.full.min.js"></script>
    <style>
        body { font-family: Tahoma, sans-serif; background-color: #f4f4f9; padding: 20px; direction: rtl; }
        .container { max-width: 600px; margin: auto; background: white; padding: 20px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
        h2 { text-align: center; color: #333; }
        input, textarea { width: 100%; padding: 10px; margin-top: 10px; border: 1px solid #ccc; border-radius: 5px; box-sizing: border-box; }
        button { width: 100%; padding: 12px; margin-top: 15px; background-color: #28a745; color: white; border: none; border-radius: 5px; cursor: pointer; font-size: 16px; }
        button:hover { background-color: #218838; }
        #tableContainer { margin-top: 20px; overflow-x: auto; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: center; font-size: 13px; }
        th { background-color: #eee; }
        .status { font-size: 13px; margin-top: 10px; color: #666; font-weight: bold; text-align: center; }
        #copyButton { background-color: #1677b8; }
        #copyButton:disabled { background-color: #999; cursor: not-allowed; }
    </style>
</head>
<body>

<div class="container">
    <h2>سیستم فاکتور کرشمه</h2>
    <label>۱. بارگذاری فایل اکسل محصولات:</label>
    <input type="file" id="excelFile" accept=".xlsx, .xls">
    <div id="fileStatus" class="status">منتظر انتخاب فایل...</div>
    <hr>
    <label>۲. وارد کردن کالاها (نام و تعداد):</label>
    <textarea id="inputText" rows="5" placeholder="مثال: کرم کرشمه 2"></textarea>
    <button onclick="generateInvoice()">تولید فاکتور</button>
    <button id="copyButton" onclick="copyInvoice()" disabled>کپی فاکتور</button>
    <div id="copyStatus" class="status"></div>
    <div id="tableContainer"></div>
</div>

<script>
    let productDatabase = {};
    let invoiceText = '';

    document.getElementById('excelFile').addEventListener('change', function(e) {
        let file = e.target.files[0];
        if (!file) return;
        let reader = new FileReader();
        reader.onload = function(e) {
            try {
                let data = new Uint8Array(e.target.result);
                let workbook = XLSX.read(data, {type: 'array'});
                let json = XLSX.utils.sheet_to_json(workbook.Sheets[workbook.SheetNames[0]], {header: 1});
                productDatabase = {};
                for(let i = 1; i < json.length; i++) {
                    if(json[i][0] && json[i][1]) productDatabase[String(json[i][0]).trim()] = parseFloat(json[i][1]);
                }
                document.getElementById('fileStatus').innerText = "✅ " + Object.keys(productDatabase).length + " محصول بارگذاری شد.";
            } catch (err) { alert("خطا در خواندن فایل اکسل!"); }
        };
        reader.readAsArrayBuffer(file);
    });

    function findBestMatch(inputName) {
        for (let dbName in productDatabase) {
            if (dbName.toLowerCase().includes(inputName.toLowerCase())) return dbName;
        }
        return null;
    }

    function generateInvoice() {
        let input = document.getElementById('inputText').value.trim();
        let container = document.getElementById('tableContainer');
        if (!input) return alert("لطفاً متن را وارد کنید.");
        if (Object.keys(productDatabase).length === 0) return alert("ابتدا فایل اکسل را بارگذاری کنید!");

        let lines = input.split('\n');
        let htmlTable = '<table><tr><th>کالا</th><th>تعداد</th><th>قیمت واحد</th><th>جمع</th></tr>';
        let grandTotal = 0;
        let plainInvoiceLines = ['فاکتور فروشگاه کرشمه', '', 'کالا | تعداد | قیمت واحد | جمع هر ردیف'];

        for (let line of lines) {
            if (!line.trim()) continue;
            let parts = line.trim().split(/\s+/);
            let lastPart = parts[parts.length - 1];
            let qty = parseInt(lastPart);
            let searchName = "";

            if (!isNaN(qty)) {
                searchName = parts.slice(0, -1).join(' ');
            } else {
                qty = 1;
                searchName = line.trim();
            }

            let matchedName = findBestMatch(searchName);
            if (matchedName) {
                let price = productDatabase[matchedName];
                let lineTotal = price * qty;
                grandTotal += lineTotal;
                plainInvoiceLines.push(`${matchedName} | ${qty} | ${price.toLocaleString()} | ${lineTotal.toLocaleString()}`);
                htmlTable += `<tr><td>${matchedName}</td><td>${qty}</td><td>${price.toLocaleString()}</td><td>${lineTotal.toLocaleString()}</td></tr>`;
            } else {
                plainInvoiceLines.push(`${searchName} (پیدا نشد) | ${qty} | — | —`);
                htmlTable += `<tr style="color:red;"><td>${searchName} (پیدا نشد)</td><td colspan="3"></td></tr>`;
            }
        }

        htmlTable += `<tr style="background:#eee; font-weight:bold;"><td colspan="3">جمع کل</td><td>${grandTotal.toLocaleString()}</td></tr></table>`;
        container.innerHTML = htmlTable;
        plainInvoiceLines.push('', `جمع کل: ${grandTotal.toLocaleString()}`);
        invoiceText = plainInvoiceLines.join('\n');
        document.getElementById('copyButton').disabled = false;
        document.getElementById('copyStatus').innerText = '';
    }

    async function copyInvoice() {
        if (!invoiceText) return alert('ابتدا فاکتور را تولید کنید.');
        const status = document.getElementById('copyStatus');
        try {
            if (navigator.clipboard && typeof navigator.clipboard.writeText === 'function') {
                await navigator.clipboard.writeText(invoiceText);
                status.style.color = 'green';
                status.innerText = '✅ متن فاکتور کپی شد.';
            } else {
                throw new Error('Clipboard API not available');
            }
        } catch (error) {
            const textarea = document.createElement('textarea');
            textarea.value = invoiceText;
            textarea.style.position = 'fixed';
            textarea.style.top = '0';
            textarea.style.opacity = '0';
            document.body.appendChild(textarea);
            textarea.select();
            document.execCommand('copy');
            document.body.removeChild(textarea);
            status.style.color = 'green';
            status.innerText = '✅ متن فاکتور کپی شد (حالت پشتیبان).';
        }
    }
</script>
</body>
</html>
