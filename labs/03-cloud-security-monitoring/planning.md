# Planning

## Lab Title
AWS SIEM Lab: Cloud Security Monitoring, Detection, and Alerting

## Objective
Build an AWS-native security monitoring lab that centralizes host-level and control-plane logs, detects suspicious activity, and triggers alerts for investigation.

## Core Goal
Demonstrate how a security team can monitor an AWS environment by collecting logs from multiple sources, identifying malicious or suspicious behavior, and generating alerts using cloud-native services.

## Scenario
A small AWS environment is monitored with centralized logging and alerting to detect:
- host-level authentication attacks against an EC2 instance
- suspicious AWS account activity captured through CloudTrail

## What This Lab Will Prove
This lab will demonstrate:
- centralized log collection in AWS
- visibility across both operating-system activity and AWS API activity
- detection engineering using defined patterns and thresholds
- alerting for security-relevant events
- a realistic monitoring workflow that maps attack activity to telemetry and alerts

## Planned AWS Services
- Amazon EC2
- Amazon CloudWatch Logs
- AWS CloudTrail
- Amazon S3
- Amazon SNS

## Planned Log Sources
- EC2 system and authentication logs
- CloudTrail management events

## Planned Attack / Activity Simulations
- repeated failed SSH login attempts against an EC2 instance
