FROM python:3.8-slim

ARG BUILD_ENV
ARG OVERCOOKED_BRANCH
ARG GRAPHICS

WORKDIR /app

# Install system dependencies including git
RUN apt-get update && apt-get install -y \
    git \
    libgl1-mesa-dev \
    libhdf5-dev \
    pkg-config \
    build-essential \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Upgrade pip
RUN pip install --upgrade pip

# Copy requirements and install Python dependencies
COPY ./requirements.txt ./requirements.txt
RUN pip install -r requirements.txt

# Install eventlet production server if production build
RUN if [ "$BUILD_ENV" = "production" ] ; then pip install eventlet ; fi

# Clone chai code
RUN git clone --recursive https://github.com/HumanCompatibleAI/overcooked_ai.git --branch $OVERCOOKED_BRANCH --single-branch /overcooked_ai

# Update ray[rllib] version in setup.py
RUN sed -i "s/ray\[rllib\]==2.0.0/ray\[rllib\]>=2.2.0/" /overcooked_ai/setup.py

# Dummy data_dir so things don't break
RUN echo "import os; DATA_DIR=os.path.abspath('.')" >> /overcooked_ai/src/human_aware_rl/data_dir.py

# Install chai dependencies
RUN pip install -e '/overcooked_ai[harl]'

# Copy over remaining files
COPY ./static ./static
COPY ./*.py ./
COPY ./graphics/$GRAPHICS ./static/js/graphics.js
COPY ./config.json ./config.json

# Set environment variables that will be used by app.py
ENV HOST 0.0.0.0
ENV PORT 5000
ENV CONF_PATH config.json

# Expose the application port
EXPOSE 5000

# Run the application
CMD ["python", "-u", "app.py"]
