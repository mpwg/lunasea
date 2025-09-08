# LunaSea Project Documentation

![LunaSea Logo](https://raw.githubusercontent.com/JagandeepBrar/LunaSea/master/lunasea/assets/images/branding_logo.png)

> ⚠️ **This project is no longer being actively maintained and this repository is archived.** ⚠️

## Overview

LunaSea is a fully featured, open source self-hosted controller focused on giving you a seamless experience between all of your self-hosted media software remotely on your devices. This wiki serves as comprehensive documentation for the entire LunaSea project ecosystem.

## Project Information

- **Version**: 11.0.0+1
- **License**: GNU General Public License v3
- **Platform**: Cross-platform Flutter application
- **Repository Type**: Mono-repository
- **Status**: Archived (no longer actively maintained)

## Repository Structure

This is a mono-repository containing four main components:

| Component | Description | Technology |
|-----------|-------------|------------|
| `lunasea/` | Main Flutter application | Dart/Flutter 3.0+ |
| `lunasea-cloud-functions/` | Firebase cloud functions | Node.js |
| `lunasea-docs/` | Documentation site | GitBook |
| `lunasea-notification-service/` | Notification service | TypeScript |

## Supported Services

LunaSea provides remote control functionality for the following self-hosted applications:

### Media Management
- **[Radarr](https://github.com/radarr/radarr)** - Movie collection manager
- **[Sonarr](https://github.com/sonarr/sonarr)** - TV series collection manager  
- **[Lidarr](https://github.com/lidarr/lidarr)** - Music collection manager

### Download Clients
- **[NZBGet](https://github.com/nzbget/nzbget)** - Usenet downloader
- **[SABnzbd](https://github.com/sabnzbd/sabnzbd)** - Usenet downloader

### Indexers & Search
- **[Newznab](https://newznab.readthedocs.io/en/latest/misc/api/)** - Indexer API
- **[NZBHydra2](https://github.com/theotherp/nzbhydra2)** - Meta search engine

### Monitoring & Utilities
- **[Tautulli](https://github.com/Tautulli/Tautulli)** - Plex monitoring and analytics
- **[Wake on LAN](https://en.wikipedia.org/wiki/Wake-on-LAN)** - Remote system wake

## Key Features

- **Cross-Platform**: Android, iOS, Windows, macOS, Linux, Web
- **Push Notifications**: Webhook-based notifications
- **Multiple Profiles**: Support for multiple instances
- **Backup & Restore**: Configuration backup/restore
- **AMOLED Theme**: Dark theme optimized for OLED displays
- **Localization**: Multi-language support

## Documentation Sections

### Development
- [Development Setup](Development-Setup) - Prerequisites and environment setup
- [Architecture Overview](Architecture-Overview) - Application structure and design patterns
- [Build Process](Build-Process) - Building and deployment
- [Code Generation](Code-Generation) - Generated code and tooling

### Technical Reference
- [Database Schema](Database-Schema) - Hive database models and structure
- [API Documentation](API-Documentation) - Service integrations and endpoints
- [Module System](Module-System) - Modular architecture details
- [CI/CD Pipeline](CICD-Pipeline) - Automated build and deployment

### User Guide
- [Installation Guide](Installation-Guide) - Platform-specific installation
- [Configuration](Configuration) - Setting up services and profiles
- [Troubleshooting](Troubleshooting) - Common issues and solutions

## Important Notes

> **Note**: LunaSea is purely a remote control application. It does not offer any functionality without software installed on a server/computer.

> **Archive Status**: As this project is archived, this documentation serves as a comprehensive knowledge base for future reference, educational purposes, or for those who wish to fork and continue development.

## Contact & Links

- **Website**: [lunasea.app](https://www.lunasea.app)
- **Email**: [hello@lunasea.app](mailto:hello@lunasea.app)
- **Original Repository**: [JagandeepBrar/LunaSea](https://github.com/JagandeepBrar/LunaSea)