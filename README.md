# healthkit-notion-etl

An end-to-end serverless data pipeline for biometric telemetry, bridging iOS (Apple HealthKit) and the Notion API through a middleware orchestration layer utilizing HTTP webhooks, dynamic API routing, and strict data type validation.

## 🛠️ System Architecture

The project is structured as a decentralized ETL (Extract, Transform, Load) architecture:

1. **Client Layer (iOS Shortcuts)**: Extracts raw biometric samples from **Apple HealthKit**, serializes the fields, and pushes a clean JSON payload via an HTTP POST request.
2. **Middleware Layer (Make.com)**: Acts as an API gateway and orchestration engine, catching the webhook and executing live routing conditions.
3. **Storage Layer (Notion Database)**: A strictly-typed structural database that acts as the final data warehouse for visualization and macro-nutrient computation.

---

## 📱 Component Breakdown

### 1. iOS Client (The Shortcut)
The client-side architecture relies on the native Apple HealthKit framework to aggregate daily biometric telemetry and push it to the remote backend pipeline.
* **Production Template**: You can review and deploy the automated tracking client directly here: [Download iOS Shortcut Template](https://www.icloud.com/shortcuts/b138d4a3a4b04ebaabb6f96f678b2856)
* **Data Extraction**: Interrogates the local device database daily for metrics including `Active Energy`, `Steps`, `Dietary Protein`, `Dietary Carbohydrates`, and `Dietary Fat`.
* **Data Transformation**: Includes localized text parsing components to strip unit strings (e.g., converting `"75 g"` to a raw numeric `75`) to prevent data-type mismatches before transmitting.

### 2. Middleware Orchestration Engine (Make.com)
The middleware manages the data flow using custom conditional logic (**Upsert Routine**):
* **Idempotency Check**: Queries the Notion database using a dynamic date string (`DD.MM.YYYY`) to inspect if a log for the current day already exists.
* **Dynamic Routing**: 
  * **Branch A (Update)**: If a matching record is located, it captures its unique `Page ID` and issues an HTTP `PATCH` request to update existing values, allowing multiple triggers per day without data duplication.
  * **Branch B (Create)**: If no record matches, it issues an HTTP `POST` request to initialize a brand-new daily entry.

### 3. Database Layer (Notion)
The schema acts as a relational health ledger. It features hard validation constraints for numerical properties (`Steps`, `Weight`, `Proteine`, `Carbohidrati`, `Grasimi`).
* **Automated Caloric Engine**: To offload processing from the middleware, a local database **Formula** property automatically computes total daily ingested calories using standard thermodynamic macronutrient coefficients:
  ```text
  (prop("Proteine") * 4) + (prop("Carbohidrati") * 4) + (prop("Grasimi") * 9)