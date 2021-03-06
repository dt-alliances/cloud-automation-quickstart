# SLO evaluation demo 

For this use case you will start with the builtin task called `evaluation` that performs an automated SLO evaluation.

Cloud Automation has a built-in task called `evaluation` that will perform the SLO evaluation by sending an `evaluation.triggered` event that the builtin [Dynatrace Service](https://github.com/keptn-contrib/dynatrace-service) is listening for.   

As part of the setup, a Dynatrace dashboard was added that you can use to try out the Automated SLO evaluation.  This is one of features that has been first adopted by out Cloud Automation customers.  Typically use cases are to run a SLO evaluation after a deployment, configuration change, test or an automated problem remediation.  

The webhook you will setup will subscribe to the `evaluation.finished` event as to get the results.  For now, don't worry about all the details. For now you are just going to trigger the sequence to see it execute.  

# Review SLO Dashboard

To try it out, lets first review the configuration by going to your Dynatrace environment, choosing `Dashboards` from the left menu, and opening up the one with the name of `KQG;project=slo-demo;stage=production;service=casdemoapp`. 

The dashboard should look similar to this at it contains the SLI and criteria for the evaluation all defined in it:

<img src="images/dynatrace-dashboard.png" width="70%" height="70%">

# Review shipyard file with the evaluation task

Navigate to your GIT `slo-demo` project and open the `shipyard.yaml` file that is in the root folder.  

In this you will see a stage called `production` and a sequence called `getslo` with a task called `evaluation`.

```
apiVersion: spec.keptn.sh/0.2.2
kind: "Shipyard"
metadata:
  name: "slo-demo"
spec:
  stages:
    - name: "production"
      sequences:
      - name: "getslo"
        tasks: 
        - name: "evaluation"
          properties:
            timeframe: "5m"

```

# Configure webhooks

For a quick way to see the events that would be send to a downstream tool, you will configure the webhook subscriptions to send the HTTP request and payload to this web site `https://webhook.site`  

When you later trigger a sequence, you can view the JSON payloads configured in your webhook subscriptions within the https://webhook.site as shown below.

<img src="images/webhook-site.png" width="50%" >

### Step 1: Configure a webhook receiver

1. To get you unique webhook URL, just open https://webhook.site in a new browser tab.  Keep this tab open while using this guide. You can use the copy to clipboard button to get your unique URL that you will configure later in the guide as part of the webhooks configuration.

    <img src="images/webhook-site-copy-url.png" width="75%" height="75%"> 

### Step 2: Add webhook subscription

1. From the Cloud Automation UI, click on the `slo-demo` project`

1. On the left menu click on the `Uniform` option

1. Click on the `webhook-service`

1. Click the `Add subscription` button

1. On the `New subscription` page, fill in the following values as shown below.
    * task = `evaluation`
    * Task suffix = `finished`
    * request method = `POST`
    * URL = the wehbook.site URL you copied
    * custom payload below
        ```
        {
            "project": "{{.data.project}}",
            "service": "{{.data.service}}",
            "stage": "{{.data.stage}}",
            "event": "{{.type}}",
            "result": "{{.data.result}} ({{.data.evaluation.score}} / 100)"
        }
        ```
    * Send finished event = Nothing selected

1. Click the `Create subscription` button

### Step 4: Review

The webhooks should look like this

<img src="images/webhook-list-eval.png" width="75%" height="75%">

# Trigger sequence

A [UI enhancement is coming](https://github.com/keptn/keptn/pull/6958), but for now you need to make an API call or use the Keptn CLI to trigger a sequence to run.  To make this task easier, the `scripts/trigger.sh` script in this repo will send in the various Cloud Automation events using the Keptn CLI and JSON event templates found in the `projects/[project name]/events/` subfolder.  You will run this script from the SSH shell when directed to as part of the use cases.

Take a look at [this](projects/slo-demo/events/sh.keptn.event.production.getslo.triggered.json) file in the project to see what will be passed in.  

### Step 1: Run trigger script 

1. To trigger the sequence, from the SSH terminal run this command

    ```
    cd ~/cloud-automation-quickstart/scripts
    ./trigger.sh
    ```

1. This will prompt for a menu, choose option value of `1` as shown below.

    ```
    =============================================================================
    1) SLO-DEMO      - Send 'sh.keptn.event.production.getslo.triggered' event
    -----------------------------------------------------------------------------
    q) quit and exit
    =============================================================================
    Pick the number for the event to trigger : 1

    Running 'keptn send event --file ../projects/slo-demo/events/sh.keptn.event.production.getslo.triggered.json'
    OUTPUT = ID of Keptn context: 409d7b25-d04b-44f3-a636-d2fc8d67819a
    ```

### Step 2: Review SLO evaluation

Monitor the sequence progress in the Cloud Automation UI.  Once the sequence is complete, you can click on the icon within the sequence to open up the results page.  If the SLO evaluation because of a low calculated score goes not pass (as shown below), that OK.  

<img src="images/evaluation-sequence.png">

Click on the `graph icon` to open the evaluation results pages.  

### Step 3: Review Webhook.site

In the webhook.site to view the generated finished event.

<img src="images/webhook-eval-event.png" width="50%" height="50%">

### Step 4: Run again

You can run a the `trigger.sh` a few more times to make more SLO data as to have a more populated chart.

# More details on the SLO evaluation task

Read more about the benefits in this [Dynatrace Blog](https://www.dynatrace.com/news/blog/automating-slos-helps-sres-go-fast)

The SLO evaluation was performed by the [Cloud Automation Dynatrace service](https://github.com/keptn-contrib/dynatrace-service) that is installed as part of the Cloud Automation environment.  

You can view the dynatrace-service on the uniform page as shown below.

<img src="images/dt-service.png">

SLI data can be retrieved in a few different ways and if you look in the `scripts/create-projects.sh` file you will see command that added the `dynatrace.conf.yaml` to the demo app project with the query from a Dynatrace dashboard option.  

See this [Dynatrace service README page](https://github.com/keptn-contrib/dynatrace-service/blob/master/documentation/slis-via-files.md) for SLI setup options and this [Dynatrace service README page](https://github.com/keptn-contrib/dynatrace-service/blob/master/documentation/slis-via-dashboard.md) for dashboard setup and how you can adjust the dashboard with different SLO targets.

You can read more too on the Quality Gate page within the [Keptn docs](https://keptn.sh/docs/0.12.x/quality_gates/get_started)

<hr>

[<img src="images/prev.png" width="50px" height="50"/>](10-WEBHOOK.md) [<img src="images/next.png" width="50px" height="50"/>](12-RELEASE.md)
