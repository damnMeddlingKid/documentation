---
title: Notifications
kind: documentation
further_reading:
- link: "monitors/monitor_types"
  tag: "Documentation"
  text: Learn how to create a monitor
- link: "monitors/manage_monitor"
  tag: "Documentation"
  text: Manage your monitors
- link: "monitors/downtimes"
  tag: "Documentation"
  text: Schedule a downtime to mute a monitor
- link: "monitors/faq"
  tag: "FAQ"
  text: Monitors FAQ
---

## Overview

Notifications are a key component of any [monitor](/monitors). You want to make sure the
right people get notified so the problem can be resolved as soon as possible.

{{< img src="monitors/notifications/notification.png" alt="notification" responsive="true" popup="true" >}}

1. Give your monitor a **title**. It is often useful to use a succinct
   explanation of the monitor so a notified team member can quickly understand
   what is going on.

2. Enter a **message** for the monitor. This field allows standard [markdown formatting](http://daringfireball.net/projects/markdown/syntax) as well as Datadog's @-notification syntax.  
  Note: you can notify any non-Datadog users via email by adding `@their-email` to the message. A common use-case for the monitor message is to include a step-by-step way to resolve the problem.  
  For example if you are monitoring a database then you might want to include steps for failing over to a standby node. All in all, you should attempt to give as much context to the monitor as possible.

3. Optionally enable **monitor renotification**. This option is useful to remind your team that a problem is not solved until the monitor is marked as resolved. If enabled, you can configure an escalation message to be sent anytime the monitor renotifies. The original message is included as well.


## Say what's happening
### Message template variables

Message template variables can be used to customize your monitor notifications.
This feature is supported in all monitor types. There are two primary use cases
for template variables: 

1. Displaying a different message depending on the
notification type (e.g. triggered, recovered, no data)
2. Incorporating the triggering scope into the message of multi alerts.

#### Tag variables

When your monitor is a multi alert, instead of having a generic message (and finding the triggering tag scope in the alert query definition), a variable can be used in the message for explicitly identifying the triggering scope.

Here is an example of how you can use template variables for a multi alert:

{{< img src="monitors/notifications/templatevareditor.png" alt="template var editor" responsive="true" popup="true" style="width:80%;">}}

and the corresponding event notification:

{{< img src="monitors/notifications/templatevar.png" alt="template var" responsive="true" popup="true" style="width:80%;">}}

**Note**:  

* The tag template variables available depend on the tag group selected in Step 1 of the monitor editor. The possible options automatically populate at the bottom of the "Use message template variables" help box in Step 3 of the editor.  

* Template variable content is escaped by default. If your variable
contains JSON or code that you would NOT like to be escaped, use triple braces instead of double braces (e.g. `{{{event.text}}}`).

* See a complete list of contextual template variables available to your monitor by clicking the "Use message template variables" link or in the list of suggestions that appears when you type `{{` to begin a template variable name.

* The tag template variables can also be used in the monitor titles (names), but the variables are only populated in the text of Datadog child events (not the parent, which displays an aggregation summary). 

* Some tags identifying your triggering scope are automatically inserted into the title of your multi alert. If your scope is defined by a lot of tags, your alert title may end up being undesirably long.

#### {{comparator}} template variables

The `{{comparator}}` template variable's value is always a relational operator. It corresponds to the relational value selected in the monitor's "Set alert conditions" section:

{{< img src="monitors/notifications/comparator_alert.png" alt="comparator_alert" responsive="true" popup="true" style="width:80%;">}}

For example, when an alert is set to trigger when a value rises "above" 50, the following syntax:
```
  {{value}} {{comparator}} {{threshold}}
```
would yield a notification message like the following:
```
    51.47 > 50
```

### Conditional variables
Use conditional variables to display a different notification content depending of the context of your notification.

The conditional variables available are:

* `{{is_alert}}`, 
* `{{is_alert_recovery}}`,
* `{{is_warning}}`, 
* `{{is_warning_recovery}}`, 
* `{{is_recovery}}`, 
* `{{is_no_data}}`,
* `{{is_match}}`.

These can also be seen in the "Use message template variables" help box in
Step 3 of the monitor editor.

#### `{{is_alert}}` & `{{is_warning}}`

These variables use simple `if-else` logic to display a different message depending on the event type (*warning*, *recovery*, *no data*...)

{{< img src="monitors/notifications/conditionalvars.png" alt="conditional vars" responsive="true" popup="true" style="width:80%;">}}

Here is an example of how you can set it up in the editor:

{{< img src="monitors/notifications/templateconditionaleditor.png" alt="template conditional editor" responsive="true" popup="true" style="width:80%;">}}

The corresponding trigger event notification looks like this:

{{< img src="monitors/notifications/templateconditionaltrigger.png" alt="template conditional trigger" responsive="true" popup="true" style="width:80%;">}}

and the recovery notification:

{{< img src="monitors/notifications/templateconditionalrecover.png" alt="template conditional recover" responsive="true" popup="true" style="width:80%;">}}

#### `{{is_recovery}}` or `{{is_alert_recovery}}` 

* `{{is_recovery}}` triggers and a monitor recovers indifferently either from a **WARNING** state or an **ALERT** state.
* `{{is_alert_recovery}}` triggers when a monitor recovers directly from an **ALERT** state to an **OK** state.
* `{{is_warning_recovery}}` triggers when a monitor recovers from a **WARNING** state to an **OK** state

This means that if the monitor switches from an **ALERT** to a **WARNING** to an **OK** state:

* the `{{is_recovery}}` would trigger
* the `{{is_alert_recovery}}` wouldn't trigger
* the `{{is_warning_recovery}}` would trigger.

The @ notification inside the template variables follows the same rules.

#### `{{is_match}}`

The `{{is_match}}` conditional allows you to match the triggering context to some string to display a different message.  
For example, you might want to notify your db team if a triggering host has `role:db` but notify your app team if the host has `role:app`.

You can use any of the available tag variables in your condition. A match
is made if the comparison string is anywhere in the resolved variable.

The variable uses the following format:
```
{{#is_match "tag_variable" "comparison_string"}}
  This shows if comparison_string is in tag_variable.
{{/is_match}}
```

Here is an example of how you can give a different message depending on the triggering context:

{{< img src="monitors/notifications/scope_match_editor.png" alt="scope match editor" responsive="true" popup="true" style="width:80%;">}}

**Note**: To use the `{{is_match}}` conditional to check if a `tag_variable` is **NOT** empty, append `.name` after your tag name, for instance:  
  
  ```
  {{#is_match "tag_variable.name" ""}}
    This shows if tag_variable is not empty.
  {{/is_match}}
  ```

### Variable availability

We provide a number of different types of monitors and not all variables are available for each type of monitor. Integration monitor variables are largely dependent on the specific integration and monitor configuration.  
  
  
*(Scroll right to see all variable availability)*

{{% table responsive="true" %}}
||[host](/monitors/monitor_types/host)| [metric](/monitors/monitor_types/metric)| [integration](/monitors/monitor_types/integration)| [process](/monitors/monitor_types/process)| [network](/monitors/monitor_types/network)| [custom check](/monitors/monitor_types/custom_check)| [event](/monitors/monitor_types/event)|
| :-------|:-----|:-----|:-------|:-------|:---------|:-------|:------|
| **Conditionals**      |
| `is_alert`            | Y                 | Y                             | Y                                     | Y                                 | Y                                                     | Y                         | Y                 |
| `is_alert_recovery`   |                   | Y                             | Y                                     | Y                                 | Y                                                     | Y                         |                   |
| `is_warning`          |                   | Y                             | Y                                     | Y                                 | Y                                                     | Y                         |                   |
| `is_warning_recovery` |                   | Y                             | Y                                     | Y                                 | Y                                                     | Y                         |                   |
| `is_recovery`         | Y                 | Y                             | Y                                     | Y                                 | Y                                                     | Y                         | Y                 |
| `is_no_data`          | Y                 | Y                             | Y                                     | Y                                 | Y                                                     | Y                         | Y                 |
| `is_match`            | Y                 | Y                             | Y                                     | Y                                 | Y                                                     | Y                         | Y                 |
| **Variables**         |
| `{{value}}`           |                   | Y                             | Y                                     |                                   |                                                       |                           |                   |
| `{{threshold}}`       | Y (cluster)       | Y                             | Y                                     | Y                                 | Y                                                     | Y                         | Y                 |
| `{{warn_threshold}}`  | Y (cluster)       | Y                             | Y                                     | Y                                 | Y                                                     | Y                         |                   |
| `{{ok_threshold}}`    |                   |                               | Y                                     | Y                                 | Y                                                     | Y                         |                   |
| `{{comparator}}`      | Y                 | Y                             | Y                                     | Y                                 | Y                                                     | Y                         | Y                 |
| Additional variables  | Contextual        |                               | Contextual<br/>`{{check_message}}`    | Contextual<br/>`{{process.name}}` | Contextual<br/>`{{url.name}}`<br/>`{{instance.name}}` | `{{check_message}}`       |                   |
{{% /table %}}
<style>
  .tpl-var-table tr td {
    text-align: center;
    border: 1px #9d6ebf solid;
    padding: 5px;
  }
  .tpl-var-table tr td:first-child {
    text-align: right;
  }
</style>

## Advanced notification configuration
### Include links to appropriate dashboards

Many organizations today would like to include additional context to their Alerts. Quick links to relevant dashboards as a part of the Alert has proven to reduce the overall time it takes during the break fix process to reduce time to resolution.

Datadog makes message template variables available to each defined monitor. Using these variables, you can dynamically build a URL that links Datadog Users to an appropriate dashboard using the scope of the monitor.

Here are a few examples of providing links to items like System Dashboards, Integration Dashboards, HostMaps and Managed Monitors pages.  

First example to review is the most common. Let’s say you would like to provide a link to a System Dashboard when a monitor for a specific system metric has exceeded your defined threshold. The message template variable that can be leveraged in this instance would be {{host.name}}. Include the following URL as a part of your Monitor “Say What’s Happening” section:

```
https://app.datadoghq.com/dash/integration/system_overview?tpl_var_scope=host:{{host.name}}
```

As you can see, `{{host.name}}` is replaced with the offending host of the monitor in question.

{{< img src="monitors/notifications/system_dashboard_url.png" alt="system_dashboard_url" responsive="true" popup="true" style="width:70%;" >}}

Find below additional examples of links that could be added to Monitors to provide Datadog Users quick access to common pages leveraged during the break fix and triage process.

* **Hostmaps** - If you would like to include a HostMap to compare metrics with other similar hosts, you can use a link like below to be included in your Monitor:
  ```
  https://app.datadoghq.com/infrastructure/map?fillby=avg%3Acpuutilization&sizeby=avg%3Anometric&filter=cassandra
  ```
  The above link has more customizable options than your standard System Dashboard. Here you have additional variables to define. Most common variables passed into this URL are the following **fillby, sizeby, filter**:

  * `fillby` is defined by adding `fillby:avg:<MetricName>`.  
  * `sizeby` is defined by adding `sizeby:avg:<SecondMetricName>`.
  * `filter` is used to specify a specific integration (i.e. Cassandra, mysql, apache, snmp, etc) by adding `filter=<integration_name>`
  {{< img src="monitors/notifications/hostmap_url.png" alt="hostmap_url" responsive="true" popup="true" style="width:70%;">}}
  The above colors fill the hexagons by `system.cpu.system`, it sizes the hexagons by `system.cpu.stolen` and add a filter to only include Cassandra hosts.   
  

*  **Integration Dashboards**- If you are building Application or Integration specific Monitors, link to that specific Integration Dashboard as well as adding a scope for the host that triggered the monitor.  
  In the example below all that is necessary to populate is the `<integration_name>` section for something like Cassandra, apache, SNMP, etc as well as providing the scope for the offending host:
  ```
  https://app.datadoghq.com/dash/integration/<integration_name>?tpl_var_scope=host:{{host.name}}
  ```
  {{< img src="monitors/notifications/integration_url.png" alt="integration_url" responsive="true" popup="true" style="width:70%;">}}

* **Manage Monitors Page** – To link to a Manage monitors page that displays all of the monitors for the host in question, define a link like below:
  ```
  https://app.datadoghq.com/monitors/manage?q=scope:host:{{host.name}}
  ```
  The above link links to all monitors for this host. You have other options available to further refine the link.  
  For example, if you would only like monitors that are in an Alert State, you can add the following `status:Alert` (other statuses that can be leveraged are WARN, NO%20DATA, OK and MUTED). Below is an example link:
  ```
  https://app.datadoghq.com/monitors/manage?q=scope:host:{{host.name}}&status:Alert
  ```
  If you would like all monitors for a specific application or integration,  add the following query to the URL `q=<integration_name> `:
  ```
  https://app.datadoghq.com/monitors/manage?q=cassandra
  ```

  {{< img src="monitors/notifications/monitor_url.png" alt="monitor_url" responsive="true" popup="true" style="width:70%;">}}

### Slack integration

#### @-mentions in Slack from monitor alert

Wrap the `@username` in `< >` as seen below in your monitors message template to **@ notify** the defined user within slack notifications.  
For example this configuration: 

{{< img src="monitors/notifications/notification_template.png" alt="notification_template" responsive="true" popup="true" style="width:50%;" >}}

Would produce this slack message:

{{< img src="monitors/notifications/notification_slack_preview.png" alt="notification_slack_preview" responsive="true" popup="true" style="width:50%;" >}}

#### Using message template variables to dynamically create @-mentions 

Use message template variables within a monitor message to dynamically build **@-mentions**.  

For example, if the rendered variable is setup as a channel in the Slack integration: 

* `@slack-{{owner.name}}` post a slack DM message directly the owner of this monitor.

* `@slack-{{host.name}}` post a slack message to the #host.name channel in Slack.

[Learn more about how to setup conditional contacts and messages in a single monitor](/monitors/faq/how-do-i-setup-conditional-contacts-and-messages-in-a-single-monitor)

## Further Reading 

{{< partial name="whats-next/whats-next.html" >}}
