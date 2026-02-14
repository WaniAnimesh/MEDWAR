# Design Document: MEDWAR Clinical Threat Intelligence Engine

## Overview

MEDWAR is a production-grade Clinical Threat Intelligence Engine that detects, quantifies, and mitigates semantic fractures in hospital care delivery through multi-agent adversarial analysis. The system operates on the principle that hospital failures often stem from predictable communication breakdowns between clinical roles operating under asymmetric information and competing objectives.

### Core Architecture Principles

1. **Multi-Agent Adversarial Analysis**: Six specialized agents (Nurse, Doctor, PCP, Patient, Adversary, Compliance) conduct structured debates to identify semantic fractures from different role perspectives
2. **Evidence-Grounded Detection**: Three-layer verification ensures all fracture claims are backed by verifiable source data, preventing hallucinations
3. **Calibrated Risk Scoring**: Four-layer entropy scoring combines heuristics, LLM consistency, data quality, and statistical calibration
4. **Real-Time and Retrospective Modes**: Supports triggered analysis (Snapshot OODA), continuous monitoring (Streaming OODA), and historical audit
5. **Patient-Centric Output**: Generates multilingual, grade-6 readable discharge instructions with teach-back validation
6. **Continuous Learning**: Feedback loop updates calibration, thresholds, and hospital-specific patterns

### System Boundaries

**In Scope:**
- Semantic fracture detection and risk quantification
- Role-specific countermeasure generation
- Patient comprehension aids and discharge instructions
- Real-time monitoring and retrospective analysis
- EHR integration via HL7/FHIR standards
- Feedback collection and learning loop

**Out of Scope:**
- Autonomous diagnosis or treatment decisions
- Autonomous prescription or order generation
- Direct modification of EHR records
- Real-time vital sign monitoring hardware
- Medical device integration beyond data ingestion

### Technology Stack

- **Frontend**: React PWA (offline-first), TypeScript, TailwindCSS
- **Backend**: AWS Lambda (Node.js/Python), Step Functions, EventBridge, SNS/SQS
- **Data Layer**: DynamoDB (patient state), S3 (raw data), ElastiCache (caching)
- **AI Layer**: AWS Bedrock (Claude 3 Sonnet/Opus/Haiku), structured JSON outputs
- **Integration**: HL7 v2.x, FHIR R4, REST APIs
- **Security**: AWS KMS, Cognito, IAM, VPC, Mumbai region data residency
- **Monitoring**: CloudWatch, X-Ray, custom dashboards


## Architecture

### High-Level System Architecture

```mermaid
graph TB
    subgraph "Hospital Infrastructure"
        EHR[EHR System<br/>Epic/Cerner/HMIS]
        Clinicians[Clinical Users<br/>Nurses/Doctors]
        Patients[Patients/Caregivers]
    end
    
    subgraph "MEDWAR Ingestion Layer"
        HL7[HL7 Connector]
        FHIR[FHIR Connector]
        Manual[Manual Entry API]
        OCR[OCR Processor]
        ASR[ASR Processor]
    end
    
    subgraph "MEDWAR Core Pipeline"
        Normalize[Normalization Engine]
        Timeline[Patient Timeline Store]
        OODA[OODA Orchestrator]
        Agents[Multi-Agent System]
        Entropy[Entropy Score Engine]
        Evidence[Evidence Verifier]
        Counter[Countermeasure Generator]
    end
    
    subgraph "MEDWAR Output Layer"
        Dashboard[Battlefield Dashboard]
        NurseUI[Nurse Tactical View]
        DoctorUI[Doctor Review View]
        PatientUI[Patient Printout Generator]
        API[REST API]
    end
    
    subgraph "MEDWAR Learning Layer"
        Feedback[Feedback Collector]
        Calibration[Calibration Engine]
        Analytics[Analytics Engine]
    end
    
    EHR -->|HL7 ADT/ORM/ORU| HL7
    EHR -->|FHIR Resources| FHIR
    Clinicians -->|Manual Data| Manual
    Clinicians -->|Scanned Docs| OCR
    Clinicians -->|Voice Notes| ASR
    
    HL7 --> Normalize
    FHIR --> Normalize
    Manual --> Normalize
    OCR --> Normalize
    ASR --> Normalize
    
    Normalize --> Timeline
    Timeline --> OODA
    OODA --> Agents
    Agents --> Entropy
    Entropy --> Evidence
    Evidence --> Counter
    
    Counter --> Dashboard
    Counter --> NurseUI
    Counter --> DoctorUI
    Counter --> PatientUI
    Counter --> API
    
    Dashboard --> Feedback
    NurseUI --> Feedback
    DoctorUI --> Feedback
    Feedback --> Calibration
    Calibration --> Agents
    Calibration --> Entropy
    Feedback --> Analytics
```

### Deployment Architecture

```mermaid
graph TB
    subgraph "AWS Mumbai Region"
        subgraph "VPC - Public Subnet"
            ALB[Application Load Balancer]
            NAT[NAT Gateway]
        end
        
        subgraph "VPC - Private Subnet"
            API[API Gateway]
            Lambda[Lambda Functions]
            StepFn[Step Functions]
            EventBridge[EventBridge]
        end
        
        subgraph "Data Layer"
            DDB[DynamoDB]
            S3[S3 Buckets]
            Cache[ElastiCache]
        end
        
        subgraph "AI Layer"
            Bedrock[AWS Bedrock<br/>Claude Models]
        end
        
        subgraph "Security"
            Cognito[Cognito User Pools]
            KMS[KMS Keys]
            WAF[WAF]
        end
        
        subgraph "Monitoring"
            CW[CloudWatch]
            XRay[X-Ray]
        end
    end
    
    subgraph "Hospital Network"
        Users[Clinical Users]
        EHR[EHR System]
    end
    
    Users -->|HTTPS| WAF
    WAF --> ALB
    ALB --> API
    EHR -->|HL7/FHIR| API
    
    API --> Lambda
    Lambda --> StepFn
    StepFn --> EventBridge
    Lambda --> Bedrock
    Lambda --> DDB
    Lambda --> S3
    Lambda --> Cache
    
    Lambda --> Cognito
    DDB -.->|Encrypted| KMS
    S3 -.->|Encrypted| KMS
    
    Lambda --> CW
    Lambda --> XRay
```


### OODA Loop Architecture

```mermaid
stateDiagram-v2
    [*] --> Trigger
    
    Trigger --> Observe: Clinical Moment or EHR Event
    
    Observe --> Orient: Update Patient Timeline
    
    Orient --> Decide: Build Agent World Models
    
    Decide --> Round1: Start Multi-Agent Debate
    
    Round1 --> Round2: Parallel Analysis Complete
    Round2 --> Round3: Sequential Rebuttal Complete
    Round3 --> Convergence: Synthesis Complete
    
    Convergence --> Act: Converged
    Convergence --> Round4: Not Converged (< max rounds)
    Round4 --> Round5: Not Converged
    Round5 --> Act: Max Rounds Reached
    
    Act --> VerifyEvidence: Generate Fractures & Countermeasures
    
    VerifyEvidence --> ComputeEntropy: Evidence Verified
    VerifyEvidence --> DropFracture: Evidence Failed
    
    DropFracture --> ComputeEntropy
    
    ComputeEntropy --> Output: Score Computed
    
    Output --> Learn: Display to Clinicians
    
    Learn --> [*]: Collect Feedback
```

### Data Flow Architecture

```mermaid
sequenceDiagram
    participant EHR as EHR System
    participant Connector as EHR Connector
    participant Normalize as Normalizer
    participant Timeline as Timeline Store
    participant Orchestrator as OODA Orchestrator
    participant Agents as Multi-Agent System
    participant Synthesis as Synthesis Agent
    participant Evidence as Evidence Verifier
    participant Entropy as Entropy Engine
    participant UI as Clinical UI
    
    EHR->>Connector: HL7/FHIR Message
    Connector->>Normalize: Raw Clinical Data
    Normalize->>Timeline: Canonical Facts + Timeline
    Timeline->>Orchestrator: Trigger OODA (Snapshot/Streaming)
    
    Orchestrator->>Agents: Round 1 - Parallel Analysis
    Agents-->>Orchestrator: JSON Claims + Evidence
    
    Orchestrator->>Agents: Round 2 - Sequential Rebuttal
    Agents-->>Orchestrator: Agreements/Disagreements
    
    Orchestrator->>Synthesis: Round 3 - Synthesize
    Synthesis-->>Orchestrator: Merged Fractures
    
    Orchestrator->>Evidence: Verify All Claims
    Evidence-->>Orchestrator: Verified Fractures
    
    Orchestrator->>Entropy: Compute Scores
    Entropy-->>Orchestrator: Entropy Scores
    
    Orchestrator->>UI: Fractures + Countermeasures
    UI->>Orchestrator: User Feedback
    Orchestrator->>Timeline: Update Calibration Data
```


## Components and Interfaces

### 1. EHR Connector Component

**Purpose**: Ingest clinical data from hospital EHR systems via HL7 v2.x and FHIR R4 standards.

**Interfaces**:
```typescript
interface EHRConnector {
  // HL7 v2.x ingestion
  ingestHL7Message(message: HL7Message): Promise<IngestionResult>;
  
  // FHIR R4 ingestion
  ingestFHIRResource(resource: FHIRResource): Promise<IngestionResult>;
  
  // Connection management
  configureEndpoint(config: EHREndpointConfig): Promise<void>;
  testConnection(): Promise<ConnectionStatus>;
  
  // Error handling
  retryFailedMessage(messageId: string): Promise<IngestionResult>;
}

interface HL7Message {
  messageType: 'ADT' | 'ORM' | 'ORU' | 'MDM';
  segments: HL7Segment[];
  timestamp: Date;
  sourceSystem: string;
}

interface FHIRResource {
  resourceType: 'Patient' | 'Observation' | 'MedicationRequest' | 'Encounter' | 'Condition';
  id: string;
  data: any; // FHIR R4 compliant JSON
}

interface IngestionResult {
  success: boolean;
  messageId: string;
  patientId: string;
  errors?: string[];
  normalizedFacts: ClinicalFact[];
}
```

**Implementation Details**:
- Supports Epic, Cerner, and Indian HMIS vendors
- Implements exponential backoff retry (3 attempts, 1s/2s/4s delays)
- Validates message structure against HL7 v2.x and FHIR R4 schemas
- Extracts patient demographics, vitals, lab results, medications, encounters
- Stores raw messages in S3 for audit trail
- Publishes ingestion events to EventBridge for downstream processing

### 2. Normalization Engine Component

**Purpose**: Convert heterogeneous clinical data into canonical Patient Timeline format with extracted structured facts.

**Interfaces**:
```typescript
interface NormalizationEngine {
  normalize(rawData: RawClinicalData): Promise<NormalizationResult>;
  expandAbbreviation(abbrev: string, hospitalId: string): string;
  extractFacts(text: string): Promise<ClinicalFact[]>;
  resolveTemporalReference(ref: string, context: TemporalContext): Date;
}

interface RawClinicalData {
  source: 'HL7' | 'FHIR' | 'PDF' | 'OCR' | 'ASR' | 'MANUAL';
  patientId: string;
  content: any;
  timestamp: Date;
  confidence?: number; // For OCR/ASR
}

interface NormalizationResult {
  patientTimeline: PatientTimeline;
  extractedFacts: ClinicalFact[];
  qualityFlags: DataQualityFlag[];
}

interface PatientTimeline {
  patientId: string;
  events: TimelineEvent[];
  currentState: PatientState;
}

interface TimelineEvent {
  eventId: string;
  timestamp: Date;
  eventType: 'ADMISSION' | 'DISCHARGE' | 'TRANSFER' | 'MEDICATION' | 'LAB' | 'VITAL' | 'NOTE' | 'PROCEDURE';
  facts: ClinicalFact[];
  sourceReference: SourceReference;
}

interface ClinicalFact {
  factId: string;
  factType: string;
  value: any;
  unit?: string;
  confidence: number;
  sourceReference: SourceReference;
}

interface SourceReference {
  sourceType: string;
  sourceId: string;
  location: string; // Substring location or structured path
}

interface DataQualityFlag {
  flagType: 'LOW_OCR_CONFIDENCE' | 'LOW_ASR_CONFIDENCE' | 'MISSING_FIELD' | 'AMBIGUOUS_TEMPORAL';
  severity: 'LOW' | 'MEDIUM' | 'HIGH';
  affectedFacts: string[];
}
```

**Implementation Details**:
- Uses AWS Bedrock Claude for fact extraction from unstructured text
- Maintains hospital-specific abbreviation dictionaries in DynamoDB
- Applies OCR/ASR confidence thresholds (0.85 minimum)
- Resolves relative temporal references ("yesterday", "2 hours ago") to absolute timestamps
- Maintains bidirectional links between normalized facts and source data
- Flags low-quality data for entropy score penalties


### 3. OODA Orchestrator Component

**Purpose**: Coordinate the Observe-Orient-Decide-Act loop for semantic fracture detection.

**Interfaces**:
```typescript
interface OODAOrchestrator {
  // Mode-specific triggers
  triggerSnapshotOODA(clinicalMoment: ClinicalMoment): Promise<OODAResult>;
  startStreamingOODA(patientId: string): Promise<StreamingSession>;
  stopStreamingOODA(sessionId: string): Promise<void>;
  runRetrospectiveAudit(params: AuditParams): Promise<AuditResult>;
  
  // OODA loop execution
  observe(patientId: string): Promise<PatientState>;
  orient(patientState: PatientState): Promise<AgentWorldModel[]>;
  decide(worldModels: AgentWorldModel[]): Promise<SemanticFracture[]>;
  act(fractures: SemanticFracture[]): Promise<Countermeasure[]>;
}

interface ClinicalMoment {
  momentType: 'SHIFT_HANDOVER' | 'DISCHARGE' | 'ICU_TRANSFER' | 'MEDICATION_CHANGE' | 'POST_OP_TRANSFER' | 'HIGH_RISK_LAB';
  patientId: string;
  timestamp: Date;
  triggerData: any;
}

interface OODAResult {
  sessionId: string;
  patientId: string;
  fractures: SemanticFracture[];
  countermeasures: Countermeasure[];
  executionTime: number;
  tokenUsage: number;
}

interface StreamingSession {
  sessionId: string;
  patientId: string;
  startTime: Date;
  status: 'ACTIVE' | 'PAUSED' | 'STOPPED';
}

interface AuditParams {
  startDate: Date;
  endDate: Date;
  patientCohort?: PatientFilter;
  fractureTypes?: FractureType[];
}

interface AuditResult {
  totalCases: number;
  fracturesByType: Map<FractureType, number>;
  highRiskCases: CaseReference[];
  complianceReport: ComplianceReport;
}
```

**Implementation Details**:
- Implemented as AWS Step Functions state machine
- Snapshot OODA: Triggered by EventBridge rules matching clinical moments
- Streaming OODA: Long-running Lambda polling DynamoDB Streams for patient updates
- Retrospective Audit: Batch processing using Lambda with S3 event triggers
- Enforces 40K token budget per case across all agent invocations
- Implements early termination when convergence score > 0.85
- Tracks execution metrics (latency, token usage, costs) in CloudWatch

### 4. Multi-Agent System Component

**Purpose**: Execute adversarial multi-round debates with six specialized agents to identify semantic fractures.

**Interfaces**:
```typescript
interface MultiAgentSystem {
  // Debate orchestration
  executeRound1(patientState: PatientState): Promise<AgentClaim[]>;
  executeRound2(round1Claims: AgentClaim[]): Promise<AgentRebuttal[]>;
  executeRound3(round2Rebuttals: AgentRebuttal[]): Promise<SemanticFracture[]>;
  
  // Dynamic round control
  computeConvergenceScore(claims: AgentClaim[]): number;
  shouldTerminateEarly(convergenceScore: number): boolean;
  
  // Agent invocation
  invokeAgent(agent: AgentType, input: AgentInput): Promise<AgentOutput>;
}

interface AgentInput {
  patientTimeline: PatientTimeline;
  patientState: PatientState;
  roundNumber: number;
  priorClaims?: AgentClaim[];
}

interface AgentOutput {
  agentType: AgentType;
  claims: AgentClaim[];
  reasoning: string;
  tokenUsage: number;
}

interface AgentClaim {
  claimId: string;
  agentType: AgentType;
  fractureType: FractureType;
  description: string;
  evidence: Evidence[];
  affectedRoles: ClinicalRole[];
  confidence: number;
}

interface Evidence {
  evidenceId: string;
  sourceReference: SourceReference;
  excerpt: string;
  evidenceType: 'DIRECT_QUOTE' | 'STRUCTURED_FACT' | 'TEMPORAL_PATTERN';
}

interface AgentRebuttal {
  rebuttalId: string;
  agentType: AgentType;
  targetClaimId: string;
  stance: 'AGREE' | 'DISAGREE' | 'PARTIAL';
  reasoning: string;
  counterEvidence?: Evidence[];
}

type AgentType = 'NURSE' | 'DOCTOR' | 'PCP' | 'PATIENT' | 'ADVERSARY' | 'COMPLIANCE';
type FractureType = 'MEDICATION_RECONCILIATION' | 'FOLLOW_UP_COORDINATION' | 'CONTRAINDICATION_OMISSION' | 
                    'HANDOVER_TASK_OMISSION' | 'CRITICAL_CONTEXT_LOSS' | 'PATIENT_COMPREHENSION' | 
                    'RESPONSIBILITY_AMBIGUITY' | 'TIMELINE_AMBIGUITY' | 'DEVICE_AFTERCARE';
type ClinicalRole = 'NURSE' | 'DOCTOR' | 'PCP' | 'PATIENT' | 'CAREGIVER';
```

**Implementation Details**:
- Each agent implemented as separate AWS Bedrock Claude invocation with role-specific system prompts
- Round 1: Parallel Lambda invocations (6 concurrent)
- Round 2: Sequential Lambda invocations with prior round context
- Round 3: Synthesis Agent (separate invocation) merges and deduplicates claims
- Adversary Agent constrained to cite evidence only (no invented facts)
- Convergence score: 1 - (disagreement_rate + contradiction_rate) / 2
- JSON schema enforcement for structured outputs
- Prompt templates stored in S3, versioned for A/B testing


### 5. Evidence Verification Component

**Purpose**: Verify all semantic fracture claims against source data using three-layer verification to prevent hallucinations.

**Interfaces**:
```typescript
interface EvidenceVerifier {
  verifyFracture(fracture: SemanticFracture): Promise<VerificationResult>;
  verifyEvidence(evidence: Evidence, sourceData: SourceData): Promise<boolean>;
  
  // Three-layer verification
  substringMatch(excerpt: string, sourceText: string): boolean;
  fuzzyMatch(excerpt: string, sourceText: string, threshold: number): number;
  structuredFactMatch(evidence: Evidence, facts: ClinicalFact[]): boolean;
}

interface SemanticFracture {
  fractureId: string;
  fractureType: FractureType;
  description: string;
  evidence: Evidence[];
  affectedRoles: ClinicalRole[];
  agentConsensus: AgentConsensusInfo;
  entropyScore?: number; // Computed later
}

interface VerificationResult {
  verified: boolean;
  verificationMethod: 'SUBSTRING' | 'FUZZY' | 'STRUCTURED' | 'FAILED';
  confidence: number;
  failedEvidence?: Evidence[];
}

interface AgentConsensusInfo {
  supportingAgents: AgentType[];
  opposingAgents: AgentType[];
  consensusScore: number;
}

interface SourceData {
  sourceId: string;
  sourceType: string;
  content: string | any;
  facts: ClinicalFact[];
}
```

**Implementation Details**:
- Layer 1 (Substring): Exact string matching using Boyer-Moore algorithm
- Layer 2 (Fuzzy): Levenshtein distance with 0.85 similarity threshold
- Layer 3 (Structured): Semantic matching against normalized facts using embeddings
- Failure behavior: Drop unsupported fracture, log hallucination event, reduce entropy score
- Verification results cached in ElastiCache for performance
- All verified fractures include evidence source links for clinical review

### 6. Entropy Score Engine Component

**Purpose**: Compute calibrated risk scores (0.0-1.0) for semantic fractures using four-layer scoring model.

**Interfaces**:
```typescript
interface EntropyScoreEngine {
  computeEntropyScore(fracture: SemanticFracture, context: ScoringContext): Promise<number>;
  
  // Four-layer scoring
  computeLayer1Heuristic(fracture: SemanticFracture): number;
  computeLayer2LLMConsistency(fracture: SemanticFracture): number;
  computeLayer3DataQuality(context: ScoringContext): number;
  computeLayer4Calibration(rawScore: number, context: ScoringContext): number;
  
  // Threshold management
  getThreshold(hospitalId: string, unitId?: string): number;
  updateThreshold(hospitalId: string, unitId: string | null, threshold: number): Promise<void>;
}

interface ScoringContext {
  patientState: PatientState;
  dataQualityFlags: DataQualityFlag[];
  hospitalId: string;
  unitId?: string;
  historicalFeedback?: FeedbackRecord[];
}

interface EntropyScoreBreakdown {
  finalScore: number;
  layer1Score: number;
  layer2Score: number;
  layer3Score: number;
  layer4Score: number;
  weights: LayerWeights;
  threshold: number;
  isHighRisk: boolean;
}

interface LayerWeights {
  layer1: 0.55;
  layer2: 0.30;
  layer3: 0.15;
}
```

**Implementation Details**:

**Layer 1 - Heuristic Scoring (w=0.55)**:
- Fracture type base risk: CONTRAINDICATION_OMISSION (0.9), MEDICATION_RECONCILIATION (0.8), CRITICAL_CONTEXT_LOSS (0.75), etc.
- Patient risk factors: ICU status (+0.1), polypharmacy (+0.05), age >75 (+0.05), comorbidities (+0.05)
- Temporal urgency: Discharge within 4 hours (+0.1), shift change within 1 hour (+0.05)
- Weighted sum normalized to 0.0-1.0

**Layer 2 - LLM Consistency (w=0.30)**:
- Agent disagreement rate: (opposing_agents / total_agents)
- Contradiction detection: Semantic similarity between conflicting claims
- Evidence quality: Average confidence across all evidence items
- Score = 1 - (disagreement_rate * 0.5 + contradiction_rate * 0.5)

**Layer 3 - Data Quality (w=0.15)**:
- OCR/ASR confidence penalties: (1 - avg_confidence) if < 0.85
- Missing field penalties: 0.1 per critical missing field
- Temporal ambiguity penalties: 0.05 per unresolved temporal reference
- Score = 1 - total_penalties (clamped to 0.0-1.0)

**Layer 4 - Calibration**:
- Logistic regression: P(true_positive | raw_score, hospital_features)
- Isotonic regression: Monotonic mapping from raw scores to calibrated probabilities
- Trained on feedback data (true_positive, false_positive labels)
- Separate models per hospital when sufficient data (n > 100 feedback records)

**Threshold Management**:
- Default threshold: 0.7
- Hospital-specific thresholds: Computed from feedback to achieve target false positive rate (20%)
- Unit-specific thresholds: Further refinement for high-volume units (ICU, ED)
- Stored in DynamoDB with version history


### 7. Countermeasure Generator Component

**Purpose**: Generate role-specific actionable recommendations to prevent semantic fractures.

**Interfaces**:
```typescript
interface CountermeasureGenerator {
  generateCountermeasures(fracture: SemanticFracture): Promise<Countermeasure[]>;
  generateRoleSpecificCountermeasure(fracture: SemanticFracture, role: ClinicalRole): Promise<Countermeasure>;
}

interface Countermeasure {
  countermeasureId: string;
  fractureId: string;
  targetRole: ClinicalRole;
  priority: 'HIGH' | 'MEDIUM' | 'LOW';
  actionSteps: ActionStep[];
  completionCriteria: string;
  estimatedTime: number; // minutes
}

interface ActionStep {
  stepNumber: number;
  description: string;
  actionType: 'VERIFY' | 'COMMUNICATE' | 'DOCUMENT' | 'ESCALATE' | 'EDUCATE';
  specificDetails: string;
}
```

**Implementation Details**:
- Nurse countermeasures: Focus on immediate task clarity, time-efficient actions, patient safety checks
- Doctor countermeasures: Focus on clinical coherence, liability mitigation, treatment plan clarification
- PCP countermeasures: Focus on continuity of care, follow-up scheduling, outpatient handover
- Patient countermeasures: Focus on comprehension aids, simplified instructions, teach-back prompts
- Generated using AWS Bedrock with role-specific prompt templates
- Prioritized by entropy score (HIGH: >0.7, MEDIUM: 0.5-0.7, LOW: <0.5)
- Stored in DynamoDB linked to fracture records

### 8. Patient Comprehension System Component

**Purpose**: Generate multilingual, grade-6 readable discharge instructions with teach-back validation.

**Interfaces**:
```typescript
interface PatientComprehensionSystem {
  generatePatientInstructions(patientState: PatientState, language: Language): Promise<PatientInstructions>;
  generateCaregiverInstructions(patientState: PatientState, language: Language): Promise<CaregiverInstructions>;
  generateTeachBackValidation(instructions: PatientInstructions): Promise<TeachBackValidation>;
  
  // Readability analysis
  analyzeReadability(text: string): ReadabilityMetrics;
  simplifyText(text: string, targetGrade: number): Promise<string>;
}

interface PatientInstructions {
  instructionId: string;
  patientId: string;
  language: Language;
  sections: InstructionSection[];
  warningSignsList: WarningSigns;
  followUpSchedule: FollowUpSchedule;
  readabilityGrade: number;
  doctorApproved: boolean;
}

interface InstructionSection {
  sectionType: 'MEDICATIONS' | 'DIET' | 'ACTIVITY' | 'WOUND_CARE' | 'SYMPTOMS_TO_WATCH';
  title: string;
  content: string[];
  visualAids?: string[]; // URLs to diagrams/images
}

interface WarningSigns {
  urgentSigns: string[]; // Call 108/emergency
  concerningSigns: string[]; // Call doctor
  normalExpectations: string[]; // What's normal
}

interface FollowUpSchedule {
  appointments: Appointment[];
  labTests: LabTest[];
  medicationRefills: MedicationRefill[];
}

interface TeachBackValidation {
  simplifiedParaphrase: string;
  comprehensionQuestions: ComprehensionQuestion[];
  keyPointsChecklist: string[];
}

interface ComprehensionQuestion {
  question: string;
  expectedAnswer: string;
  assessmentCriteria: string;
}

type Language = 'ENGLISH' | 'HINDI' | 'TAMIL' | 'MARATHI' | 'TELUGU' | 'BENGALI';
```

**Implementation Details**:
- Uses AWS Bedrock Claude for multilingual generation
- Readability analysis: Flesch-Kincaid Grade Level, target ≤ 6
- Sentence length: Maximum 15 words per sentence
- Vocabulary: Avoids medical jargon, expands abbreviations
- Translation: Native speaker validation for medical accuracy
- Teach-back: Patient_Agent generates paraphrases and questions
- Doctor review workflow: Editable preview before printout approval
- Printout format: A4 PDF with large fonts (14pt minimum), high contrast


### 9. Feedback and Learning Component

**Purpose**: Collect clinician feedback and update calibration models to improve accuracy over time.

**Interfaces**:
```typescript
interface FeedbackCollector {
  submitFeedback(feedback: FeedbackSubmission): Promise<void>;
  getFeedbackStats(hospitalId: string, dateRange: DateRange): Promise<FeedbackStats>;
}

interface FeedbackSubmission {
  fractureId: string;
  userId: string;
  userRole: ClinicalRole;
  label: FeedbackLabel;
  comments?: string;
  timestamp: Date;
}

type FeedbackLabel = 'TRUE_POSITIVE' | 'FALSE_POSITIVE' | 'UNCLEAR' | 'HELPFUL_BUT_NOT_RISK' | 'MISSED_RISK_REPORTED';

interface FeedbackStats {
  totalFeedback: number;
  labelDistribution: Map<FeedbackLabel, number>;
  alertAcknowledgementRate: number;
  medianTimeToAcknowledge: number;
  falsePositiveRate: number;
}

interface CalibrationEngine {
  updateCalibration(hospitalId: string): Promise<CalibrationUpdate>;
  updateFracturePriors(feedbackData: FeedbackRecord[]): Promise<void>;
  tuneThresholds(feedbackData: FeedbackRecord[], targetFPR: number): Promise<ThresholdUpdate>;
  updateAgentPrompts(feedbackData: FeedbackRecord[]): Promise<PromptUpdate>;
  expandAbbreviationDictionary(feedbackData: FeedbackRecord[]): Promise<DictionaryUpdate>;
}

interface CalibrationUpdate {
  hospitalId: string;
  updateType: 'PRIORS' | 'THRESHOLDS' | 'PROMPTS' | 'DICTIONARY';
  previousMetrics: PerformanceMetrics;
  updatedMetrics: PerformanceMetrics;
  changesApplied: any;
}

interface PerformanceMetrics {
  alertAckRate: number;
  falsePositiveRate: number;
  missRate: number;
  precision: number;
  recall: number;
}
```

**Implementation Details**:

**Feedback Collection**:
- In-app buttons on all fracture displays
- Nurse Supervisor review workflow: Batch review of unit alerts
- Doctor review workflow: Individual case review with detailed comments
- Quality Team audit workflow: Retrospective case analysis
- Stored in DynamoDB with indexes on hospitalId, fractureType, label, timestamp

**Calibration Updates**:
- Triggered weekly or when feedback count > 50 new records
- Fracture priors: Bayesian update of P(fracture_type | features)
- Threshold tuning: Binary search to achieve target FPR (20%) while maximizing recall
- Agent prompts: Identify common false positive patterns, add negative examples to prompts
- Abbreviation dictionary: Extract hospital-specific abbreviations from feedback comments

**Learning Objectives**:
- Target alert acknowledgement rate: 75%
- Target false positive rate: 20%
- Target miss rate: 10%
- Computed per hospital, per unit, per fracture type

**Model Retraining**:
- Layer 4 calibration: Retrain logistic/isotonic regression monthly
- Minimum data requirements: 100 feedback records for hospital-level, 50 for unit-level
- Cross-validation: 80/20 train/test split, monitor calibration curves
- A/B testing: New models deployed to 10% of cases, promoted if metrics improve

### 10. User Interface Components

#### Battlefield Dashboard

**Purpose**: Real-time command center for Clinical Administrators and Patient Safety Officers.

**Key Features**:
- Live threat map: All active high-risk fractures across hospital
- Unit grouping: Collapsible sections per hospital unit (ICU, ED, Med-Surg, etc.)
- Sorting: By entropy score (descending), timestamp, fracture type
- Filtering: By fracture type, unit, entropy score range, date range
- Detail view: Click fracture to see evidence, countermeasures, patient context
- Acknowledgement tracking: Visual indicators for acknowledged vs. unacknowledged alerts
- Auto-refresh: 10-second polling for new fractures

**Technology**:
- React + TypeScript
- Real-time updates via WebSocket (AWS API Gateway WebSocket API)
- State management: Redux Toolkit
- Visualization: D3.js for threat map, Recharts for analytics
- Responsive design: Desktop-first, tablet-compatible

#### Nurse Tactical View

**Purpose**: Focused interface for Charge Nurses and bedside nurses.

**Key Features**:
- Patient list: Assigned unit patients with fracture counts
- High-risk highlighting: Red badges for entropy score > 0.7
- Quick actions: One-tap acknowledge, quick feedback buttons
- Nurse-specific countermeasures: Time-efficient action steps
- Offline mode: Service worker caching, background sync
- Shift handover mode: Summary of all active fractures for incoming shift

**Technology**:
- Progressive Web App (PWA)
- Offline-first architecture: IndexedDB for local storage
- Background sync: Queue actions during offline, sync on reconnect
- Push notifications: Web Push API for high-risk alerts
- Mobile-optimized: Touch-friendly, large tap targets

#### Doctor Review View

**Purpose**: Detailed review interface for attending physicians and consultants.

**Key Features**:
- Patient-centric view: All fractures for assigned patients
- Evidence display: Full source data with highlighting
- Agent debate summary: Reasoning from each agent, consensus visualization
- Approve/reject workflow: Validate fracture findings
- Patient instruction editor: WYSIWYG editor for discharge instructions
- Approval workflow: Sign-off before printout generation
- Clinical context: Integrated view of patient timeline, medications, labs

**Technology**:
- React + TypeScript
- Rich text editor: Quill.js for instruction editing
- PDF preview: React-PDF for printout preview
- Signature capture: Electronic signature for approvals
- HIPAA-compliant audit logging

#### Patient Printout Generator

**Purpose**: Generate patient-friendly discharge instructions.

**Key Features**:
- Language selector: Six supported languages
- Preview mode: Real-time preview of formatted instructions
- Font size adjustment: Accessibility controls
- Section customization: Enable/disable sections
- Print optimization: A4 format, high contrast, large fonts
- QR code generation: Link to digital copy of instructions
- Caregiver version: Separate printout with additional details

**Technology**:
- React + TypeScript
- PDF generation: jsPDF + html2canvas
- Internationalization: i18next for translations
- Print CSS: Optimized for physical printouts
- Accessibility: WCAG 2.1 AA compliance


## Data Models

### Core Data Models

```typescript
// Patient State Model
interface PatientState {
  patientId: string;
  hospitalId: string;
  unitId: string;
  
  // Demographics
  demographics: {
    age: number;
    gender: string;
    language: Language;
  };
  
  // Current clinical status
  currentStatus: {
    admissionDate: Date;
    location: string;
    primaryDiagnosis: string;
    comorbidities: string[];
    allergies: Allergy[];
    currentMedications: Medication[];
    recentLabs: LabResult[];
    recentVitals: VitalSigns[];
  };
  
  // Care team
  careTeam: {
    attendingPhysician: string;
    residents: string[];
    nurses: string[];
    pcp?: string;
  };
  
  // Risk factors
  riskFactors: {
    isICU: boolean;
    polypharmacy: boolean;
    highRiskMedications: boolean;
    cognitiveImpairment: boolean;
    languageBarrier: boolean;
  };
  
  // Timeline reference
  timelineId: string;
  lastUpdated: Date;
}

// Semantic Fracture Model
interface SemanticFracture {
  fractureId: string;
  patientId: string;
  hospitalId: string;
  unitId: string;
  
  // Fracture details
  fractureType: FractureType;
  description: string;
  detectedAt: Date;
  
  // Evidence
  evidence: Evidence[];
  verificationStatus: 'VERIFIED' | 'FAILED' | 'PENDING';
  
  // Scoring
  entropyScore: number;
  entropyBreakdown: EntropyScoreBreakdown;
  isHighRisk: boolean;
  
  // Agent consensus
  agentConsensus: AgentConsensusInfo;
  
  // Affected parties
  affectedRoles: ClinicalRole[];
  
  // Countermeasures
  countermeasures: Countermeasure[];
  
  // Lifecycle
  status: 'ACTIVE' | 'ACKNOWLEDGED' | 'RESOLVED' | 'DISMISSED';
  acknowledgedBy?: string;
  acknowledgedAt?: Date;
  
  // Feedback
  feedback?: FeedbackSubmission;
}

// Agent Prompt Template Model
interface AgentPromptTemplate {
  templateId: string;
  agentType: AgentType;
  version: string;
  
  systemPrompt: string;
  userPromptTemplate: string;
  
  // Optimization parameters
  temperature: number;
  maxTokens: number;
  
  // Performance tracking
  performanceMetrics: {
    avgTokenUsage: number;
    avgLatency: number;
    feedbackScore: number;
  };
  
  // Versioning
  createdAt: Date;
  createdBy: string;
  isActive: boolean;
  previousVersion?: string;
}

// Hospital Configuration Model
interface HospitalConfiguration {
  hospitalId: string;
  hospitalName: string;
  
  // Thresholds
  entropyThresholds: {
    default: number;
    unitSpecific: Map<string, number>;
  };
  
  // Clinical moment triggers
  clinicalMomentRules: {
    shiftHandoverTimes: string[]; // HH:MM format
    dischargeLeadTime: number; // hours
    highRiskLabThresholds: Map<string, number>;
  };
  
  // Abbreviation dictionary
  abbreviationDictionary: Map<string, string>;
  
  // Multi-agent debate config
  debateConfig: {
    minRounds: number;
    maxRounds: number;
    convergenceThreshold: number;
  };
  
  // Notification preferences
  notificationConfig: {
    enablePush: boolean;
    enableEmail: boolean;
    enableSMS: boolean;
    rateLimitPerHour: number;
  };
  
  // EHR integration
  ehrConfig: {
    vendor: 'EPIC' | 'CERNER' | 'HMIS';
    hl7Endpoint: string;
    fhirEndpoint: string;
    credentials: string; // Encrypted
  };
  
  // Learning parameters
  learningConfig: {
    targetAckRate: number;
    targetFPR: number;
    targetMissRate: number;
    calibrationFrequency: 'WEEKLY' | 'MONTHLY';
  };
}
```

### Database Schema (DynamoDB)

**Table: Patients**
- Partition Key: `patientId` (String)
- Sort Key: `hospitalId` (String)
- Attributes: PatientState (JSON)
- GSI: `hospitalId-unitId-index` for unit-level queries
- TTL: 90 days after discharge

**Table: PatientTimelines**
- Partition Key: `timelineId` (String)
- Sort Key: `timestamp` (Number)
- Attributes: TimelineEvent (JSON)
- GSI: `patientId-timestamp-index` for patient history queries
- TTL: 7 years for compliance

**Table: SemanticFractures**
- Partition Key: `fractureId` (String)
- Sort Key: `detectedAt` (Number)
- Attributes: SemanticFracture (JSON)
- GSI: `patientId-detectedAt-index` for patient fracture history
- GSI: `hospitalId-status-entropyScore-index` for dashboard queries
- GSI: `hospitalId-fractureType-detectedAt-index` for analytics
- TTL: 7 years for compliance

**Table: Feedback**
- Partition Key: `fractureId` (String)
- Sort Key: `timestamp` (Number)
- Attributes: FeedbackSubmission (JSON)
- GSI: `hospitalId-label-timestamp-index` for learning queries
- No TTL (permanent for learning)

**Table: HospitalConfigurations**
- Partition Key: `hospitalId` (String)
- Attributes: HospitalConfiguration (JSON)
- No TTL (permanent)

**Table: AgentPromptTemplates**
- Partition Key: `agentType` (String)
- Sort Key: `version` (String)
- Attributes: AgentPromptTemplate (JSON)
- GSI: `agentType-isActive-index` for active template lookup
- No TTL (permanent)

**Table: CalibrationModels**
- Partition Key: `hospitalId` (String)
- Sort Key: `modelType-version` (String)
- Attributes: Serialized model parameters (JSON)
- No TTL (permanent)


## API Specifications

### REST API Endpoints

#### Analysis API

**POST /api/v1/analysis/snapshot**
```typescript
// Trigger snapshot OODA analysis
Request: {
  hospitalId: string;
  patientId: string;
  clinicalMoment: {
    type: 'SHIFT_HANDOVER' | 'DISCHARGE' | 'ICU_TRANSFER' | 'MEDICATION_CHANGE' | 'POST_OP_TRANSFER' | 'HIGH_RISK_LAB';
    timestamp: string; // ISO 8601
    triggerData: any;
  };
  inputData: {
    source: 'MANUAL' | 'OCR' | 'ASR' | 'EHR';
    content: string | object;
    metadata?: any;
  };
}

Response: {
  sessionId: string;
  status: 'QUEUED' | 'PROCESSING' | 'COMPLETED' | 'FAILED';
  estimatedCompletionTime: string; // ISO 8601
}
```

**GET /api/v1/analysis/{sessionId}**
```typescript
// Get analysis results
Response: {
  sessionId: string;
  patientId: string;
  status: 'PROCESSING' | 'COMPLETED' | 'FAILED';
  result?: {
    fractures: SemanticFracture[];
    countermeasures: Countermeasure[];
    patientInstructions?: PatientInstructions;
    executionMetrics: {
      totalTime: number; // milliseconds
      tokenUsage: number;
      cost: number;
    };
  };
  error?: string;
}
```

**POST /api/v1/analysis/streaming/start**
```typescript
// Start streaming OODA for a patient
Request: {
  hospitalId: string;
  patientId: string;
  config?: {
    triggerRules?: string[];
    notificationPreferences?: object;
  };
}

Response: {
  streamingSessionId: string;
  status: 'ACTIVE';
  startedAt: string; // ISO 8601
}
```

**POST /api/v1/analysis/streaming/{sessionId}/stop**
```typescript
// Stop streaming OODA
Response: {
  streamingSessionId: string;
  status: 'STOPPED';
  stoppedAt: string; // ISO 8601
  summary: {
    totalFracturesDetected: number;
    highRiskFractures: number;
    duration: number; // seconds
  };
}
```


**POST /api/v1/analysis/audit**
```typescript
// Run retrospective audit
Request: {
  hospitalId: string;
  startDate: string; // ISO 8601
  endDate: string; // ISO 8601
  filters?: {
    unitIds?: string[];
    fractureTypes?: FractureType[];
    minEntropyScore?: number;
  };
}

Response: {
  auditId: string;
  status: 'QUEUED' | 'PROCESSING' | 'COMPLETED';
  estimatedCompletionTime: string; // ISO 8601
}
```

#### Fracture API

**GET /api/v1/fractures**
```typescript
// List fractures with filtering and pagination
Query Parameters: {
  hospitalId: string;
  unitId?: string;
  patientId?: string;
  status?: 'ACTIVE' | 'ACKNOWLEDGED' | 'RESOLVED' | 'DISMISSED';
  minEntropyScore?: number;
  fractureType?: FractureType;
  startDate?: string; // ISO 8601
  endDate?: string; // ISO 8601
  limit?: number; // default 50, max 200
  cursor?: string; // pagination cursor
}

Response: {
  fractures: SemanticFracture[];
  pagination: {
    nextCursor?: string;
    hasMore: boolean;
    total: number;
  };
}
```

**GET /api/v1/fractures/{fractureId}**
```typescript
// Get detailed fracture information
Response: SemanticFracture & {
  patientContext: PatientState;
  agentDebateTranscript: {
    round1: AgentOutput[];
    round2: AgentRebuttal[];
    synthesis: AgentOutput;
  };
  verificationDetails: VerificationResult;
}
```

**POST /api/v1/fractures/{fractureId}/acknowledge**
```typescript
// Acknowledge a fracture
Request: {
  userId: string;
  userRole: ClinicalRole;
  notes?: string;
}

Response: {
  fractureId: string;
  status: 'ACKNOWLEDGED';
  acknowledgedBy: string;
  acknowledgedAt: string; // ISO 8601
}
```


**POST /api/v1/fractures/{fractureId}/feedback**
```typescript
// Submit feedback on a fracture
Request: {
  userId: string;
  userRole: ClinicalRole;
  label: 'TRUE_POSITIVE' | 'FALSE_POSITIVE' | 'UNCLEAR' | 'HELPFUL_BUT_NOT_RISK' | 'MISSED_RISK_REPORTED';
  comments?: string;
  suggestedCorrection?: string;
}

Response: {
  feedbackId: string;
  fractureId: string;
  submittedAt: string; // ISO 8601
}
```

#### Patient Instructions API

**POST /api/v1/patients/{patientId}/instructions**
```typescript
// Generate patient discharge instructions
Request: {
  language: Language;
  includeTeachBack: boolean;
  customSections?: string[];
}

Response: {
  instructionId: string;
  instructions: PatientInstructions;
  teachBackValidation?: TeachBackValidation;
  pdfUrl?: string; // Pre-signed S3 URL
}
```

**PUT /api/v1/patients/{patientId}/instructions/{instructionId}**
```typescript
// Update patient instructions (doctor review)
Request: {
  userId: string;
  sections: InstructionSection[];
  approved: boolean;
}

Response: {
  instructionId: string;
  version: number;
  approved: boolean;
  approvedBy?: string;
  approvedAt?: string; // ISO 8601
  pdfUrl: string; // Pre-signed S3 URL
}
```

#### Dashboard API

**GET /api/v1/dashboard/threat-map**
```typescript
// Get real-time threat map data
Query Parameters: {
  hospitalId: string;
  unitIds?: string[]; // comma-separated
  minEntropyScore?: number;
}

Response: {
  timestamp: string; // ISO 8601
  units: Array<{
    unitId: string;
    unitName: string;
    activeFractures: number;
    highRiskFractures: number;
    patients: Array<{
      patientId: string;
      fractureCount: number;
      maxEntropyScore: number;
      urgentFractures: SemanticFracture[];
    }>;
  }>;
  summary: {
    totalActiveFractures: number;
    totalHighRiskFractures: number;
    unacknowledgedFractures: number;
  };
}
```


**GET /api/v1/dashboard/analytics**
```typescript
// Get analytics and trends
Query Parameters: {
  hospitalId: string;
  startDate: string; // ISO 8601
  endDate: string; // ISO 8601
  groupBy?: 'DAY' | 'WEEK' | 'MONTH';
  unitId?: string;
}

Response: {
  timeSeries: Array<{
    timestamp: string; // ISO 8601
    fractureCount: number;
    highRiskCount: number;
    avgEntropyScore: number;
    acknowledgementRate: number;
  }>;
  fractureTypeDistribution: Map<FractureType, number>;
  topRiskPatients: Array<{
    patientId: string;
    fractureCount: number;
    avgEntropyScore: number;
  }>;
  performanceMetrics: {
    avgTimeToAcknowledge: number; // minutes
    falsePositiveRate: number;
    alertAcknowledgementRate: number;
  };
}
```

#### Configuration API

**GET /api/v1/hospitals/{hospitalId}/config**
```typescript
// Get hospital configuration
Response: HospitalConfiguration
```

**PUT /api/v1/hospitals/{hospitalId}/config**
```typescript
// Update hospital configuration
Request: Partial<HospitalConfiguration>

Response: {
  hospitalId: string;
  updatedAt: string; // ISO 8601
  config: HospitalConfiguration;
}
```

**POST /api/v1/hospitals/{hospitalId}/calibration/trigger**
```typescript
// Manually trigger calibration update
Response: {
  calibrationJobId: string;
  status: 'QUEUED';
  estimatedCompletionTime: string; // ISO 8601
}
```

### WebSocket API

**Connection**: `wss://api.medwar.io/ws`

**Authentication**: JWT token in query parameter or header

**Message Types**:

```typescript
// Client -> Server: Subscribe to updates
{
  type: 'SUBSCRIBE';
  channels: Array<{
    type: 'FRACTURES' | 'NOTIFICATIONS' | 'ANALYTICS';
    filters: {
      hospitalId: string;
      unitId?: string;
      patientId?: string;
    };
  }>;
}

// Server -> Client: New fracture detected
{
  type: 'FRACTURE_DETECTED';
  data: SemanticFracture;
  timestamp: string; // ISO 8601
}

// Server -> Client: Fracture status updated
{
  type: 'FRACTURE_UPDATED';
  data: {
    fractureId: string;
    status: string;
    updatedFields: string[];
  };
  timestamp: string; // ISO 8601
}

// Server -> Client: High-risk alert
{
  type: 'HIGH_RISK_ALERT';
  data: {
    fracture: SemanticFracture;
    urgency: 'IMMEDIATE' | 'URGENT';
    recommendedAction: string;
  };
  timestamp: string; // ISO 8601
}

// Client -> Server: Heartbeat
{
  type: 'PING';
}

// Server -> Client: Heartbeat response
{
  type: 'PONG';
  timestamp: string; // ISO 8601
}
```



## Security Architecture

### Authentication and Authorization

**Authentication Methods**:
1. **Clinician Authentication**: AWS Cognito User Pools with MFA
   - Username/password with OTP via SMS
   - Optional: SAML 2.0 integration with hospital SSO
   - Session duration: 8 hours (shift length)
   - Refresh token: 30 days

2. **EHR System Authentication**: API Keys + mTLS
   - Hospital-specific API keys stored in AWS Secrets Manager
   - Mutual TLS for HL7/FHIR connections
   - Key rotation: Every 90 days
   - IP whitelisting for hospital networks

3. **Mobile App Authentication**: OAuth 2.0 + Biometric
   - OAuth 2.0 authorization code flow
   - Biometric authentication (fingerprint/face) for quick access
   - Device binding to prevent token theft

**Authorization Model**: Role-Based Access Control (RBAC)

```typescript
enum Role {
  NURSE = 'NURSE',
  CHARGE_NURSE = 'CHARGE_NURSE',
  DOCTOR = 'DOCTOR',
  ATTENDING_PHYSICIAN = 'ATTENDING_PHYSICIAN',
  PATIENT_SAFETY_OFFICER = 'PATIENT_SAFETY_OFFICER',
  QUALITY_OFFICER = 'QUALITY_OFFICER',
  HOSPITAL_ADMIN = 'HOSPITAL_ADMIN',
  IT_ADMIN = 'IT_ADMIN',
  SYSTEM_ADMIN = 'SYSTEM_ADMIN'
}

interface Permission {
  resource: 'FRACTURES' | 'PATIENTS' | 'ANALYTICS' | 'CONFIG' | 'AUDIT_LOGS';
  actions: Array<'READ' | 'WRITE' | 'DELETE' | 'ACKNOWLEDGE' | 'APPROVE'>;
  scope: 'OWN' | 'UNIT' | 'HOSPITAL' | 'ALL';
}

const RolePermissions: Map<Role, Permission[]> = {
  NURSE: [
    { resource: 'FRACTURES', actions: ['READ', 'ACKNOWLEDGE'], scope: 'OWN' },
    { resource: 'PATIENTS', actions: ['READ'], scope: 'OWN' }
  ],
  CHARGE_NURSE: [
    { resource: 'FRACTURES', actions: ['READ', 'ACKNOWLEDGE'], scope: 'UNIT' },
    { resource: 'PATIENTS', actions: ['READ'], scope: 'UNIT' },
    { resource: 'ANALYTICS', actions: ['READ'], scope: 'UNIT' }
  ],
  DOCTOR: [
    { resource: 'FRACTURES', actions: ['READ', 'ACKNOWLEDGE', 'APPROVE'], scope: 'OWN' },
    { resource: 'PATIENTS', actions: ['READ', 'WRITE'], scope: 'OWN' }
  ],
  PATIENT_SAFETY_OFFICER: [
    { resource: 'FRACTURES', actions: ['READ', 'ACKNOWLEDGE'], scope: 'HOSPITAL' },
    { resource: 'ANALYTICS', actions: ['READ'], scope: 'HOSPITAL' },
    { resource: 'AUDIT_LOGS', actions: ['READ'], scope: 'HOSPITAL' }
  ],
  HOSPITAL_ADMIN: [
    { resource: 'FRACTURES', actions: ['READ'], scope: 'HOSPITAL' },
    { resource: 'ANALYTICS', actions: ['READ'], scope: 'HOSPITAL' },
    { resource: 'CONFIG', actions: ['READ', 'WRITE'], scope: 'HOSPITAL' },
    { resource: 'AUDIT_LOGS', actions: ['READ'], scope: 'HOSPITAL' }
  ],
  IT_ADMIN: [
    { resource: 'CONFIG', actions: ['READ', 'WRITE'], scope: 'HOSPITAL' },
    { resource: 'AUDIT_LOGS', actions: ['READ'], scope: 'HOSPITAL' }
  ],
  SYSTEM_ADMIN: [
    { resource: 'FRACTURES', actions: ['READ', 'WRITE', 'DELETE'], scope: 'ALL' },
    { resource: 'PATIENTS', actions: ['READ', 'WRITE', 'DELETE'], scope: 'ALL' },
    { resource: 'ANALYTICS', actions: ['READ'], scope: 'ALL' },
    { resource: 'CONFIG', actions: ['READ', 'WRITE'], scope: 'ALL' },
    { resource: 'AUDIT_LOGS', actions: ['READ'], scope: 'ALL' }
  ]
};
```


### Data Encryption

**Encryption at Rest**:
- **DynamoDB**: AWS KMS encryption with customer-managed keys (CMK)
  - Separate CMK per hospital for data isolation
  - Key rotation: Automatic annual rotation
  - Key policy: Hospital-specific IAM roles only

- **S3**: Server-Side Encryption with KMS (SSE-KMS)
  - Separate bucket prefix per hospital
  - Bucket policy: Enforce encryption, deny unencrypted uploads
  - Versioning enabled for audit trail

- **ElastiCache**: Encryption at rest enabled
  - Redis AUTH for access control
  - TLS for in-transit encryption

**Encryption in Transit**:
- **API Gateway**: TLS 1.3 only, strong cipher suites
- **EHR Connections**: Mutual TLS (mTLS) with certificate pinning
- **Internal Services**: VPC endpoints with PrivateLink
- **WebSocket**: WSS (WebSocket Secure) with TLS 1.3

**PII/PHI Redaction**:
```typescript
interface PIIRedactor {
  redact(text: string, hospitalId: string): RedactionResult;
  restore(redactedText: string, redactionMap: RedactionMap): string;
}

interface RedactionResult {
  redactedText: string;
  redactionMap: RedactionMap;
  entitiesDetected: PIIEntity[];
}

interface PIIEntity {
  type: 'NAME' | 'MRN' | 'PHONE' | 'EMAIL' | 'ADDRESS' | 'AADHAAR' | 'DATE_OF_BIRTH';
  originalValue: string;
  redactedValue: string; // e.g., [PATIENT_001]
  location: { start: number; end: number };
}

// Redaction rules
const RedactionRules = {
  NAME: /\b[A-Z][a-z]+ [A-Z][a-z]+\b/g,
  MRN: /\bMRN[:\s]*[A-Z0-9]{6,12}\b/gi,
  PHONE: /\b(\+91|0)?[6-9]\d{9}\b/g,
  EMAIL: /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g,
  AADHAAR: /\b\d{4}\s?\d{4}\s?\d{4}\b/g,
  DATE_OF_BIRTH: /\b(0?[1-9]|[12][0-9]|3[01])[\/\-](0?[1-9]|1[012])[\/\-]\d{4}\b/g
};
```

**Implementation**:
- Redaction applied before sending data to AWS Bedrock (cross-region)
- Original data stored in S3 with KMS encryption
- Redaction map stored separately with additional encryption layer
- Restoration only for authorized users with audit logging


### Audit Logging

**Audit Log Structure**:
```typescript
interface AuditLog {
  logId: string;
  timestamp: string; // ISO 8601
  hospitalId: string;
  
  // Actor information
  actor: {
    userId: string;
    userRole: Role;
    ipAddress: string;
    userAgent: string;
    sessionId: string;
  };
  
  // Action details
  action: {
    type: 'READ' | 'WRITE' | 'DELETE' | 'ACKNOWLEDGE' | 'APPROVE' | 'LOGIN' | 'LOGOUT' | 'CONFIG_CHANGE';
    resource: string; // e.g., 'FRACTURE', 'PATIENT', 'CONFIG'
    resourceId: string;
    outcome: 'SUCCESS' | 'FAILURE' | 'PARTIAL';
    errorMessage?: string;
  };
  
  // Data access tracking
  dataAccess?: {
    patientId?: string;
    fractureId?: string;
    fieldsAccessed: string[];
    piiAccessed: boolean;
  };
  
  // Change tracking
  changes?: {
    before: any;
    after: any;
    diff: string; // JSON patch format
  };
}
```

**Audit Log Retention**:
- **Hot storage** (DynamoDB): 90 days
- **Warm storage** (S3 Standard): 1 year
- **Cold storage** (S3 Glacier): 7 years (compliance requirement)
- **Immutable**: Write-once, no updates or deletes allowed

**Audit Events Tracked**:
1. All PHI/PII access (read, write, delete)
2. Authentication events (login, logout, failed attempts)
3. Authorization failures
4. Configuration changes
5. Fracture acknowledgements and feedback
6. Patient instruction approvals
7. EHR data ingestion
8. Calibration model updates
9. Threshold changes
10. Export/download of reports

**Audit Log Analysis**:
- Real-time anomaly detection using CloudWatch Logs Insights
- Alerts for suspicious patterns:
  - Multiple failed login attempts
  - Unusual data access patterns
  - Access outside normal hours
  - Bulk data exports
- Monthly audit reports for compliance teams


### Network Security

**VPC Architecture**:
```
VPC: 10.0.0.0/16

Public Subnets (10.0.1.0/24, 10.0.2.0/24):
- Application Load Balancer
- NAT Gateway
- Bastion Host (for emergency access)

Private Subnets (10.0.10.0/24, 10.0.11.0/24):
- Lambda Functions
- ECS Tasks (if using containers)
- ElastiCache

Data Subnets (10.0.20.0/24, 10.0.21.0/24):
- DynamoDB VPC Endpoints
- S3 VPC Endpoints
```

**Security Groups**:
- **ALB Security Group**: Allow HTTPS (443) from internet, HTTP (80) redirect to HTTPS
- **Lambda Security Group**: Allow outbound to DynamoDB, S3, Bedrock endpoints
- **ElastiCache Security Group**: Allow inbound from Lambda SG only
- **Bastion Security Group**: Allow SSH (22) from specific IP ranges only

**Network ACLs**:
- Deny all by default
- Allow inbound HTTPS to public subnets
- Allow outbound to AWS service endpoints
- Block known malicious IP ranges

**AWS WAF Rules**:
1. **Rate Limiting**: Max 100 requests per 5 minutes per IP
2. **Geo-Blocking**: Allow only India and specific countries
3. **SQL Injection Protection**: Block common SQL injection patterns
4. **XSS Protection**: Block common XSS patterns
5. **Known Bad Inputs**: Block requests with known malicious payloads
6. **IP Reputation**: Block requests from known malicious IPs (AWS Managed Rules)

**DDoS Protection**:
- AWS Shield Standard (automatic)
- CloudFront for static assets with caching
- API Gateway throttling: 1000 requests/second per hospital
- Lambda concurrency limits: 100 concurrent executions per hospital



## Integration Specifications

### HL7 v2.x Integration

**Supported Message Types**:

**ADT (Admission, Discharge, Transfer)**:
```
ADT^A01 - Patient Admission
ADT^A02 - Patient Transfer
ADT^A03 - Patient Discharge
ADT^A08 - Patient Information Update
ADT^A11 - Cancel Admission
ADT^A13 - Cancel Discharge
```

**ORM (Order Entry)**:
```
ORM^O01 - General Order Message
```

**ORU (Observation Result)**:
```
ORU^R01 - Unsolicited Observation Message (Lab Results, Vitals)
```

**MDM (Medical Document Management)**:
```
MDM^T02 - Document Status Change (Clinical Notes)
```

**HL7 Message Parsing**:
```typescript
interface HL7Parser {
  parse(message: string): HL7Message;
  validate(message: HL7Message): ValidationResult;
  extract(message: HL7Message): ClinicalFacts;
}

// Example ADT^A01 parsing
function parseADT_A01(message: HL7Message): PatientAdmission {
  const msh = message.getSegment('MSH'); // Message Header
  const evn = message.getSegment('EVN'); // Event Type
  const pid = message.getSegment('PID'); // Patient Identification
  const pv1 = message.getSegment('PV1'); // Patient Visit
  
  return {
    messageId: msh.getField(10),
    timestamp: evn.getField(2),
    patient: {
      mrn: pid.getField(3),
      name: pid.getField(5),
      dob: pid.getField(7),
      gender: pid.getField(8)
    },
    visit: {
      visitNumber: pv1.getField(19),
      admissionDate: pv1.getField(44),
      location: pv1.getField(3),
      attendingDoctor: pv1.getField(7)
    }
  };
}
```

**HL7 Acknowledgement (ACK)**:
```
MSH|^~\&|MEDWAR|HOSPITAL|EHR|HOSPITAL|20240212120000||ACK^A01|MSG00001|P|2.5
MSA|AA|MSG00001|Message accepted
```

**Error Handling**:
- **AA (Application Accept)**: Message processed successfully
- **AE (Application Error)**: Message rejected due to validation error
- **AR (Application Reject)**: Message rejected due to business rule violation


### FHIR R4 Integration

**Supported Resources**:

**Patient Resource**:
```json
{
  "resourceType": "Patient",
  "id": "example",
  "identifier": [
    {
      "system": "http://hospital.org/mrn",
      "value": "MRN123456"
    }
  ],
  "name": [
    {
      "use": "official",
      "family": "[REDACTED]",
      "given": ["[REDACTED]"]
    }
  ],
  "gender": "male",
  "birthDate": "1980-01-01"
}
```

**Observation Resource (Labs, Vitals)**:
```json
{
  "resourceType": "Observation",
  "id": "example",
  "status": "final",
  "category": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/observation-category",
          "code": "laboratory"
        }
      ]
    }
  ],
  "code": {
    "coding": [
      {
        "system": "http://loinc.org",
        "code": "2339-0",
        "display": "Glucose"
      }
    ]
  },
  "subject": {
    "reference": "Patient/example"
  },
  "effectiveDateTime": "2024-02-12T10:00:00Z",
  "valueQuantity": {
    "value": 120,
    "unit": "mg/dL",
    "system": "http://unitsofmeasure.org",
    "code": "mg/dL"
  }
}
```

**MedicationRequest Resource**:
```json
{
  "resourceType": "MedicationRequest",
  "id": "example",
  "status": "active",
  "intent": "order",
  "medicationCodeableConcept": {
    "coding": [
      {
        "system": "http://www.nlm.nih.gov/research/umls/rxnorm",
        "code": "197361",
        "display": "Metformin 500 MG Oral Tablet"
      }
    ]
  },
  "subject": {
    "reference": "Patient/example"
  },
  "dosageInstruction": [
    {
      "text": "Take 1 tablet twice daily with meals",
      "timing": {
        "repeat": {
          "frequency": 2,
          "period": 1,
          "periodUnit": "d"
        }
      },
      "doseAndRate": [
        {
          "doseQuantity": {
            "value": 1,
            "unit": "tablet"
          }
        }
      ]
    }
  ]
}
```


**Encounter Resource**:
```json
{
  "resourceType": "Encounter",
  "id": "example",
  "status": "in-progress",
  "class": {
    "system": "http://terminology.hl7.org/CodeSystem/v3-ActCode",
    "code": "IMP",
    "display": "inpatient encounter"
  },
  "subject": {
    "reference": "Patient/example"
  },
  "period": {
    "start": "2024-02-10T08:00:00Z"
  },
  "location": [
    {
      "location": {
        "display": "ICU Bed 5"
      }
    }
  ],
  "participant": [
    {
      "individual": {
        "reference": "Practitioner/doctor123",
        "display": "Dr. [REDACTED]"
      }
    }
  ]
}
```

**FHIR API Operations**:
```typescript
interface FHIRClient {
  // Read operations
  read(resourceType: string, id: string): Promise<FHIRResource>;
  search(resourceType: string, params: SearchParams): Promise<Bundle>;
  
  // Subscription for real-time updates
  subscribe(criteria: SubscriptionCriteria): Promise<Subscription>;
  
  // Batch operations
  batch(requests: BundleEntry[]): Promise<Bundle>;
}

// Example: Search for recent lab results
const labResults = await fhirClient.search('Observation', {
  patient: 'Patient/example',
  category: 'laboratory',
  date: 'gt2024-02-01'
});

// Example: Subscribe to new medication orders
const subscription = await fhirClient.subscribe({
  resourceType: 'MedicationRequest',
  criteria: 'MedicationRequest?patient=Patient/example&status=active',
  channel: {
    type: 'rest-hook',
    endpoint: 'https://api.medwar.io/fhir/webhook',
    payload: 'application/fhir+json'
  }
});
```

**FHIR Mapping to MEDWAR Timeline**:
```typescript
function mapFHIRToTimeline(resources: FHIRResource[]): TimelineEvent[] {
  return resources.map(resource => {
    switch (resource.resourceType) {
      case 'Observation':
        return {
          eventType: resource.category[0].coding[0].code === 'laboratory' ? 'LAB' : 'VITAL',
          timestamp: new Date(resource.effectiveDateTime),
          facts: extractObservationFacts(resource)
        };
      case 'MedicationRequest':
        return {
          eventType: 'MEDICATION',
          timestamp: new Date(resource.authoredOn),
          facts: extractMedicationFacts(resource)
        };
      case 'Encounter':
        return {
          eventType: resource.status === 'finished' ? 'DISCHARGE' : 'ADMISSION',
          timestamp: new Date(resource.period.start),
          facts: extractEncounterFacts(resource)
        };
      default:
        throw new Error(`Unsupported resource type: ${resource.resourceType}`);
    }
  });
}
```



## Deployment Architecture

### Infrastructure as Code

**AWS CDK Stack Structure**:
```typescript
// lib/medwar-stack.ts
export class MedwarStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props: MedwarStackProps) {
    super(scope, id, props);
    
    // VPC and Networking
    const vpc = new VpcConstruct(this, 'VPC', {
      cidr: '10.0.0.0/16',
      maxAzs: 2,
      natGateways: 1
    });
    
    // Data Layer
    const dataLayer = new DataLayerConstruct(this, 'DataLayer', {
      vpc,
      hospitalId: props.hospitalId
    });
    
    // AI Layer
    const aiLayer = new AILayerConstruct(this, 'AILayer', {
      bedrockRegion: 'us-east-1',
      modelIds: ['anthropic.claude-3-5-sonnet-20241022-v2:0']
    });
    
    // API Layer
    const apiLayer = new APILayerConstruct(this, 'APILayer', {
      vpc,
      dataLayer,
      aiLayer
    });
    
    // Frontend
    const frontend = new FrontendConstruct(this, 'Frontend', {
      apiEndpoint: apiLayer.apiEndpoint
    });
    
    // Monitoring
    const monitoring = new MonitoringConstruct(this, 'Monitoring', {
      apiLayer,
      dataLayer,
      alarmEmail: props.alarmEmail
    });
  }
}

// lib/constructs/data-layer-construct.ts
export class DataLayerConstruct extends Construct {
  public readonly patientsTable: dynamodb.Table;
  public readonly fracturesTable: dynamodb.Table;
  public readonly feedbackTable: dynamodb.Table;
  
  constructor(scope: Construct, id: string, props: DataLayerProps) {
    super(scope, id);
    
    // KMS Key for encryption
    const kmsKey = new kms.Key(this, 'DataKey', {
      enableKeyRotation: true,
      alias: `medwar-${props.hospitalId}-data`
    });
    
    // DynamoDB Tables
    this.patientsTable = new dynamodb.Table(this, 'Patients', {
      partitionKey: { name: 'patientId', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'hospitalId', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
      encryption: dynamodb.TableEncryption.CUSTOMER_MANAGED,
      encryptionKey: kmsKey,
      pointInTimeRecovery: true,
      timeToLiveAttribute: 'ttl'
    });
    
    this.fracturesTable = new dynamodb.Table(this, 'Fractures', {
      partitionKey: { name: 'fractureId', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'detectedAt', type: dynamodb.AttributeType.NUMBER },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
      encryption: dynamodb.TableEncryption.CUSTOMER_MANAGED,
      encryptionKey: kmsKey,
      pointInTimeRecovery: true,
      stream: dynamodb.StreamViewType.NEW_AND_OLD_IMAGES
    });
    
    // GSIs for fractures table
    this.fracturesTable.addGlobalSecondaryIndex({
      indexName: 'hospitalId-status-entropyScore-index',
      partitionKey: { name: 'hospitalId', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'statusEntropyScore', type: dynamodb.AttributeType.STRING },
      projectionType: dynamodb.ProjectionType.ALL
    });
    
    // S3 Buckets
    const uploadsBucket = new s3.Bucket(this, 'Uploads', {
      encryption: s3.BucketEncryption.KMS,
      encryptionKey: kmsKey,
      versioned: true,
      lifecycleRules: [
        {
          expiration: cdk.Duration.days(30),
          transitions: [
            {
              storageClass: s3.StorageClass.GLACIER,
              transitionAfter: cdk.Duration.days(7)
            }
          ]
        }
      ]
    });
  }
}
```


### CI/CD Pipeline

**Pipeline Stages**:

```yaml
# .github/workflows/deploy.yml
name: Deploy MEDWAR

on:
  push:
    branches: [main, staging, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run unit tests
        run: npm run test:unit
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          AWS_REGION: ap-south-1
          TEST_HOSPITAL_ID: test-hospital
      
      - name: Run property-based tests
        run: npm run test:pbt
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
  
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      
      - name: Run OWASP dependency check
        run: npm audit --audit-level=moderate
  
  build:
    needs: [test, security-scan]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Lambda functions
        run: npm run build:lambda
      
      - name: Build frontend
        run: npm run build:frontend
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-artifacts
          path: dist/
  
  deploy-dev:
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: development
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1
      
      - name: Deploy CDK stack
        run: |
          npm run cdk:deploy -- --context env=dev
      
      - name: Run smoke tests
        run: npm run test:smoke
        env:
          API_ENDPOINT: ${{ steps.deploy.outputs.api_endpoint }}
  
  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/staging'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to staging
        run: npm run cdk:deploy -- --context env=staging
      
      - name: Run E2E tests
        run: npm run test:e2e
  
  deploy-production:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to production (blue-green)
        run: npm run deploy:blue-green
      
      - name: Run canary tests
        run: npm run test:canary
      
      - name: Monitor metrics
        run: npm run monitor:deployment
        timeout-minutes: 30
      
      - name: Rollback on failure
        if: failure()
        run: npm run rollback
```


### Environment Strategy

**Environments**:

1. **Development (dev)**:
   - Purpose: Active development and testing
   - Data: Synthetic test data only
   - Deployment: Automatic on push to `develop` branch
   - Cost optimization: Minimal resources, no redundancy
   - Access: All developers

2. **Staging (staging)**:
   - Purpose: Pre-production testing and validation
   - Data: De-identified production-like data
   - Deployment: Automatic on push to `staging` branch
   - Infrastructure: Mirrors production (scaled down)
   - Access: QA team, product managers, select clinicians

3. **Production (prod)**:
   - Purpose: Live hospital deployments
   - Data: Real PHI/PII (encrypted)
   - Deployment: Manual approval required, blue-green strategy
   - Infrastructure: Full redundancy, multi-AZ
   - Access: Operations team only

**Environment Configuration**:
```typescript
// config/environments.ts
export const environments = {
  dev: {
    region: 'ap-south-1',
    bedrockRegion: 'us-east-1',
    logLevel: 'DEBUG',
    tokenBudget: 10000, // Lower for cost savings
    debateRounds: { min: 1, max: 2 },
    enableMockEHR: true,
    enableDebugUI: true
  },
  staging: {
    region: 'ap-south-1',
    bedrockRegion: 'us-east-1',
    logLevel: 'INFO',
    tokenBudget: 40000,
    debateRounds: { min: 2, max: 5 },
    enableMockEHR: false,
    enableDebugUI: true
  },
  prod: {
    region: 'ap-south-1',
    bedrockRegion: 'us-east-1',
    logLevel: 'WARN',
    tokenBudget: 40000,
    debateRounds: { min: 3, max: 5 },
    enableMockEHR: false,
    enableDebugUI: false,
    enableCanaryDeployment: true,
    canaryPercentage: 10
  }
};
```

### Blue-Green Deployment Strategy

**Process**:
1. **Deploy Green Environment**: New version deployed to green stack
2. **Smoke Tests**: Automated tests run against green environment
3. **Canary Traffic**: Route 10% of traffic to green environment
4. **Monitor Metrics**: Watch error rates, latency, fracture detection accuracy
5. **Gradual Rollout**: Increase traffic to green (10% → 25% → 50% → 100%)
6. **Rollback**: If metrics degrade, instant rollback to blue environment
7. **Cleanup**: After 24 hours of stable green, decommission blue

**Implementation**:
```typescript
// scripts/blue-green-deploy.ts
async function blueGreenDeploy() {
  // 1. Deploy green stack
  const greenStack = await deployStack('medwar-green');
  
  // 2. Run smoke tests
  const smokeTestsPassed = await runSmokeTests(greenStack.apiEndpoint);
  if (!smokeTestsPassed) {
    await rollback(greenStack);
    throw new Error('Smoke tests failed');
  }
  
  // 3. Configure weighted routing (10% canary)
  await updateRoute53Weights({
    blue: 90,
    green: 10
  });
  
  // 4. Monitor for 15 minutes
  const metricsHealthy = await monitorMetrics(greenStack, 15);
  if (!metricsHealthy) {
    await rollback(greenStack);
    throw new Error('Metrics degraded during canary');
  }
  
  // 5. Gradual rollout
  for (const weight of [25, 50, 75, 100]) {
    await updateRoute53Weights({
      blue: 100 - weight,
      green: weight
    });
    await sleep(10 * 60 * 1000); // 10 minutes
    
    const healthy = await monitorMetrics(greenStack, 10);
    if (!healthy) {
      await rollback(greenStack);
      throw new Error(`Metrics degraded at ${weight}% traffic`);
    }
  }
  
  // 6. Mark green as new blue
  await tagStack(greenStack, 'blue');
  
  // 7. Schedule old blue for cleanup
  await scheduleCleanup(oldBlueStack, 24 * 60 * 60 * 1000); // 24 hours
}
```



## Monitoring and Observability

### Metrics and KPIs

**System Health Metrics**:
```typescript
interface SystemMetrics {
  // Latency metrics
  latency: {
    p50: number; // Target: < 30s
    p95: number; // Target: < 60s
    p99: number; // Target: < 90s
  };
  
  // Throughput metrics
  throughput: {
    analysesPerHour: number;
    fracturesDetectedPerHour: number;
    apiRequestsPerSecond: number;
  };
  
  // Error metrics
  errors: {
    rate: number; // Target: < 1%
    bedrockTimeouts: number;
    bedrockThrottles: number;
    databaseErrors: number;
    validationErrors: number;
  };
  
  // Cost metrics
  costs: {
    bedrockCostPerAnalysis: number; // Target: < $0.20
    totalDailyCost: number;
    costPerHospital: number;
  };
  
  // Availability metrics
  availability: {
    uptime: number; // Target: 99.9%
    apiAvailability: number;
    databaseAvailability: number;
  };
}
```

**Clinical Quality Metrics**:
```typescript
interface ClinicalMetrics {
  // Detection accuracy
  accuracy: {
    truePositiveRate: number; // Target: > 80%
    falsePositiveRate: number; // Target: < 20%
    missRate: number; // Target: < 10%
    precision: number;
    recall: number;
    f1Score: number;
  };
  
  // User engagement
  engagement: {
    alertAcknowledgementRate: number; // Target: > 75%
    medianTimeToAcknowledge: number; // Target: < 15 minutes
    feedbackSubmissionRate: number; // Target: > 30%
    patientInstructionApprovalRate: number;
  };
  
  // Fracture distribution
  fractureDistribution: Map<FractureType, {
    count: number;
    avgEntropyScore: number;
    acknowledgementRate: number;
  }>;
  
  // Agent performance
  agentPerformance: Map<AgentType, {
    avgTokenUsage: number;
    avgLatency: number;
    claimAcceptanceRate: number; // % of claims that pass evidence verification
  }>;
}
```


### CloudWatch Dashboards

**Operational Dashboard**:
- **API Performance**: Request rate, latency (p50/p95/p99), error rate
- **Lambda Performance**: Invocation count, duration, errors, throttles, concurrent executions
- **DynamoDB Performance**: Read/write capacity, throttled requests, latency
- **Bedrock Performance**: Token usage, latency, throttles, costs
- **Cost Tracking**: Daily spend by service, cost per analysis, cost per hospital

**Clinical Dashboard**:
- **Fracture Detection**: Fractures detected per hour, by type, by entropy score
- **Alert Status**: Active alerts, acknowledged alerts, unacknowledged alerts > 15 minutes
- **User Engagement**: Acknowledgement rate, feedback rate, time to acknowledge
- **Agent Performance**: Claims per agent, evidence verification pass rate, token usage

**Security Dashboard**:
- **Authentication**: Login attempts, failed logins, MFA usage
- **Authorization**: Access denied events, unusual access patterns
- **Data Access**: PHI access events, bulk exports, after-hours access
- **Audit Log**: Recent audit events, anomaly detection alerts

### Alerting Strategy

**Critical Alerts** (PagerDuty, immediate response):
```typescript
const criticalAlerts = [
  {
    name: 'API Availability < 99%',
    metric: 'AWS/ApiGateway/5XXError',
    threshold: '> 1% over 5 minutes',
    action: 'Page on-call engineer'
  },
  {
    name: 'Database Unavailable',
    metric: 'AWS/DynamoDB/SystemErrors',
    threshold: '> 10 over 5 minutes',
    action: 'Page on-call engineer + database team'
  },
  {
    name: 'Bedrock Throttling',
    metric: 'AWS/Bedrock/ThrottledRequests',
    threshold: '> 50 over 10 minutes',
    action: 'Page on-call engineer, request quota increase'
  },
  {
    name: 'High-Risk Fracture Unacknowledged',
    metric: 'Custom/UnacknowledgedHighRiskFractures',
    threshold: '> 0 for 30 minutes',
    action: 'Escalate to hospital patient safety officer'
  }
];
```

**Warning Alerts** (Slack, review within 1 hour):
```typescript
const warningAlerts = [
  {
    name: 'Latency Degradation',
    metric: 'Custom/AnalysisLatencyP95',
    threshold: '> 90 seconds over 15 minutes',
    action: 'Notify engineering team'
  },
  {
    name: 'False Positive Rate Increase',
    metric: 'Custom/FalsePositiveRate',
    threshold: '> 30% over 24 hours',
    action: 'Notify ML team for calibration review'
  },
  {
    name: 'Cost Anomaly',
    metric: 'Custom/DailyCost',
    threshold: '> 150% of 7-day average',
    action: 'Notify finance + engineering'
  },
  {
    name: 'Evidence Verification Failure Rate',
    metric: 'Custom/EvidenceVerificationFailRate',
    threshold: '> 20% over 1 hour',
    action: 'Notify ML team, possible hallucination issue'
  }
];
```

**Info Alerts** (Email, daily digest):
```typescript
const infoAlerts = [
  {
    name: 'Daily Summary',
    schedule: '8:00 AM IST',
    content: [
      'Total analyses run',
      'Fractures detected by type',
      'Acknowledgement rate',
      'Feedback summary',
      'Cost summary'
    ]
  },
  {
    name: 'Weekly Performance Report',
    schedule: 'Monday 9:00 AM IST',
    content: [
      'Week-over-week metrics comparison',
      'Top performing hospitals',
      'Calibration model updates',
      'User engagement trends'
    ]
  }
];
```


### Distributed Tracing

**AWS X-Ray Integration**:
```typescript
// Trace analysis pipeline
import { captureAWS, captureHTTPsGlobal } from 'aws-xray-sdk';

// Instrument AWS SDK
const AWS = captureAWS(require('aws-sdk'));

// Instrument HTTP requests
captureHTTPsGlobal(require('https'));

// Custom subsegments for key operations
async function analyzePatient(patientId: string): Promise<OODAResult> {
  const segment = AWSXRay.getSegment();
  
  // Observe phase
  const observeSubsegment = segment.addNewSubsegment('Observe');
  const patientState = await observePatient(patientId);
  observeSubsegment.close();
  
  // Orient phase
  const orientSubsegment = segment.addNewSubsegment('Orient');
  const worldModels = await orientAgents(patientState);
  orientSubsegment.close();
  
  // Decide phase (multi-agent debate)
  const decideSubsegment = segment.addNewSubsegment('Decide');
  decideSubsegment.addAnnotation('debateRounds', 3);
  
  const round1Subsegment = decideSubsegment.addNewSubsegment('Round1-Parallel');
  const round1Claims = await executeRound1(worldModels);
  round1Subsegment.addMetadata('claimCount', round1Claims.length);
  round1Subsegment.close();
  
  const round2Subsegment = decideSubsegment.addNewSubsegment('Round2-Rebuttal');
  const round2Rebuttals = await executeRound2(round1Claims);
  round2Subsegment.close();
  
  const synthesisSubsegment = decideSubsegment.addNewSubsegment('Round3-Synthesis');
  const fractures = await executeSynthesis(round2Rebuttals);
  synthesisSubsegment.addMetadata('fractureCount', fractures.length);
  synthesisSubsegment.close();
  
  decideSubsegment.close();
  
  // Evidence verification
  const verifySubsegment = segment.addNewSubsegment('VerifyEvidence');
  const verifiedFractures = await verifyEvidence(fractures);
  verifySubsegment.addMetadata('verifiedCount', verifiedFractures.length);
  verifySubsegment.close();
  
  // Act phase
  const actSubsegment = segment.addNewSubsegment('Act');
  const countermeasures = await generateCountermeasures(verifiedFractures);
  actSubsegment.close();
  
  return {
    sessionId: generateSessionId(),
    patientId,
    fractures: verifiedFractures,
    countermeasures,
    executionTime: segment.duration,
    tokenUsage: calculateTokenUsage(segment)
  };
}
```

**Trace Analysis Queries**:
```sql
-- Find slow analyses
SELECT * FROM traces
WHERE service.name = 'medwar-api'
  AND duration > 90
  AND annotation.operation = 'analyzePatient'
ORDER BY duration DESC
LIMIT 100;

-- Find analyses with high token usage
SELECT * FROM traces
WHERE service.name = 'medwar-api'
  AND metadata.tokenUsage > 50000
ORDER BY metadata.tokenUsage DESC;

-- Find evidence verification failures
SELECT * FROM traces
WHERE service.name = 'medwar-api'
  AND subsegment.name = 'VerifyEvidence'
  AND metadata.verifiedCount < metadata.totalCount;
```



## Error Handling and Resilience

### Retry Strategies

**Exponential Backoff with Jitter**:
```typescript
interface RetryConfig {
  maxAttempts: number;
  baseDelay: number; // milliseconds
  maxDelay: number; // milliseconds
  jitterFactor: number; // 0.0 - 1.0
  retryableErrors: string[];
}

const defaultRetryConfig: RetryConfig = {
  maxAttempts: 3,
  baseDelay: 1000,
  maxDelay: 10000,
  jitterFactor: 0.3,
  retryableErrors: [
    'ThrottlingException',
    'ServiceUnavailable',
    'InternalServerError',
    'RequestTimeout'
  ]
};

async function retryWithBackoff<T>(
  operation: () => Promise<T>,
  config: RetryConfig = defaultRetryConfig
): Promise<T> {
  let lastError: Error;
  
  for (let attempt = 0; attempt < config.maxAttempts; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error;
      
      // Don't retry non-retryable errors
      if (!config.retryableErrors.includes(error.name)) {
        throw error;
      }
      
      // Don't retry on last attempt
      if (attempt === config.maxAttempts - 1) {
        break;
      }
      
      // Calculate delay with exponential backoff and jitter
      const exponentialDelay = Math.min(
        config.baseDelay * Math.pow(2, attempt),
        config.maxDelay
      );
      const jitter = exponentialDelay * config.jitterFactor * Math.random();
      const delay = exponentialDelay + jitter;
      
      console.log(`Retry attempt ${attempt + 1} after ${delay}ms`);
      await sleep(delay);
    }
  }
  
  throw lastError;
}
```

**Service-Specific Retry Configs**:
```typescript
const retryConfigs = {
  bedrock: {
    maxAttempts: 2, // Expensive, limit retries
    baseDelay: 2000,
    maxDelay: 8000,
    jitterFactor: 0.3,
    retryableErrors: ['ThrottlingException', 'ModelTimeoutException']
  },
  dynamodb: {
    maxAttempts: 3,
    baseDelay: 100,
    maxDelay: 1000,
    jitterFactor: 0.5,
    retryableErrors: ['ProvisionedThroughputExceededException', 'RequestLimitExceeded']
  },
  ehr: {
    maxAttempts: 5, // Critical for data ingestion
    baseDelay: 5000,
    maxDelay: 30000,
    jitterFactor: 0.3,
    retryableErrors: ['ConnectionTimeout', 'ServiceUnavailable']
  }
};
```


### Circuit Breaker Pattern

```typescript
enum CircuitState {
  CLOSED = 'CLOSED',     // Normal operation
  OPEN = 'OPEN',         // Failing, reject requests
  HALF_OPEN = 'HALF_OPEN' // Testing if service recovered
}

class CircuitBreaker {
  private state: CircuitState = CircuitState.CLOSED;
  private failureCount: number = 0;
  private successCount: number = 0;
  private lastFailureTime: number = 0;
  
  constructor(
    private readonly failureThreshold: number = 5,
    private readonly successThreshold: number = 2,
    private readonly timeout: number = 60000 // 1 minute
  ) {}
  
  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.OPEN) {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = CircuitState.HALF_OPEN;
        this.successCount = 0;
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }
    
    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private onSuccess(): void {
    this.failureCount = 0;
    
    if (this.state === CircuitState.HALF_OPEN) {
      this.successCount++;
      if (this.successCount >= this.successThreshold) {
        this.state = CircuitState.CLOSED;
      }
    }
  }
  
  private onFailure(): void {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    
    if (this.failureCount >= this.failureThreshold) {
      this.state = CircuitState.OPEN;
    }
  }
  
  getState(): CircuitState {
    return this.state;
  }
}

// Usage
const bedrockCircuitBreaker = new CircuitBreaker(5, 2, 60000);

async function invokeBedrockWithCircuitBreaker(prompt: string): Promise<string> {
  return bedrockCircuitBreaker.execute(async () => {
    return await bedrock.invoke(prompt);
  });
}
```

### Fallback Mechanisms

**Bedrock Fallback**:
```typescript
async function invokeBedrockWithFallback(
  prompt: string,
  config: BedrockConfig
): Promise<BedrockResponse> {
  try {
    // Primary: Claude 3.5 Sonnet
    return await invokeBedrock('anthropic.claude-3-5-sonnet-20241022-v2:0', prompt, config);
  } catch (error) {
    if (error.name === 'ThrottlingException') {
      // Fallback 1: Claude 3 Haiku (cheaper, faster)
      try {
        return await invokeBedrock('anthropic.claude-3-haiku-20240307-v1:0', prompt, {
          ...config,
          maxTokens: config.maxTokens / 2 // Reduce token budget
        });
      } catch (fallbackError) {
        // Fallback 2: Queue for later processing
        await queueForRetry(prompt, config);
        throw new Error('Bedrock unavailable, queued for retry');
      }
    }
    throw error;
  }
}
```

**Database Fallback**:
```typescript
async function readPatientStateWithFallback(patientId: string): Promise<PatientState> {
  try {
    // Primary: DynamoDB
    return await dynamodb.getItem('Patients', { patientId });
  } catch (error) {
    if (error.name === 'ProvisionedThroughputExceededException') {
      // Fallback: ElastiCache
      const cached = await elasticache.get(`patient:${patientId}`);
      if (cached) {
        return JSON.parse(cached);
      }
    }
    throw error;
  }
}
```

**EHR Integration Fallback**:
```typescript
async function ingestClinicalDataWithFallback(data: ClinicalData): Promise<void> {
  try {
    // Primary: Real-time processing
    await processImmediately(data);
  } catch (error) {
    // Fallback: Queue for batch processing
    await sqs.sendMessage({
      QueueUrl: process.env.BATCH_QUEUE_URL,
      MessageBody: JSON.stringify(data),
      DelaySeconds: 300 // 5 minutes
    });
    
    // Notify user
    await notifyUser({
      message: 'Data queued for processing',
      estimatedTime: '5-10 minutes'
    });
  }
}
```


### Graceful Degradation

**Degraded Mode Operation**:
```typescript
enum SystemMode {
  FULL = 'FULL',                 // All features available
  DEGRADED = 'DEGRADED',         // Core features only
  EMERGENCY = 'EMERGENCY'        // Critical features only
}

interface DegradedModeConfig {
  mode: SystemMode;
  disabledFeatures: string[];
  reducedCapabilities: Map<string, any>;
}

function getDegradedModeConfig(healthStatus: HealthStatus): DegradedModeConfig {
  if (healthStatus.bedrockAvailable === false) {
    return {
      mode: SystemMode.EMERGENCY,
      disabledFeatures: [
        'snapshot_analysis',
        'streaming_analysis',
        'patient_instructions_generation'
      ],
      reducedCapabilities: new Map([
        ['audit_mode', { useCache: true, skipAIAnalysis: true }]
      ])
    };
  }
  
  if (healthStatus.bedrockLatency > 120000) { // 2 minutes
    return {
      mode: SystemMode.DEGRADED,
      disabledFeatures: ['streaming_analysis'],
      reducedCapabilities: new Map([
        ['snapshot_analysis', { maxDebateRounds: 2, tokenBudget: 20000 }],
        ['patient_instructions', { useTemplates: true, skipCustomization: true }]
      ])
    };
  }
  
  return {
    mode: SystemMode.FULL,
    disabledFeatures: [],
    reducedCapabilities: new Map()
  };
}

// Apply degraded mode
async function analyzeWithDegradation(
  patientId: string,
  mode: DegradedModeConfig
): Promise<OODAResult> {
  if (mode.disabledFeatures.includes('snapshot_analysis')) {
    throw new Error('Analysis temporarily unavailable. Please try again later.');
  }
  
  const config = mode.reducedCapabilities.get('snapshot_analysis') || {};
  
  return await analyze(patientId, {
    maxDebateRounds: config.maxDebateRounds || 3,
    tokenBudget: config.tokenBudget || 40000,
    enableCaching: true
  });
}
```

### Dead Letter Queues

**Failed Message Handling**:
```typescript
// SQS Dead Letter Queue configuration
const analysisQueue = new sqs.Queue(this, 'AnalysisQueue', {
  visibilityTimeout: cdk.Duration.minutes(5),
  retentionPeriod: cdk.Duration.days(14),
  deadLetterQueue: {
    queue: new sqs.Queue(this, 'AnalysisDLQ', {
      retentionPeriod: cdk.Duration.days(14)
    }),
    maxReceiveCount: 3 // Move to DLQ after 3 failed attempts
  }
});

// DLQ processor
async function processDLQMessages(): Promise<void> {
  const messages = await sqs.receiveMessage({
    QueueUrl: process.env.DLQ_URL,
    MaxNumberOfMessages: 10
  });
  
  for (const message of messages.Messages || []) {
    const body = JSON.parse(message.Body);
    
    // Log for investigation
    await logDLQMessage({
      messageId: message.MessageId,
      body,
      receiveCount: message.Attributes.ApproximateReceiveCount,
      timestamp: new Date()
    });
    
    // Attempt manual recovery
    try {
      await manualRecovery(body);
      await sqs.deleteMessage({
        QueueUrl: process.env.DLQ_URL,
        ReceiptHandle: message.ReceiptHandle
      });
    } catch (error) {
      // Alert operations team
      await alertOps({
        severity: 'HIGH',
        message: `DLQ message recovery failed: ${message.MessageId}`,
        error
      });
    }
  }
}
```



## Performance Requirements

### Latency Targets

**Analysis Pipeline**:
| Operation | Target (p50) | Target (p95) | Target (p99) | Max Acceptable |
|-----------|--------------|--------------|--------------|----------------|
| Snapshot OODA (simple) | 20s | 40s | 60s | 90s |
| Snapshot OODA (complex) | 40s | 70s | 90s | 120s |
| Evidence Verification | 2s | 5s | 8s | 10s |
| Entropy Score Calculation | 1s | 2s | 3s | 5s |
| Patient Instructions Generation | 10s | 20s | 30s | 45s |
| Streaming OODA (per event) | 5s | 10s | 15s | 20s |

**API Endpoints**:
| Endpoint | Target (p50) | Target (p95) | Target (p99) |
|----------|--------------|--------------|--------------|
| GET /fractures | 200ms | 500ms | 1s |
| GET /fractures/{id} | 150ms | 300ms | 500ms |
| POST /fractures/{id}/acknowledge | 100ms | 200ms | 300ms |
| GET /dashboard/threat-map | 500ms | 1s | 2s |
| GET /dashboard/analytics | 1s | 3s | 5s |
| WebSocket message delivery | 100ms | 200ms | 500ms |

### Throughput Targets

**Per Hospital**:
- Snapshot analyses: 100-500 per day
- Streaming events: 1,000-10,000 per day
- API requests: 10,000-50,000 per day
- WebSocket connections: 50-200 concurrent

**System-Wide** (100 hospitals):
- Snapshot analyses: 10,000-50,000 per day
- Streaming events: 100,000-1,000,000 per day
- API requests: 1,000,000-5,000,000 per day
- WebSocket connections: 5,000-20,000 concurrent

### Scalability Limits

**Horizontal Scaling**:
```typescript
const scalingConfig = {
  lambda: {
    concurrentExecutions: {
      perHospital: 100,
      systemWide: 10000
    },
    reservedConcurrency: {
      criticalFunctions: 500 // Reserve for high-priority hospitals
    }
  },
  
  dynamodb: {
    readCapacity: {
      onDemand: true, // Auto-scaling
      maxRCU: 40000
    },
    writeCapacity: {
      onDemand: true,
      maxWCU: 40000
    }
  },
  
  apiGateway: {
    throttling: {
      rateLimit: 10000, // requests per second
      burstLimit: 20000
    }
  },
  
  bedrock: {
    quotas: {
      tokensPerMinute: 400000, // Request increase as needed
      requestsPerMinute: 1000
    }
  }
};
```

**Vertical Scaling**:
- Lambda memory: 512MB (light operations) to 10GB (heavy AI operations)
- ElastiCache: r6g.large (13.07 GB) to r6g.4xlarge (104.56 GB)
- RDS (if used): db.r6g.large to db.r6g.8xlarge


### Caching Strategy

**Multi-Layer Caching**:

**Layer 1 - Browser Cache** (PWA):
```typescript
// Service Worker caching strategy
const cacheStrategy = {
  static: {
    strategy: 'CacheFirst',
    cacheName: 'medwar-static-v1',
    resources: ['/index.html', '/app.js', '/styles.css', '/icons/*'],
    maxAge: 7 * 24 * 60 * 60 // 7 days
  },
  
  api: {
    strategy: 'NetworkFirst',
    cacheName: 'medwar-api-v1',
    endpoints: ['/api/v1/fractures', '/api/v1/dashboard/*'],
    maxAge: 5 * 60, // 5 minutes
    fallback: 'cache'
  },
  
  images: {
    strategy: 'CacheFirst',
    cacheName: 'medwar-images-v1',
    maxEntries: 100,
    maxAge: 30 * 24 * 60 * 60 // 30 days
  }
};
```

**Layer 2 - CloudFront CDN**:
```typescript
const cloudFrontConfig = {
  defaultCacheBehavior: {
    viewerProtocolPolicy: 'redirect-to-https',
    allowedMethods: ['GET', 'HEAD', 'OPTIONS'],
    cachedMethods: ['GET', 'HEAD'],
    compress: true,
    cachePolicyId: 'CachingOptimized',
    originRequestPolicyId: 'AllViewer'
  },
  
  cacheBehaviors: [
    {
      pathPattern: '/api/*',
      cachePolicyId: 'CachingDisabled' // Don't cache API responses
    },
    {
      pathPattern: '/static/*',
      cachePolicyId: 'CachingOptimized',
      ttl: 86400 // 24 hours
    }
  ]
};
```

**Layer 3 - ElastiCache (Redis)**:
```typescript
interface CacheConfig {
  patientState: {
    ttl: 300, // 5 minutes
    keyPattern: 'patient:{patientId}:state'
  };
  
  fractureList: {
    ttl: 60, // 1 minute
    keyPattern: 'hospital:{hospitalId}:fractures'
  };
  
  agentPrompts: {
    ttl: 3600, // 1 hour
    keyPattern: 'agent:{agentType}:prompt:{version}'
  };
  
  hospitalConfig: {
    ttl: 1800, // 30 minutes
    keyPattern: 'hospital:{hospitalId}:config'
  };
  
  calibrationModel: {
    ttl: 86400, // 24 hours
    keyPattern: 'hospital:{hospitalId}:calibration:{modelType}'
  };
}

// Cache-aside pattern
async function getPatientState(patientId: string): Promise<PatientState> {
  const cacheKey = `patient:${patientId}:state`;
  
  // Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Cache miss, fetch from DynamoDB
  const patientState = await dynamodb.getItem('Patients', { patientId });
  
  // Update cache
  await redis.setex(cacheKey, 300, JSON.stringify(patientState));
  
  return patientState;
}
```

**Layer 4 - Bedrock Prompt Caching**:
```typescript
// Use Bedrock's prompt caching for agent system prompts
const bedrockConfig = {
  modelId: 'anthropic.claude-3-5-sonnet-20241022-v2:0',
  inferenceConfig: {
    temperature: 0.3,
    maxTokens: 2048
  },
  promptCaching: {
    enabled: true,
    cacheSystemPrompt: true, // Cache agent persona and instructions
    ttl: 300 // 5 minutes
  }
};

// System prompt is cached, only user prompt changes
const response = await bedrock.converse({
  modelId: bedrockConfig.modelId,
  messages: [
    {
      role: 'user',
      content: [
        {
          type: 'text',
          text: systemPrompt, // Cached
          cacheControl: { type: 'ephemeral' }
        },
        {
          type: 'text',
          text: userPrompt // Not cached, changes per request
        }
      ]
    }
  ]
});
```



## Testing Strategy

### Unit Testing

**Coverage Targets**:
- Overall code coverage: > 80%
- Critical paths (OODA loop, entropy scoring): > 95%
- Utility functions: > 90%
- UI components: > 70%

**Testing Framework**:
```typescript
// Jest configuration
export default {
  preset: 'ts-jest',
  testEnvironment: 'node',
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/core/': {
      branches: 95,
      functions: 95,
      lines: 95,
      statements: 95
    }
  },
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.test.ts',
    '!src/**/*.spec.ts'
  ]
};
```

**Example Unit Tests**:
```typescript
// tests/unit/entropy-score-engine.test.ts
describe('EntropyScoreEngine', () => {
  let engine: EntropyScoreEngine;
  
  beforeEach(() => {
    engine = new EntropyScoreEngine();
  });
  
  describe('computeLayer1Heuristic', () => {
    it('should assign high score to contraindication omission', () => {
      const fracture: SemanticFracture = {
        fractureType: 'CONTRAINDICATION_OMISSION',
        // ... other fields
      };
      
      const score = engine.computeLayer1Heuristic(fracture);
      expect(score).toBeGreaterThan(0.8);
    });
    
    it('should apply ICU risk factor bonus', () => {
      const fracture: SemanticFracture = {
        fractureType: 'MEDICATION_RECONCILIATION',
        // ... other fields
      };
      
      const context: ScoringContext = {
        patientState: {
          riskFactors: { isICU: true }
        }
      };
      
      const scoreWithICU = engine.computeLayer1Heuristic(fracture, context);
      const scoreWithoutICU = engine.computeLayer1Heuristic(fracture, {
        ...context,
        patientState: { riskFactors: { isICU: false } }
      });
      
      expect(scoreWithICU).toBeGreaterThan(scoreWithoutICU);
    });
  });
  
  describe('computeEntropyScore', () => {
    it('should clamp final score between 0 and 1', () => {
      const fracture: SemanticFracture = {
        // ... fields that might produce score > 1
      };
      
      const score = await engine.computeEntropyScore(fracture, context);
      expect(score).toBeGreaterThanOrEqual(0);
      expect(score).toBeLessThanOrEqual(1);
    });
  });
});
```


### Integration Testing

**Test Scenarios**:
```typescript
// tests/integration/ooda-pipeline.test.ts
describe('OODA Pipeline Integration', () => {
  let testHospitalId: string;
  let testPatientId: string;
  
  beforeAll(async () => {
    // Setup test hospital and patient
    testHospitalId = await createTestHospital();
    testPatientId = await createTestPatient(testHospitalId);
  });
  
  afterAll(async () => {
    // Cleanup
    await deleteTestData(testHospitalId);
  });
  
  it('should complete full OODA loop for discharge scenario', async () => {
    // Arrange
    const dischargeNote = loadTestData('discharge-note-with-conflicts.txt');
    
    // Act
    const result = await triggerSnapshotOODA({
      hospitalId: testHospitalId,
      patientId: testPatientId,
      clinicalMoment: {
        type: 'DISCHARGE',
        timestamp: new Date(),
        triggerData: { note: dischargeNote }
      }
    });
    
    // Assert
    expect(result.status).toBe('COMPLETED');
    expect(result.fractures.length).toBeGreaterThan(0);
    expect(result.fractures[0].evidence.length).toBeGreaterThan(0);
    expect(result.countermeasures.length).toBeGreaterThan(0);
    expect(result.executionTime).toBeLessThan(90000); // < 90 seconds
  });
  
  it('should verify all evidence against source data', async () => {
    const result = await triggerSnapshotOODA({
      hospitalId: testHospitalId,
      patientId: testPatientId,
      clinicalMoment: {
        type: 'SHIFT_HANDOVER',
        timestamp: new Date(),
        triggerData: { note: 'Patient on Warfarin and Aspirin...' }
      }
    });
    
    // All fractures should have verified evidence
    for (const fracture of result.fractures) {
      expect(fracture.evidence.length).toBeGreaterThan(0);
      for (const evidence of fracture.evidence) {
        expect(evidence.sourceReference).toBeDefined();
        expect(evidence.excerpt).toBeDefined();
      }
    }
  });
  
  it('should handle Bedrock timeout gracefully', async () => {
    // Mock Bedrock to timeout
    jest.spyOn(bedrock, 'invoke').mockImplementation(() => {
      return new Promise((_, reject) => {
        setTimeout(() => reject(new Error('ModelTimeoutException')), 1000);
      });
    });
    
    const result = await triggerSnapshotOODA({
      hospitalId: testHospitalId,
      patientId: testPatientId,
      clinicalMoment: {
        type: 'DISCHARGE',
        timestamp: new Date(),
        triggerData: { note: 'Simple discharge note' }
      }
    });
    
    // Should fallback or queue for retry
    expect(result.status).toMatch(/QUEUED|PARTIAL/);
  });
});
```


### Property-Based Testing

**Correctness Properties**:

```typescript
// tests/property/entropy-score.property.test.ts
import fc from 'fast-check';

describe('Entropy Score Properties', () => {
  it('Property: Entropy score is always between 0 and 1', () => {
    fc.assert(
      fc.property(
        fc.record({
          fractureType: fc.constantFrom(...Object.values(FractureType)),
          evidence: fc.array(fc.record({
            excerpt: fc.string(),
            sourceReference: fc.record({
              sourceId: fc.uuid(),
              location: fc.string()
            })
          }), { minLength: 1 }),
          agentConsensus: fc.record({
            supportingAgents: fc.array(fc.constantFrom(...Object.values(AgentType))),
            opposingAgents: fc.array(fc.constantFrom(...Object.values(AgentType))),
            consensusScore: fc.float({ min: 0, max: 1 })
          })
        }),
        async (fracture) => {
          const engine = new EntropyScoreEngine();
          const score = await engine.computeEntropyScore(fracture, mockContext);
          
          expect(score).toBeGreaterThanOrEqual(0);
          expect(score).toBeLessThanOrEqual(1);
        }
      ),
      { numRuns: 1000 }
    );
  });
  
  it('Property: Higher agent disagreement increases entropy score', () => {
    fc.assert(
      fc.property(
        fc.record({
          fractureType: fc.constantFrom(...Object.values(FractureType)),
          evidence: fc.array(fc.anything(), { minLength: 1 })
        }),
        fc.integer({ min: 0, max: 6 }), // supporting agents count
        fc.integer({ min: 0, max: 6 }), // opposing agents count
        async (baseFracture, supportingCount, opposingCount) => {
          fc.pre(supportingCount + opposingCount <= 6); // Max 6 agents
          fc.pre(supportingCount > 0); // At least one supporting agent
          
          const engine = new EntropyScoreEngine();
          
          const lowDisagreement = {
            ...baseFracture,
            agentConsensus: {
              supportingAgents: Array(supportingCount).fill('NURSE'),
              opposingAgents: Array(Math.min(opposingCount, 1)).fill('ADVERSARY'),
              consensusScore: 0.9
            }
          };
          
          const highDisagreement = {
            ...baseFracture,
            agentConsensus: {
              supportingAgents: Array(supportingCount).fill('NURSE'),
              opposingAgents: Array(Math.max(opposingCount, 3)).fill('ADVERSARY'),
              consensusScore: 0.3
            }
          };
          
          const scoreLow = await engine.computeEntropyScore(lowDisagreement, mockContext);
          const scoreHigh = await engine.computeEntropyScore(highDisagreement, mockContext);
          
          expect(scoreHigh).toBeGreaterThanOrEqual(scoreLow);
        }
      ),
      { numRuns: 500 }
    );
  });
});
```


```typescript
// tests/property/evidence-verification.property.test.ts
describe('Evidence Verification Properties', () => {
  it('Property: All verified fractures have evidence in source', () => {
    fc.assert(
      fc.property(
        fc.string({ minLength: 100, maxLength: 1000 }), // source text
        fc.array(fc.record({
          excerpt: fc.string({ minLength: 10, maxLength: 50 }),
          sourceReference: fc.anything()
        }), { minLength: 1, maxLength: 5 }),
        async (sourceText, evidenceList) => {
          // Inject some evidence excerpts into source text
          const modifiedSource = sourceText + ' ' + evidenceList.map(e => e.excerpt).join(' ');
          
          const verifier = new EvidenceVerifier();
          const fracture: SemanticFracture = {
            fractureId: fc.sample(fc.uuid(), 1)[0],
            evidence: evidenceList,
            // ... other fields
          };
          
          const result = await verifier.verifyFracture(fracture, {
            sourceId: 'test',
            sourceType: 'NOTE',
            content: modifiedSource,
            facts: []
          });
          
          if (result.verified) {
            // All evidence should be found in source
            expect(result.failedEvidence).toHaveLength(0);
          }
        }
      ),
      { numRuns: 500 }
    );
  });
  
  it('Property: Fuzzy match is symmetric', () => {
    fc.assert(
      fc.property(
        fc.string({ minLength: 10, maxLength: 100 }),
        fc.string({ minLength: 10, maxLength: 100 }),
        (text1, text2) => {
          const verifier = new EvidenceVerifier();
          const similarity1 = verifier.fuzzyMatch(text1, text2, 0.85);
          const similarity2 = verifier.fuzzyMatch(text2, text1, 0.85);
          
          expect(Math.abs(similarity1 - similarity2)).toBeLessThan(0.01);
        }
      ),
      { numRuns: 1000 }
    );
  });
});

// tests/property/patient-instructions.property.test.ts
describe('Patient Instructions Properties', () => {
  it('Property: Readability grade never exceeds target', () => {
    fc.assert(
      fc.property(
        fc.record({
          medications: fc.array(fc.string(), { minLength: 1, maxLength: 5 }),
          warnings: fc.array(fc.string(), { minLength: 1, maxLength: 3 }),
          followUp: fc.string()
        }),
        async (instructionData) => {
          const system = new PatientComprehensionSystem();
          const instructions = await system.generatePatientInstructions(
            mockPatientState,
            'ENGLISH'
          );
          
          expect(instructions.readabilityGrade).toBeLessThanOrEqual(6);
        }
      ),
      { numRuns: 200 }
    );
  });
  
  it('Property: Medication names are never translated', () => {
    fc.assert(
      fc.property(
        fc.array(fc.record({
          name: fc.constantFrom('Metformin', 'Aspirin', 'Lisinopril'),
          dosage: fc.string(),
          frequency: fc.string()
        }), { minLength: 1, maxLength: 5 }),
        fc.constantFrom('HINDI', 'TAMIL', 'MARATHI'),
        async (medications, language) => {
          const system = new PatientComprehensionSystem();
          const patientState = {
            ...mockPatientState,
            currentStatus: {
              ...mockPatientState.currentStatus,
              currentMedications: medications
            }
          };
          
          const instructions = await system.generatePatientInstructions(
            patientState,
            language
          );
          
          // Check that medication names appear unchanged
          for (const med of medications) {
            const found = instructions.sections
              .find(s => s.sectionType === 'MEDICATIONS')
              ?.content.some(c => c.includes(med.name));
            expect(found).toBe(true);
          }
        }
      ),
      { numRuns: 100 }
    );
  });
});
```


### End-to-End Testing

**E2E Test Scenarios**:
```typescript
// tests/e2e/discharge-workflow.e2e.test.ts
import { test, expect } from '@playwright/test';

test.describe('Discharge Workflow', () => {
  test('should detect fractures in discharge summary and generate patient instructions', async ({ page }) => {
    // Login as doctor
    await page.goto('/login');
    await page.fill('[data-testid="username"]', 'dr.test@hospital.com');
    await page.fill('[data-testid="password"]', 'test-password');
    await page.click('[data-testid="login-button"]');
    
    // Navigate to patient
    await page.goto('/patients/TEST-PATIENT-001');
    
    // Upload discharge summary
    await page.click('[data-testid="upload-discharge-summary"]');
    await page.setInputFiles('[data-testid="file-input"]', 'tests/fixtures/discharge-summary-with-conflicts.pdf');
    await page.click('[data-testid="analyze-button"]');
    
    // Wait for analysis to complete
    await page.waitForSelector('[data-testid="analysis-complete"]', { timeout: 120000 });
    
    // Verify fractures detected
    const fractureCount = await page.locator('[data-testid="fracture-card"]').count();
    expect(fractureCount).toBeGreaterThan(0);
    
    // Check high-risk fracture
    const highRiskFracture = page.locator('[data-testid="fracture-card"][data-risk="high"]').first();
    await expect(highRiskFracture).toBeVisible();
    
    // View fracture details
    await highRiskFracture.click();
    await expect(page.locator('[data-testid="evidence-section"]')).toBeVisible();
    await expect(page.locator('[data-testid="countermeasure-section"]')).toBeVisible();
    
    // Generate patient instructions
    await page.click('[data-testid="generate-instructions-button"]');
    await page.selectOption('[data-testid="language-selector"]', 'HINDI');
    await page.click('[data-testid="generate-button"]');
    
    // Wait for instructions
    await page.waitForSelector('[data-testid="instructions-preview"]', { timeout: 60000 });
    
    // Verify readability
    const readabilityGrade = await page.locator('[data-testid="readability-grade"]').textContent();
    expect(parseInt(readabilityGrade)).toBeLessThanOrEqual(6);
    
    // Approve and download
    await page.click('[data-testid="approve-button"]');
    const downloadPromise = page.waitForEvent('download');
    await page.click('[data-testid="download-pdf-button"]');
    const download = await downloadPromise;
    expect(download.suggestedFilename()).toMatch(/patient-instructions.*\.pdf/);
  });
});
```

### Load Testing

**Load Test Configuration**:
```typescript
// tests/load/analysis-load.test.ts
import { check } from 'k6';
import http from 'k6/http';

export const options = {
  stages: [
    { duration: '2m', target: 10 },   // Ramp up to 10 users
    { duration: '5m', target: 10 },   // Stay at 10 users
    { duration: '2m', target: 50 },   // Ramp up to 50 users
    { duration: '5m', target: 50 },   // Stay at 50 users
    { duration: '2m', target: 100 },  // Ramp up to 100 users
    { duration: '5m', target: 100 },  // Stay at 100 users
    { duration: '5m', target: 0 },    // Ramp down to 0 users
  ],
  thresholds: {
    http_req_duration: ['p(95)<90000'], // 95% of requests should be below 90s
    http_req_failed: ['rate<0.05'],     // Error rate should be below 5%
  },
};

export default function () {
  const payload = JSON.stringify({
    hospitalId: 'test-hospital',
    patientId: `patient-${__VU}-${__ITER}`,
    clinicalMoment: {
      type: 'DISCHARGE',
      timestamp: new Date().toISOString(),
      triggerData: {
        note: generateRandomDischargeNote()
      }
    }
  });
  
  const params = {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${__ENV.API_TOKEN}`
    },
  };
  
  const response = http.post(`${__ENV.API_URL}/api/v1/analysis/snapshot`, payload, params);
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 90s': (r) => r.timings.duration < 90000,
    'has sessionId': (r) => JSON.parse(r.body).sessionId !== undefined,
  });
}

function generateRandomDischargeNote(): string {
  const templates = [
    'Patient discharged on Metformin 500mg BID and Aspirin 75mg OD...',
    'Post-op patient, continue antibiotics for 7 days...',
    'Diabetic patient, follow up with PCP in 2 weeks...'
  ];
  return templates[Math.floor(Math.random() * templates.length)];
}
```



## Migration and Rollback Strategy

### Database Migrations

**Migration Framework**:
```typescript
// migrations/001-initial-schema.ts
export class InitialSchemaMigration implements Migration {
  version = '001';
  description = 'Create initial DynamoDB tables';
  
  async up(): Promise<void> {
    // Create Patients table
    await dynamodb.createTable({
      TableName: 'Patients',
      KeySchema: [
        { AttributeName: 'patientId', KeyType: 'HASH' },
        { AttributeName: 'hospitalId', KeyType: 'RANGE' }
      ],
      AttributeDefinitions: [
        { AttributeName: 'patientId', AttributeType: 'S' },
        { AttributeName: 'hospitalId', AttributeType: 'S' },
        { AttributeName: 'unitId', AttributeType: 'S' }
      ],
      GlobalSecondaryIndexes: [
        {
          IndexName: 'hospitalId-unitId-index',
          KeySchema: [
            { AttributeName: 'hospitalId', KeyType: 'HASH' },
            { AttributeName: 'unitId', KeyType: 'RANGE' }
          ],
          Projection: { ProjectionType: 'ALL' }
        }
      ],
      BillingMode: 'PAY_PER_REQUEST'
    });
    
    // Create other tables...
  }
  
  async down(): Promise<void> {
    await dynamodb.deleteTable({ TableName: 'Patients' });
    // Delete other tables...
  }
}

// migrations/002-add-fracture-priority-field.ts
export class AddFracturePriorityMigration implements Migration {
  version = '002';
  description = 'Add priority field to SemanticFractures table';
  
  async up(): Promise<void> {
    // DynamoDB is schemaless, but we need to update existing records
    const fractures = await scanAllFractures();
    
    for (const fracture of fractures) {
      const priority = calculatePriority(fracture.entropyScore);
      await dynamodb.updateItem({
        TableName: 'SemanticFractures',
        Key: {
          fractureId: fracture.fractureId,
          detectedAt: fracture.detectedAt
        },
        UpdateExpression: 'SET priority = :priority',
        ExpressionAttributeValues: {
          ':priority': priority
        }
      });
    }
  }
  
  async down(): Promise<void> {
    // Remove priority field from all records
    const fractures = await scanAllFractures();
    
    for (const fracture of fractures) {
      await dynamodb.updateItem({
        TableName: 'SemanticFractures',
        Key: {
          fractureId: fracture.fractureId,
          detectedAt: fracture.detectedAt
        },
        UpdateExpression: 'REMOVE priority'
      });
    }
  }
}
```

**Migration Execution**:
```typescript
// scripts/run-migrations.ts
async function runMigrations(targetVersion?: string): Promise<void> {
  const migrations = loadMigrations();
  const currentVersion = await getCurrentVersion();
  
  console.log(`Current version: ${currentVersion}`);
  console.log(`Target version: ${targetVersion || 'latest'}`);
  
  const pendingMigrations = migrations.filter(m => 
    m.version > currentVersion && 
    (!targetVersion || m.version <= targetVersion)
  );
  
  if (pendingMigrations.length === 0) {
    console.log('No pending migrations');
    return;
  }
  
  console.log(`Running ${pendingMigrations.length} migrations...`);
  
  for (const migration of pendingMigrations) {
    console.log(`Applying migration ${migration.version}: ${migration.description}`);
    
    try {
      await migration.up();
      await recordMigration(migration.version);
      console.log(`✓ Migration ${migration.version} completed`);
    } catch (error) {
      console.error(`✗ Migration ${migration.version} failed:`, error);
      throw error;
    }
  }
  
  console.log('All migrations completed successfully');
}
```


### Zero-Downtime Deployment

**Deployment Process**:
```typescript
// scripts/zero-downtime-deploy.ts
async function zeroDowntimeDeploy(): Promise<void> {
  console.log('Starting zero-downtime deployment...');
  
  // 1. Run database migrations (if any)
  console.log('Step 1: Running database migrations...');
  await runMigrations();
  
  // 2. Deploy new Lambda versions (but don't activate yet)
  console.log('Step 2: Deploying new Lambda versions...');
  const newVersions = await deployLambdaVersions();
  
  // 3. Create new Lambda aliases pointing to new versions
  console.log('Step 3: Creating Lambda aliases...');
  await createLambdaAliases(newVersions, 'green');
  
  // 4. Update API Gateway to use weighted routing (10% to new version)
  console.log('Step 4: Routing 10% traffic to new version...');
  await updateAPIGatewayRouting({
    blue: 90,
    green: 10
  });
  
  // 5. Monitor metrics for 5 minutes
  console.log('Step 5: Monitoring metrics...');
  const metricsHealthy = await monitorMetrics(5);
  
  if (!metricsHealthy) {
    console.error('Metrics unhealthy, rolling back...');
    await rollback();
    throw new Error('Deployment failed: metrics degraded');
  }
  
  // 6. Gradually increase traffic to new version
  for (const percentage of [25, 50, 75, 100]) {
    console.log(`Step 6.${percentage}: Routing ${percentage}% traffic to new version...`);
    await updateAPIGatewayRouting({
      blue: 100 - percentage,
      green: percentage
    });
    
    await sleep(5 * 60 * 1000); // Wait 5 minutes
    
    const healthy = await monitorMetrics(5);
    if (!healthy) {
      console.error(`Metrics unhealthy at ${percentage}%, rolling back...`);
      await rollback();
      throw new Error(`Deployment failed at ${percentage}% traffic`);
    }
  }
  
  // 7. Mark new version as stable
  console.log('Step 7: Marking new version as stable...');
  await updateLambdaAliases(newVersions, 'blue');
  
  // 8. Schedule old version cleanup
  console.log('Step 8: Scheduling old version cleanup...');
  await scheduleCleanup(24 * 60 * 60 * 1000); // 24 hours
  
  console.log('Zero-downtime deployment completed successfully!');
}
```

### Rollback Procedures

**Automated Rollback**:
```typescript
// scripts/rollback.ts
async function rollback(targetVersion?: string): Promise<void> {
  console.log('Starting rollback...');
  
  // 1. Get current and target versions
  const currentVersion = await getCurrentDeployedVersion();
  const rollbackVersion = targetVersion || await getPreviousVersion();
  
  console.log(`Rolling back from ${currentVersion} to ${rollbackVersion}`);
  
  // 2. Immediately switch all traffic to previous version
  console.log('Step 1: Switching all traffic to previous version...');
  await updateAPIGatewayRouting({
    [rollbackVersion]: 100,
    [currentVersion]: 0
  });
  
  // 3. Update Lambda aliases
  console.log('Step 2: Updating Lambda aliases...');
  await updateLambdaAliasesToVersion(rollbackVersion);
  
  // 4. Rollback database migrations (if needed)
  const currentDBVersion = await getCurrentDBVersion();
  const targetDBVersion = await getDBVersionForAppVersion(rollbackVersion);
  
  if (currentDBVersion !== targetDBVersion) {
    console.log('Step 3: Rolling back database migrations...');
    await rollbackMigrations(targetDBVersion);
  }
  
  // 5. Verify rollback success
  console.log('Step 4: Verifying rollback...');
  const healthy = await monitorMetrics(5);
  
  if (!healthy) {
    console.error('Rollback verification failed!');
    await alertOps({
      severity: 'CRITICAL',
      message: 'Rollback failed, manual intervention required'
    });
    throw new Error('Rollback verification failed');
  }
  
  console.log('Rollback completed successfully');
  
  // 6. Notify team
  await notifyTeam({
    message: `Rollback from ${currentVersion} to ${rollbackVersion} completed`,
    channel: 'deployments'
  });
}
```

**Manual Rollback Checklist**:
```markdown
# Manual Rollback Procedure

## Pre-Rollback
- [ ] Identify target version to rollback to
- [ ] Verify target version is stable and tested
- [ ] Notify team in #incidents channel
- [ ] Create incident ticket

## Rollback Steps
1. [ ] Run automated rollback script: `npm run rollback -- --version=<target>`
2. [ ] Monitor CloudWatch dashboard for 10 minutes
3. [ ] Verify API health: `npm run health-check`
4. [ ] Check error rates in CloudWatch
5. [ ] Verify database integrity: `npm run db:verify`
6. [ ] Test critical user flows manually

## Post-Rollback
- [ ] Document root cause in incident ticket
- [ ] Schedule post-mortem meeting
- [ ] Update runbook with lessons learned
- [ ] Plan fix for next deployment
```



## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system-essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### System-Level Properties

**Property 1: Entropy Score Bounds**
*For any* semantic fracture detected by the system, the computed entropy score must be between 0.0 and 1.0 inclusive.
**Validates: Requirements 8.1, 8.2, 8.3, 8.4, 8.5**

**Property 2: Evidence Grounding**
*For any* fracture displayed to clinicians, every fracture must have at least one piece of verified evidence that can be traced back to source data.
**Validates: Requirements 9.1, 9.2, 9.3, 9.4, 9.5, 9.6, 9.7**

**Property 3: Agent Consensus Consistency**
*For any* fracture with high agent agreement (>80% supporting), the consensus score must be greater than 0.7; for fractures with low agreement (<30% supporting), the consensus score must be less than 0.4.
**Validates: Requirements 6.1, 6.2, 6.3, 6.4, 6.5, 6.6, 6.7**

**Property 4: Temporal Ordering**
*For any* patient timeline, all events must be chronologically ordered by timestamp.
**Validates: Requirements 4.1, 4.2, 4.3, 4.4**

**Property 5: Data Isolation**
*For any* two different hospitals, no fracture from hospital A should be accessible when querying fractures for hospital B.
**Validates: Requirements 23.1, 23.2, 23.3, 23.4, 23.5, 24.1, 24.2, 24.3**

### Component-Level Properties

**Property 6: Evidence Verification Symmetry**
*For any* two text strings, the fuzzy match similarity score should be symmetric within a tolerance of 0.01.
**Validates: Requirements 9.2, 9.3**

**Property 7: Readability Constraint**
*For any* patient instructions generated in any supported language, the readability grade must be 6 or lower.
**Validates: Requirements 11.11, 11.12, 11.13**

**Property 8: Medication Name Preservation**
*For any* patient instructions translated to any supported language, all medication names must appear unchanged in the translated version.
**Validates: Requirements 11.1, 11.2, 11.3, 11.4, 11.5, 11.6, 11.7, 11.8, 11.9, 11.10, 36.7**

**Property 9: Token Budget Enforcement**
*For any* OODA analysis session, the total token usage across all agent invocations must not exceed the configured token budget.
**Validates: Requirements 6.8, 26.6**

**Property 10: Acknowledgement Idempotency**
*For any* fracture and user, acknowledging the same fracture multiple times should result in the same final state as acknowledging it once.
**Validates: Requirements 30.1, 30.2, 30.11**

### Safety Properties

**Property 11: No Data Loss on Failure**
*For any* data write operation, if the operation fails, the data must either be successfully written to the database or queued in a backup queue for retry.
**Validates: Requirements 35.1, 35.2, 35.3, 35.4, 35.5, 35.6**

**Property 12: Audit Trail Completeness**
*For any* PHI access event, there must exist a corresponding audit log entry with matching resource ID, user ID, and timestamp.
**Validates: Requirements 38.1, 38.2, 38.3, 38.4, 38.5, 38.6, 38.7, 38.12**

**Property 13: Encryption Invariant**
*For any* stored data containing PHI, the data must be encrypted at rest using AWS KMS.
**Validates: Requirements 24.2, 24.3, 24.4**

**Property 14: Authorization Enforcement**
*For any* user attempting to access a resource, access should only be granted if the resource's hospital ID is in the user's authorized hospitals list and the resource scope is within the user's scope.
**Validates: Requirements 23.9, 23.10, 23.11**

### Liveness Properties

**Property 15: Analysis Completion**
*For any* queued analysis, the analysis must eventually reach either COMPLETED or FAILED status within a reasonable time bound.
**Validates: Requirements 27.1, 27.2**

**Property 16: Alert Delivery**
*For any* high-risk fracture (entropy score > 0.7), a notification must eventually be delivered to the assigned clinical staff.
**Validates: Requirements 30.1, 30.2, 30.3, 30.4, 30.5, 30.6**

**Property 17: Feedback Processing**
*For any* submitted feedback, the feedback must eventually be processed and incorporated into a calibration update.
**Validates: Requirements 15.1, 15.2, 15.3, 15.4, 15.5, 15.6**

### Performance Properties

**Property 18: Latency Bounds**
*For any* 100 snapshot OODA analyses, at least 95 of them must complete within 90 seconds.
**Validates: Requirements 27.1, 27.2**

**Property 19: Throughput Guarantee**
*For any* hospital over a 24-hour period, the system must successfully process at least 100 snapshot analyses.
**Validates: Requirements 27.3, 27.4**

**Property 20: Cost Efficiency**
*For any* analysis session, the average cost per analysis must not exceed $0.20.
**Validates: Requirements 26.10**

### Offline Capability Properties

**Property 21: Offline Data Availability**
*For any* cached patient, when the network is offline, the patient's data and active fractures must be accessible from local storage.
**Validates: Requirements 29.3, 29.4, 29.5**

**Property 22: Sync Queue Persistence**
*For any* action queued while offline, the action must persist in the queue until successfully synced or explicitly removed.
**Validates: Requirements 29.6, 29.7**

**Property 23: Cache Freshness Indicator**
*For any* cached data displayed to the user, the UI must show a timestamp indicating when the data was last updated.
**Validates: Requirements 29.8, 29.9**

### Multi-Language Properties

**Property 24: Language Consistency**
*For any* clinical interface element, when the user selects a language, all labels and messages must be displayed in that language.
**Validates: Requirements 36.1, 36.2, 36.3, 36.4, 36.5, 36.6**

**Property 25: Medical Terminology Preservation**
*For any* translated interface, medical terminology must remain in English where translation would reduce clinical precision.
**Validates: Requirements 36.7**

### Training and Onboarding Properties

**Property 26: Tutorial Completion Tracking**
*For any* user, the system must track which training modules have been completed.
**Validates: Requirements 37.6, 37.7**

**Property 27: Contextual Help Availability**
*For any* feature in the system, contextual help must be available when the user accesses that feature for the first time.
**Validates: Requirements 37.5, 37.8**

### Configuration Management Properties

**Property 28: Configuration Validation**
*For any* configuration update, the system must validate all configuration values before applying the changes.
**Validates: Requirements 31.9, 31.10**

**Property 29: Configuration Version History**
*For any* configuration change, the system must maintain a version history entry with the old value, new value, user, and timestamp.
**Validates: Requirements 31.11, 31.12**

### Integration Testing Properties

**Property 30: HL7 Message Validation**
*For any* HL7 message received, the system must validate the message structure against HL7 v2.x specifications before processing.
**Validates: Requirements 40.7, 40.9**

**Property 31: FHIR Resource Validation**
*For any* FHIR resource received, the system must validate the resource structure against FHIR R4 specifications before processing.
**Validates: Requirements 40.8, 40.9**

### Minimum Data Requirements Properties

**Property 32: Required Field Presence**
*For any* patient admission event, the system must verify that all minimum required fields (patient demographics, admission date, primary diagnosis) are present before creating a patient timeline.
**Validates: Requirements 41.1, 41.2, 41.3**


## Detailed Design Expansions

### Offline Capability (PWA) - Requirement 29

#### Service Worker Implementation

**Service Worker Architecture**:
```typescript
// public/service-worker.ts
import { precacheAndRoute } from 'workbox-precaching';
import { registerRoute } from 'workbox-routing';
import { CacheFirst, NetworkFirst, StaleWhileRevalidate } from 'workbox-strategies';
import { ExpirationPlugin } from 'workbox-expiration';
import { BackgroundSyncPlugin } from 'workbox-background-sync';

// Precache static assets
precacheAndRoute(self.__WB_MANIFEST);

// Cache strategy for API responses
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/v1/fractures'),
  new NetworkFirst({
    cacheName: 'fractures-cache',
    plugins: [
      new ExpirationPlugin({
        maxEntries: 100,
        maxAgeSeconds: 60 * 60, // 1 hour
      }),
    ],
  })
);

// Cache strategy for patient data
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/v1/patients'),
  new NetworkFirst({
    cacheName: 'patients-cache',
    plugins: [
      new ExpirationPlugin({
        maxEntries: 50,
        maxAgeSeconds: 30 * 60, // 30 minutes
      }),
    ],
  })
);

// Cache strategy for static assets
registerRoute(
  ({ request }) => request.destination === 'image' || request.destination === 'font',
  new CacheFirst({
    cacheName: 'static-assets',
    plugins: [
      new ExpirationPlugin({
        maxEntries: 60,
        maxAgeSeconds: 30 * 24 * 60 * 60, // 30 days
      }),
    ],
  })
);

// Background sync for offline actions
const bgSyncPlugin = new BackgroundSyncPlugin('offline-actions-queue', {
  maxRetentionTime: 24 * 60, // Retry for up to 24 hours
});

registerRoute(
  ({ url }) => url.pathname.startsWith('/api/v1/fractures') && url.method === 'POST',
  new NetworkOnly({
    plugins: [bgSyncPlugin],
  }),
  'POST'
);
```

#### IndexedDB Schema for Offline Storage

**Database Structure**:
```typescript
// src/services/offline/db-schema.ts
import Dexie, { Table } from 'dexie';

export interface CachedPatient {
  patientId: string;
  hospitalId: string;
  data: PatientState;
  cachedAt: Date;
  expiresAt: Date;
}

export interface CachedFracture {
  fractureId: string;
  patientId: string;
  data: SemanticFracture;
  cachedAt: Date;
  expiresAt: Date;
}

export interface OfflineAction {
  actionId: string;
  actionType: 'ACKNOWLEDGE' | 'FEEDBACK' | 'APPROVE';
  resourceId: string;
  payload: any;
  createdAt: Date;
  retryCount: number;
  status: 'PENDING' | 'SYNCING' | 'SYNCED' | 'FAILED';
}

export interface SyncMetadata {
  key: string;
  lastSyncTime: Date;
  syncStatus: 'SUCCESS' | 'FAILED';
  errorMessage?: string;
}

export class OfflineDatabase extends Dexie {
  patients!: Table<CachedPatient, string>;
  fractures!: Table<CachedFracture, string>;
  actions!: Table<OfflineAction, string>;
  syncMetadata!: Table<SyncMetadata, string>;

  constructor() {
    super('MEDWAROfflineDB');
    
    this.version(1).stores({
      patients: 'patientId, hospitalId, cachedAt, expiresAt',
      fractures: 'fractureId, patientId, cachedAt, expiresAt',
      actions: 'actionId, actionType, resourceId, createdAt, status',
      syncMetadata: 'key, lastSyncTime'
    });
  }
}

export const offlineDB = new OfflineDatabase();
```

#### Sync Strategies and Conflict Resolution

**Sync Manager**:
```typescript
// src/services/offline/sync-manager.ts
export class SyncManager {
  private syncInProgress = false;
  
  async syncAll(): Promise<SyncResult> {
    if (this.syncInProgress) {
      return { status: 'ALREADY_SYNCING' };
    }
    
    this.syncInProgress = true;
    
    try {
      // 1. Sync offline actions first (user-initiated changes)
      const actionResults = await this.syncOfflineActions();
      
      // 2. Refresh cached data from server
      const dataResults = await this.refreshCachedData();
      
      // 3. Resolve any conflicts
      const conflicts = await this.detectConflicts();
      if (conflicts.length > 0) {
        await this.resolveConflicts(conflicts);
      }
      
      // 4. Update sync metadata
      await offlineDB.syncMetadata.put({
        key: 'last-full-sync',
        lastSyncTime: new Date(),
        syncStatus: 'SUCCESS'
      });
      
      return {
        status: 'SUCCESS',
        actionsSynced: actionResults.synced,
        actionsFailed: actionResults.failed,
        dataRefreshed: dataResults.refreshed
      };
    } catch (error) {
      await offlineDB.syncMetadata.put({
        key: 'last-full-sync',
        lastSyncTime: new Date(),
        syncStatus: 'FAILED',
        errorMessage: error.message
      });
      
      return {
        status: 'FAILED',
        error: error.message
      };
    } finally {
      this.syncInProgress = false;
    }
  }
  
  private async syncOfflineActions(): Promise<ActionSyncResult> {
    const pendingActions = await offlineDB.actions
      .where('status')
      .equals('PENDING')
      .toArray();
    
    let synced = 0;
    let failed = 0;
    
    for (const action of pendingActions) {
      try {
        // Update status to syncing
        await offlineDB.actions.update(action.actionId, { status: 'SYNCING' });
        
        // Send to server
        await this.sendActionToServer(action);
        
        // Mark as synced
        await offlineDB.actions.update(action.actionId, { status: 'SYNCED' });
        synced++;
      } catch (error) {
        // Increment retry count
        const newRetryCount = action.retryCount + 1;
        
        if (newRetryCount >= 3) {
          // Max retries reached, mark as failed
          await offlineDB.actions.update(action.actionId, {
            status: 'FAILED',
            retryCount: newRetryCount
          });
          failed++;
        } else {
          // Reset to pending for next sync attempt
          await offlineDB.actions.update(action.actionId, {
            status: 'PENDING',
            retryCount: newRetryCount
          });
        }
      }
    }
    
    return { synced, failed };
  }
  
  private async resolveConflicts(conflicts: Conflict[]): Promise<void> {
    for (const conflict of conflicts) {
      // Server state wins by default (last-write-wins)
      if (conflict.type === 'FRACTURE_STATUS') {
        // If server shows acknowledged but local shows pending,
        // trust server (another user may have acknowledged)
        await offlineDB.fractures.update(conflict.resourceId, {
          data: conflict.serverState
        });
      } else if (conflict.type === 'PATIENT_DATA') {
        // For patient data, always use server state
        await offlineDB.patients.update(conflict.resourceId, {
          data: conflict.serverState
        });
      }
      
      // Notify user of conflict resolution
      await this.notifyUser({
        type: 'CONFLICT_RESOLVED',
        message: `Data for ${conflict.resourceType} was updated by another user`,
        resourceId: conflict.resourceId
      });
    }
  }
}
```

#### Cache Invalidation Policies

**Cache Invalidation Strategy**:
```typescript
// src/services/offline/cache-invalidation.ts
export class CacheInvalidationManager {
  // Time-based expiration
  async invalidateExpiredCache(): Promise<void> {
    const now = new Date();
    
    // Remove expired patients
    await offlineDB.patients
      .where('expiresAt')
      .below(now)
      .delete();
    
    // Remove expired fractures
    await offlineDB.fractures
      .where('expiresAt')
      .below(now)
      .delete();
  }
  
  // Event-based invalidation
  async invalidateOnEvent(event: CacheInvalidationEvent): Promise<void> {
    switch (event.type) {
      case 'PATIENT_DISCHARGED':
        // Remove patient and all associated fractures
        await offlineDB.patients.delete(event.patientId);
        await offlineDB.fractures
          .where('patientId')
          .equals(event.patientId)
          .delete();
        break;
        
      case 'FRACTURE_RESOLVED':
        // Remove specific fracture
        await offlineDB.fractures.delete(event.fractureId);
        break;
        
      case 'USER_LOGOUT':
        // Clear all cached data
        await offlineDB.patients.clear();
        await offlineDB.fractures.clear();
        await offlineDB.actions.clear();
        break;
        
      case 'HOSPITAL_CHANGED':
        // Clear data for previous hospital
        await offlineDB.patients
          .where('hospitalId')
          .equals(event.previousHospitalId)
          .delete();
        await offlineDB.fractures
          .where('patientId')
          .anyOf(await this.getPatientIdsForHospital(event.previousHospitalId))
          .delete();
        break;
    }
  }
  
  // Size-based eviction (LRU)
  async evictLRUIfNeeded(): Promise<void> {
    const maxPatients = 50;
    const maxFractures = 100;
    
    const patientCount = await offlineDB.patients.count();
    if (patientCount > maxPatients) {
      // Remove oldest cached patients
      const toRemove = patientCount - maxPatients;
      const oldestPatients = await offlineDB.patients
        .orderBy('cachedAt')
        .limit(toRemove)
        .toArray();
      
      for (const patient of oldestPatients) {
        await offlineDB.patients.delete(patient.patientId);
      }
    }
    
    const fractureCount = await offlineDB.fractures.count();
    if (fractureCount > maxFractures) {
      // Remove oldest cached fractures
      const toRemove = fractureCount - maxFractures;
      const oldestFractures = await offlineDB.fractures
        .orderBy('cachedAt')
        .limit(toRemove)
        .toArray();
      
      for (const fracture of oldestFractures) {
        await offlineDB.fractures.delete(fracture.fractureId);
      }
    }
  }
}
```



## Appendix

### Glossary

**Semantic Fracture**: A breakdown in shared understanding between clinical roles that creates a measurable care delivery threat.

**Entropy Score**: A calibrated risk score (0.0-1.0) representing the probability of meaning loss causing a clinical workflow failure.

**OODA Loop**: Observe-Orient-Decide-Act cycle for continuous threat assessment and response.

**Agent**: A specialized AI persona (Nurse, Doctor, PCP, Patient, Adversary, Compliance) that analyzes clinical data from a specific role perspective.

**Evidence Grounding**: The requirement that all fracture claims must be backed by verifiable excerpts from source data.

**Countermeasure**: A role-specific actionable recommendation designed to prevent a predicted semantic fracture.

**Clinical Moment**: A high-risk event in patient care (shift handover, discharge, ICU transfer) that triggers analysis.

**Convergence Score**: A measure of agent agreement during multi-round debates, used to determine early termination.

**Calibration**: The process of adjusting entropy score thresholds and fracture priors based on feedback data.

**Teach-Back**: A validation method where patient instructions are paraphrased to verify comprehension.

### Acronyms

- **ADT**: Admission, Discharge, Transfer (HL7 message type)
- **API**: Application Programming Interface
- **ASR**: Automatic Speech Recognition
- **AWS**: Amazon Web Services
- **CDK**: Cloud Development Kit
- **CDN**: Content Delivery Network
- **DPDP**: Digital Personal Data Protection (Act)
- **EHR**: Electronic Health Record
- **FHIR**: Fast Healthcare Interoperability Resources
- **GSI**: Global Secondary Index (DynamoDB)
- **HL7**: Health Level 7 (healthcare data standard)
- **HMIS**: Hospital Management Information System
- **IAM**: Identity and Access Management
- **ICU**: Intensive Care Unit
- **KMS**: Key Management Service
- **LLM**: Large Language Model
- **MRN**: Medical Record Number
- **NABH**: National Accreditation Board for Hospitals
- **NABL**: National Accreditation Board for Testing and Calibration Laboratories
- **OCR**: Optical Character Recognition
- **OODA**: Observe-Orient-Decide-Act
- **ORM**: Order Entry (HL7 message type)
- **ORU**: Observation Result (HL7 message type)
- **PBT**: Property-Based Testing
- **PCP**: Primary Care Provider
- **PHI**: Protected Health Information
- **PII**: Personally Identifiable Information
- **PWA**: Progressive Web App
- **RBAC**: Role-Based Access Control
- **REST**: Representational State Transfer
- **SaaS**: Software as a Service
- **SNS**: Simple Notification Service
- **SQS**: Simple Queue Service
- **TLS**: Transport Layer Security
- **TTL**: Time To Live
- **VPC**: Virtual Private Cloud
- **WAF**: Web Application Firewall
- **WSS**: WebSocket Secure


### References

**Healthcare Standards**:
1. HL7 Version 2.x Implementation Guide: https://www.hl7.org/implement/standards/product_brief.cfm?product_id=185
2. FHIR R4 Specification: https://hl7.org/fhir/R4/
3. NABH Standards: https://nabh.co/
4. NABL Standards: https://www.nabl-india.org/

**Regulatory Compliance**:
1. Digital Personal Data Protection Act, 2023: https://www.meity.gov.in/writereaddata/files/Digital%20Personal%20Data%20Protection%20Act%202023.pdf
2. HIPAA Security Rule (for reference): https://www.hhs.gov/hipaa/for-professionals/security/index.html

**Technical Documentation**:
1. AWS Bedrock Documentation: https://docs.aws.amazon.com/bedrock/
2. AWS Lambda Best Practices: https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html
3. DynamoDB Best Practices: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html
4. AWS Well-Architected Framework: https://aws.amazon.com/architecture/well-architected/

**AI/ML Resources**:
1. Anthropic Claude Documentation: https://docs.anthropic.com/
2. Prompt Engineering Guide: https://www.promptingguide.ai/
3. Property-Based Testing with fast-check: https://github.com/dubzzz/fast-check

**Testing Frameworks**:
1. Jest Documentation: https://jestjs.io/
2. Playwright Documentation: https://playwright.dev/
3. k6 Load Testing: https://k6.io/docs/

### Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2024-02-12 | MEDWAR Team | Initial production-grade design document |

### Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Technical Lead | | | |
| Product Manager | | | |
| Security Officer | | | |
| Compliance Officer | | | |

---

**End of Design Document**



### Alert Notification System - Requirement 30

#### Escalation Policies and Workflows

**Escalation Policy Engine**:
```typescript
// src/services/notifications/escalation-engine.ts
export interface EscalationPolicy {
  policyId: string;
  hospitalId: string;
  fractureType?: FractureType;
  minEntropyScore: number;
  escalationChain: EscalationLevel[];
}

export interface EscalationLevel {
  level: number;
  delayMinutes: number;
  targetRoles: ClinicalRole[];
  channels: NotificationChannel[];
  requireAcknowledgement: boolean;
}

export class EscalationPolicyEngine {
  async triggerEscalation(fracture: SemanticFracture): Promise<void> {
    const policy = await this.getEscalationPolicy(fracture);
    
    // Create escalation tracker
    const escalation: EscalationTracker = {
      escalationId: generateId(),
      fractureId: fracture.fractureId,
      policyId: policy.policyId,
      currentLevel: 0,
      startedAt: new Date(),
      status: 'ACTIVE'
    };
    
    await this.saveEscalationTracker(escalation);
    
    // Start escalation chain
    await this.executeEscalationLevel(escalation, policy.escalationChain[0]);
    
    // Schedule next escalation levels
    for (let i = 1; i < policy.escalationChain.length; i++) {
      const level = policy.escalationChain[i];
      await this.scheduleEscalation(escalation.escalationId, level, level.delayMinutes);
    }
  }
  
  private async executeEscalationLevel(
    escalation: EscalationTracker,
    level: EscalationLevel
  ): Promise<void> {
    // Get target users for this level
    const targetUsers = await this.getTargetUsers(
      escalation.fractureId,
      level.targetRoles
    );
    
    // Send notifications via all configured channels
    for (const user of targetUsers) {
      for (const channel of level.channels) {
        await this.sendNotification({
          userId: user.userId,
          channel,
          fractureId: escalation.fractureId,
          escalationLevel: level.level,
          requireAcknowledgement: level.requireAcknowledgement
        });
      }
    }
    
    // Update escalation tracker
    await this.updateEscalationTracker(escalation.escalationId, {
      currentLevel: level.level,
      lastEscalatedAt: new Date()
    });
  }
  
  async handleAcknowledgement(fractureId: string, userId: string): Promise<void> {
    // Find active escalation for this fracture
    const escalation = await this.getActiveEscalation(fractureId);
    
    if (escalation) {
      // Cancel pending escalations
      await this.cancelScheduledEscalations(escalation.escalationId);
      
      // Mark escalation as resolved
      await this.updateEscalationTracker(escalation.escalationId, {
        status: 'RESOLVED',
        resolvedBy: userId,
        resolvedAt: new Date()
      });
    }
  }
}
```

**Default Escalation Policies**:
```typescript
const defaultEscalationPolicies: EscalationPolicy[] = [
  {
    policyId: 'high-risk-default',
    hospitalId: '*', // Applies to all hospitals
    minEntropyScore: 0.7,
    escalationChain: [
      {
        level: 0,
        delayMinutes: 0,
        targetRoles: ['NURSE'],
        channels: ['PUSH', 'SMS'],
        requireAcknowledgement: true
      },
      {
        level: 1,
        delayMinutes: 2,
        targetRoles: ['CHARGE_NURSE'],
        channels: ['PUSH', 'SMS', 'EMAIL'],
        requireAcknowledgement: true
      },
      {
        level: 2,
        delayMinutes: 5,
        targetRoles: ['DOCTOR', 'ATTENDING_PHYSICIAN'],
        channels: ['PUSH', 'SMS', 'PHONE'],
        requireAcknowledgement: true
      },
      {
        level: 3,
        delayMinutes: 10,
        targetRoles: ['PATIENT_SAFETY_OFFICER'],
        channels: ['PUSH', 'SMS', 'PHONE', 'EMAIL'],
        requireAcknowledgement: false
      }
    ]
  },
  {
    policyId: 'critical-medication-error',
    hospitalId: '*',
    fractureType: 'CONTRAINDICATION_OMISSION',
    minEntropyScore: 0.8,
    escalationChain: [
      {
        level: 0,
        delayMinutes: 0,
        targetRoles: ['NURSE', 'DOCTOR'],
        channels: ['PUSH', 'SMS', 'PHONE'],
        requireAcknowledgement: true
      },
      {
        level: 1,
        delayMinutes: 1, // Faster escalation for critical issues
        targetRoles: ['ATTENDING_PHYSICIAN', 'PATIENT_SAFETY_OFFICER'],
        channels: ['PUSH', 'SMS', 'PHONE'],
        requireAcknowledgement: true
      }
    ]
  }
];
```

#### Notification Channels

**Multi-Channel Notification Service**:
```typescript
// src/services/notifications/notification-service.ts
export class NotificationService {
  async sendNotification(notification: NotificationRequest): Promise<NotificationResult> {
    const user = await this.getUserPreferences(notification.userId);
    const channels = this.selectChannels(notification.channel, user.preferences);
    
    const results: ChannelResult[] = [];
    
    for (const channel of channels) {
      try {
        const result = await this.sendViaChannel(channel, notification);
        results.push(result);
        
        if (result.success) {
          // If any channel succeeds, consider notification sent
          break;
        }
      } catch (error) {
        results.push({
          channel,
          success: false,
          error: error.message
        });
      }
    }
    
    // Log notification attempt
    await this.logNotification({
      notificationId: generateId(),
      userId: notification.userId,
      fractureId: notification.fractureId,
      channels: results,
      sentAt: new Date()
    });
    
    return {
      success: results.some(r => r.success),
      channels: results
    };
  }
  
  private async sendViaChannel(
    channel: NotificationChannel,
    notification: NotificationRequest
  ): Promise<ChannelResult> {
    switch (channel) {
      case 'PUSH':
        return await this.sendPushNotification(notification);
      case 'SMS':
        return await this.sendSMSNotification(notification);
      case 'EMAIL':
        return await this.sendEmailNotification(notification);
      case 'PHONE':
        return await this.initiatePhoneCall(notification);
      default:
        throw new Error(`Unsupported channel: ${channel}`);
    }
  }
  
  private async sendPushNotification(notification: NotificationRequest): Promise<ChannelResult> {
    // Web Push API for browser notifications
    const payload = {
      title: `High-Risk Alert: ${notification.fractureType}`,
      body: `Patient ${notification.patientId} - Entropy Score: ${notification.entropyScore}`,
      icon: '/icons/alert-icon.png',
      badge: '/icons/badge-icon.png',
      data: {
        fractureId: notification.fractureId,
        url: `/fractures/${notification.fractureId}`
      },
      actions: [
        { action: 'view', title: 'View Details' },
        { action: 'acknowledge', title: 'Acknowledge' }
      ]
    };
    
    const subscription = await this.getUserPushSubscription(notification.userId);
    await webpush.sendNotification(subscription, JSON.stringify(payload));
    
    return { channel: 'PUSH', success: true };
  }
  
  private async sendSMSNotification(notification: NotificationRequest): Promise<ChannelResult> {
    // Amazon SNS for SMS
    const phoneNumber = await this.getUserPhoneNumber(notification.userId);
    
    const message = `MEDWAR Alert: High-risk ${notification.fractureType} detected for patient ${notification.patientId}. Entropy Score: ${notification.entropyScore}. View: ${process.env.APP_URL}/fractures/${notification.fractureId}`;
    
    await sns.publish({
      PhoneNumber: phoneNumber,
      Message: message,
      MessageAttributes: {
        'AWS.SNS.SMS.SMSType': {
          DataType: 'String',
          StringValue: 'Transactional' // High priority
        }
      }
    }).promise();
    
    return { channel: 'SMS', success: true };
  }
  
  private async sendEmailNotification(notification: NotificationRequest): Promise<ChannelResult> {
    // Amazon SES for email
    const email = await this.getUserEmail(notification.userId);
    const fracture = await this.getFractureDetails(notification.fractureId);
    
    const htmlBody = this.renderEmailTemplate('fracture-alert', {
      fracture,
      user: await this.getUser(notification.userId),
      viewUrl: `${process.env.APP_URL}/fractures/${notification.fractureId}`
    });
    
    await ses.sendEmail({
      Source: 'alerts@medwar.io',
      Destination: {
        ToAddresses: [email]
      },
      Message: {
        Subject: {
          Data: `MEDWAR Alert: ${notification.fractureType}`
        },
        Body: {
          Html: {
            Data: htmlBody
          }
        }
      }
    }).promise();
    
    return { channel: 'EMAIL', success: true };
  }
}
```

#### Rate Limiting and Alert Fatigue Prevention

**Rate Limiter**:
```typescript
// src/services/notifications/rate-limiter.ts
export class NotificationRateLimiter {
  private readonly limits = {
    PUSH: { maxPerHour: 10, maxPerDay: 50 },
    SMS: { maxPerHour: 5, maxPerDay: 20 },
    EMAIL: { maxPerHour: 10, maxPerDay: 30 }
  };
  
  async checkRateLimit(
    userId: string,
    channel: NotificationChannel
  ): Promise<RateLimitResult> {
    const now = new Date();
    const oneHourAgo = new Date(now.getTime() - 60 * 60 * 1000);
    const oneDayAgo = new Date(now.getTime() - 24 * 60 * 60 * 1000);
    
    // Count notifications in last hour
    const hourlyCount = await this.countNotifications(userId, channel, oneHourAgo);
    
    // Count notifications in last day
    const dailyCount = await this.countNotifications(userId, channel, oneDayAgo);
    
    const limit = this.limits[channel];
    
    if (hourlyCount >= limit.maxPerHour) {
      return {
        allowed: false,
        reason: 'HOURLY_LIMIT_EXCEEDED',
        retryAfter: this.calculateRetryAfter(userId, channel, 'HOURLY')
      };
    }
    
    if (dailyCount >= limit.maxPerDay) {
      return {
        allowed: false,
        reason: 'DAILY_LIMIT_EXCEEDED',
        retryAfter: this.calculateRetryAfter(userId, channel, 'DAILY')
      };
    }
    
    return { allowed: true };
  }
  
  async applyIntelligentBatching(
    userId: string,
    notifications: NotificationRequest[]
  ): Promise<NotificationRequest[]> {
    // Group similar notifications
    const grouped = this.groupSimilarNotifications(notifications);
    
    // Create batched notifications
    const batched: NotificationRequest[] = [];
    
    for (const group of grouped) {
      if (group.length === 1) {
        batched.push(group[0]);
      } else {
        // Combine multiple similar notifications into one
        batched.push({
          userId,
          channel: group[0].channel,
          title: `${group.length} High-Risk Alerts`,
          body: `You have ${group.length} new high-risk fractures requiring attention`,
          data: {
            fractureIds: group.map(n => n.fractureId),
            url: `/fractures?status=ACTIVE&highRisk=true`
          }
        });
      }
    }
    
    return batched;
  }
  
  private groupSimilarNotifications(
    notifications: NotificationRequest[]
  ): NotificationRequest[][] {
    // Group by fracture type and time window (5 minutes)
    const groups = new Map<string, NotificationRequest[]>();
    
    for (const notification of notifications) {
      const key = `${notification.fractureType}-${Math.floor(notification.timestamp.getTime() / (5 * 60 * 1000))}`;
      
      if (!groups.has(key)) {
        groups.set(key, []);
      }
      
      groups.get(key).push(notification);
    }
    
    return Array.from(groups.values());
  }
}
```

#### Notification Preferences Management

**User Preferences**:
```typescript
// src/services/notifications/preferences.ts
export interface NotificationPreferences {
  userId: string;
  channels: {
    push: {
      enabled: boolean;
      highRiskOnly: boolean;
    };
    sms: {
      enabled: boolean;
      phoneNumber: string;
      highRiskOnly: boolean;
      quietHours?: {
        start: string; // HH:MM
        end: string; // HH:MM
      };
    };
    email: {
      enabled: boolean;
      emailAddress: string;
      digestMode: boolean; // Send daily digest instead of individual emails
      digestTime?: string; // HH:MM
    };
  };
  fractureTypeFilters: {
    [key in FractureType]?: boolean; // Only notify for specific types
  };
  minEntropyScore: number; // Only notify if entropy score >= this value
}

export class NotificationPreferencesManager {
  async getPreferences(userId: string): Promise<NotificationPreferences> {
    const prefs = await dynamodb.getItem({
      TableName: 'NotificationPreferences',
      Key: { userId }
    }).promise();
    
    return prefs.Item || this.getDefaultPreferences(userId);
  }
  
  async updatePreferences(
    userId: string,
    updates: Partial<NotificationPreferences>
  ): Promise<void> {
    await dynamodb.updateItem({
      TableName: 'NotificationPreferences',
      Key: { userId },
      UpdateExpression: 'SET #channels = :channels, #filters = :filters, #minScore = :minScore',
      ExpressionAttributeNames: {
        '#channels': 'channels',
        '#filters': 'fractureTypeFilters',
        '#minScore': 'minEntropyScore'
      },
      ExpressionAttributeValues: {
        ':channels': updates.channels,
        ':filters': updates.fractureTypeFilters,
        ':minScore': updates.minEntropyScore
      }
    }).promise();
  }
  
  private getDefaultPreferences(userId: string): NotificationPreferences {
    return {
      userId,
      channels: {
        push: {
          enabled: true,
          highRiskOnly: false
        },
        sms: {
          enabled: false,
          phoneNumber: '',
          highRiskOnly: true
        },
        email: {
          enabled: true,
          emailAddress: '',
          digestMode: false
        }
      },
      fractureTypeFilters: {},
      minEntropyScore: 0.5
    };
  }
}
```


### Configuration Management - Requirement 31

#### Configuration Validation Rules

**Configuration Validator**:
```typescript
// src/services/config/validator.ts
export class ConfigurationValidator {
  private validationRules: Map<string, ValidationRule> = new Map([
    ['entropyThresholds.default', {
      type: 'number',
      min: 0.0,
      max: 1.0,
      required: true
    }],
    ['debateConfig.minRounds', {
      type: 'number',
      min: 1,
      max: 10,
      required: true
    }],
    ['debateConfig.maxRounds', {
      type: 'number',
      min: 1,
      max: 10,
      required: true,
      customValidation: (value, config) => {
        return value >= config.debateConfig.minRounds;
      },
      errorMessage: 'maxRounds must be >= minRounds'
    }],
    ['notificationConfig.rateLimitPerHour', {
      type: 'number',
      min: 1,
      max: 100,
      required: true
    }],
    ['ehrConfig.hl7Endpoint', {
      type: 'string',
      pattern: /^https?:\/\/.+/,
      required: true
    }]
  ]);
  
  async validate(config: Partial<HospitalConfiguration>): Promise<ValidationResult> {
    const errors: ValidationError[] = [];
    
    // Validate each field
    for (const [path, rule] of this.validationRules) {
      const value = this.getNestedValue(config, path);
      
      // Check required
      if (rule.required && (value === undefined || value === null)) {
        errors.push({
          path,
          message: `${path} is required`
        });
        continue;
      }
      
      if (value !== undefined && value !== null) {
        // Check type
        if (rule.type && typeof value !== rule.type) {
          errors.push({
            path,
            message: `${path} must be of type ${rule.type}`
          });
          continue;
        }
        
        // Check numeric bounds
        if (rule.type === 'number') {
          if (rule.min !== undefined && value < rule.min) {
            errors.push({
              path,
              message: `${path} must be >= ${rule.min}`
            });
          }
          if (rule.max !== undefined && value > rule.max) {
            errors.push({
              path,
              message: `${path} must be <= ${rule.max}`
            });
          }
        }
        
        // Check string pattern
        if (rule.type === 'string' && rule.pattern) {
          if (!rule.pattern.test(value)) {
            errors.push({
              path,
              message: `${path} does not match required pattern`
            });
          }
        }
        
        // Custom validation
        if (rule.customValidation) {
          if (!rule.customValidation(value, config)) {
            errors.push({
              path,
              message: rule.errorMessage || `${path} failed custom validation`
            });
          }
        }
      }
    }
    
    // Cross-field validations
    if (config.ehrConfig) {
      if (!config.ehrConfig.hl7Endpoint && !config.ehrConfig.fhirEndpoint) {
        errors.push({
          path: 'ehrConfig',
          message: 'At least one of hl7Endpoint or fhirEndpoint must be provided'
        });
      }
    }
    
    return {
      valid: errors.length === 0,
      errors
    };
  }
  
  private getNestedValue(obj: any, path: string): any {
    return path.split('.').reduce((current, key) => current?.[key], obj);
  }
}
```

#### Version Control and Rollback Mechanisms

**Configuration Version Manager**:
```typescript
// src/services/config/version-manager.ts
export interface ConfigurationVersion {
  versionId: string;
  hospitalId: string;
  version: number;
  config: HospitalConfiguration;
  changedBy: string;
  changedAt: Date;
  changeReason?: string;
  previousVersionId?: string;
}

export class ConfigurationVersionManager {
  async saveVersion(
    hospitalId: string,
    config: HospitalConfiguration,
    userId: string,
    reason?: string
  ): Promise<ConfigurationVersion> {
    // Get current version
    const currentVersion = await this.getCurrentVersion(hospitalId);
    
    // Create new version
    const newVersion: ConfigurationVersion = {
      versionId: generateId(),
      hospitalId,
      version: currentVersion ? currentVersion.version + 1 : 1,
      config,
      changedBy: userId,
      changedAt: new Date(),
      changeReason: reason,
      previousVersionId: currentVersion?.versionId
    };
    
    // Save to DynamoDB
    await dynamodb.putItem({
      TableName: 'ConfigurationVersions',
      Item: newVersion
    }).promise();
    
    // Update current configuration pointer
    await dynamodb.putItem({
      TableName: 'HospitalConfigurations',
      Item: {
        hospitalId,
        currentVersionId: newVersion.versionId,
        config,
        updatedAt: new Date()
      }
    }).promise();
    
    return newVersion;
  }
  
  async rollback(hospitalId: string, targetVersionId: string, userId: string): Promise<void> {
    // Get target version
    const targetVersion = await this.getVersion(targetVersionId);
    
    if (!targetVersion || targetVersion.hospitalId !== hospitalId) {
      throw new Error('Invalid version ID');
    }
    
    // Create new version with rolled-back config
    await this.saveVersion(
      hospitalId,
      targetVersion.config,
      userId,
      `Rollback to version ${targetVersion.version}`
    );
    
    // Log rollback event
    await this.logConfigurationEvent({
      eventType: 'ROLLBACK',
      hospitalId,
      userId,
      fromVersionId: (await this.getCurrentVersion(hospitalId)).versionId,
      toVersionId: targetVersionId,
      timestamp: new Date()
    });
  }
  
  async getVersionHistory(
    hospitalId: string,
    limit: number = 50
  ): Promise<ConfigurationVersion[]> {
    const result = await dynamodb.query({
      TableName: 'ConfigurationVersions',
      KeyConditionExpression: 'hospitalId = :hospitalId',
      ExpressionAttributeValues: {
        ':hospitalId': hospitalId
      },
      ScanIndexForward: false, // Descending order (newest first)
      Limit: limit
    }).promise();
    
    return result.Items as ConfigurationVersion[];
  }
  
  async compareVersions(
    versionId1: string,
    versionId2: string
  ): Promise<ConfigurationDiff> {
    const version1 = await this.getVersion(versionId1);
    const version2 = await this.getVersion(versionId2);
    
    return this.computeDiff(version1.config, version2.config);
  }
  
  private computeDiff(
    config1: HospitalConfiguration,
    config2: HospitalConfiguration
  ): ConfigurationDiff {
    const diff: ConfigurationDiff = {
      added: [],
      removed: [],
      modified: []
    };
    
    // Deep comparison logic
    this.compareObjects(config1, config2, '', diff);
    
    return diff;
  }
}
```

#### Configuration Deployment Strategies

**Deployment Strategy Manager**:
```typescript
// src/services/config/deployment-strategy.ts
export class ConfigurationDeploymentManager {
  async deployConfiguration(
    hospitalId: string,
    config: HospitalConfiguration,
    strategy: DeploymentStrategy
  ): Promise<DeploymentResult> {
    // Validate configuration
    const validator = new ConfigurationValidator();
    const validation = await validator.validate(config);
    
    if (!validation.valid) {
      return {
        success: false,
        errors: validation.errors
      };
    }
    
    switch (strategy.type) {
      case 'IMMEDIATE':
        return await this.deployImmediate(hospitalId, config);
        
      case 'CANARY':
        return await this.deployCanary(hospitalId, config, strategy.canaryPercentage);
        
      case 'SCHEDULED':
        return await this.scheduleDeployment(hospitalId, config, strategy.scheduledTime);
        
      default:
        throw new Error(`Unknown deployment strategy: ${strategy.type}`);
    }
  }
  
  private async deployImmediate(
    hospitalId: string,
    config: HospitalConfiguration
  ): Promise<DeploymentResult> {
    try {
      // Save new version
      const version = await this.versionManager.saveVersion(
        hospitalId,
        config,
        'system',
        'Immediate deployment'
      );
      
      // Invalidate cache
      await this.invalidateConfigCache(hospitalId);
      
      // Notify all active sessions
      await this.notifyConfigurationChange(hospitalId);
      
      return {
        success: true,
        versionId: version.versionId,
        deployedAt: new Date()
      };
    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  }
  
  private async deployCanary(
    hospitalId: string,
    config: HospitalConfiguration,
    canaryPercentage: number
  ): Promise<DeploymentResult> {
    // Save new version
    const version = await this.versionManager.saveVersion(
      hospitalId,
      config,
      'system',
      `Canary deployment (${canaryPercentage}%)`
    );
    
    // Create canary deployment record
    await dynamodb.putItem({
      TableName: 'CanaryDeployments',
      Item: {
        deploymentId: generateId(),
        hospitalId,
        versionId: version.versionId,
        canaryPercentage,
        startedAt: new Date(),
        status: 'ACTIVE'
      }
    }).promise();
    
    // Monitor metrics for 15 minutes
    setTimeout(async () => {
      const metrics = await this.monitorCanaryMetrics(hospitalId, version.versionId);
      
      if (metrics.healthy) {
        // Promote to 100%
        await this.deployImmediate(hospitalId, config);
      } else {
        // Rollback
        await this.versionManager.rollback(
          hospitalId,
          version.previousVersionId,
          'system'
        );
      }
    }, 15 * 60 * 1000);
    
    return {
      success: true,
      versionId: version.versionId,
      deployedAt: new Date(),
      canaryPercentage
    };
  }
}
```

#### Audit Trail for Configuration Changes

**Configuration Audit Logger**:
```typescript
// src/services/config/audit-logger.ts
export interface ConfigurationAuditLog {
  logId: string;
  hospitalId: string;
  eventType: 'CREATE' | 'UPDATE' | 'ROLLBACK' | 'DELETE';
  userId: string;
  userRole: string;
  timestamp: Date;
  versionId: string;
  previousVersionId?: string;
  changes: ConfigurationChange[];
  reason?: string;
  ipAddress: string;
  userAgent: string;
}

export interface ConfigurationChange {
  path: string;
  oldValue: any;
  newValue: any;
  changeType: 'ADDED' | 'MODIFIED' | 'REMOVED';
}

export class ConfigurationAuditLogger {
  async logConfigurationChange(
    hospitalId: string,
    userId: string,
    eventType: string,
    versionId: string,
    changes: ConfigurationChange[],
    metadata: AuditMetadata
  ): Promise<void> {
    const auditLog: ConfigurationAuditLog = {
      logId: generateId(),
      hospitalId,
      eventType: eventType as any,
      userId,
      userRole: metadata.userRole,
      timestamp: new Date(),
      versionId,
      previousVersionId: metadata.previousVersionId,
      changes,
      reason: metadata.reason,
      ipAddress: metadata.ipAddress,
      userAgent: metadata.userAgent
    };
    
    // Store in DynamoDB
    await dynamodb.putItem({
      TableName: 'ConfigurationAuditLogs',
      Item: auditLog
    }).promise();
    
    // Also store in S3 for long-term retention
    await s3.putObject({
      Bucket: 'medwar-audit-logs',
      Key: `config-changes/${hospitalId}/${auditLog.logId}.json`,
      Body: JSON.stringify(auditLog),
      ServerSideEncryption: 'aws:kms'
    }).promise();
    
    // Send to CloudWatch for monitoring
    await cloudwatch.putMetricData({
      Namespace: 'MEDWAR/Configuration',
      MetricData: [
        {
          MetricName: 'ConfigurationChanges',
          Value: 1,
          Unit: 'Count',
          Dimensions: [
            { Name: 'HospitalId', Value: hospitalId },
            { Name: 'EventType', Value: eventType }
          ],
          Timestamp: new Date()
        }
      ]
    }).promise();
  }
  
  async getAuditTrail(
    hospitalId: string,
    startDate: Date,
    endDate: Date
  ): Promise<ConfigurationAuditLog[]> {
    const result = await dynamodb.query({
      TableName: 'ConfigurationAuditLogs',
      KeyConditionExpression: 'hospitalId = :hospitalId AND #timestamp BETWEEN :start AND :end',
      ExpressionAttributeNames: {
        '#timestamp': 'timestamp'
      },
      ExpressionAttributeValues: {
        ':hospitalId': hospitalId,
        ':start': startDate.toISOString(),
        ':end': endDate.toISOString()
      }
    }).promise();
    
    return result.Items as ConfigurationAuditLog[];
  }
  
  async generateComplianceReport(
    hospitalId: string,
    reportPeriod: { start: Date; end: Date }
  ): Promise<ComplianceReport> {
    const auditLogs = await this.getAuditTrail(
      hospitalId,
      reportPeriod.start,
      reportPeriod.end
    );
    
    return {
      hospitalId,
      reportPeriod,
      totalChanges: auditLogs.length,
      changesByType: this.groupByEventType(auditLogs),
      changesByUser: this.groupByUser(auditLogs),
      unauthorizedAttempts: auditLogs.filter(log => log.eventType === 'UNAUTHORIZED'),
      criticalChanges: auditLogs.filter(log => this.isCriticalChange(log)),
      generatedAt: new Date()
    };
  }
}
```


### Multi-Language Support for Clinical Interfaces - Requirement 36

#### Internationalization (i18n) Architecture

**i18n Configuration**:
```typescript
// src/i18n/config.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';
import Backend from 'i18next-http-backend';

export const supportedLanguages = {
  en: { name: 'English', nativeName: 'English' },
  hi: { name: 'Hindi', nativeName: 'हिंदी' },
  ta: { name: 'Tamil', nativeName: 'தமிழ்' },
  mr: { name: 'Marathi', nativeName: 'मराठी' },
  te: { name: 'Telugu', nativeName: 'తెలుగు' },
  bn: { name: 'Bengali', nativeName: 'বাংলা' }
};

i18n
  .use(Backend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    supportedLngs: Object.keys(supportedLanguages),
    
    // Namespace organization
    ns: ['common', 'fractures', 'patients', 'dashboard', 'medical'],
    defaultNS: 'common',
    
    // Backend configuration
    backend: {
      loadPath: '/locales/{{lng}}/{{ns}}.json',
      addPath: '/locales/add/{{lng}}/{{ns}}'
    },
    
    // Language detection
    detection: {
      order: ['localStorage', 'navigator', 'htmlTag'],
      caches: ['localStorage'],
      lookupLocalStorage: 'i18nextLng'
    },
    
    // Interpolation
    interpolation: {
      escapeValue: false, // React already escapes
      format: (value, format, lng) => {
        if (format === 'uppercase') return value.toUpperCase();
        if (format === 'lowercase') return value.toLowerCase();
        if (value instanceof Date) {
          return new Intl.DateTimeFormat(lng).format(value);
        }
        return value;
      }
    },
    
    // React specific
    react: {
      useSuspense: true,
      bindI18n: 'languageChanged loaded',
      bindI18nStore: 'added removed',
      transEmptyNodeValue: '',
      transSupportBasicHtmlNodes: true,
      transKeepBasicHtmlNodesFor: ['br', 'strong', 'i', 'p']
    }
  });

export default i18n;
```

**Translation Files Structure**:
```json
// public/locales/en/fractures.json
{
  "types": {
    "MEDICATION_RECONCILIATION": "Medication Reconciliation Issue",
    "FOLLOW_UP_COORDINATION": "Follow-up Coordination Gap",
    "CONTRAINDICATION_OMISSION": "Contraindication Omission",
    "HANDOVER_TASK_OMISSION": "Handover Task Missing",
    "CRITICAL_CONTEXT_LOSS": "Critical Context Loss",
    "PATIENT_COMPREHENSION": "Patient Comprehension Issue",
    "RESPONSIBILITY_AMBIGUITY": "Responsibility Ambiguity",
    "TIMELINE_AMBIGUITY": "Timeline Ambiguity",
    "DEVICE_AFTERCARE": "Device Aftercare Gap"
  },
  "status": {
    "ACTIVE": "Active",
    "ACKNOWLEDGED": "Acknowledged",
    "RESOLVED": "Resolved",
    "DISMISSED": "Dismissed"
  },
  "actions": {
    "acknowledge": "Acknowledge",
    "view_details": "View Details",
    "provide_feedback": "Provide Feedback",
    "generate_countermeasures": "Generate Countermeasures"
  },
  "messages": {
    "high_risk_alert": "High-risk semantic fracture detected",
    "entropy_score": "Entropy Score: {{score}}",
    "detected_at": "Detected at {{time}}",
    "affected_roles": "Affected Roles: {{roles}}"
  }
}

// public/locales/hi/fractures.json
{
  "types": {
    "MEDICATION_RECONCILIATION": "दवा समाधान समस्या",
    "FOLLOW_UP_COORDINATION": "फॉलो-अप समन्वय अंतर",
    "CONTRAINDICATION_OMISSION": "विरोधाभास चूक",
    "HANDOVER_TASK_OMISSION": "हस्तांतरण कार्य गायब",
    "CRITICAL_CONTEXT_LOSS": "महत्वपूर्ण संदर्भ हानि",
    "PATIENT_COMPREHENSION": "रोगी समझ समस्या",
    "RESPONSIBILITY_AMBIGUITY": "जिम्मेदारी अस्पष्टता",
    "TIMELINE_AMBIGUITY": "समयरेखा अस्पष्टता",
    "DEVICE_AFTERCARE": "उपकरण देखभाल अंतर"
  },
  "status": {
    "ACTIVE": "सक्रिय",
    "ACKNOWLEDGED": "स्वीकृत",
    "RESOLVED": "हल",
    "DISMISSED": "खारिज"
  },
  "actions": {
    "acknowledge": "स्वीकार करें",
    "view_details": "विवरण देखें",
    "provide_feedback": "प्रतिक्रिया दें",
    "generate_countermeasures": "प्रतिकार उपाय बनाएं"
  },
  "messages": {
    "high_risk_alert": "उच्च जोखिम अर्थ विभाजन का पता चला",
    "entropy_score": "एंट्रॉपी स्कोर: {{score}}",
    "detected_at": "{{time}} पर पता चला",
    "affected_roles": "प्रभावित भूमिकाएं: {{roles}}"
  }
}
```

#### Translation Management Workflow

**Translation Service**:
```typescript
// src/services/i18n/translation-service.ts
export class TranslationService {
  private medicalTermsPreserved = new Set([
    // Medication names
    'Metformin', 'Aspirin', 'Lisinopril', 'Atorvastatin', 'Insulin',
    // Medical conditions
    'Diabetes', 'Hypertension', 'Myocardial Infarction', 'Pneumonia',
    // Medical procedures
    'Angioplasty', 'Appendectomy', 'Colonoscopy',
    // Lab tests
    'CBC', 'HbA1c', 'Creatinine', 'Troponin',
    // Units
    'mg', 'ml', 'mmHg', 'bpm'
  ]);
  
  async translateWithMedicalPreservation(
    text: string,
    targetLanguage: string
  ): Promise<string> {
    // Extract medical terms
    const medicalTerms = this.extractMedicalTerms(text);
    
    // Replace medical terms with placeholders
    let textWithPlaceholders = text;
    const placeholderMap = new Map<string, string>();
    
    medicalTerms.forEach((term, index) => {
      const placeholder = `__MEDICAL_TERM_${index}__`;
      textWithPlaceholders = textWithPlaceholders.replace(term, placeholder);
      placeholderMap.set(placeholder, term);
    });
    
    // Translate text with placeholders
    const translated = await this.translateText(textWithPlaceholders, targetLanguage);
    
    // Restore medical terms
    let finalTranslation = translated;
    placeholderMap.forEach((term, placeholder) => {
      finalTranslation = finalTranslation.replace(placeholder, term);
    });
    
    return finalTranslation;
  }
  
  private extractMedicalTerms(text: string): string[] {
    const terms: string[] = [];
    
    // Check against preserved terms list
    this.medicalTermsPreserved.forEach(term => {
      if (text.includes(term)) {
        terms.push(term);
      }
    });
    
    // Extract drug names (capitalized words followed by dosage)
    const drugPattern = /\b[A-Z][a-z]+(?:\s+\d+\s*(?:mg|ml|mcg|g|units?))\b/g;
    const drugs = text.match(drugPattern) || [];
    terms.push(...drugs);
    
    // Extract medical abbreviations (2-5 uppercase letters)
    const abbreviationPattern = /\b[A-Z]{2,5}\b/g;
    const abbreviations = text.match(abbreviationPattern) || [];
    terms.push(...abbreviations);
    
    return [...new Set(terms)]; // Remove duplicates
  }
  
  private async translateText(text: string, targetLanguage: string): Promise<string> {
    // Use AWS Translate for translation
    const result = await translate.translateText({
      Text: text,
      SourceLanguageCode: 'en',
      TargetLanguageCode: targetLanguage,
      TerminologyNames: ['medical-terminology'] // Custom terminology
    }).promise();
    
    return result.TranslatedText;
  }
  
  async validateTranslation(
    originalText: string,
    translatedText: string,
    targetLanguage: string
  ): Promise<ValidationResult> {
    const issues: ValidationIssue[] = [];
    
    // Check medical term preservation
    const originalTerms = this.extractMedicalTerms(originalText);
    originalTerms.forEach(term => {
      if (!translatedText.includes(term)) {
        issues.push({
          type: 'MEDICAL_TERM_MISSING',
          term,
          severity: 'HIGH'
        });
      }
    });
    
    // Check length ratio (translated text shouldn't be >2x original)
    const lengthRatio = translatedText.length / originalText.length;
    if (lengthRatio > 2.0) {
      issues.push({
        type: 'LENGTH_ANOMALY',
        message: `Translation is ${lengthRatio.toFixed(1)}x longer than original`,
        severity: 'MEDIUM'
      });
    }
    
    // Check for untranslated placeholders
    const placeholderPattern = /__[A-Z_]+__/g;
    const untranslatedPlaceholders = translatedText.match(placeholderPattern);
    if (untranslatedPlaceholders) {
      issues.push({
        type: 'UNTRANSLATED_PLACEHOLDER',
        placeholders: untranslatedPlaceholders,
        severity: 'HIGH'
      });
    }
    
    return {
      valid: issues.filter(i => i.severity === 'HIGH').length === 0,
      issues
    };
  }
}
```

#### Language-Specific UI Considerations

**UI Adaptation Service**:
```typescript
// src/services/i18n/ui-adapter.ts
export class UIAdaptationService {
  private languageConfigs = {
    en: {
      direction: 'ltr',
      fontFamily: 'Inter, sans-serif',
      fontSize: {
        base: '16px',
        small: '14px',
        large: '18px'
      },
      lineHeight: 1.5
    },
    hi: {
      direction: 'ltr',
      fontFamily: 'Noto Sans Devanagari, sans-serif',
      fontSize: {
        base: '18px', // Slightly larger for Devanagari
        small: '16px',
        large: '20px'
      },
      lineHeight: 1.6 // More line height for complex scripts
    },
    ta: {
      direction: 'ltr',
      fontFamily: 'Noto Sans Tamil, sans-serif',
      fontSize: {
        base: '18px',
        small: '16px',
        large: '20px'
      },
      lineHeight: 1.6
    },
    mr: {
      direction: 'ltr',
      fontFamily: 'Noto Sans Devanagari, sans-serif',
      fontSize: {
        base: '18px',
        small: '16px',
        large: '20px'
      },
      lineHeight: 1.6
    },
    te: {
      direction: 'ltr',
      fontFamily: 'Noto Sans Telugu, sans-serif',
      fontSize: {
        base: '18px',
        small: '16px',
        large: '20px'
      },
      lineHeight: 1.6
    },
    bn: {
      direction: 'ltr',
      fontFamily: 'Noto Sans Bengali, sans-serif',
      fontSize: {
        base: '18px',
        small: '16px',
        large: '20px'
      },
      lineHeight: 1.6
    }
  };
  
  applyLanguageStyles(language: string): void {
    const config = this.languageConfigs[language] || this.languageConfigs.en;
    
    // Apply to document root
    document.documentElement.dir = config.direction;
    document.documentElement.style.fontFamily = config.fontFamily;
    document.documentElement.style.fontSize = config.fontSize.base;
    document.documentElement.style.lineHeight = config.lineHeight.toString();
    
    // Update CSS custom properties
    document.documentElement.style.setProperty('--font-size-base', config.fontSize.base);
    document.documentElement.style.setProperty('--font-size-small', config.fontSize.small);
    document.documentElement.style.setProperty('--font-size-large', config.fontSize.large);
  }
  
  getDateTimeFormat(language: string): Intl.DateTimeFormat {
    return new Intl.DateTimeFormat(language, {
      year: 'numeric',
      month: 'long',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    });
  }
  
  getNumberFormat(language: string): Intl.NumberFormat {
    return new Intl.NumberFormat(language, {
      minimumFractionDigits: 0,
      maximumFractionDigits: 2
    });
  }
  
  adjustLayoutForLanguage(language: string): LayoutAdjustments {
    const config = this.languageConfigs[language];
    
    return {
      // Increase button padding for languages with taller scripts
      buttonPadding: config.fontSize.base === '18px' ? '12px 24px' : '10px 20px',
      
      // Adjust input height for taller scripts
      inputHeight: config.fontSize.base === '18px' ? '44px' : '40px',
      
      // Adjust modal width for longer translations
      modalWidth: language === 'en' ? '600px' : '700px',
      
      // Adjust table column widths
      tableColumnWidthMultiplier: language === 'en' ? 1.0 : 1.2
    };
  }
}
```

#### RTL Support (Future Enhancement)

**RTL Support Framework**:
```typescript
// src/services/i18n/rtl-support.ts
export class RTLSupportService {
  private rtlLanguages = new Set(['ar', 'he', 'ur']); // Arabic, Hebrew, Urdu
  
  isRTL(language: string): boolean {
    return this.rtlLanguages.has(language);
  }
  
  applyRTLStyles(language: string): void {
    if (this.isRTL(language)) {
      document.documentElement.dir = 'rtl';
      
      // Apply RTL-specific CSS
      document.body.classList.add('rtl');
      
      // Flip icons and directional elements
      this.flipDirectionalElements();
    } else {
      document.documentElement.dir = 'ltr';
      document.body.classList.remove('rtl');
    }
  }
  
  private flipDirectionalElements(): void {
    // Flip chevrons, arrows, and other directional icons
    const directionalIcons = document.querySelectorAll('[data-directional="true"]');
    directionalIcons.forEach(icon => {
      icon.classList.add('rtl-flip');
    });
  }
  
  getFlexDirection(language: string, defaultDirection: 'row' | 'column'): string {
    if (this.isRTL(language) && defaultDirection === 'row') {
      return 'row-reverse';
    }
    return defaultDirection;
  }
}
```


### Training and Onboarding Support - Requirement 37

#### Interactive Tutorial System Design

**Tutorial Manager**:
```typescript
// src/services/training/tutorial-manager.ts
export interface TutorialStep {
  stepId: string;
  title: string;
  description: string;
  targetElement?: string; // CSS selector
  position: 'top' | 'bottom' | 'left' | 'right';
  action?: 'click' | 'input' | 'scroll';
  validation?: () => boolean;
  skippable: boolean;
}

export interface Tutorial {
  tutorialId: string;
  title: string;
  description: string;
  targetRole: ClinicalRole;
  estimatedDuration: number; // minutes
  steps: TutorialStep[];
  prerequisites?: string[]; // Other tutorial IDs
}

export class TutorialManager {
  private tutorials: Map<string, Tutorial> = new Map();
  
  constructor() {
    this.initializeTutorials();
  }
  
  private initializeTutorials(): void {
    // Nurse Tactical View Tutorial
    this.tutorials.set('nurse-tactical-view-basics', {
      tutorialId: 'nurse-tactical-view-basics',
      title: 'Nurse Tactical View Basics',
      description: 'Learn how to use the Nurse Tactical View to monitor patients and respond to alerts',
      targetRole: 'NURSE',
      estimatedDuration: 10,
      steps: [
        {
          stepId: 'welcome',
          title: 'Welcome to MEDWAR',
          description: 'This tutorial will guide you through the Nurse Tactical View interface',
          position: 'bottom',
          skippable: false
        },
        {
          stepId: 'patient-list',
          title: 'Patient List',
          description: 'This is your patient list. Patients with active high-risk fractures are highlighted in red.',
          targetElement: '[data-tutorial="patient-list"]',
          position: 'right',
          skippable: true
        },
        {
          stepId: 'fracture-alert',
          title: 'Fracture Alerts',
          description: 'Click on a patient to see their active fractures. High-risk fractures require immediate attention.',
          targetElement: '[data-tutorial="patient-card"]:first-child',
          position: 'right',
          action: 'click',
          skippable: true
        },
        {
          stepId: 'acknowledge',
          title: 'Acknowledge Alerts',
          description: 'Click the "Acknowledge" button to confirm you have seen the alert. This stops escalation.',
          targetElement: '[data-tutorial="acknowledge-button"]',
          position: 'top',
          action: 'click',
          validation: () => this.checkAcknowledgement(),
          skippable: true
        },
        {
          stepId: 'countermeasures',
          title: 'View Countermeasures',
          description: 'Countermeasures are specific actions you can take to prevent the predicted issue.',
          targetElement: '[data-tutorial="countermeasures"]',
          position: 'left',
          skippable: true
        },
        {
          stepId: 'feedback',
          title: 'Provide Feedback',
          description: 'Help improve MEDWAR by providing feedback on alerts. Was this alert helpful?',
          targetElement: '[data-tutorial="feedback-buttons"]',
          position: 'top',
          skippable: true
        },
        {
          stepId: 'offline-mode',
          title: 'Offline Mode',
          description: 'MEDWAR works offline! Your actions are queued and synced when connection is restored.',
          targetElement: '[data-tutorial="offline-indicator"]',
          position: 'bottom',
          skippable: true
        },
        {
          stepId: 'complete',
          title: 'Tutorial Complete!',
          description: 'You are now ready to use the Nurse Tactical View. Access this tutorial anytime from the Help menu.',
          position: 'bottom',
          skippable: false
        }
      ]
    });
    
    // Doctor Review View Tutorial
    this.tutorials.set('doctor-review-basics', {
      tutorialId: 'doctor-review-basics',
      title: 'Doctor Review View Basics',
      description: 'Learn how to review fractures and approve patient instructions',
      targetRole: 'DOCTOR',
      estimatedDuration: 15,
      steps: [
        {
          stepId: 'welcome',
          title: 'Welcome to Doctor Review View',
          description: 'This tutorial will show you how to review semantic fractures and approve patient instructions',
          position: 'bottom',
          skippable: false
        },
        {
          stepId: 'fracture-details',
          title: 'Fracture Details',
          description: 'Each fracture shows detailed evidence from the patient record. Click to expand.',
          targetElement: '[data-tutorial="fracture-card"]',
          position: 'right',
          action: 'click',
          skippable: true
        },
        {
          stepId: 'evidence',
          title: 'Evidence Grounding',
          description: 'All fractures are backed by verifiable evidence. Click on evidence to see the source.',
          targetElement: '[data-tutorial="evidence-section"]',
          position: 'left',
          skippable: true
        },
        {
          stepId: 'agent-debate',
          title: 'Agent Debate Summary',
          description: 'See how different clinical perspectives analyzed this case.',
          targetElement: '[data-tutorial="agent-debate"]',
          position: 'left',
          skippable: true
        },
        {
          stepId: 'patient-instructions',
          title: 'Patient Instructions',
          description: 'Review and edit discharge instructions before approval.',
          targetElement: '[data-tutorial="instructions-editor"]',
          position: 'top',
          skippable: true
        },
        {
          stepId: 'approve',
          title: 'Approve Instructions',
          description: 'Once satisfied, approve the instructions for printing.',
          targetElement: '[data-tutorial="approve-button"]',
          position: 'top',
          skippable: true
        },
        {
          stepId: 'complete',
          title: 'Tutorial Complete!',
          description: 'You can now review fractures and approve patient instructions.',
          position: 'bottom',
          skippable: false
        }
      ]
    });
  }
  
  async startTutorial(tutorialId: string, userId: string): Promise<TutorialSession> {
    const tutorial = this.tutorials.get(tutorialId);
    
    if (!tutorial) {
      throw new Error(`Tutorial not found: ${tutorialId}`);
    }
    
    // Check prerequisites
    if (tutorial.prerequisites) {
      const completed = await this.getCompletedTutorials(userId);
      const missingPrereqs = tutorial.prerequisites.filter(p => !completed.includes(p));
      
      if (missingPrereqs.length > 0) {
        throw new Error(`Missing prerequisites: ${missingPrereqs.join(', ')}`);
      }
    }
    
    // Create session
    const session: TutorialSession = {
      sessionId: generateId(),
      tutorialId,
      userId,
      startedAt: new Date(),
      currentStep: 0,
      status: 'IN_PROGRESS'
    };
    
    await this.saveTutorialSession(session);
    
    return session;
  }
  
  async completeStep(sessionId: string, stepId: string): Promise<void> {
    const session = await this.getTutorialSession(sessionId);
    const tutorial = this.tutorials.get(session.tutorialId);
    
    const stepIndex = tutorial.steps.findIndex(s => s.stepId === stepId);
    
    if (stepIndex === -1) {
      throw new Error(`Step not found: ${stepId}`);
    }
    
    // Validate step completion if validation function exists
    const step = tutorial.steps[stepIndex];
    if (step.validation && !step.validation()) {
      throw new Error('Step validation failed');
    }
    
    // Update session
    session.currentStep = stepIndex + 1;
    
    if (session.currentStep >= tutorial.steps.length) {
      session.status = 'COMPLETED';
      session.completedAt = new Date();
      
      // Record completion
      await this.recordTutorialCompletion(session.userId, session.tutorialId);
    }
    
    await this.saveTutorialSession(session);
  }
}
```

#### Role-Specific Training Modules

**Training Module Manager**:
```typescript
// src/services/training/module-manager.ts
export interface TrainingModule {
  moduleId: string;
  title: string;
  description: string;
  targetRoles: ClinicalRole[];
  type: 'VIDEO' | 'INTERACTIVE' | 'DOCUMENT' | 'QUIZ';
  duration: number; // minutes
  content: ModuleContent;
  assessmentRequired: boolean;
  passingScore?: number; // percentage
}

export interface ModuleContent {
  videoUrl?: string;
  documentUrl?: string;
  interactiveTutorialId?: string;
  quizQuestions?: QuizQuestion[];
}

export interface QuizQuestion {
  questionId: string;
  question: string;
  type: 'MULTIPLE_CHOICE' | 'TRUE_FALSE' | 'SHORT_ANSWER';
  options?: string[];
  correctAnswer: string | string[];
  explanation: string;
}

export class TrainingModuleManager {
  private modules: Map<string, TrainingModule> = new Map();
  
  constructor() {
    this.initializeModules();
  }
  
  private initializeModules(): void {
    // Semantic Fracture Basics
    this.modules.set('semantic-fractures-101', {
      moduleId: 'semantic-fractures-101',
      title: 'Understanding Semantic Fractures',
      description: 'Learn what semantic fractures are and why they matter for patient safety',
      targetRoles: ['NURSE', 'DOCTOR', 'CHARGE_NURSE'],
      type: 'VIDEO',
      duration: 15,
      content: {
        videoUrl: '/training/videos/semantic-fractures-101.mp4'
      },
      assessmentRequired: true,
      passingScore: 80
    });
    
    // Entropy Score Interpretation
    this.modules.set('entropy-scores', {
      moduleId: 'entropy-scores',
      title: 'Interpreting Entropy Scores',
      description: 'Understand how entropy scores are calculated and what they mean',
      targetRoles: ['NURSE', 'DOCTOR', 'PATIENT_SAFETY_OFFICER'],
      type: 'INTERACTIVE',
      duration: 10,
      content: {
        interactiveTutorialId: 'entropy-score-tutorial'
      },
      assessmentRequired: true,
      passingScore: 75
    });
    
    // Countermeasure Implementation
    this.modules.set('countermeasures', {
      moduleId: 'countermeasures',
      title: 'Implementing Countermeasures',
      description: 'Learn how to effectively implement countermeasures to prevent semantic fractures',
      targetRoles: ['NURSE', 'DOCTOR'],
      type: 'VIDEO',
      duration: 20,
      content: {
        videoUrl: '/training/videos/countermeasures.mp4',
        quizQuestions: [
          {
            questionId: 'q1',
            question: 'What is the first step when implementing a countermeasure?',
            type: 'MULTIPLE_CHOICE',
            options: [
              'Acknowledge the alert',
              'Read the evidence',
              'Contact the patient',
              'Update the EHR'
            ],
            correctAnswer: 'Read the evidence',
            explanation: 'Always review the evidence first to understand the context of the fracture'
          },
          {
            questionId: 'q2',
            question: 'Countermeasures should be implemented even if you disagree with the alert.',
            type: 'TRUE_FALSE',
            correctAnswer: 'false',
            explanation: 'Use clinical judgment. If you disagree, provide feedback to help improve the system.'
          }
        ]
      },
      assessmentRequired: true,
      passingScore: 80
    });
  }
  
  async assignModule(userId: string, moduleId: string, assignedBy: string): Promise<void> {
    const module = this.modules.get(moduleId);
    
    if (!module) {
      throw new Error(`Module not found: ${moduleId}`);
    }
    
    await dynamodb.putItem({
      TableName: 'TrainingAssignments',
      Item: {
        assignmentId: generateId(),
        userId,
        moduleId,
        assignedBy,
        assignedAt: new Date(),
        dueDate: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // 7 days
        status: 'ASSIGNED'
      }
    }).promise();
    
    // Send notification
    await this.notifyUserOfAssignment(userId, module);
  }
  
  async recordModuleCompletion(
    userId: string,
    moduleId: string,
    assessmentScore?: number
  ): Promise<void> {
    const module = this.modules.get(moduleId);
    
    // Check if assessment is required and passed
    if (module.assessmentRequired && assessmentScore < module.passingScore) {
      throw new Error(`Assessment score ${assessmentScore}% is below passing score ${module.passingScore}%`);
    }
    
    await dynamodb.putItem({
      TableName: 'TrainingCompletions',
      Item: {
        completionId: generateId(),
        userId,
        moduleId,
        completedAt: new Date(),
        assessmentScore,
        passed: !module.assessmentRequired || assessmentScore >= module.passingScore
      }
    }).promise();
  }
}
```

#### Progress Tracking Implementation

**Progress Tracker**:
```typescript
// src/services/training/progress-tracker.ts
export interface UserTrainingProgress {
  userId: string;
  totalModulesAssigned: number;
  modulesCompleted: number;
  modulesInProgress: number;
  modulesPending: number;
  completionPercentage: number;
  lastActivityAt: Date;
  certifications: Certification[];
}

export interface Certification {
  certificationId: string;
  name: string;
  earnedAt: Date;
  expiresAt?: Date;
  requiredModules: string[];
}

export class TrainingProgressTracker {
  async getProgress(userId: string): Promise<UserTrainingProgress> {
    // Get all assignments
    const assignments = await this.getAssignments(userId);
    
    // Get all completions
    const completions = await this.getCompletions(userId);
    
    // Calculate progress
    const completedModuleIds = new Set(completions.map(c => c.moduleId));
    const inProgressModuleIds = new Set<string>();
    
    // Check for in-progress modules (started but not completed)
    for (const assignment of assignments) {
      if (!completedModuleIds.has(assignment.moduleId)) {
        const session = await this.getActiveSession(userId, assignment.moduleId);
        if (session) {
          inProgressModuleIds.add(assignment.moduleId);
        }
      }
    }
    
    const totalAssigned = assignments.length;
    const completed = completedModuleIds.size;
    const inProgress = inProgressModuleIds.size;
    const pending = totalAssigned - completed - inProgress;
    
    // Check for certifications
    const certifications = await this.checkCertifications(userId, completions);
    
    return {
      userId,
      totalModulesAssigned: totalAssigned,
      modulesCompleted: completed,
      modulesInProgress: inProgress,
      modulesPending: pending,
      completionPercentage: totalAssigned > 0 ? (completed / totalAssigned) * 100 : 0,
      lastActivityAt: this.getLastActivityDate(completions),
      certifications
    };
  }
  
  async generateProgressReport(
    hospitalId: string,
    startDate: Date,
    endDate: Date
  ): Promise<HospitalTrainingReport> {
    const users = await this.getHospitalUsers(hospitalId);
    
    const userProgress = await Promise.all(
      users.map(user => this.getProgress(user.userId))
    );
    
    return {
      hospitalId,
      reportPeriod: { start: startDate, end: endDate },
      totalUsers: users.length,
      usersWithCompletedTraining: userProgress.filter(p => p.completionPercentage === 100).length,
      averageCompletionPercentage: this.calculateAverage(userProgress.map(p => p.completionPercentage)),
      moduleCompletionRates: await this.getModuleCompletionRates(hospitalId),
      certificationCounts: this.countCertifications(userProgress),
      generatedAt: new Date()
    };
  }
}
```

#### Contextual Help System

**Contextual Help Manager**:
```typescript
// src/services/training/contextual-help.ts
export interface ContextualHelp {
  helpId: string;
  featureId: string;
  title: string;
  content: string;
  videoUrl?: string;
  relatedArticles?: string[];
  showOnFirstUse: boolean;
}

export class ContextualHelpManager {
  private helpContent: Map<string, ContextualHelp> = new Map();
  
  constructor() {
    this.initializeHelpContent();
  }
  
  private initializeHelpContent(): void {
    this.helpContent.set('fracture-acknowledge', {
      helpId: 'fracture-acknowledge',
      featureId: 'acknowledge-button',
      title: 'Acknowledging Fractures',
      content: 'Acknowledging a fracture confirms you have reviewed it and stops escalation. This does not resolve the fracture - you still need to implement the countermeasures.',
      videoUrl: '/help/videos/acknowledge.mp4',
      relatedArticles: ['countermeasures', 'escalation-policy'],
      showOnFirstUse: true
    });
    
    this.helpContent.set('entropy-score', {
      helpId: 'entropy-score',
      featureId: 'entropy-score-badge',
      title: 'Understanding Entropy Scores',
      content: 'Entropy scores range from 0.0 to 1.0 and represent the risk of a semantic fracture causing harm. Scores above 0.7 are considered high-risk and require immediate attention.',
      relatedArticles: ['entropy-calculation', 'risk-thresholds'],
      showOnFirstUse: true
    });
  }
  
  async getHelp(featureId: string, userId: string): Promise<ContextualHelp | null> {
    const help = this.helpContent.get(featureId);
    
    if (!help) {
      return null;
    }
    
    // Check if user has seen this help before
    const hasSeenBefore = await this.hasUserSeenHelp(userId, help.helpId);
    
    if (!hasSeenBefore && help.showOnFirstUse) {
      // Record that user has seen this help
      await this.recordHelpView(userId, help.helpId);
      return help;
    }
    
    return hasSeenBefore ? null : help;
  }
  
  async searchHelp(query: string): Promise<ContextualHelp[]> {
    const results: ContextualHelp[] = [];
    
    for (const help of this.helpContent.values()) {
      if (
        help.title.toLowerCase().includes(query.toLowerCase()) ||
        help.content.toLowerCase().includes(query.toLowerCase())
      ) {
        results.push(help);
      }
    }
    
    return results;
  }
}
```


### Agent Prompt Management - Requirement 39

#### Prompt Versioning and A/B Testing Framework

**Prompt Version Manager**:
```typescript
// src/services/agents/prompt-version-manager.ts
export interface PromptTemplate {
  templateId: string;
  agentType: AgentType;
  version: string;
  systemPrompt: string;
  userPromptTemplate: string;
  temperature: number;
  maxTokens: number;
  createdAt: Date;
  createdBy: string;
  status: 'DRAFT' | 'TESTING' | 'ACTIVE' | 'DEPRECATED';
  performanceMetrics?: PromptPerformanceMetrics;
}

export interface PromptPerformanceMetrics {
  totalInvocations: number;
  avgTokenUsage: number;
  avgLatency: number;
  evidenceVerificationPassRate: number;
  feedbackScore: number; // Average from user feedback
  falsePositiveRate: number;
}

export class PromptVersionManager {
  async createVersion(
    agentType: AgentType,
    systemPrompt: string,
    userPromptTemplate: string,
    userId: string
  ): Promise<PromptTemplate> {
    // Get current version number
    const currentVersion = await this.getCurrentVersion(agentType);
    const newVersionNumber = currentVersion ? this.incrementVersion(currentVersion.version) : '1.0.0';
    
    const template: PromptTemplate = {
      templateId: generateId(),
      agentType,
      version: newVersionNumber,
      systemPrompt,
      userPromptTemplate,
      temperature: 0.7,
      maxTokens: 4000,
      createdAt: new Date(),
      createdBy: userId,
      status: 'DRAFT'
    };
    
    await dynamodb.putItem({
      TableName: 'AgentPromptTemplates',
      Item: template
    }).promise();
    
    return template;
  }
  
  async testAgainstHistoricalCases(
    templateId: string,
    testCaseIds: string[]
  ): Promise<TestResults> {
    const template = await this.getTemplate(templateId);
    const results: TestResult[] = [];
    
    for (const caseId of testCaseIds) {
      const testCase = await this.getHistoricalCase(caseId);
      
      // Run agent with new prompt
      const output = await this.runAgentWithPrompt(template, testCase.input);
      
      // Compare with expected output
      const comparison = this.compareOutputs(output, testCase.expectedOutput);
      
      results.push({
        caseId,
        passed: comparison.score >= 0.8,
        score: comparison.score,
        differences: comparison.differences,
        tokenUsage: output.tokenUsage,
        latency: output.latency
      });
    }
    
    return {
      templateId,
      totalCases: testCaseIds.length,
      passed: results.filter(r => r.passed).length,
      failed: results.filter(r => !r.passed).length,
      avgScore: this.calculateAverage(results.map(r => r.score)),
      avgTokenUsage: this.calculateAverage(results.map(r => r.tokenUsage)),
      avgLatency: this.calculateAverage(results.map(r => r.latency)),
      results
    };
  }
  
  async startABTest(
    controlTemplateId: string,
    experimentTemplateId: string,
    trafficPercentage: number
  ): Promise<ABTest> {
    const abTest: ABTest = {
      testId: generateId(),
      controlTemplateId,
      experimentTemplateId,
      trafficPercentage,
      startedAt: new Date(),
      status: 'RUNNING',
      metrics: {
        control: this.initializeMetrics(),
        experiment: this.initializeMetrics()
      }
    };
    
    await dynamodb.putItem({
      TableName: 'ABTests',
      Item: abTest
    }).promise();
    
    return abTest;
  }
  
  async analyzeABTest(testId: string): Promise<ABTestAnalysis> {
    const test = await this.getABTest(testId);
    
    // Calculate statistical significance
    const significance = this.calculateStatisticalSignificance(
      test.metrics.control,
      test.metrics.experiment
    );
    
    // Determine winner
    const winner = this.determineWinner(test.metrics.control, test.metrics.experiment);
    
    return {
      testId,
      duration: Date.now() - test.startedAt.getTime(),
      controlMetrics: test.metrics.control,
      experimentMetrics: test.metrics.experiment,
      statisticalSignificance: significance,
      winner,
      recommendation: this.generateRecommendation(winner, significance)
    };
  }
  
  async promoteToProduction(templateId: string, userId: string): Promise<void> {
    const template = await this.getTemplate(templateId);
    
    // Deactivate current active template
    const currentActive = await this.getActiveTemplate(template.agentType);
    if (currentActive) {
      await this.updateTemplateStatus(currentActive.templateId, 'DEPRECATED');
    }
    
    // Activate new template
    await this.updateTemplateStatus(templateId, 'ACTIVE');
    
    // Log promotion event
    await this.logPromptEvent({
      eventType: 'PROMOTION',
      templateId,
      agentType: template.agentType,
      userId,
      timestamp: new Date()
    });
  }
  
  async rollback(agentType: AgentType, userId: string): Promise<void> {
    // Get current active template
    const currentActive = await this.getActiveTemplate(agentType);
    
    // Get previous active template
    const previousActive = await this.getPreviousActiveTemplate(agentType);
    
    if (!previousActive) {
      throw new Error('No previous version to rollback to');
    }
    
    // Deactivate current
    await this.updateTemplateStatus(currentActive.templateId, 'DEPRECATED');
    
    // Reactivate previous
    await this.updateTemplateStatus(previousActive.templateId, 'ACTIVE');
    
    // Log rollback event
    await this.logPromptEvent({
      eventType: 'ROLLBACK',
      templateId: previousActive.templateId,
      agentType,
      userId,
      fromTemplateId: currentActive.templateId,
      timestamp: new Date()
    });
  }
}
```

#### Testing Against Historical Cases

**Historical Case Manager**:
```typescript
// src/services/agents/historical-case-manager.ts
export interface HistoricalCase {
  caseId: string;
  patientId: string;
  hospitalId: string;
  clinicalMoment: ClinicalMoment;
  input: {
    patientTimeline: PatientTimeline;
    patientState: PatientState;
  };
  expectedOutput: {
    fractures: SemanticFracture[];
    countermeasures: Countermeasure[];
  };
  actualOutput?: {
    fractures: SemanticFracture[];
    countermeasures: Countermeasure[];
  };
  feedback?: FeedbackSubmission;
  createdAt: Date;
}

export class HistoricalCaseManager {
  async createTestCase(
    sessionId: string,
    feedback: FeedbackSubmission
  ): Promise<HistoricalCase> {
    // Get original analysis session
    const session = await this.getAnalysisSession(sessionId);
    
    // Create test case
    const testCase: HistoricalCase = {
      caseId: generateId(),
      patientId: session.patientId,
      hospitalId: session.hospitalId,
      clinicalMoment: session.clinicalMoment,
      input: {
        patientTimeline: session.patientTimeline,
        patientState: session.patientState
      },
      expectedOutput: this.deriveExpectedOutput(session, feedback),
      actualOutput: {
        fractures: session.fractures,
        countermeasures: session.countermeasures
      },
      feedback,
      createdAt: new Date()
    };
    
    await dynamodb.putItem({
      TableName: 'HistoricalTestCases',
      Item: testCase
    }).promise();
    
    return testCase;
  }
  
  private deriveExpectedOutput(
    session: AnalysisSession,
    feedback: FeedbackSubmission
  ): { fractures: SemanticFracture[]; countermeasures: Countermeasure[] } {
    // Adjust expected output based on feedback
    if (feedback.label === 'FALSE_POSITIVE') {
      // Remove the fracture that was marked as false positive
      return {
        fractures: session.fractures.filter(f => f.fractureId !== feedback.fractureId),
        countermeasures: session.countermeasures.filter(c => c.fractureId !== feedback.fractureId)
      };
    } else if (feedback.label === 'MISSED_RISK_REPORTED') {
      // Add the missed fracture (if provided in feedback)
      const missedFracture = this.extractMissedFractureFromFeedback(feedback);
      return {
        fractures: [...session.fractures, missedFracture],
        countermeasures: session.countermeasures
      };
    } else {
      // TRUE_POSITIVE or HELPFUL_BUT_NOT_RISK - keep as is
      return {
        fractures: session.fractures,
        countermeasures: session.countermeasures
      };
    }
  }
  
  async getCuratedTestSuite(agentType: AgentType): Promise<HistoricalCase[]> {
    // Get diverse set of test cases
    const testCases = await dynamodb.query({
      TableName: 'HistoricalTestCases',
      IndexName: 'agentType-feedback-index',
      KeyConditionExpression: 'agentType = :agentType',
      ExpressionAttributeValues: {
        ':agentType': agentType
      }
    }).promise();
    
    // Ensure diversity
    return this.selectDiverseTestCases(testCases.Items as HistoricalCase[], 50);
  }
  
  private selectDiverseTestCases(cases: HistoricalCase[], count: number): HistoricalCase[] {
    // Stratified sampling to ensure diversity
    const byFractureType = this.groupBy(cases, c => c.expectedOutput.fractures[0]?.fractureType);
    const byFeedbackLabel = this.groupBy(cases, c => c.feedback?.label);
    
    const selected: HistoricalCase[] = [];
    
    // Select proportionally from each group
    for (const [fractureType, typeCases] of Object.entries(byFractureType)) {
      const proportion = typeCases.length / cases.length;
      const toSelect = Math.ceil(count * proportion);
      selected.push(...this.randomSample(typeCases, toSelect));
    }
    
    return selected.slice(0, count);
  }
}
```

#### Performance Metrics Tracking

**Prompt Performance Tracker**:
```typescript
// src/services/agents/performance-tracker.ts
export class PromptPerformanceTracker {
  async recordInvocation(
    templateId: string,
    invocation: AgentInvocation
  ): Promise<void> {
    await dynamodb.updateItem({
      TableName: 'AgentPromptTemplates',
      Key: { templateId },
      UpdateExpression: `
        SET performanceMetrics.totalInvocations = performanceMetrics.totalInvocations + :one,
            performanceMetrics.avgTokenUsage = :avgTokenUsage,
            performanceMetrics.avgLatency = :avgLatency
      `,
      ExpressionAttributeValues: {
        ':one': 1,
        ':avgTokenUsage': invocation.tokenUsage,
        ':avgLatency': invocation.latency
      }
    }).promise();
  }
  
  async updateFeedbackMetrics(
    templateId: string,
    feedback: FeedbackSubmission
  ): Promise<void> {
    // Get current metrics
    const template = await this.getTemplate(templateId);
    const metrics = template.performanceMetrics;
    
    // Update feedback score
    const newFeedbackScore = this.calculateNewFeedbackScore(
      metrics.feedbackScore,
      feedback.label,
      metrics.totalInvocations
    );
    
    // Update false positive rate
    const newFPR = this.calculateNewFPR(
      metrics.falsePositiveRate,
      feedback.label === 'FALSE_POSITIVE',
      metrics.totalInvocations
    );
    
    await dynamodb.updateItem({
      TableName: 'AgentPromptTemplates',
      Key: { templateId },
      UpdateExpression: `
        SET performanceMetrics.feedbackScore = :feedbackScore,
            performanceMetrics.falsePositiveRate = :fpr
      `,
      ExpressionAttributeValues: {
        ':feedbackScore': newFeedbackScore,
        ':fpr': newFPR
      }
    }).promise();
  }
  
  async generatePerformanceReport(
    agentType: AgentType,
    startDate: Date,
    endDate: Date
  ): Promise<PerformanceReport> {
    const templates = await this.getTemplatesForPeriod(agentType, startDate, endDate);
    
    return {
      agentType,
      reportPeriod: { start: startDate, end: endDate },
      templates: templates.map(t => ({
        templateId: t.templateId,
        version: t.version,
        status: t.status,
        metrics: t.performanceMetrics
      })),
      bestPerforming: this.identifyBestPerforming(templates),
      recommendations: this.generateRecommendations(templates)
    };
  }
}
```

#### Rollback Procedures

**Rollback Manager**:
```typescript
// src/services/agents/rollback-manager.ts
export class PromptRollbackManager {
  async initiateRollback(
    agentType: AgentType,
    reason: string,
    userId: string
  ): Promise<RollbackResult> {
    // Get current active template
    const currentTemplate = await this.getActiveTemplate(agentType);
    
    // Get previous stable template
    const previousTemplate = await this.getPreviousStableTemplate(agentType);
    
    if (!previousTemplate) {
      return {
        success: false,
        error: 'No previous stable version available for rollback'
      };
    }
    
    // Create rollback record
    const rollback: RollbackRecord = {
      rollbackId: generateId(),
      agentType,
      fromTemplateId: currentTemplate.templateId,
      toTemplateId: previousTemplate.templateId,
      reason,
      initiatedBy: userId,
      initiatedAt: new Date(),
      status: 'IN_PROGRESS'
    };
    
    await this.saveRollbackRecord(rollback);
    
    try {
      // Deactivate current template
      await this.updateTemplateStatus(currentTemplate.templateId, 'DEPRECATED');
      
      // Reactivate previous template
      await this.updateTemplateStatus(previousTemplate.templateId, 'ACTIVE');
      
      // Invalidate agent cache
      await this.invalidateAgentCache(agentType);
      
      // Update rollback status
      rollback.status = 'COMPLETED';
      rollback.completedAt = new Date();
      await this.saveRollbackRecord(rollback);
      
      // Notify team
      await this.notifyRollback(rollback);
      
      return {
        success: true,
        rollbackId: rollback.rollbackId,
        previousVersion: previousTemplate.version,
        currentVersion: currentTemplate.version
      };
    } catch (error) {
      rollback.status = 'FAILED';
      rollback.error = error.message;
      await this.saveRollbackRecord(rollback);
      
      return {
        success: false,
        error: error.message
      };
    }
  }
  
  async monitorPostRollback(rollbackId: string, durationMinutes: number): Promise<void> {
    const rollback = await this.getRollbackRecord(rollbackId);
    const template = await this.getTemplate(rollback.toTemplateId);
    
    // Monitor metrics for specified duration
    const startTime = Date.now();
    const endTime = startTime + (durationMinutes * 60 * 1000);
    
    while (Date.now() < endTime) {
      await this.sleep(60 * 1000); // Check every minute
      
      const metrics = await this.getCurrentMetrics(template.agentType);
      
      // Check if metrics are healthy
      if (!this.areMetricsHealthy(metrics)) {
        await this.alertOps({
          severity: 'CRITICAL',
          message: `Metrics unhealthy after rollback for ${template.agentType}`,
          rollbackId
        });
      }
    }
  }
}
```


### Integration Testing and Validation - Requirement 40

#### Test Mode Implementation Details

**Test Mode Manager**:
```typescript
// src/services/testing/test-mode-manager.ts
export class TestModeManager {
  async enableTestMode(hospitalId: string, userId: string): Promise<TestModeSession> {
    const session: TestModeSession = {
      sessionId: generateId(),
      hospitalId,
      enabledBy: userId,
      enabledAt: new Date(),
      status: 'ACTIVE',
      isolationLevel: 'FULL' // No production data affected
    };
    
    await dynamodb.putItem({
      TableName: 'TestModeSessions',
      Item: session
    }).promise();
    
    // Create isolated test environment
    await this.createTestEnvironment(session);
    
    return session;
  }
  
  private async createTestEnvironment(session: TestModeSession): Promise<void> {
    // Create test-specific DynamoDB tables with session prefix
    await this.createTestTables(session.sessionId);
    
    // Create test-specific S3 bucket prefix
    await this.createTestBucketPrefix(session.sessionId);
    
    // Configure test-specific endpoints
    await this.configureTestEndpoints(session.sessionId);
  }
  
  async processTestMessage(
    sessionId: string,
    message: HL7Message | FHIRResource
  ): Promise<TestMessageResult> {
    const session = await this.getTestSession(sessionId);
    
    if (session.status !== 'ACTIVE') {
      throw new Error('Test session is not active');
    }
    
    // Process message in isolated environment
    const result = await this.processInTestEnvironment(session, message);
    
    // Generate validation report
    const validation = await this.validateProcessing(message, result);
    
    return {
      messageId: generateId(),
      sessionId,
      messageType: this.getMessageType(message),
      processedAt: new Date(),
      result,
      validation
    };
  }
}
```


#### Synthetic Data Generation

**Synthetic Data Generator**:
```typescript
// src/services/testing/synthetic-data-generator.ts
export class SyntheticDataGenerator {
  async generateHL7Message(messageType: string): Promise<string> {
    switch (messageType) {
      case 'ADT^A01': // Admission
        return this.generateADT_A01();
      case 'ADT^A03': // Discharge
        return this.generateADT_A03();
      case 'ORM^O01': // Medication Order
        return this.generateORM_O01();
      case 'ORU^R01': // Lab Result
        return this.generateORU_R01();
      default:
        throw new Error(`Unsupported message type: ${messageType}`);
    }
  }
  
  private generateADT_A01(): string {
    const mrn = this.generateMRN();
    const timestamp = this.formatHL7Timestamp(new Date());
    
    return `MSH|^~\\&|TEST_EHR|TEST_HOSPITAL|MEDWAR|MEDWAR|${timestamp}||ADT^A01|${this.generateMessageId()}|P|2.5
EVN|A01|${timestamp}
PID|1||${mrn}||TEST^PATIENT^A||19800101|M|||123 TEST ST^^MUMBAI^MH^400001^IN|||||||
PV1|1|I|ICU^101^01^TEST_HOSPITAL||||DOC123^TEST^DOCTOR^A|||MED||||||||V12345|||||||||||||||||||||||||${timestamp}`;
  }
  
  async generateFHIRResource(resourceType: string): Promise<any> {
    switch (resourceType) {
      case 'Patient':
        return this.generatePatientResource();
      case 'Observation':
        return this.generateObservationResource();
      case 'MedicationRequest':
        return this.generateMedicationRequestResource();
      default:
        throw new Error(`Unsupported resource type: ${resourceType}`);
    }
  }
  
  private generatePatientResource(): any {
    return {
      resourceType: 'Patient',
      id: this.generateId(),
      identifier: [
        {
          system: 'http://test-hospital.org/mrn',
          value: this.generateMRN()
        }
      ],
      name: [
        {
          use: 'official',
          family: 'TestPatient',
          given: ['Test', 'A']
        }
      ],
      gender: 'male',
      birthDate: '1980-01-01',
      address: [
        {
          line: ['123 Test Street'],
          city: 'Mumbai',
          state: 'MH',
          postalCode: '400001',
          country: 'IN'
        }
      ]
    };
  }
}
```


#### Validation Report Structure

**Validation Report Generator**:
```typescript
// src/services/testing/validation-report.ts
export interface ValidationReport {
  reportId: string;
  sessionId: string;
  generatedAt: Date;
  messagesTested: number;
  validationResults: ValidationResult[];
  summary: ValidationSummary;
}

export interface ValidationResult {
  messageId: string;
  messageType: string;
  validationChecks: ValidationCheck[];
  overallStatus: 'PASS' | 'FAIL' | 'WARNING';
}

export interface ValidationCheck {
  checkName: string;
  status: 'PASS' | 'FAIL' | 'WARNING';
  details: string;
  expectedValue?: any;
  actualValue?: any;
}

export class ValidationReportGenerator {
  async generateReport(sessionId: string): Promise<ValidationReport> {
    const session = await this.getTestSession(sessionId);
    const results = await this.getValidationResults(sessionId);
    
    return {
      reportId: generateId(),
      sessionId,
      generatedAt: new Date(),
      messagesTested: results.length,
      validationResults: results,
      summary: this.generateSummary(results)
    };
  }
  
  private generateSummary(results: ValidationResult[]): ValidationSummary {
    return {
      totalMessages: results.length,
      passed: results.filter(r => r.overallStatus === 'PASS').length,
      failed: results.filter(r => r.overallStatus === 'FAIL').length,
      warnings: results.filter(r => r.overallStatus === 'WARNING').length,
      passRate: (results.filter(r => r.overallStatus === 'PASS').length / results.length) * 100,
      commonIssues: this.identifyCommonIssues(results)
    };
  }
}
```


#### Sample Message Library

**Sample Message Library**:
```typescript
// src/services/testing/sample-library.ts
export class SampleMessageLibrary {
  private samples: Map<string, SampleMessage> = new Map();
  
  constructor() {
    this.initializeSamples();
  }
  
  private initializeSamples(): void {
    // HL7 ADT^A01 - Normal Admission
    this.samples.set('hl7-adt-a01-normal', {
      id: 'hl7-adt-a01-normal',
      type: 'HL7',
      messageType: 'ADT^A01',
      description: 'Normal patient admission with complete demographics',
      content: `MSH|^~\\&|EPIC|HOSPITAL|MEDWAR|MEDWAR|20240212120000||ADT^A01|MSG001|P|2.5
EVN|A01|20240212120000
PID|1||MRN123456||DOE^JOHN^A||19800101|M|||123 MAIN ST^^MUMBAI^MH^400001^IN|||||||
PV1|1|I|ICU^101^01^HOSPITAL||||DOC123^SMITH^JANE^A|||MED||||||||V12345|||||||||||||||||||||||||20240212120000`,
      expectedOutcome: 'Patient timeline created with admission event'
    });
    
    // HL7 ORM^O01 - Medication Order with Allergy Conflict
    this.samples.set('hl7-orm-o01-allergy-conflict', {
      id: 'hl7-orm-o01-allergy-conflict',
      type: 'HL7',
      messageType: 'ORM^O01',
      description: 'Medication order that conflicts with patient allergy',
      content: `MSH|^~\\&|EPIC|HOSPITAL|MEDWAR|MEDWAR|20240212130000||ORM^O01|MSG002|P|2.5
PID|1||MRN123456||DOE^JOHN^A||19800101|M
AL1|1|DA|PENICILLIN^Penicillin|SV|Anaphylaxis
ORC|NW|ORD123|||||||20240212130000
RXO|AMOXICILLIN 500MG^Amoxicillin 500mg|500||MG|||||||`,
      expectedOutcome: 'Contraindication fracture detected'
    });
    
    // FHIR Patient Resource
    this.samples.set('fhir-patient-complete', {
      id: 'fhir-patient-complete',
      type: 'FHIR',
      resourceType: 'Patient',
      description: 'Complete patient resource with all demographics',
      content: JSON.stringify({
        resourceType: 'Patient',
        id: 'patient-001',
        identifier: [{ system: 'http://hospital.org/mrn', value: 'MRN123456' }],
        name: [{ use: 'official', family: 'Doe', given: ['John', 'A'] }],
        gender: 'male',
        birthDate: '1980-01-01',
        address: [{
          line: ['123 Main Street'],
          city: 'Mumbai',
          state: 'MH',
          postalCode: '400001',
          country: 'IN'
        }]
      }),
      expectedOutcome: 'Patient demographics updated in timeline'
    });
  }
  
  getSample(sampleId: string): SampleMessage | undefined {
    return this.samples.get(sampleId);
  }
  
  getSamplesByType(type: 'HL7' | 'FHIR'): SampleMessage[] {
    return Array.from(this.samples.values()).filter(s => s.type === type);
  }
}
```


### Minimum Required Data Inputs - Requirement 41

#### Minimum Viable Data Contract Specification

**Data Contract Validator**:
```typescript
// src/services/integration/data-contract-validator.ts
export interface MinimumDataContract {
  admission: {
    required: string[];
    optional: string[];
  };
  discharge: {
    required: string[];
    optional: string[];
  };
  medicationOrder: {
    required: string[];
    optional: string[];
  };
  labResult: {
    required: string[];
    optional: string[];
  };
}

export const minimumDataContract: MinimumDataContract = {
  admission: {
    required: [
      'patientId',
      'mrn',
      'admissionDate',
      'location',
      'primaryDiagnosis'
    ],
    optional: [
      'attendingPhysician',
      'allergies',
      'currentMedications',
      'comorbidities'
    ]
  },
  discharge: {
    required: [
      'patientId',
      'dischargeDate',
      'dischargeDiagnosis',
      'dischargeDisposition'
    ],
    optional: [
      'dischargeMedications',
      'followUpInstructions',
      'followUpAppointments'
    ]
  },
  medicationOrder: {
    required: [
      'patientId',
      'medicationName',
      'dosage',
      'route',
      'frequency'
    ],
    optional: [
      'indication',
      'duration',
      'prescribingPhysician'
    ]
  },
  labResult: {
    required: [
      'patientId',
      'testName',
      'result',
      'unit',
      'resultDate'
    ],
    optional: [
      'referenceRange',
      'abnormalFlag',
      'orderingPhysician'
    ]
  }
};

export class DataContractValidator {
  validateAdmission(data: any): ValidationResult {
    const missing = this.checkRequiredFields(
      data,
      minimumDataContract.admission.required
    );
    
    if (missing.length > 0) {
      return {
        valid: false,
        errors: missing.map(field => ({
          field,
          message: `Required field '${field}' is missing`,
          severity: 'ERROR'
        }))
      };
    }
    
    // Validate field formats
    const formatErrors = this.validateFieldFormats(data, 'admission');
    
    return {
      valid: formatErrors.length === 0,
      errors: formatErrors,
      warnings: this.checkOptionalFields(data, minimumDataContract.admission.optional)
    };
  }
  
  private checkRequiredFields(data: any, requiredFields: string[]): string[] {
    const missing: string[] = [];
    
    for (const field of requiredFields) {
      if (!this.hasValue(data, field)) {
        missing.push(field);
      }
    }
    
    return missing;
  }
  
  private hasValue(data: any, field: string): boolean {
    const value = this.getNestedValue(data, field);
    return value !== undefined && value !== null && value !== '';
  }
}
```


#### Required vs Optional Data Fields

**Field Classification**:
```typescript
// src/services/integration/field-classification.ts
export interface FieldClassification {
  fieldName: string;
  required: boolean;
  dataType: 'string' | 'number' | 'date' | 'boolean' | 'array' | 'object';
  format?: string;
  validation?: ValidationRule;
  impactOnAnalysis: 'CRITICAL' | 'HIGH' | 'MEDIUM' | 'LOW';
  fallbackStrategy?: FallbackStrategy;
}

export const fieldClassifications: Record<string, FieldClassification> = {
  // Patient Demographics
  'patientId': {
    fieldName: 'patientId',
    required: true,
    dataType: 'string',
    impactOnAnalysis: 'CRITICAL',
    validation: { pattern: /^[A-Z0-9]{6,12}$/ }
  },
  'mrn': {
    fieldName: 'mrn',
    required: true,
    dataType: 'string',
    impactOnAnalysis: 'CRITICAL',
    validation: { pattern: /^MRN[0-9]{6,10}$/ }
  },
  'dateOfBirth': {
    fieldName: 'dateOfBirth',
    required: false,
    dataType: 'date',
    format: 'YYYY-MM-DD',
    impactOnAnalysis: 'MEDIUM',
    fallbackStrategy: {
      type: 'ESTIMATE',
      method: 'Use age if available'
    }
  },
  
  // Clinical Data
  'primaryDiagnosis': {
    fieldName: 'primaryDiagnosis',
    required: true,
    dataType: 'string',
    impactOnAnalysis: 'HIGH',
    validation: { minLength: 3 }
  },
  'allergies': {
    fieldName: 'allergies',
    required: false,
    dataType: 'array',
    impactOnAnalysis: 'HIGH',
    fallbackStrategy: {
      type: 'ASSUME_NONE',
      method: 'Assume no known allergies if not provided'
    }
  },
  'currentMedications': {
    fieldName: 'currentMedications',
    required: false,
    dataType: 'array',
    impactOnAnalysis: 'HIGH',
    fallbackStrategy: {
      type: 'PARTIAL_ANALYSIS',
      method: 'Analyze with available medication data only'
    }
  },
  
  // Medication Orders
  'medicationName': {
    fieldName: 'medicationName',
    required: true,
    dataType: 'string',
    impactOnAnalysis: 'CRITICAL',
    validation: { minLength: 2 }
  },
  'dosage': {
    fieldName: 'dosage',
    required: true,
    dataType: 'string',
    impactOnAnalysis: 'CRITICAL',
    validation: { pattern: /^\d+(\.\d+)?\s*(mg|ml|g|mcg|units?)$/i }
  },
  'route': {
    fieldName: 'route',
    required: true,
    dataType: 'string',
    impactOnAnalysis: 'HIGH',
    validation: {
      enum: ['PO', 'IV', 'IM', 'SC', 'SL', 'PR', 'TOPICAL', 'INHALATION']
    }
  }
};
```


#### Data Quality Thresholds

**Quality Threshold Manager**:
```typescript
// src/services/integration/quality-thresholds.ts
export interface DataQualityThresholds {
  minimumCompleteness: number; // Percentage of required fields present
  minimumAccuracy: number; // Percentage of fields passing validation
  minimumTimeliness: number; // Maximum age of data in hours
  minimumConsistency: number; // Percentage of cross-field validations passing
}

export const defaultQualityThresholds: DataQualityThresholds = {
  minimumCompleteness: 80, // 80% of required fields must be present
  minimumAccuracy: 95, // 95% of fields must pass validation
  minimumTimeliness: 24, // Data must be less than 24 hours old
  minimumConsistency: 90 // 90% of cross-field validations must pass
};

export class DataQualityAssessor {
  async assessQuality(data: any, dataType: string): Promise<QualityAssessment> {
    const completeness = this.assessCompleteness(data, dataType);
    const accuracy = this.assessAccuracy(data, dataType);
    const timeliness = this.assessTimeliness(data);
    const consistency = this.assessConsistency(data, dataType);
    
    const overallScore = (
      completeness.score * 0.3 +
      accuracy.score * 0.3 +
      timeliness.score * 0.2 +
      consistency.score * 0.2
    );
    
    return {
      overallScore,
      completeness,
      accuracy,
      timeliness,
      consistency,
      meetsThresholds: this.checkThresholds({
        completeness: completeness.score,
        accuracy: accuracy.score,
        timeliness: timeliness.score,
        consistency: consistency.score
      }),
      recommendations: this.generateRecommendations({
        completeness,
        accuracy,
        timeliness,
        consistency
      })
    };
  }
  
  private assessCompleteness(data: any, dataType: string): CompletenessAssessment {
    const contract = minimumDataContract[dataType];
    const requiredFields = contract.required;
    const presentFields = requiredFields.filter(field => this.hasValue(data, field));
    
    return {
      score: (presentFields.length / requiredFields.length) * 100,
      totalRequired: requiredFields.length,
      present: presentFields.length,
      missing: requiredFields.filter(field => !this.hasValue(data, field))
    };
  }
  
  private assessAccuracy(data: any, dataType: string): AccuracyAssessment {
    const validations: ValidationResult[] = [];
    
    for (const [fieldName, classification] of Object.entries(fieldClassifications)) {
      if (this.hasValue(data, fieldName) && classification.validation) {
        const valid = this.validateField(data[fieldName], classification.validation);
        validations.push({
          fieldName,
          valid,
          value: data[fieldName]
        });
      }
    }
    
    const validCount = validations.filter(v => v.valid).length;
    
    return {
      score: validations.length > 0 ? (validCount / validations.length) * 100 : 100,
      totalValidated: validations.length,
      passed: validCount,
      failed: validations.length - validCount,
      failures: validations.filter(v => !v.valid)
    };
  }
}
```


#### Graceful Degradation Strategies

**Degradation Strategy Manager**:
```typescript
// src/services/integration/degradation-strategy.ts
export interface DegradationStrategy {
  strategyType: 'SKIP_ANALYSIS' | 'PARTIAL_ANALYSIS' | 'ESTIMATE_MISSING' | 'USE_DEFAULTS';
  applicableWhen: (data: any, quality: QualityAssessment) => boolean;
  apply: (data: any) => EnhancedData;
  impactOnResults: string;
}

export class GracefulDegradationManager {
  private strategies: DegradationStrategy[] = [
    {
      strategyType: 'ESTIMATE_MISSING',
      applicableWhen: (data, quality) => {
        return quality.completeness.score >= 60 && quality.completeness.score < 80;
      },
      apply: (data) => {
        // Estimate missing non-critical fields
        if (!data.dateOfBirth && data.age) {
          data.dateOfBirth = this.estimateDOBFromAge(data.age);
        }
        
        if (!data.allergies) {
          data.allergies = [];
          data._estimatedFields = [...(data._estimatedFields || []), 'allergies'];
        }
        
        return data;
      },
      impactOnResults: 'Analysis proceeds with estimated values. Results flagged as partial.'
    },
    {
      strategyType: 'PARTIAL_ANALYSIS',
      applicableWhen: (data, quality) => {
        return quality.completeness.score >= 40 && quality.completeness.score < 60;
      },
      apply: (data) => {
        // Mark analysis as partial
        data._analysisMode = 'PARTIAL';
        data._limitedCapabilities = this.identifyLimitedCapabilities(data);
        
        return data;
      },
      impactOnResults: 'Limited analysis performed. Some fracture types cannot be detected.'
    },
    {
      strategyType: 'SKIP_ANALYSIS',
      applicableWhen: (data, quality) => {
        return quality.completeness.score < 40;
      },
      apply: (data) => {
        data._analysisSkipped = true;
        data._skipReason = 'Insufficient data quality';
        
        return data;
      },
      impactOnResults: 'Analysis skipped. Data quality too low for reliable results.'
    }
  ];
  
  async applyDegradation(
    data: any,
    quality: QualityAssessment
  ): Promise<DegradationResult> {
    // Find applicable strategy
    const strategy = this.strategies.find(s => s.applicableWhen(data, quality));
    
    if (!strategy) {
      // No degradation needed - quality is sufficient
      return {
        strategyApplied: null,
        enhancedData: data,
        impactOnResults: 'None - data quality sufficient for full analysis'
      };
    }
    
    // Apply strategy
    const enhancedData = strategy.apply(data);
    
    // Log degradation event
    await this.logDegradationEvent({
      dataType: data.type,
      patientId: data.patientId,
      qualityScore: quality.overallScore,
      strategyApplied: strategy.strategyType,
      timestamp: new Date()
    });
    
    return {
      strategyApplied: strategy.strategyType,
      enhancedData,
      impactOnResults: strategy.impactOnResults
    };
  }
  
  private identifyLimitedCapabilities(data: any): string[] {
    const limited: string[] = [];
    
    if (!data.allergies) {
      limited.push('CONTRAINDICATION_DETECTION');
    }
    
    if (!data.currentMedications) {
      limited.push('MEDICATION_RECONCILIATION');
    }
    
    if (!data.followUpInstructions) {
      limited.push('FOLLOW_UP_COORDINATION');
    }
    
    return limited;
  }
  
  async generateDataQualityReport(
    hospitalId: string,
    startDate: Date,
    endDate: Date
  ): Promise<DataQualityReport> {
    const assessments = await this.getQualityAssessments(hospitalId, startDate, endDate);
    
    return {
      hospitalId,
      reportPeriod: { start: startDate, end: endDate },
      totalRecords: assessments.length,
      averageQualityScore: this.calculateAverage(assessments.map(a => a.overallScore)),
      qualityDistribution: this.calculateDistribution(assessments),
      commonIssues: this.identifyCommonIssues(assessments),
      degradationFrequency: this.calculateDegradationFrequency(assessments),
      recommendations: this.generateImprovementRecommendations(assessments)
    };
  }
}
```


## Appendix