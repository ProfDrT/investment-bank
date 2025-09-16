# Tasks: Google Cloud-based Automated Research Platform

**Input**: Design documents from `/specs/001-google-cloud-research/`
**Prerequisites**: plan.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅, quickstart.md ✅

## Execution Summary

**Tech Stack**: Python 3.11+ with LangGraph, FastAPI, PostgreSQL 16 + pgvector, Google Cloud Services
**Structure**: Web application (backend services + frontend portal)
**Architecture**: Microservices on Cloud Run with Pub/Sub communication

**Generated Tasks**:
- 3 contract files → 9 contract tests
- 10 entities → 10 model tasks
- 4 user stories → 4 integration tests
- 3 microservices → multiple implementation tasks
- Total: 45 tasks across 5 phases

## Format: `[ID] [P?] Description`
- **[P]**: Can run in parallel (different files, no dependencies)
- File paths assume backend/src/ and frontend/src/ structure per plan.md

## Phase 3.1: Project Setup

- [ ] T001 Create backend and frontend project structure per Google Cloud architecture
- [ ] T002 Initialize Python backend with FastAPI, LangGraph, psycopg2, Google Cloud SDK dependencies
- [ ] T003 [P] Configure pytest, black, flake8 for backend development
- [ ] T004 [P] Configure Terraform modules for Google Cloud infrastructure
- [ ] T005 [P] Set up frontend React/TypeScript project with authentication components
- [ ] T006 [P] Configure CI/CD with Cloud Build and Artifact Registry

## Phase 3.2: Infrastructure & Database Setup

- [ ] T007 Deploy PostgreSQL 16 with pgvector extension on Cloud SQL
- [ ] T008 [P] Set up Google Cloud KMS with CMEK encryption keys
- [ ] T009 [P] Configure VPC network with private Google access and NAT
- [ ] T010 [P] Deploy Pub/Sub topics: rr.ingest, rr.model, rr.qc
- [ ] T011 Run database migration to create all tables from data-model.md

## Phase 3.3: Contract Tests First (TDD) ⚠️ MUST COMPLETE BEFORE 3.4
**CRITICAL: These tests MUST be written and MUST FAIL before ANY implementation**

### Document Ingestion API Tests
- [ ] T012 [P] Contract test POST /v1/documents/ingest in backend/tests/contract/test_ingestion_post.py
- [ ] T013 [P] Contract test GET /v1/documents/{id}/status in backend/tests/contract/test_ingestion_status.py
- [ ] T014 [P] Contract test POST /v1/documents/batch in backend/tests/contract/test_ingestion_batch.py

### Research API Tests
- [ ] T015 [P] Contract test POST /v1/reports/generate in backend/tests/contract/test_research_generate.py
- [ ] T016 [P] Contract test GET /v1/reports/{id}/status in backend/tests/contract/test_research_status.py
- [ ] T017 [P] Contract test GET /v1/reports/{id} in backend/tests/contract/test_research_get.py
- [ ] T018 [P] Contract test POST /v1/voice/record in backend/tests/contract/test_voice_upload.py
- [ ] T019 [P] Contract test GET /v1/voice/{id}/transcript in backend/tests/contract/test_voice_transcript.py

### Portal API Tests
- [ ] T020 [P] Contract test POST /api/v1/auth/login in backend/tests/contract/test_portal_auth.py
- [ ] T021 [P] Contract test GET /api/v1/reports in backend/tests/contract/test_portal_reports.py
- [ ] T022 [P] Contract test GET /api/v1/reports/{id} in backend/tests/contract/test_portal_report_detail.py

### Integration Tests (User Stories)
- [ ] T023 [P] Integration test document ingestion pipeline in backend/tests/integration/test_document_flow.py
- [ ] T024 [P] Integration test voice insight integration in backend/tests/integration/test_voice_flow.py
- [ ] T025 [P] Integration test AI report generation in backend/tests/integration/test_research_flow.py
- [ ] T026 [P] Integration test client portal access in backend/tests/integration/test_portal_flow.py

## Phase 3.4: Data Models (ONLY after tests are failing)

- [ ] T027 [P] FinancialDocument model in backend/src/models/financial_document.py
- [ ] T028 [P] DocumentChunk model with vector embedding support in backend/src/models/document_chunk.py
- [ ] T029 [P] Company model in backend/src/models/company.py
- [ ] T030 [P] PeerRelationship model in backend/src/models/peer_relationship.py
- [ ] T031 [P] ResearchReport model in backend/src/models/research_report.py
- [ ] T032 [P] VoiceRecording and VoiceTranscript models in backend/src/models/voice_data.py
- [ ] T033 [P] AnalystProfile model in backend/src/models/analyst_profile.py
- [ ] T034 [P] MarketIntelligence and ResearchSource models in backend/src/models/market_data.py
- [ ] T035 [P] Client and ClientUser models in backend/src/models/client.py
- [ ] T036 [P] AuditEvent model (immutable) in backend/src/models/audit_event.py

## Phase 3.5: Core Services

### Document Processing Service
- [ ] T037 DocumentIngestionService in backend/src/services/document_ingestion.py
- [ ] T038 DocumentAIProcessor for Google Document AI integration in backend/src/services/document_ai.py
- [ ] T039 EmbeddingService for Vertex AI embeddings in backend/src/services/embedding_service.py

### Research Generation Service
- [ ] T040 LangGraphResearchAgent in backend/src/services/research_agent.py
- [ ] T041 VoiceProcessingService for Speech-to-Text in backend/src/services/voice_processing.py
- [ ] T042 MarketResearchService for external data integration in backend/src/services/market_research.py
- [ ] T043 ComplianceValidationService in backend/src/services/compliance_service.py

### Portal Service
- [ ] T044 ClientAuthenticationService in backend/src/services/client_auth.py
- [ ] T045 ReportAccessService with permission checking in backend/src/services/report_access.py

## Phase 3.6: API Endpoints

### Document Ingestion Service
- [ ] T046 POST /v1/documents/ingest endpoint in backend/src/api/ingestion/ingest.py
- [ ] T047 GET /v1/documents/{id}/status endpoint in backend/src/api/ingestion/status.py
- [ ] T048 POST /v1/documents/batch endpoint in backend/src/api/ingestion/batch.py

### Research Generation Service
- [ ] T049 POST /v1/reports/generate endpoint in backend/src/api/research/generate.py
- [ ] T050 GET /v1/reports/{id}/status endpoint in backend/src/api/research/status.py
- [ ] T051 GET /v1/reports/{id} endpoint in backend/src/api/research/reports.py
- [ ] T052 POST /v1/voice/record endpoint in backend/src/api/research/voice.py
- [ ] T053 GET /v1/voice/{id}/transcript endpoint in backend/src/api/research/transcript.py

### Portal Service
- [ ] T054 POST /api/v1/auth/login endpoint in backend/src/api/portal/auth.py
- [ ] T055 GET /api/v1/reports endpoint with filtering in backend/src/api/portal/reports.py
- [ ] T056 GET /api/v1/reports/{id} endpoint with access control in backend/src/api/portal/report_detail.py

## Phase 3.7: External Integrations

- [ ] T057 [P] Google Cloud Storage integration for document and report storage in backend/src/integrations/gcs_client.py
- [ ] T058 [P] Pub/Sub publisher and subscriber in backend/src/integrations/pubsub_client.py
- [ ] T059 [P] Cloud KMS client for CMEK encryption in backend/src/integrations/kms_client.py
- [ ] T060 MCP server integration for external tools in backend/src/integrations/mcp_client.py

## Phase 3.8: Frontend Portal

- [ ] T061 [P] Authentication components in frontend/src/components/auth/
- [ ] T062 [P] Report list and search components in frontend/src/components/reports/
- [ ] T063 [P] Report detail viewer in frontend/src/components/reports/ReportViewer.tsx
- [ ] T064 [P] User dashboard in frontend/src/pages/Dashboard.tsx

## Phase 3.9: Infrastructure Deployment

- [ ] T065 Document Ingestion Service deployment to Cloud Run
- [ ] T066 Research Generation Service deployment to Cloud Run
- [ ] T067 Portal Service deployment to Cloud Run
- [ ] T068 Configure Cloud Scheduler for automated report generation
- [ ] T069 Set up VPC Service Controls perimeter with restricted Google APIs

## Phase 3.10: Validation & Polish

- [ ] T070 [P] Unit tests for document processing logic in backend/tests/unit/test_document_processing.py
- [ ] T071 [P] Unit tests for voice processing in backend/tests/unit/test_voice_processing.py
- [ ] T072 [P] Unit tests for compliance validation in backend/tests/unit/test_compliance.py
- [ ] T073 [P] Performance tests for report generation (<30 minutes SLA) in backend/tests/performance/test_report_timing.py
- [ ] T074 [P] Security tests for authentication and authorization in backend/tests/security/test_access_control.py
- [ ] T075 Run complete quickstart.md validation scenarios
- [ ] T076 [P] Update API documentation with OpenAPI specs
- [ ] T077 [P] Create operational runbooks for monitoring and troubleshooting
- [ ] T078 Load testing with sample institutional client load
- [ ] T079 Compliance audit preparation with full audit trail validation

## Dependencies

**Critical Path**:
- Setup (T001-T006) → Infrastructure (T007-T011) → Contract Tests (T012-T026) → Models (T027-T036) → Services (T037-T045) → Endpoints (T046-T056) → Integration (T057-T060) → Deployment (T065-T069) → Validation (T070-T079)

**Key Blockers**:
- T011 (database migration) blocks all model tasks
- All contract tests (T012-T026) must FAIL before any implementation
- Models (T027-T036) block services (T037-T045)
- Services block API endpoints (T046-T056)
- Core backend blocks frontend (T061-T064)

**Parallel Opportunities**:
- Contract tests T012-T026 can run simultaneously
- Model creation T027-T036 can run in parallel after DB setup
- Integration components T057-T060 can run in parallel
- Frontend tasks T061-T064 can run in parallel
- Unit/performance tests T070-T074 can run in parallel

## Parallel Execution Examples

### Phase 3.3 - All Contract Tests Together
```bash
# Launch contract tests simultaneously (different files):
Task: "Contract test POST /v1/documents/ingest in backend/tests/contract/test_ingestion_post.py"
Task: "Contract test GET /v1/documents/{id}/status in backend/tests/contract/test_ingestion_status.py"
Task: "Contract test POST /v1/documents/batch in backend/tests/contract/test_ingestion_batch.py"
Task: "Contract test POST /v1/reports/generate in backend/tests/contract/test_research_generate.py"
Task: "Contract test GET /v1/reports/{id}/status in backend/tests/contract/test_research_status.py"
# ... all 15 contract tests
```

### Phase 3.4 - All Models Together
```bash
# Launch model creation simultaneously (different files):
Task: "FinancialDocument model in backend/src/models/financial_document.py"
Task: "DocumentChunk model with vector embedding support in backend/src/models/document_chunk.py"
Task: "Company model in backend/src/models/company.py"
Task: "ResearchReport model in backend/src/models/research_report.py"
# ... all 10 model files
```

### Phase 3.7 - Integration Components
```bash
# Launch integration components (independent systems):
Task: "Google Cloud Storage integration in backend/src/integrations/gcs_client.py"
Task: "Pub/Sub publisher and subscriber in backend/src/integrations/pubsub_client.py"
Task: "Cloud KMS client for CMEK encryption in backend/src/integrations/kms_client.py"
```

## Validation Checklist
*GATE: Must be complete before deployment*

- [x] All 3 contract files have corresponding tests (9 total)
- [x] All 10 entities have model creation tasks
- [x] All 15 contract tests come before any implementation
- [x] All [P] tasks operate on different files
- [x] Each task specifies exact file path
- [x] TDD flow: failing tests → implementation → validation
- [x] Google Cloud services properly integrated with CMEK/VPC-SC
- [x] EU data sovereignty maintained throughout pipeline
- [x] Comprehensive audit trail for regulatory compliance

**Task Generation Complete** - 79 tasks across 5 development phases
**Ready for Implementation** - All tasks are specific and executable