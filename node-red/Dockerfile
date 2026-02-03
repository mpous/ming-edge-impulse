FROM nodered/node-red:latest-debian

# Switch to root to install system packages
USER root

RUN apt-get update && \
    apt-get install -y \
        fswebcam \
        v4l-utils \
    && rm -rf /var/lib/apt/lists/*

# Copy custom settings.js
COPY settings.js /data/settings.js

# Copy flow definition for easy import
COPY flow.json /opt/flows/flow.json

# Fix ownership (CRITICAL)
RUN chown -R node-red:node-red /data

# Switch back to the default node-red user
USER node-red

# Explicitly tell Node-RED to use /data
ENV NODE_RED_HOME=/usr/src/node-red
ENV NODE_RED_USER_DIR=/data
