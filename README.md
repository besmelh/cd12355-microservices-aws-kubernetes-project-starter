# Coworking Space Analytics Service Deployment

## Overview

This project deploys a Python Flask analytics API to AWS EKS using a containerized microservices architecture. The service provides business intelligence on coworking space usage patterns through REST endpoints.

## Technologies & Architecture

- **Container Runtime**: Docker with Python 3.8 base image
- **Container Registry**: AWS ECR for image storage and versioning
- **Orchestration**: Amazon EKS (Kubernetes 1.32) with 2-node cluster
- **Database**: PostgreSQL deployed via Helm (Bitnami chart) within the cluster
- **CI/CD**: AWS CodeBuild with buildspec.yml for automated builds
- **Monitoring**: Application logs accessible via kubectl

## Deployment Architecture

The application runs as a Kubernetes Deployment with 1 replica behind a LoadBalancer service. Database credentials are managed through Kubernetes Secrets, while non-sensitive configuration uses ConfigMaps. The PostgreSQL database runs as a StatefulSet within the same cluster, accessible at `mypostgres-postgresql.default.svc.cluster.local:5432`.

## Configuration Files

- `analytics/Dockerfile`: Multi-stage build with dependency caching
- `buildspec.yml`: CodeBuild pipeline for automated ECR pushes
- `deployment/configmap.yaml`: Environment variables and database connection details
- `deployment/coworking.yaml`: Kubernetes Service (LoadBalancer) and Deployment manifests

## Releasing New Builds

1. **Code changes**: Push to GitHub main branch
2. **Build**: CodeBuild automatically triggers on commit, builds Docker image, tags with commit hash, and pushes to ECR
3. **Deploy**: Update `deployment/coworking.yaml` with new image tag and run `kubectl apply -f deployment/coworking.yaml`
4. **Verify**: Check rollout status with `kubectl rollout status deployment/coworking`

Alternatively, update the image directly: `kubectl set image deployment/coworking coworking=<ECR_URI>:<NEW_TAG>` for zero-downtime rolling updates.

## API Endpoints

- `/health_check`: Application health status
- `/readiness_check`: Database connectivity verification
- `/api/reports/daily_usage`: User check-ins grouped by date
- `/api/reports/user_visits`: Check-in counts per user
