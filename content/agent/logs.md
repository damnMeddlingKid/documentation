---
title: Log Parsing in the Agent
kind: documentation
aliases:
    - /guides/logs/
further_reading:
- link: "/agent/faq/want-more-flexibility-with-your-custom-log-parser-add-dogstatsd"
  tag: "FAQ"
  text: Want more flexibility with your custom log parser? Add DogStatsD  
---

Log files contain tons of valuable application and business data.
Unfortunately, this value is oftentimes never realized because log files go
ignored. The Datadog Agent can help remedy this by parsing metrics and events from
logs, so the data within can be graphed in real-time, all the time.

## Parsing Metrics

The Datadog Agent can read metrics directly from your log files:

- from the Datadog canonical log format, without any additional programming
- from any other log format, with a customized log parsing function

### Datadog Canonical Log Format

Datadog logs are formatted as follows:

    metric unix_timestamp value [attribute1=v1 attributes2=v2 ...]

For example, imagining the content of `/var/log/web.log` to be:

    me.web.requests 1320786966 157 metric_type=counter unit=request
    me.web.latency 1320786966 250 metric_type=gauge unit=ms

Then all you need for Datadog to read metrics is to add this line to your Agent
configuration file (usually at `/etc/dd-agent/datadog.conf`):

    dogstreams: /var/log/web.log

You can also specify multiple log files like this:

    dogstreams: /var/log/web.log, /var/log/db.log, /var/log/cache.log

### Parsing Custom Log Formats

If you want to parse a different log format - say for a piece of vendor
or legacy software - you can use a custom Python function to extract the proper
fields from the log by specifying your log file in your Agent configuration
file in the following format:

    dogstreams: /var/log/web.log:parsers:parse_web

The `parsers:parse_web` portion indicates that the custom Python function lives
in a package called  parsers in the Agent's `PYTHONPATH`, and the parsers package
has a function named `parse_web`. The Agent's `PYTHONPATH` is set in the Agent
startup script, `/etc/init.d/datadog-agent` for Agent versions < 2.0, and in the
supervisor config for Agent version >= 2.0.

If your parser does **not** live on the Agent's `PYTHONPATH`, you can use an
alternative syntax to configure your line parser:

    dogstreams: /path/to/log1:/path/to/my/parsers_module.py:custom_parser

In this format, the agent attempts to import a function called
`custom_parser` from `/path/to/my/parsers_module.py`.

If your custom log parser is not working, the first place to check is the
Agent collector logs: 

* If the Agent is unable to import your function, there is a line with `Could not load Dogstream line parser`. 

* If all goes well you should see `dogstream: parsing {filename} with
{function name} (requested {config option text})`.

<div class="alert alert-warning">
To test that dogstreams are working, append a line—don't edit an existing one—to any log file you've configured the Agent to watch. The Agent only tails the end of each log file, so it won't notice any changes you make elsewhere in the file.
</div>

### Writing Parsing Functions

Custom parsing functions must:

- take two parameters: a Python logger object (for debugging) and a string parameter of the current line to parse.
- return a tuple or list of tuples of the form:

     `(metric (str), timestamp (unix timestamp), value (float), attributes (dict))`

    Where attributes should at least contain the key metric_type, specifying whether the given metric is a counter or gauge.

    If the line doesn't match, instead return `None`.

### Example for Metrics Collecting

Let's imagine that you're collecting metrics from logs that are not canonically formatted, but which are intelligently delimited by a unique character, logged as the following:

```
user.crashes|2016-05-28 20:24:43.463930|24|LotusNotes,Outlook,Explorer
```

We could set up a log-parser like the following to collect a metric from this logged data in our Datadog account:

```python

import time
from datetime import datetime
...
def my_log_parser(logger, test):
    metric_name, date, metric_value, extras = line.split('|')
    # Convert the iso8601 date into a unix timestamp, assuming the timestamp
    # string is in the same timezone as the machine that's parsing it.
    date = datetime.strptime(date, "%Y-%m-%d %H:%M:%S.%f")
    tags = extras.split(',')
    date = time.mktime(date.timetuple())
    metric_attributes = {
        'tags': tags,
        'metric_type': 'gauge',
    }
    return (metric_name, date, metric_value, metric_attributes)
```

And then we would configure our `datadog.conf` to include the dogstream option as follows:
```
dogstreams: /path/to/mylogfile.log:/path/to/mylogparser.py:my_log_parser
# (N.B., Windows users should replace each "/" with the escaped "\\")
```

This example would collect a gauge-type metric called "user.crashes" with a value of 24, and tagged with the 3 applications named at the end.

A word of warning: there is a limit to how many times the same metric can be collected in the same log-pass; effectively the agent starts to over-write logged metrics with the subsequent submissions of the same metric, even if they have different attributes (like tags). This can be somewhat mitigated if the metrics collected from the logs have sufficiently different time-stamps, but it is generally recommended to only submit one metric to the logs for collection once every 10 seconds or so. This over-writing is not an issue for metrics collected with differing names.

## Parsing Events

Event parsing is done via the same custom parsing functions as described above, except if you return a
`dict` (or a `list` of `dict`) from your custom parsing function, Datadog treats it as an event instead of a metric.

Here are the event fields (bold means the field is required):

| Field | Type | Value |
| --- | --- | --- |
| **msg_title** | string | Title of the event, gets indexed by our full-text search. |
| **timestamp** | integer | Unix epoch timestamp. If omitted, it defaults to the time that the Agent parsed the event. |
| **msg_text** | string | Body of the event, get indexed by our full-text search. |
| alert_type | string enum | Indicates the severity of the event. Must be one of `error`, `warning`, `success` or `info`. If omitted, it defaults to `info`. Searchable by `alert_type:value` |
| event_type | string | Describes what kind of event this is. Used as part of the aggregation key |
| aggregation_key | string | Describes what this event affected, if anything. Used as part of the aggregation key |
| host | string | Name of the host this event originated from. The event automatically gets tagged with any tags you've given this host using the [tagging page](https://app.datadoghq.com/infrastructure#tags) or the [tagging api](/api/#tags). The host value is used as part of the aggregation key. |
| **priority** | string | Determines whether the event is visible or hidden by default in the stream; Must be one of `low` or `normal` |

The events with the same aggregation key within a 24 hour time window gets aggregated together on the stream.
The aggregation key is a combination of the following fields:

- event_type
- aggregation_key
- host

For an example of an event parser, see our [Cassandra compaction event parser](https://github.com/DataDog/dd-agent/blob/master/dogstream/cassandra.py) that is bundled with the Agent.

### Example for Events Collecting

Let's imagine that you want to collect events from logging where you have enough control to add all sorts of relevant information, intelligently delimited by a unique character, logged as the following:

```
2016-05-28 18:35:31.164705|Crash_Report|Windows95|A terrible crash happened!|A crash was reported on Joe M's computer|LotusNotes,Outlook,InternetExplorer
```

We could set up a log parser like the following to create an event from this logged data in our Datadog [event stream](/graphing/event_stream/):

```python

import time
from datetime import datetime
...
def my_log_parser(logger, line):

    # Split the line into fields
    date, report_type, system, title, message, extras = line.split('|')
    # Further split the extras into tags
    tags = extras.split(',')
    # Convert the iso8601 date into a unix timestamp, assuming the timestamp
    # string is in the same timezone as the machine that's parsing it.
    date = datetime.strptime(date, "%Y-%m-%d %H:%M:%S.%f")
    date = time.mktime(date.timetuple()) 
    logged_event = {
        'msg_title': title,
        'timestamp': date,
        'msg_text': message,
        'priority': 'normal',
        'event_type': report_type,
        'aggregation_key': system,
        'tags': tags,
        'alert_type': 'error'
    }
    return logged_event
```

And then we would configure our `datadog.conf` to include the dogstream option as follows:
```
dogstreams: /path/to/mylogfile.log:/path/to/mylogparser.py:my_log_parser
# (N.B., Windows users should replace each "/" with the escaped "\\") 
```

This specific log-line parsed with this parser created the following event in datadog:

{{< img src="agent/faq/log_event_in_dd.jpg" alt="Log event in Datadog" responsive="true" popup="true" style="width:70%;">}}

## Send extra parameters to your custom parsing function

Once you have setup your custom parser to send metric or events to your platform, you should have something like this in your `datadog.conf`:

```
 dogstreams: /path/to/log1:/path/to/my/parsers_module.py:custom_parser
```

And in your parsers_module.py a function defined as:  
```python

def custom_parser(logger, line)
```

You can now change the parity of your function to take extra parameter as shown [in this agent example](https://github.com/DataDog/dd-agent/blob/5.13.x/checks/datadog.py#L210)

So if you change your configuration file to:

```
dogstreams: /path/to/log1:/path/to/my/parsers_module.py:custom_parser:customvar1:customvar2
```

And your parsing function as:

```python

def custom_parser(logger, line, parser_state, *parser_args):
```

You have a tuple parameter in **parser_args** as (customvar1, customvar2) which is ready to use in your code by using parser_args[0] and parser_args[1].

**Note**: the parameter **parser_state** does not have to be used but it has to be in the signature of the function. And if you have only one parameter, you have to use **parser_args[1]** to get it.

As an example, if we have the same parser as in the documentation but this time we do not want to extract the metric name from the log but to set it thanks to this parameter:

In my configuration file I would have: 

```
dogstreams: /Users/Documents/Parser/test.log:/Users/Documents/Parser/myparser.py:parse_web:logmetric
```

## Troubleshooting Your Custom Log-Parser

Bugs happen, so being able to see the traceback from your log-parsers is very important. You can do this if you are running the agent with its [agent logs](/agent/faq/log-locations) set at the "DEBUG" level. The agent's log-level can be set in the `datadog.conf` by uncommenting and editing [this line](https://github.com/DataDog/dd-agent/blob/5.7.x/datadog.conf.example#L211), and then [restarting the agent](/agent/faq/start-stop-restart-the-datadog-agent). Once that's configured properly, traceback resulting from errors in your custom log-parser can be found in the *collector.log* file ([read here for where to find your agent logs](/agent/faq/log-locations)), and it generally includes the string checks.collector(datadog.py:278) | Error while parsing line in them ([here's the agent code where the error is likely to be thrown](https://github.com/DataDog/dd-agent/blob/5.7.x/checks/datadog.py#L278)).

Note that whenever you make a change to your custom log-parser, [restart the agent](/agent/faq/start-stop-restart-the-datadog-agent) to put that change into effect.

If you suspect there is some error occurring beyond the scope of your custom log-parser function, feel free to [reach out to support](/help), but do first set the agent's log-level at "DEBUG", run the agent for a few minutes while ensuring that new logs are being added to your files, and then [run the flare command](/agent/faq/send-logs-and-configs-to-datadog-via-flare-command) from your agent. That gives to the support team the information needed to effectively troubleshoot the issue.

## Further Reading

{{< partial name="whats-next/whats-next.html" >}}