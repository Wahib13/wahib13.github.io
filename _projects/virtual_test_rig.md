---
title: Virtual Test Rig
order: 1
summary: My master's thesis. A digital twin of a drone wing that forecasts wing-tip deflection, watches its own prediction error, and retrains and promotes a new model on its own when the predictions drift from reality.
stack: [Python, FastAPI, TensorFlow, TimescaleDB, MQTT, Prefect, MLflow, GitLab CI, Docker, Raspberry Pi]
status: Master's thesis, TU Clausthal Databases and Information Systems Group
project_link: 'https://github.com/Wahib13/virtual-test-rig'
pipeline: [stream, predict, score, aggregate, detect drift, retrain, promote, reload]
---

## Why

Most ML demos stop at "train a model, get a number". The thesis is about the part after that: what happens when the model is live, the physical system changes, and nobody is watching. The full title is *Design and Implementation of a Data-Driven Digital Twin with Continuous Learning Using MLOps Principles*, and the platform is a test bench for exactly that loop. An aeroelastic wing simulation is the data source because it produces a continuous, physically meaningful signal that can be perturbed on demand.

## How it works

- **Simulation.** SHARPy generates wing deflection time series. There is also an ArduPilot software-in-the-loop setup and an IMU integration for a physical drone, so the same loop can run on real sensor data.
- **Streaming.** Readings are published over MQTT through NanoMQ and stored in TimescaleDB hypertables. Continuous aggregates keep the rolling error statistics cheap to query.
- **Prediction.** A sequence-to-sequence model forecasts the next 16 timesteps, and each forecast is scored once the real values arrive.
- **Drift detection.** Aggregated error is compared against a threshold. Crossing it triggers a Prefect flow that retrains on recent data.
- **Promotion.** The new model is logged to MLflow. A webhook on model promotion tells the prediction service to reload with zero downtime, and a seed model covers cold starts.
- **Deployment.** The whole stack is containerised and runs on a Raspberry Pi, with GitLab CI handling builds, tests and rollouts.

A FastAPI layer exposes telemetry, attitude data and model output to a small React dashboard that shows the live signal, forecasts and error trend.

## Design choices

- The backend is asset-agnostic. Swapping the wing for a different sensor stream means adding a data source, not reworking the pipeline.
- The retraining decision lives in the data layer, as TimescaleDB continuous aggregates, rather than in application code. That made the whole system easier to reason about.
- MLflow's registry plus a webhook is a clean way to promote models without a bespoke deploy step.
