# Predictive Maintenance with Linear Regression-Based Alerts

## Project Summary

This project implements a **predictive maintenance system** that uses linear regression to detect anomalies in industrial current data across 8 axes. The system generates alerts and errors when current values deviate significantly from expected trends, enabling early detection of potential equipment failures.

### Key Features
- Connects to Neon.tech PostgreSQL database for training data
- Builds univariate linear regression models (Time → Current) for each axis
- Analyzes residuals to discover optimal alert/error thresholds
- Generates synthetic test data with proper normalization and standardization
- Implements real-time anomaly detection with configurable thresholds
- Visualizes regression lines with alert/error annotations

---

## Setup Instructions

### Prerequisites
- Python 3.8 or higher
- PostgreSQL database (Neon.tech account)
- Git

### Installation Steps

1. **Clone the repository**
   ```bash
   git clone <your-repo-url>
   cd predictive_maintenance_lab
   ```

2. **Create a virtual environment** (recommended)
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Set up Neon.tech Database**
   
   a. Create a free account at [neon.tech](https://neon.tech)
   
   b. Create a new project and database
   
   c. Note your connection string (it will look like):
      ```
      postgresql://user:password@ep-xxx-xxx.us-east-2.aws.neon.tech/neondb?sslmode=require
      ```
   
   d. Create a `.env` file in the project root:
      ```bash
      touch .env
      ```
   
   e. Add your database credentials to `.env`:
      ```
      DB_HOST=ep-xxx-xxx.us-east-2.aws.neon.tech
      DB_NAME=neondb
      DB_USER=your_username
      DB_PASSWORD=your_password
      DB_PORT=5432
      ```

5. **Run the Jupyter Notebook**
   ```bash
   jupyter notebook notebooks/predictive_maintenance_analysis.ipynb
   ```

---

## Regression & Alert Rules

### Regression Model
For each axis (#1 through #8), we fit a **univariate linear regression model**:

```
Current = β₀ + β₁ × Time + ε
```

Where:
- **β₀** = intercept (baseline current)
- **β₁** = slope (rate of change over time)
- **ε** = residual error

### Threshold Discovery Process

The thresholds were discovered through systematic residual analysis:

1. **Residual Calculation**: For each data point, we computed:
   ```
   Residual = Observed_Current - Predicted_Current
   ```

2. **Statistical Analysis**:
   - Computed mean and standard deviation of residuals
   - Identified outliers using the Interquartile Range (IQR) method
   - Analyzed distribution patterns using boxplots and histograms

3. **Threshold Selection**:
   - **MinC (Alert Threshold)**: Set at **Mean + 2σ** (2 standard deviations above regression line)
   - **MaxC (Error Threshold)**: Set at **Mean + 3σ** (3 standard deviations above regression line)
   - **T (Time Window)**: Set at **10 seconds** of continuous deviation

### Alert & Error Logic

**Alert Condition:**
- Triggered when current exceeds regression prediction by ≥ MinC kWh
- Sustained for ≥ T seconds continuously
- Indicates early warning of potential issue

**Error Condition:**
- Triggered when current exceeds regression prediction by ≥ MaxC kWh
- Sustained for ≥ T seconds continuously
- Indicates critical anomaly requiring immediate attention

### Justification

These thresholds were chosen based on:
- **Statistical significance**: Using 2σ and 3σ captures 95% and 99.7% of normal variations
- **Predictive maintenance context**: Early alerts (2σ) allow preventive action, while errors (3σ) indicate urgent issues
- **Data analysis**: Residual distributions showed minimal outliers beyond 2σ, confirming appropriate sensitivity
- **Time window**: 10 seconds filters transient spikes while catching sustained anomalies

---

## Project Structure

```
predictive_maintenance_lab/
├── README.md                          # This file
├── requirements.txt                   # Python dependencies
├── .env                              # Database credentials (not in git)
├── data/
│   ├── RMBR4-2_export_test.csv       # Training data
│   └── synthetic_test_data.csv       # Generated test data
├── notebooks/
│   └── predictive_maintenance_analysis.ipynb  # Main analysis notebook
├── scripts/
│   └── database_setup.py             # Database initialization script
└── outputs/
    ├── alerts_log.csv                # Detected alerts
    ├── errors_log.csv                # Detected errors
    └── plots/                        # Visualization outputs
```

---

## Results & Visualizations

### Sample Results

**Axes with Detected Anomalies:**
- Axis #3: 5 alerts, 2 errors
- Axis #5: 3 alerts, 1 error
- Axis #7: 4 alerts, 0 errors

**Example Alert:**
- Timestamp: 2022-10-17 14:23:15
- Axis: #3
- Deviation: +2.3 kWh above prediction
- Duration: 15 seconds
- Type: Alert

See `outputs/plots/` for detailed visualizations showing:
- Regression lines for each axis
- Alert markers (yellow triangles)
- Error markers (red circles)
- Residual distributions

---

## How to Use

1. **Load training data into database**: Run all cells in the notebook sequentially
2. **Train regression models**: Models are automatically fitted for all 8 axes
3. **Generate test data**: Synthetic data is created with matching statistics
4. **Run anomaly detection**: Real-time monitoring with alert/error detection
5. **Review results**: Check `outputs/alerts_log.csv` and visualizations

---

## Database Schema

The PostgreSQL database contains the following table:

```sql
CREATE TABLE current_readings (
    id SERIAL PRIMARY KEY,
    timestamp TIMESTAMP,
    axis_1 FLOAT,
    axis_2 FLOAT,
    axis_3 FLOAT,
    axis_4 FLOAT,
    axis_5 FLOAT,
    axis_6 FLOAT,
    axis_7 FLOAT,
    axis_8 FLOAT
);
```

---

## Author

**Your Name**  
Student ID: XXXXXXXXX  
Course: CSCN8010-26W-Sec1 - Foundations of Machine Learning Frameworks  

---

## License

This project is for educational purposes as part of the CSCN8010 course assignment.
