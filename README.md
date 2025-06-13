# Agent Zero Docker Setup with GPU Support

This repository contains a complete Docker setup for running [Agent Zero](https://github.com/frdel/agent-zero) with GPU support (NVIDIA CUDA or AMD ROCm) in a containerized environment.

## 🚀 Features

- **GPU Support**: Full NVIDIA CUDA or AMD ROCm GPU acceleration for AI models
- **Complete System Persistence**: Entire system state persists across container rebuilds using Docker volumes
- **SSH Access**: Internal SSH for agent code execution
- **Remote Function Calls (RFC)**: Proper SSH configuration for agent self-interaction
- **Web Interface**: Clean web UI accessible from host machine

## 📋 Prerequisites

### For NVIDIA GPU Support

- Docker and Docker Compose installed
- NVIDIA Drivers installed
- NVIDIA Container Toolkit installed
- Git (for version control)

### For AMD GPU Support

- Docker and Docker Compose installed
- AMD ROCm drivers installed
- Git (for version control)

## 🔧 Configuration

### Docker Compose Services

Choose the appropriate docker-compose file for your GPU:

- **NVIDIA**: Use `docker-compose_nvidia.yaml`
- **AMD**: Use `docker-compose_amd.yaml`

Both configurations define:

- **Ports**:
  - `55080:5000` - Web UI access
  - `55022:22` - SSH access (internal use)
- **GPU Access**: Full GPU allocation (NVIDIA or AMD)
- **Persistent Storage**: Complete system persistence using Docker named volumes

### Environment Variables

Key environment variables in both docker-compose files:

- `HOST=0.0.0.0` - Bind to all interfaces
- `PORT=5000` - Internal HTTP port
- `BASE_URL=http://localhost:5000` - Internal base URL for RFC

## 🚀 Quick Start

1. **Clone this repository**:

   ```bash
   git clone <your-repo-url>
   cd agent-zero-docker
   ```

2. **Choose your GPU type and build the container**:

   **For NVIDIA GPUs:**

   ```bash
   docker compose -f docker-compose_nvidia.yaml up --build
   ```

   **For AMD GPUs:**

   ```bash
   docker compose -f docker-compose_amd.yaml up --build
   ```

3. **Access the web interface**:
   Open your browser and navigate to:

   ```
   http://localhost:55080
   ```

4. **Configure Agent Zero**:
   - Set up your API keys in the Settings
   - Configure RFC settings:
     - RFC Destination URL: `http://localhost`
     - RFC Password: Choose a secure password
     - RFC HTTP port: `5000`
     - RFC SSH port: `22`

**Note**: All your configuration, installed packages, and system changes will automatically persist using Docker volumes!

## ⚙️ Detailed Setup

### First-Time Configuration

1. **Access Settings**: Click the settings icon in the web interface
2. **Configure Models**: Set up your preferred AI model providers (OpenAI, Ollama, etc.)
3. **Set RFC Password**: This password enables agent self-interaction via SSH
4. **Save Settings**: All settings will persist automatically in the Docker volume

### System Persistence

All changes are automatically persisted using Docker named volumes:

- **Location**: `/var/lib/docker/volumes/agent-zero-docker_agent-zero-root/_data`
- **What's persisted**: Complete `/root` directory including conda environment, installed packages, configuration files, agent-zero code, and any modifications you make
- **Access**: You can browse the persisted data with `ls -la /var/lib/docker/volumes/agent-zero-docker_agent-zero-root/_data`

### SSH Configuration

The container automatically configures SSH with:

- Root user enabled
- Password authentication enabled
- Host key verification disabled for localhost
- Temporary password `temp123` (overridden by Agent Zero at runtime)

### GPU Verification

**For NVIDIA GPUs:**

```bash
# Execute into the running container
docker exec -it agent-zero-cuda bash

# Check GPU availability
nvidia-smi

# Test with Python
python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}')"
```

**For AMD GPUs:**

```bash
# Execute into the running container
docker exec -it agent-zero-amd bash

# Check GPU availability
rocm-smi

# Test with Python
python -c "import torch; print(f'ROCm available: {torch.cuda.is_available()}')"
```

## 🔒 Security Considerations

- **Local Use Only**: This setup is designed for local development
- **SSH Access**: SSH is configured for internal agent use only
- **Password Security**: Change default passwords in production
- **Firewall**: Consider firewall rules if exposing ports externally

## 🛠️ Troubleshooting

### Common Issues

1. **Container exits with "conda: command not found"**:

   **Cause**: This usually happens if there's a volume mounting issue or the container wasn't built properly.

   **Solution**:

   ```bash
   # Stop and rebuild completely
   docker compose -f docker-compose_amd.yaml down  # or nvidia
   docker compose -f docker-compose_amd.yaml up --build
   ```

2. **Authentication Failed (SSH)**:

   - Ensure RFC password is set in the web interface
   - Check that SSH service is running in the container

3. **Bad Request Errors in Logs**:

   - These are harmless SSL handshake attempts to HTTP server
   - Always use `http://` (not `https://`) when accessing the interface

4. **GPU Not Available (NVIDIA)**:

   - Verify NVIDIA Docker runtime installation
   - Check GPU drivers on host system
   - Ensure NVIDIA Container Toolkit is properly installed

5. **GPU Not Available (AMD)**:

   - Verify ROCm drivers are installed on host system
   - Check that `/dev/kfd` and `/dev/dri` devices are accessible
   - Ensure user is in the `video` group on host system

6. **Configuration Not Persisting**:

   - Docker volumes should handle persistence automatically
   - If issues persist, check Docker volume status: `docker volume ls`
   - Inspect volume location: `docker volume inspect agent-zero-docker_agent-zero-root`

7. **First Time Boot**:
   Note that on the first time Agent Zero is indexing and so it might take a long time until it responds the first time.

### Log Access

View container logs:

```bash
# Follow logs in real-time
docker compose logs -f

# View specific service logs
docker compose logs app
```

### Container Access

Execute commands in the running container:

**For NVIDIA containers:**

```bash
# Get shell access
docker exec -it agent-zero-cuda bash

# Test SSH connection (internal)
docker exec -it agent-zero-cuda ssh root@localhost -p 22
```

**For AMD containers:**

```bash
# Get shell access
docker exec -it agent-zero-amd bash

# Test SSH connection (internal)
docker exec -it agent-zero-amd ssh root@localhost -p 22
```

## 🔄 Development Workflow

### Rebuilding

When making changes to the Dockerfile:

**For NVIDIA:**

```bash
# Stop and rebuild
docker compose -f docker-compose_nvidia.yaml down
docker compose -f docker-compose_nvidia.yaml up --build
```

**For AMD:**

```bash
# Stop and rebuild
docker compose -f docker-compose_amd.yaml down
docker compose -f docker-compose_amd.yaml up --build
```

### Updating Agent Zero

To update to the latest Agent Zero version:

**For NVIDIA:**

```bash
# Rebuild with fresh clone
docker compose -f docker-compose_nvidia.yaml down
docker compose -f docker-compose_nvidia.yaml build --no-cache
docker compose -f docker-compose_nvidia.yaml up
```

**For AMD:**

```bash
# Rebuild with fresh clone
docker compose -f docker-compose_amd.yaml down
docker compose -f docker-compose_amd.yaml build --no-cache
docker compose -f docker-compose_amd.yaml up
```

### Backup Configuration

Backup your persistent data:

```bash
# Create backup of the entire persisted system
tar -czf agent-zero-backup.tar.gz -C /var/lib/docker/volumes/agent-zero-docker_agent-zero-root/_data .

# Alternative: Backup using Docker volume
docker run --rm -v agent-zero-docker_agent-zero-root:/data -v $(pwd):/backup ubuntu tar czf /backup/agent-zero-backup.tar.gz -C /data .
```

Restore backup:

```bash
# Stop container first
docker compose -f docker-compose_amd.yaml down  # or nvidia

# Remove old volume
docker volume rm agent-zero-docker_agent-zero-root

# Recreate and restore
docker volume create agent-zero-docker_agent-zero-root
docker run --rm -v agent-zero-docker_agent-zero-root:/data -v $(pwd):/backup ubuntu tar xzf /backup/agent-zero-backup.tar.gz -C /data

# Start container
docker compose -f docker-compose_amd.yaml up  # or nvidia
```

## 📚 Additional Resources

- [Agent Zero Repository](https://github.com/frdel/agent-zero)
- [Agent Zero Documentation](https://github.com/frdel/agent-zero/blob/main/docs/README.md)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [NVIDIA Docker Documentation](https://github.com/NVIDIA/nvidia-docker)
- [AMD ROCm Documentation](https://rocm.docs.amd.com/en/latest/)

## 🤝 Contributing

1. Fork this repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly with both NVIDIA and AMD configurations if possible
5. Submit a pull request

## 📄 License

This project follows the same license as Agent Zero. See the [Agent Zero repository](https://github.com/frdel/agent-zero) for license details.

## 🆘 Support

If you encounter issues:

1. **Check Docker volume status**:

   ```bash
   # List volumes
   docker volume ls | grep agent-zero

   # Inspect volume location and details
   docker volume inspect agent-zero-docker_agent-zero-root

   # Browse persisted data
   ls -la /var/lib/docker/volumes/agent-zero-docker_agent-zero-root/_data
   ```

2. Check the troubleshooting section above
3. Review Agent Zero's official documentation
4. Check Docker and GPU driver installation (NVIDIA Container Toolkit or ROCm)
5. Open an issue in this repository with detailed logs and specify your GPU type (NVIDIA/AMD)
