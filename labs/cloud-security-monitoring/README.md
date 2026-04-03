# AWS SIEM Lab: Cloud Security Monitoring, Detection, and Alerting

## Overview

This lab demonstrates how to build a cloud-native security monitoring workflow in AWS by collecting logs from multiple sources, detecting suspicious behavior, and generating alerts for investigation.

The environment is designed to simulate a small AWS deployment where both host-level activity and AWS control-plane activity are monitored centrally. The lab focuses on visibility, detection engineering, and alerting rather than only system hardening or one-off attack prevention.

This project is intended to show how a security team can move from raw telemetry to actionable alerts using AWS-native services.

---

## Objectives

- Centralize log collection from AWS and operating-system sources
- Monitor host-level authentication activity on an EC2 instance
- Monitor AWS account activity using CloudTrail
- Build detection logic for suspicious events
- Trigger alerts for investigation using cloud-native services
- Document the full workflow from log ingestion to alert generation

---

## Scenario

A small AWS environment is monitored with centralized logging and alerting to detect:

- repeated failed SSH login attempts against an EC2 instance
- suspicious AWS account activity captured through CloudTrail

This lab is designed to reflect a realistic monitoring use case where a defender needs visibility across both the operating system and the AWS account itself.

---

## Planned AWS Services

- Amazon EC2
- Amazon CloudWatch Logs
- AWS CloudTrail
- Amazon S3
- Amazon SNS

---

## High-Level Workflow

1. Activity is generated in the environment
2. Logs are collected from host and AWS sources
3. Logs are centralized in AWS-native logging services
4. Detection logic identifies suspicious patterns
5. Alerts are triggered for investigation

---

## Lab Structure

- `01-architecture` — architecture diagram and component design
- `02-setup-and-build` — AWS resource creation and configuration
- `03-log-sources-and-ingestion` — log sources, log flow, and ingestion setup
- `04-attack-simulation` — controlled attack and suspicious activity generation
- `05-detection-engineering` — detection logic, filters, and thresholds
- `06-alerting-and-response` — alarms, notifications, and response workflow
- `07-reflections-and-improvements` — lessons learned, limitations, and next steps
- `screenshots` — supporting evidence and validation images

---

## Status

In progress.
