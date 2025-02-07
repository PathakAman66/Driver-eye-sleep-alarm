<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Driver Eye Sleep Alarm</title>
    <script defer src="https://cdn.jsdelivr.net/npm/@tensorflow/tf.min.js"></script>
    <script defer src="https://cdn.jsdelivr.net/npm/face-api.js"></script>
    <style>
        body {
            text-align: center;
            font-family: Arial, sans-serif;
        }
        video {
            width: 60%;
            border: 2px solid black;
            margin-top: 20px;
        }
        canvas {
            position: absolute;
            top: 0;
            left: 0;
        }
        #status {
            font-size: 20px;
            font-weight: bold;
            margin-top: 10px;
            color: red;
        }
    </style>
</head>
<body>

    <h1>Driver Eye Sleep Alarm</h1>
    <video id="video" autoplay></video>
    <p id="status">Detecting...</p>
    <audio id="alarm" src="alarm.mp3"></audio>

    <script>
        const video = document.getElementById("video");
        const statusText = document.getElementById("status");
        const alarm = document.getElementById("alarm");

        async function setupCamera() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: {} });
                video.srcObject = stream;
            } catch (err) {
                console.error("Error accessing webcam: ", err);
            }
        }

        async function loadModels() {
            await faceapi.nets.tinyFaceDetector.loadFromUri('https://cdn.jsdelivr.net/gh/justadudewhohacks/face-api.js/models');
            await faceapi.nets.faceLandmark68Net.loadFromUri('https://cdn.jsdelivr.net/gh/justadudewhohacks/face-api.js/models');
        }

        async function detectEyes() {
            const detections = await faceapi.detectSingleFace(video, new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks();

            if (detections) {
                const landmarks = detections.landmarks;
                const leftEye = landmarks.getLeftEye();
                const rightEye = landmarks.getRightEye();

                const eyeOpenness = (leftEye[1].y - leftEye[5].y + rightEye[1].y - rightEye[5].y) / 2;

                if (eyeOpenness < 6) {
                    statusText.textContent = "Drowsiness Detected! Wake Up!";
                    if (alarm.paused) alarm.play();
                } else {
                    statusText.textContent = "Normal";
                    alarm.pause();
                    alarm.currentTime = 0;
                }
            }

            setTimeout(detectEyes, 500);
        }

        async function start() {
            await loadModels();
            setupCamera();
            video.addEventListener("playing", detectEyes);
        }

        start();
    </script>

</body>
</html>
# Driver-eye-sleep-alarm
Developed a machine learning based eye capture alarm which helps to prevent from night accident
