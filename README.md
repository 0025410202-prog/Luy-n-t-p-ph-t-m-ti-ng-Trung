<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>App Luyện Phát Âm Tiếng Trung</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f5f7fa;
            color: #333;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: white;
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 8px 24px rgba(0,0,0,0.1);
            text-align: center;
            max-width: 450px;
            width: 100%;
        }
        h1 {
            color: #4f46e5;
            margin-bottom: 20px;
        }
        .word-box {
            font-size: 48px;
            font-weight: bold;
            margin: 20px 0;
            color: #1e293b;
        }
        .pinyin {
            font-size: 20px;
            color: #64748b;
            margin-bottom: 30px;
        }
        .btn {
            background-color: #4f46e5;
            color: white;
            border: none;
            padding: 12px 24px;
            font-size: 18px;
            border-radius: 50px;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 4px 6px rgba(79, 70, 229, 0.2);
        }
        .btn:hover {
            background-color: #4338ca;
        }
        .btn:disabled {
            background-color: #cbd5e1;
            cursor: not-allowed;
        }
        .btn-listen {
            background-color: #10b981;
            margin-right: 10px;
        }
        .btn-listen:hover {
            background-color: #059669;
        }
        #status {
            margin-top: 20px;
            font-weight: 500;
            font-size: 16px;
        }
        .success { color: #10b981; }
        .error { color: #ef4444; }
        .info { color: #3b82f6; }
        
        .result-box {
            margin-top: 20px;
            padding: 15px;
            border-radius: 8px;
            background-color: #f8fafc;
            display: none;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>汉语发음 Luyện Phát Âm</h1>
    
    <div class="word-box" id="target-word">你好</div>
    <div class="pinyin" id="target-pinyin">nǐ hǎo</div>
    
    <div>
        <button class="btn btn-listen" id="btn-listen">🔈 Nghe Mẫu</button>
        <button class="btn" id="btn-speak">🎤 Bấm để Đọc</button>
    </div>

    <div id="status" class="info">Sẵn sàng! Hãy nghe mẫu hoặc bấm nút để đọc.</div>

    <div class="result-box" id="result-box">
        <p>Bạn đã nói: <strong id="user-speech" style="font-size: 20px; color: #4f46e5;">...</strong></p>
        <p id="feedback-text" style="font-weight: bold;"></p>
    </div>
</div>

<script>
    // 1. Cấu hình từ vựng cần luyện tập
    const targetWord = "你好";
    
    // 2. Khởi tạo Web Speech API (SpeechRecognition) để nhận diện giọng nói
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    
    if (!SpeechRecognition) {
        document.getElementById('status').innerHTML = "Trình duyệt của bạn không hỗ trợ nhận diện giọng nói. Hãy dùng Google Chrome!";
        document.getElementById('btn-speak').disabled = true;
    }

    const recognition = new SpeechRecognition();
    recognition.lang = 'zh-CN'; // Đặt ngôn ngữ là tiếng Trung Quốc đại lục
    recognition.interimResults = false; // Chỉ lấy kết quả cuối cùng
    recognition.maxAlternatives = 1;

    const btnSpeak = document.getElementById('btn-speak');
    const btnListen = document.getElementById('btn-listen');
    const statusText = document.getElementById('status');
    const resultBox = document.getElementById('result-box');
    const userSpeechText = document.getElementById('user-speech');
    const feedbackText = document.getElementById('feedback-text');

    // TÍNH NĂNG 1: Phát âm mẫu (Sử dụng SpeechSynthesis)
    btnListen.addEventListener('click', () => {
        const utterance = new SpeechSynthesisUtterance(targetWord);
        utterance.lang = 'zh-CN'; // Tiếng Trung
        window.speechSynthesis.speak(utterance);
    });

    // TÍNH NĂNG 2: Nhận diện giọng nói và chấm điểm
    btnSpeak.addEventListener('click', () => {
        recognition.start();
        statusText.innerText = "正在听... Đang nghe, bạn hãy nói đi!";
        statusText.className = "info";
        btnSpeak.disabled = true;
    });

    // Khi nhận diện thành công
    recognition.onresult = (event) => {
        const resultText = event.results[0][0].transcript;
        // Xóa dấu câu nếu Google tự động thêm vào để so sánh chính xác hơn
        const cleanResult = resultText.replace(/[.,\/#!$%\^&\*;:{}=\-_`~()？。，！]/g,""); 
        
        userSpeechText.innerText = resultText;
        resultBox.style.display = "block";

        // So sánh kết quả của người dùng với từ mẫu
        if (cleanResult === targetWord) {
            statusText.innerText = "Thành công!";
            statusText.className = "success";
            feedbackText.innerText = "🎉 Tuyệt vời! Bạn phát âm chính xác 100%.";
            feedbackText.className = "success";
        } else {
            statusText.innerText = "Chưa chính xác!";
            statusText.className = "error";
            feedbackText.innerText = `❌ Thử lại xem sao nhé!`;
            feedbackText.className = "error";
        }
    };

    // Khi kết thúc quá trình nhận diện (dù đúng hay sai)
    recognition.onend = () => {
        btnSpeak.disabled = false;
        if(statusText.innerText.includes("正在听")) {
            statusText.innerText = "Sẵn sàng!";
        }
    };

    // Xử lý lỗi
    recognition.onerror = (event) => {
        statusText.innerText = "Có lỗi xảy ra: " + event.error;
        statusText.className = "error";
        btnSpeak.disabled = false;
    };
</script>

</body>
</html>
