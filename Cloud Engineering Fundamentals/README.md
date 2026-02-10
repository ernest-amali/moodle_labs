# README - AWS Well-Architected & Cloud Adoption Framework Assessment

## Project Overview

This capstone project evaluates a two-tier web application migration from on-premises to AWS, applying the **AWS Well-Architected Framework (WAF)** and **AWS Cloud Adoption Framework (CAF)** to ensure alignment with best practices from day one.

## Repository Folder Contents
```
├── README.md                          # This file
├── aws_waf_caf_assessment.pdf        # Complete assessment report
└── architecture_diagram.jpg           # AWS architecture diagram
```

## Approach & Methodology

### 1. Current State Analysis
Identified the existing two-tier architecture components or workloads (web servers, application code, authentication module, database, network infrastructure) and documented eight potential weaknesses including single points of failure, missing security controls (WAF, MFA), lack of scalability, no backup strategy, and resource waste during off-hours.

### 2. Well-Architected Framework Evaluation
Systematically evaluated the architecture against five WAF pillars (Operational Excellence, Security, Reliability, Performance Efficiency and Cost Optimization). For each pillar, identified one strength in the current existing architecture and one improvement area, then provide recommendations and a specific AWS service(s) to address the potential gaps. This structured approach ensured comprehensive coverage of architectural concerns.

### 3. Cloud Adoption Framework Readiness Assessment
Analyzed organizational readiness (using low or medium or high) beyond technology across six perspectives: Business, People, Governance, Platform, Security, and Operations. Identified that while technical migration is feasible, success requires parallel investments in skills development (training), governance frameworks (multi-account strategy, cost management), and operational transformation (monitoring, automation, incident response).

### 4. Architecture Design
Designed a cloud-native solution implementing:
- **Multi-AZ deployment** across 2 Availability Zones eliminating single points of failure
- **Auto Scaling** with scheduled policies addressing scalability and off-hour waste
- **Defense-in-depth security** with CloudFront, WAF, encryption at rest/in transit, Secrets Manager to oversee passwords and API/Database keys
- **RDS Multi-AZ** with automated backups solving database reliability gaps
- **Infrastructure as Code** and **CI/CD pipelines** enabling operational excellence
- **Cost optimization** through dynamic scaling, right-sizing, and resource scheduling

## Key Architectural Decisions

**Multi-AZ over Single-AZ:** Prioritized availability (99.99% SLA) while minimizing cost to eliminate critical single points of failure identified in current architecture.

**Managed Services Strategy:** Selected RDS over self-managed EC2 databases and ALB over manual load balancing solutions to reduce operational overhead and leverage AWS expertise in reliability/security.

**Scheduled Scaling:** Implemented time-based scaling policies to automatically reduce capacity during off-hours, directly reducing costs while maintaining optimal performance during business hours.

**CloudFront + WAF:** Combined global CDN with web application firewall to simultaneously solve performance (latency for distant users) and security (missing WAF protection) gaps with a single integrated solution.

## Tools & Resources

- **AWS Documentation:** Well-Architected Framework, Cloud Adoption Framework, service-specific guides
- **Design Tools:** Draw.io with AWS architecture icons


## Reflection

This exercise reinforced that cloud migration success requires systematic evaluation across technical and organizational dimensions. The Well-Architected Framework provided a structured technical lens, while the Cloud Adoption Framework revealed that people readiness, governance mechanisms, and organizational alignment are equally critical. The interconnection between frameworks became evident—CI/CD pipelines (CAF Platform perspective) enable Operational Excellence (WAF pillar), while Cloud Financial Management (CAF Governance perspective) supports Cost Optimization (WAF pillar). Real migration success demands addressing technical architecture, people capabilities, governance frameworks, and business outcomes holistically.

---

**Project Date:** February 2026  
**Assessment Report:** See `aws_waf_caf_assessment.pdf` for detailed analysis