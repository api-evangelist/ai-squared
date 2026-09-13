---
name: Multiwoven
description: Use when building data pipelines, connecting AI models to business applications, creating workflows, embedding insights into CRMs/ERPs, configuring data syncs, or deploying AI-powered features at scale. Reach for this skill when agents need to integrate disparate data sources, operationalize AI models, set up data movement workflows, or build conversational AI assistants.
metadata:
    mintlify-proj: multiwoven
    version: "1.0"
---

# AI Squared Skill Reference

## Product Summary

AI Squared is a unified platform for integrating AI and data across enterprise systems. It connects disparate data sources (warehouses, databases, CRMs, APIs), operationalizes AI/ML models (OpenAI, Anthropic, Vertex, SageMaker), and embeds insights directly into business applications where decisions happen. The platform handles data movement via connectors, workflow orchestration through visual builders, and deployment across cloud, local, or on-premise environments.

**Key files and concepts:**
- **Sources**: Data warehouses, databases, APIs, file storage (Snowflake, BigQuery, PostgreSQL, S3, etc.)
- **Destinations**: CRMs, marketing platforms, databases, collaboration tools (Salesforce, HubSpot, Slack, etc.)
- **Models**: SQL-based or visual data transformations; AI/ML model endpoints
- **Syncs**: Data movement operations (full refresh or incremental)
- **Workflows**: Visual builder for chaining inputs → prompts → LLM → data retrieval → outputs
- **Data Apps**: Visualizations embedded in business applications
- **API Base URL**: `https://api.squared.ai/api/v1/`
- **Authentication**: JWT Bearer token in Authorization header

## When to Use

Reach for this skill when:
- Building end-to-end data pipelines from source to destination
- Connecting AI models (OpenAI, Anthropic, Vertex, SageMaker) to business workflows
- Creating automated data syncs between systems (e.g., warehouse → CRM)
- Building conversational AI workflows with knowledge bases and agents
- Embedding AI predictions into Salesforce, ServiceNow, or custom web apps
- Setting up feedback loops to improve model performance
- Deploying AI solutions in regulated environments (on-premise, air-gapped)
- Configuring data transformations and modeling before sync
- Troubleshooting sync failures or connector authentication issues
- Scaling AI from POC to production across enterprise systems

## Quick Reference

### Core Objects and API Endpoints

| Object | Purpose | Key Endpoints |
|--------|---------|---------------|
| **Connector** | Connection to a data source or destination | `POST /connectors`, `GET /connectors`, `PUT /connectors/{id}`, `DELETE /connectors/{id}` |
| **Model** | Data transformation or AI/ML model definition | `POST /models`, `GET /models`, `PUT /models/{id}`, `DELETE /models/{id}` |
| **Sync** | Data movement operation between source and destination | `POST /syncs`, `GET /syncs`, `PUT /syncs/{id}`, `DELETE /syncs/{id}`, `POST /syncs/{id}/test` |
| **Workflow** | Visual orchestration of inputs, prompts, LLMs, and data retrieval | Build via UI or API; publish to expose as Chat Assistant |
| **Data App** | Visualization of model output embedded in business apps | Create from connected model with input/output schemas |

### Sync Modes

| Mode | Use Case | Behavior |
|------|----------|----------|
| **Full Refresh** | Complete data replacement | Replaces all destination data with fresh source data each run |
| **Incremental** | Only new/changed data | Syncs only rows changed since last run using primary key deduplication |
| **Cursor Incremental** | Optimized incremental | Uses cursor field (timestamp, ID) to minimize source queries |

### AI Model Sources

| Provider | Endpoint | Auth |
|----------|----------|------|
| OpenAI | `https://api.openai.com/v1/chat/completions` | API Key |
| Anthropic | Native integration | API Key |
| Google Vertex AI | Project-specific endpoint | JSON service account key |
| AWS SageMaker | Endpoint URL | IAM credentials |
| AWS Bedrock | Region-specific | IAM credentials |
| Custom HTTP | Your endpoint URL | API Key, OAuth 2.0, or none |

### Workflow Components

| Component | Function |
|-----------|----------|
| **Chat Input** | Accept user questions to start workflow |
| **File Input** | Attach documents (CSV, PDF, DOCX) as context |
| **Prompt** | Template with placeholders for LLM instructions |
| **LLM** | Execute reasoning with OpenAI, Anthropic, etc. |
| **Database** | Query structured data from connected sources |
| **Vector Search** | Semantic retrieval from Pinecone, Qdrant, Weaviate |
| **Agent** | Multi-step reasoning with tool access |
| **Guardrails** | PII masking, content policy, compliance rules |
| **Human Approval** | Pause for manual review before proceeding |
| **Python Executor** | Run custom Python code (numpy, pandas, requests, etc.) |

### Deployment Options

| Option | Use Case | Setup |
|--------|----------|-------|
| **Cloud (SaaS)** | Fast setup, managed by AI Squared | Sign up at squared.ai |
| **Local Docker** | Development, testing, tight control | `docker-compose up` with `.env` |
| **Self-Hosted** | Regulated environments, air-gapped | Docker Compose, Kubernetes, Helm, cloud VMs (EC2, ECS, EKS, GKE, AKS) |

### Environment Variables (Self-Hosted)

| Variable | Purpose | Example |
|----------|---------|---------|
| `RAILS_ENV` | Rails environment | `production` |
| `UI_HOST` | Frontend hostname | `localhost:8000` |
| `API_HOST` | Backend API hostname | `localhost:3000` |
| `DB_HOST`, `DB_USERNAME`, `DB_PASSWORD` | PostgreSQL connection | `postgres`, `user`, `pass` |
| `JWT_SECRET` | Token signing key | Generate with `SecureRandom.hex(32)` |
| `ALLOWED_HOST` | CORS/DNS rebinding protection | `yourdomain.com` |

## Decision Guidance

### When to Use Full Refresh vs. Incremental Sync

| Scenario | Choose |
|----------|--------|
| Small dataset, infrequent changes, need latest snapshot | **Full Refresh** |
| Large dataset, frequent updates, minimize data transfer | **Incremental** |
| Have timestamp or ID cursor field, need optimized queries | **Cursor Incremental** |
| First sync ever, no prior state | **Full Refresh** (incremental will behave like full refresh on first run) |

### When to Use Workflow vs. Data App

| Goal | Use |
|------|-----|
| Build conversational AI assistant with multi-step reasoning | **Workflow** → publish as Chat Assistant |
| Embed single model prediction in CRM/dashboard | **Data App** (simpler, no-code) |
| Complex logic: retrieve data, transform, route conditionally | **Workflow** with Database/Vector/Agent components |
| Simple visualization of model output | **Data App** with Table/Chart display |

### When to Use Vector Search vs. Database Query

| Scenario | Choose |
|----------|--------|
| Semantic search: "Find similar customer issues" | **Vector Search** (Pinecone, Qdrant, Weaviate) |
| Structured SQL query: "Get sales > $10k" | **Database** component |
| Hybrid: semantic + structured filtering | **Vector Search** → **Database** in sequence |

### When to Deploy Cloud vs. Self-Hosted

| Requirement | Choose |
|-------------|--------|
| Fast time-to-value, managed infrastructure | **Cloud (SaaS)** |
| Tight data residency, HIPAA/FedRAMP compliance | **Self-Hosted** |
| Development/testing, local iteration | **Local Docker** |
| Air-gapped environment, no external dependencies | **Self-Hosted** (on-premise) |

## Workflow

### Typical Data Pipeline Setup

1. **Create Source Connector**
   - Navigate to Sources → Add Source
   - Select connector type (Snowflake, PostgreSQL, S3, etc.)
   - Enter credentials (host, username, password, API key, etc.)
   - Test connection to verify authentication
   - Save connector

2. **Create Destination Connector**
   - Navigate to Destinations → Add Destination
   - Select destination type (Salesforce, HubSpot, Slack, etc.)
   - Enter credentials and configuration
   - Test connection
   - Save connector

3. **Define Model (Data Transformation)**
   - Go to Models → Create Model
   - Choose method: SQL Editor, Visual Table Selector, or dbt
   - Define primary key (required for incremental syncs)
   - Save model

4. **Create Sync**
   - Go to Syncs → Create Sync
   - Select source connector, destination connector, model
   - Choose sync mode (full refresh or incremental)
   - Set schedule (manual, interval, or cron)
   - If incremental: specify cursor field (optional, for optimization)
   - Test sync to validate configuration
   - Enable and save

5. **Monitor Sync**
   - View sync runs in Syncs dashboard
   - Check status: Healthy, Pending, Failed, Disabled
   - Review error logs if sync fails
   - Adjust configuration or retry as needed

### Typical AI Model Activation Workflow

1. **Connect AI Model Source**
   - Go to AI Activation → Add AI Source
   - Select provider (OpenAI, Anthropic, Vertex, SageMaker, etc.)
   - Enter API key, endpoint URL, or service account credentials
   - Test connection
   - Save source

2. **Define Input Schema**
   - Go to AI Modeling → Connect Source
   - Select your AI model source
   - Define input fields: name, type (String/Integer/Float/Boolean), dynamic/static
   - Add preprocessing logic if needed (optional)
   - Save schema

3. **Define Output Schema**
   - Specify output fields expected from model response
   - Map field names and types
   - Save schema

4. **Create Data App**
   - Go to Data Apps → Create New Data App
   - Select the configured model
   - Choose display type (Table, Bar Chart, Pie Chart, Text Card)
   - Customize appearance (colors, labels, dark/light mode)
   - Enable feedback collection (thumbs up/down, ratings, text)
   - Save and preview

5. **Embed in Business App**
   - Copy iframe snippet from Data App
   - Paste into target application (Salesforce, custom dashboard, etc.)
   - Verify embedding works and data displays correctly
   - Monitor feedback and usage in Reports

### Typical Workflow Builder Setup

1. **Create Workflow**
   - Go to Workflows → Create Workflow
   - Start with template or blank canvas
   - Add Chat Input component (entry point)

2. **Add Data Retrieval**
   - Add Database or Vector Search component
   - Connect to data source
   - Define query or search parameters

3. **Add LLM Reasoning**
   - Add Prompt component with instructions
   - Add LLM component (OpenAI, Anthropic, etc.)
   - Connect prompt output to LLM input

4. **Add Guardrails (Optional)**
   - Add guardrail rules for PII masking, content policy
   - Configure compliance checks

5. **Add Output**
   - Add Chat Output component
   - Connect LLM output to display

6. **Test in Playground**
   - Submit test inputs
   - Inspect intermediate and final outputs
   - Verify behavior before publishing

7. **Publish and Deploy**
   - Publish workflow
   - Expose as Chat Assistant
   - Deploy as hosted chatbot, embed in iframe, or integrate via API

## Common Gotchas

- **Missing Primary Key in Model**: Incremental syncs require a unique primary key. If not defined, sync will fail. Always specify a primary key when creating models.

- **Connector Test Passes but Sync Fails**: Test connection only validates authentication, not data access. Verify the source has data and destination has write permissions. Check sync error logs for details.

- **Incremental Sync Processes All Data on First Run**: This is expected. Incremental syncs behave like full refresh on the first run since there's no prior state to compare against.

- **Cursor Field Not Optimizing Queries**: Cursor Incremental only works if the source supports cursor fields (timestamp, ID). Not all sources support this. Check source documentation for cursor field support.

- **Data App Not Displaying in Embedded iframe**: Ensure the host application's domain is added to Allowed Origins in Settings → Embed Origins. CORS will block requests from unlisted domains.

- **Workflow Execution Timeout**: Long-running queries or LLM calls may timeout. Default HTTP timeout is 30 seconds. Increase via configuration if needed.

- **JWT Token Expired**: API calls fail with 401 Unauthorized. Regenerate JWT token from dashboard and update Authorization header.

- **Sync Stuck in Pending State**: Check if Temporal workflow orchestration is running. Restart services if needed. Review logs for heartbeat timeouts.

- **Model Output Not Matching Schema**: If model returns unexpected field names or types, update output schema to match actual response. Test with sample payload first.

- **File Input Component Truncates Large Files**: Files are truncated to fit token limits before passing to LLM. For large documents, split into smaller chunks or use Vector Search for semantic retrieval.

- **Incremental Sync Duplicates Data**: If primary key is not unique or cursor field is not monotonically increasing, duplicates may appear. Verify data quality in source.

- **Connector Credentials Exposed in Logs**: Never log or share JWT tokens, API keys, or passwords. Use environment variables and secure vaults. Credentials are masked in API responses.

## Verification Checklist

Before submitting work with AI Squared:

- [ ] **Connectors**: Test connection successful for both source and destination
- [ ] **Model**: Primary key defined (required for incremental syncs)
- [ ] **Sync**: Test sync run completed without errors
- [ ] **Sync Schedule**: Cron expression or interval correctly configured
- [ ] **Data Quality**: Sample rows from destination match source data
- [ ] **AI Model**: Input and output schemas match actual model request/response format
- [ ] **Data App**: Preview displays correctly with sample data
- [ ] **Data App Embedding**: iframe loads in target application without CORS errors
- [ ] **Workflow**: Playground test passes with expected output
- [ ] **Workflow Publishing**: Chat Assistant accessible and responds to test queries
- [ ] **Feedback Collection**: Feedback mechanism enabled and capturing responses
- [ ] **Error Handling**: Sync failures trigger alerts (email/Slack)
- [ ] **Audit Logs**: Verify connector creation, data access, and sync runs are logged
- [ ] **Deployment**: Environment variables set correctly for self-hosted deployments
- [ ] **Security**: JWT tokens, API keys, and credentials not exposed in code or logs

## Resources

**Comprehensive Navigation**: Visit https://docs.squared.ai/llms.txt for a complete page-by-page listing of all documentation.

**Critical Documentation**:
- [Getting Started Introduction](https://docs.squared.ai/getting-started/introduction) — Core concepts, architecture, and platform overview
- [Core Concepts Guide](https://docs.squared.ai/guides/core-concepts) — Sources, Destinations, Models, Syncs explained
- [API Reference](https://docs.squared.ai/api-reference/introduction) — REST endpoints, authentication, pagination, rate limits
- [Workflows Overview](https://docs.squared.ai/workflows/overview) — Workflow builder components, templates, and orchestration
- [AI Activation Introduction](https://docs.squared.ai/activation/ai-modelling/introduction) — Connecting AI models and creating Data Apps
- [Deployment & Security Overview](https://docs.squared.ai/deployment-and-security/overview) — Deployment options, RBAC, compliance

---

> For additional documentation and navigation, see: https://docs.squared.ai/llms.txt