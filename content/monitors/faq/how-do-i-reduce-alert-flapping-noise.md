---
title: How do I reduce alert flapping / noise?
kind: faq
further_reading:
- link: "monitors/faq/how-do-i-setup-conditional-contacts-and-messages-in-a-single-monitor"
  tag: "FAQ"
  text: How do I setup conditional contacts and messages in a single monitor?
- link: "monitors/faq/how-do-i-add-custom-template-variables-to-my-monitor-message"
  tag: "FAQ"
  text: How do I add custom template variables to my monitor message?
- link: "monitors/downtimes"
  tag: "Documentation"
  text: Schedule a downtime to mute a monitor
---

We often discuss alerting with our clients and a frequent issue or pain point is alert fatigue, or when alerts ‘flap’ (rapidly switching from an ‘ok’ to an ‘alert’ status).  

Your individual Datadog alerts with groups will [have notification](https://www.datadoghq.com/blog/alert-rollup/) rollups on by default, but there is functionality within Datadog that often leads to less noisy, more meaningful alerts.

* Re-Evaluate the Alert Threshold Value
    * The easiest way to reduce flapping when the alert <-> ok or state changes are frequent could be to increase/decrease the threshold condition.
* Utilize the ‘At all times’ threshold
    * This triggers the alert only when all data points for the metric in the timeframe violate the threshold

* Reframe the query using Functions- rates, moving averages, or time-shift differentials
    * This means, you can compare the difference between a metric stream’s values with the values from a week ago and set alert conditions based off the difference
    * A time-shift differential allows you to combine functions and can give a historical view as well. For example:
 abs(system.cpu.system{*} - week_before(system.cpu.system{*}))
    * If your metric frequently spikes, and those spikes are not inherently indicative of issues, applying a rate or average to it will allow you to set a more meaningful threshold.

* Consider the states of other monitors using Composite alerts
    * The most recent addition to Datadog’s alerting capabilities, composite alerts will allow you to combine two or more previously created alerts.
    For example: if CPU is high AND disk is high on a host, trigger the alert.

* Use some built-in analysis modules with Anomaly or Outlier
    * Anomaly Detection utilizes some seasonality analysis to issue an alert when a data stream behaves in a historically inconsistent way.
    * Outlier Detection utilizes other data streams of the same context to issue an alert when a stream behaves in a way different compared with its peers
    * Both can also be used in conjunction with Composite alerts.
    * If you would like a visual introduction to anomaly and outlier detection, this gist is a [screenboard](/graphing/dashboards/screenboard) post to the Datadog API with examples and documentation links for both.


If the issue is alert routing, template variables and the separation of **warning** or **alert** states will be of interest!

{{< partial name="whats-next/whats-next.html" >}}
