# Agentic-Insights-Workflow
Automated Multi-Agent Orchestration for Data Cleaning, EDA, Visualization, and Insight Reporting

## 📘 Overview

The **Agentic Insights Pipeline** is a fully autonomous, multi-agent AI system implemented in **n8n**.  
It performs end‑to‑end data analysis using:

- A **Controller Agent** for orchestration  
- Multiple **Specialized Agents** for schema detection, data cleaning, EDA, feature scoring, and report building  
- **Built‑in n8n tools** (HTTP Request, File Processing, CSV Parser, LLM node, etc.)  
- A **Custom Tool** for insight normalization and report construction  
- Automated **chart generation** using QuickChart  
- Automated **Markdown + PDF report generation**

This project fulfills all requirements for the **Data Analysis** domain of the *Building Agentic Systems Assignment*.

---

# 🧠 System Architecture

This system is built using a **centralized multi-agent framework**.

Below is the ASCII architecture diagram requested:

```
                               +-----------------------+
                               |      CSV Input        |
                               +-----------+-----------+
                                           |
                                           v
                                +----------+----------+
                                | Schema Detector     |
                                |      Agent          |
                                +----------+----------+
                                           |
                                           v
                                +----------+----------+
                                |   Controller Agent  |
                                | (Decision Logic)    |
                                +----------+----------+
                                 /       |       \
                                /        |        \
                               v         v         v
                +--------------+--+  +---+--------------+  +-------------------+
                | Data Cleaning |  |   EDA Agent       |  | Feature Importance |
                |     Agent     |  | (Stats + Dist.)   |  |       Agent       |
                +------+--------+  +----------+---------+  +---------+---------+
                       |                      |                     |
                       +-----------+----------+----------+----------+
                                   |                     |
                                   v                     v
                      +------------+--+        +---------+-----------+
                      | Normalize      |        | Chart Generators   |
                      | Insights Tool  |        |  (5 HTTP Requests) |
                      +--------+-------+        +---------+----------+
                               |                           |
                               +------------+--------------+
                                            |
                                            v
                                 +----------+-----------+
                                 |  Report Builder      |
                                 |    (Custom Tool)     |
                                 +----------+-----------+
                                            |
                                            v
                               +------------+------------+
                               |  PDF / MD / JSON Output |
                               +--------------------------+
```

---

# 🧩 Components

## 1️⃣ Controller Agent (Primary Orchestrator)

The **Controller Agent** is responsible for:

- Determining workflow order  
- Delegating tasks to specialized agents  
- Maintaining global memory  
- Handling errors and fallback logic  
- Ensuring workflow stops gracefully on invalid data  
- Passing structured output between agents  
- Validating tool outputs  

### Responsibilities
| Capability | Description |
|-----------|-------------|
| Task Delegation | Routes the dataset through schema → cleaning → EDA → insights → charts |
| Error Handling | Detects malformed CSV, corrupted JSON, or tool failures |
| Communication | Ensures consistent data transfer between agents |
| Memory | Stores schema, statistics, and intermediate insights |

---

## 2️⃣ Specialized Agents

### 🔹 **Schema Detector Agent**
- Infers data types  
- Detects numeric vs categorical columns  
- Outputs schema JSON  
- Alerts Controller of missing fields  

### 🔹 **Data Cleaning Agent**
- Removes/normalizes missing values  
- Converts string numbers → numeric  
- Standardizes category names  
- Saves cleaned CSV  

### 🔹 **EDA Agent**
- Computes distributions  
- Summary statistics (mean, median, std)  
- Min/max/outliers  
- Correlation matrix  
- Outputs structured EDA JSON  

### 🔹 **Feature Importance Agent**
- Computes variance-based importance  
- Ranks input features  
- Outputs normalized scores  

### 🔹 **Report Builder Agent (Custom Tool)**
- Converts processed results into Markdown  
- Inserts chart URLs  
- Adds narrative insights  
- Prepares PDF  

---

# 🛠 Tools Used

## ✔ Built-In Tools (Required: 3 — Used: 5)
| Tool | Purpose |
|------|---------|
| **CSV Parser** | Convert dataset into JSON objects |
| **Read Binary File** | Load uploaded CSV |
| **Write Binary File** | Save cleaned CSV, charts, and PDF |
| **HTTP Request** | Generate charts through QuickChart |
| **OpenAI Node** | Summaries, insights, interpretations |

---

# 🔧 Custom Tool

### **Normalize Insights Tool**
A custom-built n8n "Code" node that:

- Flattens EDA results  
- Converts stats into readable JSON  
- Extracts key insights dynamically  
- Produces consistent inputs for LLM summary  

### **Report Builder Tool**
- Generates Markdown  
- Embeds image URLs  
- Formats insights into an executive-style report  
- Converts Markdown → PDF file  

---

# 📈 Automated Chart Generation

The system generates **5 analytic charts**:

1. Missingness Chart  
2. Boxplot Distribution  
3. Feature Importance Chart  
4. Correlation Heatmap  
5. Outlier Scatter Plot  

These charts are produced using **individual QuickChart API calls**.

---

# 📄 Output Files

All files saved automatically:

```
/data
    cleaned_data.csv
    eda.json
    insights_normalized.json

/charts
    chart_0.png
    chart_1.png
    chart_2.png
    chart_3.png
    chart_4.png

```

---

# ▶️ Execution Flow

1. **Upload CSV**  
2. **Schema Detection**  
3. **Cleaning**  
4. **EDA + Importance**  
5. **Chart Generation**  
6. **LLM Insight Summaries**  
7. **Report Assembly**  
8. **PDF Export**  

---

# 📦 Requirements

Although n8n manages packages internally, here is the reproducibility list:

```
pandas
numpy
matplotlib
reportlab
json5
python-dotenv
openai
requests
```

---

Fallback Strategies Implemented in the Workflow

Multi-Field LLM Response Fallback
The workflow uses a tiered extraction strategy in the Insights Normalization Agent. If the LLM’s output is not found in the expected field (message.content), the system automatically checks several alternative fields (message, result, response, content) and ultimately falls back to converting the entire JSON into a readable string. This ensures the workflow continues functioning even when the LLM changes its response structure.

Robust Data Cleaning Fallbacks
The Cleaning Agent incorporates built-in fallback rules that guarantee successful preprocessing regardless of data variability. Numeric columns fall back to median imputation, categorical columns default to mode imputation, and outlier handling gracefully skips capping when insufficient data is present. These strategies ensure consistent cleaning without workflow interruption.

Safe Missingness & EDA Fallback Computation
The EDA Agent includes fallback logic that prevents failures when encountering empty or irregular columns. Missingness is safely computed by defaulting to zero when no missing values are detected, and statistical summaries return null or safe defaults when data is insufficient—ensuring the report always remains complete and interpretable.

Report Builder Auto-Fallback Messaging
To maintain report integrity, the Report Builder automatically inserts human-readable fallback messages (e.g., “No cleaning operations logged”, “No missing data detected”, “No feature importance scores computed”) whenever a section has no available data. This guarantees that the final report is always comprehensive, clear, and non-expert friendly—even when upstream agents produce minimal outputs.

# 🧪 Evaluation Report Summary

### Accuracy  
- Schema detection correct for all known fields  
- Variance-based feature scoring consistent  

### Reliability  
- Works under missingness  
- Handles non-numeric corruption  

### Efficiency  
- Workflow completes in seconds  
- Low API overhead  

### Limitations  
- Variance-based importance is not model-driven  
- Heatmap resolution limited by QuickChart  
- Complex PDFs require formatting constraints  

---

# 🧭 Future Enhancements

- Replace variance importance → SHAP values  
- Add anomaly detection model  
- Add reinforcement learning loop for agent improvement  
- Add interactive HTML dashboard generation  
- Add database ingestion (Snowflake, PostgreSQL)  
- Add on-chain feedback loop for QA  

---

# 📬 Author

**Kunal Kale**  
MS in Information Systems  
Northeastern University  
Boston, MA  

---

Video Demonstration of the workflow

https://drive.google.com/file/d/1S8JgahMvvIsIoSwi7jfJgsompvv5-XEG/view?usp=sharing

# ✅ End of README
