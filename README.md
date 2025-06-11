# Agent Zero Docker Setup with CUDA Support

This repository contains a complete Docker setup for running [Agent Zero](https://github.com/frdel/agent-zero) with NVIDIA CUDA GPU support in a containerized environment.

## 🚀 Features

- **CUDA GPU Support**: Full NVIDIA GPU acceleration for AI models
- **Persistent Configuration**: Settings and data persist across container rebuilds
- **SSH Access**: Internal SSH for agent code execution
- **Remote Function Calls (RFC)**: Proper SSH configuration for agent self-interaction
- **Web Interface**: Clean web UI accessible from host machine

## 📋 Prerequisites

- Docker and Docker Compose installed
- NVIDIA Docker runtime (nvidia-docker2)
- NVIDIA GPU with compatible drivers
- Git (for version control)

### Installing NVIDIA Docker Support

```bash
# Install nvidia-docker2
sudo apt-get update
sudo apt-get install -y nvidia-docker2

# Restart Docker daemon
sudo systemctl restart docker

# Test NVIDIA Docker
docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

## 🏗️ Project Structure

```
agent-zero-docker/
├── dockerfile                 # Custom Dockerfile with CUDA and Agent Zero
├── docker-compose.yaml       # Docker Compose configuration
├── agent-zero-config/        # Persistent settings and chat history
├── agent-zero-logs/          # Application logs
├── agent-zero-env            # Environment variables (.env file)
└── README.md                 # This file
```

## 🔧 Configuration

### Docker Compose Services

The `docker-compose.yaml` defines:

- **Ports**: 
  - `55080:5000` - Web UI access
  - `55022:22` - SSH access (internal use)
- **GPU Access**: Full NVIDIA GPU allocation
- **Persistent Volumes**: Configuration, logs, and environment variables

### Environment Variables

Key environment variables in `docker-compose.yaml`:
- `HOST=0.0.0.0` - Bind to all interfaces
- `PORT=5000` - Internal HTTP port
- `BASE_URL=http://localhost:5000` - Internal base URL for RFC

## 🚀 Quick Start

1. **Clone this repository**:
   ```bash
   git clone <your-repo-url>
   cd agent-zero-docker
   ```

2. **Build and start the container**:
   ```bash
   docker compose up --build
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

To verify GPU access within the container:
```bash
# Execute into the running container
docker exec -it agent-zero-cuda bash

# Check GPU availability
nvidia-smi

# Test with Python
python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}')"
```

## 🔒 Security Considerations

- **Local Use Only**: This setup is designed for local development
- **SSH Access**: SSH is configured for internal agent use only
- **Password Security**: Change default passwords in production
- **Firewall**: Consider firewall rules if exposing ports externally

## 🛠️ Troubleshooting

### Common Issues

1. **Authentication Failed (SSH)**:
   - Ensure RFC password is set in the web interface
   - Check that SSH service is running in the container

2. **Bad Request Errors in Logs**:
   - These are harmless SSL handshake attempts to HTTP server
   - Always use `http://` (not `https://`) when accessing the interface

3. **GPU Not Available**:
   - Verify NVIDIA Docker runtime installation
   - Check GPU drivers on host system
   - Ensure `--gpus all` flag is working

4. **Configuration Not Persisting**:
   - Verify volume mounts in docker-compose.yaml
   - Check permissions on host directories

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
```bash
# Get shell access
docker exec -it agent-zero-cuda bash

# Test SSH connection (internal)
docker exec -it agent-zero-cuda ssh root@localhost -p 22
```

## 🔄 Development Workflow

### Rebuilding

When making changes to the Dockerfile:
```bash
# Stop and rebuild
docker compose down
docker compose up --build
```

### Updating Agent Zero

To update to the latest Agent Zero version:
```bash
# Rebuild with fresh clone
docker compose down
docker compose build --no-cache
docker compose up
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

## 🤝 Contributing

1. Fork this repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

This project follows the same license as Agent Zero. See the [Agent Zero repository](https://github.com/frdel/agent-zero) for license details.

## 🆘 Support

If you encounter issues:

1. Check the troubleshooting section above
2. Review Agent Zero's official documentation
3. Check Docker and NVIDIA Docker installation
4. Open an issue in this repository with detailed logs
