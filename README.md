# Surge Docker Setup

This repository contains a Dockerfile and docker-compose configuration for running [Surge](https://github.com/surge-downloader/surge) - a blazing fast download manager written in Go - in server mode.

## Features

- **Always up-to-date**: Automatically downloads the latest Surge release from GitHub
- Multi-architecture support (amd64 and arm64)
- Persistent configuration and downloads
- Health checks
- Easy deployment with docker-compose

## Quick Start

### Using Docker Compose (Recommended)

1. **Build and start the container:**
   ```bash
   docker compose up -d
   ```

2. **Get the API token:**
   ```bash
   docker compose exec surge surge token
   ```
   
   Save this token - you'll need it to authenticate API requests and connect remotely.

3. **Check server status:**
   ```bash
   docker compose exec surge surge server status
   ```

4. **View logs:**
   ```bash
   docker compose logs -f surge
   ```

### Using Docker CLI

1. **Build the image:**
   ```bash
   docker build -t surge:latest .
   ```

2. **Run the container:**
   ```bash
   docker run -d \
     --name surge-server \
     -p 1700:1700 \
     -v $(pwd)/downloads:/downloads \
     -v $(pwd)/surge-config:/root/.surge \
     surge:latest
   ```

3. **Get the API token:**
   ```bash
   docker exec surge-server surge token
   ```

## Usage

### Adding Downloads

You can add downloads to the server using:

1. **Remote TUI connection:**
   ```bash
   surge connect localhost:1700 --token <your-token>
   ```

2. **HTTP API:**
   ```bash
   curl -X POST http://localhost:1700/api/download \
     -H "Authorization: Bearer <your-token>" \
     -H "Content-Type: application/json" \
     -d '{"url": "https://example.com/file.zip"}'
   ```

3. **Browser Extension:**
   Install the [Surge browser extension](https://github.com/surge-downloader/surge#browser-extension) and configure it to connect to `http://localhost:1700` with your token.

### Managing the Container

- **Stop the server:**
  ```bash
  docker compose down
  ```

- **Restart the server:**
  ```bash
  docker compose restart
  ```

- **Update to latest Surge version:**
  ```bash
  make build-latest
  docker compose up -d
  ```
  
  Or without make:
  ```bash
  docker compose build --no-cache
  docker compose up -d
  ```

- **View container logs:**
  ```bash
  docker compose logs -f
  ```

- **Access downloads:**
  Downloads are saved to the `./downloads` directory on your host machine.

## Configuration

The docker-compose setup includes two volumes:

- `./downloads` - Where downloaded files are stored
- `./surge-config` - Stores surge configuration and the API token

You can modify these paths in the `compose.yml` file to suit your needs.

### Version Pinning (Optional)

By default, the Dockerfile fetches the latest Surge release. If you want to pin to a specific version:

1. Edit the `Dockerfile` and replace the dynamic version fetch with a hardcoded version:
   ```dockerfile
   # Replace the curl/jq lines with:
   SURGE_VERSION="0.6.5"
   ```

2. Rebuild the image:
   ```bash
   docker compose build --no-cache
   ```

## Port Configuration

By default, Surge runs on port 1700. If you need to change this:

1. Update the port mapping in `compose.yml`:
   ```yaml
   ports:
     - "8080:1700"  # Maps host port 8080 to container port 1700
   ```

2. Restart the container:
   ```bash
   docker compose up -d
   ```

## Building for Specific Architecture

To build for a specific architecture:

```bash
# For amd64
docker build --platform linux/amd64 -t surge:amd64 .

# For arm64
docker build --platform linux/arm64 -t surge:arm64 .
```

## Troubleshooting

### Container won't start

Check the logs:
```bash
docker compose logs surge
```

### Can't connect to server

1. Verify the container is running:
   ```bash
   docker compose ps
   ```

2. Check if the port is accessible:
   ```bash
   curl http://localhost:1700/health
   ```

3. Ensure you're using the correct token

### Downloads not persisting

Ensure the volumes are properly mounted and the `downloads` directory exists with proper permissions.

## Additional Resources

- [Surge GitHub Repository](https://github.com/surge-downloader/surge)
- [Surge Documentation](https://github.com/surge-downloader/surge/tree/main/docs)
- [Surge Settings Guide](https://github.com/surge-downloader/surge/blob/main/docs/SETTINGS.md)
