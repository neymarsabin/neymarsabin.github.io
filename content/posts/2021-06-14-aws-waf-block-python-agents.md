---
layout: post
title: AWS-WAF Block Python Agents
type: blog
author: neymarsabin
tags:
- AWS
- firewall
---

Create a new rule that will block requests with python/curl as user agents.
### JSON for Agent Check

```json
{
  "Name": "User-Agent-Rule",
  "Priority": 0,
  "Statement": {
    "OrStatement": {
      "Statements": [
        {
          "ByteMatchStatement": {
            "SearchString": "python",
            "FieldToMatch": {
              "SingleHeader": {
                "Name": "user-agent"
              }
            },
            "TextTransformations": [
              {
                "Priority": 0,
                "Type": "LOWERCASE"
              }
            ],
            "PositionalConstraint": "CONTAINS"
          }
        },
        {
          "ByteMatchStatement": {
            "SearchString": "curl",
            "FieldToMatch": {
              "SingleHeader": {
                "Name": "user-agent"
              }
            },
            "TextTransformations": [
              {
                "Priority": 0,
                "Type": "NONE"
              }
            ],
            "PositionalConstraint": "CONTAINS"
          }
        }
      ]
    }
  },
  "Action": {
    "Block": {}
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "User-Agent-Rule"
  }
}
```
