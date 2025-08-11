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
```

2. Prepare OSRM Data
Process the OSM data for each region:

```bash
# For each region, run these commands (example for Finland)
docker run -t -v "${PWD}/finland:/data" -v "${PWD}/osrm-profiles:/profiles" ghcr.io/project-osrm/osrm-backend osrm-extract -p /profiles/car.lua /data/finland-latest.osm.pbf
docker run -t -v "${PWD}/finland:/data" ghcr.io/project-osrm/osrm-backend osrm-partition /data/finland-latest.osrm
docker run -t -v "${PWD}/finland:/data" ghcr.io/project-osrm/osrm-backend osrm-customize /data/finland-latest.osrm
```

3. Start Services

```bash
docker-compose up -d
```

4. Verify Setup
Test each server:

```bash
# Finland
curl "http://localhost:5001/route/v1/driving/24.9384,60.1699;24.9570,60.1699?overview=false"

# Ireland  
curl "http://localhost:5002/route/v1/driving/-6.2603,53.3498;-6.2493,53.3506?overview=false"

# Sydney
curl "http://localhost:5003/route/v1/driving/151.2093,-33.8688;151.2173,-33.8678?overview=false"

# Wales
curl "http://localhost:5004/route/v1/driving/-3.1791,51.4816;-3.1681,51.4826?overview=false"

# England
curl "http://localhost:5005/route/v1/driving/-0.1276,51.5074;-0.1176,51.5084?overview=false"
```
Usage
The OSRM servers are used by our Python interpolation scripts to:

- Snap GPS points to roads using the /nearest endpoint
- Generate routes between points using the /route endpoint
- Build plausible vehicle paths from raw IoT data

Configuration

- Profiles: Custom routing profiles are stored in osrm-profiles/
- Ports: Each region runs on a dedicated port (5001-5005)
- Algorithm: Using MLD (Multi-Level Dijkstra) for optimal performance

Maintenance
- Updating OSM Data
- To update the OSM data:

Download new OSM files
Stop the containers: docker-compose down
Re-process the data (extract, partition, customize)
Restart: docker-compose up -d
