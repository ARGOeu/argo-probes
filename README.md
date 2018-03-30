# argo-probes

This package includes probes for ARGO internal services. 

Currently, there are probes for:

- ARGO EGI Connectors
- ARGO Messaging service
- ARGO Messaging Nagios publisher
- ARGO Web API
- POEM service
- Compute Engine dataflow

## ARGO Messaging service

Probe is inspecting AMS service by trying to publish and consume randomly
generated messages. Probe creates a topic and subscription, generates random
100 messages with payload about 500 bytes that tries to publish to service
following immediate try of consuming them. If the integrity of messages is
preserved on publishing and consuming side, service is working fine and probe
will return successfull status.

The usage is:

```sh
$ usage: ams-probe [-h] [--host HOST] --token TOKEN --project PROJECT
                   [--topic TOPIC] [--subscription SUBSCRIPTION]
                   [--timeout TIMEOUT]

```

where:
- (--host): the FQDN of the AMS service. Default: messaging-devel.argo.grnet.gr
- (--token): secret used to authenticate to AMS service
- (--project): project created on AMS service
- (--topic): topic that will be created in project and that will hold published
             messages. Default: nagios_sensor_topic.
- (--subscription): subscription created for topic from which messages will be
                    pulled. Default: nagios_sensor_sub
- (--timeout): Timeout after connection is considered dead. Default: 180 

### Usage example

```sh
$ ./ams-probe --token T0K3N --host messaging-devel.argo.grnet.gr --project EGI --topic probetest --subscription probetestsub --timeout 30
```

## ARGO Messaging Nagios publisher

Probe is inspecting AMS publisher running on Nagios monitoring instances. It's
inspecting trends of published messages for each spawned worker and raises alarm if
number of published messages of any worker is below expected threshold. It queries local
inspection socket that publisher exposes and reports back status with the help of NRPE
Nagios system.

The usage is:

```sh
usage: amspub_check.py [-h] -s SOCKET -q QUERY -c THRESHOLD [-t TIMEOUT]
```

where:
- (-s): local path of publisher inspection socket
- (-q): simple query that can be specified multiple times consisted of worker name and identifier of published or consumed
    messages in specified minute interval, e.g. `w:metrics+g:published15`
    - `metrics` is name of worker that will be inspected
    - `published15` is identifier designating that caller is interested in number of
        published messages in last 15 minutes
- (-c): threshold corresponding to each query 
- (-t): optional timeout after which probe will no longer wait for answer from socket

### Usage example

```sh
./ams-publisher-probe -s /var/run/argo-nagios-ams-publisher/sock -q 'w:metrics+g:published180' -c 50000 -q 'w:alarms+g:published180' -c 1
```

## ARGO Web API 

This is a probe for checking AR and status reports are properly working. 
It checks if there are available AR and status data for a selected day. 

The usage of the script is:
```sh
$ usage: web-api [-h] [-H HOSTNAME] [--tenant TENANT] [--rtype RTYPE] [--token TOKEN]
              [--day DAY] [--unused-reports Report1 Report2] [-t TIMEOUT] [-v DEBUG]
```

where:

 - (-H): the hostname of the web api 
 - (--tenant): the tenant name (ex. EGI)
 - (--rtype): the report type (ar or status)
 - (--token): the authorization token
 - (--unused-reports): Report names that are not used anymore. 
 - (--day): the day to check (1,2 ..3 means 1 or 2 or 3 days back)
 - (-t): the timeout
 - (-v): prints some debug data when is set to on  (by default off)
 
### Usage example

```sh
$ ./web-api -H web-api.test.com --tenant tenantname --rtype ar --token 12321312313123 --unused-reports  Report1 Report2  --day 1 -t 180 -v
```

## Compute Engine dataflow

This is a probe for checking the compute engine's dataflow, making sure that all components work as intented.
The checking involves the probe publishing a message to AMS, and expecting after some time, to find the same message produced by the system.If the message is identical, and has been delivered in reasonable time, then everything is ok, otherwise, we examine the result, to figure out, what went wrong with the system.

Usage of the script:
```sh
$ ce_check.py [-h] [-H HOSTNAME] [--project Project]  [--token TOKEN]
              [--push_topic Push Topic] [--pull_subscription Pull Subscription] [-t TIMEOUT]
			  
```
 - (-H): the hostname of the AMS endpoint.
 - (--project): the project that holds the topics and subscriptions.
 - (--token): the authorization token.
 - (--push_topic): the name of the topic, where the probe should publish its data.
 - (--pull_subscription): the name of the subscription, where the probe will check for system's response.
 - (--push_subscription): the name of the subscription, where the System will read from.
 - (-t): A timeout option(seconds) for AMS library requests.
 - (-i): a timewindow(seconds) between publishing and retrieving the message that is expected and considered 'healthy' for the system.
 
### Usage example
 
 ```sh
 $ ce_check -H ams-endpoint.gr --project TEST_PR --token test_token --push_topic test_topic --pull_subscription test_sub --push_subscription test_sub_2 -t 180 -i 500
 
 ```
