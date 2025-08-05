# vLLM Production Stack Deployment on Coolify

This guide provides instructions for deploying the vLLM Production Stack on Coolify.

## Prerequisites

- Coolify instance with GPU support enabled
- NVIDIA drivers and Docker GPU runtime configured
- Sufficient GPU memory (minimum 16GB recommended)
- Hugging Face API token (for downloading models)

## Deployment Options

### Option 1: Using Docker Compose (Recommended)

1. **Fork or Clone the Repository**
   ```bash
   git clone https://github.com/vllm-project/production-stack.git
   cd production-stack
   ```

2. **Configure Environment Variables**
   - Copy `.env.coolify` to `.env`
   - Update the following variables:
     - `HUGGING_FACE_HUB_TOKEN`: Your HF token
     - `VLLM_API_KEY`: Secure API key for vLLM
     - `MODEL_NAME`: Model to deploy (e.g., `mistralai/Mistral-7B-v0.1`)
     - `GRAFANA_PASSWORD`: Secure password for Grafana

3. **Deploy via Coolify**
   - In Coolify, create a new "Docker Compose" resource
   - Point to your repository
   - The default `docker-compose.yml` will be used automatically
   - Enable GPU support in the resource settings
   - Deploy

### Option 2: Individual Container Deployment

If you prefer to deploy components separately:

#### 1. vLLM Engine
```yaml
# In Coolify, create a Docker resource with:
Image: vllm/vllm-openai:latest
Port: 7100
GPU: Enabled
Environment Variables:
  - HUGGING_FACE_HUB_TOKEN=your-token
  - MODEL_NAME=facebook/opt-125m
  - VLLM_API_KEY=your-api-key
Command: --host 0.0.0.0 --port 8000 --model facebook/opt-125m --api-key your-api-key
```

#### 2. Router (Optional)
```yaml
Image: lmcache/lmstack-router:latest
Port: 7100
Environment Variables:
  - SERVICE_DISCOVERY=static
  - STATIC_BACKENDS=http://vllm-engine:8000
  - STATIC_MODELS=facebook/opt-125m
  - VLLM_API_KEY=your-api-key
```

## Configuration

### Model Selection

Popular models and their GPU requirements:

| Model | GPU Memory | Recommended GPU |
|-------|------------|-----------------|
| facebook/opt-125m | 2GB | Any GPU |
| mistralai/Mistral-7B-v0.1 | 16GB | T4, V100, A10 |
| meta-llama/Llama-2-13b | 28GB | A100-40GB |
| meta-llama/Llama-2-70b | 140GB | 2x A100-80GB |

### GPU Memory Optimization

Adjust these parameters based on your GPU:

```env
# For smaller GPUs (T4, RTX 3090)
MAX_MODEL_LEN=8192
GPU_MEMORY_UTILIZATION=0.90

# For larger GPUs (A100)
MAX_MODEL_LEN=32768
GPU_MEMORY_UTILIZATION=0.95
```

### Advanced Configuration

#### Enable Tensor Parallelism (Multi-GPU)
```env
TENSOR_PARALLEL_SIZE=2  # For 2 GPUs
```

#### Enable Prefix Caching
Already enabled in the default configuration for better performance.

#### Custom Models
```env
MODEL_NAME=your-org/your-model
TRUST_REMOTE_CODE=true  # If model requires custom code
```

## Monitoring

### Access Points

After deployment, access:
- **API Endpoint**: `http://your-coolify-domain:7100`
- **Prometheus**: `http://your-coolify-domain:7101`
- **Grafana**: `http://your-coolify-domain:7102`

### Grafana Dashboard

1. Login to Grafana (default: admin/admin)
2. Import the vLLM dashboard from `observability/grafana/dashboards/`
3. Monitor:
   - Request latency
   - GPU utilization
   - Cache hit rates
   - Active/pending requests

## API Usage

### Test the Deployment

```bash
# Health check
curl http://your-coolify-domain:7100/health

# List models
curl http://your-coolify-domain:7100/v1/models \
  -H "Authorization: Bearer your-api-key"

# Chat completion
curl http://your-coolify-domain:7100/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d '{
    "model": "facebook/opt-125m",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### OpenAI SDK Compatible

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-api-key",
    base_url="http://your-coolify-domain:7100/v1"
)

response = client.chat.completions.create(
    model="facebook/opt-125m",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

## Troubleshooting

### GPU Not Detected
- Ensure Coolify has GPU runtime enabled
- Check NVIDIA drivers: `nvidia-smi`
- Verify Docker GPU support: `docker run --gpus all nvidia/cuda:12.1.0-base-ubuntu22.04 nvidia-smi`

### Out of Memory Errors
- Reduce `MAX_MODEL_LEN`
- Lower `GPU_MEMORY_UTILIZATION`
- Use a smaller model
- Enable tensor parallelism for multi-GPU

### Model Download Issues
- Verify `HUGGING_FACE_HUB_TOKEN` is set
- Check network connectivity
- Ensure sufficient disk space in `/root/.cache/huggingface`

### Performance Optimization
- Enable prefix caching (already enabled by default)
- Use `--enable-chunked-prefill` for better latency
- Adjust `--max-num-seqs` based on your workload

## Scaling

### Horizontal Scaling
Deploy multiple vLLM engines with the router for load balancing:

1. Deploy multiple vLLM engine instances
2. Configure router with all backend URLs
3. Use session-based routing for better cache utilization

### Vertical Scaling
- Use larger GPUs (A100, H100)
- Enable tensor parallelism for model parallelism
- Increase memory and compute resources

## Security

1. **API Key**: Always use a strong API key in production
2. **Network**: Use Coolify's internal network for inter-service communication
3. **HTTPS**: Enable SSL/TLS through Coolify's proxy settings
4. **Firewall**: Restrict access to Prometheus and Grafana

## Support

- [vLLM Documentation](https://docs.vllm.ai)
- [Production Stack Documentation](https://docs.vllm.ai/projects/production-stack)
- [GitHub Issues](https://github.com/vllm-project/production-stack/issues)
- [Slack Channel](https://vllm-dev.slack.com/archives/C089SMEAKRA)