FROM python:3.11-slim

# System dependencies for audio/video processing and health checks
RUN apt-get update && apt-get install -y --no-install-recommends \
    ffmpeg \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Upgrade pip
RUN pip install --no-cache-dir --upgrade pip

# 1. Install lightweight CPU-only PyTorch first (saves ~4GB disk and avoids CUDA build timeouts)
RUN pip install --no-cache-dir torch torchvision --index-url https://download.pytorch.org/whl/cpu

# 2. Install remaining application dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 3. Pre-download YOLOv8 nano weights so container boots instantly
RUN python -c "from ultralytics import YOLO; YOLO('yolov8n.pt')" || true

# 4. Copy application source
COPY . .

# 5. Ensure storage directories exist
RUN mkdir -p weights uploads && (cp yolov8n.pt weights/yolov8n.pt 2>/dev/null || true)

# Hugging Face Spaces uses 7860, Render uses 10000 / 
EXPOSE 7860 10000 8000

# Set memory-friendly defaults (tiny Whisper model fits in free tier RAM)
ENV WHISPER_MODEL_SIZE=tiny
ENV PORT=7860

# Run uvicorn server (binds dynamically to , defaulting to 7860 for Hugging Face or  on Render)
CMD ["sh", "-c", "uvicorn backend.app.main:app --host 0.0.0.0 --port "]
