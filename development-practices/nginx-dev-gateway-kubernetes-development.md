# nginx-dev-gateway: Simplifying Kubernetes Microservice Development with an API Gateway

## Overview

[nginx-dev-gateway](https://github.com/nathanfox/nginx-dev-gateway) is a lightweight, namespace-aware NGINX-based API gateway designed specifically for Kubernetes development environments. It solves a common pain point for developers working with microservices: managing multiple `kubectl port-forward` commands to access different services during development.

## The Problem

When developing microservices in Kubernetes, accessing multiple services typically requires:
- Opening separate terminal windows for each service
- Running individual `kubectl port-forward` commands for each service
- Tracking different local ports for each service
- Manually switching between debug and stable versions of services

This becomes particularly cumbersome when working with complex microservice architectures where you need to access 5, 10, or more services simultaneously. In team environments, this tool enables developers to utilize their own Kubernetes namespace, working on their services independently while seamlessly accessing shared services from other namespaces.

## The Solution

nginx-dev-gateway provides a single API gateway entry point for all your microservices. Instead of managing multiple port-forwards, you run one `kubectl port-forward` to the gateway, and it routes requests to your services based on URL paths.

Key capabilities:
- **Path-based routing**: Access services via `/api/servicename`
- **Namespace-aware**: Automatically routes to services in the correct namespace
- **WebSocket support**: Full bidirectional communication for real-time features
- **ConfigMap configuration**: Update routing dynamically without redeploying
- **Minimal overhead**: Alpine-based image (~15MB) with low resource usage

## Use Cases

### Team Development with Per-Developer Namespaces
Each developer deploys the gateway in their own namespace, allowing them to work on specific microservices while routing requests to shared services in other namespaces. This enables isolated debugging without interfering with teammates' work.

### Local Development
Access all your microservices through `localhost:8080` instead of remembering multiple ports.

### Debug vs. Stable Routing
Easily switch between debugging a specific service while using stable versions of others by updating the gateway configuration.

### Pod Debugging
When combined with remote debugging tools like [k8s-vscode-remote-debug](https://github.com/nathanfox/k8s-vscode-remote-debug), the gateway simplifies access to debug-enabled pods, allowing you to route traffic to instrumented services while maintaining normal access to others.

## Getting Started

For installation, configuration, and detailed usage instructions, see the [nginx-dev-gateway repository](https://github.com/nathanfox/nginx-dev-gateway). The README includes:
- Deployment manifests
- Configuration examples
- Routing patterns
- Troubleshooting guidance

## Why Not a Standard API Gateway?

Unlike production API gateways (Kong, Ambassador, Istio), nginx-dev-gateway is purpose-built for development workflows:

- **No cluster-admin required**: Runs in your namespace with no special permissions
- **Lightweight and fast**: ~15MB image, starts in seconds
- **Developer-centric**: ConfigMap-based routing means quick iteration without complex UIs
- **Namespace-aware by default**: Designed for per-developer namespace isolation
- **Zero dependencies**: Just NGINX, no databases or control planes

Production API gateways are optimized for traffic management, security policies, and observability at scale. nginx-dev-gateway trades those features for simplicity, speed, and ease of setup—perfect for development but not intended for production use.

## Technical Details

nginx-dev-gateway is built on standard Kubernetes primitives:
- Deployment for the NGINX gateway
- ConfigMap for routing configuration
- Service for internal cluster access

The gateway runs in your development namespace and requires no cluster-level permissions, making it safe to deploy in shared development environments.