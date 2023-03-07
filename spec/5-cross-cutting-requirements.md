# 5 Cross-Cutting Requirements

{% hint style="success" %}
The Cross-cutting requirements described in this section are an extension of the cross-cutting requirements defined in the architecture blueprint and nonfunctional requirements document. This section will describe any additional cross-cutting requirements that apply to this building block, or any requirements that are defined in the non-functional requirements document that are NOT applicable to this Building Block.

Cross-cutting requirements will use the same language (REQUIRED, RECOMMENDED or OPTIONAL) as specified in the architecture document.
{% endhint %}

_\<Example Cross-Cutting Requirements>_

## 5.1 Personal data must be kept private (REQUIRED)

Personal data MUST be kept private and never shared with any parties, except where specific authorisation has been granted. The Consent BB shall follow the privacy principles as laid out in the Govstack architecture.

## 5.2 All transactions must be Audit Logged (RECOMMENDED)

Logs SHOULD be kept in a database of all created, updated, or deleted records. Logs MUST include timestamps and identify the user and affiliation that performed the transaction.

All audit logs shall be integrity protected against tampering. The Consent BB shall follow the data policy and audit logging requirements as laid out in the Govstack architecture.

