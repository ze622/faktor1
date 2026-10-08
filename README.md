<!DOCTYPE html>
<html lang="fa" dir="rtl">
<body>
    <input type="text" id="val" placeholder="مثلا: سیب:100">
    <button onclick="doSave()">ذخیره</button>
    <div id="msg">---</div>

    <script>
        function doSave() {
            try {
                let data = document.getElementById('val').value;
                localStorage.setItem('debug_test', data);
                
                // تست فوری بعد از ذخیره
                let check = localStorage.getItem('debug_test');
                if (check === data) {
                    alert("موفقیت! ذخیره شد.");
                } else {
                    alert("خطا: ذخیره شد ولی خوانده نشد!");
                }
            } catch (e) {
                alert("خطای سیستم: " + e.message);
            }
        }
        
        // نمایش دیتای فعلی هنگام لود
        document.getElementById('msg').innerText = "دیتای فعلی: " + localStorage.getItem('debug_test');
    </script>
</body>
</html>
