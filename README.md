# OSRM Server Setup

This repository contains the configuration and setup for our OSRM (Open Source Routing Machine) server infrastructure used for processing IoT device GPS data.

## Overview

Our setup includes OSRM servers for multiple regions:
- Finland (Port 5001)
- Ireland (Port 5002) 
- Sydney (Port 5003)
- Wales (Port 5004)
- England (Port 5005)

## Architecture

IoT Devices → Flespi → Raw JSON → CSV Conversion → Python Interpolation Script → OSRM Server → Route Snapping

## Setup Instructions

### Prerequisites
- Docker and Docker Compose
- OSM data files for each region
- Python environment for interpolation scripts

### 1. Download OSM Data

Download the required OSM data files and place them in their respective directories:

```bash
# Create directories
mkdir -p {finland,ireland,sydney,wales,england}

# Download data files (example for Finland)
cd finland
wget https://download.geofabrik.de/europe/finland-latest.osm.pbf
cd ..

# Repeat for other regions...

# For each region, run these commands (example for Finland)
docker run -t -v "${PWD}/finland:/data" -v "${PWD}/osrm-profiles:/profiles" ghcr.io/project-osrm/osrm-backend osrm-extract -p /profiles/car.lua /data/finland-latest.osm.pbf
docker run -t -v "${PWD}/finland:/data" ghcr.io/project-osrm/osrm-backend osrm-partition /data/finland-latest.osrm
docker run -t -v "${PWD}/finland:/data" ghcr.io/project-osrm/osrm-backend osrm-customize /data/finland-latest.osrm

docker-compose up -d
