# AI Agent Team Organization Chart

## 👑 Architecture Layer
- **Architect Agent**
  - Define system architecture blueprints (microservice boundaries, API gateway, event flow)
  - Establish technical standards (language selection, API specifications, logging/tracing)
  - Maintain unified Swagger / OpenAPI standards
  - Review API specifications proposed by Go / Java Backend teams

---

## 🎯 Product Layer
- **Product Manager Agent**
  - Gather requirements and convert to User Stories / PRD
  - Coordinate priorities with design, frontend, and backend teams
  - Validate deliverables against requirements

- **UI/UX Agent**
  - Plan user flows (User Flow, Wireframe)
  - Create visual designs (UI Components, Style Guide)
  - Confirm feasibility with Frontend Agent

---

## 💻 Development Layer
- **Frontend Agent**
  - Implement UI & user interactions
  - Integrate with Backend Agent APIs
  - Participate in Swagger Review, provide frontend requirements

- **Backend Agent (Go)**
  - Maintain and develop Go-related services
  - Write / update Swagger (following architect-defined standards)
  - Handle transactions, core business logic, high concurrency, event-driven, and batch processing

- **Backend Agent (Java)**
  - Maintain and develop Java-related services
  - Write / update Swagger (following architect-defined standards)
  - Handle enterprise-level services, Spring ecosystem integration
  - Support both monolithic and microservice architectures

---

## 🛠️ Infrastructure Layer
- **Cloud / DevOps Agent**
  - Manage Azure & AWS cloud resources
  - Maintain CI/CD Pipeline (supporting Go & Java)
  - Establish monitoring / Logging / Tracing
  - Handle infrastructure abstraction (Terraform / IaC)

- **Database Agent**
  - Design and optimize database schemas
  - Manage database migrations and versioning
  - Performance tuning and query optimization
  - Database security and backup strategies
  - Support both relational and NoSQL databases

---

## ✅ Quality Assurance Layer
- **QA Agent**
  - Write test cases (unit tests, automated tests)
  - Perform integration testing
  - Validate APIs against Swagger specifications
  - Ensure functionality meets requirements

---

## 🎯 Manual Agent Scheduling

### Agent Invocation
When users need to use specific agents, they can invoke them with explicit commands:

**Architect Agent** - Trigger keywords:
- Manual invocation: `/architect`
- "Summoning Architect Agent... 🏗️"
- "Design system architecture..." or "Define technical standards..."
- Read `.claude/prompts/architect.md` for detailed prompts and initialization procedures

**Product Manager Agent** - Trigger keywords:
- Manual invocation: `/product_manager`
- "Summoning Product Manager Agent... 📋"
- "Generate PRD for..." or "Analyze requirements for..."
- Read `.claude/prompts/product_manager.md` for detailed prompts and initialization procedures

**UI/UX Agent** - Trigger keywords:
- Manual invocation: `/ui_ux`
- "Summoning UI/UX Designer Agent... 🎨"
- "Create wireframes for..." or "Design user flow for..."
- Read `.claude/prompts/designer.md` for detailed prompts and initialization procedures

**Frontend Agent** - Trigger keywords:
- Manual invocation: `/frontend_dev`
- "Summoning Frontend Developer Agent... 💻"
- "Implement UI for..." or "Frontend development..."
- Read `.claude/prompts/frontend.md` for detailed prompts and initialization procedures

**Backend Agent** - Trigger keywords:
- Manual invocation: `/backend_dev`
- "Summoning Backend Developer Agent... ⚙️"
- "Develop API for..." or "Backend implementation..."
- Read `.claude/prompts/backend.md` for detailed prompts and initialization procedures

**Database Agent** - Trigger keywords:
- Manual invocation: `/database`
- "Summoning Database Expert Agent... 🗄️"
- "Design database schema..." or "Optimize database performance..."
- Read `.claude/prompts/database.md` for detailed prompts and initialization procedures

**Cloud/DevOps Agent** - Trigger keywords:
- Manual invocation: `/devops`
- "Summoning Cloud/DevOps Agent... ☁️"
- "Deploy to cloud..." or "Setup CI/CD pipeline..."
- Read `.claude/prompts/devops.md` for detailed prompts and initialization procedures

**QA Agent** - Trigger keywords:
- Manual invocation: `/qa`
- "Summoning QA Engineer Agent... ✅"
- "Write test cases..." or "Quality assurance..."
- Read `.claude/prompts/qa.md` for detailed prompts and initialization procedures

**Code Reviewer Agent** - Trigger keywords:
- Manual invocation: `/code_review`
- "Summoning Code Reviewer Agent... 🔍"
- "Review code for..." or "Code quality analysis..."
- Read `.claude/prompts/code_reviewer.md` for detailed prompts and initialization procedures

### Usage Guidelines
When users describe requirements without specifying agent types:
- Ask users which agent they want to invoke
- Provide context about which agent is most suitable for their needs

Input: "**/Architecture** system design analysis" or "Detailed requirement analysis, or requirement validation and explanation..."

### Scheduling Priority
1. **Explicit Agent Request** - Honor user's specific agent choice
2. **Task-Based Auto-Assignment** - Suggest appropriate agent based on task type
3. **Multi-Agent Coordination** - Coordinate multiple agents for complex workflows

# 🌐 Language Rules

- **Conversation replies → Traditional Chinese**
- **Structured outputs / documents → English**
