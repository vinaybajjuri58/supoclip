# Use Python 3.11 slim image
FROM python:3.11-slim

# Install system dependencies including ffmpeg and unzip (for deno).
# fonts-noto-color-emoji lets libass render colour emojis in burned captions.
RUN apt-get update && apt-get install -y \
    ffmpeg \
    curl \
    unzip \
    fonts-noto-color-emoji \
    fontconfig \
    && fc-cache -f \
    && rm -rf /var/lib/apt/lists/*

# Install deno (required by yt-dlp for YouTube JS challenge solving)
RUN curl -fsSL https://deno.land/install.sh | sh
ENV DENO_DIR=/root/.deno
ENV PATH="/root/.deno/bin:${PATH}"

# Install uv package manager
RUN pip install uv

# Set working directory
WORKDIR /app

# Copy pyproject.toml for dependency installation
COPY pyproject.toml ./

# Install deps (mediapipe has no linux/arm64 wheel; made optional)
RUN printf 'import tomllib\nwith open("pyproject.toml","rb") as f:\n d=tomllib.load(f)["project"]["dependencies"]\nfor x in d:\n if "mediapipe" not in x:\n  print(x)\n' > /tmp/extract.py && \
    python3 /tmp/extract.py > requirements.txt && \
    uv venv .venv && \
    uv pip install --python .venv/bin/python -r requirements.txt && \
    (uv pip install --python .venv/bin/python "mediapipe>=0.10.0" 2>/dev/null || \
     echo "mediapipe skipped (face detection uses OpenCV fallback)")

# Force update yt-dlp to the absolute latest version to handle YouTube API changes
# Also install from git to get the very latest commits
RUN uv pip install --upgrade --force-reinstall yt-dlp
RUN uv pip install --upgrade --force-reinstall "yt-dlp[default]"

# Copy source code
COPY src/ ./src/

# Copy fonts and transitions directories
COPY fonts/ ./fonts/
COPY transitions/ ./transitions/

# Create necessary directories for video processing
RUN mkdir -p /app/uploads /app/clips /app/logs /tmp

# Expose the port FastAPI will run on
EXPOSE 8000

# Activate virtual environment and start the application
CMD [".venv/bin/uvicorn", "src.main_refactored:app", "--host", "0.0.0.0", "--port", "8000"]
