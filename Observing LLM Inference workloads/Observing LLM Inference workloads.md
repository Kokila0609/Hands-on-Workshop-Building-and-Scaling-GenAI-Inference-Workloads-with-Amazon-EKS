****Observing LLM Inference workloads****

In this section, we'll explore how to implement comprehensive observability for LLM inference workloads running on Amazon EKS. We'll cover monitoring solutions for both vLLM and Ray-served models, focusing on key metrics that help understand model performance, resource utilization, and system health.

While this module demonstrates how to instrument observability tools directly on EKS for learning purposes, for production environments at scale, we recommend using AWS managed services such as Amazon Managed Service for Prometheus (AMP)  and Amazon Managed Grafana (AMG)  for improved scalability, reduced operational overhead, and better integration with the AWS ecosystem.

Module Overview

This module guides you through exploring and implementing GPU and LLM inference monitoring, divided into four sequential sections:

**Setting-up Observability Stack**

1. Understand the deployed monitoring components
2. Review Prometheus and Grafana architecture
3. Configure Grafana Operator
4. Deploy NVIDIA DCGM exporter
5. Configuring NVIDIA DCGM Monitoring Dashboard

**Verify exporter functionality**

1. Prepare for dashboard visualization
2. vLLM Model Monitoring

**Implement vLLM-specific metrics collection**

1. Create custom performance dashboards
2. Track token generation and latency metrics
3. Monitor inference queue and processing times
4. Ray Monitoring

**Deploy Ray monitoring components**

Track distributed cluster performance
Monitor worker node health
Analyze task scheduling efficiency
Let's begin by exploring the observability stack already deployed in our cluster, which provides the foundation for our GPU and LLM inference monitoring.

