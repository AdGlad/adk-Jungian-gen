# ADK Jungian Book Generator

A Google ADK-based system for generating books inspired by Carl Jung's psychology, with parallel chapter writing and EPUB conversion.

## Architecture

- **Agent Workflow** (`p_book_gen/`): Sequential agents generate book content using Jungian quotes
- **Cloud Run Service** (`main.py`): Converts Markdown to EPUB using pandoc

## Setup

### Prerequisites

- Python 3.11+
- Google Cloud Project with billing enabled
- GCS bucket for storage
- pandoc (for EPUB conversion)

### Authentication

This application uses **Application Default Credentials (ADC)** for all Google Cloud services. No API keys required.

#### Local Development

1. Install Google Cloud SDK
2. Authenticate: `gcloud auth application-default login`
3. Set project: `gcloud config set project YOUR_PROJECT_ID`
4. Set environment variables:
   ```bash
   export GOOGLE_CLOUD_PROJECT=YOUR_PROJECT_ID
   export GOOGLE_CLOUD_LOCATION=us-central1
   export GOOGLE_GENAI_USE_VERTEXAI=true
   export BOOK_GEN_BUCKET=your-bucket-name
   ```

#### Production (Cloud Run)

Attach a service account with the following permissions:
- Cloud Storage Admin (for GCS access)
- Vertex AI User (for ADK/genai)
- Cloud Run Invoker (for Eventarc triggers)

### Environment Variables

- `BOOK_GEN_BUCKET`: GCS bucket name for storing manuscripts and EPUBs
- `GOOGLE_CLOUD_PROJECT`: Your GCP project ID (required for Vertex AI)
- `GOOGLE_CLOUD_LOCATION`: GCP region for Vertex AI (default: us-central1)
- `GOOGLE_GENAI_USE_VERTEXAI`: Set to 'true' to use Vertex AI instead of Google AI API

### Installation

```bash
pip install -r requirements.txt
```

### Usage

1. Create a JSON spec file (see `working_minds_spec.json` for example)
2. Run the ADK agent with the JSON as input
3. The system will generate chapters, merge into manuscript, and queue EPUB conversion

### Deployment

```bash
# Build and deploy Cloud Run service
gcloud run deploy epub-builder \
  --source . \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars BOOK_GEN_BUCKET=your-bucket-name,GOOGLE_CLOUD_PROJECT=your-project-id,GOOGLE_CLOUD_LOCATION=us-central1,GOOGLE_GENAI_USE_VERTEXAI=true
```

Set up Eventarc trigger for GCS bucket events to automatically process job files.