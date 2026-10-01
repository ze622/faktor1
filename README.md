<!-- بخش جدید برای وارد کردن لینک کانال -->
<div class="section">
    <h4>۲. دریافت از لینک کانال (آزمایشی)</h4>
    <input type="url" id="channelUrl" placeholder="لینک کانال را اینجا وارد کنید" style="width: 100%; padding: 8px; box-sizing: border-box;">
    <button onclick="fetchFromChannel()" style="background-color: #ff9800; margin-top: 10px;">دریافت محصولات از لینک</button>
    <div id="fetchStatus" style="font-size: 12px; margin-top: 5px;"></div>
</div>

<script>
    async function fetchFromChannel() {
        const url = document.getElementById('channelUrl').value;
        const statusEl = document.getElementById('fetchStatus');
        statusEl.innerText = "در حال تلاش برای خواندن...";
        
        try {
            // تلاش برای گرفتن محتوای صفحه
            const response = await fetch(url);
            const htmlText = await response.text();
            
            // در اینجا باید HTML کانال را تحلیل کنیم که بسیار پیچیده است
            // فعلاً فقط متن را در فیلد قیمت‌ها می‌ریزیم تا ببینیم کار می‌کند یا نه
            document.getElementById('priceListInput').value = "خواندن موفق بود! (نیاز به تحلیل HTML دارد)";
            statusEl.innerText = "اطلاعات دریافت شد!";
            statusEl.style.color = "green";
        } catch (error) {
            console.error(error);
            statusEl.innerText = "خطا! احتمالاً مرورگر اجازه دسترسی نداده است (CORS Error).";
            statusEl.style.color = "red";
        }
    }
</script>
