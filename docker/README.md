# Docker Templates

This repository contains my Docker templates based on actual production configurations running on my servers. Rather than creating generic examples, I wanted to share real-world implementations - minus the sensitive information. I believe it's more valuable to see how others handle their production environments compared to basic, vanilla templates. While these configurations include specific elements like Traefik labels and network segregation that may not match your setup, they're meant to serve as inspiration rather than copy-paste solutions. For my fellow nerds out there (I salute you!), I've included a detailed explanation of my architecture to help you understand the structure and content.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Architecture Breakdown](#architecture-breakdown)
  - [Folder Structure](#folder-structure)
  - [Environment Variables](#environment-variables)
  - [Network Structure](#network-structure)
  - [Traefik Configuration](#traefik-configuration)
  - [Watchtower Configuration](#watchtower-configuration)
- [Security Considerations](#security-considerations)
- [Potential Improvements](#potential-improvements)

## Prerequisites
- Docker Engine and Docker Compose
- Traefik v2.x for reverse proxy
- Watchtower for automated container updates
- Basic understanding of Docker networking and container orchestration

## Architecture Breakdown

### Folder Structure
Each application resides in its dedicated folder named after the application. The minimal folder structure includes:
- `compose-<ApplicationName>.yml`: Docker Compose configuration
- `env-<ApplicationName>.env`: Application-specific environment variables

Additional configuration files may be present depending on the application's requirements.

Folders prefixed with *network* contain compose files that explicitly define networks using a whoami container as a placeholder.

### Environment Variables
Environment variables are organized in two levels:
- Global: `global-compose-variable.env` in the docker folder contains variables used across multiple containers (Docker versions, network names, etc.)
- Local: `local-compose-variable.env` in each application folder contains application-specific variables

### Network Structure
The infrastructure uses multiple segregated networks for enhanced security and organization:
- `traefik-network`: Connects containers to the Traefik reverse proxy
- `notification-network`: Enables container access to the mailrise notification service
- Additional networks as needed for specific applications

### Traefik Configuration
[Traefik](https://traefik.io/traefik/) serves as the reverse proxy. Container integration requires specific labels:

```yaml
labels:
# Basic Traefik Setup
- traefik.enable=true                                           # Enables Traefik for this container
- traefik.docker.network=traefik-network                        # Specifies which Docker network Traefik should use for this container

# HTTP Router Configuration
- traefik.http.routers.yourapplication-http.rule=Host(`applicationname.yourdomain.com`)     # Creates HTTP router for your domain
- traefik.http.routers.yourapplication-http.entrypoints=http                                # Sets the entry point to HTTP for this router

# HTTP to HTTPS Redirect
- traefik.http.middlewares.httptohttps.redirectscheme.scheme=https                 # Creates middleware to redirect HTTP to HTTPS
- traefik.http.routers.yourapplication-http.middlewares=httptohttps                # Applies the redirect middleware to HTTP router

# HTTPS Router Configuration
- traefik.http.routers.yourapplication-https.rule=Host(`applicationname.yourdomain.com`)   # Creates HTTPS router for the same domain
- traefik.http.routers.yourapplication-https.entrypoints=https                             # Sets the entry point to HTTPS for this router

# SSL/TLS Configuration
- traefik.http.routers.yourapplication-https.tls.certresolver=dnschallengeovh             # Specifies DNS challenge resolver for SSL certificates
- traefik.http.routers.yourapplication-https.tls.domains[0].main=*.yourdomain.com         # Sets up wildcard certificate for all subdomains
- traefik.http.routers.yourapplication-https.tls.domains[0].sans=yourdomain.com           # Adds main domain as Subject Alternative Name

# Service Configuration
- traefik.http.routers.yourapplication-https.service=yourapplication_server_service        # Links HTTPS router to the defined service
- traefik.http.services.yourapplication_server_service.loadbalancer.server.port=6767       # Specifies container port to forward requests to your application port
```

### Watchtower Configuration
[Watchtower](https://containrrr.dev/watchtower/) automates container updates. Enable monitoring for a container by adding:

```yaml
labels:
# Watchtower Configuration
- "com.centurylinklabs.watchtower.enable=true"                 # Enables Watchtower monitoring for this container - will check and apply updates automatically when available
```

## Security Considerations
- Network segregation for enhanced security
- Automatic HTTPS redirection
- SSL/TLS certificate management through Traefik
- Container update automation with Watchtower

## Potential Improvements
Infrastructure:
- Replace network containers with `docker network create` commands
- Upgrade to Traefik 3
- Implement more elegant HTTP to HTTPS redirection in Traefik 3
- Add health checks for all containers
- Implement resource constraints
