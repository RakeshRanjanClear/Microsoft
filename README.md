# MicrosoftELT

This repository contains two Microsoft Dynamics 365 Finance & Operations (D365 F&O) ISV solution packages, exported as `.axpp` project archives, developed by **Clear (ClearTax)**.

| File | Model Name | Publisher | Layer |
|---|---|---|---|
| `ClearTaxISV_ELT.axpp` | `ClearELT` | Clear | 12 (ISV) |
| `ClearTaxISV_ETL.axpp` | `Clear_OmniInvoice_Core` | Acumant | 8 (ISV) |

---

## 1. ClearELT — Generic Data Extraction & Posting Framework

**Model:** `ClearELT`
**Purpose:** A reusable ELT (Extract-Load-Transform) engine embedded in D365 F&O for extracting data out of the application, staging/uploading it externally (e.g. to S3), and posting/writing results back in — independent of any single business process.

### Key components
- **Query engine** — `ELTSysDaQueryBuilder`, `ELTQueryBuilder_I`, `ELTQueryExecutor`, and data contracts (`ELTSelectDataContract`, `ELTWhereDataContract`, `ELTJoinDataContract`, `ELTOrderByDataContract`, `ELTColumnDataContract`, etc.) — build and execute dynamic queries against AX tables from external configuration.
- **Data extraction & posting** — `ELTDataExtraction`, `ELTDataPosting`, `ELTDataPostingService`, `ELTDataPostingHelper`, `ELTInsertTableHandler` — extract records and post/insert data back into F&O tables.
- **S3 integration** — `ELTS3Uploader`, `ELTS3UploaderSync`, `ELTPresignedURLGenerator` — upload extracted data/files to AWS S3 via pre-signed URLs.
- **Ingestion & triggers** — `ELTTriggerIngestion`, `ELTCommandFetcherSync`, `ELTIngestionMetadataDataContract` — trigger and fetch ingestion commands from an external orchestrator.
- **Telemetry & cleanup** — `ELTTelemetryLogger`, `ELTTelemetryUploader`, `ELTTelemetryLogsUploaderService/Controller`, `ELTCleanUpService`/`ELTCleanUpController` — logging and periodic cleanup batch jobs.
- **Authentication & config** — `ELTAuthenticationDataContract`, `ELTConfigParser`, `ELTConfigParserManual`, `ELTContextHandler`.
- **API surface** — `ELTAPIHandler`, `ELTIntegrationService`, `ELTIntegrationServiceSyncHelper`, `ELTTestConnection`.

### Tables
`ELTAPILogs`, `ELTAuthenticationDetails`, `ELTBaseResponse`, `ELTDataPostingFailureLogs`, `ELTDataPostingSuccessLogs`, `ELTIRN_WRITEBACK`, `ELTParameterStaging`, `ELTParameters`, `ELTQueryResults`, `ELTTelemetryLogs`, `ELTTelemetryLogsArchive`

### Forms
`ELTAPILogs`, `ELTParameters`, `ELTTelemetryLogs`, `ELTTelemetryLogsArchive`, `ELTWriteBack`

### Security
Security role `ELTMaintain` with privileges `ELTParameterView`, `ELTParameterMaintain`, `ELTMaintain`, exposed under a `System administration` menu extension (`ClearELT`).

---

## 2. Clear.OmniInvoice.Core (ETL) — e-Invoicing Integration

**Model:** `Clear_OmniInvoice_Core`
**Publisher:** Acumant
**Description (from model metadata):** *"ISV-layer framework providing reusable, extensible, and upgrade-safe components for e-Invoicing integration."*

**Purpose:** Generates, submits, and tracks e-invoices (and related documents) from D365 F&O sales/purchase transactions against the ClearTax e-invoicing API, including status tracking, batch processing, email notifications, and reporting.

### Key components
- **Payload generation** — `ClearTaxGenerateEInvoiceAPIPayload`, `ClearTaxGenerateEInvoiceAPIPayloadBase`, `ClearTaxGenerateResponseContract`, `ClearTaxCustomFieldsContract`.
- **API integration** — `ClearTaxEInvoiceAPIHelper`, `ClearTaxGetDocumentStatusAPIHelper`, `ClearTaxDocumentResponseContract`, `Elteinvoicestatusresponsecontract` (status check response), `Elteinvoicestatuscheckbatch` (batch job for polling e-invoice status).
- **Batch processing** — `ClearTaxEInvoiceBatchService`/`Controller`/`Contract`, `ClearTaxEInvoiceDayWiseBatchService`/`Controller`/`Contract` — bulk/day-wise e-invoice generation batches.
- **Email notifications** — `ClearTaxEInvoiceEmailBuilder`, `ClearTaxEInvoiceEmailProcessor`, tables `ClearTaxEInvoiceDayWiseEmailQueue`, `ClearTaxEInvoiceEmailResponseLog`, form `ClearTaxEInvoiceEmailResponseLogForm`.
- **Printing** — `ClearTaxEInvoicePrint` (invoice print integration, likely embedding QR code/IRN).
- **Reporting** — `ClearTaxEInvoiceDailySummary` (SSRS report) with RDP class `ClearTaxEInvoiceDailySummaryRDP` and helper `ClearTaxEInvoiceDailySummaryHelper`.
- **Event handlers** — `ClearTaxCustInvoiceJourEventHandler`, `SalesTableForm_Clear_OmniInvoice_Core_EventHandler`, plus extensions on `CustInvoiceJournal`, `CustInvoiceJour`, `SalesTable`, `SalesLine`, `TaxTable`, `CustInvoiceTable`, `CustInvoiceTrans`, `ProjProposalJour`, `LogisticsPostalAddress`, `TaxItemGroupHeading` — hook e-invoice generation into standard sales/AR posting flows.

### Tables
`ClearTaxAllowanceReasonCodes`, `ClearTaxEInvoiceAPISetUpTable`, `ClearTaxEInvoiceDailySummaryTmp`, `ClearTaxEInvoiceDayWiseEmailQueue`, `ClearTaxEInvoiceEmailResponseLog`, `ClearTaxEInvoicePEmailSetupEnablement`, `ClearTaxEInvoiceParameterEmailSentTypeEnablement`, `ClearTaxEInvoiceParameterTable`, `ClearTaxEInvoicePurchaseInvoiceHeader`, `ClearTaxEInvoicePurchaseInvoiceLines`, `ClearTaxEInvoiceResponseTable`, `ClearTaxInvoiceAddressTmp`, `ClearTaxInvoiceLinesTmp`

### Forms / Workspaces
`CLEInvWorkspaceV2`, `ClearTaxEinvoiceWorkspace`, `ClearTaxAPISetupForm`, `ClearTaxAllowanceReasonCode`, `ClearTaxEInvoiceParameterForm`, `ClearTaxEInvoiceResponseForm`, `ClearTaxEInvoiceEmailResponseLogForm`

### Enumerations
`ClearTaxEInvoiceEmailSentTypeBase`, `ClearTaxEInvoiceLineType`, `ClearTaxRequestAPIType`, `ClearTaxReqType`, `ClearTaxEmailQueueStatus`, `ClearTaxEInvoiceInvoiceSource`, `ClearTaxEInvoiceValidationsSuccess`, `ClearTaxEmailLogStatus`, `ClearTaxEinvoiceType`

---

## Repository Structure

Each `.axpp` file is a zipped Visual Studio D365 F&O project export and unpacks into the standard model layout:

```
<ModelName>/
├── Model/<ModelName>.xml          # Model descriptor (name, publisher, layer, module refs)
├── Project/<ProjectName>.rnrproj  # Visual Studio project file
└── ProjectItem/
    ├── AxClass/                   # X++ classes
    ├── AxTable/                   # Tables
    ├── AxTableExtension/          # Table extensions on standard AX tables
    ├── AxForm/ AxFormExtension/   # Forms and form extensions
    ├── AxEnum/                    # Base enums
    ├── AxEdt/                     # Extended data types
    ├── AxMenu/ AxMenuExtension/   # Menus
    ├── AxMenuItemAction/ AxMenuItemDisplay/
    ├── AxSecurityRole/ AxSecurityPrivilege/
    ├── AxReport/                  # SSRS reports
    ├── AxTile/                    # Workspace tiles
    └── AxLabelFile/               # Label resources
```

## How to Import

1. Open Visual Studio with the D365 F&O development tools installed on the target environment.
2. Use **Dynamics 365 → Import model** (or unzip the `.axpp` and open the `.rnrproj` file directly) to load the project.
3. Build and synchronize the database.
4. Deploy/promote through your standard DEV → UAT → PROD pipeline.

## Notes

- Both models are ISV (custom) layer solutions (`ClearELT` at layer 12, `Clear_OmniInvoice_Core` at layer 8) and depend on standard modules including `ApplicationSuite`, `Tax`, `GeneralLedger`/`Ledger`, `Dimensions`, and `SourceDocumentation`.
- `ClearELT` is a general-purpose data extraction/posting/telemetry framework and does not itself contain e-invoicing logic.
- `Clear_OmniInvoice_Core` (the ETL package) contains the ClearTax e-invoicing business logic — payload generation, API submission, status polling, and email/report notifications — and extends standard sales/purchase/tax tables to hook into it.
