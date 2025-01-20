# card-creation<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Customize Your Card</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(120deg, #f7e7d3, #a1d6b0); /* Soft pastel cream and green */
            margin: 0;
            padding: 0;
            color: #333;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            perspective: 1000px;
        }

        h1 {
            margin: 20px 0;
            text-align: center;
            color: #fff;
            font-size: 36px;
            text-shadow: 3px 3px 5px rgba(0, 0, 0, 0.2);
        }

        .card {
            width: 320px;
            height: 300px;
            margin: 40px;
            position: relative;
            transform-style: preserve-3d;
            transform: rotateY(0);
            transition: transform 0.5s;
            border-radius: 20px;
        }

        .card .card-front, .card .card-back {
            position: absolute;
            width: 100%;
            height: 100%;
            background-color: #fdf6e3; /* Pastel cream color */
            border-radius: 20px;
            box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1);
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
            backface-visibility: hidden;
        }

        .card .card-front {
            display: flex;
            align-items: center;
            justify-content: center;
            flex-direction: column;
        }

        .card .card-text {
            font-size: 18px;
            font-weight: bold;
            color: #4f8a3e; /* Dark pastel green */
            position: absolute;
            cursor: move;
            user-select: none;
        }

        .card .card-back {
            transform: rotateY(180deg);
            background-color: #fae1c4; /* Pastel orange color */
        }

        .controls {
            width: 80%;
            max-width: 500px;
            margin: 40px auto;
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 20px;
            align-items: center;
        }

        .controls label {
            font-size: 16px;
            color: #333;
        }

        .controls input, .controls button, .controls select {
            padding: 12px;
            border: none;
            border-radius: 8px;
            font-size: 14px;
            cursor: pointer;
            background-color: #fff;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
        }

        .controls button {
            background-color: #fae1c4; /* Light pastel orange */
            color: white;
            transition: background-color 0.3s;
        }

        .controls button:hover {
            background-color: #f7e7d3; /* Light pastel cream */
        }

        .controls input:focus, .controls select:focus, .controls button:focus {
            box-shadow: 0 0 8px rgba(255, 120, 0, 0.7);
            outline: none;
        }

        .image-preview {
            width: 100%;
            height: 100%;
            object-fit: cover;
            position: absolute;
            border-radius: 15px;
            cursor: grab;
            display: none;
        }

        /* AI tools design - placed below image preview */
        .ai-tools {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            margin-top: 20px;
            background-color: #a1d6b0; /* Soft pastel green */
            padding: 20px;
            border-radius: 10px;
        }

        .ai-tools input {
            width: 80%;
            padding: 10px;
            margin: 10px 0;
            border-radius: 8px;
            border: none;
            font-size: 16px;
            outline: none;
        }

        .ai-tools button {
            background-color: #fae1c4; /* Soft pastel orange */
            padding: 12px 20px;
            border-radius: 8px;
            color: white;
            font-size: 14px;
            cursor: pointer;
            margin-top: 10px;
        }

        .ai-tools button:hover {
            background-color: #f7e7d3; /* Soft pastel cream */
        }
    </style>
</head>
<body>

    <h1>Customize Your Card</h1>

    <div class="card" id="card">
        <div class="card-front" id="card-front">
            <p contenteditable="true" class="card-text" id="card-text" onmousedown="startDrag(event, 'text')">Edit your text here...</p>
        </div>
        <div class="card-back">
            <img src="" alt="" class="image-preview" id="image-preview" onmousedown="startDrag(event, 'image')">
        </div>
    </div>

    <div class="controls">
        <div>
            <label for="color">Card Color:</label>
            <input type="color" id="color" name="color">
        </div>
        <div>
            <label for="text-color">Text Color:</label>
            <input type="color" id="text-color" name="text-color">
        </div>
        <div>
            <label for="upload-image">Upload Image (Front):</label>
            <input type="file" id="upload-image" accept="image/*">
        </div>
        <div>
            <label for="resize-range">Resize Image:</label>
            <input type="range" id="resize-range" min="50" max="300" value="100">
        </div>
        <div>
            <label for="font-size">Font Size:</label>
            <select id="font-size">
                <option value="14px">Small</option>
                <option value="18px" selected>Normal</option>
                <option value="24px">Large</option>
                <option value="32px">Extra Large</option>
            </select>
        </div>
        <div class="button-group">
            <button onclick="resetChanges()">Reset Changes</button>
            <button onclick="flipCard()">Flip Card</button>
        </div>
    </div>

    <div class="ai-tools">
        <input type="text" id="ai-prompt" placeholder="Type here for AI generation...">
        <button onclick="generateContent()">Generate Text</button>
        <button onclick="generateImage()">Generate Image</button>
    </div>

    <script>
        const card = document.getElementById("card");
        const cardText = document.getElementById("card-text");
        const imagePreview = document.getElementById("image-preview");
        const cardFront = document.getElementById("card-front");
        const cardColor = document.getElementById("color");
        const textColor = document.getElementById("text-color");
        const uploadImage = document.getElementById("upload-image");
        const resizeRange = document.getElementById("resize-range");
        const fontSizeSelector = document.getElementById("font-size");
        const aiPrompt = document.getElementById("ai-prompt");

        let dragElement = null;
        let offsetX = 0;
        let offsetY = 0;

        cardColor.addEventListener("input", (event) => {
            cardFront.style.backgroundColor = event.target.value;
        });

        textColor.addEventListener("input", (event) => {
            cardText.style.color = event.target.value;
        });

        uploadImage.addEventListener("change", (event) => {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    imagePreview.src = e.target.result;
                    imagePreview.style.display = "block";
                };
                reader.readAsDataURL(file);
            }
        });

        resizeRange.addEventListener("input", (event) => {
            const size = event.target.value;
            imagePreview.style.width = `${size}%`;
            imagePreview.style.height = `${size}%`;
        });

        fontSizeSelector.addEventListener("change", (event) => {
            cardText.style.fontSize = event.target.value;
        });

        function startDrag(event, type) {
            dragElement = type === 'text' ? cardText : imagePreview;

            offsetX = event.clientX - dragElement.offsetLeft;
            offsetY = event.clientY - dragElement.offsetTop;

            document.addEventListener('mousemove', moveElement);
            document.addEventListener('mouseup', stopDrag);
        }

        function moveElement(event) {
            if (dragElement) {
                const x = event.clientX - offsetX;
                const y = event.clientY - offsetY;

                const maxX = card.offsetWidth - dragElement.offsetWidth;
                const maxY = card.offsetHeight - dragElement.offsetHeight;

                dragElement.style.left = `${Math.max(0, Math.min(x, maxX))}px`;
                dragElement.style.top = `${Math.max(0, Math.min(y, maxY))}px`;
            }
        }

        function stopDrag() {
            document.removeEventListener('mousemove', moveElement);
            document.removeEventListener('mouseup', stopDrag);
        }

        function flipCard() {
            const currentRotation = card.style.transform.match(/rotateY\((\d+)deg\)/);
            const newRotation = currentRotation ? (parseInt(currentRotation[1]) + 180) % 360 : 180;
            card.style.transform = `rotateY(${newRotation}deg)`;
        }

        function resetChanges() {
            card.style.backgroundColor = "#fdf6e3";
            cardText.style.color = "#4f8a3e";
            cardText.textContent = "Edit your text here...";
            cardText.style.left = "50%";
            cardText.style.top = "50%";
            cardText.style.transform = "translate(-50%, -50%)";
            imagePreview.style.display = "none";
            imagePreview.style.left = "0";
            imagePreview.style.top = "0";
            uploadImage.value = "";
            resizeRange.value = 100;
            fontSizeSelector.value = "18px";
        }

        async function generateContent() {
            const prompt = aiPrompt.value;
            if (!prompt) return alert("Please provide a prompt for text generation.");

            const generatedText = `AI-generated content for: "${prompt}"`;
            cardText.textContent = generatedText;
        }

        function generateImage() {
            alert("Image generation feature is a placeholder!");
        }
    </script>

</body>
</html>
