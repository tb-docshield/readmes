# DocShield Application Architecture Diagrams

## 1. User Flow Diagram

This diagram shows the user journey through the application:

```mermaid
flowchart TD
    Login[Login Page] -->|Successful Auth| Onboarding
    Onboarding[Onboarding Controller] -->|First Step| UserOrg[User & Organization Setup]
    UserOrg -->|Next Step| PhysicianRoster[Physician Roster Setup]
    PhysicianRoster -->|Next Step| CarrierApp[Carrier Application]
    CarrierApp -->|Quick Quote| QuickQuote[Quick Quote Confirmation]
    CarrierApp -->|No Quote| NoQuote[No Quote Confirmation]
    CarrierApp -->|Next Step| Application[Detailed Application]
    QuickQuote -->|Proceed To| Application
    NoQuote -->|Proceed To| Application
    Application -->|Complete| Dashboard[Dashboard]
```

## 2. UI Component Hierarchy

This diagram shows the component structure of the onboarding process:

```mermaid
flowchart TD
    OnboardingController[Onboarding Controller] --> ProgressBar
    OnboardingController --> UserOrgOnboarding
    OnboardingController --> PhysicianOnboardingRoster
    OnboardingController --> CarrierOnboardingController
    OnboardingController --> OnboardingApplicationController
    
    CarrierOnboardingController --> IndigoApplication
    CarrierOnboardingController --> SubmissionAnimation
    CarrierOnboardingController --> QuickQuoteConfirmation
    CarrierOnboardingController --> NoQuoteConfirmation
    
    PhysicianOnboardingRoster --> NpiSelect
    PhysicianOnboardingRoster --> PhysicianAddress
    PhysicianOnboardingRoster --> SelectApplicants
    PhysicianOnboardingRoster --> OnboardingAddressModal
```

## 3. Backend Services and API Connections

This diagram shows how UI components connect to backend services:

```mermaid
flowchart TD
    Login -->|Auth| SupabaseAuth[Supabase Auth Service]
    
    UserOrgOnboarding -->|constructOrgAndPolicyPeriod_EXTERNAL| OnboardingService
    PhysicianOnboardingRoster -->|searchNpiByName_EXTERNAL & searchNpi_EXTERNAL| DataBridgeService
    PhysicianOnboardingRoster -->|enrichNpiWithDataBridge_EXTERNAL| DataBridgeService
    
    CarrierOnboardingController -->|constructProspectiveUser_EXTERNAL| OnboardingService
    CarrierOnboardingController -->|submitIndigoApplication_EXTERNAL| CarrierServices
    
    OnboardingApplicationController -->|sendCAQHRequestEmail_EXTERNAL| OnboardingService
    OnboardingApplicationController -->|updateOnboardingApplicant_EXTERNAL| OnboardingService
    
    DataBridgeService -->|External API| NpiRegistry[NPI Registry API]
    DataBridgeService -->|External API| CandorHealth[Candor Health API]
    
    CarrierServices -->|Email Service| Postmark[Postmark Email API]
```

## 4. Data Flow Diagram

This diagram shows how data moves through the application:

```mermaid
flowchart TD
    Login -->|Email & OTP| User[User Creation]
    
    UserOrgOnboarding -->|User & Org Data| ProspectiveUser
    UserOrgOnboarding -->|Org Creation| Organization
    UserOrgOnboarding -->|Policy Period| PolicyPeriod
    
    PhysicianOnboardingRoster -->|NPI Search| NpiResults[NPI Data]
    NpiResults -->|Enrichment| EnrichedApplicantData
    EnrichedApplicantData -->|Storage| Applicants
    
    CarrierOnboardingController -->|Applicant Data| IndigoSubmission
    CarrierOnboardingController -->|Practice Data| PracticeAddresses
    
    IndigoSubmission -->|Carrier Integration| ExternalQuote[Indigo Quote]
    
    OnboardingApplicationController -->|CAQH Credentials| CredentialsRequest
    OnboardingApplicationController -->|Application Data| CompletedApplication
```

## 5. External Systems Integration

```mermaid
flowchart LR
    DocShield[DocShield Application] -->|NPI Lookup| NpiRegistry[NPI Registry]
    DocShield -->|Provider Data| CandorHealth[Candor Health API]
    DocShield -->|Email Services| Postmark[Postmark]
    DocShield -->|Auth| Supabase[Supabase Auth]
    DocShield -->|Insurance Quotes| Indigo[Indigo Insurance]
    DocShield -->|Credentials| CAQH[CAQH]
    
    subgraph DataOutpost[Black Box - DataOutpost]
    end
    
    DocShield <-->|Data Integration| DataOutpost
```

## Summary of Application Structure

The DocShield application follows a multi-step onboarding flow where:

1. **Login/Registration**: Users enter through the login page using email and OTP authentication via Supabase.

2. **Onboarding Process** (sequential steps):
   - **User & Organization Setup**: Basic information collection
   - **Physician Roster Setup**: NPI lookup and data enrichment from external sources
   - **Carrier Application**: Insurance carrier integration (primarily Indigo)
   - **Detailed Application**: CAQH credentials and detailed information collection

3. **Backend Services**:
   - **DataBridgeService**: Acts as a bridge to external provider data sources
   - **OnboardingService**: Handles the core onboarding workflow and data persistence
   - **CarrierServices**: Manages insurance carrier integrations

4. **External Integrations**:
   - NPI Registry for provider lookups
   - Candor Health for data enrichment
   - Indigo for insurance quotes
   - Postmark for email communications
   - Supabase for authentication

The application is built with Next.js using the app router pattern, with server components for API functionality ("use server" directives) and client components for UI rendering ("use client" directives).
