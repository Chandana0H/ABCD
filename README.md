# Operations Support Systems (OSS) Self Healing Agent

## What is the OSS Self-Healing Agent

The OSS Self Healing Agent (code named Nightingale) helps service teams automatically address issues to restore their service to a healthy state.  The objective is to decrease Mean Time to Service Restoration (MTSR) as well as avoid the need to page humans to fix issues.  The self-healing agent is deployed into a service's account and will retrieve alerts for the service and pass the alert to custom code written by the service team to restore the service.  The self-healing agent depends on monitoring provided by the service.  The monitoring alerts are sent to OSS TIP where they can be correlated, deduplicated and the agent will pull alerts based on filters provided by the service team.  In the event the self-healing code is unable to rectify the issue, then the team will be paged.

The following is a picture of the oss self-healing architecture:

<img src="images/nightingale-sidecar.drawio.png"/>

## Why is there an OSS Self-Healing Agent

The self-healing agent was created to simplify the prerequisites to using TIP self-healing.  The self-healing agent will simplify the TIP self-healing onboarding experience for IBM Cloud services by providing an agent which will be deployed in service team's account and handle the retrieval and update of alerts with TIP. The agent relieves service team's requirements to register subscriptions with TIP and stand up endpoints for catching alerts. The self-healing agent provides:

- **Secure deployment**. The agent runs in IBM Cloud service team’s account and sends the alerts to the service team's container which takes the recovery action.
- **Simplified deployment**. The agent requires no prerequisite dependency other than kubernetes. VSI deployment is also supported and requires Docker.
- **Pull based TIP event subscription support**. The agent will query TIP on a regular basis (query interval is specified by service team inside the agent configuration) to fetch the latest TIP alerts. In this communication model, the agent only requires the egress connection to OSS but has no ingress exposure outside of service team's account.

## How

Once your team is ready to start using the agent, the following instructions will help you get started. Also, this [8 minute video](https://ibm.box.com/s/uq7wkwtazw5drvwk5y7ukxjce5pet4cr) can also help you understand the steps to get running.

Another item to consider when adopting self-healing is building trust. It is extremely common for teams who first adopt self-healing automation to be very suspect about how well it will work.  A common fear is self-healing will go too far in trying to resolve an issue.  Many questions and concerns get raised.  These are *all excellent questions*!  Teams need to think through the scenarios they want to heal through automation.  Our best practice to to start small.  Choose a situation that is well understood by your team and not too hard to implement.  Once that case is running and proving itself, then confidence will build and the team can move to more scenarios.

### Onboarding

The first step to using OSS self-healing is to get onboarded.  To do this, fill out an [onboarding request](https://github.ibm.com/cloud-sre/oss-self-healing/issues/new/choose) in this repo.  The onboarding request contains the following fields:

- **Environment (test/production)**: Simply provide `test` or `production` depending on the environment you need to be onboarded.
- **Owning team**: Provide the name of your team so we have a contact point if needed.
- **Owner email**: Provide your email so we have your contact information.
- **Owning manager email**: Provide the email of your manager.  Your manager will also need to approve this request.
- **IBM Container Registry ServiceID**: This is an optional ID which is required in two situations: when pulling a non-GA version of the OSS Self-Healing agent, or when your self-healing processor image requires authentication to be pulled. The GA version of the self-healing agent is stored in a public registry which does not require authentication (this has been approved by IBM Cloud security). As an example, if you are only using the GA version of the OSS Self-healing Agent and your self-healing processor image does not require authentication to be pulled, then you do not need to provide this ID.  If provided, this ServiceID **must** be in production (cloud.ibm.com) regardless of the environment to which you are onboarding because ICR runs in production. See additional comments below about usage of IAM ServiceIDs. Example: ServiceId-f0eedaac-1905-4b69-a54e-001bbf678ac5
- **IBM Container Registry ServiceID Account ID**: Only required if the IBM Container Registry ServiceID was provided. Provide the account ID in which the IBM Container Registry ServiceID was created. Example: 5b7a6f956d2f49fcb98451cafe5b25b6
- **Agent ServiceID**: This is an IAM ServiceID which is used by the agent to retrieve alerts on your behalf.  If onboarding to test, this ID must be created in test.cloud.ibm.com, and if onboarding to production, this must be created in cloud.ibm.com. See additional comments below about usage of IAM ServiceIDs. Example: ServiceId-f0eedaac-1905-4b69-a54e-001bbf678ac5
- **Agent ServiceID Account ID**: Provide the account ID in which the Agent ServiceID was created. Example: 5b7a6f956d2f49fcb98451cafe5b25b6
- **Business justification**: Provide a brief sentence about why you need to onboard.

Once you have submitted the request, please ask your manager to add a simple comment to the issue indicating approval.  Just as easy as "I approve.".

**CLI Usage**: If you would like to use the [CLI](https://github.ibm.com/cloud-sre/nightingale-cli) to test your filter, then you will also need a credential for it.  You may use your own ID by requesting access via [AccessHub](https://ibm.idaccesshub.com/ECMv6/request/requestHome).
- Request New Access
- Select `Cloud Operations Availability Platform - staging` for test or `Cloud Operations Availability Platform` for production.
- Add `SelfHealing API`

#### Next steps

Upon completion of the onboarding request, the OSS team will start the process of authorizing your ServiceIDs to the appropriate IAM Access Groups.  OSS will generate access requests on your behalf in the [OSS ServiceID access request repository](https://ibm.biz/ossserviceids). Once the internal approvals and automation has completed, then your onboarding request will be marked as completed and closed.

#### Creating IAM ServiceIDs

The IAM ServiceIDs you create can be created in any account.  Here are some tips from experiences we have already encountered:
- You do not need to create the ServiceID in an OSS account.  In fact, that is not recommended.
- Depending on your level of access, you may be able to create a ServiceID in an account and create API keys for it. However, you may be prevented from deleting the API keys you've created.  Be sure to create your ServiceID in an account to which you have enough access.
- Creation of ServiceIDs in personal accounts is not recommended. If the individual deletes the ID by accident or by leaving the company, then the ID will be deleted from the OSS accounts as well.
- The ServiceID you use to pull the non-GA version of the self-healing agent will also need to be the ID used to pull from your repository if pulling your self-healing processor image requires authentication.  If this doesn't work, then you can download the bundle from OSS and put it on your own repository.  There is no support for one ID to pull the non-GA version and another ID to pull your self-healing processor image.

#### Planning for Onboarding

The following estimates are intended to provide a general sense of how long it will take to enable self-healing.  Obviously times can vary from team to team based on their environment and particular problem being solved.

| Activity                                      | Estimate | Description                                                                                                                                                                                                                                                                               |
| --------------------------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Complete onboarding form                      | 1 day    | You'll create 1 or 2 IAM Service IDs, and fill out the [request form](https://github.ibm.com/cloud-sre/oss-self-healing/issues/new/choose) with manager approval. OSS should complete the request in 1 business day.                                                                      |
| Identify scenario to automate                 | 1 day    | Choose the first scenario to self-heal. We recommend starting with a simple scenario. For example, do you have runbooks instructing operations to restart a pod? This could be a good candidate.                                                                                          |
| Create self-healing processor                 | 2 days   | Creation of the self-healing processor primary code can vary greatly depending on complexity of actions needed. Here we assume a simple action such as pod restart and developer familiarity with their environment.                                                                      |
| Integration with self-healing agent container | 2 days   | The developer will need to read the [specification API](https://ibm.biz/ossselfhealingagent) and provide expected endpoints so that the agent can communicate with the processor. If you are unable to see the documentation because you get an error indicating the page isn't available or that it might have been moved, please update the cloud account in the console to your personal account and retry the link. |
| Credential handling                           | 1 day    | A further assumption is the kubernetes model using kube secrets is being followed and the service team already has a well established method of storing and handling credentials. If not and new way to retrieve and store credentials is needed, this could require a full sprint alone. |
| Unit tests                                    | 2 days   | The developer should provide unit tests to test the self-healing code which will be executed during CICD to ensure the code is operating correctly.                                                                                                                                       |
| Integration with CICD                         | 2 days   | The new self-healing code will need to be integrated to the service's CICD.  Assuming someone familiar with CICD is involved, this should not be a lengthy process.                                                                                                                       |
| Test self-healing in test                     | 3 days   | Once the self healing pod is created and integrated with CICD, there will be a period of testing and bug fixing in the test environment.                                                                                                                                                  |
| Documentation and education                   | 3 days   | Documentation and perhaps an education session with the service's operations team will be necessary so that everyone is aware of the new capability.                                                                                                                                      |

TOTAL: 17 days. Roughly 2 two-week sprints.

### Offboarding

When your team is ready to offboard and stop using the self-healing agent, the only thing that is required is for you to delete your ServiceIDs.  This will automatically delete the access inside of IAM.

### Agent Deployment

Deployment bundles for the agent can be obtained from https://github.ibm.com/cloud-sre/nightingale-bundles.  The bundle has a README file that describes how to deploy the agent for a particular environment.  The agent is deployed in a container within the same pod as the self-healing processor container provided by the service team. The two containers communicate over http.

If you are interested in contributing to the charts provided in the bundles, the location of the [repo is here](https://github.ibm.com/cloud-sre/nightingale-chart).  The repo also contains information regarding the env variables that can be configured in the helm chart.  Normally you should not need to deal with this repo, but it is referenced here for additional information.

#### Agent Versioning

When downloading bundles, the bundle has the following naming pattern:

`Selfhealing on <platform> V<v>.<r>.<p> <designation>`

where

`<platform>`
- K8S = Kubernetes bundle
- VSI = Virtual Server bundle

`<v>`
- Version = Major change. Not backward compatible with previous versions.

`<r>`
- Release = New function added. Backward compatible with previous in this version.

`<p>`
- Patch = Fixes only. Backward compatible with previous in this version.

`<designation>`
- GA = tested and ready for deployment by all deployers.
- RC = Release Candidate currently in test and may be promoted to GA. May have bugs.
- Beta = Experimental release only for early adopters. Likely some bugs.
- Dev = Development release for OSS team. Has bugs.


### Agent Configuration

The agent configuration file is contained within the deployment bundle.  When you unpack the bundle, look for the README file which will describe all of the configuration options.

### Filters

Filters describe the alerts that are of interest for self-healing.  The self-healing agent will retrieve alerts based on matches with the filter.  The self-healing agent will only return alerts that have been marked as self-healing enabled by the monitoring system.  The filter format follows that of the TIP subscription filter. The following example shows a filter to return all severity 1 alerts for service "oss-platform".
```
{
  "filter":[
    {
      "term": {
        "crn_service_name": "oss-platform"
      }
    },
    {
      "term": {
        "console": "toc"
      }
    },
    {
      "range": {
        "severity": {
          "eq":1
        }
      }
    }
  ]
}
```
Another example where syntax is a JSON array containing two filter objects.

Each filter object represents the criteria for retrieving alerts from different services using the same agent.
- First filter to return all severity 1, 2, and 3 alerts for service "oss-platform"
- Second filter to return all severity 1, 2, and 3 alerts for service "tip-test"
```
[{
  "filter":[
    {
      "term": {
        "crn_service_name": "oss-platform"
      }
    },
    {
      "range": {
        "severity": {
          "lte":3
        }
      }
    }
  ]
},{
  "filter":[
    {
      "term": {
        "crn_service_name": "tip-test"
      }
    },
    {
      "range": {
        "severity": {
          "lte":3
        }
      }
    }
  ]
}]
```


Available terms for the filter are as follow:

| Term                  | Description                                                                 | Values                                                                                                                                                                                                       |
|---------------------- |-----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| crn_service_name      | Name of the service as presented in a CRN or ServiceNow configuration item. | Use a service name for services onboarded to TIP. (see [CRN Specification](https://github.ibm.com/ibmcloud/builders-guide/blob/master/specifications/crn/CRN.md)). Either this or `tribe_name` is required.  |
| crn_location          | Location as present in the location segment of the CRN.                     | A valid location such as (but not limited to) us-east, us-south. (see [CRN Specification](https://github.ibm.com/ibmcloud/builders-guide/blob/master/specifications/crn/CRN.md))                             |
| crn_service_instance  | The service instance provided in the CRN.                                   | A valid service instance in the CRN. (see [CRN Specification](https://github.ibm.com/ibmcloud/builders-guide/blob/master/specifications/crn/CRN.md))                                                         |
| crn_scope             | The scope provided in the CRN.                                              | A valid CRN scope. (see [CRN Specification](https://github.ibm.com/ibmcloud/builders-guide/blob/master/specifications/crn/CRN.md))                                                                           |
| crn_resource_type     | The resource type provided in the CRN.                                      | A valid resource type. (see [CRN Specification](https://github.ibm.com/ibmcloud/builders-guide/blob/master/specifications/crn/CRN.md))                                                                       |
| crn_resource          | The resource provided in the CRN.                                           | A CRN resource. (see [CRN Specification](https://github.ibm.com/ibmcloud/builders-guide/blob/master/specifications/crn/CRN.md))                                                                              |
| tip_msg_type          | Describes the type of alert message such as create, update, or close.       | One of "create.notice", "update.notice", or "close.notice"                                                                                                                                                   |
| console               | Describes if the alert is sent to ServiceNow or used only for analytics.    | "toc" = ServiceNow, "analytics" = analytics only                                                                                                                                                             |
| tribe_name            | Name of the tribe to which the service belongs.                             | Name of tribe as present in ServiceNow. Either this or `crn_service_name` is required.                                                                                                                       |
| situation             | The alert's situation name.                                                 | A situation name as present on alerts from monitoring.                                                                                                                                                       |
| severity              | The severity of the alert when it arrives at TIP.                           | Values 1, 2, 3, or 4. Use "range" in the filter with operators gte, gt, lte, lt.                                                                                                                             |
| short_description     | Search for a value present within the short description of the alert.       | Any string value.                                                                                                                                                                                            |

Either `crn_service_name` or `tribe_name` is required.  Keep in mind if you use `tribe_name`, then you will get alerts for multiple services.  You must coordinate with all of the service teams for which you will be self-healing so you do not interfere other self-healing processes.

#### Monitoring Alert Configuration <a id="monitoring-alert-configuration"></a>

In addition to the filter, OSS self-healing will only select alerts that have been marked for self-healing.  Alerts are marked for self-healing by the monitoring system using a special attribute in the alert named `self_healing_enabled`.  In fact, there are a number of self-healing specific attributes.

These 4 fields have been added to the TIP alert Concern API to allow for the integration of the alert flow with a custom self-healing processor.

| Field Name           | Required | Updatable | Description |
| -------------------- | :------: | :-------: | ----------- |
| self_healing_enabled |    No    |    No     | "false"\|"true"\|"alphanumeric-string-no-spec-char-but-dash"<br>- If omitted, self_healing_enabled is set to `"false"`<br>- Any value other than `"false"` will enable the self-healing flow. String values other than `"true"` are allowed and will be included in notifications forwarded to the self-healing processor. |
| self_healing_timeout |    No    |    Once   | Integer to specify how long in seconds to wait for `self_healing_status` update.<br>- Default set to 300 seconds if not supplied.<br>This means your self-healing processor should update this value or return the final `self_healing_status` value before the timeout expires. When the timeout expires, the incident will page responder. |
| self_healing_status  |    No    |    Once   | `self_healing_status` allows the client to indicate the status of self-healing using one of the following status values.  By default, when an alert arrives at TIP from the monitoring system with `self_healing_enabled`=true, then TIP will set the `self_healing_status` to `in_progress` automatically as long as it isn't already set by the monitoring system. Self-healing will only forward alerts with a self-healing status of `in_progress` to the self-healing processor.  After that, when the self-healing processor takes action, it can update the `self_healing_status`.  <br><br> **disabled** - Self-healing is disabled for this alert and will not be attempted.<br>- `"self_healing_status": "disabled"` is a predetermined value that should only be set by the alert source in the create.notice.<br>When the self-healing processor receives a message with `"self_healing_status": "disabled"`, the processor should discard the message because it is predetermined that there is no action required for self-healing.<br>`"self_healing_status": "disabled"` should never be returned by the self-healing procesor. Instead, the self-healing processor would set `"self_healing_status": "unavailable"`<br>- Incident will page responder.<br><br>**unavailable** - Indicates **the self-healing processor has determined** that no self healing is available or defined for this alert.<br>- Incident will page responder.<br>- Similar to **disabled**, but is set by an update.notice from the self-healing processor after the processor determines that there is no self-healing action available for the alert.<br><br>**succeeded** - Self healing finished and the incident is resolved based on the results.<br>- Incident will be resolved.<br>- Alert remains active until close.notice or the alert expires from inactivity.<br> Your self-healing processor may set `"self_healing_status": "succeeded"` using `"tip_msg_type":"close.notice"` rather than using `update.notice` to close the alert too; however, the monitoring solution that generated the alert may not be aware that the alert has been closed and may not raise a new alert for this problem because it believes that the alert has already been sent. Best solution would be for the self-healing processor to reset the monitor to allow it to re-evaluate the condition when the self-healing processor is closing the alert.<br><br>**analyzing** - Self healing has started, but an operator must respond immediately regardless of result.<br>- Incident will page responder.<br><br>**analyzed** - Self healing completed, but no determination of success or fail is made.<br>- Incident will page responder if not already paged by **analyzing**. <br><br>**failed** - Self healing completed, but the incident is not resolved.<br>- Incident will page responder.<br><br>**timeout** - Self healing took too long to complete.<br>- Incident will page responder.<br>- May be set by update.notice or `self_healing_timeout` expiration.<br><br>**bypassed** - Self healing cannot be executed.<br>- Incident will page responder.<br>Indicates self-healing processor may not be able to execute the required action.<br><br>**active** - Self-healing is currently executing and the processor wishes to no longer receive updates for this `alert_id`.<br><br>|
| self_healing_ui_url  |    No    |    Once   | Link to display self-healing results, logs or other diagnostic results collected by self-healing action. See [Collecting Data](#collecting-data) for more information for more information. |

#### Agent Metrics and Monitoring <a id="agent-metrics"></a>

For the alerts with self-healing enabled, the agent produces two sets of metrics.
  - Kibana metrics
  - Prometheus metrics.

**Kibana Metrics**

Metrics inside Kibana include:
- The timestamp Kibana received the metric
- AlertID
- Alert Situation
- Execution time in milliseconds
- Service Name
- The result of self-healing

**Prometheus Metrics** 

Prometheus metrics can be found by making a GET request to the agent at `http://localhost:<agentPort>/metrics`
  - `<agentPort>` can be set in the Helm installation bundle under environment variables. By default the value is 8080.

These metrics include:

| Metric                                 | Description                                                                                                             | Values                                                    |
|--------------------------------------- |-------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------- |
| selfhealing_alerts_total_count         | The total number of self-healing alerts pulled from TIP in the agents lifetime. Not all alerts may have been processed  | Positive integer value as a counter                       |
| selfhealing_alerts_failure_total_count | The total number of failures to pull self-healing alerts from TIP in the agents lifetime                                | Positive integer value as a counter                       |
| selfhealing_duration_seconds           | Duration in seconds taken to perform selfhealing actions over the agents lifetime in buckets                            | Linear buckets with a positive integer value as a counter |
| selfhealing_duration_seconds_sum       | Uses the alert situation as a key and adds all durations of that key together                                           | Decimal value as a sum                                    |
| selfhealing_duration_seconds_count     | A counter for the total number of times self-healing actions have been perfomed for a single alert situation            | Positive integer value as a counter                       |
| selfhealing_result                     | The results of the selfhealing action taken by the pulled alerts                                                        | A list of counters containing a positve integer value.    |

*The *selfhealing_duration_seconds* bucket spread is customizable. If default values are not desired set the startBucket, bucketWidth, and numBuckets environment variables. 

For example: 
- if startBucket = 5 then the first buckets value is 5 seconds
- if bucketWidth = 5 buckets are at intervals of 5 seconds from the starting bucket
- if numBuckets = 5 there are 5 total buckets 

Using these values you would have buckets with values 5, 10, 15, 20, and 25. Durations less than or equal to 5 seconds would increment the first bucket's count, and durations less than or equal to 10 seconds would increment the second bucket. This pattern continues for all 5 buckets.

**Examples**
| Metric                                 | Format                                                                                                                                     | Example                                                                  |
|--------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| selfhealing_alerts_total_count         | selfhealing_alerts_total_count "value"                                                                                                     | selfhealing_alerts_total_count 5                                         |
| selfhealing_alerts_failure_total_count | selfhealing_alerts_failure_total_count "value"                                                                                             | selfhealing_alerts_failure_total_count 1                                 |
| selfhealing_duration_seconds           | selfhealing_duration_seconds_bucket{alert_type="*Your alert situation*",le="*startBucket*"} "value"                                        | selfhealing_duration_seconds_bucket{alert_type="out_of_memory",le="5"} 3 |
| selfhealing_duration_seconds_sum       | {alert_type= *"Your alert situation"*} *"sum of durations in seconds"*                                                                     | selfhealing_duration_seconds_sum{alert_type="out_of_memory"} 4.04        |
| selfhealing_duration_seconds_count     | {alert_type= *"Your alert situation"*} *"number of alerts processed with your alert situation"*                                            | selfhealing_duration_seconds_count{alert_type="out_of_memory"} 3         |
| selfhealing_result                     | {alert_type= *"Your alert situation"* result= *"success, failed, unavailable, analyzing, analyzed, bypassed, or timeout"*} *"gauge value"* | selfhealing_result{alert_type= "out_of_memory", result= "failed"} 5      |

### API documentation

Since the agent and self-healing processor communicate over http, there is an API specification for the APIs implemented by the agent and the processor.  Please see the agent [API specification](https://ibm.biz/ossselfhealingagent). If you are unable to see the documentation because you get an error indicating the page isn't available or that it might have been moved, please update the cloud account in the console to your personal account and retry the link.

For more detailed insight into the flow of messages between the agent and the service self-healing processor including the alert to TIP, see the [Agent To Self-healing Processor Flows](https://github.ibm.com/cloud-sre/oss-platform-architecture/blob/master/nightingale/agent-to-service-flows.md).  Additionally, we have created a [self-healing processor flow chart](./processor-flow.md) which may help you understand a recommended approach to processor design.

### Limitations

#### Duplicate alerts are possible

There are situtations where the self-healing processor may recieve duplicate alerts.  The infrastructure is built to prevent duplicates; however, there can be timing issues that may present themselves especially in the event of a regional failover.  Regional failovers occur when an entire region, such as us-south, goes down and the infrastructure automatically sends traffic to a different region.  Depending on the flow of alerts, some could be duplicated.

Alerts may also appear to be duplicated to the self-healing logic when in fact they are not, but the self-healing processor should be prepared for these cases.  As an example, the following table illustrates a chronology of the lifecycle of an alert, and it is possible for a self-healing processor to see each state based on timing and polling frequency:
| Timestamp           | TIP msg type  | Incident ID | Description                                                                 |
| ------------------- | ------------- | ----------- | --------------------------------------------------------------------------- |
| 2023-02-03T07:49:54 | create.notice |             | Received when the alert has been sent to TIP but not yet seen by ServiceNow |
| 2023-02-03T07:50:01 | create.notice | INC5932108  | Received when the incident is created in ServiceNow for this alert          |
| 2023-02-03T08:32:46 | update.notice | INC5932108  | Received when the incident is modified inside ServiceNow                    |
| 2023-02-03T08:50:08 | close.notice  |             | Received when the monitoring system sends a closing notices for this alert  |
| 2023-02-03T08:50:45 | close.notice  | INC5932108  | Received when the incident is closed in ServiceNow                          |

A self-healing processor may not get all of these transitions since the alert will be deduplicated by TIP, but it should be prepared to process them in the event these transitions are received.

#### Delayed alerts

A delayed alert is defined as an alert that is successfully delivered to TIP but is not available to the self-healing engine for a delayed period of time.  Delayed alerts within the self-healing engine may occur for a variety of reasons.  For example, network congestion, resource utilization issues, and other unforeseen problems may delay an alert.  The OSS self-healing engine needs to take this possibility into consideration and deal with it as best it can.  If a delayed alert occurs, the assumption is the service team will become aware of the alert and already be handling it.  The service team becomes aware of the alert because of the `self_healing_timeout` which will page the team when self-healing does not resolve an alert within the specified period of time.  In this case it would be incorrect for the self-healing engine to deliver the alert for self-healing. To address this, the self-healing engine imposes a maximum time limit over which it will query for alerts. The current maximum time limit over which the engine will query is the last 10 minutes.  If the engine is unable to deliver the alert within the maximum time limit, then it will not be delivered at all.

## Security Considerations

The agent is based on containers running side by side and communicating via localhost.  This absolutely requires that you ensure inbound connections from other external sources is blocked.  By default, a pod is non-isolated for ingress; all inbound connections are allowed. Therefore you must take some action to ensure that the pod that contains the agent and self-healing processor blocks all external traffic attempting to enter the pod.  Please see the [Kubernetes NetworkPolicy](https://kubernetes.io/docs/concepts/services-networking/network-policies/) or use [calico](https://docs.tigera.io/calico/latest/getting-started/kubernetes/). Additionally, be sure you do not create a Kubernetes service which exposes the agent pod to other applications in the Kubernetes cluster.

If you are using a VSI, you need to implement IPTables to restrict access the port exposed by the processes running on the localhost, and making sure those are accessible only from 127.0.0.1. Any other IP is blocked. Also, please see and comply with all the [ITSS standards](https://pages.github.ibm.com/ciso-psg/main/standards/itss.html) to ensure your VSI is fully compliant and have fully isolated these containers from any outside traffic.

The agent will make outbound external HTTPS calls to URLs with hostname `tip-oss-flow.cloud.ibm.com` (production) or `tip-oss-flow.test.cloud.ibm.com` (test). In some cases, the agent will attempt to directly call the regional hostname URLs (e.g. `us-south.tip-oss-flow.cloud.ibm.com`, `us-east.tip-oss-flow.cloud.ibm.com`, `eu-de.tip-oss-flow.cloud.ibm.com`). Please ensure that your networking configuration allows these outbound calls to go through.

Since the agent is deployed into the the adopting service's account, it is the responsibility of the adopting service to take the agent deployment through their own security reviews and audits.  The usage of communication without authentication or encryption as been approved by the Cloud Security team only if communication is happening on localhost only and external inbound communications are blocked. You will fail a security review/audit if you don't take care of this.

## Quick Start

To help you start using the OSS self-healing agent faster, there exists the following quick start material:

1. [Self-healing sidecar sample](https://github.ibm.com/cloud-sre/selfhealing-sidecar-sample): This is a simplified project that can be used to jump start your own processor.
2. [Nightingale agent sidecar deployment document](https://ibm.ent.box.com/integrations/officeonline/openOfficeOnline?fileId=1298610191514&wdPid=5b650de1): This is a document that has information about setting up the nightingale sidecar agent.

## Additional information

If you need a reference for getting started with a Jenkins pipeline, please see https://github.ibm.com/cloud-sre/nightingale-scripts/blob/main/jenkins-template/pipeline.sh. For testing purpose, it can be run locally to simulate the Jenkins operation.

To get started with Artifactory, we suggest you onboard with the TaaS Artifactory to create a repository for your team.  To get information go to https://taas.cloud.ibm.com/tools/artifactory, and you can ask for support in Slack on channel #taas-artifactory-help.

To ask questions, visit slack channel [#oss-self-healing](https://ibm-cloudplatform.slack.com/archives/C03EL14TMDM)

## Use Cases and Best Practices

### Event Driven Ansible

A recommended best practice is to make use of [Event Driven Ansible](https://www.ansible.com/use-cases/event-driven-automation) (EDA). EDA will allow you to use the power of Ansible to remedy situations in your environment.  The following diagram shows a real use case which uses both EDA and Terraform.  This self-healing scenario is designed to heal failed VPE gateways.  When a VPE gateway fails, the only way to fix it is recreate it.  In the below picture: 
1. The service monitor will watch for VPE gateway failure and when the gateway fails, it sends an alert to TIP.  Note if multiple alerts are sent to TIP, TIP will deduplicate them.
2. The OSS Self-healing agent will reach out to TIP to get the alert and pass it to the Self-healing processor.  The webhook listening for the alert is provided by EDA.
3. EDA receives the alert and parses it.  It matches the alert to a playbook to be run.
4. The Ansible playbook invokes Terraform to execute a module to recreate the VPE gateway.

<img src="images/Nightingale_ansible_terraform_usecase.drawio.png"/>

### Big Picture

The following depicts a very generalized view of the agent to give an idea of the placement of the OSS self-healing agent.

<img src="images/big-picture.drawio.png"/>

### Collecting Data <a id="collecting-data"></a>

Many times it is useful for the self-healing code to collect information at the time of failure to be used by the operations team to help investigate an issue.  Data could be large and it's not appropriate to add that data to an incident.  Alerts contain an attribute called `self_healing_ui_url` which is intended to be a URL at which collected data can be accessed.  The idea is the data is collected and stored at some appropriate location, such as COS, and a URL to the data is provided in the alert to allow an SME to view the data whenever needed.  Described here is an example implementation of storing data for later retrieval.  This is of course not the only way to do this, but this example may be useful.

- [Video walkthrough](https://ibm.box.com/s/a0o52bmmj47g75j90g9szbcc1iae381l)
- [Presentation shown in video](https://ibm.box.com/s/fdr7be08xv0801magl8o3anbuz9vs3bw)

### Agent Logs Collection <a id="agent-logs"></a>

The Nightingale agent outputs all logs to STDOUT. You may setup/integrate your Kubernetes/VSI/OpenShift cluster with [IBM Cloud Log Analysis](https://cloud.ibm.com/docs/log-analysis?topic=log-analysis-getting-started) to conveniently view and record the logs. If you do not have a working solution in place to view or record the logs of containers running in your environment, then below is some basic information on how to retrieve logs from a container.

**If using Docker:**
[Official documentation](https://docs.docker.com/reference/cli/docker/container/logs/) regarding Docker container logs command
```
docker container logs [OPTIONS] CONTAINER
```
If you would like to save the logs to a local file, you may pipe the output to a file with the `>` symbol. For example
```
docker container logs CONTAINER > filename.log
```

**If using Kubernetes:**
[Official documentation](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/) regarding kubectl logs command
```
kubectl logs POD [-c CONTAINER]
```
If you would like to save the logs to a local file, you may pipe the output to a file with the `>` symbol. For example
```
kubectl logs POD [-c CONTAINER] > filename.log
```
When you run the self-healing processor as a side-car in the same pod, you will be required to provide the container ID using the -c parameter. For example
```
kubectl logs nightingale-agent-7f77c64799-72jp2 -c nightingale-agent
```

Below is a sample log of a running instance of Nightingale agent
```
Mar 26 03:24:00 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:00 checking nightingale prerequisites
Mar 26 03:24:00 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:00 Secret value updated for AGENT_APIKEY 0 44
Mar 26 03:24:00 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:00 nightingale agent agent-28d12bc95bf004a4ef4a7f5e3073dc17-39da0115ac825d81 is running...
Mar 26 03:24:00 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:00 Waiting 10 seconds for network to stabilize before trying to connect...
Mar 26 03:24:10 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:10 Starting to check for connections to dependencies
Mar 26 03:24:10 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:10 ApiKey2Token get the token from cache
Mar 26 03:24:10 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:10 Check for connections to dependencies is successful!
Mar 26 03:24:10 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:10 Alerts call triggered for agent-28d12bc95bf004a4ef4a7f5e3073dc17-39da0115ac825d81
Mar 26 03:24:10 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:10 TraceParent: 00-3ca94a4a9e8630794cc2a205207b6546-3da731d2adffe518-01
Mar 26 03:24:10 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:10 TraceState: ibmosstip=1
Mar 26 03:24:10 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:10 ApiKey2Token get the token from cache
Mar 26 03:24:10 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:10 Running Query {"filter":[{"term":{"crn_service_name":"oss-platform"}},{"term":{"situation":"VPE_Failure"}}]}
Mar 26 03:24:11 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:11 Alerts call returned 0 alerts for agent-28d12bc95bf004a4ef4a7f5e3073dc17-39da0115ac825d81
Mar 26 03:24:20 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:20 Queue is empty
Mar 26 03:24:20 nightingale-agent-597786bf86-6nmxl nightingale-agent 2024/03/26 07:24:20 agent-28d12bc95bf004a4ef4a7f5e3073dc17-39da0115ac825d81 has proccessed all alerts waiting until next pull
```
