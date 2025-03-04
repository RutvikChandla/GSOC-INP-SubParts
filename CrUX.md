# Chrome User Experience Report (CrUX) Sample

The **Chrome User Experience Report (CrUX)** is a **public dataset and API** that provides real-user experience data as collected from Chrome users in the field. In essence, **CrUX is how Google samples and aggregates performance metrics from actual users on actual websites**.  

According to Google:  
> *“The Chrome User Experience Report (CrUX) is a public dataset of real user performance data”*  
> – [DEVELOPER.CHROME.COM](https://developer.chrome.com)  

These data come from users who have **opted in** to share anonymized performance metrics and browsing info with Google  
[NODE.SUAYAN.COM](https://node.suayan.com).

### **What Data Does CrUX Contain?**
CrUX contains **user experience metrics** for **millions of websites**, including **Core Web Vitals (CWV)**:
- **Largest Contentful Paint (LCP)**
- **First Input Delay (FID)** / **Interaction to Next Paint (INP)**
- **Cumulative Layout Shift (CLS)**  

CrUX also **breaks down data** by various dimensions, such as:
- **Device type**
- **Connection type**
- **Geography**, etc.

### **How CrUX Collects and Samples Data**
- **Not every page view is recorded** – Chrome employs **sampling strategies** to **limit data collection and preserve privacy**.
- When users **enable usage statistics**, the browser will **occasionally** send up performance metrics for visited pages.
- **URL-keyed data** – metrics are tied to the **page’s URL** (usually aggregated at the **origin level** in public data).
- **Data is anonymized and aggregated** to remove personal identifiers.
- CrUX **builds a statistical picture** of performance, typically reporting **percentile metrics** (e.g., **p75 is often used for CWV**).

For example, **CrUX might report**:  
> *For example.com, the **75th percentile INP** is **220 ms** (i.e., 75% of visits had **INP ≤ 220 ms**, 25% were worse).*

The scale of Chrome’s user base ensures that **even a small sampled percentage yields significant coverage**.

### **CrUX Data Availability & Usage**
- **Collected continuously** and **released monthly**.
- Developers can access CrUX data via:
  - **PageSpeed Insights**
  - **CrUX API**
  - **BigQuery**
- CrUX **powers Google Search Console’s CWV report**.
- **CrUX provides "ground truth" user experience data** – as opposed to lab tests.

### **Limitations of CrUX**
- CrUX **focuses on high-level metrics and distributions**.
- It **does not** provide:
  - Which **specific interaction** was slow.
  - Which **element** caused instability.
- For **detailed diagnostics**, sites must **implement their own RUM analytics** or use **other performance monitoring tools**.

### **Why is CrUX Important?**
As an **official dataset**, CrUX:
- **Tracks improvements** objectively.
- **Compares against thresholds** for CWV compliance.
- **Bridges** the browser’s **internal metrics collection** and **public web performance analytics**.

> *“CrUX samples performance data by capturing metrics from opted-in Chrome users and aggregates them by URL. It provides the data backbone for Core Web Vitals in the field.”*  
> – [SUPPORT.GOOGLE.COM](https://support.google.com)

This **carefully designed sampling** balances **data quality and privacy**:
- **Millions of data points** ensure statistical **significance**.
- **Data is sampled & aggregated** to avoid **storing individual user sessions**.

### **CrUX & Web Vitals**
CrUX plays a **huge role** in **CWV**:
- It’s **how Google determines** whether a site **"passes"** Web Vitals thresholds for **ranking purposes**.
- It allows **developers** to see **real-world performance outcomes** of optimizations.

By providing **real-user data**, CrUX remains a **critical tool** in **web performance measurement and improvement**.
