# OSRM Server Git Setup Guide

## Step 1: Initialize Git Repository

```bash
cd ~/osrm-server
git init
git remote add origin https://github.com/SeeSense-AK/osrm-server.git
```

## Step 2: Create .gitignore

Create a `.gitignore` file to exclude large data files:

```gitignore
# OSM data files (too large for Git)
*.osm.pbf
*.osrm
*.osrm.*

# Docker volumes and build artifacts
*.egg-info/
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# IDE and editor files
.vscode/
.idea/
*.swp
*.swo
*~

# OS generated files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Logs
*.log
logs/

# Environment variables
.env
.env.local
.env.production
```

## Step 3: Create Essential Configuration Files

### docker-compose.yml
```yaml
version: '3.8'

services:
  osrm-finland:
    image: ghcr.io/project-osrm/osrm-backend
    container_name: osrm-finland
    ports:
      - "5001:5000"
    volumes:
      - ./finland:/data
      - ./osrm-profiles:/profiles
    command: osrm-routed --algorithm mld --max-viaroute-size 10000 --max-table-size 10000 /data/finland-latest.osrm
    restart: unless-stopped

  osrm-ireland:
    image: ghcr.io/project-osrm/osrm-backend
    container_name: osrm-ireland
    ports:
      - "5002:5000"
    volumes:
      - ./ireland:/data
      - ./osrm-profiles:/profiles
    command: osrm-routed --algorithm mld --max-viaroute-size 10000 --max-table-size 10000 /data/ireland-and-northern-ireland-latest.osrm
    restart: unless-stopped

  osrm-sydney:
    image: ghcr.io/project-osrm/osrm-backend
    container_name: osrm-sydney
    ports:
      - "5003:5000"
    volumes:
      - ./sydney:/data
      - ./osrm-profiles:/profiles
    command: osrm-routed --algorithm mld --max-viaroute-size 10000 --max-table-size 10000 /data/sydney-latest.osrm
    restart: unless-stopped

  osrm-wales:
    image: ghcr.io/project-osrm/osrm-backend
    container_name: osrm-wales
    ports:
      - "5004:5000"
    volumes:
      - ./wales:/data
      - ./osrm-profiles:/profiles
    command: osrm-routed --algorithm mld --max-viaroute-size 10000 --max-table-size 10000 /data/wales-latest.osrm
    restart: unless-stopped

  osrm-england:
    image: ghcr.io/project-osrm/osrm-backend:v6.0.0
    container_name: osrm-england
    ports:
      - "5005:5000"
    volumes:
      - ./england:/data
      - ./osrm-profiles:/profiles
    command: osrm-routed --algorithm mld --max-viaroute-size 10000 --max-table-size 10000 /data/england-latest.osrm
    restart: unless-stopped
```

### README.md
```markdown
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

```
IoT Devices → Flespi → Raw JSON → CSV Conversion → Python Interpolation Script → OSRM Server → Route Snapping
```

## Setup Instructions

### Prerequisites
- Docker and Docker Compose
- OSM data files for each region
- Python environment for interpolation scripts

### 1. Download OSM Data

Download the required OSM data files and place them in their respective directories:

# Create directories
mkdir -p {finland,ireland,sydney,wales,england}

# Download data files (example for Finland)
cd finland
wget https://download.geofabrik.de/europe/finland-latest.osm.pbf
cd ..

# Repeat for other regions...
```

### 2. Prepare OSRM Data

Process the OSM data for each region:

```bash
# For each region, run these commands (example for Finland)
docker run -t -v "${PWD}/finland:/data" -v "${PWD}/osrm-profiles:/profiles" ghcr.io/project-osrm/osrm-backend osrm-extract -p /profiles/car.lua /data/finland-latest.osm.pbf
docker run -t -v "${PWD}/finland:/data" ghcr.io/project-osrm/osrm-backend osrm-partition /data/finland-latest.osrm
docker run -t -v "${PWD}/finland:/data" ghcr.io/project-osrm/osrm-backend osrm-customize /data/finland-latest.osrm
```

### 3. Start Services

```bash
docker-compose up -d
```

### 4. Verify Setup

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

## Usage

The OSRM servers are used by our Python interpolation scripts to:
1. Snap GPS points to roads using the `/nearest` endpoint
2. Generate routes between points using the `/route` endpoint
3. Build plausible vehicle paths from raw IoT data

## Configuration

- **Profiles**: Custom routing profiles are stored in `osrm-profiles/`
- **Ports**: Each region runs on a dedicated port (5001-5005)
- **Algorithm**: Using MLD (Multi-Level Dijkstra) for optimal performance

## Maintenance

### Updating OSM Data

To update the OSM data:

1. Download new OSM files
2. Stop the containers: `docker-compose down`
3. Re-process the data (extract, partition, customize)
4. Restart: `docker-compose up -d`

### Monitoring

Check container status:
```bash
docker-compose ps
docker-compose logs [service-name]
```

## Data Sources

- Finland: [Geofabrik Finland](https://download.geofabrik.de/europe/finland.html)
- Ireland: [Geofabrik Ireland](https://download.geofabrik.de/europe/ireland-and-northern-ireland.html)
- Sydney: [Geofabrik Australia](https://download.geofabrik.de/australia-oceania.html)
- Wales: [Geofabrik Wales](https://download.geofabrik.de/europe/great-britain/wales.html)
- England: [Geofabrik England](https://download.geofabrik.de/europe/great-britain/england.html)
```

### scripts/setup.sh
```bash
#!/bin/bash

# OSRM Server Setup Script
# This script sets up OSRM servers for multiple regions

set -e

REGIONS=("finland" "ireland" "sydney" "wales" "england")
PROFILE="car.lua"

echo "🚀 Setting up OSRM servers..."

# Function to process a region
process_region() {
    local region=$1
    local osm_file=""
    
    case $region in
        "finland") osm_file="finland-latest.osm.pbf" ;;
        "ireland") osm_file="ireland-and-northern-ireland-latest.osm.pbf" ;;
        "sydney") osm_file="sydney-latest.osm.pbf" ;;
        "wales") osm_file="wales-latest.osm.pbf" ;;
        "england") osm_file="england-latest.osm.pbf" ;;
    esac
    
    echo "📍 Processing $region..."
    
    # Extract
    echo "  - Extracting..."
    docker run --rm -t \
        -v "${PWD}/${region}:/data" \
        -v "${PWD}/osrm-profiles:/profiles" \
        ghcr.io/project-osrm/osrm-backend \
        osrm-extract -p /profiles/${PROFILE} /data/${osm_file}
    
    # Partition
    echo "  - Partitioning..."
    docker run --rm -t \
        -v "${PWD}/${region}:/data" \
        ghcr.io/project-osrm/osrm-backend \
        osrm-partition /data/${osm_file%.osm.pbf}.osrm
    
    # Customize
    echo "  - Customizing..."
    docker run --rm -t \
        -v "${PWD}/${region}:/data" \
        ghcr.io/project-osrm/osrm-backend \
        osrm-customize /data/${osm_file%.osm.pbf}.osrm
    
    echo "  ✅ $region processing complete"
}

# Check if OSM files exist
for region in "${REGIONS[@]}"; do
    if [ ! -d "$region" ]; then
        echo "❌ Directory $region not found. Please ensure OSM data is downloaded."
        exit 1
    fi
done

# Process all regions
for region in "${REGIONS[@]}"; do
    process_region $region
done

echo "🎉 All regions processed successfully!"
echo "Run 'docker-compose up -d' to start the servers."
```

### scripts/test-servers.sh
```bash
#!/bin/bash

# Test script for all OSRM servers

echo "🧪 Testing OSRM servers..."

# Test coordinates for each region
declare -A TEST_COORDS=(
    ["5001"]="24.9384,60.1699;24.9570,60.1699"  # Finland
    ["5002"]="-6.2603,53.3498;-6.2493,53.3506"  # Ireland
    ["5003"]="151.2093,-33.8688;151.2173,-33.8678"  # Sydney
    ["5004"]="-3.1791,51.4816;-3.1681,51.4826"  # Wales
    ["5005"]="-0.1276,51.5074;-0.1176,51.5084"  # England
)

declare -A REGION_NAMES=(
    ["5001"]="Finland"
    ["5002"]="Ireland" 
    ["5003"]="Sydney"
    ["5004"]="Wales"
    ["5005"]="England"
)

for port in "${!TEST_COORDS[@]}"; do
    echo "Testing ${REGION_NAMES[$port]} (port $port)..."
    
    response=$(curl -s -w "%{http_code}" "http://localhost:$port/route/v1/driving/${TEST_COORDS[$port]}?overview=false")
    http_code="${response: -3}"
    
    if [ "$http_code" -eq 200 ]; then
        echo "  ✅ ${REGION_NAMES[$port]} server is working"
    else
        echo "  ❌ ${REGION_NAMES[$port]} server failed (HTTP $http_code)"
    fi
done

echo "🏁 Testing complete!"
```

## Step 4: Execute Setup Commands

Run these commands on your Mac Mini:

```bash
# Navigate to your osrm-server directory
cd ~/osrm-server

# Create the files above
# (Copy the content from the artifact into respective files)

# Make scripts executable
chmod +x scripts/*.sh

# Initialize git repository
git init
git remote add origin https://github.com/SeeSense-AK/osrm-server.git

# Add files to git
git add .
git commit -m "Initial OSRM server setup

- Added Docker Compose configuration for 5 regions
- Added setup and testing scripts  
- Added comprehensive documentation
- Configured gitignore for large data files"

# Push to GitHub
git push -u origin main
```

## Step 5: Optional - Create Data Download Script

If you want to include a script for downloading OSM data:

### scripts/download-data.sh
```bash
#!/bin/bash

# Download OSM data for all regions

echo "📥 Downloading OSM data files..."

# Create directories
mkdir -p finland ireland sydney wales england

# Download files
echo "Downloading Finland..."
cd finland && wget -N https://download.geofabrik.de/europe/finland-latest.osm.pbf && cd ..

echo "Downloading Ireland..."
cd ireland && wget -N https://download.geofabrik.de/europe/ireland-and-northern-ireland-latest.osm.pbf && cd ..

echo "Downloading Wales..."  
cd wales && wget -N https://download.geofabrik.de/europe/great-britain/wales-latest.osm.pbf && cd ..

echo "Downloading England..."
cd england && wget -N https://download.geofabrik.de/europe/great-britain/england-latest.osm.pbf && cd ..

echo "Downloading Sydney (Australia excerpt)..."
cd sydney && wget -N https://download.geofabrik.de/australia-oceania/australia-latest.osm.pbf && cd ..

echo "✅ All downloads complete!"
```

This setup will give you a professional, maintainable repository that others can easily clone and set up. The large OSM data files won't be stored in Git, but the instructions for obtaining them are clear.
