# 🇮🇳 DostAI: The Offline AI Lifeline for Rural India
> **Track:** [Student Track] AI for Communities, Access & Public Impact

![DostAI Architecture](assets/architecture_diagram.jpg)

## 🚀 Problem Statement
**Internet is not a luxury; knowledge is.**
50% of rural India suffers from intermittent or no internet connectivity. Current AI assistants (Alexa, Google, ChatGPT) fail completely without high-speed data. Furthermore, literacy and language barriers prevent millions of villagers from accessing critical government schemes (*Yojnas*), agricultural data, and educational resources.

## 💡 The Solution: DostAI
**DostAI** (Friend-AI) is a **Hybrid Edge-AI Voice Assistant** designed specifically for the "Last Mile." It runs on a low-cost **Raspberry Pi 5** to provide 24/7 access to critical information—even when the internet is down.

It uses a **Hybrid Architecture**:
1.  **Offline (Edge):** Handles daily queries (Weather, Crop Prices, Govt Schemes) instantly using a local SLM (Gemma-2B) and offline Voice AI.
2.  **Online (Cloud):** Syncs with **AWS Cloud** when connectivity is available to fetch updates and process complex reasoning tasks.

---

## ✨ Key Features
* **🗣️ Vernacular Voice First:** A completely hands-free interface that speaks **Hindi & Bhojpuri**. No typing required.
* **📡 100% Offline Capability:** Works without an active data plan for essential tasks using quantized local models.
* **📜 "Yojna Scanner":** An offline RAG (Retrieval Augmented Generation) system that matches farmers to government schemes based on their profile.
* **🆘 Emergency SOS:** Voice-activated safety alerts ("Help", "Bachao") that trigger SMS alerts even on low bandwidth.
* **🔄 Smart Cloud Sync:** Uses **AWS IoT Core** to efficiently sync data (weather/prices) only when connectivity is detected.

---

## 🏗️ System Architecture

Our **Hybrid Edge-Cloud** design ensures privacy, speed, and reliability.

| Layer | Component | Technology |
| :--- | :--- | :--- |
| **Hardware** | Edge Device | Raspberry Pi 5 (8GB) + USB Mic |
| **Voice AI** | STT / TTS | OpenAI Whisper (Quantized) + Piper (Local) |
| **Intelligence** | Edge Model | **Gemma-2B** or **Llama-3-8B** (GGUF Format) |
| **Cloud AI** | Complex Reasoning | **Amazon Bedrock** (Claude 3 Haiku) |
| **Connectivity** | Sync & Mgmt | **AWS IoT Core** (MQTT) |
| **Storage** | Data Lake | **Amazon S3** (Anonymized Analytics) |

---

## 📂 Project Documentation
This repository contains the full technical specifications generated for the Hackathon Phase 1 submission:

* **[📄 Requirements Document](requirements.md):** Detailed User Stories, Functional & Non-Functional Requirements.
* **[🏗️ Design Document](design.md):** Complete System Architecture, API Specifications, Database Schema, and Security Design.
* **[✅ Implementation Tasks](tasks.md):** A phased execution plan to build the prototype.

---

## 🔮 Future Roadmap
* **Computer Vision:** Crop disease detection using the Pi Camera.
* **Telemedicine:** Integration with Amazon Chime SDK for doctor consultations.
* **Solar Power:** Battery optimization for 100% off-grid usage.

---
*Built for the **AI for Bharat Hackathon 2026**.*
