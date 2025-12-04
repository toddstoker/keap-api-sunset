# Keap REST v2 API - Missing & Incomplete Features

This document outlines functionality available in XML-RPC/REST v1 that is **missing or limited** in REST v2.

## Purpose
Document missing, incomplete, or inefficiently implemented features between XML-RPC/v1 and REST v2 to communicate requirements to the Keap API team to ensure feature parity exists prior to sunsetting older APIs.

> [!IMPORTANT]
> ## Contributing
> We need your help! If you identify additional missing features or limitations in REST v2, please contribute by submitting a pull request to this repository with the details.
>
> If you've identified an issue but are unsure of how to contribute, please open an issue so we can discuss and document it.

## Summary

- The XML-RPC API provided a robust Data Service endpoint that allowed for fetching data from various tables with flexible querying capabilities.
- The following features were part of the Data Service that was widely used and are currently not implemented in REST v2 for any endpoint:
    - Counting records based on a filter
    - Filtering with SQL-like syntax (e.g., `%name%`)
    - Pagination with limits and offsets vs token-based pagination to allow for asynchronous fetching
    - Filtering/limiting by an array of identifiers
    - Filtering using "NOT EQUALS" (`~<>~`)
    - Specifying which columns to return to reduce payload size
- There are a handful of specific endpoints that are currently in use that are intentionally not being ported but the reasoning/migration path forward is unclear
- Some new REST v2 endpoints are not efficient requiring multiple API calls (sometimes up to 1000 api calls) to achieve what was possible in a single XML-RPC call

---

## Specific XML-RPC endpoints not available (or lacking some functionality) in REST v2:

### `FunnelService.achieveGoal`
- **Does not exist**
- In [GapAnalysisJustifcations.xlsx](https://thryv.sharepoint.com/:x:/t/KeapGryffindor/EUYJ7---f_JEvRt4YKzyQwcBa3x_vpivPai1GEw2HOC3EA?e=9EHDtO): 
  - "Achieve a goal" is listed under "Campaign sequence". Maybe not the same?
  - Reason: "Was not a part of V1, out dated and bug inducing"
  - This still exists in-app, so how will goals be triggered going forward?

### `ContactService.addWithDupCheck`
- Can be completed with 2 calls (search + create if not found)

### `ContactService.runActionSequence`
- Maybe "[Add Contact to an automation sequence](https://developer.infusionsoft.com/docs/restv2/#tag/Automation/operation/addContactsToAutomationSequenceUsingPOST)"? 
    - Requires `automation_id` & `sequence_id` instead of *only* `actionSetId`
- In [GapAnalysisJustifcations.xlsx](https://thryv.sharepoint.com/:x:/t/KeapGryffindor/EUYJ7---f_JEvRt4YKzyQwcBa3x_vpivPai1GEw2HOC3EA?e=9EHDtO):
  - "Run Action" is listed under "Campaign sequence". Maybe not the same?
  - Reason: "Was not a part of V1, out dated and bug inducing"
      
### `SearchService.getSavedSearchResultsAllFields`
- [Looks like it might exist, but docs indicate it is deprecated](https://developer.infusionsoft.com/docs/restv2/#tag/Reporting/operation/runReportUsingPOST)
- From developer forum post:
  - [Deprecated reporting endpoints: These provide legacy Saved Search reports in REST format, rather than a full new reporting solution. We won’t extend them beyond v2, but we’ll make sure a new reporting solution is in place before retiring them.](https://integration.keap.com/t/rest-v2-improvements-request/93810/2)
  - So is the advice to migrate from XML-RPC to REST v2 Reporting right now, or wait until new reporting endpoints are released?
  
### `DataService.getAppSetting`
- Module `ContactAction` & setting `optionstype` to get all Note Types
  - [Get Application Configuration](https://developer.infusionsoft.com/docs/restv2/#tag/Settings/operation/getApplicationConfigurationsUsingGET) exists, but maybe `ContactAction` module is now split into `appointments`, `tasks`, and `notes` modules?

## DataService tables without REST v2 endpoints (or lacking functionality):

### `Contact`
- Get multiple specific contacts by IDs
  - Contacts endpoint exists, but cannot filter by multiple IDs
- Count total/filtered contacts
  - Contacts endpoint exists, but no way to get total count without pagination and loading every contact

### `ContactGroup`
- Tag Search with LIKE (`%search%`)
    - Contact Groups endpoint exists, but only supports exact name matches when filtering
- Fetch specific tags by IDs
    - Contact Groups endpoint exists, but cannot filter by multiple IDs

### `ContactGroupAssign`
- Get DateCreated for a contact & tag
    - Tags endpoint exists, but cannot filter by a contact id, only email
- Get count of contacts for a tag
    - Tags endpoint exists, but must iterate through all contacts to count them
- Get specific tags by IDs
    - Tags endpoint exists, but cannot filter by multiple IDs

### `CreditCard`
- Creating a new entry
  - Payment Methods endpoints exist, but creation endpoint does not
  - In [GapAnalysisJustifcations.xlsx](https://thryv.sharepoint.com/:x:/t/KeapGryffindor/EUYJ7---f_JEvRt4YKzyQwcBa3x_vpivPai1GEw2HOC3EA?e=9EHDtO):
    - Reason: "CreditCard deprecated and phasing out in V2, so no longer supported."
    - What is the alternative for storing payment methods going forward?

### `DataFormField`
- Get Custom Fields for Companies (FormId = -6)
  - **Does not exist**
- Get Custom Fields for Opportunities (FormId = -4)
  - **Does not exist**

### `DataFormGroup`
- Get Custom Field group `Name`
  - Can get group ids by retrieving contact model, but group names are not included
  - Use case: Organize custom fields in UI for grouped display
- Filter based on DataFormTab IDs
  - **Does not exist**

### `DataFormTab`
- Get Custom Field tabs
  - **Does not exist**
  - Use case: Organize custom fields in UI for grouped display
- Filter Tabs based on type: Contact vs Company vs Opportunity
  - **Does not exist**

### `EmailAddStatus`
- Retrieving `LastClickDate`, `LastOpenDate`, `LastSentDate` (no filtering)
  - Endpoint exists to get opt status only
- Retrieve multiple records (1,000 per call) at once
  - Endpoint only supports single contact lookups

### `Job`
- Query by `JobTitle`
  - Orders endpoint exists, but cannot query by title

### `Lead`
- Find by contact ID
  - Opportunities endpoint exists, but cannot filter by contact ID
- Filter based on title
  - Opportunities endpoint exists, but cannot filter by opportunity title
- Exclude based on User ID
  - No endpoints exist that allow filtering using `~<>~` operators

### `Referral`
- Query by single or multiple ContactIds
  - **Does not exist**

### `SavedFilter`
- List all saved searches
  - Might exist, but docs indicate it is deprecated
  - See [SearchService.getSavedSearchResultsAllFields](#searchservicegetsavedsearchresultsallfields) above

---

## REST v1 Endpoints Missing from REST v2

**Note:** These features are available in REST v1 but missing in REST v2. The focus is on prioritizing features/methods/calls that are disappearing with XML-RPC deprecation.
These are all **lower priority** since v1 is not being sunset at this time, but should be considered for future parity.

### 1. Custom Field Creation
**REST v1 Capability:**
- `POST /rest/v1/contacts/model/customFields` - Create custom fields programmatically

**Current REST v2 Solution:**
- ❌ No endpoint exists

---

### 2. Resthook Management (Webhooks)
**REST v1 Capability:**
- `GET /rest/v1/hooks` - List all webhooks
- `POST /rest/v1/hooks` - Create webhook
- `GET /rest/v1/hooks/{id}` - Get webhook details
- `PUT/PATCH /rest/v1/hooks/{id}` - Update webhook
- `DELETE /rest/v1/hooks/{id}` - Delete webhook
- `POST /rest/v1/hooks/{id}/verify` - Verify webhook

**Current REST v2 Solution:**
- ❌ No endpoint exists

---

### 3. Bulk Tag Operations for Single Contact
**REST v1 Capability:**
- `POST /rest/v1/contacts/{id}/tags` - Apply multiple tags to one contact in single call
- `DELETE /rest/v1/contacts/{id}/tags` - Remove multiple tags from one contact in single call

**Current REST v2 Solution:**
- ❌ v2 only supports single tag operations per contact
- `POST /rest/v2/tags/{tagId}/contacts:applyTags` - Apply one tag to multiple contacts
- `POST /rest/v2/tags/{tagId}/contacts:removeTags` - Remove one tag from multiple contacts
- Inverse operation not available

---
