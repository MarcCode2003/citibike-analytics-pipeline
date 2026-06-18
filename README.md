# CitiBike Urban Mobility Analytics Pipeline

## 📌 Project Overview
An end-to-end relational infrastructure engineered to map, process, and analyze distributed hardware telemetry logs from the **CitiBike** smart-bike sharing system. This project demonstrates backend database design, query optimization, and technical data analytics to monitor live node statuses, tracking fleet availability, and minimizing infrastructure failure risks.

---

## 💼 Business Problem Statement
Urban mobility systems generate massive, unstructured telemetry logs across thousands of active hardware stations. Without a structured relational framework, operations management suffers from:
1. **Node Collisions:** Inability to track real-time bike allocations leading to empty or overflowing stations.
2. **Security Vulnerabilities:** Risk of ID enumeration and unauthorized node tampering due to poorly structured relational mapping.
3. **Operational Blindspots:** Lack of fast, optimized querying to detect exactly which stations require urgent fleet redistribution.

This pipeline solves these problems by mapping live status metrics (`station_status`) back to core station data (`citibike_stations`), allowing operations managers to pull real-time hardware metrics in milliseconds.

---

## 🧬 Database Architecture & Schema (ERD Context)
The infrastructure utilizes a highly normalized relational model to map tracking nodes cleanly while maintaining reference integrity.

* **`citibike_stations` (Dimension Table):** Stores structural station metadata. Key column mapped: `short_name` (Station Code identifier exposed to hardware logs).
* **`station_status` (Fact Table):** Captures real-time, volatile telemetry data from the field. Key column mapped: `bikes_available` (Total active fleet count per node).

### Relational Strategy against Security Risks:
* **Anti-ID Enumeration:** The internal `station_id` is kept strictly relational and decoupled from external hardware endpoints.
* **Collision Mitigation:** Enforced explicit database constraints ensuring that `short_name` records remain uniquely distinct and isolated during bulk updates.

---

## 🛠️ Core Analytics: The Fleet Optimization Query
To instantly detect which active smart-stations have the highest number of available bikes for immediate redistribution, the following production-grade SQL query was engineered and optimized:

```sql
SELECT 
    cbs.short_name, 
    SUM(ss.bikes_available) AS total_bikes 
FROM 
    citibike_stations AS cbs 
INNER JOIN 
    station_status AS ss ON cbs.station_id = ss.station_id 
WHERE 
    short_name IS NOT NULL 
GROUP BY 
    cbs.short_name 
ORDER BY 
    total_bikes DESC;

```

---

## 📊 Analytics Dashboard & Data Insights
The processed data from the query above is piped directly into a native visual component (**Chart.js**) on the monitoring interface, capturing real-time asset tracking data points:

| Station Code (`short_name`) | Total Bikes Available (`total_bikes`) | Operational Status / Action Item |
| :--- | :--- | :--- |
| **5216.06** | **45** | 🟢 **Surplus Risk:** Station near max capacity. Requires fleet extraction. |
| **5033.01** | **12** | 🟡 **Stable:** Optimal operating threshold for commuter balancing. |
| **3762.08** | **0** | 🔴 **Critical Blindspot:** Station completely empty. Requires immediate deployment. |
| **5667.08** | **0** | 🔴 **Critical Blindspot:** Station completely empty. Requires immediate deployment. |

### 💡 Key Business Recommendations:
1. **Dynamic Redistribution:** Fleet logistics teams should prioritize deploying distribution trucks to extract units from node **`5216.06`** (45 bikes available) and immediately refill nodes **`3762.08`** and **`5667.08`** (0 bikes available) to prevent commuter friction and service downtime.
2. **Predictive Capacity Planning:** Stations flashing consecutive zero-bike telemetry markers during morning/evening rush hours flag high-demand commuter corridors, signaling a strategic need for physical dock expansion and capital resource allocation.

---

## 📂 Data Pipeline Artifacts & Datasets
To review the structural transformation engineered through this relational optimization tracking matrix, the specific raw dataset logs and optimized output streams can be directly accessed below:

* **Source File:** [📁 citibike_raw_telemetry.csv](citibike_raw_telemetry.csv) — The raw, unstructured distributed hardware telemetry logs capturing volatile node updates directly from the field.
* **Output File:** [📊 citibike_optimized_output.csv](citibike_optimized_output.csv) — The cleaned, processed, and aggregated relational schema results filtered via the fleet optimization engine ready for production dashboard consumption.
