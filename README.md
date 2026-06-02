<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>App Luyện Phát Âm Tiếng Trung v2</title>
    <style>
        body { font-family: Arial, sans-serif; background: #f0f2f5; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
        .card { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); text-align: center; max-width: 400px; width: 100%; }
        .word { font-size: 42px; font-weight: bold; color: #333; margin: 10px 0; }
        .pinyin { font-size: 18px; color: #666; margin-bottom: 25px; }
        .btn { padding: 12px 24px; font-size: 16px; border: none; border-radius: 25px; cursor: pointer; font-weight: bold; transition: 0.2s; }
        .btn-record { background: #ff4757; color: white; }
        .btn-record.recording { background: #2ed573; animation: pulse 1.5s infinite; }
        #status { margin-top: 20px; font-size: 14px; color: #555; }
        #result { margin-top: 15px; font-weight: bold; font-size: 18px; }
        @keyframes pulse { 0% { transform: scale(1); } 50% { transform: scale(1.05); } 100% { transform: scale(1); } }
    </style>
</head>
<body>

<div class="card">
    <h2>Luyện Phát Âm</h2>
    <div class="word" id="target">谢谢</div>
    <div class="pinyin">xièxie</div>

    <button id="recordBtn" class="btn btn-record">🎤 Bấm để ghi âm</button>
    
    <div id="status">Sẵn sàng thu âm...</div>
    <div id="result"></div>
</div>

<script>
    const recordBtn = document.getElementById('recordBtn');
    const statusText = document.getElementById('status');
    const resultText = document.getElementById('result');
    const targetWord = document.getElementById('target').innerText;

    let mediaRecorder;
    let audioChunks = [];
    let isRecording = false;

    recordBtn.addEventListener('click', async () => {
        if (!isRecording) {
            // 1. Yêu cầu quyền Micro và bắt đầu thu âm
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
                mediaRecorder = new MediaRecorder(stream);
                audioChunks = [];

                mediaRecorder.ondataavailable = event => {
                    audioChunks.push(event.data);
                };

                mediaRecorder.onstop = async () => {
                    statusText.innerText = "🔄 Đang gửi âm thanh lên server phân tích...";
                    const audioBlob = new Blob(audioChunks, { type: 'audio/wav' });
                    
                    // 2. Gửi file âm thanh qua Backend Node.js
                    const formData = new FormData();
                    formData.append('audio', audioBlob);
                    formData.append('target', targetWord);

                    try {
                        const response = await fetch('http://localhost:3000/api/recognize', {
                            method: 'POST',
                            body: formData
                        });
                        const data = await response.json();
                        
                        if(data.success) {
                            resultText.innerHTML = `Bạn nói: <span style="color:#2ed573">${data.text}</span><br>${data.match ? '🎉 Chính xác!' : '❌ Chưa đúng, thử lại nhé!'}`;
                        } else {
                            resultText.innerText = "❌ Lỗi: " + data.error;
                        }
                    } catch (err) {
                        resultText.innerText = "❌ Không kết nối được tới Server Backend.";
                    }
                    statusText.innerText = "Sẵn sàng thu âm...";
                };

                mediaRecorder.start();
                isRecording = true;
                recordBtn.innerText = "🛑 Bấm để Dừng & Chấm điểm";
                recordBtn.classList.add('recording');
                statusText.innerText = "🔴 Đang ghi âm giọng nói của bạn...";
                resultText.innerText = "";
            } catch (err) {
                statusText.innerText = "❌ Không thể truy cập Micro: " + err.message;
            }
        } else {
            // 3. Dừng thu âm
            mediaRecorder.stop();
            isRecording = false;
            recordBtn.innerText = "🎤 Bấm để ghi âm";
            recordBtn.classList.remove('recording');
        }
    });
</script>

</body>
</html>
