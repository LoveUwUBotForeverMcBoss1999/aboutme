# Flask Trading Dashboard System

A Flask-based web application to track and visualize trading performance with tables, charts, and summaries.

## Project Structure
```
trading_dashboard/
├── static/
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── charts.js
├── templates/
│   ├── base.html
│   └── dashboard.html
├── app.py
├── trade_analyzer.py
└── requirements.txt
```

## Requirements

```txt
flask==3.0.0
pandas==2.1.1
plotly==5.18.0
python-dotenv==1.0.0
```

## Installation

1. Create a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

## Implementation

### 1. Main Flask Application (app.py)
```python
from flask import Flask, render_template, request, jsonify
import pandas as pd
from trade_analyzer import analyze_trades

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('dashboard.html')

@app.route('/analyze', methods=['POST'])
def analyze():
    try:
        # Get trading data from form
        trades_data = request.form.get('trades')
        
        # Convert string data to DataFrame
        trades_list = []
        for line in trades_data.strip().split('\n'):
            if line:
                result, stake, profit, capital, rs_value = line.split('\t')
                trades_list.append({
                    'Result': result,
                    'Stake': float(stake.replace('$', '')),
                    'Profit': float(profit.replace('$', '')),
                    'Available_Capital': float(capital.replace('$', '')),
                    'Rs_Value': float(rs_value.replace('Rs.', '').replace('-Rs.', '-'))
                })
        
        df = pd.DataFrame(trades_list)
        
        # Analyze trades
        analysis = analyze_trades(df)
        
        return jsonify(analysis)
    except Exception as e:
        return jsonify({'error': str(e)}), 400

if __name__ == '__main__':
    app.run(debug=True)
```

### 2. Trade Analysis Module (trade_analyzer.py)
```python
import pandas as pd

def analyze_trades(df):
    """Analyze trading data and return summary statistics."""
    
    # Calculate summary statistics
    total_trades = len(df)
    winning_trades = len(df[df['Result'] == 'W'])
    losing_trades = len(df[df['Result'] == 'L'])
    win_rate = (winning_trades / total_trades) * 100
    
    net_profit_usd = df['Profit'].sum()
    net_profit_rs = df['Rs_Value'].sum()
    
    largest_win = df[df['Result'] == 'W']['Rs_Value'].max()
    largest_loss = df[df['Result'] == 'L']['Rs_Value'].min()
    
    # Prepare trade list for display
    trades = df.to_dict('records')
    
    return {
        'summary': {
            'total_trades': total_trades,
            'winning_trades': winning_trades,
            'losing_trades': losing_trades,
            'win_rate': round(win_rate, 1),
            'net_profit_usd': round(net_profit_usd, 2),
            'net_profit_rs': round(net_profit_rs, 2),
            'largest_win': round(largest_win, 2),
            'largest_loss': round(largest_loss, 2)
        },
        'trades': trades
    }
```

### 3. Base Template (templates/base.html)
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Trading Dashboard</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
    <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
</head>
<body>
    {% block content %}{% endblock %}
    <script src="{{ url_for('static', filename='js/charts.js') }}"></script>
</body>
</html>
```

### 4. Dashboard Template (templates/dashboard.html)
```html
{% extends "base.html" %}

{% block content %}
<div class="container">
    <h1>Trading Performance Dashboard</h1>
    
    <!-- Input Form -->
    <div class="input-section">
        <h2>Enter Trading Data</h2>
        <form id="trade-form">
            <textarea id="trades-input" placeholder="Result&#9;Stake&#9;Profit&#9;Available Capital&#9;Rs Value"></textarea>
            <button type="submit">Analyze</button>
        </form>
    </div>

    <!-- Results Section -->
    <div class="results-section" id="results" style="display: none;">
        <!-- Summary Cards -->
        <div class="summary-cards">
            <div class="card">
                <h3>Net Profit (USD)</h3>
                <p id="net-profit-usd"></p>
            </div>
            <div class="card">
                <h3>Net Profit (Rs)</h3>
                <p id="net-profit-rs"></p>
            </div>
            <div class="card">
                <h3>Win Rate</h3>
                <p id="win-rate"></p>
            </div>
        </div>

        <!-- Charts -->
        <div class="charts-section">
            <div id="profit-chart"></div>
            <div id="cumulative-chart"></div>
        </div>

        <!-- Trades Table -->
        <div class="table-section">
            <h2>Trade History</h2>
            <table id="trades-table">
                <thead>
                    <tr>
                        <th>Trade #</th>
                        <th>Result</th>
                        <th>Stake ($)</th>
                        <th>Profit ($)</th>
                        <th>Available Capital ($)</th>
                        <th>Rs Value</th>
                    </tr>
                </thead>
                <tbody></tbody>
            </table>
        </div>
    </div>
</div>
{% endblock %}
```

### 5. Styling (static/css/style.css)
```css
/* Base styles */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: Arial, sans-serif;
    line-height: 1.6;
    background-color: #f5f5f5;
}

.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

/* Input section */
.input-section {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    margin-bottom: 20px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

textarea {
    width: 100%;
    height: 150px;
    padding: 10px;
    margin-bottom: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-family: monospace;
}

button {
    background-color: #007bff;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

button:hover {
    background-color: #0056b3;
}

/* Summary cards */
.summary-cards {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
    margin-bottom: 20px;
}

.card {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    text-align: center;
}

/* Charts section */
.charts-section {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    margin-bottom: 20px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* Table section */
.table-section {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
}

th, td {
    padding: 12px;
    text-align: right;
    border: 1px solid #ddd;
}

th {
    background-color: #f8f9fa;
    font-weight: bold;
}

tr:nth-child(even) {
    background-color: #f9f9f9;
}

.win { 
    background-color: rgba(40, 167, 69, 0.1);
}

.loss {
    background-color: rgba(220, 53, 69, 0.1);
}
```

### 6. Charts JavaScript (static/js/charts.js)
```javascript
document.getElementById('trade-form').addEventListener('submit', async (e) => {
    e.preventDefault();
    
    const tradesData = document.getElementById('trades-input').value;
    
    try {
        const response = await fetch('/analyze', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
            },
            body: `trades=${encodeURIComponent(tradesData)}`
        });
        
        const data = await response.json();
        
        if (data.error) {
            alert(data.error);
            return;
        }
        
        updateDashboard(data);
    } catch (error) {
        alert('Error analyzing trades: ' + error);
    }
});

function updateDashboard(data) {
    // Show results section
    document.getElementById('results').style.display = 'block';
    
    // Update summary cards
    document.getElementById('net-profit-usd').textContent = `$${data.summary.net_profit_usd}`;
    document.getElementById('net-profit-rs').textContent = `Rs ${data.summary.net_profit_rs}`;
    document.getElementById('win-rate').textContent = `${data.summary.win_rate}%`;
    
    // Update table
    const tbody = document.querySelector('#trades-table tbody');
    tbody.innerHTML = '';
    
    data.trades.forEach((trade, index) => {
        const row = document.createElement('tr');
        row.className = trade.Result === 'W' ? 'win' : 'loss';
        
        row.innerHTML = `
            <td>${index + 1}</td>
            <td>${trade.Result}</td>
            <td>$${trade.Stake}</td>
            <td>${trade.Profit >= 0 ? '+' : ''}$${trade.Profit}</td>
            <td>$${trade.Available_Capital}</td>
            <td>${trade.Rs_Value >= 0 ? '+' : ''}Rs ${trade.Rs_Value}</td>
        `;
        
        tbody.appendChild(row);
    });
    
    // Create charts
    createProfitChart(data.trades);
    createCumulativeChart(data.trades);
}

function createProfitChart(trades) {
    const trace = {
        x: trades.map((_, i) => `Trade ${i + 1}`),
        y: trades.map(t => t.Rs_Value),
        type: 'bar',
        marker: {
            color: trades.map(t => t.Result === 'W' ? '#28a745' : '#dc3545')
        }
    };
    
    const layout = {
        title: 'Profit/Loss per Trade (Rs)',
        yaxis: { title: 'Profit/Loss (Rs)' }
    };
    
    Plotly.newPlot('profit-chart', [trace], layout);
}

function createCumulativeChart(trades) {
    let cumulative = 0;
    const cumulativeData = trades.map((t, i) => {
        cumulative += t.Rs_Value;
        return { x: `Trade ${i + 1}`, y: cumulative };
    });
    
    const trace = {
        x: cumulativeData.map(d => d.x),
        y: cumulativeData.map(d => d.y),
        type: 'scatter',
        mode: 'lines+markers',
        line: { color: '#007bff' }
    };
    
    const layout = {
        title: 'Cumulative Profit/Loss (Rs)',
        yaxis: { title: 'Cumulative Profit/Loss (Rs)' }
    };
    
    Plotly.newPlot('cumulative-chart', [trace], layout);
}
```

## Usage

1. Start the Flask application:
```bash
python app.py
```

2. Open your browser and navigate to `http://localhost:5000`

3. Enter your trading data in the textarea using tab-separated format:
```
Result  Stake   Profit  Available Capital   Rs Value
W       $7.54   $4.60   $30.85             Rs.1,353.61
L       $2.16   -$2.16  $2.84              -Rs.634.23
```

4. Click "Analyze" to see:
- Summary statistics cards
- Profit/Loss bar chart
- Cumulative profit line chart
- Detailed trades table

## Features

- Real-time analysis of trading data
- Responsive design
- Interactive charts using Plotly.js
- Color-coded trade results
- Summary statistics
- Detailed trade history table
- Support for both USD and Rs values
- Error handling and validation

## Customization

You can customize the appearance by modifying the CSS in `static/css/style.css`. To add new features:

1. Add new analysis functions in `trade_analyzer.py`
2. Update the Flask route in `app.py`
3. Add new UI elements in `dashboard.html`
4. Add corresponding styling in `style.css`
5. Add new chart functions in `charts.js`

This system provides a foundation that you can build upon based on your specific needs. You might want to add:

- Data persistence (database)
- User authentication
- More advanced analytics
- Export functionality
- Additional visualization types
- Real-time data updates
