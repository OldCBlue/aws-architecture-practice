# aws-architecture-practice
AWS high availability architecture practice

# AWS High Availability Architecture Practice

## Overview
This project demonstrates a high-availability and cost-optimized AWS architecture.

## Architecture
- VPC (10.0.0.0/16)
- Public Subnet / Private Subnet
- Internet Gateway
- Application Load Balancer
- EC2 (Multi-AZ)
- S3 + Gateway Endpoint

## Key Design

### 1. Network Isolation
Public and private subnets are separated to enhance security.

### 2. Cost Optimization
Replaced NAT Gateway with S3 Gateway Endpoint.

### 3. High Availability
Used ALB with Multi-AZ deployment.

## Result
- Reduced infrastructure cost
- Improved security
- Increased availability

## Diagram
(Coming soon)
