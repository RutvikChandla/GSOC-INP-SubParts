# Last Year’s Work on LCP

Last year, significant improvements were made to the **Largest Contentful Paint (LCP)** metric’s implementation in **Chrome**. These changes laid the groundwork for the kind of detailed instrumentation we are now pursuing with **INP**. In essence, **LCP was broken down into “sub-parts”**, similar to how we plan to break down INP. By understanding **LCP’s subcomponents**, developers can better pinpoint which part of the page load is slow.

The **four LCP sub-parts** that were identified and instrumented are:

- **Time to First Byte (TTFB)** – The network latency from the user’s navigation start until the first byte of the HTML response. This measures **server responsiveness** and **network latency**.
- **Resource Load Delay** – The gap between receiving HTML and starting to load the LCP resource.  
  - For example, if the **LCP image URL** is discovered only after parsing some HTML/JS, that delay counts here ([GitHub](https://github.com)).  
  - Essentially, this is **idle time** when the LCP resource is **not yet being fetched** (often due to late discovery or prioritization issues).
- **Resource Load Time** – The time spent actually downloading the **LCP resource** (e.g., image file), including **network transfer** and **any download blocking**.  
  - This plus the previous parts covers **getting the bytes** of the **LCP content**.
- **Element Render Delay** – The time from when the **resource is fully loaded** until it is **displayed (painted)** as the largest content element ([GitHub](https://github.com)).  
  - This can include **time to decode an image** and **time spent by the browser to render it** on screen (**layout, paint, composite**).

Together, these cover the entire **timeline of LCP**, from the **initial request** up to **rendering the largest element**.

## Instrumentation in Chrome

Last year’s project added **instrumentation** in Chrome to measure each of these components for **every page**. Chrome’s **Performance API** and **internal metrics** were enhanced so that:

> _“Every single page can have its LCP value broken down into these four sub-parts… collectively they add up to the full LCP time.”_ ([GitHub](https://github.com))

This meant **developers (and Chrome engineers)** could see, for a **slow LCP**, how much time was:

- **Spent waiting for network** vs.
- **Blocking on resource discovery** vs.
- **Rendering the element** on screen.

Such **insight was not possible** with the **raw LCP number alone**.

## Impact and Future Implications

These improvements not only helped **developers optimize LCP** more effectively but also **established patterns for instrumenting complex metrics**. Specifically, the **Chrome team** built out the **pipeline** for collecting **sub-metrics** in the field:

1. Capturing data in the **renderer process**.
2. Sending it via **Mojo** to the browser.
3. Recording multiple values (**one for each sub-part**) in **UKM**.
4. Eventually exposing it (initially in **internal dashboards**, via the **Web Vitals extension**, and potentially in **CrUX**).

The **success with LCP sub-parts** demonstrated that **breaking a metric into components** is **feasible and valuable**. It:

- _“Set the stage for INP subparts”_ by providing a **template** on how to do it.
- Provided both **technical plumbing** and **understanding of the benefits**.

For instance, just as **LCP’s subparts** revealed whether **slow server response** or **late lazy-loading** was the culprit, **INP subparts** will reveal:

- Whether a **poor INP** is due to **input delay**.
- Whether **slow JavaScript execution** is the issue.
- Whether a **rendering delay after input** is causing problems.

## Conclusion

In summary, last year’s **LCP work** provided both the **inspiration** and the **implementation blueprint** for tackling **INP** in a **similar way**.
