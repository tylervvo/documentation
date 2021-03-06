---
title: Log Management Security
kind: documentation
aliases:
    - /logs/security/
further_reading:
- link: "/security/"
  tag: "Documentation"
  text: "Review the main categories of data submitted to Datadog"
---

<div class="alert alert-info">This page is about the security of Datadog; if you're looking for the Security Monitoring product, see the <a href="/security_monitoring" target="_blank">Security Monitoring section</a>.</div>

This article is part of a [series on data security][1].

The Log Management product supports multiple [environments and formats][2], allowing customers the flexibility to submit to Datadog nearly any data they choose. This article describes the main security guarantees and filtering controls available to users when submitting logs to Datadog.

## Information Security

The Datadog Agent submits logs to datadog either through HTTPS or through TLS-encrypted TCP connection on port 10516, requiring outbound communication (see [Agent Transport for logs][3]).

Datadog uses symmetric encryption at rest (AES-256) for indexed logs. Indexed logs are deleted from the Datadog platform once their retention period, as defined by the customer, expires.

## Logs Filtering

For customers using release 6 or above, the Agent can be configured to filter logs sent by the Agent to the Datadog application. To prevent the submission of specific logs, use the `log_processing_rules` [setting][4], with the **exclude_at_match** or **include_at_match** `type`. This setting enables the creation of a list containing one or more regular expressions, which instructs the Agent to filter out logs based on the inclusion or exclusion rules supplied.

## Logs Obfuscation

For customers using release 6, the Agent can be configured to obfuscate specific patterns within logs sent by the Agent to the Datadog application. To mask sensitive sequences within your logs, use the `log_processing_rules` [setting][5], with the  **mask_sequences** `type`. This setting enables the creation of a list containing one or more regular expressions, which instructs the Agent to redact sensitive data within your logs.

## HIPAA-enabled Customers

Datadog will sign a Business Associate Agreement (BAA) with customers that transmit protected health information (ePHI) via Datadog’s Log Management Service.

These features are not available to customers who have signed Datadog's BAA:

* Users cannot request support via chat
* Generation of metrics from logs is disabled
* Notifications from Log Monitors cannot include log samples
* Log Monitors cannot be configured with a `group-by` clause
* You cannot [share][8] logs (nor traces) from the explorer through web integrations.

If you have any questions about how the Log Management Service satisfies the applicable requirements under HIPAA, please contact your account manager.

**Note:**

Previously, HIPAA-enabled customers needed to use specific endpoints to submit logs in order to enforce specific encryptions. This is no longer necessary. The encryptions are enabled on all log submission endpoints.

These legacy endpoints are still supported:

* `tcp-encrypted-intake.logs.datadoghq.com`
* `lambda-tcp-encrypted-intake.logs.datadoghq.com`
* `gcp-encrypted-intake.logs.datadoghq.com`
* `http-encrypted-intake.logs.datadoghq.com`

### Further Reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: /security/
[2]: /logs/log_collection/
[3]: /agent/logs/log_transport
[4]: /agent/logs/advanced_log_collection/#filter-logs
[5]: /agent/logs/advanced_log_collection/#scrub-sensitive-data-from-your-logs
[6]: /integrations/amazon_lambda/#log-collection
[7]: /integrations/google_cloud_platform/?tab=datadogussite#log-collection
[8]: /logs/explorer/#share-views
