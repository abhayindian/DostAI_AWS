# DostAI - Requirements Document

## 1. Overview

DostAI is a Hybrid Edge-AI Voice Assistant device built on Raspberry Pi that bridges the digital divide for rural India. It provides critical information to villagers who lack internet access and literacy through voice-first interaction in local languages.

## 2. Problem Statement

- 50% of rural India has intermittent or no internet connectivity
- Villagers cannot use standard AI tools due to:
  - Requirement for high-speed data
  - English literacy barriers
  - Lack of typing skills
- Critical information (weather, crop prices, government schemes) is inaccessible

## 3. User Stories

### 3.1 Primary Users: Rural Villagers/Farmers

**US-1: Voice Query in Local Language**
- As a farmer, I want to ask questions in Hindi/Bhojpuri using my voice
- So that I can get information without needing to read or type

**US-2: Offline Information Access**
- As a villager with no internet, I want to get answers to basic questions offline
- So that I can access critical information anytime

**US-3: Crop Price Information**
- As a farmer, I want to know current crop prices
- So that I can make informed selling decisions

**US-4: Weather Updates**
- As a farmer, I want to know weather forecasts
- So that I can plan my farming activities

**US-5: Government Scheme Discovery**
- As a villager, I want to find government schemes I'm eligible for
- So that I can access benefits and support programs

**US-6: Emergency Assistance**
- As a villager, I want to trigger emergency alerts using voice
- So that I can get help during medical emergencies

### 3.2 Secondary Users: System Administrators

**US-7: Data Sync Management**
- As an admin, I want the system to sync data when internet is available
- So that information stays current

**US-8: Usage Analytics**
- As an admin, I want to analyze anonymized usage patterns
- So that I can improve the service

## 4. Functional Requirements

### 4.1 Voice Interface

**FR-1.1: Speech-to-Text (STT)**
- System MUST support offline Hindi and Bhojpuri speech recognition
- System MUST use Whisper or equivalent for local STT processing
- System MUST achieve >85% accuracy for clear speech

**FR-1.2: Text-to-Speech (TTS)**
- System MUST support offline Hindi and Bhojpuri voice synthesis
- System MUST use Piper or equivalent for local TTS
- System SHOULD use Amazon Polly for higher quality when online

**FR-1.3: Voice Activation**
- System MUST support wake word detection
- System MUST respond within 2 seconds of query completion

### 4.2 Hybrid Processing Architecture

**FR-2.1: Edge Processing (Offline Mode)**
- System MUST run quantized SLM (Gemma-2B or Llama-3-8B) on Raspberry Pi 5
- System MUST handle queries offline for:
  - Basic weather information
  - Cached crop prices
  - Educational content
  - Government scheme matching
- System MUST operate in 100% offline mode for essential tasks

**FR-2.2: Cloud Processing (Online Mode)**
- System MUST detect internet connectivity automatically
- System MUST route complex queries to Amazon Bedrock when online
- System MUST sync with AWS IoT Core for data updates
- System MUST fallback to edge processing if cloud fails

**FR-2.3: Query Routing**
- System MUST classify queries as simple (edge) or complex (cloud)
- System MUST route queries based on connectivity and complexity

### 4.3 Yojna-Scanner (Government Scheme Matching)

**FR-3.1: Offline RAG System**
- System MUST maintain local vector database of government schemes
- System MUST match user profile to eligible schemes
- System MUST provide scheme details in local language

**FR-3.2: Scheme Database**
- System MUST store scheme eligibility criteria
- System MUST update scheme database during sync

### 4.4 Emergency SOS

**FR-4.1: Voice-Activated Alerts**
- System MUST recognize emergency keywords in local languages
- System MUST trigger alerts to pre-configured contacts
- System MUST work offline using SMS/local network

**FR-4.2: Emergency Response**
- System MUST provide immediate acknowledgment
- System MUST attempt multiple notification channels

### 4.5 Data Management

**FR-5.1: Data Synchronization**
- System MUST sync when internet connectivity is detected
- System MUST prioritize critical updates (weather, prices)
- System MUST handle interrupted syncs gracefully

**FR-5.2: Privacy & Anonymization**
- System MUST anonymize user data before cloud storage
- System MUST store anonymized data in Amazon S3
- System MUST comply with data privacy regulations

**FR-5.3: Local Storage**
- System MUST cache essential data locally
- System MUST manage storage efficiently on Raspberry Pi

## 5. Non-Functional Requirements

### 5.1 Performance

**NFR-1.1: Response Time**
- Edge queries MUST respond within 3 seconds
- Cloud queries SHOULD respond within 10 seconds
- Voice recognition latency MUST be <1 second

**NFR-1.2: Availability**
- System MUST operate 24/7 in offline mode
- System uptime MUST be >99% for edge processing

### 5.2 Scalability

**NFR-2.1: Device Scalability**
- Architecture MUST support deployment to 1000+ devices
- Cloud infrastructure MUST handle concurrent requests from all devices

### 5.3 Reliability

**NFR-3.1: Fault Tolerance**
- System MUST gracefully degrade when cloud is unavailable
- System MUST recover from power interruptions
- System MUST handle corrupted local data

### 5.4 Usability

**NFR-4.1: Accessibility**
- System MUST be usable by illiterate users
- System MUST support elderly users with clear audio
- System MUST provide audio feedback for all interactions

### 5.5 Security

**NFR-5.1: Data Security**
- System MUST encrypt data in transit to AWS
- System MUST secure AWS credentials on device
- System MUST implement device authentication with AWS IoT Core

## 6. Technical Constraints

### 6.1 Hardware
- Raspberry Pi 5 (4GB+ RAM recommended)
- USB Microphone with noise cancellation
- Speaker with sufficient volume for outdoor use
- SD Card (64GB+ for model storage)

### 6.2 Software Stack
- Python 3.9+
- LangChain for LLM orchestration
- AWS SDK (boto3) for cloud integration
- Whisper for STT
- Piper for TTS
- Quantized LLM (Gemma-2B or Llama-3-8B)

### 6.3 Cloud Services
- AWS IoT Core for device management
- Amazon Bedrock for advanced AI processing
- Amazon S3 for data storage
- Amazon Polly for high-quality TTS (optional)

## 7. Acceptance Criteria

### 7.1 Core Functionality
- [ ] Device responds to voice queries in Hindi/Bhojpuri
- [ ] System operates fully offline for basic queries
- [ ] System syncs with cloud when internet available
- [ ] Emergency SOS triggers successfully

### 7.2 Performance
- [ ] Voice recognition accuracy >85%
- [ ] Response time <3 seconds for edge queries
- [ ] System runs continuously for 24+ hours

### 7.3 User Experience
- [ ] Non-literate users can operate device independently
- [ ] Audio quality is clear in noisy environments
- [ ] System provides helpful error messages in local language

## 8. Out of Scope (V1)

- Multi-user voice recognition
- Video/image processing
- Payment processing
- Real-time translation between languages
- Mobile app companion
