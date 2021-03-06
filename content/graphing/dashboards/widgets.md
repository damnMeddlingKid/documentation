---
title: Widgets
kind: documentation
---

Use widgets through the [Screenboard API](/api/#screenboards):
For [create](/api/#create-a-screenboard)/[read](/api/#get-a-screenboard)/[update](/api/#update-a-screenboard) endpoints, the body is one JSON payload describing the Screenboard widgets: 

Base Payload :
```
# Replace the API and/or APP key below
# with the ones for your account

{
  "board_title": "My Board",

  // Options, e.g.: height, width, etc.

  "widgets": [
    // Put a list of widgets here.
  ]
}
```

Each type of widget is described below:

## Timeseries
*Supported on Screenboards and Timeboards*

The Timeseries visualization is great for showing one or more metrics over time. The time window depends on what is selected on the [Timeboard](/graphing/dashboards/timeboard/) or in the graph on a [Screenboard](/graphing/dashboards/screenboard/).  
Timeseries can be displayed as **lines**, **areas**, and **bars**.

{{< img src="graphing/miscellaneous/visualization/references-graphing-timeseries-example.png" alt="Timeseries" responsive="true" popup="true" style="width:80%;">}}

Example of Timeseries widget for the [API](/api/#screenboards)
```
{
  "type": "timeseries",

  // Title
  "title": true,
  "title_size": 16,
  "title_align": "left",
  "title_text": "My Metric (m/s)",

  // Sizing
  "height": 13,
  "width": 47,

  // Positioning
  "y": 28,
  "x": 32,

  // Widget Parameters
  "timeframe": "1h", // Choose from: [1h, 4h, 1d, 2d, 1w]
  "tile_def": {
    "viz": "timeseries",
    "requests": [
      {
         "q": "sum:my.important.metric{*} by {host}"
      }
    ],
    "events": [
      {
        "q": "tags:release"
      }
    ]
  }
}
```

## Query Value
*Supported on Screenboards and Timeboards*

Query values display the current value of a given metric query, with conditional formatting (such as a green/yellow/red background) to convey whether or not the value is in the expected range.  
The value displayed by a query value need not represent an instantaneous measurement.  
The widget can display the latest value reported, or an aggregate computed from all query values across the time window. These visualizations provide a narrow but unambiguous window into your infrastructure.query

{{< img src="graphing/miscellaneous/visualization/references-graphing-queryvalue-example.png" alt="Query value widget" responsive="true" popup="true" style="width:50%;">}}

Example of Query Value widget for the [API](/api/#screenboards)

```
{
  "type": "query_value",

  // Title
  "title": true,
  "title_size": 16,
  "title_text": "Throughput",
  "title_align": "left",

  // Sizing
  "height": 4,
  "width": 14,

  // Positioning
  "y": 21,
  "x": 65,

  // Widget Parameters
  "aggregator": "avg", // Choose from: [avg, sum, min, max]
  "query": "sum:dd.sobotka.throughput.actual{*}",
  "conditional_formats": [
    {
      "color": "white_on_green",
      "invert": false,
      "comparator": ">", // Choose from: [>, >=, <, <=]
      "value": 20000
    }
  ],
  "text_align": "left",
  "precision": 1,
  "timeframe": "5m", // Choose from: [5m, 10m, 1h, 4h, 1d, 2d, 1w]
  "text_size": "auto",
  "unit": "/s" // Give a custom unit or use "auto"
}
```

### What does "Take the X value from the displayed timeframe" ?

{{< img src="graphing/miscellaneous/visualization/query_value_widget.png" alt="query_value_widget" responsive="true" popup="true" style="width:50%;">}}

The Query Value Widget only displays one Value, unlike a timeseries for example, that displays several points. 

Let's say you are on a Timeseries and you are currently displaying the past hour, this button allows you to either display the `avg` / `max` / `min` / `avg` / `sum` / `last value` of ALL points that are rendered during that 1 hour range timeframe - depending on the aggregation chosen above. 

## Heatmap
*Supported on Screenboards and Timeboards*

The Heatmap visualization is great for showing metrics aggregated across many tags, such as *hosts*. The more hosts that have a particular value, the darker that square is.

{{< img src="graphing/miscellaneous/visualization/references-graphing-heatmap-example.png" alt="Heatmap" responsive="true" popup="true" style="width:80%;">}}

## Distribution
*Supported on Screenboards and Timeboards*

The Distribution visualization is another way of showing metrics aggregated across many tags, such as *hosts*. Unlike the Heatmap, Distribution's x-axis is the quantity rather than time. 

{{< img src="graphing/miscellaneous/visualization/references-graphing-distribution-example.png" alt="Distribution" responsive="true" popup="true" style="width:80%;">}}

## Toplist
*Supported on Screenboards and Timeboards*

The Toplist visualization is perfect when you want to see the list of hosts with the most or least of any metric value, such as highest consumers of CPU, hosts with the least disk space, etc.

{{< img src="graphing/miscellaneous/visualization/references-graphing-toplist-example.png" alt="TopList" responsive="true" popup="true" style="width:80%;">}}

## Change
*Supported on Screenboards and Timeboards*

The Change graph shows you the change in a value over the time period chosen.

{{< img src="graphing/miscellaneous/visualization/references-graphing-change-example.png" alt="Change graph" responsive="true" popup="true" style="width:80%;">}}

## Hostmap
*Supported on Screenboards and Timeboards*

The Hostmap graphs any metric for any subset of hosts on the same hostmap visualization available from the main [Infrastructure Hostmap](/graphing/infrastructure/hostmap) menu. 

{{< img src="graphing/miscellaneous/visualization/references-graphing-hostmap-example.png" alt="Hostmap" responsive="true" popup="true">}}

## Free Text
*Supported on Screenboards only*

Free text is a widget that allows you to add headings to your [Screenboard](/graphing/dashboards/screenboard).  

This is commonly used to state the overall purpose of the dashboard.

## Event Timeline
*Supported on Screenboards only*

The event timeline is a widget version of the timeline that appears at the top of the [Event Stream view](https://app.datadoghq.com/event/stream).

{{< img src="graphing/miscellaneous/visualization/references-graphing-eventtimeline-example.png" alt="Timeseries" responsive="true" popup="true" style="width:80%;">}}

## Event Stream
*Supported on Screenboards only*

The event stream is a widget version of the stream of events on the [Event Stream view](/graphing/event_stream/).

{{< img src="graphing/miscellaneous/visualization/references-graphing-eventstream-example.png" alt="Timeseries" responsive="true" popup="true" style="width:70%;">}}

**This widgets displays only the 100 most recent events**

Example of Event Stream widget for the [API](/api/#screenboards)

```
{
    "type": "event_stream"

    // Title
    "title": false,

    // Sizing
    "height": 57,
    "width": 59,

    // Positioning
    "y": 18,
    "x": 84,

    // Widgets Parameters
    "query": "tags:release",
    "timeframe": "1w" // Choose from: [1h, 4h, 1d, 2d, 1w]
}
```

## Image
*Supported on Screenboards only*

Image allows you to embed an image on your dashboard. Images can be PNG, JPG, or even animated GIF files.

Example of Image widget for the [API](/api/#screenboards)

```
{
  "type": "image",

  // Sizing
  "height": 20,
  "width": 32,

  // Positioning
  "y": 7,
  "x": 32

  // Widget Parameters
  "url": "http://path/to/image.jpg" // Note that you shouldn't hotlink to images you don't own.
}
```

## Note
*Supported on Screenboards only*

Note widget is similar to Free Text widget, but allows for more formatting options: 

* An arrow can be added to the text box that appears on the dashboard. This is commonly used to document the structure of the dashboard.  

* Use `href` to create internal links in Datadog. 
  {{< img src="graphing/dashboards/widgets/using_link_note_widget.gif" alt="Using links in note widget" responsive="true" popup="true">}}

Example of Note widget for the [API](/api/#screenboards)

```
{
    "type": "note",

    // Title
    "title": false,

    // Sizing
    "height": 15,
    "width": 30,

    // Positioning
    "x": 1,
    "y": 28,

    // Widget Parameters
    "html": "Put your note text in here.",
    "text_align": "left",
    "font_size": 14,
    "bgcolor": "yellow", // Choose from: [yellow, blue, pink, gray, white, red, green]
    "tick": true,
    "tick_pos": "50%",
    "tick_edge": "right" // Choose from: [top, right, bottom, left]
    "auto_refresh": true // Defaults to false
}
```

## Alert Graph
*Supported on Screenboards only*

Alert graphs are time series graphs showing the current status of any monitor defined on your system.

Example of Alert Graph widget for the [API](/api/#screenboards)

```
{
   "title_size": 16,
   "viz_type": "timeseries",
   "title": true,
   "title_align": "left",
   "title_text": "u",
   "height": 13,
   "width": 47,
   "alert_id": "<ENTER ID>",
   "timeframe": "1mo",
   "y": 2,
   "x": 6,
   "isShared": false,
   "type": "alert_graph",
   "add_timeframe": false
}
```

## Alert Value
*Supported on Screenboards only*

Alert values are query values showing the current value of the metric in any monitor defined on your system.
## Iframe
*Supported on Screenboards only*

The Iframe widget allows you to embed a portion of any other web page on your dashboard.

## Check Status
*Supported on Screenboards only*

Check status shows the current status or number of results for any check performed.
## Service Summary
*Supported on Screenboards only*

The service summary displays the top portion of any APM trace in your Screenboard.

{{< img src="graphing/miscellaneous/visualization/references-graphing-servicesummary-example.png" alt="Timeseries" responsive="true" popup="true" style="width:80%;">}}

## Monitor Summary
*Supported on Screenboards only*

Monitor summary is a summary view of all monitors on your system, or a subset based on a query.

{{< img src="graphing/miscellaneous/visualization/references-graphing-monitorsummary-example.png" alt="Timeseries" responsive="true" popup="true" style="width:80%;">}}

