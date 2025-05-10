<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Options Expiration Calculator</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 30px; max-width: 700px; }
    label, input, select, button { display: block; width: 100%; margin: 10px 0; }
    .result { font-weight: bold; margin-top: 20px; white-space: pre-wrap; }
    canvas { margin-top: 30px; }
  </style>
</head>
<body>
  <h2>Options Expiration & Strike Suggestion Calculator</h2><label for="stockPrice">Stock Price ($):</label> <input type="number" id="stockPrice" step="0.01" required>

<label for="iv">Implied Volatility (%):</label> <input type="number" id="iv" step="0.01" required>

<label for="expectedMove">Expected Move ($):</label> <input type="number" id="expectedMove" step="0.01" required>

<label for="premium">Option Premium Paid ($):</label> <input type="number" id="premium" step="0.01" required>

<label for="direction">Trade Direction:</label> <select id="direction"> <option value="call">Call (Bullish)</option> <option value="put">Put (Bearish)</option> </select>

<button onclick="calculateExpiration()">Calculate Best Expiration & Strike</button>

  <div class="result" id="result"></div><canvas id="chart" width="600" height="300"></canvas>

  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>  <script>
    let chart;

    function calculateExpiration() {
      const S = parseFloat(document.getElementById("stockPrice").value);
      const IV = parseFloat(document.getElementById("iv").value) / 100;
      const move = parseFloat(document.getElementById("expectedMove").value);
      const premium = parseFloat(document.getElementById("premium").value);
      const direction = document.getElementById("direction").value;

      if (!S || !IV || !move || !premium) {
        document.getElementById("result").innerText = "Please fill all fields correctly.";
        return;
      }

      const days = Math.pow((move / (S * IV)), 2) * 365;
      const roundedDays = Math.round(days);
      let recommendation = "";

      if (roundedDays <= 1) {
        recommendation = "Use same-day (0DTE) or 1DTE options.";
      } else if (roundedDays <= 5) {
        recommendation = "Use weekly options with 3–5 DTE.";
      } else if (roundedDays <= 21) {
        recommendation = "Use 1–3 week expirations (ideal for swing trades).";
      } else {
        recommendation = "Consider monthly or longer expirations (position trading or hedging).";
      }

      let strikeSuggestion = "";
      let strikePrice = 0;
      let breakeven = 0;
      let potentialProfit = 0;

      if (direction === "call") {
        strikePrice = S + move;
        breakeven = strikePrice + premium;
        potentialProfit = Math.max(0, (S + move) - breakeven);
        strikeSuggestion = `Suggested Call Strike: $${strikePrice.toFixed(2)} (target OTM)`;
      } else {
        strikePrice = S - move;
        breakeven = strikePrice - premium;
        potentialProfit = Math.max(0, breakeven - (S - move));
        strikeSuggestion = `Suggested Put Strike: $${strikePrice.toFixed(2)} (target OTM)`;
      }

      const riskReward = `Tip: Use tight spreads, check volume > 100, and delta ~0.3–0.5 for balance of cost and probability.`;
      const profitInfo = `Breakeven Price: $${breakeven.toFixed(2)}\nPotential Profit (if move hits target): $${potentialProfit.toFixed(2)}`;

      document.getElementById("result").innerText =
        `Estimated time for move: ${roundedDays} days\n` +
        `Recommendation: ${recommendation}\n` +
        `${strikeSuggestion}\n` +
        `${riskReward}\n` +
        `${profitInfo}`;

      drawChart(S, strikePrice, breakeven);
    }

    function drawChart(stockPrice, strikePrice, breakeven) {
      const ctx = document.getElementById("chart").getContext("2d");
      const labels = ["Current Price", "Strike Price", "Breakeven"];
      const data = [stockPrice, strikePrice, breakeven];

      if (chart) chart.destroy();

      chart = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [{
            label: 'Price ($)',
            data: data,
            backgroundColor: ['#4CAF50', '#FF5733', '#3366CC']
          }]
        },
        options: {
          scales: {
            y: {
              beginAtZero: false
            }
          }
        }
      });
    }
  </script></body>
</html>