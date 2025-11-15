# enterprise-application-Group-2
| Name| ID Number|
|---|---|
|BEAMLAK FEKADU|UGR/8928/15|
|BELEN BERHANU|UGR/0379/15|
|BEMNET ASSEGED|UGR/2591/15|
|ETSUB NADEW|UGR/4283/15|
|HASET WENDESEN|UGR/4331/15|

# Project Proposal for Enterprise Application
# Smart Agriculture Advisory and Farm Management System
## 1. Problem Statement
Smallholder farmers often struggle with limited access to timely agricultural information, including planting schedules, soil needs, crop health, climate patterns, fertilizer use, and pest control.
 This leads to low yields, incorrect farming practices, and financial losses.
 Agricultural extension services are not always accessible or responsive, creating a large information gap between experts and farmers.
Our Smart Agriculture Advisory and Farm Management System aims to bridge this gap using domain-driven design principles. The system centralizes farm data, crop cycles, and advisory recommendations, while enabling asynchronous event-driven coordination between services.
 The platform supports farmers, agronomists, cooperatives, and agricultural officers by providing updated insights, smart recommendations, and automated farm operations workflows.

## 2. Subdomain Decomposition (Strategic DDD)
### Core Subdomain
1. Advisory Subdomain (Core)
Generates personalized farming recommendations (fertilizer schedules, irrigation, pest prevention).
Uses weather, soil, crop category, and farmer data to deliver actionable insights.
High business value because it directly impacts farming outcomes and productivity.
### Supporting Subdomains  
2. Farm Management Subdomain (Supporting)
Stores farm profiles, farmer details, field locations, and soil characteristics.
Provides baseline data that the Advisory subdomain depends on for decision support. 
3. Crop Lifecycle Subdomain (Supporting)
Manages planting cycles, seed types, growth stages, and yield outcomes.
Supports the Advisory domain by triggering events at key growth milestones.
### Generic Subdomains  
4. Notification Subdomain (Generic)
Sends SMS, push notifications, or email reminders to farmers.
Generic because any system could use it (not agriculture-specific).    
5. Identity & Access Management (Generic)  
Handled entirely by Keycloak for authentication, SSO, and RBAC.
Generic across all enterprise systems.
## 3. Bounded Contexts (at least 3 within the Core Domain)
A. Farm Context  
Responsibility:  
Manages farmer profiles, farm fields, soil type, and geo-coordinates.  
Acts as the authoritative source for farm-related data.  
B. Crop Context  
Responsibility:  
Handles crop fields, planting events, growth stages, and harvest results.  
Defines aggregates such as CropCycle, GrowthStage, and YieldRecord.  


C. Advisory Context (Core Context)  
Responsibility:  
Generates recommendations (irrigation frequency, fertilizer schedule, pest control).  


Consumes events from Farm and Crop contexts.  


Publishes advisory events to Notification context.  



## 4. Ubiquitous Language (short summary)
- Farm — a physical plot of land owned or managed by a user.
- Crop Cycle — a specific planting-to-harvest lifecycle for a crop.
- Growth Stage — predefined phases of crop development.
- Recommendation — system-generated farming advice.
- Soil Profile — nutrient and moisture characteristics of a farm field.
- Advisory Trigger — an event when new advice should be generated (e.g., planting, rain forecast).
- Yield Record — harvested amount of a crop cycle.



## 5. User Stories Across Multiple Bounded Contexts
### User Story 1 — Automated Planting Advisory
As a farmer, when I register a new crop cycle, I want the system to generate initial planting recommendations.
Contexts involved:
Crop Context
Advisory Context
Published Event:
CropCycleStarted
Subscribed Event:
Advisory Context subscribes to CropCycleStarted
Eventual Consistency:
Crop Context publishes event → RabbitMQ
Advisory Context asynchronously processes event and generates initial recommendations
Farmers receive results later (no synchronous dependency)



### User Story 2 — Irrigation Schedule Update Using Weather Data
As an agronomist, I want irrigation recommendations to update automatically when weather forecasts change.
Contexts involved:
Farm Context (owns field & soil data)
Advisory Context
External Weather Service (not a full BC, but an integration point)
Published Event:
WeatherForecastUpdated
Subscribed Event:
Advisory Context
Eventual Consistency:
Weather service publishes event → RabbitMQ
Advisory Context recalculates irrigation schedule
Updates appear asynchronously in the farmer’s dashboard



#### User Story 3 — Pest Outbreak Alert
As a farmer, I want to be notified if nearby farms report pest infestation so I can take preventive action.
Contexts involved:
Farm Context
Crop Context
Advisory Context
Notification Context
Published Event:
PestDetected
Subscribed Events:
Advisory Context → generates advisory
Notification Context → sends alerts
Eventual Consistency:
Crop Context detects pest → publishes PestDetected
Advisory Context generates recommended prevention measures
Notification Context sends alerts to nearby farms
All updates propagate through asynchronous messaging



## 6. AI/ML Feature 
### AI-Driven Crop Yield Prediction
We will integrate a small ML model that predicts crop yield based on:
- Soil characteristics
- Weather patterns
- Crop type
- Planting date
- Historical yield patterns  
### Purpose:
 To help farmers plan harvest expectations, manage resources, and estimate profitability.
### Integration:
Implemented as a small ML microservice or library.  
Advisory Context will call this service to provide predicted yield inside advisory reports.



## 7. Summary & Fit With Course Requirements
- Strategic DDD:
 Clear subdomain classification, bounded contexts, and ubiquitous language.
- Tactical DDD:
 Each BC will define aggregates (e.g., Farm, CropCycle), domain services, domain events, and repositories (POCO entities).
- Modular Monolith (Phase 1):
 Farm, Crop, Advisory, Notifications are separate internal modules.
- Microservices (Phase 2):
 Each BC becomes independent: Farm Service, Crop Service, Advisory Service, Notification Service.
- Event-Driven Architecture:
 Uses RabbitMQ/Kafka for domain event publication with the Outbox Pattern.
- AI Integration:
 Yield prediction fits naturally in the Advisory Context.
- Security:
 Keycloak handles SSO + RBAC across services and the Angular front-end.
