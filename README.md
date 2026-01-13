# MING + Edge Impulse
## (MQTT, InfluxDB, Node-RED, Grafana and Edge Impulse)

This repository contains an educational and experimental docker-based stack called MING + Edge Impulse.

It extends the original MING concept (MQTT, InfluxDB, Node-RED and Grafana) by adding an Edge Impulse container for running on-device computer inference. In this case we will run object detection computer vision inference.

The goal of this project is to provide an open, modular, and reproducible edge AI stack that can capture data from sensors or cameras, perform local inference, store structured results, and visualize them in real-time, all local.

The stack is designed to be simple to deploy using `Docker Compose` and suitable for edge devices, industrial gateways, or local development environments.

## Architecture Overview

Each component runs in its own Docker container and communicates over a shared Docker bridge network.

Included services:

* Mosquitto: MQTT broker used for publishing and subscribing to sensor data, inference events, and insights.
* Node-RED: Visual programming environment used to orchestrate data flows, trigger image capture, call Edge Impulse inference APIs, parse results, publish MQTT messages, and write data into InfluxDB.
* Edge Impulse: Edge AI inference service used to run machine learning models such as image-based models (for example object detection) via a REST API.
* InfluxDB: Time-series database used to store structured inference results such as detections, confidence scores, timestamps, camera identifiers, and bounding box metadata.
* Grafana: Visualization platform used to build dashboards for inference metrics, object counts, and image visualization.

All services are connected via a dedicated Docker network.

## Requirements

### Hardware

* Arduino UNO Q
* USB-C Hub
* USB camera 

### Software

* [Edge Impulse free account](https://studio.edgeimpulse.com/signup?utm_medium=live_event&utm_source=conference&utm_campaign=21157154-new-user-acquisition_fall2025&utm_content=ming-github-tutorial)


## Getting started

Access via SSH to your Arduino UNO Q.

If you can't use git in your Arduino UNO Q, try:

```
sudo apt install git
```

Clone the repository:

```
git clone https://github.com/your-org/ming-edge-impulse.git

cd ming-edge-impulse
```

Start all services:

```
docker compose up -d
```

Verify that containers are running:

```
docker ps
```

You should be able to access from your computer or from the Arduino UNO Q browser to the deployed services:

* Node-RED at http://localhost or http://<local ip address of the Arduino UNO Q>
* Grafana at http://localhost:8080
* InfluxDB at http://localhost:8086
* Edge Impulse API at http://localhost:1337


### Node-RED Usage

We are going to use Node-RED as the central orchestration layer in the MING stack. Then, we will use Node-RED to develop an application using all the elements from the MING stack. 

We are going to build an application that does object detection running inference with Edge Impulse, depending on the objects detected in the images captured from the camera, store the detected objects in InfluxDB and visualize the aggregated information in Grafana dahsboards.

Node-RED data and images will be persisted using a Docker volume mapped to `/data` and `/data/images`.

### Edge Impulse Inference

The Edge Impulse container exposes an HTTP API for image inference.

```
POST http://edge-impulse:1337/api/image
```

The endpoint expects a multipart/form-data request with a file field containing a JPG or PNG image.

Inference responses include bounding boxes, labels, confidence scores, and timing information.

This stack assumes that inference is run locally and synchronously as part of a Node-RED flow.

<img width="400" alt="Node-RED flow with the MING stack" src="https://github.com/user-attachments/assets/a81901af-aeaa-4feb-ac0b-4888e1103d8a" />


### InfluxDB Data Model

A recommended InfluxDB structure is:

* Organization: edge-impulse
* Bucket: edge-impulse-detections
* Measurement: edge_ai_detection

Suggested tags:

* camera
* label
* model

Suggested fields:

* confidence
* bbox_x
* bbox_y
* bbox_width
* bbox_height

Each detected object is stored as a separate point with its own timestamp.

### Grafana Dashboards

Grafana is used to visualize both numeric inference data and images.

Typical dashboards include:

* Time series showing number of detected objects per camera
* Tables listing detections with confidence scores
* Panels counting detections by label
* HTML panels displaying the latest captured image served by Node-RED

Grafana uses InfluxDB as its data source.

#### Exposing Images to Grafana

Images are not stored in InfluxDB. Instead, Grafana references an HTTP endpoint exposed by Node-RED to render images. Node-RED will serve images from the /data/images directory via HTTP.

Images captured and saved under /data/images can be accessed at:

```
http://node-red/images/image.jpg
```

Grafana can render these images using an HTML panel or text panel in HTML mode.


## Disclaimer

This project is intended for educational and experimental purposes only.
It is not hardened for production use.
Do not deploy in industrial or safety-critical environments without proper security, testing, and validation.

## Attribution

This project is inspired by the original MING concept and various community-driven Node-RED, MQTT, and edge AI projects.

It builds on open knowledge from the Node-RED, InfluxDB, Grafana, Mosquitto, and Edge Impulse ecosystems.






