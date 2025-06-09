<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>StepSolver AI</title>
  <script src="https://cdn.jsdelivr.net/npm/tesseract.js@2.1.5/dist/tesseract.min.js"></script>
  <style>
    body {
      margin: 0;
      padding: 2rem;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #d3cce3 0%, #e9e4f0 100%);
      display: flex;
      justify-content: center;
      align-items: flex-start;
      min-height: 100vh;
    }

    .app-container {
      background: #fff;
      border-radius: 20px;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
      padding: 2rem;
      max-width: 700px;
      width: 100%;
      animation: fadeIn 0.8s ease;
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(-10px); }
      to { opacity: 1; transform: translateY(0); }
    }

    h1 {
      text-align: center;
      font-size: 2.5rem;
      margin-bottom: 1.5rem;
      background: linear-gradient(to right, #667eea, #764ba2);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
    }

    input[type="text"] {
      width: 100%;
      padding: 1rem;
      font-size: 1.2rem;
      border: 2px solid #ddd;
      border-radius: 12px;
      margin-bottom: 1rem;
      transition: border 0.3s, box-shadow 0.3s;
    }

    input[type="text"]:focus {
      outline: none;
      border-color: #764ba2;
      box-shadow: 0 0 8px rgba(118, 75, 162, 0.2);
    }

    .buttons {
      display: flex;
      flex-wrap: wrap;
      gap: 0.75rem;
      margin-bottom: 1rem;
      justify-content: center;
    }

    button {
      flex: 1 1 45%;
      padding: 0.8rem;
      background: linear-gradient(to right, #667eea, #764ba2);
      color: white;
      border: none;
      border-radius: 10px;
      font-size: 1rem;
      cursor: pointer;
      transition: transform 0.2s, box-shadow 0.3s;
    }

    button:hover {
      transform: translateY(-2px);
      box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
    }

    .file-upload-wrapper {
      position: relative;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      border: 2px dashed #764ba2;
      border-radius: 12px;
      padding: 1.5rem;
      background-color: #f8f6ff;
      color: #764ba2;
      transition: background 0.3s ease-in-out, transform 0.2s ease-in-out;
      cursor: pointer;
    }

    .file-upload-wrapper:hover {
      background-color: #ece6ff;
      transform: scale(1.02);
    }

    .file-upload-wrapper span {
      font-size: 1.1rem;
      margin-bottom: 0.5rem;
    }

    #imageInput {
      opacity: 0;
      position: absolute;
      height: 100%;
      width: 100%;
      cursor: pointer;
    }

    #result {
      margin-top: 1.5rem;
      padding: 1rem;
      background: #f4f4f4;
      border-radius: 10px;
      font-size: 1.2rem;
      min-height: 3rem;
      transition: background 0.5s ease;
    }
  </style>
</head>
<body>
  <div class="app-container">
    <h1>StepSolver AI</h1>
    <input type="text" id="expression" placeholder="Enter numbers separated by +, -, *, /" />
    <div class="buttons">
      <button onclick="calculate('+')">Add</button>
      <button onclick="calculate('-')">Subtract</button>
      <button onclick="calculate('*')">Multiply</button>
      <button onclick="calculate('/')">Divide</button>
    </div>
    <div class="buttons">
      <button onclick="startVoiceInput()">🎤 Voice Input</button>
    </div>
    <label class="file-upload-wrapper">
      <span>📁 Choose Image File</span>
      <input type="file" id="imageInput" accept="image/*" onchange="extractFromImage()">
    </label>
    <div id="result"></div>
  </div>

  <script>
    function calculate(operator) {
      const input = document.getElementById('expression').value;
      const resultEl = document.getElementById('result');

      if (!input.trim()) {
        resultEl.textContent = "Please enter a list of numbers.";
        return;
      }

      let numbers = input.split(/[^0-9.\-]+/).map(num => parseFloat(num)).filter(n => !isNaN(n));
      if (numbers.length < 2) {
        resultEl.textContent = "Enter at least two numbers.";
        return;
      }

      let result = numbers[0];
      let steps = `${numbers[0]}`;

      for (let i = 1; i < numbers.length; i++) {
        if (operator === '/' && numbers[i] === 0) {
          resultEl.textContent = "Cannot divide by zero.";
          return;
        }
        result = eval(`${result} ${operator} ${numbers[i]}`);
        steps += ` ${operator} ${numbers[i]}`;
      }

      steps += ` = ${result}`;
      resultEl.textContent = steps;
    }

    function startVoiceInput() {
      const recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
      recognition.lang = 'en-US';
      recognition.interimResults = false;
      recognition.maxAlternatives = 1;

      recognition.start();

      recognition.onresult = function(event) {
        const transcript = event.results[0][0].transcript;
        document.getElementById('expression').value = transcript;
      };

      recognition.onerror = function(event) {
        alert('Voice recognition error: ' + event.error);
      };
    }

    function extractFromImage() {
      const input = document.getElementById('imageInput');
      const file = input.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = function () {
        Tesseract.recognize(
          reader.result,
          'eng',
          { logger: m => console.log(m) }
        ).then(({ data: { text } }) => {
          document.getElementById('expression').value = text.replace(/\n/g, ' ').trim();
        }).catch(err => {
          alert('OCR failed: ' + err);
        });
      };
      reader.readAsDataURL(file);
    }
  </script>
</body>
</html>
