# faktor
اپلیکیشن مدیریت و صدور فاکتور هوشمند 
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>سیستم فاکتور ساز کرشمه</title>

<!-- بارگذاری مستقیم کتابخانه از اینترنت برای جلوگیری از خطای فایل پیدا نشد -->
<script src="https://cdn.sheetjs.com/xlsx-0.20.2/package/dist/xlsx.full.min.js"></script>

<style>
body { font-family: Tahoma, sans-serif; background-color: #f4f4f9; padding: 20px; direction: rtl; }
.container { max-width: 600px; margin: auto; background: white; padding: 20px; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
h2 { text-align: center; color: #333; }
input, textarea { width: 100%; padding: 10px; margin-top: 10px; border: 1px solid #ccc; border-radius: 5px; box-sizing: border-box; }
button { width: 100%; padding: 12px; margin-top: 15px; background-color: #28a745; color: white; border: none; border-radius: 5px; cursor: pointer; font-size: 16px; }
button:hover { background-color: #218838; }
#tableContainer { margin-top: 20px; }
table { width: 100%; border-collapse: collapse; margin-top: 10px; }
th, td { border: 1px solid #ddd; padding: 10px; text-align: center; }
th { background-color: #eee; }
.status { font-size: 12px; margin-top: 5px; color: #666; }
#copyButton { background-color: #1677b8; }
#copyButton:hover { background-color: #12669e; }
#copyButton:disabled { background-color: #999; cursor: not-allowed; }
</style>
</head>
<body>

<div class="container">
<h2>سیستم فاکتور </h2>

<label>۱. بارگذاری فایل اکسل محصولات:</label>
<input type="file" id="excelFile" accept=".xlsx, .xls">
<div id="fileStatus" class="status">منتظر انتخاب فایل...</div>

<hr>

<label>۲. وارد کردن لیست کالاها (نام کالا + تعداد):</label>
<textarea id="inputText" rows="5" placeholder="مثال: مداد  2 پاک کن  1"></textarea>

<button onclick="generateInvoice()">تولید فاکتور</button>
<button id="copyButton" onclick="copyInvoice()" disabled>کپی فاکتور</button>
<div id="copyStatus" class="status" aria-live="polite"></div>

<div id="tableContainer"></div>
</div>

<script>
let productDatabase = {};
let invoiceText = '';

// تابع خواندن فایل اکسل
document.getElementById('excelFile').addEventListener('change', function(e) {
let file = e.target.files[0];
if (!file) return;

let reader = new FileReader();
reader.onload = function(e) {
try {
let data = new Uint8Array(e.target.result);
let workbook = XLSX.read(data, {type: 'array'});
let firstSheetName = workbook.SheetNames[0];
let worksheet = workbook.Sheets[firstSheetName];

// تبدیل اکسل به فرمت JSON
let json = XLSX.utils.sheet_to_json(worksheet, {header: 1});

productDatabase = {};
// فرض: ستون اول نام، ستون دوم قیمت (ردیف اول سرستون است)
for(let i = 1; i < json.length; i++) {
let row = json[i];
if(row[0] !== undefined && row[1] !== undefined) {
let name = String(row[0]).trim();
let price = parseFloat(row[1]);
if(!isNaN(price)) {
productDatabase[name] = price;
}
}
}

document.getElementById('fileStatus').innerText = "✅ فایل با موفقیت خوانده شد. تعداد محصولات: " + Object.keys(productDatabase).length;
document.getElementById('fileStatus').style.color = "green";
} catch (err) {
console.error(err);
alert("خطا در خواندن فایل! مطمئن شوید فایل اکسل صحیح است.");
document.getElementById('fileStatus').innerText = "❌ خطا در خواندن فایل.";
document.getElementById('fileStatus').style.color = "red";
}
};
reader.readAsArrayBuffer(file);
});

// تابع پیدا کردن بهترین تطابق نام (جستجوی منعطف)
function findBestMatch(inputName) {
let bestMatch = null;
let highestScore = 0;
let inputWords = inputName.toLowerCase().split(/\s+/);

for (let dbName in productDatabase) {
let dbNameLower = dbName.toLowerCase();
let score = 0;
inputWords.forEach(word => {
if (word.length > 1 && dbNameLower.includes(word)) {
score += word.length;
}
});
if (score > highestScore) {
highestScore = score;
bestMatch = dbName;
}
}
return highestScore > 0 ? bestMatch : null;
}

// تابع اصلی تولید فاکتور
function generateInvoice() {
let input = document.getElementById('inputText').value.trim();
let container = document.getElementById('tableContainer');

if (!input) return alert("لطفاً متن را وارد کنید.");
if (Object.keys(productDatabase).length === 0) return alert("ابتدا فایل اکسل را بارگذاری کنید!");

let lines = input.split('\n');
let htmlTable = '<table><tr><th>کالا</th><th>تعداد</th><th>قیمت واحد</th><th>جمع</th></tr>';
let grandTotal = 0;
let plainInvoiceLines = ['فاکتور فروشگاه ', '', 'کالا | تعداد | قیمت واحد | جمع هر ردیف'];

for (let line of lines) {
if (!line.trim()) continue;

// جدا کردن تعداد از انتهای خط (اگر عدد باشد)
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
plainInvoiceLines.push(${matchedName} | ${qty} | ${price.toLocaleString()} | ${lineTotal.toLocaleString()});
htmlTable += <tr&gt; &lt;td&gt;${matchedName}</td>
<td>${qty}&lt;/td&gt; &lt;td&gt;${price.toLocaleString()}</td>
<td>${lineTotal.toLocaleString()}&lt;/td&gt; &lt;/tr>;
} else {
plainInvoiceLines.push(${searchName} (پیدا نشد) | ${qty} | — | —);
htmlTable += <tr style="color:red;"&gt;&lt;td&gt;${searchName} (پیدا نشد)</td><td colspan="3"></td></tr>`;
}
}

htmlTable += <tr style="background:#eee; font-weight:bold;"&gt; &lt;td colspan="3"&gt;جمع کل&lt;/td&gt; &lt;td&gt;${grandTotal.toLocaleString()}</td>
</tr></table>`;

container.innerHTML = htmlTable;
plainInvoiceLines.push('', جمع کل: ${grandTotal.toLocaleString()});
invoiceText = plainInvoiceLines.join('\n');
document.getElementById('copyButton').disabled = false;
document.getElementById('copyStatus').innerText = '';
}

// Clipboard API در مرورگرهای پشتیبان؛ textarea مخفی برای Android و file://
async function copyInvoice() {
if (!invoiceText) {
alert('ابتدا فاکتور را تولید کنید.');
return;
}

const status = document.getElementById('copyStatus');
try {
if (navigator.clipboard && typeof navigator.clipboard.writeText === 'function') {
try {
await navigator.clipboard.writeText(invoiceText);
status.style.color = 'green';
status.innerText = '✅ متن فاکتور کپی شد.';
return;
} catch (clipboardError) {
// ممکن است مجوز یا محدودیت file:// مانع API اصلی شود؛ fallback را امتحان می‌کنیم.
}
}

const textarea = document.createElement('textarea');
textarea.value = invoiceText;
textarea.setAttribute('readonly', '');
textarea.style.position = 'fixed';
textarea.style.top = '0';
textarea.style.right = '-9999px';
textarea.style.opacity = '0';
document.body.appendChild(textarea);
textarea.focus();
textarea.select();
textarea.setSelectionRange(0, textarea.value.length);
let copied = false;
try {
copied = document.execCommand('copy');
} finally {
document.body.removeChild(textarea);
}
if (!copied) throw new Error('دستور کپی توسط مرورگر پذیرفته نشد.');
status.style.color = 'green';
status.innerText = '✅ متن فاکتور کپی شد.';
} catch (error) {
console.error('Copy invoice failed:', error);
status.style.color = 'red';
status.innerText = '❌ کپی خودکار انجام نشد. لطفاً مجوز کپی مرورگر را بررسی کنید یا متن فاکتور را دستی انتخاب و کپی کنید.';
alert('کپی فاکتور انجام نشد. لطفاً مجوز کپی مرورگر را بررسی کنید یا متن فاکتور را دستی انتخاب و کپی کنید.');
}
}
</script>

</body>
</html>

