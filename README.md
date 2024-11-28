# Systeminteg
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Crypto & Stock Prices with Quiz and Joke</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script> <!-- Chart.js -->
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script> <!-- Axios -->
<style>
body {
font-family: Arial, sans-serif;
background-color: #f4f4f4;
padding: 20px;
}
.container {
max-width: 900px;
margin: 0 auto;
}
.section {
margin-bottom: 40px;
padding: 20px;
background-color: #fff;
border-radius: 8px;
box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}
h2, h3 {
text-align: center;
color: #333;
}
canvas {
width: 100% !important;
height: 400px !important;
}
.timestamp {
text-align: center;
font-size: 14px;
color: #777;
margin-bottom: 10px;
}
.value-labels {
display: flex;
justify-content: space-around;
margin-top: 10px;
font-size: 14px;
color: #555;
}
.value-label {
text-align: center;
width: 18%;
}
#joke-container, #quiz-container {
background-color: #f9f9f9;
border-radius: 10px;
padding: 20px;
}
.question {
font-size: 20px;
margin: 20px 0;
}
.options {
list-style-type: none;
padding: 0;
}
.options li {
margin: 10px 0;
}
button {
display: block;
margin: 20px auto;
padding: 10px 20px;
background-color: #4CAF50;
color: white;
border: none;
border-radius: 5px;
cursor: pointer;
}
button:hover {
background-color: #45a049;
}
.result {
text-align: center;
font-size: 18px;
margin-top: 20px;
}
.joke-text {
font-size: 18px;
text-align: center;
}
</style>
</head>
<body>

<div class="container">
<!-- Cryptocurrency Prices Section -->
<div class="section">
<h2>Cryptocurrency Prices (PHP)</h2>
<div id="cryptoTimestamp" class="timestamp"></div>
<div class="chart-container">
<canvas id="cryptoPricesChart"></canvas>
<div id="cryptoValues" class="value-labels"></div>
</div>
</div>

<!-- Stock Prices Section -->
<div class="section">
<h2>Stock Prices (PHP)</h2>
<div id="stockTimestamp" class="timestamp"></div>
<div class="chart-container">
<canvas id="stockPricesChart"></canvas>
<div id="stockValues" class="value-labels"></div>
</div>
</div>

<!-- Joke Section -->
<div id="joke-container" class="section">
<h3>Programming Joke</h3>
<p class="joke-text" id="joke">Loading a joke...</p>
<button onclick="getJoke()">Get a Joke</button>
</div>

<!-- Trivia Quiz Section -->
<div id="quiz-container" class="section">
<h3>Trivia Quiz</h3>
<div id="quiz-content">
<!-- Questions will be injected here -->
</div>
<button onclick="nextQuestion()">Next Question</button>
<div class="result" id="result"></div>
</div>
</div>

<script>
// Fetch cryptocurrency prices from CoinGecko
const fetchCryptoPrices = async () => {
const url = 'https://api.coingecko.com/api/v3/simple/price';
try {
const response = await axios.get(url, {
params: {
ids: 'bitcoin,ethereum,cardano,solana,ripple',
vs_currencies: 'php'
}
});
return { data: response.data, timestamp: new Date() };
} catch (error) {
console.error("Error fetching cryptocurrency prices:", error);
return null;
}
};

// Fetch stock prices from Alpha Vantage
const fetchStockPrices = async () => {
const url = 'https://www.alphavantage.co/query';
const apiKey = 'YOUR_ALPHA_VANTAGE_API_KEY'; // Replace with your API key
const symbols = ['AAPL', 'MSFT', 'GOOGL', 'AMZN', 'TSLA'];
try {
const promises = symbols.map(symbol =>
axios.get(url, {
params: {
function: 'TIME_SERIES_INTRADAY',
symbol: symbol,
interval: '1min',
apikey: apiKey
}
})
);
const responses = await Promise.all(promises);
const pricesInUSD = responses.map(response => {
const timeSeries = response.data['Time Series (1min)'];
const latestKey = Object.keys(timeSeries)[0];
return parseFloat(timeSeries[latestKey]['1. open']);
});
const conversionRate = 56;
return { data: pricesInUSD.map(price => price * conversionRate), timestamp: new Date() };
} catch (error) {
console.error("Error fetching stock prices:", error);
return null;
}
};

// Render the charts
const renderCharts = async () => {
const cryptoResponse = await fetchCryptoPrices();
const stockResponse = await fetchStockPrices();

if (!cryptoResponse || !stockResponse) {
alert("Error fetching data. Check the console for details.");
return;
}

// Update Timestamps
document.getElementById('cryptoTimestamp').innerText = `Data fetched on: ${cryptoResponse.timestamp.toLocaleString()}`;
document.getElementById('stockTimestamp').innerText = `Data fetched on: ${stockResponse.timestamp.toLocaleString()}`;

// Chart 1: Cryptocurrency Prices in PHP
const cryptoLabels = ['Bitcoin', 'Ethereum', 'Cardano', 'Solana', 'Ripple'];
const cryptoData = [
cryptoResponse.data.bitcoin.php,
cryptoResponse.data.ethereum.php,
cryptoResponse.data.cardano.php,
cryptoResponse.data.solana.php,
cryptoResponse.data.ripple.php
];
const cryptoValuesContainer = document.getElementById('cryptoValues');
cryptoValuesContainer.innerHTML = cryptoData.map((value, index) =>
`<div class="value-label">${cryptoLabels[index]}<br>₱${value.toFixed(2)}</div>`
).join('');

new Chart(document.getElementById('cryptoPricesChart').getContext('2d'), {
type: 'bar',
data: {
labels: cryptoLabels,
datasets: [{
label: 'Cryptocurrency Prices (PHP)',
data: cryptoData,
backgroundColor: 'rgba(255, 206, 86, 0.2)',
borderColor: 'rgba(255, 206, 86, 1)',
borderWidth: 1
}]
},
options: {
responsive: true,
scales: {
y: { beginAtZero: true }
}
}
});

// Chart 2: Stock Prices in PHP
const stockLabels = ['Apple', 'Microsoft', 'Google', 'Amazon', 'Tesla'];
const stockData = stockResponse.data;
const stockValuesContainer = document.getElementById('stockValues');
stockValuesContainer.innerHTML = stockData.map((value, index) =>
`<div class="value-label">${stockLabels[index]}<br>₱${value.toFixed(2)}</div>`
).join('');

new Chart(document.getElementById('stockPricesChart').getContext('2d'), {
type: 'bar',
data: {
labels: stockLabels,
datasets: [{
label: 'Stock Prices (PHP)',
data: stockData,
backgroundColor: 'rgba(75, 192, 192, 0.2)',
borderColor: 'rgba(75, 192, 192, 1)',
borderWidth: 1
}]
},
options: {
responsive: true,
scales: {
y: { beginAtZero: true }
}
}
});
};

// Fetch trivia data
let currentQuestionIndex = 0;
let score = 0;
let quizData = [];

fetch('https://opentdb.com/api.php?amount=10&type=multiple')
.then(response => response.json())
.then(data => {
quizData = data.results;
loadQuestion();
})
.catch(error => console.error("Error fetching quiz data:", error));

const loadQuestion = () => {
const question = quizData[currentQuestionIndex];
const quizContent = document.getElementById('quiz-content');
const questionHTML = `
<div class="question">${question.question}</div>
<ul class="options">
${[...question.incorrect_answers, question.correct_answer].map(option =>
`<li><input type="radio" name="option" value="${option}"> ${option}</li>`
).join('')}
</ul>
`;
quizContent.innerHTML = questionHTML;
};

const nextQuestion = () => {
const selectedOption = document.querySelector('input[name="option"]:checked');
if (selectedOption) {
const correctAnswer = quizData[currentQuestionIndex].correct_answer;
if (selectedOption.value === correctAnswer) {
score++;
}
}
currentQuestionIndex++;
if (currentQuestionIndex < quizData.length) {
loadQuestion();
} else {
document.getElementById('result').innerText = `Your score is ${score} out of ${quizData.length}`;
score = 0;
currentQuestionIndex = 0; // Reset for next round
}
};

// Joke API function
const getJoke = async () => {
const jokeElement = document.getElementById('joke');
try {
const response = await axios.get('https://v2.jokeapi.dev/joke/Programming?type=single');
jokeElement.innerText = response.data.joke || 'No joke available';
} catch (error) {
jokeElement.innerText = 'Failed to load joke!';
}
};

// Initialize charts
renderCharts();
</script>

</body>
</html>
