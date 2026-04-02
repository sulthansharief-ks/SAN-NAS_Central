# 📅 Standard Operating Procedure: Generating the Monthly Storage Capacity Report
**System:** Hitachi VSP Series (All Generations)  
**Management Tools:** Hitachi Ops Center Analyzer (Preferred) / Storage Navigator (Fallback)  

---

## **1. Purpose 🎯**
The Monthly Storage Capacity Report is a critical deliverable for IT leadership. It provides a macro-level view of storage utilization across the entire data center, tracks the effectiveness of Data Reduction (compression/deduplication), and serves as the primary data point for forecasting future hardware purchases before DP Pools hit their depletion thresholds.

---

## **2. Key Metrics to Capture 📊**
Management does not usually need to see individual LUN sizes. Your monthly report should aggregate the following data points per array:
1. **Total Physical Usable Capacity:** The total usable hardware capacity after RAID overhead.
2. **Total Allocated Capacity (Physical Used):** The actual hardware footprint consumed by data.
3. **Total Subscribed Capacity (Provisioned):** The logical amount of space promised to the hosts.
4. **Overall Free Space:** The physical capacity remaining before the array is completely full.
5. **Data Reduction Ratio (Saving Effect):** The compression/deduplication multiplier (e.g., 2.5:1).
6. **DP Pool Health:** Highlighting any specific pools that have crossed the 70% Warning Threshold.

---

## **3. Procedure A: Using Hitachi Ops Center Analyzer (Best Practice) 🚀**
*Hitachi Ops Center Analyzer is purpose-built for this task and can automate the entire process using its predictive analytics engine.*

### **Step 1: Access the Capacity Dashboard**
1. Log in to **Hitachi Ops Center Analyzer**.
2. On the main dashboard, navigate to the **Capacity** tab.
3. Here, you will see a fleet-wide overview of all connected VSP arrays. 

### **Step 2: Generate the Array-Level Report**
1. Click on **Storage Systems** in the left-hand menu.
2. Click the **Export** icon (downward arrow).
3. Select **Capacity Report**. Ops Center will generate a comprehensive CSV/PDF detailing the Total, Used, and Free capacity for every array in your environment, including the Data Reduction ratios.

### **Step 3: Review the "Zero Space" Forecast (Crucial for Management)**
1. In Ops Center Analyzer, navigate to **Analytics** > **Capacity Forecast**.
2. Ops Center uses AI to analyze the last 30-90 days of consumption and projects exactly when each array will hit 100% full.
3. Take a screenshot or export this forecast graph. *This is the most important piece of data for the monthly report, as it dictates the procurement budget.*

---

## **4. Procedure B: Using Storage Navigator (Manual Fallback) 🛠️**
*If Ops Center is unavailable, you must manually pull pool data directly from the SVP of each array and aggregate it in Excel.*

### **Step 1: Export Array-Level Capacity**
1. Log in to **Storage Navigator**.
2. In the left tree, select the top-level **Storage Systems** icon.
3. The main pane will display the **Summary** tab showing Total Capacity, Free Capacity, and Saving Effect (Data Reduction).
4. Take a screenshot or manually record these top-level numbers for your executive summary.

### **Step 2: Export DP Pool Utilization**
1. Navigate to **Storage Systems** > **Pools**.
2. Click the **Column Settings** grid and ensure you have enabled:
   * **Pool Name**
   * **Total Capacity** (Physical limit of the pool)
   * **Pool Used Capacity** (Allocated physical space)
   * **Subscribed Capacity** (Total LUN sizes presented)
   * **Subscription Limit %**
3. In the bottom right corner, click **More Actions** > **Export**.
4. Download the `.csv` file.

### **Step 3: Excel Aggregation & Formatting 📈**
1. Open the CSV in Excel.
2. Add a new column titled **"Physical % Full"**. Use the formula: `=(Pool Used Capacity / Total Capacity) * 100`.
3. Apply Conditional Formatting to this new column:
   * 🟢 **Green:** 0% - 60%
   * 🟡 **Yellow:** 61% - 74%
   * 🔴 **Red:** 75%+ (Requires immediate attention)
4. Hide unnecessary engineering columns (like System Area or Cache Mode) to keep the report clean for non-technical management.

---

## **5. Structuring the Final Report (Email/Presentation format) ✉️**
When distributing the report to leadership, always lead with an Executive Summary before attaching the Excel data.

**Example Executive Summary Template:**
> **Monthly Storage Capacity Review – [Month, Year]**
> 
> **1. Global Capacity Overview:**
> * **Total Raw Usable:** 2.5 PB
> * **Total Physical Used:** 1.2 PB (48% Utilized)
> * **Data Reduction Ratio:** 3.1:1 (Saving roughly 2.5 PB of physical disk purchases).
>
> **2. DP Pool Health & Alerts:**
> * `PROD_VMware_Pool_01` has reached **76% capacity**. We recommend initiating a hardware expansion quote this month to avoid hitting the 85% depletion threshold by Q3.
> * All other pools are operating optimally below 60% utilization.
>
> **3. Decommissioning Updates:**
> * Reclaimed 50 TB of physical space this month by shredding LDEVs from the retired SQL cluster.
