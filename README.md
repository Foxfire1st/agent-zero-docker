# Agent Zero Docker Setup with GPU Support

This repository contains a complete Docker setup for running [Agent Zero](https://github.com/frdel/agent-zero) with GPU support (NVIDIA CUDA or AMD ROCm) in a containerized environment.

## 🚀 Features

- **GPU Support**: Full NVIDIA CUDA or AMD ROCm GPU acceleration for AI models
- **Persistent Configuration**: Settings and data persist across container rebuilds
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
- **Persistent Volumes**: Configuration, logs, and environment variables

### Environment Variables

Key environment variables in both docker-compose files:
- `HOST=0.0.0.0` - Bind to all interfaces
- `PORT=5000` - Internal HTTP port
- `BASE_URL=http://localhost:5000` - Internal base URL for RFC

## 🚀 Quick Start

**🚨 CRITICAL: Create Required Files and Directories**

Before running Docker, you MUST create the required directories and the `.env` file:

1. **Clone this repository**:
   ```bash
   git clone <your-repo-url>
   cd agent-zero-docker
   ```

2. **Create required directories and files**:
   ```bash
   # Create required directories
   mkdir -p agent-zero-env agent-zero-logs agent-zero-config
   
   # Create the .env FILE (not directory) - this is critical!
   touch agent-zero-env/.env
   ```

**⚠️ Common Error**: If you accidentally mount the `agent-zero-env` directory to `/root/agent-zero/.env` instead of the `.env` file, you'll get:
```
IsADirectoryError: [Errno 21] Is a directory: '/root/agent-zero/.env'
```

**✅ Correct volume mount**: `./agent-zero-env/.env:/root/agent-zero/.env` (file to file)  
**❌ Incorrect volume mount**: `./agent-zero-env:/root/agent-zero/.env` (directory to file)

3. **Choose your GPU type and build the container**:

   **For NVIDIA GPUs:**
   ```bash
   docker compose -f docker-compose_nvidia.yaml up --build
   ```

   **For AMD GPUs:**
   ```bash
   docker compose -f docker-compose_amd.yaml up --build
   ```

4. **Access the web interface**:
   Open your browser and navigate to:
   ```
   http://localhost:55080
   ```

5. **Configure Agent Zero**:
   - Set up your API keys in the Settings
   - Configure RFC settings:
     - RFC Destination URL: `http://localhost`
     - RFC Password: Choose a secure password
     - RFC HTTP port: `5000`
     - RFC SSH port: `22`

## ⚙️ Detailed Setup

### First-Time Configuration

1. **Access Settings**: Click the settings icon in the web interface
2. **Configure Models**: Set up your preferred AI model providers (OpenAI, Ollama, etc.)
3. **Set RFC Password**: This password enables agent self-interaction via SSH
4. **Save Settings**: All settings will persist in the `agent-zero-config` volume

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

1. **IsADirectoryError: [Errno 21] Is a directory: '/root/agent-zero/.env'**:
   
   **Cause**: The `.env` file wasn't created properly, or you're mounting a directory instead of a file.
   
   **Solution**:
   ```bash
   # Ensure the file exists as a FILE, not directory
   touch agent-zero-env/.env
   
   # Verify it's a file, not a directory
   ls -la agent-zero-env/
   # Should show: -rw-r--r-- 1 user user size date .env
   
   # Check your docker-compose file has the CORRECT mount:
   # ✅ Correct: - ./agent-zero-env/.env:/root/agent-zero/.env
   # ❌ Wrong:   - ./agent-zero-env:/root/agent-zero/.env
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
   - Verify volume mounts in docker-compose file
   - Check permissions on host directories
   - Ensure `.env` file exists and is not a directory

7. **First Time Boot**
Note that on the first time Agent Zero is indexing and so it might take a long time until it responds the first time.

### Log Access

View container logs:
```bash
# Follow logs in real-time
docker compose logs -f

# View specific service logs
docker compose logs app
```

Access persistent logs:
```bash
# HTML logs are saved in agent-zero-logs/
ls -la agent-zero-logs/
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
# Create backup
tar -czf agent-zero-backup.tar.gz agent-zero-config agent-zero-env agent-zero-logs

# Restore backup
tar -xzf agent-zero-backup.tar.gz
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

1. **First, verify your setup**:
   ```bash
   # Check if .env file exists as a FILE (not directory)
   ls -la agent-zero-env/.env
   
   # Should show file details, not "No such file or directory"
   # If it shows as a directory, you have the wrong mount configuration
   ```
2. Check the troubleshooting section above
3. Review Agent Zero's official documentation
4. Check Docker and GPU driver installation (NVIDIA Container Toolkit or ROCm)
5. Open an issue in this repository with detailed logs and specify your GPU type (NVIDIA/AMD)
