<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WebCut - Видеоредактор</title>
    <style>
        body {
            margin: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #121212;
            color: #ffffff;
            display: flex;
            flex-direction: column;
            height: 100vh;
        }
        header {
            background-color: #1e1e1e;
            padding: 15px 20px;
            font-size: 1.2rem;
            font-weight: bold;
            border-bottom: 1px solid #333;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .main-workspace {
            display: flex;
            flex: 1;
            padding: 20px;
            gap: 20px;
        }
        .preview-area {
            flex: 2;
            background-color: #000;
            border-radius: 8px;
            display: flex;
            justify-content: center;
            align-items: center;
            border: 1px solid #333;
        }
        .controls-area {
            flex: 1;
            background-color: #1e1e1e;
            padding: 20px;
            border-radius: 8px;
            border: 1px solid #333;
        }
        .timeline {
            height: 150px;
            background-color: #1e1e1e;
            border-top: 1px solid #333;
            padding: 20px;
            display: flex;
            gap: 10px;
            overflow-x: auto;
            align-items: center;
        }
        .track-item {
            background-color: #3a3a3a;
            height: 80px;
            min-width: 120px;
            border-radius: 4px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 0.9rem;
            border: 2px solid #555;
        }
        .btn {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 1rem;
            font-weight: bold;
            transition: 0.3s;
        }
        .btn:hover { background-color: #45a049; }
        .file-input-wrapper { margin-bottom: 20px; }
        video { max-width: 100%; max-height: 100%; border-radius: 8px; }
    </style>
</head>
<body>

    <header>
        <div>WebCut 🎬</div>
        <button class="btn" id="exportBtn">Склеить и скачать</button>
    </header>

    <div class="main-workspace">
        <div class="preview-area">
            <video id="videoPreview" controls>
                Ваш браузер не поддерживает видео.
            </video>
        </div>
        <div class="controls-area">
            <h3>Добавить медиа</h3>
            <div class="file-input-wrapper">
                <label>Видео 1:</label><br>
                <input type="file" id="video1" accept="video/mp4,video/webm">
            </div>
            <div class="file-input-wrapper">
                <label>Видео 2:</label><br>
                <input type="file" id="video2" accept="video/mp4,video/webm">
            </div>
            <p id="statusText" style="color: #aaa; font-size: 0.9rem;">Ожидание файлов...</p>
        </div>
    </div>

    <div class="timeline" id="timeline">
        <div style="color: #777;">Таймлайн:</div>
        </div>

    <script src="https://unpkg.com/@ffmpeg/ffmpeg@0.11.0/dist/ffmpeg.min.js"></script>
    
    <script>
        const { createFFmpeg, fetchFile } = FFmpeg;
        const ffmpeg = createFFmpeg({ log: true });

        const video1Input = document.getElementById('video1');
        const video2Input = document.getElementById('video2');
        const exportBtn = document.getElementById('exportBtn');
        const statusText = document.getElementById('statusText');
        const timeline = document.getElementById('timeline');
        const videoPreview = document.getElementById('videoPreview');

        // Функция для отображения видео на таймлайне
        function addToTimeline(name) {
            const el = document.createElement('div');
            el.className = 'track-item';
            el.innerText = name;
            timeline.appendChild(el);
        }

        video1Input.addEventListener('change', (e) => addToTimeline(e.target.files[0].name));
        video2Input.addEventListener('change', (e) => addToTimeline(e.target.files[0].name));

        exportBtn.addEventListener('click', async () => {
            const file1 = video1Input.files[0];
            const file2 = video2Input.files[0];

            if (!file1 || !file2) {
                alert("Пожалуйста, загрузите оба видео!");
                return;
            }

            statusText.innerText = "Загрузка движка редактора...";
            if (!ffmpeg.isLoaded()) {
                await ffmpeg.load();
            }

            statusText.innerText = "Обработка видео (это может занять время)...";
            
            // Записываем файлы в виртуальную файловую систему браузера
            ffmpeg.FS('writeFile', '1.mp4', await fetchFile(file1));
            ffmpeg.FS('writeFile', '2.mp4', await fetchFile(file2));

            // Создаем текстовый файл со списком видео для склейки
            const listContent = "file '1.mp4'\nfile '2.mp4'";
            ffmpeg.FS('writeFile', 'list.txt', listContent);

            // Команда склейки (конкатенации)
            await ffmpeg.run('-f', 'concat', '-safe', '0', '-i', 'list.txt', '-c', 'copy', 'output.mp4');

            statusText.innerText = "Готово! Сохранение...";

            // Достаем результат
            const data = ffmpeg.FS('readFile', 'output.mp4');
            const videoBlob = new Blob([data.buffer], { type: 'video/mp4' });
            const videoUrl = URL.createObjectURL(videoBlob);

            // Показываем в плеере
            videoPreview.src = videoUrl;

            // Скачиваем файл
            const a = document.createElement('a');
            a.href = videoUrl;
            a.download = 'webcut_merged.mp4';
            a.click();

            statusText.innerText = "Успешно склеено!";
        });
    </script>
</body>
</html>
