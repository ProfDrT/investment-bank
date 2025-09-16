# Data Model: Google Cloud-based Automated Research Platform

**Feature**: 001-google-cloud-research
**Date**: 2025-09-16
**Status**: Phase 1 Complete

## Core Entities

### 1. Financial Documents
**Purpose**: Source documents from various data feeds
**Storage**: PostgreSQL + GCS

```sql
CREATE TABLE financial_documents (
    id BIGSERIAL PRIMARY KEY,
    company_id BIGINT NOT NULL REFERENCES companies(id),
    source_type VARCHAR(50) NOT NULL, -- 'ir_website', 'regulatory_filing', 'external_feed'
    document_type VARCHAR(50) NOT NULL, -- 'annual_report', 'earnings_call', 'press_release'
    source_url TEXT,
    title TEXT NOT NULL,
    publication_date DATE NOT NULL,
    ingestion_timestamp TIMESTAMPTZ DEFAULT NOW(),
    extraction_status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'processing', 'completed', 'failed'
    gcs_raw_path TEXT, -- Original document in GCS
    gcs_processed_path TEXT, -- Processed/structured data
    document_hash VARCHAR(64) UNIQUE, -- For deduplication
    metadata JSONB, -- Flexible metadata storage
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_financial_documents_company_date ON financial_documents(company_id, publication_date);
CREATE INDEX idx_financial_documents_status ON financial_documents(extraction_status);
CREATE INDEX idx_financial_documents_type ON financial_documents(document_type, source_type);
```

### 2. Document Chunks
**Purpose**: Processed text segments with embeddings for semantic search
**Storage**: PostgreSQL with pgvector

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE document_chunks (
    id BIGSERIAL PRIMARY KEY,
    document_id BIGINT NOT NULL REFERENCES financial_documents(id),
    chunk_index INTEGER NOT NULL, -- Order within document
    content TEXT NOT NULL,
    content_type VARCHAR(50) NOT NULL, -- 'paragraph', 'table', 'list', 'header'
    page_number INTEGER,
    embedding VECTOR(768), -- Vertex AI embedding dimension
    extraction_confidence DECIMAL(3,2), -- Document AI confidence score
    processed_at TIMESTAMPTZ DEFAULT NOW(),

    CONSTRAINT unique_document_chunk UNIQUE(document_id, chunk_index)
);

CREATE INDEX idx_document_chunks_embedding ON document_chunks USING hnsw (embedding vector_cosine_ops);
CREATE INDEX idx_document_chunks_document ON document_chunks(document_id);
CREATE INDEX idx_document_chunks_type ON document_chunks(content_type);
```

### 3. Companies
**Purpose**: Investment targets with associated data and relationships
**Storage**: PostgreSQL

```sql
CREATE TABLE companies (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    ticker VARCHAR(20),
    isin VARCHAR(12),
    country VARCHAR(3), -- ISO country code
    sector VARCHAR(100),
    industry VARCHAR(100),
    market_cap BIGINT, -- In base currency cents
    currency VARCHAR(3) DEFAULT 'EUR',
    is_public BOOLEAN DEFAULT true,
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_companies_ticker ON companies(ticker);
CREATE INDEX idx_companies_isin ON companies(isin);
CREATE INDEX idx_companies_sector ON companies(sector, industry);
```

### 4. Peer Relationships
**Purpose**: Company peer analysis and competitive positioning
**Storage**: PostgreSQL

```sql
CREATE TABLE peer_relationships (
    id BIGSERIAL PRIMARY KEY,
    company_id BIGINT NOT NULL REFERENCES companies(id),
    peer_company_id BIGINT NOT NULL REFERENCES companies(id),
    relationship_type VARCHAR(50) NOT NULL, -- 'direct_competitor', 'industry_peer', 'comparable'
    similarity_score DECIMAL(3,2), -- 0-1 similarity rating
    rationale TEXT,
    created_by VARCHAR(50) NOT NULL, -- 'system' or analyst_id
    created_at TIMESTAMPTZ DEFAULT NOW(),

    CONSTRAINT unique_peer_relationship UNIQUE(company_id, peer_company_id, relationship_type),
    CONSTRAINT no_self_reference CHECK (company_id != peer_company_id)
);

CREATE INDEX idx_peer_relationships_company ON peer_relationships(company_id);
CREATE INDEX idx_peer_relationships_type ON peer_relationships(relationship_type);
```

### 5. Research Reports
**Purpose**: Generated investment analysis documents
**Storage**: PostgreSQL + GCS (WORM)

```sql
CREATE TABLE research_reports (
    id BIGSERIAL PRIMARY KEY,
    company_id BIGINT NOT NULL REFERENCES companies(id),
    analyst_id VARCHAR(50) NOT NULL,
    title VARCHAR(500) NOT NULL,
    report_type VARCHAR(50) NOT NULL, -- 'initiation', 'update', 'flash_note'
    investment_thesis TEXT NOT NULL,
    recommendation VARCHAR(20) NOT NULL, -- 'BUY', 'HOLD', 'SELL'
    price_target DECIMAL(10,2),
    price_target_currency VARCHAR(3) DEFAULT 'EUR',
    risk_rating VARCHAR(20), -- 'LOW', 'MEDIUM', 'HIGH'

    -- Compliance and audit
    compliance_status VARCHAR(20) DEFAULT 'draft', -- 'draft', 'review', 'approved', 'published'
    compliance_notes TEXT,
    gcs_report_path TEXT, -- Final report in WORM storage
    source_attribution JSONB, -- All sources used with timestamps

    -- Workflow tracking
    generation_started_at TIMESTAMPTZ,
    generation_completed_at TIMESTAMPTZ,
    published_at TIMESTAMPTZ,

    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_research_reports_company ON research_reports(company_id);
CREATE INDEX idx_research_reports_analyst ON research_reports(analyst_id);
CREATE INDEX idx_research_reports_status ON research_reports(compliance_status);
CREATE INDEX idx_research_reports_published ON research_reports(published_at);
```

### 6. Analyst Voice Data
**Purpose**: Audio recordings, transcripts, and analyst profiles
**Storage**: PostgreSQL + GCS

```sql
CREATE TABLE voice_recordings (
    id BIGSERIAL PRIMARY KEY,
    analyst_id VARCHAR(50) NOT NULL,
    company_id BIGINT REFERENCES companies(id), -- Optional: company-specific insight
    recording_type VARCHAR(50) NOT NULL, -- 'solo_insight', 'expert_interview', 'team_discussion'
    title VARCHAR(255),
    duration_seconds INTEGER,
    gcs_audio_path TEXT NOT NULL,
    transcript_status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'processing', 'completed', 'failed'

    recorded_at TIMESTAMPTZ NOT NULL,
    processed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE voice_transcripts (
    id BIGSERIAL PRIMARY KEY,
    recording_id BIGINT NOT NULL REFERENCES voice_recordings(id),
    speaker_id VARCHAR(50), -- For multi-speaker diarization
    sequence_number INTEGER NOT NULL,
    start_time_seconds DECIMAL(8,2),
    end_time_seconds DECIMAL(8,2),
    transcript_text TEXT NOT NULL,
    confidence_score DECIMAL(3,2),
    embedding VECTOR(768), -- For semantic search

    processed_at TIMESTAMPTZ DEFAULT NOW(),

    CONSTRAINT unique_recording_sequence UNIQUE(recording_id, sequence_number)
);

CREATE INDEX idx_voice_transcripts_recording ON voice_transcripts(recording_id);
CREATE INDEX idx_voice_transcripts_embedding ON voice_transcripts USING hnsw (embedding vector_cosine_ops);

CREATE TABLE analyst_profiles (
    analyst_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    sectors JSONB, -- Array of sectors they cover
    analytical_style JSONB, -- Preferences and patterns
    voice_insights_count INTEGER DEFAULT 0,
    reports_count INTEGER DEFAULT 0,

    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 7. Market Research Data
**Purpose**: External research sources and intelligence
**Storage**: PostgreSQL

```sql
CREATE TABLE research_sources (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    source_type VARCHAR(50) NOT NULL, -- 'market_data_api', 'news_feed', 'regulatory_db'
    base_url TEXT,
    reliability_rating INTEGER, -- 1-10 scale
    access_method VARCHAR(50), -- 'api', 'web_scraping', 'manual'
    is_active BOOLEAN DEFAULT true,
    rate_limit_per_hour INTEGER,

    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE market_intelligence (
    id BIGSERIAL PRIMARY KEY,
    company_id BIGINT REFERENCES companies(id),
    source_id BIGINT NOT NULL REFERENCES research_sources(id),
    data_type VARCHAR(50) NOT NULL, -- 'sentiment', 'price_data', 'news', 'analyst_estimate'
    content JSONB NOT NULL,
    sentiment_score DECIMAL(3,2), -- -1 to 1 for sentiment data
    relevance_score DECIMAL(3,2), -- 0 to 1

    extracted_at TIMESTAMPTZ NOT NULL,
    processed_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_market_intelligence_company ON market_intelligence(company_id);
CREATE INDEX idx_market_intelligence_type ON market_intelligence(data_type);
CREATE INDEX idx_market_intelligence_extracted ON market_intelligence(extracted_at);
```

### 8. Audit Events
**Purpose**: Comprehensive audit trail for regulatory compliance
**Storage**: PostgreSQL (immutable logs)

```sql
CREATE TABLE audit_events (
    id BIGSERIAL PRIMARY KEY,
    event_type VARCHAR(100) NOT NULL, -- 'document_ingested', 'report_generated', 'user_access'
    entity_type VARCHAR(50) NOT NULL, -- 'company', 'document', 'report'
    entity_id BIGINT NOT NULL,
    user_id VARCHAR(50),
    action VARCHAR(100) NOT NULL,
    details JSONB,
    ip_address INET,
    user_agent TEXT,

    occurred_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- Immutability constraint
    CONSTRAINT no_updates CHECK (false) DEFERRABLE INITIALLY DEFERRED
);

CREATE INDEX idx_audit_events_type ON audit_events(event_type);
CREATE INDEX idx_audit_events_entity ON audit_events(entity_type, entity_id);
CREATE INDEX idx_audit_events_user ON audit_events(user_id);
CREATE INDEX idx_audit_events_occurred ON audit_events(occurred_at);
```

### 9. Clients
**Purpose**: Institutional users with access controls
**Storage**: PostgreSQL

```sql
CREATE TABLE clients (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    client_type VARCHAR(50) NOT NULL, -- 'institutional', 'corporate', 'individual'
    subscription_level VARCHAR(50) NOT NULL, -- 'basic', 'premium', 'enterprise'
    country VARCHAR(3), -- For jurisdiction-specific compliance
    is_active BOOLEAN DEFAULT true,

    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE client_users (
    id BIGSERIAL PRIMARY KEY,
    client_id BIGINT NOT NULL REFERENCES clients(id),
    username VARCHAR(100) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL DEFAULT 'viewer', -- 'viewer', 'admin'
    last_login TIMESTAMPTZ,
    is_active BOOLEAN DEFAULT true,

    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_client_users_client ON client_users(client_id);
CREATE INDEX idx_client_users_username ON client_users(username);
```

### 10. AI Models and Prompts
**Purpose**: Version-controlled AI configurations
**Storage**: PostgreSQL

```sql
CREATE TABLE ai_models (
    id BIGSERIAL PRIMARY KEY,
    model_name VARCHAR(100) NOT NULL,
    model_version VARCHAR(50) NOT NULL,
    provider VARCHAR(50) NOT NULL, -- 'vertex_ai', 'openai'
    model_type VARCHAR(50) NOT NULL, -- 'embedding', 'llm', 'speech'
    configuration JSONB,
    is_active BOOLEAN DEFAULT true,

    created_at TIMESTAMPTZ DEFAULT NOW(),

    CONSTRAINT unique_model_version UNIQUE(model_name, model_version)
);

CREATE TABLE prompt_templates (
    id BIGSERIAL PRIMARY KEY,
    template_name VARCHAR(100) NOT NULL,
    section VARCHAR(100) NOT NULL, -- 'executive_summary', 'valuation', 'risks'
    template_text TEXT NOT NULL,
    variables JSONB, -- Expected variables and descriptions
    analyst_id VARCHAR(50), -- NULL for system templates
    is_active BOOLEAN DEFAULT true,

    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_prompt_templates_section ON prompt_templates(section);
CREATE INDEX idx_prompt_templates_analyst ON prompt_templates(analyst_id);
```

## Relationships and Constraints

### Entity Relationships
```
Companies (1) ← (N) Financial Documents ← (N) Document Chunks
Companies (N) ← → (N) Peer Relationships
Companies (1) ← (N) Research Reports
Companies (1) ← (N) Voice Recordings (optional)
Companies (1) ← (N) Market Intelligence

Analysts (1) ← (N) Voice Recordings
Analysts (1) ← (N) Research Reports
Analysts (1) ← (N) Prompt Templates

Clients (1) ← (N) Client Users
```

### Data Validation Rules
- Document hash uniqueness prevents duplicate ingestion
- ISIN and ticker validation for company identification
- Embedding dimensions must match model configuration
- Audit events are immutable (constraint prevents updates)
- Price targets must have corresponding currency
- Confidence scores range 0.0-1.0
- Sentiment scores range -1.0 to 1.0

### State Transitions
- Document extraction: pending → processing → completed/failed
- Research report compliance: draft → review → approved → published
- Voice transcript processing: pending → processing → completed/failed

## Security and Compliance

### Data Classification
- **Public**: Company basic info, published research reports
- **Internal**: Draft reports, analyst profiles, system configuration
- **Confidential**: Voice recordings, client data, audit logs
- **Restricted**: Encryption keys, authentication credentials

### Encryption Strategy
- All columns containing sensitive data encrypted with CMEK
- GCS objects encrypted with customer-managed keys
- Database connections over TLS with certificate validation
- Application-level encryption for voice transcripts

### Retention Policies
- Research reports: Configurable by jurisdiction (7+ years typical)
- Voice recordings: 5 years for regulatory compliance
- Audit logs: 10 years minimum, immutable storage
- Document chunks: Retain as long as source documents exist

**Ready for Contract Generation (Phase 1 continuation)**