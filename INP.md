# Technical Breakdown of the INP Project

The Interaction to Next Paint (**INP**) “subparts” project aims to granularly measure the different phases of user interaction latency. Instead of treating **INP** as a single opaque number, we want to break it down into its constituent parts to better understand where time is being spent. Conceptually, an interaction’s latency has three components: **Input delay, Processing time, and Presentation delay**  
[SPEEDCURVE.COM](https://www.speedcurve.com)  
[SPEEDCURVE.COM](https://www.speedcurve.com). The goal of the project is to instrument Chrome to capture all three for each interaction and report them for analysis.

## What INP Subparts Measure and Why They Matter

- **Input Delay** is the time from the user’s action (e.g. tapping a button or pressing a key) until the moment the event handling actually begins on the main thread  
  [SPEEDCURVE.COM](https://www.speedcurve.com). In other words, how long did the event sit in a queue waiting? This usually reflects main-thread blockage by other tasks at the time of the interaction. A long input delay means the page was unresponsive initially – the browser couldn’t even begin processing the input promptly.

- **Processing Time** (sometimes called event processing duration) is the time spent executing all the event handlers associated with the interaction  
  [ALMANAC.HTTPARCHIVE.ORG](https://httparchive.org). This covers the JavaScript callback functions and any related work triggered by them (up to the point the event is fully handled). If this is large, it means the interaction’s code was slow (e.g. a heavy onclick handler or other JS work).

- **Presentation Delay** is the time from when event processing completes to when the next frame is painted on screen with the results of that interaction  
  [ALMANAC.HTTPARCHIVE.ORG](https://httparchive.org). Even after handlers finish, the browser may need to do layout, style recalculation, painting and compositing – especially if the interaction caused DOM updates or visual changes. This delay captures that rendering pipeline time before the user actually sees a response.

These subparts are important because they pinpoint where an interaction is slow. A poor overall **INP** (say 600 ms) could be caused by very different issues: maybe input was delayed 400 ms due to a long task clogging the thread, or maybe input started quickly but the handler itself took 500 ms of computation, or perhaps the handler was quick but triggered an expensive layout taking 500 ms to render. The user’s experience is the same (a 600 ms wait to see something happen), but the solution is completely different in each case. By measuring each subpart, Chrome can provide developers with actionable insight: e.g., “Your worst interactions mostly spend time in input delay – likely the main thread was busy, consider breaking up long tasks.” or “Your interactions are bottlenecked by presentation delay – perhaps large reflows or graphics rendering are slow.” In short, **INP subparts** diagnose responsiveness problems. As the SpeedCurve team put it, after identifying a slow interaction, “you'll want to understand where that time is being spent. This is provided with INP subparts.”  
[SPEEDCURVE.COM](https://www.speedcurve.com) Knowing the breakdown helps target the right optimization (yielding a better user experience). It also allows Chrome’s metrics pipeline to track improvements in each area separately.

## Using the Event Timing API for INP Latency Breakdown

Under the hood, the **Event Timing API** is the primary means by which we capture these interaction timings. Every relevant user event in Chrome (click, key, tap, etc.) produces a **PerformanceEventTiming** entry. This entry includes timestamps that correspond to the phases of the event. Notably, it provides: 

- **startTime** (when the event was first received)
- **processingStart** (when the event listener actually began executing)
- **processingEnd** (when the event listener finished)
- **duration** (which is defined as time from startTime to the next paint after the event)  
  [DEVELOPER.MOZILLA.ORG](https://developer.mozilla.org)  
  [DEVELOPER.MOZILLA.ORG](https://developer.mozilla.org).

Using these, we can derive the subparts:

- **Input delay** = processingStart - startTime (time from user input to handler start).
- **Processing time** = processingEnd - processingStart (time spent running the handler code)  
  [DEVELOPER.MOZILLA.ORG](https://developer.mozilla.org)  
  [DEVELOPER.MOZILLA.ORG](https://developer.mozilla.org).

- **Presentation delay** = (next paint time) - processingEnd. Since duration on the PerformanceEventTiming is effectively (nextPaintTime - startTime)  
  [DEVELOPER.MOZILLA.ORG](https://developer.mozilla.org), another way to get presentation delay is: duration - (processingEnd - startTime).

In the browser’s implementation, when an interaction occurs, the Event Timing API machinery records these times. For **INP**, we care about the longest interaction on the page (at least up to the point of reporting). Chrome’s **INP algorithm** observes all **EventTiming** entries and selects the worst one (with some outlier filtering)  
[WEB.DEV](https://web.dev). To break that interaction down, we must capture its three components as outlined above.

So effectively, the **Event Timing API** is the source of truth for **INP subparts**. The project will use the data it provides to compute and surface the values. For example, if a button click is delayed because the main thread was busy, the **PerformanceEventTiming** entry for that click might have **startTime = 5000.0 ms, processingStart = 5230.0 ms, processingEnd = 5245.0 ms, and duration = 5310.0 ms**. From this, we calculate: **Input delay ≈ 230 ms, Processing ≈ 15 ms, Presentation ≈ 65 ms**. Chrome would log these three numbers. (These calculations can be done in JavaScript as well – MDN’s example shows logging an event’s delay and handler time from a **PerformanceObserver**  
  [DEVELOPER.MOZILLA.ORG](https://developer.mozilla.org) – but here we are doing it inside the browser for aggregation.)

One complexity is that interactions can comprise multiple events (e.g. a mousedown and mouseup together). The **Event Timing API** assigns an **interactionId** to related events to group them  
[DEVELOPER.MOZILLA.ORG](https://developer.mozilla.org). Chrome’s **INP logic** uses this to consider an interaction complete (for **INP**, usually the end of the final event’s processing and paint). Our instrumentation will ensure we attribute the timing of the whole interaction (which might include a couple of events) to a single set of subparts. The **Event Timing API** is critical for this attribution as well. Overall, without the **Event Timing API**, we wouldn’t have an easy, standardized way to get these timestamps. It was designed exactly for measuring input latency and is “particularly useful for measuring **INP**”  
[DEVELOPER.MOZILLA.ORG](https://developer.mozilla.org). This project essentially takes the granular data from that API and integrates it into Chrome’s internal metrics and reports.

## Moving Data from the Renderer to the Browser (Mojo IPC)

In Chrome’s architecture, web page events are handled in the **renderer process**, whereas metrics logging (UKM, etc.) happens in the **browser process**. This means that the detailed timing info we gather (which lives in the renderer side where the page is running) must be sent to the browser process to be recorded. The mechanism for this is **Mojo IPC** – Chrome’s inter-process communication framework. 

> “Mojo is Chromium’s IPC framework, providing a language-agnostic way of defining interfaces and data structures that can be marshalled across process boundaries.”  
> [CHROMIUM.ORG](https://www.chromium.org)

In simpler terms, Mojo lets us send a structured message from the renderer to the browser.

For this project, we will extend the existing **PageLoadMetrics** Mojo interface (which already carries other metrics like **LCP, CLS, etc.** from renderer to browser) to include the **INP subpart data**. Concretely, when an interaction occurs, the renderer’s **PageLoadMetrics** code can call something like `RecordInteractionLatency(input_delay, processing_time, presentation_delay)` over Mojo to the browser. Mojo will serialize these numbers (along with an identifier for which page load and perhaps which interaction ID) and deliver them to the browser process.

Using Mojo ensures that the data transfer is efficient and typesafe. The Mojo interface definition will likely be updated to carry three new fields for the **INP breakdown**. This separation (renderer vs browser) is important: we don’t want to log metrics directly from the renderer for security/privacy reasons, and the browser can coordinate all logging. So, the renderer gathers the raw data using Blink (the rendering engine) and Event Timing, then packages it up via Mojo IPC. The browser process receives the message and updates the page’s metrics. This design also allows the browser to decide when to finalize and report the **INP metrics** (for example, at the end of the page or after a certain delay).

From an implementation standpoint, moving data with Mojo involves adding entries to the `page_load_metrics.mojom` file for the new data, and updating the handler in the browser (**PageLoadMetricsConsumer**) to handle the incoming data. The Mojo IPC layer will take care of the rest. By the end, the browser process will have the interaction’s input delay, processing, and presentation times ready to be logged as telemetry.

## Field Metric Reporting with UKM (URL-Keyed Metrics)

Once the browser process has the **INP subparts data**, the next step is to record it in Chrome’s telemetry system so it can be aggregated across many users. Chrome uses **UKM (URL-Keyed Metrics)** for logging most Core Web Vitals data from real users. UKM is a system that logs metrics associated with page URLs in a pseudonymous, aggregated way. (In contrast to UMA histograms which are not keyed by URL.) In field usage, “UKM stands for URL-Keyed Metrics, which is a larger set of field metrics that Chromium-based browsers can collect.”  
[DEBUGBEAR.COM](https://www.debugbear.com) It’s the pipeline through which data flows to Google’s servers (and eventually into **CrUX**).

For this project, we will define new UKM events/metrics for the **INP subparts**. Likely an event named something like **InteractiveTiming** or extending the existing **PageLoad** event, with metrics such as `InteractionLatency.InputDelay`, `InteractionLatency.ProcessingTime`, `InteractionLatency.PresentationDelay`. These definitions go into Chrome’s `ukm.xml` configuration  
[CHROMIUM.GOOGLESOURCE.COM](https://chromium.googlesource.com)  
[CHROMIUM.GOOGLESOURCE.COM](https://chromium.googlesource.com), including proper units (milliseconds) and descriptions. Each metric will record the duration of that phase for the worst interaction of a page.

When a page is finished (or when the metric is ready to be reported), Chrome will create a UKM entry. This involves getting a **SourceId** for the page (which ties the data to the page’s URL/origin, without revealing the URL in plaintext)  
[CHROMIUM.GOOGLESOURCE.COM](https://chromium.googlesource.com), then logging an event with the three values. In code, this is done via the `ukm::UkmRecorder` interface in the browser process  
[CHROMIUM.GOOGLESOURCE.COM](https://chromium.googlesource.com). The **PageLoadMetrics** component will take the **INP subpart times** and call the generated UKM event recorder (e.g., `ukm::builders::InteractiveTiming(sourceId).SetInputDelay(input_ms)...Record(ukm_recorder)`) – this is a typical pattern.

Once logged, these UKM events are buffered and periodically sent up to the Chrome backend. On Google’s side, they are collected (with user privacy in mind – e.g., requiring a minimum number of samples before being used). Eventually, aggregated statistics can be derived from them. The field data pipeline thus goes: **Renderer -> Mojo -> Browser -> UKM log -> Google servers -> Aggregation -> CrUX**. By instrumenting UKM, we ensure the new **INP subparts** will be available in field metrics analysis. This is crucial, because the ultimate aim is to see these numbers in datasets like **CrUX** or at least in Chrome’s internal analysis to validate the feature.

One important aspect is that UKM data is sampled. Not every single page view’s metrics are sent; Chrome may sample at e.g. 1% for certain metrics. We will need to verify the sampling settings for the new metrics (these can be configured in `ukm.xml` or server-side). Also, UKM has constraints – for example, it won’t log if the user is in an incognito session, and it only sends up data when certain privacy criteria are met. As developers, we define the metrics and log them, and the existing UKM infrastructure handles the rest.

In summary, the **INP subparts** will be reported as three numeric fields in Chrome’s URL-keyed telemetry. This allows Chrome (and by extension web developers, via **CrUX** or the **Web Vitals extension**) to see not just a single **INP value**, but a breakdown of that value for real page visits. Field metric reporting via **UKM** is what turns our instrumentation into actionable data at scale.

## Integrating INP Subparts into CrUX (Experimental Data)

After the metrics are logged via UKM, the next step is to integrate them into the **Chrome User Experience Report (CrUX)** pipeline. Initially, because this is a new metric breakdown, it will likely appear as experimental data. Google often introduces new metrics in CrUX on an experimental basis before fully launching them. For example, **INP** itself was available as an experimental metric in CrUX (when it was still evolving).

The steps to integrate are roughly:

1. **Ensure UKM collection is active** – Once the Chrome code is shipping (even behind a flag or in Canary/Dev channels), start collecting data from real users. Possibly run an origin trial or field trial if needed to gather data from a subset of users first. Since **INP subparts** don’t affect user-facing behavior and are privacy-safe (they're just durations), Google might even enable logging in stable builds relatively quickly (hidden behind the scenes). The data would start flowing into the UKM backend.

2. **Add the metric to CrUX processing** – The CrUX team needs to incorporate these new UKM fields into their data aggregation. CrUX data is based on the open-source UKM definitions  
   [GROUPS.GOOGLE.COM](https://groups.google.com). This means if Chrome starts sending a new UKM event and it’s whitelisted for CrUX, it can be aggregated. Typically, experimental metrics show up in BigQuery and the API under an `experimental.` prefix or only for specific collections. We might coordinate with the CrUX team (or submit the request) to include “INP subparts” in the next data release, likely behind a flag that only surfaces to those who know where to look.

3. **Gather enough sample size** – CrUX usually requires a certain volume of data (and distinct users) for metrics to be reported for an origin. As the data comes in, initially it might be only from Chrome Canary or a small percent of users, so not all sites will have it. Over time, as it rolls out, more origins will have measurable values. During the experimental phase, Google might publish some aggregate findings (for example, in a blog or case studies) using the data to validate its usefulness.

4. **Feedback and iterate** – We will use the experimental CrUX data to see how the **INP subparts** behave in the wild. Are the distributions reasonable? Are all three subparts contributing significantly or is one always negligible? This could inform further tuning. For example, if we notice that presentation delay is consistently a major chunk for many sites, that might prompt emphasis on optimizing the rendering pipeline.

From a contributor’s perspective, integrating into **CrUX** doesn’t require code changes to Chrome beyond the UKM logging. It’s more about communication: ensuring that those who maintain CrUX know about the new metrics. Often, metrics like these remain “dark launched” for a while – collected but not publicly reported – until Google is confident in them. Eventually, if all goes well, **INP subpart data** could be exposed as part of the **CrUX public dataset** (perhaps in a future schema update, or via the CrUX API experimental fields). This means developers around the world could query something like “the 75th percentile input delay for my site last month”. That’s the endgame.

In short, integrating into **CrUX** involves turning on the tap of data via **UKM**, coordinating with the CrUX pipeline to aggregate it, and analyzing the experimental results. It’s an essential step to validate the feature at scale and to ultimately deliver the insights to developers via tools like **PageSpeed Insights**. We want to make sure the data is not only collected but also actionable in the real world.

## Writing Tests for New INP Instrumentation

As with any new feature in Chromium, especially one involving metrics, writing tests is a critical part of the development. We need to ensure that our **INP subparts** collection works correctly and doesn’t regress over time. There are a few levels of testing we’ll employ:

- **Unit tests:**  
  We can write unit tests for any helper functions or algorithm logic (for example, if there’s a function that selects the worst interaction or calculates the presentation delay from timestamps). These would run as part of the standard test suites and can verify that given synthetic timing data, the computed values match expected results.

- **Browser integration tests:**  
  More importantly, we should add integration tests (e.g., a browser test or web test in Chromium) that simulate actual page interactions and check that the metrics are recorded. Chromium has a framework where we can create a test page (with known behavior), automate an interaction, and then inspect the **PageLoadMetrics** output. For example, we could create a test page that, on a button click, deliberately performs a long task (to simulate a slow handler). In the test, we trigger a click (using DevTools automation or a script), then at the end of the page, retrieve the logged metrics. We might have instrumentation to expose the values (perhaps via a JavaScript hook or checking UKM entries in a testing mode). The test should assert that: the input delay was roughly the length of the intentional block, the processing time matches the handler’s duration, etc. This ensures our plumbing from **Event Timing -> Mojo -> UKM** is working end-to-end.

- **Mojo message tests:**  
  We can also test the Mojo interface at a lower level – for instance, using a fake Mojo receiver in a unit test to ensure that when an interaction happens, the renderer sends the expected message with correct values. Chromium’s Mojo framework allows writing fake message pipes for testing  
  [CHROMIUM.ORG](https://www.chromium.org).

- **UKM logging tests:**  
  Chrome has a **TestUkmRecorder** that can capture UKM events in tests  
  [CHROMIUM.GOOGLESOURCE.COM](https://chromium.googlesource.com). We can utilize that to verify that our **InteractionLatency** (or whatever it’s called) UKM event is actually being recorded with the right values in a test scenario. For example, after simulating an interaction in a browser test, query the TestUkmRecorder for the latest events and check that our event appears and carries the three metrics with plausible values.

Writing these tests is important not only to catch bugs now but also to guard against future changes. Since this code touches multiple components (Blink, Mojo, Browser, Metrics), it’s easy for a refactor in one area to break the chain. Good tests will flag if, say, an **Event Timing** entry we expect is not being generated, or if the Mojo message isn’t arriving.

Additionally, performance metrics can have edge cases – e.g., what if no interaction occurs? (We should then not log anything or log zeros appropriately.) Or what if an interaction happens during page unload? Tests can help verify those corner cases (Chromium’s testing allows simulating things like navigation away).

In summary, we will create a suite of tests to verify the **INP subparts** feature from end to end. This gives confidence in the implementation’s correctness. It’s particularly important because these metrics, once in the field, become part of Google’s public guidance; they must be accurate. Also, by having tests, new contributors or reviewers can better understand the expected behavior (the tests serve as documentation of the intended outputs). Ensuring robust tests for the instrumentation is a best practice so that this new performance measurement is reliable from day one and remains so in future Chrome releases.
