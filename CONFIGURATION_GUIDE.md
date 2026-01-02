# Quick Configuration Guide

## 🚀 Quick Start - 3 Steps

### Step 1: Set Your API Credentials & MLflow Experiment (Section 1)
```python
# Genie API Configuration
WORKSPACE_URL = "https://your-workspace.cloud.databricks.com"
GENIE_SPACE_ID = "your-genie-space-id"
API_TOKEN = dbutils.secrets.get(scope="your-scope", key="your-key")

# MLflow Configuration
MLFLOW_EXPERIMENT_NAME = "/Shared/genie-load-test"  # Change to your desired path
```

### Step 2: Choose Your Load Test Scenario (Section 2)
```python
ACTIVE_SCENARIO = "high_load"  # Change this to any scenario below
```

### Step 3: Run the Notebook
Execute all cells - the test will automatically use your selected scenario!

---

## 📊 Available Load Test Scenarios

### 1. `single_test` - Validation Test
- **Questions**: 1
- **Duration**: Immediate
- **Workers**: 1
- **Use**: Initial validation and testing

### 2. `light_load` - Light Load
- **Questions**: 10 over 60 seconds
- **Workers**: 3
- **Use**: Baseline performance measurement

### 3. `moderate_load` - Moderate Load
- **Questions**: 20 over 30 seconds
- **Workers**: 5
- **Use**: Medium load testing

### 4. `high_load` - High Load ⭐ (Default)
- **Questions**: 35 over 30 seconds
- **Workers**: 10
- **Use**: Production-like load simulation
- **Rate**: ~1.17 questions/second

### 5. `stress_test` - Stress Test
- **Questions**: 50 over 20 seconds
- **Workers**: 15
- **Use**: High stress testing
- **Rate**: 2.5 questions/second

### 6. `burst_test` - Burst Test
- **Questions**: 50 immediately
- **Workers**: 20
- **Use**: Maximum burst load, rate limit testing

### 7. `sustained_load` - Sustained Load
- **Questions**: 100 over 120 seconds
- **Workers**: 10
- **Use**: Long duration load testing
- **Rate**: ~0.83 questions/second

---

## 🎯 Common Use Cases

### Scenario 1: I want to test 35 questions in 30 seconds
```python
ACTIVE_SCENARIO = "high_load"
```

### Scenario 2: I want to test 100 questions in 1 minute
```python
CUSTOM_SCENARIO = {
    "num_questions": 100,
    "target_duration": 60,
    "max_workers": 15,
    "description": "100 questions in 1 minute"
}
USE_CUSTOM_SCENARIO = True
```

### Scenario 3: I want to submit 200 questions immediately
```python
CUSTOM_SCENARIO = {
    "num_questions": 200,
    "target_duration": None,  # None = immediate submission
    "max_workers": 25,
    "description": "200 questions burst"
}
USE_CUSTOM_SCENARIO = True
```

### Scenario 4: I want to test sustained load for 5 minutes
```python
CUSTOM_SCENARIO = {
    "num_questions": 250,
    "target_duration": 300,  # 5 minutes
    "max_workers": 10,
    "description": "5-minute sustained load"
}
USE_CUSTOM_SCENARIO = True
```

---

## ⚙️ Configuration Parameters Explained

### `num_questions`
- **Type**: Integer
- **Description**: Total number of questions to submit in the test
- **Example**: `35` = submit 35 questions total

### `target_duration`
- **Type**: Integer (seconds) or `None`
- **Description**: Time period to spread questions over
- **Examples**:
  - `30` = spread questions over 30 seconds
  - `None` = submit all questions immediately (burst mode)

### `max_workers`
- **Type**: Integer
- **Description**: Maximum number of concurrent threads/workers
- **Recommendation**: Start with 5-10, increase based on your needs
- **Note**: Higher values = more concurrent requests

### `description`
- **Type**: String
- **Description**: Human-readable description of the test
- **Use**: Helps identify tests in MLflow UI

---

## 📈 How to Calculate Expected Rate

**Formula**: `rate = num_questions / target_duration`

**Examples**:
- 35 questions in 30 seconds = 1.17 questions/second
- 50 questions in 20 seconds = 2.5 questions/second
- 100 questions in 120 seconds = 0.83 questions/second

---

## 🔄 Switching Scenarios

### To switch from high_load to stress_test:
1. Go to **Section 2** in the notebook
2. Change: `ACTIVE_SCENARIO = "stress_test"`
3. Re-run the notebook

### To test multiple scenarios sequentially:
Run the notebook multiple times, changing `ACTIVE_SCENARIO` each time. MLflow will track all runs separately.

---

## 🛠️ Advanced Customization

### Custom Questions
Modify the `SAMPLE_QUESTIONS` list in **Section 6**:
```python
SAMPLE_QUESTIONS = [
    "Your custom question 1",
    "Your custom question 2",
    # Add more...
]
```

### MLflow Experiment Name (Section 1)
```python
MLFLOW_EXPERIMENT_NAME = "/Shared/genie-load-test"
```
**Common patterns**:
- `/Shared/genie-load-test` - Shared experiment for team
- `/Users/your.email@company.com/genie-experiments` - Personal experiment
- `/Teams/data-engineering/genie-api-tests` - Team-specific experiment
- `/Projects/api-performance/genie-load-tests` - Project-specific experiment

### Retry Configuration (Section 1)
```python
MAX_RETRIES = 5           # Number of retries for 429 errors
BASE_WAIT_TIME = 1        # Initial wait time
MAX_WAIT_TIME = 60        # Maximum wait time
JITTER_MAX = 2            # Random jitter (prevents thundering herd)
TIMEOUT = 300             # Query timeout in seconds
```

---

## 📝 Example Workflow

1. **Start small**: `ACTIVE_SCENARIO = "single_test"`
2. **Baseline**: `ACTIVE_SCENARIO = "light_load"`
3. **Increase load**: `ACTIVE_SCENARIO = "moderate_load"`
4. **Production simulation**: `ACTIVE_SCENARIO = "high_load"`
5. **Stress test**: `ACTIVE_SCENARIO = "stress_test"`
6. **Max capacity**: `ACTIVE_SCENARIO = "burst_test"`

Each test is tracked in MLflow for comparison!

---

## 💡 Tips

1. **Always start with** `single_test` to validate connectivity
2. **Use** `light_load` to establish baseline performance
3. **Gradually increase** load to identify breaking points
4. **Monitor MLflow** after each test to analyze results
5. **Check rate limits** - if you see many 429 errors, reduce load
6. **Compare scenarios** using MLflow's comparison features

---

## 🆘 Quick Troubleshooting

**Problem**: Too many 429 (rate limit) errors  
**Solution**: Reduce `num_questions`, increase `target_duration`, or reduce `max_workers`

**Problem**: Tests too slow  
**Solution**: Increase `max_workers` (if within rate limits)

**Problem**: Want faster submission rate  
**Solution**: Reduce `target_duration` or set to `None` for burst mode

**Problem**: Need longer test duration  
**Solution**: Increase both `num_questions` and `target_duration`

