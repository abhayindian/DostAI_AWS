# DostAI - Design Document

## 1. System Architecture

### 1.1 High-Level Architecture

DostAI follows a hybrid edge-cloud architecture with three main layers:

```
┌─────────────────────────────────────────────────────────────┐
│                    VILLAGE (EDGE)                           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │           Raspberry Pi 5 Device                      │  │
│  │  ┌────────────┐  ┌──────────────┐  ┌─────────────┐ │  │
│  │  │   Voice    │  │   Local AI   │  │   Local     │ │  │
│  │  │  Interface │→ │   Engine     │→ │   Storage   │ │  │
│  │  │ (STT/TTS)  │  │ (Gemma/Llama)│  │  (SQLite)   │ │  │
│  │  └────────────┘  └──────────────┘  └─────────────┘ │  │
│  └──────────────────────────────────────────────────────┘  │
│                           ↕                                 │
│                  (Periodic Sync / Complex Queries)          │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                    AWS CLOUD                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   AWS IoT    │→ │    Amazon    │  │   Amazon     │     │
│  │     Core     │  │   Bedrock    │  │      S3      │     │
│  │  (Device     │  │  (Advanced   │  │  (Analytics  │     │
│  │   Mgmt)      │  │     AI)      │  │    Data)     │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Architecture Principles

1. **Offline-First**: Core functionality works without internet
2. **Graceful Degradation**: System falls back to edge when cloud unavailable
3. **Intelligent Routing**: Queries routed based on complexity and connectivity
4. **Privacy-Preserving**: Data anonymized before cloud transmission
5. **Resource-Efficient**: Optimized for Raspberry Pi constraints

## 2. Component Design

### 2.1 Voice Interface Layer

#### 2.1.1 Speech-to-Text (STT) Component

**Technology**: Whisper (quantized model)

**Responsibilities**:
- Capture audio from microphone
- Detect voice activity (VAD)
- Convert speech to text in Hindi/Bhojpuri
- Handle background noise

**Implementation**:
```python
class STTEngine:
    def __init__(self, model_path: str, language: str):
        """Initialize Whisper model"""
        
    def transcribe(self, audio_data: bytes) -> str:
        """Convert audio to text"""
        
    def detect_wake_word(self, audio_stream) -> bool:
        """Detect activation phrase"""
```

**Model**: Whisper Tiny/Base (quantized to INT8)
- Size: ~75MB
- Languages: Hindi, Bhojpuri (via Hindi model)
- Latency: <1 second

#### 2.1.2 Text-to-Speech (TTS) Component

**Technology**: Piper (offline), Amazon Polly (online fallback)

**Responsibilities**:
- Convert text responses to speech
- Support multiple voices
- Adjust speech rate for clarity

**Implementation**:
```python
class TTSEngine:
    def __init__(self, offline_model: str, use_cloud: bool):
        """Initialize TTS engines"""
        
    def synthesize(self, text: str, language: str) -> bytes:
        """Convert text to audio"""
        
    def set_voice_parameters(self, rate: float, pitch: float):
        """Adjust voice characteristics"""
```

### 2.2 AI Processing Layer

#### 2.2.1 Query Router

**Responsibilities**:
- Classify query complexity
- Check internet connectivity
- Route to edge or cloud processor
- Handle fallback scenarios

**Implementation**:
```python
class QueryRouter:
    def __init__(self, edge_engine, cloud_engine, connectivity_checker):
        """Initialize routing components"""
        
    def route_query(self, query: str, context: dict) -> str:
        """
        Determine processing location and execute
        Returns: response text
        """
        
    def classify_complexity(self, query: str) -> str:
        """Returns: 'simple' or 'complex'"""
        
    def check_connectivity(self) -> bool:
        """Check internet availability"""
```

**Routing Logic**:
- Simple queries (weather, prices, schemes): Edge
- Complex queries (multi-step reasoning): Cloud (if available)
- Emergency queries: Edge (immediate response)

#### 2.2.2 Edge AI Engine

**Technology**: Quantized Gemma-2B or Llama-3-8B

**Responsibilities**:
- Process queries locally
- Access local knowledge base
- Generate responses in local language

**Implementation**:
```python
class EdgeAIEngine:
    def __init__(self, model_path: str, tokenizer_path: str):
        """Load quantized LLM"""
        
    def generate_response(self, query: str, context: dict) -> str:
        """Generate response using local model"""
        
    def load_prompt_template(self, template_name: str) -> str:
        """Load language-specific prompts"""
```

**Model Configuration**:
- Format: GGUF (4-bit quantization)
- Context window: 2048 tokens
- Temperature: 0.7
- Max tokens: 256

#### 2.2.3 Cloud AI Engine

**Technology**: Amazon Bedrock (Claude or Llama models)

**Responsibilities**:
- Handle complex queries
- Provide higher accuracy responses
- Access broader knowledge

**Implementation**:
```python
class CloudAIEngine:
    def __init__(self, bedrock_client, model_id: str):
        """Initialize Bedrock client"""
        
    def generate_response(self, query: str, context: dict) -> str:
        """Generate response using Bedrock"""
        
    def handle_timeout(self, timeout_seconds: int):
        """Fallback to edge on timeout"""
```

### 2.3 Knowledge Management Layer

#### 2.3.1 Yojna-Scanner (RAG System)

**Responsibilities**:
- Store government scheme information
- Match schemes to user profiles
- Retrieve relevant scheme details

**Implementation**:
```python
class YojnaScanner:
    def __init__(self, vector_db_path: str, embeddings_model: str):
        """Initialize vector database and embeddings"""
        
    def find_eligible_schemes(self, user_profile: dict) -> list:
        """Match user to schemes"""
        
    def get_scheme_details(self, scheme_id: str) -> dict:
        """Retrieve scheme information"""
        
    def update_scheme_database(self, schemes_data: list):
        """Sync new schemes from cloud"""
```

**Vector Database**: ChromaDB (embedded)
**Embeddings**: Sentence-Transformers (multilingual-MiniLM)

#### 2.3.2 Local Data Store

**Technology**: SQLite + JSON files

**Schema**:
```sql
-- User profiles (anonymized)
CREATE TABLE user_profiles (
    device_id TEXT PRIMARY KEY,
    language_preference TEXT,
    location TEXT,
    occupation TEXT
);

-- Cached data
CREATE TABLE cached_data (
    data_type TEXT,
    data_key TEXT,
    data_value TEXT,
    last_updated TIMESTAMP,
    PRIMARY KEY (data_type, data_key)
);

-- Query history (for offline analytics)
CREATE TABLE query_log (
    query_id TEXT PRIMARY KEY,
    timestamp TIMESTAMP,
    query_text_hash TEXT,
    response_source TEXT,
    response_time_ms INTEGER
);
```

### 2.4 Cloud Integration Layer

#### 2.4.1 AWS IoT Core Integration

**Responsibilities**:
- Device authentication
- Bidirectional messaging
- Device shadow management
- OTA updates

**Implementation**:
```python
class IoTCoreClient:
    def __init__(self, thing_name: str, certificates_path: str):
        """Initialize IoT Core connection"""
        
    def publish_telemetry(self, data: dict):
        """Send device metrics"""
        
    def subscribe_to_updates(self, callback):
        """Receive configuration updates"""
        
    def sync_data(self):
        """Synchronize cached data"""
```

**MQTT Topics**:
- `dostai/{device_id}/telemetry` - Device metrics
- `dostai/{device_id}/queries` - Query logs
- `dostai/{device_id}/updates` - Data updates
- `dostai/{device_id}/commands` - Remote commands

#### 2.4.2 Data Sync Manager

**Responsibilities**:
- Detect connectivity
- Prioritize sync operations
- Handle sync failures
- Manage bandwidth

**Implementation**:
```python
class DataSyncManager:
    def __init__(self, iot_client, s3_client):
        """Initialize sync components"""
        
    def sync_when_available(self):
        """Background sync process"""
        
    def prioritize_updates(self) -> list:
        """Order: weather > prices > schemes > analytics"""
        
    def upload_analytics(self, anonymized_data: dict):
        """Upload to S3"""
```

### 2.5 Emergency SOS System

**Responsibilities**:
- Detect emergency keywords
- Trigger alerts
- Notify contacts
- Log incidents

**Implementation**:
```python
class EmergencySOSSystem:
    def __init__(self, contacts: list, sms_gateway):
        """Initialize emergency system"""
        
    def detect_emergency(self, query: str) -> bool:
        """Check for emergency keywords"""
        
    def trigger_alert(self, location: str, user_info: dict):
        """Send emergency notifications"""
        
    def send_sms(self, message: str, recipients: list):
        """Use local GSM module"""
```

**Emergency Keywords**: "आपातकाल" (emergency), "मदद" (help), "डॉक्टर" (doctor)

## 3. Data Flow

### 3.1 Offline Query Flow

```
User Voice → STT → Query Router → Edge AI Engine → TTS → Speaker
                                        ↓
                                  Local Knowledge Base
```

### 3.2 Online Query Flow

```
User Voice → STT → Query Router → Cloud AI Engine (Bedrock) → TTS → Speaker
                        ↓                    ↓
                  Connectivity Check    AWS IoT Core
```

### 3.3 Sync Flow

```
Connectivity Detected → Data Sync Manager → AWS IoT Core
                                ↓
                        ┌───────┴────────┐
                        ↓                ↓
                  Fetch Updates    Upload Analytics
                  (Weather, Prices)     (S3)
```

## 4. API Specifications

### 4.1 Internal APIs

#### Voice Interface API

```python
# STT API
def transcribe_audio(audio_stream: AudioStream) -> TranscriptionResult:
    """
    Args:
        audio_stream: Raw audio data
    Returns:
        TranscriptionResult(text: str, confidence: float, language: str)
    """

# TTS API
def synthesize_speech(text: str, language: str, voice: str) -> AudioData:
    """
    Args:
        text: Text to convert
        language: 'hi' or 'bho'
        voice: Voice identifier
    Returns:
        AudioData(audio_bytes: bytes, duration: float)
    """
```

#### AI Processing API

```python
# Query Processing
def process_query(query: str, context: QueryContext) -> QueryResponse:
    """
    Args:
        query: User question
        context: QueryContext(user_id, location, history)
    Returns:
        QueryResponse(answer: str, source: str, confidence: float)
    """

# Yojna Scanner API
def find_schemes(user_profile: UserProfile) -> List[Scheme]:
    """
    Args:
        user_profile: UserProfile(occupation, income, location, age)
    Returns:
        List of matching Scheme objects
    """
```

### 4.2 AWS Integration APIs

#### IoT Core Messages

```json
// Telemetry Message
{
  "device_id": "dostai_device_001",
  "timestamp": "2026-02-15T10:30:00Z",
  "metrics": {
    "cpu_usage": 45.2,
    "memory_usage": 62.1,
    "temperature": 52.3,
    "queries_processed": 127
  }
}

// Data Update Request
{
  "device_id": "dostai_device_001",
  "request_type": "data_update",
  "data_types": ["weather", "crop_prices", "schemes"]
}

// Data Update Response
{
  "update_id": "upd_20260215_001",
  "data_type": "crop_prices",
  "data": {
    "wheat": {"price": 2100, "unit": "quintal", "market": "Patna"},
    "rice": {"price": 1950, "unit": "quintal", "market": "Patna"}
  },
  "valid_until": "2026-02-16T10:30:00Z"
}
```

#### Bedrock API

```python
# Bedrock Request
{
    "modelId": "anthropic.claude-3-haiku-20240307-v1:0",
    "contentType": "application/json",
    "accept": "application/json",
    "body": {
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 256,
        "messages": [
            {
                "role": "user",
                "content": "Translated query in English"
            }
        ],
        "system": "You are DostAI, helping rural Indian farmers..."
    }
}
```

## 5. Security Design

### 5.1 Device Security

- AWS IoT certificates stored in secure location
- Private keys encrypted at rest
- Certificate rotation every 90 days
- Secure boot enabled on Raspberry Pi

### 5.2 Data Privacy

- User queries hashed before logging
- Personal identifiers removed before cloud sync
- Location data generalized to district level
- Voice recordings not stored

### 5.3 Network Security

- TLS 1.3 for all cloud communication
- Certificate-based device authentication
- MQTT over TLS for IoT Core
- API request signing

## 6. Deployment Architecture

### 6.1 Device Setup

```
1. Flash Raspberry Pi OS Lite
2. Install dependencies (Python, models, libraries)
3. Configure AWS IoT certificates
4. Download initial data cache
5. Test voice interface
6. Deploy to field
```

### 6.2 AWS Infrastructure

```
- IoT Core: Device registry and message broker
- Lambda: Process incoming telemetry
- S3: Store anonymized analytics
- Bedrock: AI model inference
- CloudWatch: Monitoring and alerts
- DynamoDB: Device metadata
```

### 6.3 Monitoring

**Device Metrics**:
- CPU/Memory usage
- Query response times
- Model inference latency
- Sync success rate

**Cloud Metrics**:
- IoT Core message throughput
- Bedrock API latency
- S3 storage usage
- Lambda execution times

## 7. Correctness Properties

### 7.1 Voice Interface Properties

**Property 1.1**: Voice Recognition Accuracy
- For any clear audio input in Hindi/Bhojpuri, STT accuracy must be ≥85%

**Property 1.2**: Response Latency
- For any edge query, response time must be ≤3 seconds

### 7.2 Hybrid Processing Properties

**Property 2.1**: Offline Availability
- System must respond to queries even when internet is unavailable

**Property 2.2**: Graceful Degradation
- If cloud processing fails, system must fallback to edge processing within 2 seconds

**Property 2.3**: Query Routing Correctness
- Simple queries must be processed on edge
- Complex queries must be routed to cloud when available

### 7.3 Data Sync Properties

**Property 3.1**: Sync Consistency
- After successful sync, local cache must match cloud data

**Property 3.2**: Sync Idempotency
- Multiple sync operations must produce same result as single sync

### 7.4 Emergency SOS Properties

**Property 4.1**: Emergency Detection
- System must detect emergency keywords with 100% recall

**Property 4.2**: Alert Delivery
- Emergency alerts must be sent within 5 seconds of detection

### 7.5 Privacy Properties

**Property 5.1**: Data Anonymization
- No personally identifiable information must be sent to cloud

**Property 5.2**: Voice Data Retention
- Voice recordings must not be stored beyond processing

## 8. Testing Strategy

### 8.1 Unit Tests
- Individual component testing
- Mock external dependencies
- Test edge cases

### 8.2 Integration Tests
- End-to-end query flow
- Cloud sync operations
- Emergency alert system

### 8.3 Property-Based Tests
- Voice recognition accuracy across inputs
- Response time consistency
- Sync correctness

### 8.4 Field Tests
- Real-world deployment in villages
- User acceptance testing
- Performance under actual conditions

## 9. Technology Stack Summary

### Edge (Raspberry Pi)
- **OS**: Raspberry Pi OS Lite (64-bit)
- **Runtime**: Python 3.11
- **STT**: Whisper (quantized)
- **TTS**: Piper
- **LLM**: Gemma-2B or Llama-3-8B (GGUF 4-bit)
- **Vector DB**: ChromaDB
- **Database**: SQLite
- **Orchestration**: LangChain

### Cloud (AWS)
- **Device Management**: AWS IoT Core
- **AI Models**: Amazon Bedrock
- **Storage**: Amazon S3
- **TTS (Optional)**: Amazon Polly
- **Compute**: AWS Lambda
- **Monitoring**: CloudWatch

### Libraries
- `boto3` - AWS SDK
- `paho-mqtt` - MQTT client
- `llama-cpp-python` - LLM inference
- `faster-whisper` - STT
- `piper-tts` - TTS
- `langchain` - LLM orchestration
- `chromadb` - Vector database
- `sentence-transformers` - Embeddings

## 10. Future Enhancements (Post-V1)

- Multi-user voice recognition
- Image-based crop disease detection
- Real-time translation between dialects
- Mobile app for family members
- Solar power integration
- Mesh networking for village-wide deployment
