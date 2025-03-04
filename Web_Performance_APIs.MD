# Web Performance APIs

Web performance APIs are a collection of browser interfaces that allow developers (and the browser itself) to measure various aspects of page performance. These APIs include standardized timing mechanisms to capture key events and durations during page load and user interactions. The **Performance API** (also called the **Performance Timeline**) provides a unified way to access these metrics in JavaScript. It acts as a comprehensive toolkit for reporting metrics like page load times and Core Web Vitals in real time ([Smashing Magazine](https://www.smashingmagazine.com)).

## Major Web Performance APIs

- **Navigation Timing API** – Measures high-level page load milestones (e.g., redirect time, DNS lookup, DOM content loaded, `onload`, etc.), giving insight into page navigation and resource loading timeline.
- **Resource Timing API** – Provides timing details for network requests (images, scripts, CSS, etc.), helping analyze how each resource contributes to load time.
- **Paint Timing API** – Exposes paint events like **First Paint** and **First Contentful Paint**, indicating when the browser first renders pixels to the screen ([W3.org](https://www.w3.org)).
- **Largest Contentful Paint (LCP) API** – Reports the render timestamp of the largest element in the viewport, used to measure loading performance.
- **Cumulative Layout Shift (CLS) API** – Measures layout instability by tracking layout “shift” events, used for visual stability metrics.
- **Event Timing API** – Provides timestamps and durations for user input events (clicks, key presses, taps), which is crucial for measuring interactivity ([Mozilla Developer Network](https://developer.mozilla.org)).  
  - For example, it lets you measure the delay before an event handler runs and how long the handler takes.
  - This API is the foundation for metrics like **First Input Delay (FID)** and **Interaction to Next Paint (INP)** in the field.
- **Long Tasks API** – Reports tasks on the main thread that run longer than **50ms**, which can block user input. This helps identify scripts that cause jank or input delays.
- **PerformanceObserver** – A JavaScript interface to asynchronously collect performance entries (for any of the above types) as they occur, rather than polling after the fact ([Dev.to](https://dev.to)).

## How These APIs Relate to Core Web Vitals

The **Core Web Vitals** metrics are defined in terms of these low-level performance entries. For instance:

- **LCP** uses the **Largest Contentful Paint** entry from the Performance API.
- **CLS** uses **Layout Shift** entries.
- **INP** is computed from **Event Timing** entries.

In **real-user monitoring (RUM)**, scripts (or the browser itself) can observe these performance events via **PerformanceObserver** to gather data. In **Chromium’s case**, the browser internally uses these same mechanisms to capture metrics for each page load and interaction.

Essentially, the performance APIs provide the measurement data, and the **Core Web Vitals logic** interprets that data (applying certain aggregation or algorithms) to produce a single **user-centric metric** value. This means that what developers can measure in the browser via the **Performance API** is directly what **Chrome** itself relies on to report metrics.

For example, Chrome will track a **PerformanceEventTiming** for each user interaction; the timing properties (**startTime**, **processingStart**, **processingEnd**, and **duration**) from those entries are what feed into calculating **INP**.

### Summary

Web performance APIs are the **building blocks** that enable both developers and browsers to **quantify performance** in a standardized way, forming the basis of metrics that reflect **real-user experience**.
