# Smart Traffic Data Analysis

A data-driven analysis and secure processing pipeline for IoT-based smart traffic management using the **Metro Interstate Traffic Volume** dataset. The project combines:

- Real-world traffic data analytics
- Machine learning–based congestion prediction
- AES-256-GCM encryption for traffic records
- A lightweight blockchain-inspired ledger for tamper-evident logging
- Role-Based Access Control (RBAC) for controlled decryption
- Performance evaluation of plaintext vs secure pipelines

All code is implemented in a single Jupyter/Colab notebook and is suitable for data analytics, IoT security, and smart city coursework or portfolio projects.

---

## 1. Project Overview

Modern smart city traffic systems rely on dense IoT deployments—cameras, loop detectors, vehicle sensors, and roadside units—to monitor congestion and control traffic in real time. Traditional **centralized** architectures collect all data on a small number of servers, which creates serious risks:

- Single point of failure  
- Unauthorized manipulation of traffic data  
- Privacy leakage through mass tracking of vehicles  
- Limited transparency and auditability  

This project implements a **secure, analytics-driven pipeline** that addresses these issues by:

1. Using a **real traffic dataset** to emulate IoT sensor events.
2. Building **machine learning models** (Random Forest and Logistic Regression) for congestion prediction.
3. Encrypting each traffic record with **AES-256-GCM**.
4. Hashing ciphertext with **SHA-256** and storing it in a **blockchain-style ledger**.
5. Enforcing **RBAC** so only authorized roles can decrypt records.
6. Comparing performance of plaintext vs secure pipelines.

The result is an end-to-end demonstration of how data analytics, cryptography, and blockchain concepts can be combined to build trustworthy IoT traffic systems.

---

## 2. Dataset

**Name:** Metro Interstate Traffic Volume  
**Source:** Kaggle (historical traffic volume on I-94)  
**Size:** ~48,000 records  

Each record includes:

- `date_time` – timestamp
- `traffic_volume` – vehicle count
- `temp`, `rain_1h`, `snow_1h`, `clouds_all` – weather features
- Additional contextual information

In this project, the dataset is transformed into **IoT-like events** with:

- `timestamp`
- `junction_id` (simulated J1–J3)
- `sensor_id` (simulated S1–S3)
- `vehicle_count`
- `avg_speed` (derived)
- Weather features
- `congestion_level` (LOW/MEDIUM/HIGH)
- `is_congested` (binary label for ML)

Cleaned data is saved as:

- `data/Metro_Interstate_Traffic_Volume.csv`
- `data/cleaned_traffic_data.csv`

---

## 3. Methodology

### 3.1 Data Preprocessing & Feature Engineering

- Load raw dataset using **Pandas**.
- Convert `date_time` to proper `datetime` type.
- Derive useful time-based features (hour, day, etc. if needed).
- Create a `congestion_level` label based on `vehicle_count` (LOW/MEDIUM/HIGH).
- Create a binary target `is_congested` for ML (`1` for MEDIUM/HIGH, `0` for LOW).
- Simulate multiple junctions and sensors to mimic a realistic IoT environment.
- Save cleaned data to `cleaned_traffic_data.csv`.

Basic visualization includes:

- Traffic volume over time (first few hundred records).
- Distribution of congestion levels.

### 3.2 Machine Learning Pipeline

Two models are trained:

- **Random Forest Classifier** (main model)
- **Logistic Regression** (baseline)

Steps:

1. Select features:  
   `vehicle_count`, `avg_speed`, `temp`, `rain_1h`, `snow_1h`, `clouds_all`, etc.
2. Split into train and test sets using `train_test_split` with stratification.
3. Scale numerical features using `StandardScaler` (for Logistic Regression).
4. Train both models.
5. Evaluate with:
   - Accuracy
   - Precision
   - Recall
   - F1-score
   - ROC-AUC
6. Visualize:
   - Confusion matrix
   - ROC curve
   - Random Forest feature importance
   - Time-series traffic volume plot

The Random Forest model provides strong congestion prediction performance and highlights key drivers such as **vehicle_count**, **avg_speed**, and weather variables.

### 3.3 AES-256-GCM Encryption Workflow

To protect traffic data:

- Generate a **256-bit symmetric key**.
- For each traffic record:
  1. Convert the record (Python dict) to a JSON string and then to bytes.
  2. Encrypt using `AESGCM` (from the `cryptography` library) with:
     - 256-bit key
     - 12-byte random nonce (`os.urandom(12)`)
  3. Store `(nonce, ciphertext)` in an in-memory encrypted store.

AES-GCM provides:

- **Confidentiality** – encrypted data cannot be read.
- **Integrity & Authenticity** – any change in ciphertext invalidates the GCM tag.

### 3.4 Blockchain-Inspired Ledger

A lightweight blockchain is implemented in Python using `dataclasses`:

- Each **Block** stores:
  - Index
  - Timestamp
  - Junction ID / device info
  - SHA-256 hash of the ciphertext
  - Previous block hash
- The chain is maintained in a list; `is_valid()` verifies:
  - Hash consistency for each block
  - Correct linkage via `prev_hash`

If any ciphertext is modified, its hash mismatch breaks the chain and `is_valid()` fails—providing **tamper evidence**, similar to real blockchain systems.

### 3.5 Role-Based Access Control (RBAC)

A simple RBAC module controls access:

- Roles: e.g., `"AUTHORITY"` and `"RESEARCHER"`.
- Only entities with the `"AUTHORITY"` role are allowed to decrypt records.
- Unauthorized attempts raise an exception and are logged in an `access_log`.

This simulates smart-contract–like policies where only traffic authorities can see sensitive traffic data, but analytics or research roles may only access aggregated or encrypted information.

### 3.6 Performance and Security Evaluation

Two pipelines are implemented:

1. **Baseline (Plaintext) Storage**
   - Simply stores raw Python dictionaries in a list.
   - No encryption, hashing, or blockchain logging.
2. **Secure Pipeline**
   - AES-256-GCM encryption for each record.
   - SHA-256 hash computation.
   - Block creation + appending to the ledger.

Metrics:

- Total time to store N records in baseline vs secure pipeline.
- Per-record encryption/decryption latency.
- Validation of blockchain integrity.
- RBAC enforcement checks.

Visualization:

- Bar chart of plaintext vs secure storage time.
- Encryption vs decryption timing.
- Multi-junction record distribution (J1–J3).

---

## 4. Implementation Details

### 4.1 Environment

- **Language:** Python 3  
- **Notebook:** Jupyter / Google Colab (`notebooks/fianl_IoT.ipynb`)

### 4.2 Key Libraries

- `pandas`, `numpy` – data handling and numerical operations
- `matplotlib` – visualizations
- `scikit-learn` – ML models and metrics
- `cryptography` – AES-GCM encryption (`AESGCM`)
- `hashlib` – SHA-256 hashing
- `dataclasses` – Block structure
- `time`, `os`, `json` – utility and serialization

---

## 5. Results Summary

### 5.1 Machine Learning

- **Random Forest**
  - Achieves strong metrics on the congestion prediction task
  - Feature importance ranking highlights:
    1. `vehicle_count`
    2. `avg_speed`
    3. `clouds_all`
    4. `temp`
    5. `rain_1h`
    6. `snow_1h`
- **Logistic Regression**
  - Slightly lower but comparable performance; useful as a baseline.
- Confusion matrix and ROC curve confirm robust separation between congested and non-congested states.

### 5.2 Encryption Correctness

- Every decrypted record exactly matches the original plaintext.
- Any single-bit modification in ciphertext causes authentication failure.
- Demonstrates **confidentiality, integrity, and authenticity** of AES-GCM.

### 5.3 Blockchain Integrity

- `blockchain.is_valid()` returns **True** after normal operation.
- Tampering with ciphertext or block contents breaks the hash chain and invalidates the ledger.
- Confirms that the system can **detect unauthorized modifications**.

### 5.4 Performance Overhead

- Secure pipeline is slower than plain storage (due to encryption + hashing + block creation).
- However, the measured overhead remains within acceptable limits for traffic systems that operate on **second-level decision cycles**, not microseconds.
- Encryption and decryption take fractions of a millisecond per record in the experimental setup.

### 5.5 Access Control

- Authorized `"AUTHORITY"` users can decrypt.
- `"RESEARCHER"` or other roles are denied decryption and logged.
- Shows how RBAC can be layered on top of encryption and blockchain logging.

Overall, the system demonstrates that **strong security controls** can be added to a traffic analytics pipeline without making it unusable.

---

## 6. Repository Structure

```text
smart-traffic-data-analysis/
│
├── notebooks/
│   └── fianl_IoT.ipynb          # Main Colab/Jupyter notebook
│
├── data/
│   ├── Metro_Interstate_Traffic_Volume.csv
│   └── cleaned_traffic_data.csv
│
├── images/
│   ├── Diagram A.png            # System architecture
│   ├── Diagram C.png            # Secure pipeline flow
│   ├── Diagram F.png            # Blockchain / ledger design
│   ├── Traffic_timestamp.png    # Traffic volume over time
│   ├── confusion_matrix.png
│   ├── feature_importances.png
│   ├── roc_curve.png
│   └── encrypted_vs_plaintext_timing.png
│
├── README.md
├── LICENSE
└── .gitignore
