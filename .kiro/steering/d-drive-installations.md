---
inclusion: always
---
# E Drive Installation Rule

All installations, downloads, and data storage must use the E: drive, not C:.

- The C: drive has very limited space
- Docker data, node_modules, package caches, and any other large files should go on E:
- When installing software via winget or other package managers, specify E: drive paths when possible
- Docker Desktop data (images, containers, volumes) should be configured to use E: drive
- Any temp files or build artifacts should use E: drive paths
- D: drive no longer exists — use E: for everything
