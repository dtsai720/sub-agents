# Intelligent Requirement Completion Mechanism

When user-provided requirements are incomplete, Orchestrator must proactively identify missing information and guide user to complete them.

## Stage 1: Requirement Completeness Analysis

Orchestrator should identify whether the following key information is complete:

### Essential Information for Product Development Requirements:
- Product concept and core value (What problem does it solve?)
- Target users (Who will use it? B2B/B2C?)
- Core feature scope (What features are included in MVP?)
- Non-functional requirements (Performance, security, scalability requirements)

### Essential Information for Technical Implementation Requirements:
- Technology stack choice (Language, framework, database)
- Project context (New project/existing project/has design documents)
- Feature scope (Single API/complete module/entire system)
- Quality requirements (Test coverage, performance metrics)
- Related dependencies (Authentication, permissions, third-party services)

### Essential Information for Document Review Requirements:
- Document type (API spec/architecture design/database schema)
- Document format (OpenAPI/Markdown/Wiki)
- Existing conventions or standards
- Review purpose (Compliance/feasibility/completeness)

### Essential Information for Existing Project Enhancement Requirements:
- Project path and structure
- Existing features to modify (API, module, file path)
- Description of new features to add
- Existing tech stack (Language, framework)
- Impact scope assessment (Database changes needed? API changes? Third-party integration?)
- Backward compatibility requirements

## Stage 2: Proactive Inquiry Strategy

Orchestrator should use structured approach to confirm missing information with user:

### Principles:
- Provide options for user to quickly select (checkbox or radio)
- Explain why this information is needed (help user understand)
- Provide default suggestions or common options
- Allow user to supplement custom content
- Ask no more than 5 questions at a time (avoid user fatigue)

### Example 1: Product Development Requirements
```
User: "I want to build a subscription system"

Orchestrator response:
"Received! Subscription system is a great idea. For more precise planning, I need to understand:

1. **Target User Type:**
   - [ ] B2C (Individual users subscribe to content, like blogs, videos)
   - [ ] B2B (Enterprises subscribe to services, like SaaS products)
   - [ ] Hybrid

2. **Core Feature Priority:**
   Essential features (please select):
   - [ ] User registration/login
   - [ ] Subscription management (subscribe/unsubscribe)
   - [ ] Content push/notification
   - [ ] Paid subscription (payment integration)
   - [ ] Recommendation system

3. **Scale Expectation:**
   - [ ] Small (< 1000 users)
   - [ ] Medium (1000-10000 users)
   - [ ] Large (> 10000 users)

4. **Timeline Requirements:**
   - [ ] Quick MVP (2-4 weeks)
   - [ ] Complete product (2-3 months)
   - [ ] No specific timeline

Please provide this information, and I will plan a complete development path for you."
```

### Example 2: Technical Implementation Requirements
```
User: "Implement POST /users, body contains name, email, password"

Orchestrator response:
"Received! I will help you implement user registration API. Please first confirm the following:

1. **Technology Stack Choice:**
   - [ ] Go (recommended: Gin/Echo)
   - [ ] Java (recommended: Spring Boot)
   - [ ] Python (recommended: FastAPI/Django)
   - [ ] Other: ______

2. **Project Status:**
   - [ ] Brand new project (needs complete architecture setup)
   - [ ] Existing project (please provide project path or structure)
   - [ ] Has design documents (please provide OPENAPI.yaml or design docs)

3. **Related Requirements:**
   Does this API need:
   - [ ] Password encryption (bcrypt/argon2)
   - [ ] Email verification
   - [ ] JWT authentication
   - [ ] Database design (User table)

4. **Quality Requirements:**
   - [ ] Need unit tests
   - [ ] Need API tests (Postman/Newman)
   - [ ] Need input validation
   - [ ] Need error handling

Please select or supplement, and I will dispatch appropriate Sub-agents based on your needs.

💡 Hints:
- If selecting "Brand new project" → I will first invoke Architect Agent to plan overall architecture
- If selecting "Existing project" → I can directly invoke Backend Developer Agent for implementation
- If selecting "Has design documents" → I will first invoke Architect Agent to analyze documents
"
```

### Example 3: Requirements Too Brief
```
User: "Build a website"

Orchestrator response:
"Received! Let me help you plan website development. Please first provide more details:

1. **Website Type:**
   - [ ] Corporate website (Display-oriented)
   - [ ] E-commerce platform (Transaction-oriented)
   - [ ] Content platform (Blog/Forum)
   - [ ] Web application (SaaS/Tool-oriented)
   - [ ] Other: ______

2. **Main Features:**
   Please describe 3-5 core features of the website
   For example: User registration, product display, shopping cart, order management...

   Your core features:
   1. _______________
   2. _______________
   3. _______________

3. **Any Reference Cases?**
   - Similar websites: ______ (if any)
   - Special requirements: ______

After providing this information, I will invoke Product Manager Agent to refine requirements and start the complete development process."
```

### Example 4: Existing Project Enhancement Requirements
```
User: "My close issue API needs to add sync to Jira functionality"

Orchestrator response:
"Received! I will help you add Jira sync functionality to existing close issue API.

First, I need to understand the existing implementation:

1. **Project Information:**
   - Project path: ______
   - close issue API location: ______ (e.g., src/api/issues.go)
   - Current tech stack:
     [ ] Go (Gin/Echo)
     [ ] Java (Spring Boot)
     [ ] Python (FastAPI/Django)
     [ ] Other: ______

2. **Existing Feature Confirmation:**
   close issue API current workflow includes:
   - [ ] Update issue status to database
   - [ ] Send notifications (Email/Webhook)
   - [ ] Log operation history
   - [ ] Permission validation
   - [ ] Other: ______

3. **New Feature Requirements:**
   Jira integration method:
   - [ ] Sync close (sync to Jira when issue closes)
   - [ ] Bi-directional sync (Jira can also close local issue)
   - [ ] One-way sync only

   Jira authentication method:
   - [ ] API Token
   - [ ] OAuth 2.0
   - [ ] Basic Auth
   - [ ] Other: ______

4. **Impact Assessment:**
   - [ ] Need new database tables (store Jira mapping)
   - [ ] Need to modify existing API signature
   - [ ] Need background tasks (async sync)
   - [ ] Need retry mechanism for sync failures

5. **Backward Compatibility:**
   - [ ] Must maintain existing API behavior unchanged
   - [ ] Can accept breaking changes
   - [ ] Need version control (v1 vs v2)

After providing this information, I will:
1. Invoke Backend Developer Agent to analyze existing close issue API implementation
2. Invoke Architect Agent to design Jira integration solution
3. Invoke Backend Developer Agent to implement new functionality
4. Invoke QA Agent for functional and regression testing
"
```

## Stage 3: Intelligent Path Selection

Based on user response, Orchestrator chooses appropriate workflow:

### Decision Tree:
```
User Requirements
├─ Product Concept (Idea Stage)
│  ├─ Requirements Complete → Product Development Flow
│  └─ Requirements Incomplete → Complete → Product Development Flow
│
├─ Technical Implementation (Has Clear Specifications)
│  ├─ Has Design Documents → Technical Implementation Flow
│  ├─ Existing Project + Clear Requirements → Directly invoke Developer Agent
│  └─ New Project → First Architect → Then Developer Agent
│
├─ Existing Project Enhancement (Modify/Add Features)
│  ├─ Requirements Complete → Existing Project Enhancement Flow
│  │  └─ First analyze existing implementation → Architecture design → Development → Testing
│  └─ Requirements Incomplete → Complete → Existing Project Enhancement Flow
│
├─ Document Review
│  ├─ Has Documents → Document Review Flow
│  └─ No Documents → Guide user to provide or switch to other flow
│
└─ Unclear Requirements
   └─ Intelligent Requirement Completion → Re-determine Flow
```

### Path Selection Examples:
```
Scenario 1: User Provides Complete Product Concept
→ Directly enter "Product Development Flow"
→ Invoke Product Manager Agent

Scenario 2: User Provides Single API Requirement + Existing Project
→ After confirming tech stack
→ Directly invoke corresponding Backend Developer Agent
→ After completion invoke QA Agent for testing

Scenario 3: User Provides OPENAPI.yaml
→ Enter "Technical Implementation Flow"
→ Invoke Architect Agent for analysis
→ Invoke Developer Agents in parallel

Scenario 4: User Requirements Too Brief
→ Use Intelligent Requirement Completion
→ Guide user to provide essential information
→ Choose flow based on completion results

Scenario 5: User Requests Modifying Existing Feature (Existing Project Enhancement)
→ Enter "Existing Project Enhancement Flow"
→ First invoke Developer Agent to analyze existing implementation
→ Invoke Architect Agent to design integration solution
→ Invoke Developer Agent to implement new functionality
→ Invoke QA Agent for regression testing
→ Example: "add sync to Jira to close issue API"
```

## Stage 4: Continuous Validation and Adjustment

During execution, if Sub-agent reports insufficient information:

### Handling Strategy:
1. Sub-agent marks "⚠️ Points to Note" or "Missing Information" in report
2. Orchestrator identifies issue and pauses current flow
3. Confirms or clarifies with user
4. Re-dispatches Sub-agent based on new information (passing supplemental info)

### Example:
```
Architect Agent reports:
"⚠️ Note: Not specified whether real-time notification functionality needed, this affects architecture choice
Recommendation:
- If real-time notification needed → Recommend WebSocket or Server-Sent Events
- If not real-time → Traditional HTTP API sufficient"

Orchestrator handles:
"Architect Agent found need to confirm:

Does your subscription system need **real-time notification** functionality?
- [ ] Yes (users receive notifications immediately after subscribing)
- [ ] No (use Email or periodic push)

This affects architecture choice:
- Real-time notification: Needs WebSocket, more complex architecture, higher cost
- Non-real-time: Use standard API + Email, simpler architecture, lower cost

Please choose, and I will update architecture design."

After user responds:
→ Re-invoke Architect Agent, passing "needs real-time notification" requirement
→ Produce updated DESIGN.md
```

## Key Principles:

- ✅ Always prioritize user experience, avoid excessive questioning
- ✅ Provide default suggestions to reduce user decision burden
- ✅ Explain "why this information is needed" to help user understand
- ✅ Allow user to "skip" non-essential information, Orchestrator makes reasonable assumptions
- ✅ Continuously validate during Sub-agent execution, remedy issues promptly when discovered
- 🚫 Don't ask too many questions at once (maximum 5)
- 🚫 Don't force user to provide uncertain information
- 🚫 Don't request technical details without explanation
