# Quickstart Guide: Google Cloud Research Platform

**Feature**: 001-google-cloud-research
**Date**: 2025-09-16
**Purpose**: Validate core user stories through manual testing scenarios

## Prerequisites

### Infrastructure
- [ ] Google Cloud Project created with billing enabled
- [ ] Required APIs enabled: Cloud Run, Cloud SQL, Cloud Storage, Document AI, Vertex AI, Speech-to-Text, Pub/Sub
- [ ] VPC network configured with private Google access
- [ ] Cloud KMS key ring and CMEK key created
- [ ] PostgreSQL 16 instance with pgvector extension installed

### Services
- [ ] Document Ingestion Service deployed to Cloud Run
- [ ] Research Generation Service deployed to Cloud Run
- [ ] Client Portal Service deployed to Cloud Run
- [ ] Database schema deployed and migrated
- [ ] Service accounts configured with least privilege

### Test Data
- [ ] Sample companies loaded (TICKER: EXPL, ISIN: DE0000000001)
- [ ] Test analyst profile created (analyst_id: "test_analyst_001")
- [ ] Test client account created (username: "test_client", client_type: "institutional")
- [ ] Sample financial document URLs available for ingestion

## User Story 1: Document Ingestion & Processing

**Story**: Analyst initiates document ingestion and processing pipeline

### Steps
1. **Initiate Document Ingestion**
   ```bash
   curl -X POST https://ingestion-service.example.com/v1/documents/ingest \
     -H "Content-Type: application/json" \
     -d '{
       "company_id": 1,
       "source_url": "https://example.com/annual-report-2024.pdf",
       "document_type": "annual_report",
       "source_type": "ir_website"
     }'
   ```

   **Expected Response**: `202 Accepted` with `document_id` and status `pending`

2. **Monitor Processing Status**
   ```bash
   curl https://ingestion-service.example.com/v1/documents/{document_id}/status
   ```

   **Expected Progression**:
   - `pending` → `processing` → `completed`
   - `progress_percentage`: 0 → 50 → 100
   - `chunks_extracted` > 0 when completed

3. **Verify Database State**
   ```sql
   -- Check document was stored
   SELECT id, title, extraction_status, chunks_count
   FROM financial_documents
   WHERE id = {document_id};

   -- Check chunks were created with embeddings
   SELECT COUNT(*), AVG(extraction_confidence)
   FROM document_chunks
   WHERE document_id = {document_id};
   ```

   **Expected Results**:
   - Document record with `extraction_status = 'completed'`
   - Multiple chunks with non-null embeddings
   - Average confidence > 0.7

### Success Criteria
- ✅ Document ingested without errors
- ✅ Structured data extracted via Document AI
- ✅ Text chunks created with vector embeddings
- ✅ Audit trail created for ingestion event

## User Story 2: Voice Insight Integration

**Story**: Analyst records voice insights that get incorporated into reports

### Steps
1. **Upload Voice Recording**
   ```bash
   curl -X POST https://research-service.example.com/v1/voice/record \
     -H "Content-Type: multipart/form-data" \
     -F "analyst_id=test_analyst_001" \
     -F "company_id=1" \
     -F "recording_type=solo_insight" \
     -F "title=Q4 Performance Analysis" \
     -F "audio_file=@test_recording.wav"
   ```

   **Expected Response**: `201 Created` with `recording_id` and transcription status

2. **Monitor Transcription**
   ```bash
   curl https://research-service.example.com/v1/voice/{recording_id}/transcript
   ```

   **Expected Result**:
   - Status progresses to `completed`
   - Multiple transcript segments with speaker identification
   - Confidence scores > 0.8 for financial terminology

3. **Verify Voice Profile Building**
   ```sql
   -- Check voice data was stored
   SELECT COUNT(*), AVG(confidence_score)
   FROM voice_transcripts
   WHERE recording_id = {recording_id};

   -- Check analyst profile updated
   SELECT voice_insights_count, analytical_style
   FROM analyst_profiles
   WHERE analyst_id = 'test_analyst_001';
   ```

### Success Criteria
- ✅ Audio file processed via Speech-to-Text API
- ✅ Transcript generated with speaker diarization
- ✅ Content embedded for semantic search
- ✅ Analyst profile updated with insights

## User Story 3: AI Research Report Generation

**Story**: System performs autonomous research and generates comprehensive report

### Steps
1. **Initiate Report Generation**
   ```bash
   curl -X POST https://research-service.example.com/v1/reports/generate \
     -H "Content-Type: application/json" \
     -d '{
       "company_id": 1,
       "analyst_id": "test_analyst_001",
       "report_type": "initiation",
       "include_voice_insights": true,
       "target_length": "standard"
     }'
   ```

   **Expected Response**: `202 Accepted` with `report_id` and estimated completion time

2. **Monitor Report Generation**
   ```bash
   curl https://research-service.example.com/v1/reports/{report_id}/status
   ```

   **Expected Progression**:
   - `queued` → `research` → `analysis` → `writing` → `compliance` → `completed`
   - `sources_analyzed` > 5 (documents + market intelligence)
   - `peer_companies_analyzed` >= 3
   - `compliance_status = 'passed'`

3. **Retrieve Generated Report**
   ```bash
   # Get JSON version
   curl https://research-service.example.com/v1/reports/{report_id}

   # Get PDF version
   curl https://research-service.example.com/v1/reports/{report_id}?format=pdf
   ```

   **Expected Content**:
   - Investment thesis with clear recommendation (BUY/HOLD/SELL)
   - Price target with rationale
   - Comprehensive source attribution
   - Voice insights integrated naturally

4. **Verify Report Quality**
   ```sql
   -- Check report was created
   SELECT title, recommendation, price_target, compliance_status
   FROM research_reports
   WHERE id = {report_id};

   -- Check source attribution
   SELECT jsonb_array_length(source_attribution) as source_count
   FROM research_reports
   WHERE id = {report_id};
   ```

### Success Criteria
- ✅ Autonomous research performed using external tools/APIs
- ✅ Peer analysis completed with market sentiment
- ✅ Voice insights incorporated into analysis
- ✅ Compliance validation passed
- ✅ Report stored in WORM-compliant storage

## User Story 4: Client Portal Access

**Story**: Institutional client accesses and downloads research reports securely

### Steps
1. **Client Authentication**
   ```bash
   curl -X POST https://portal.example.com/api/v1/auth/login \
     -H "Content-Type: application/json" \
     -d '{
       "username": "test_client",
       "password": "secure_password_123"
     }'
   ```

   **Expected Response**: Session token and client permissions

2. **Browse Available Reports**
   ```bash
   curl -H "Authorization: Basic dGVzdF9jbGllbnQ6c2VjdXJlX3Bhc3N3b3JkXzEyMw==" \
     https://portal.example.com/api/v1/reports?company_id=1
   ```

   **Expected Results**: List of reports filtered by client access level

3. **Access Specific Report**
   ```bash
   curl -H "Authorization: Basic dGVzdF9jbGllbnQ6c2VjdXJlX3Bhc3N3b3JkXzEyMw==" \
     https://portal.example.com/api/v1/reports/{report_id}
   ```

   **Expected Content**: Full report with compliance disclaimers

4. **Download Report**
   ```bash
   curl -H "Authorization: Basic dGVzdF9jbGllbnQ6c2VjdXJlX3Bhc3N3b3JkXzEyMw==" \
     https://portal.example.com/api/v1/reports/{report_id}/download?format=pdf \
     -o report.pdf
   ```

5. **Verify Audit Trail**
   ```sql
   -- Check access was logged
   SELECT event_type, user_id, entity_id, occurred_at
   FROM audit_events
   WHERE event_type = 'report_accessed'
     AND entity_id = {report_id}
   ORDER BY occurred_at DESC;
   ```

### Success Criteria
- ✅ Client authenticated with username/password
- ✅ Reports filtered by subscription level and permissions
- ✅ Secure access with comprehensive audit logging
- ✅ Multiple download formats available

## Integration Test Scenarios

### End-to-End Workflow
**Duration**: ~30 minutes for full pipeline

1. Upload company documents (5 min processing)
2. Record analyst voice insight (2 min transcription)
3. Generate research report (15 min for quality analysis)
4. Client accesses report (immediate)
5. Verify all audit trails and compliance records

### Error Handling Tests

1. **Document Processing Failure**
   - Upload corrupted PDF
   - Verify graceful failure with error message
   - Check no partial data in database

2. **Voice Transcription Failure**
   - Upload audio with poor quality
   - Verify fallback handling
   - Check analyst profile not corrupted

3. **Compliance Failure**
   - Generate report with flagged content
   - Verify report blocked from publication
   - Check compliance team notification

### Performance Validation

1. **Batch Document Processing**
   - Ingest 50 documents simultaneously
   - Verify all complete within SLA (2 hours)
   - Check database performance remains stable

2. **Concurrent Report Generation**
   - Generate 5 reports in parallel
   - Verify quality not degraded by parallel processing
   - Check resource utilization stays within limits

## Success Metrics

### Quality Metrics
- Document extraction accuracy > 90%
- Voice transcription accuracy > 95% for financial terms
- Report compliance pass rate > 98%
- Source attribution completeness > 95%

### Performance Metrics
- Document processing: < 5 minutes per document
- Voice transcription: < 30 seconds per minute of audio
- Report generation: < 30 minutes for standard report
- Portal response time: < 200ms for report list

### Security Metrics
- 100% audit trail coverage
- 0 data sovereignty violations
- 100% encryption compliance (data at rest and in transit)
- 0 unauthorized access attempts succeeded

## Troubleshooting

### Common Issues

1. **Document AI Service Unavailable**
   - Check VPC Service Controls configuration
   - Verify CMEK key permissions
   - Review service account IAM roles

2. **Vector Search Performance Poor**
   - Check HNSW index creation on embeddings
   - Verify pgvector extension enabled
   - Review query optimization

3. **Voice Transcription Errors**
   - Validate audio format (WAV, 16kHz recommended)
   - Check Speech-to-Text API quotas
   - Review custom vocabulary configuration

4. **Client Portal Authentication Issues**
   - Verify user account active status
   - Check client subscription level
   - Review password policy compliance

### Monitoring Commands

```bash
# Check service health
curl https://ingestion-service.example.com/health
curl https://research-service.example.com/health
curl https://portal.example.com/health

# Check database connections
psql -h $DB_HOST -U $DB_USER -d research_db -c "SELECT 1;"

# Monitor processing queues
gcloud pubsub topics list --filter="name:rr.*"
gcloud pubsub subscriptions list --filter="topic:rr.*"
```

**Quickstart Validation Complete** ✅
Ready for task generation and implementation phase.