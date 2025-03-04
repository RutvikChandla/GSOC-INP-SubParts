# Core Web Vitals (CWV) Program

Core Web Vitals (CWV) is Google’s initiative to establish a set of essential metrics that gauge key aspects of user experience on the web. These metrics focus on three crucial aspects of user experience: **loading, interactivity, and visual stability**  
[RUMVISION.COM](https://rumvision.com)

The current primary Core Web Vitals (as of 2024/2025) are:

- **Largest Contentful Paint (LCP)** – for loading performance  
- **Interaction to Next Paint (INP)** – for interactivity (previously **First Input Delay (FID)** was used)  
- **Cumulative Layout Shift (CLS)** – for visual stability  
  [NITROPACK.IO](https://nitropack.io)

Each CWV has defined threshold guidelines (**Good / Needs Improvement / Poor**) to help developers understand performance in a user-centric way.  
For example:
- An **INP under 200ms** is considered *good*  
- An **INP above 500ms** is considered *poor*  

These thresholds align with user expectations of responsiveness.

The **CWV program’s role** is to guide performance improvements and ensure websites meet user experience standards. Google uses these metrics in tools like **PageSpeed Insights, Search Console**, and even as ranking signals in search (encouraging sites to stay within “Good” ranges). 

### **Field Data vs. Lab Data**
A crucial aspect of CWV is the **emphasis on field data** – measurements from real users in the wild, not just synthetic lab tests. Chrome collects these metrics from actual page visits and aggregates them to determine how sites perform for real users. 

This is done by leveraging the **web performance APIs**:
- Chrome captures **LCP times** using the **LCP API** in the user’s browser  
- Chrome captures **input delays** via the **Event Timing API** for **INP**  

These raw measurements are then processed (e.g., taking the **75th percentile value**) to yield the Core Web Vital reported for that page view  
[SPEEDCURVE.COM](https://speedcurve.com)

### **How Data is Collected**
To gather this at scale, Google uses the **Chrome User Experience Report (CrUX)** pipeline.  
In fact:  

> *“The core web vital metrics, including INP, come from real users of your site, via the Chrome User Experience Report”*  
> – [SUPPORT.GOOGLE.COM](https://support.google.com)

That means the CWV program relies on Chrome’s **telemetry** (anonymous performance data from consenting users) to collect metrics like **LCP, INP, CLS** for millions of websites.  

The performance APIs run in each browser session to record timings, and those get reported back (via **URL-keyed aggregated metrics**) to Google’s dataset.

### **Summary**
The CWV program ties together:  

1. **Browser instrumentation** (via performance APIs)  
2. **Field data collection** (via CrUX)  
3. **Metric definitions & thresholds**  
4. **Public reporting & guidance**  

By using standardized **performance APIs** as the data source, CWV ensures that the metrics reflect real user experiences consistently across browsers and platforms. This program has been key in pushing the web ecosystem toward **better performance** by highlighting concrete, measurable targets for developers.
