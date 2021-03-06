---
title: Why events don't appear to be showing up in the event stream with my github integration ?
kind: faq
---

First you need to configure your Github integration, see [this dedicated documentation article](/integrations/github/).

Then, If you have setup your Webhook on the relevant Github repositories and you can see it sending data but events don't appear to be showing up in the event stream this might come from your Webhook settings:

Instead of having your Webhook configured with content-type:application/x-www-form-urlencoded

You should set your Webhook with content-type:application/json:

{{< img src="integrations/faq/github_webhook_config.png" alt="github_webhook_config" responsive="true" popup="true">}}

Once updated you should see events flowing normally in your Datadog application. If not, feel free to reach out directly to [us](/help).