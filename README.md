# Local AI Coding Assistant with Ollama + Continue

A fully offline, Docker-based AI coding assistant for VS Code using Ollama and Continue with multiple Qwen and DeepSeek Coder models.

## 📋 Prerequisites

- **Windows 10 Pro** (or any OS with Docker support)
- **Docker Desktop** installed and running (with Docker Compose)
- **VS Code** installed
- **32GB RAM** (recommended for smooth performance)
- **~20GB free disk space** (for Docker images and models)

## 🚀 Quick Start

### 1. Clone this repository

```bash
git clone https://github.com/your-username/your-repo-name.git
cd your-repo-name
```

### 2. Start Ollama with Docker Compose

```bash
# Start Ollama container in detached mode
docker-compose up -d

# Verify it's running
docker-compose ps
```

### 3. Pull the Models

```bash
# Pull the recommended models
docker-compose exec ollama ollama pull qwen2.5-coder:7b-instruct
docker-compose exec ollama ollama pull nomic-embed-text
docker-compose exec ollama ollama pull qwen2.5-coder:14b-instruct
docker-compose exec ollama ollama pull deepseek-coder:6.7b-instruct
```

**Model descriptions:**
- `qwen2.5-coder:7b-instruct` - Best balance of speed and quality (recommended for daily use)
- `qwen2.5-coder:14b-instruct` - Highest quality, slower (for complex tasks)
- `deepseek-coder:6.7b-instruct` - Alternative model, good for code generation
- `nomic-embed-text` - For embeddings and code search (used automatically by Continue)

### 4. Verify Ollama is running

Open your browser and go to: `http://localhost:11434`

You should see: `Ollama is running`

Or test via command line:
```bash
docker-compose exec ollama ollama list
```

### 5. Install VS Code Extensions

1. Open VS Code
2. Go to Extensions (Ctrl+Shift+X)
3. Search for and install: **Continue - Codestral, Claude, and more**
   - Extension ID: `Continue.continue`

### 6. Configure Continue

1. In VS Code, open the Continue extension settings
2. Click on the gear icon ⚙️ in the Continue sidebar
3. Select "Open config file" (this creates `~/.continue/config.yaml`)
4. Replace the contents with the `config.yaml` from this repository

**Important Notes:**
- The current Continue schema supports the `models` array with role assignments
- Embeddings are automatically configured when you pull the `nomic-embed-text` model
- Add `autocomplete` to a model's roles to enable code autocomplete
- Use `provider: ollama` for local models with `apiBase: http://localhost:11434`

## 📁 Configuration Files

### config.yaml

```yaml
name: Main Config
version: 1.0.0
schema: v1
models:
  - name: Qwen 2.5 Coder 7B
    provider: ollama
    model: qwen2.5-coder:7b-instruct
    apiBase: http://localhost:11434
    roles:
      - chat
      - edit
      - apply
      - autocomplete

  - name: Qwen 2.5 Coder 14B
    provider: ollama
    model: qwen2.5-coder:14b-instruct
    apiBase: http://localhost:11434
    roles:
      - chat
      - edit
      - apply

  - name: DeepSeek Coder 6.7B
    provider: ollama
    model: deepseek-coder:6.7b-instruct
    apiBase: http://localhost:11434
    roles:
      - chat
      - edit
      - apply
      - autocomplete
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    ports:
      - "11434:11434"
    volumes:
      - ./ollama-data:/root/.ollama
    restart: unless-stopped
    environment:
      - OLLAMA_KEEP_ALIVE=24h
      - OLLAMA_HOST=0.0.0.0
    healthcheck:
      test: ["CMD", "ollama", "list"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
    networks:
      - ollama-network

networks:
  ollama-network:
    driver: bridge
```

## 📖 Usage Guide

### Starting the System

1. **Start Docker Desktop** (if not already running)
2. **Start Ollama container** (if not running):
   ```bash
   docker-compose up -d
   ```
3. **Open VS Code** and start coding!

### Using Continue in VS Code

- **Open Continue sidebar**: Click the Continue icon in the activity bar (or `Ctrl+L`)
- **Chat with AI**: Type your questions in the chat box
- **Switch models**: Use the model dropdown in the chat interface to switch between installed models
- **Code autocomplete**: Start typing and Continue will suggest completions (press `Tab` to accept)
- **Select code and ask**: Highlight code, right-click, select "Ask Continue"
- **Quick commands**: Use `/` in chat for commands like `/edit`, `/explain`, `/fix`

### Model Selection Guide

| Model | Best For | Speed | Quality |
|-------|----------|-------|---------|
| Qwen 2.5 Coder 7B | Daily coding, autocomplete | Fast | Good |
| DeepSeek Coder 6.7B | Code generation, alternative | Fast | Good |
| Qwen 2.5 Coder 14B | Complex tasks, architecture | Slow | Excellent |

### Docker Compose Commands Cheat Sheet

```bash
# View logs
docker-compose logs -f ollama

# Stop Ollama
docker-compose down

# Restart Ollama
docker-compose restart

# Stop and remove container (keeps data)
docker-compose down

# Remove everything including data
docker-compose down -v && rm -rf ollama-data

# Check container status
docker-compose ps

# View resource usage
docker stats ollama
```

### Managing Models

```bash
# List installed models
docker-compose exec ollama ollama list

# Pull a different model
docker-compose exec ollama ollama pull model-name

# Remove a model
docker-compose exec ollama ollama rm model-name

# Check running models
docker-compose exec ollama ollama ps

# Load a model into memory (for faster first response)
docker-compose exec ollama ollama run qwen2.5-coder:7b-instruct ""
```

### Creating Custom Models with Parameters

If you need to adjust model parameters (temperature, context length, etc.):

```bash
# Create a Modelfile
cat > Modelfile << EOF
FROM qwen2.5-coder:7b-instruct
PARAMETER temperature 0.7
PARAMETER num_ctx 4096
PARAMETER num_predict 1024
EOF

# Create the custom model
docker-compose exec ollama ollama create qwen2.5-coder-7b-custom -f Modelfile

# Use it in config.yaml as "qwen2.5-coder-7b-custom"
```

## ⚙️ Performance Tuning

### Docker Compose Resource Allocation

Add these lines to `docker-compose.yml` under the `ollama` service:

```yaml
services:
  ollama:
    # ... existing config ...
    deploy:
      resources:
        limits:
          memory: 16G
        reservations:
          memory: 8G
    cpus: 8
```

### For Better Performance on CPU

1. **Use smaller models for autocomplete**: Add `autocomplete` role only to the 7B model
2. **Docker Desktop settings**:
   - Open Docker Desktop → Settings → Resources
   - Allocate at least 8GB RAM (16GB recommended)
   - Set CPUs to at least 4 cores
   - Enable "Use the WSL 2 based engine" for better performance

### Memory Usage Guide

| Model | RAM Usage | Loading Time |
|-------|-----------|--------------|
| 6.7B-7B | ~6GB | 5-10 seconds |
| 14B | ~12GB | 15-30 seconds |

## 🔧 Troubleshooting

### Ollama won't start
```bash
# Check if port 11434 is in use
netstat -ano | findstr :11434

# Kill process using the port (if needed)
taskkill /PID [PID] /F

# Restart Docker Desktop
# Then restart the container
docker-compose up -d
```

### Connection refused in Continue
- Ensure Ollama container is running: `docker-compose ps`
- Check if Ollama is accessible: `curl http://localhost:11434/api/tags`
- Verify the API base URL in Continue config (should be `http://localhost:11434`)

### Config validation errors
- Use only the `models` array in config.yaml
- Remove unsupported properties like `embeddingsProvider`, `tabAutocompleteModel`, `completionOptions`, `contextLength`
- Use `provider: ollama` instead of `provider: openai` for local models
- Check that the API base URL doesn't have `/v1` at the end
- Ensure each model has appropriate roles assigned

### Model switching issues
- Make sure the model is pulled: `docker-compose exec ollama ollama list`
- Try unloading all models: `docker-compose exec ollama ollama stop`
- Restart Ollama: `docker-compose restart`

### Autocomplete not working
- Ensure at least one model has `autocomplete` in its roles
- Check if the autocomplete model is running: `docker-compose exec ollama ollama ps`
- Try using a smaller model for autocomplete (7B instead of 14B)

### Slow performance
- Close other memory-intensive applications
- Ensure Docker has enough resources allocated
- Use 7B model instead of 14B for faster responses
- Consider disabling autocomplete if it's too slow
- Check if the model is loaded in memory: `docker-compose exec ollama ollama ps`

### Model download stuck
```bash
# Check disk space
df -h

# Pull model with progress bars disabled
docker-compose exec ollama ollama pull qwen2.5-coder:7b-instruct --verbose

# Clear Docker cache if needed
docker system prune -a
```

### Reset everything
```bash
# Complete reset (WARNING: deletes all models)
docker-compose down -v
rm -rf ollama-data
docker system prune -a
docker-compose up -d
# Re-pull models
docker-compose exec ollama ollama pull qwen2.5-coder:7b-instruct
docker-compose exec ollama ollama pull nomic-embed-text
docker-compose exec ollama ollama pull qwen2.5-coder:14b-instruct
docker-compose exec ollama ollama pull deepseek-coder:6.7b-instruct
```

## 📁 Project Structure

```
your-repo-name/
├── README.md           # This file
├── docker-compose.yml  # Docker Compose configuration
├── config.yaml         # Continue configuration
├── ollama-data/        # Ollama models and data (created on first run)
├── .gitignore          # Git ignore file
```

## 🤝 Contributing

Feel free to open issues or submit pull requests with improvements.

## 📝 License

MIT License - See LICENSE file for details

## 🔗 Resources

- [Ollama Documentation](https://docs.ollama.com/)
- [Continue Documentation](https://docs.continue.dev/)
- [Qwen2.5 Coder Model](https://ollama.com/library/qwen2.5-coder)
- [DeepSeek Coder Model](https://ollama.com/library/deepseek-coder)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

---

**Note**: 
- First model pull may take 10-15 minutes per model depending on your internet speed
- Total download size for all models: ~15GB
- For best performance, use the 7B model for daily coding and switch to 14B for complex tasks
- The nomic-embed-text model is used automatically by Continue for embeddings
- If you need to customize model parameters, use the Modelfile approach described above


-----------------------
Getting Free API Keys
Groq (Recommended - Best Free Tier)
1.	Go to https://console.groq.com/
2.	Sign up (Google/GitHub)
3.	Go to API Keys section
4.	Create new key
5.	Copy key (starts with gsk_)
Google Gemini
1.	Go to https://aistudio.google.com/
2.	Sign in with Google account
3.	Click "Get API key"
4.	Create key
5.	Copy key (starts with AIza)
OpenRouter
1.	Go to https://openrouter.ai/
2.	Sign up (GitHub/Google)
3.	Go to Keys section
4.	Create new key
5.	Copy key (starts with sk-or-)



Step-by-Step Guide to Get Cloudflare Credentials
1. Sign Up / Log In to Cloudflare
1.	Go to: https://dash.cloudflare.com/
2.	Sign up for a free account (email/password) or log in
3.	You don't need a domain - you can use Workers AI without one
2. Get Your Account ID
Method 1: From the Dashboard
1.	After logging in, you'll see the dashboard
2.	Look at the right sidebar on the main page
3.	You'll see "Account ID" under your account name
4.	Click the copy icon to copy it
Method 2: From Workers & Pages
1.	Click on "Workers & Pages" in the left sidebar
2.	Look at the right side of the page
3.	Account ID is displayed in the "Account details" section
Method 3: From URL
•	When you're logged in, the URL looks like:
https://dash.cloudflare.com/your-account-id/
•	The string after dash.cloudflare.com/ is your Account ID
3. Create an API Token
1.	Go to API Tokens page:
o	Click on your profile icon (top right)
o	Select "My Profile"
o	Click on "API Tokens" tab on the left
o	OR go directly to: https://dash.cloudflare.com/profile/api-tokens
2.	Create a new token:
o	Click "Create Token" button
o	Look for "Workers AI" template (or "Custom token")
o	Click "Use template" next to Workers AI
3.	Configure the token (if using custom):
o	Token name: Anything you want (e.g., "Continue AI")
o	Permissions:
	Account → Workers AI → Edit
	Account → Workers Scripts → Edit (optional, for deploying workers)
o	Account Resources: Include → Your account
o	Client IP Address Filtering: Leave blank (optional)
o	TTL: Choose how long the token lasts (or leave indefinite)
4.	Create the token:
o	Click "Continue to summary"
o	Review settings
o	Click "Create Token"
5.	Copy your token:
o	IMPORTANT: The token is only shown once!
o	Copy it immediately
o	It should start with something like Bearer or a random string
o	Store it securely

