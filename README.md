<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Gimble Gamble</title>
  <style>
    body {
      font-family: "Segoe UI", sans-serif;
      background: linear-gradient(135deg, #667eea, #764ba2);
      color: #fff;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      margin: 0;
      padding: 15px;
      box-sizing: border-box;
      overflow-y: auto;
    }
    .quiz-box {
      background: rgba(0, 0, 0, 0.75);
      padding: 2rem;
      border-radius: 20px;
      box-shadow: 0 8px 24px rgba(0,0,0,0.5);
      width: 90%;
      max-width: 450px;
      text-align: center;
      position: relative;
    }
    h2 {
      margin-bottom: 1rem;
    }
    select, button {
      width: 100%;
      padding: 0.75rem;
      margin: 0.5rem 0;
      border: none;
      border-radius: 12px;
      font-size: 1rem;
      cursor: pointer;
      transition: transform 0.2s ease, background 0.3s ease;
    }
    select {
      background: #f1c40f;
      color: #333;
    }
    button {
      background: linear-gradient(to right, #00c6ff, #0072ff);
      color: white;
      font-weight: bold;
      border: 2px solid transparent;
    }
    button:hover, select:hover {
      transform: scale(1.03);
    }
    .question {
      font-size: 1.3rem;
      margin-bottom: 1rem;
      min-height: 60px;
      display: flex;
      align-items: center;
      justify-content: center;
      word-wrap: break-word;
      overflow-wrap: break-word;
    }
    .options {
      display: flex;
      flex-direction: column;
      margin-bottom: 1rem;
    }
    .option {
      background: #2c3e50;
      color: white;
      padding: 0.75rem;
      margin: 0.3rem 0;
      border-radius: 10px;
      cursor: pointer;
      transition: background 0.3s ease, transform 0.1s ease;
      border: 1px solid #2c3e50;
      word-wrap: break-word;
      overflow-wrap: break-word;
      text-align: center;
    }
    .option:hover {
      background: #34495e;
      border-color: #f1c40f;
    }
    .option.correct {
      background: #27ae60;
      border-color: #27ae60;
    }
    .option.incorrect {
      background: #e74c3c;
      border-color: #e74c3c;
    }
    .score, .result {
      margin-top: 1rem;
      font-weight: bold;
    }
    .restart {
      background: linear-gradient(to right, #ff416c, #ff4b2b);
    }
    #timer {
      font-size: 1.8rem;
      font-weight: bold;
      margin-bottom: 1.5rem;
      color: #f1c40f;
      text-shadow: 0 0 8px rgba(241, 196, 15, 0.6);
      display: none;
    }
    .result {
      font-size: 1.8rem;
      color: #2ecc71;
    }
    .restart {
        margin-top: 1.5rem;
    }
    .message-box {
        background-color: #38b2ac;
        color: #e2e8f0;
        padding: 15px;
        border-radius: 8px;
        margin-top: 20px;
        display: none;
        font-size: 1rem;
        font-weight: 500;
        word-wrap: break-word;
    }
    .message-box.error {
        background-color: #e53e3e;
    }
    .message-box.success {
        background-color: #48bb78;
    }
    .sound-control {
      position: absolute;
      top: 15px;
      right: 15px;
      background: rgba(255, 255, 255, 0.2);
      border-radius: 50%;
      width: 40px;
      height: 40px;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      z-index: 10;
      border: 1px solid rgba(255, 255, 255, 0.3);
    }
    .sound-control i {
      font-size: 20px;
    }
    .sound-control.muted {
      background: rgba(231, 76, 60, 0.5);
    }
  </style>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
</head>
<body>
  <div class="quiz-box" id="quiz">
    <div class="sound-control" id="soundControl" title="Toggle sound">
      <i class="fas fa-volume-up"></i>
    </div>
    
    <div id="setup">
      <h2>🎯 Choose Topic & Difficulty</h2>
      <select id="topicSelect">
        <option value="general">🌍 General</option>
        <option value="science">🔬 Science</option>
        <option value="math">➗ Math</option>
        <option value="coding" disabled>💻 Coding (Coming Soon)</option>
      </select>
      <select id="difficultySelect">
        <option value="easy">😀 Easy</option>
        <option value="medium">😎 Medium</option>
        <option value="hard">🤯 Hard</option>
      </select>
      <button onclick="startQuiz()">🚀 Start Quiz</button>
    </div>

    <div id="timer" style="display:none;"></div>
    <div class="question" id="question" style="display:none"></div>
    <div class="options" id="options" style="display:none"></div>
    <div class="score" id="score" style="display:none"></div>
    <div id="messageBox" class="message-box"></div>
  </div>

  <script>
    // Sound objects - fixed approach
    const correctSound = new Audio("https://www.soundjay.com/buttons/sounds/button-3.mp3");
    const wrongSound = new Audio("https://www.soundjay.com/buttons/sounds/button-10.mp3");
    const timerEndSound = new Audio("https://www.soundjay.com/buttons/sounds/button-09.mp3");
    
    // Preload sounds
    correctSound.preload = 'auto';
    wrongSound.preload = 'auto';
    timerEndSound.preload = 'auto';
    
    // Sound state management
    let soundEnabled = true;

    const allQuestions = {
      general: {
        easy: [
          { question: "What is the capital of France?", options: ["Paris", "Berlin", "Madrid", "London"], answer: "Paris" },
          { question: "Which color is a banana?", options: ["Red", "Yellow", "Blue", "Green"], answer: "Yellow" },
          { question: "Which ocean is the largest?", options: ["Atlantic", "Indian", "Pacific", "Arctic"], answer: "Pacific" },
          { question: "What day comes after Friday?", options: ["Monday", "Saturday", "Thursday", "Sunday"], answer: "Saturday" },
          { question: "How many legs does a spider have?", options: ["6", "8", "4", "10"], answer: "8" },
          { question: "What is the opposite of 'up'?", options: ["Down", "Left", "Right", "Side"], answer: "Down" },
          { question: "Which animal says 'Moo'?", options: ["Dog", "Cat", "Cow", "Chicken"], answer: "Cow" },
          { question: "How many fingers do humans have on one hand?", options: ["3", "4", "5", "6"], answer: "5" },
          { question: "Which shape has no sides?", options: ["Square", "Triangle", "Circle", "Rectangle"], answer: "Circle" },
          { question: "What do you use to write on a blackboard?", options: ["Pen", "Pencil", "Chalk", "Marker"], answer: "Chalk" }
        ],
        medium: [
          { question: "What is the longest river in the world?", options: ["Amazon", "Nile", "Mississippi", "Yangtze"], answer: "Nile" },
          { question: "Which continent is the largest by land area?", options: ["Africa", "Europe", "Asia", "North America"], answer: "Asia" },
          { question: "What is the most populous country in the world?", options: ["India", "USA", "China", "Indonesia"], answer: "China" },
          { question: "Which planet is known as the 'Morning Star' or 'Evening Star'?", options: ["Mars", "Venus", "Jupiter", "Saturn"], answer: "Venus" },
          { question: "What is the national animal of Australia?", options: ["Kangaroo", "Koala", "Emu", "Dingo"], answer: "Kangaroo" },
          { question: "How many bones are in the adult human body?", options: ["206", "300", "150", "250"], answer: "206" },
          { question: "Which country is famous for the Eiffel Tower?", options: ["Italy", "Germany", "France", "Spain"], answer: "France" },
          { question: "What is the largest land animal?", options: ["Giraffe", "Elephant", "Rhinoceros", "Hippopotamus"], answer: "Elephant" },
          { question: "Which musical instrument has black and white keys?", options: ["Guitar", "Drums", "Piano", "Violin"], answer: "Piano" },
          { question: "What is the highest mountain in Africa?", options: ["Mount Everest", "Mount Kilimanjaro", "Mount Elbrus", "Mount Fuji"], answer: "Mount Kilimanjaro" }
        ],
        hard: [
          { question: "What is the capital of Mongolia?", options: ["Ulaanbaatar", "Astana", "Tashkent", "Bishkek"], answer: "Ulaanbaatar" },
          { question: "Who painted the Mona Lisa?", options: ["Vincent van Gogh", "Pablo Picasso", "Leonardo da Vinci", "Claude Monet"], answer: "Leonardo da Vinci" },
          { question: "Which chemical element has the atomic number 79?", options: ["Silver", "Gold", "Mercury", "Lead"], answer: "Gold" },
          { question: "In which year did the Titanic sink?", options: ["1905", "1912", "1918", "1923"], answer: "1912" },
          { question: "Who wrote 'Romeo and Juliet'?", options: ["Charles Dickens", "William Shakespeare", "Jane Austen", "Mark Twain"], answer: "William Shakespeare" },
          { question: "What is the smallest country in the world?", options: ["Monaco", "Vatican City", "San Marino", "Liechtenstein"], answer: "Vatican City" },
          { question: "Which of the following is not a primary color?", options: ["Red", "Yellow", "Green", "Blue"], answer: "Green" },
          { question: "What is the deepest ocean trench?", options: ["Kermadec Trench", "Java Trench", "Mariana Trench", "Puerto Rico Trench"], answer: "Mariana Trench" },
          { question: "Who invented the light bulb?", options: ["Nikola Tesla", "Alexander Graham Bell", "Thomas Edison", "Isaac Newton"], answer: "Thomas Edison" },
          { question: "What is the currency of Japan?", options: ["Yuan", "Won", "Yen", "Rupee"], answer: "Yen" }
        ]
      },
      science: {
        easy: [
          { question: "What planet is known as the Red Planet?", options: ["Earth", "Mars", "Jupiter", "Saturn"], answer: "Mars" },
          { question: "Water freezes at what temperature (°C)?", options: ["0", "100", "32", "-10"], answer: "0" },
          { question: "Which gas do plants breathe in?", options: ["Oxygen", "Nitrogen", "Carbon Dioxide", "Helium"], answer: "Carbon Dioxide" },
          { question: "How many planets are in our solar system?", options: ["7", "8", "9", "10"], answer: "8" },
          { question: "What part of the body pumps blood?", options: ["Brain", "Heart", "Lungs", "Liver"], answer: "Heart" },
          { question: "What keeps us on the ground?", options: ["Wind", "Gravity", "Magnetism", "Air"], answer: "Gravity" },
          { question: "What do bees make?", options: ["Milk", "Honey", "Bread", "Butter"], answer: "Honey" },
          { question: "Which animal lays eggs?", options: ["Dog", "Cow", "Chicken", "Cat"], answer: "Chicken" },
          { question: "What is the opposite of 'hot'?", options: ["Warm", "Cold", "Boiling", "Mild"], answer: "Cold" },
          { question: "What force makes an apple fall from a tree?", options: ["Lift", "Push", "Friction", "Gravity"], answer: "Gravity" }
        ],
        medium: [
          { question: "What is the chemical symbol for gold?", options: ["Au", "Ag", "Fe", "Cu"], answer: "Au" },
          { question: "What is the powerhouse of the cell?", options: ["Nucleus", "Mitochondria", "Ribosome", "Endoplasmic Reticulum"], answer: "Mitochondria" },
          { question: "Which force pulls objects towards the center of the Earth?", options: ["Friction", "Tension", "Gravity", "Magnetism"], answer: "Gravity" },
          { question: "What is the most abundant gas in Earth's atmosphere?", options: ["Oxygen", "Carbon Dioxide", "Nitrogen", "Argon"], answer: "Nitrogen" },
          { question: "What is the process by which plants make their own food?", options: ["Respiration", "Transpiration", "Photosynthesis", "Digestion"], answer: "Photosynthesis" },
          { question: "What is the speed of light in a vacuum (approximately)?", options: ["300,000 km/s", "150,000 km/s", "500,000 km/s", "1,000,000 km/s"], answer: "300,000 km/s" },
          { question: "What is the largest organ in the human body?", options: ["Heart", "Brain", "Skin", "Liver"], answer: "Skin" },
          { question: "Which of these is a vertebrate?", options: ["Spider", "Earthworm", "Fish", "Jellyfish"], answer: "Fish" },
          { question: "What is the study of weather called?", options: ["Geology", "Biology", "Meteorology", "Astronomy"], answer: "Meteorology" },
          { question: "What is the chemical symbol for water?", options: ["O2", "CO2", "H2O", "NaCl"], answer: "H2O" }
        ],
        hard: [
          { question: "Which subatomic particle has a negative charge?", options: ["Proton", "Neutron", "Electron", "Photon"], answer: "Electron" },
          { question: "What is the study of fungi called?", options: ["Botany", "Zoology", "Mycology", "Geology"], answer: "Mycology" },
          { question: "What is the hardest natural substance on Earth?", options: ["Gold", "Iron", "Diamond", "Quartz"], answer: "Diamond" },
          { question: "Which gas is responsible for the green color of plants?", options: ["Oxygen", "Carbon Dioxide", "Chlorophyll", "Nitrogen"], answer: "Chlorophyll" },
          { question: "What is the unit of electrical resistance?", options: ["Volt", "Ampere", "Ohm", "Watt"], answer: "Ohm" },
          { question: "Which scientist developed the theory of relativity?", options: ["Isaac Newton", "Albert Einstein", "Galileo Galilei", "Marie Curie"], answer: "Albert Einstein" },
          { question: "What is the process of a liquid turning into a gas called?", options: ["Melting", "Freezing", "Condensation", "Evaporation"], answer: "Evaporation" },
          { question: "What is the largest bone in the human body?", options: ["Tibia", "Femur", "Humerus", "Patella"], answer: "Femur" },
          { question: "What is the chemical formula for table salt?", options: ["KCl", "H2O", "NaCl", "CO2"], answer: "NaCl" },
          { question: "Which type of rock is formed from cooled magma or lava?", options: ["Sedimentary", "Metamorphic", "Igneous", "Fossil"], answer: "Igneous" }
        ]
      },
      math: {
        easy: [
          { question: "What is 2 + 2?", options: ["3", "4", "5", "6"], answer: "4" },
          { question: "What shape has 4 equal sides?", options: ["Triangle", "Square", "Circle", "Rectangle"], answer: "Square" },
          { question: "What is 10 - 4?", options: ["5", "6", "7", "4"], answer: "6" },
          { question: "How many sides does a triangle have?", options: ["2", "3", "4", "5"], answer: "3" },
          { question: "What is 5 x 3?", options: ["8", "15", "10", "12"], answer: "15" },
          { question: "What is 8 divided by 2?", options: ["2", "4", "6", "8"], answer: "4" },
          { question: "What is the next number in the sequence: 1, 2, 3, 4, __?", options: ["5", "6", "7", "8"], answer: "5" },
          { question: "If you have 5 apples and eat 2, how many are left?", options: ["2", "3", "4", "5"], answer: "3" },
          { question: "What is 10 + 7?", options: ["15", "16", "17", "18"], answer: "17" },
          { question: "Which number comes before 10?", options: ["8", "9", "11", "12"], answer: "9" }
        ],
        medium: [
          { question: "What is the area of a rectangle with length 5 and width 3?", options: ["8", "15", "10", "25"], answer: "15" },
          { question: "If x + 7 = 15, what is x?", options: ["6", "7", "8", "9"], answer: "8" },
          { question: "What is 12 multiplied by 5?", options: ["50", "60", "70", "125"], answer: "60" },
          { question: "What is the perimeter of a square with side length 4?", options: ["4", "8", "12", "16"], answer: "16" },
          { question: "If a shirt costs $20 and is on sale for 25% off, how much does it cost?", options: ["$10", "$15", "$5", "$20"], answer: "$15" },
          { question: "What is 2.5 + 3.7?", options: ["5.2", "6.2", "5.7", "6.7"], answer: "6.2" },
          { question: "What is 1/2 as a decimal?", options: ["0.2", "0.5", "1.0", "2.0"], answer: "0.5" },
          { question: "What is the next number in the sequence: 2, 4, 6, 8, __?", options: ["9", "10", "11", "12"], answer: "10" },
          { question: "If a car travels 60 miles in 1 hour, how far will it travel in 3 hours?", options: ["60 miles", "120 miles", "180 miles", "240 miles"], answer: "180 miles" },
          { question: "What is the value of pi (rounded to two decimal places)?", options: ["3.04", "3.14", "3.41", "3.10"], answer: "3.14" }
        ],
        hard: [
          { question: "What is the square root of 144?", options: ["10", "12", "14", "16"], answer: "12" },
          { question: "What is 3 squared?", options: ["6", "9", "3", "12"], answer: "9" },
          { question: "Solve for x: 2x + 5 = 15", options: ["x = 5", "x = 7", "x = 10", "x = 20"], answer: "x = 5" },
          { question: "What is the greatest common divisor of 12 and 18?", options: ["2", "3", "6", "9"], answer: "6" },
          { question: "What is the circumference of a circle with a radius of 7 units (use π ≈ 22/7)?", options: ["22 units", "44 units", "14 units", "7 units"], answer: "44 units" },
          { question: "If a fair coin is tossed twice, what is the probability of getting two heads?", options: ["1/4", "1/2", "3/4", "1"], answer: "1/4" },
          { question: "What is the next number in the Fibonacci sequence: 0, 1, 1, 2, 3, 5, __?", options: ["7", "8", "9", "10"], answer: "8" },
          { question: "Simplify the expression: (a^2)^3", options: ["a^5", "a^6", "3a^2", "a^9"], answer: "a^6" },
          { question: "What is the derivative of x^2?", options: ["x", "2x", "x^3", "1"], answer: "2x" },
          { question: "If a car travels at 50 mph for 2 hours, how far does it travel?", options: ["50 miles", "75 miles", "100 miles", "150 miles"], answer: "100 miles" }
        ]
      },
      coding: {
        easy: [],
        medium: [],
        hard: []
      }
    };

    let currentQuestionIndex = 0;
    let score = 0;
    let selectedQuestions = [];
    let timeLeft = 0;
    let timerInterval;
    const timePerQuestion = 15; // 15 seconds per question

    // DOM Elements
    const setupDiv = document.getElementById("setup");
    const questionEl = document.getElementById("question");
    const optionsEl = document.getElementById("options");
    const scoreEl = document.getElementById("score");
    const timerEl = document.getElementById("timer");
    const messageBox = document.getElementById("messageBox");
    const soundControl = document.getElementById("soundControl");

    // Function to play a sound
    function playSound(sound) {
      if (!soundEnabled) return;
      
      try {
        // Stop and reset before playing
        sound.pause();
        sound.currentTime = 0;
        sound.play().catch(error => {
          console.log("Sound playback error:", error);
          showMessage("Sound playback blocked. Please interact with the page first.", "error");
        });
      } catch (error) {
        console.log("Sound error:", error);
      }
    }

    // Toggle sound
    soundControl.addEventListener("click", () => {
      soundEnabled = !soundEnabled;
      if (soundEnabled) {
        soundControl.classList.remove("muted");
        soundControl.innerHTML = '<i class="fas fa-volume-up"></i>';
        showMessage("Sound enabled", "success");
      } else {
        soundControl.classList.add("muted");
        soundControl.innerHTML = '<i class="fas fa-volume-mute"></i>';
        showMessage("Sound disabled", "error");
      }
    });

    // Function to display messages
    function showMessage(message, type = 'info') {
      messageBox.textContent = message;
      messageBox.className = 'message-box';
      if (type === 'error') {
        messageBox.classList.add('error');
      } else if (type === 'success') {
        messageBox.classList.add('success');
      }
      messageBox.style.display = 'block';
      setTimeout(() => {
        messageBox.style.display = 'none';
      }, 3000);
    }

    function startQuiz() {
      const topic = document.getElementById("topicSelect").value;
      const difficulty = document.getElementById("difficultySelect").value;
      selectedQuestions = allQuestions[topic][difficulty];

      if (!selectedQuestions || selectedQuestions.length === 0) {
        showMessage("No questions available for this topic and difficulty. Please choose another combination.", "error");
        setupDiv.style.display = "block";
        questionEl.style.display = "none";
        optionsEl.style.display = "none";
        scoreEl.style.display = "none";
        timerEl.style.display = "none";
        return;
      }

      setupDiv.style.display = "none";
      timerEl.style.display = "block";
      questionEl.style.display = "flex";
      optionsEl.style.display = "flex";
      scoreEl.style.display = "block";
      messageBox.style.display = "none";

      currentQuestionIndex = 0;
      score = 0;
      showQuestion();
    }

    function showQuestion() {
      clearInterval(timerInterval);

      if (currentQuestionIndex >= selectedQuestions.length) {
        showResult();
        return;
      }

      const current = selectedQuestions[currentQuestionIndex];
      questionEl.textContent = current.question;
      optionsEl.innerHTML = "";

      current.options.forEach(option => {
        const btn = document.createElement("button");
        btn.classList.add("option");
        btn.textContent = option;
        btn.onclick = () => selectOption(option);
        optionsEl.appendChild(btn);
      });

      scoreEl.textContent = `Question ${currentQuestionIndex + 1} of ${selectedQuestions.length}`;

      // Start timer
      timeLeft = timePerQuestion;
      timerEl.textContent = `Time: ${timeLeft}s`;
      timerInterval = setInterval(() => {
        timeLeft--;
        timerEl.textContent = `Time: ${timeLeft}s`;

        if (timeLeft <= 5) {
          timerEl.style.color = '#ff416c';
        } else {
          timerEl.style.color = '#f1c40f';
        }

        if (timeLeft <= 0) {
          clearInterval(timerInterval);
          playSound(timerEndSound);
          showMessage("Time's up for this question!", "error");
          selectOption(null);
        }
      }, 1000);
    }

    function selectOption(selected) {
      clearInterval(timerInterval);

      const correctAnswer = selectedQuestions[currentQuestionIndex].answer;
      Array.from(optionsEl.children).forEach(button => {
        button.disabled = true;
        button.style.pointerEvents = 'none';
        if (button.textContent === correctAnswer) {
          button.classList.add('correct');
        } else if (selected !== null && button.textContent === selected) {
          button.classList.add('incorrect');
        }
      });

      if (selected === correctAnswer) {
        score++;
        playSound(correctSound);
      } else if (selected !== null) {
        playSound(wrongSound);
      } else {
        playSound(wrongSound);
      }

      currentQuestionIndex++;
      setTimeout(showQuestion, 1500);
    }

    function showResult() {
      clearInterval(timerInterval);

      const quizBox = document.getElementById("quiz");
      quizBox.innerHTML = `
        <h2 class="result">🎉 Quiz Over!</h2>
        <div class="score">You scored ${score} out of ${selectedQuestions.length}!</div>
        <button class="restart" onclick="location.reload()">🔁 Play Again!</button>
      `;
      timerEl.style.display = "none";
      messageBox.style.display = "none";
    }
    
    // Play a silent sound to unlock audio on user interaction
    document.addEventListener('click', function unlockAudio() {
      try {
        // Play and immediately pause a silent sound
        const silentSound = new Audio("data:audio/mp3;base64,SUQzBAAAAAABEVRYWFgAAAAtAAADY29tbWVudABCaWdTb3VuZEJhbmsuY29tIC8gTGFTb25vdGhlcXVlLm9yZwBURU5DAAAAHQAAA1N3aXRjaCBQbHVzIMKpIE5DSCBTb2Z0d2FyZQBUSVQyAAAABgAAAzIyMzUAVFNTRQAAAA8AAANMYXZmNTcuODMuMTAwAAAAAAAAAAAAAAD/80DEAAAAA0gAAAAATEFNRTMuMTAwVVVVVVVVVVVVVUxBTUUzLjEwMFVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVf/zQsRbAAADSAAAAABVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVf/zQMSkAAADSAAAAABVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVV");
        silentSound.volume = 0;
        silentSound.play().then(() => {
          silentSound.pause();
        }).catch(error => {
          console.log("Audio unlock failed:", error);
        });
      } catch (error) {
        console.log("Audio unlock error:", error);
      }
      // Remove the event listener after the first interaction
      document.removeEventListener('click', unlockAudio);
    }, { once: true });
  </script>
</body>
</html>
