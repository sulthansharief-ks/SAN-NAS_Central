
### **Why Both Exist (and why your find is critical) ⚖️:**
* **The GUI Method  🖱️:** Triggered via a web browser. It is easier and doesn't require OS-level access to the SVP. 
* **The CLI Method  ⌨️:** Executing `Dump_Normal.bat` directly from `C:\MAPP\wk\...`. **This is the bulletproof method. 🛡️** If the web interface crashes, Java hangs, or the Storage Navigator is completely unresponsive, the CLI method is your *only* way to get logs. (In fact, clicking "Export" in the GUI actually just runs that exact `.bat` file in the background!).


---

# 🗄️ Standard Operating Procedure: Collecting System Dumps (Logs)
**System:** Hitachi VSP G-Series / F-Series / G1000 / G1500  
**Tools:** Maintenance Utility (Web) 🌐 OR SVP Command Prompt (CLI) 💻

---

## **1. Purpose 🎯**
To safely collect system configuration, hardware status, and error logs (System Dump) for Hitachi Vantara Support engineering to diagnose array issues. 🩺

---

## **2. Method A: Web GUI (Primary Method) 🖱️**
*Use this method if the Storage Navigator web interface is healthy and accessible.* ✅

1. Open your web browser and navigate to the **SVP IP address** 🔗.
2. Select the **Maintenance Utility (MU)** option and log in 🔑.
3. On the left-hand navigation tree, click on **Administration** > **Export Tool** 📂.
4. Select **Normal Dump** 📄.
5. Click **Export** (or Execute) 🚀. 
6. Wait for the task to reach **Completed** status ⏳ *(this can take 20–60 minutes).*
7. Download the resulting `.tgz` file to your workstation 📥.

---

## **3. Method B: CLI via SVP (Advanced/Fallback Method) 👨‍💻**
*Use this method if the web interface is hung, inaccessible, or if support specifically requests a manual OS-level dump.* 🚨

### **🚪 Step 1: SVP Access & Preparation**
* Establish an RDP (Remote Desktop) session directly to the Windows OS of the SVP 🖥️.
* **CRITICAL 🛑:** Close all active Device Manager - Storage Navigator sessions on the SVP to prevent file locks.

### **⚙️ Step 2: Execute the Dump Tool**
* Open a Windows **Command Prompt** as Administrator 🛠️.
* Change the directory to the specific sequence for your storage array using its serial number (e.g., starting with an 8).
  * *💡 Tip: Type `cd "C:\MAPP\wk\8` and hit the `TAB` key to auto-fill the rest of the sequence.*
  * **Command:** `cd "C:\MAPP\wk\[Serial_Number]\DKC200\mp\pc"`
* Execute the dump batch file and specify an output destination folder (ensure there is a space between the command and the destination) 📝.
  * **Command:** `Dump_Normal.bat C:\Result_[Serial_Number]`
  * *(⚠️ Only run `Dump_Detail.bat` if explicitly instructed by Hitachi Support).*

### **👀 Step 3: Monitor & Acknowledge**
* The command prompt will display `"Executing..."` 🔄
* Wait for the process to finish. It will eventually display `"zSv_AutoDump.exe is completed."` ✅
* A completion message box will pop up on the Windows desktop. Press any key to acknowledge and close it 🆗.

### **🗂️ Step 4: Retrieve the Files**
* Navigate to your output destination (e.g., `C:\Result_[Serial_Number]`) 🔍.
* Locate the file named **`hdcp.tgz`** 📦. This is the system dump.
* *(Note: If the dump fails, locate `zSv_AutoDump.log` in the same directory to provide to support 📑).*

---

## **4. Data Upload to Hitachi (TUF) ☁️**
* System Dumps are too large for email 🚫📧.
* Log in to the **Hitachi Vantara Support Connect** portal 🔐.
* Use the Secure File Transfer link (TUF) provided in your Service Request (SR) ticket 🔗.
* Upload **`hdcp.tgz`** ⬆️ and update the ticket notes to notify Global Support 💬.

---
**Prepared by:** Storage Administration Team 🧑‍💼  
**Last Updated:** February 2026 📅

---

**Would you like me to add a section to this on how to collect the specific "Storage Navigator Trace Logs" in case the web UI itself is crashing?** 🤔
