# 🗄️ Standard Operating Procedure: Hitachi System Dump Collection

When opening a Service Request (SR) with Hitachi Vantara Support for hardware failures, performance issues, or configuration bugs on a VSP array, engineering will almost always ask for a **"System Dump"** (or simply, a "Dump").

Here is exactly what they need and how to securely collect it. 🚀

---

## **1. The "Holy Grail" of Hitachi Logs: The System Dump 📦**
The System Dump is a compressed archive containing everything Hitachi engineering needs to diagnose the array. There are two types, but you will use the first one 99% of the time:

* ✅ **Normal Dump (Always start here):** Contains system configuration, hardware status, error logs (SIMs), and recent I/O traces. This is the standard file Hitachi requests.
* 🛑 **Detail Dump (Only if explicitly asked):** Contains deeper memory dumps and extensive traces. It takes *hours* to generate and is requested only for extremely complex, unresolved performance or microcode bugs.

> ⚠️ **Note:** Never generate a Detail Dump unless Hitachi Support specifically instructs you to do so, as it can temporarily impact SVP performance.

---

## **2. Procedure: Collecting a Normal System Dump 🛠️**
**Management Tool:** Hitachi Maintenance Utility (via SVP)

### **Step 1: Access the Maintenance Utility 🌐**
1. Open your web browser and navigate to the **SVP IP address**.
2. Instead of logging into Storage Navigator, select the **Maintenance Utility (MU)** option.
3. Log in using an account with **Storage Administrator** or **Security Administrator** privileges.

### **Step 2: Navigate to the Export Tool 📂**
1. On the left-hand navigation tree, click on **Administration**.
2. From the expanded menu, select **Export Tool** *(Note: On some older microcode versions, this may be labeled as **Dump Tool**)*.

### **Step 3: Generate the Dump ⚙️**
1. In the Export Tool window, locate the options for the type of dump.
2. Select **Normal Dump**.
3. Click the **Export** (or **Execute**) button at the bottom right.
4. ⏳ **Important Warning:** A prompt will appear warning you that the process takes time. Click **OK** to proceed.
    * *💡 Pro-Tip:* Generating a Normal Dump takes anywhere from **20 to 60 minutes** depending on the size of the array and the current workload. **Do not close the browser tab or refresh the page while the status says "In Progress."**

### **Step 4: Download the File 📥**
1. Once the task reaches **Completed** status, a **Download** button will appear (or the download will start automatically, depending on your browser).
2. Save the `.tgz` or `.zip` file to your local machine.
    * *📝 File Naming Convention:* The file usually looks something like `Normal_Dump_[Serial_Number]_[Date]_[Time].tgz`.

---

## **3. Edge Cases: Other Logs They Might Request 📎**
Depending on the issue, support might ask for these secondary files:

* 💾 **Configuration Backup (`.bak` or `.zip`):**
    * **Why:** Needed if you are performing a microcode upgrade or recovering from a major system failure.
    * **How:** Navigate to `Maintenance Utility > Administration > Configuration Backup`, then click **Backup** and download the file.
* 🖥️ **Storage Navigator (HDvM-SN) Trace Logs:**
    * **Why:** Needed if the array is healthy, but the *web interface* is crashing, throwing Java errors, or tasks are hanging at 0%.
    * **How:** There is a specific "Dump Tool" batch script located on the SVP Windows OS itself (usually in `C:\MAPP\wk\Super\` or similar) that collects web-server traces. Support will usually provide a specific KB article with instructions if this is required.

---



