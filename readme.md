# Keap REST v2 API - Missing & Incomplete Features

This document outlines functionality available in XML-RPC/REST v1 that is **missing or limited** in REST v2.

## Purpose
Document missing, incomplete, or inefficiently implemented features between XML-RPC/v1 and REST v2 to communicate requirements to the Keap API team to ensure feature parity exists prior to sunsetting older APIs.

## Contributing
We need your help! If you identify additional missing features or limitations in REST v2, please contribute by submitting a pull request to this repository with the details.

If you've identified an issue but are unsure of how to contribute, please open an issue so we can discuss and document it.

## Key Points

- The XML-RPC API provided a robust Data Service endpoint that allowed for fetching data from various tables with flexible querying capabilities.
- The following features were part of the Data Service that was widely used and are currently not implemented in REST v2 for any endpoint:
  - Counting records based on a filter
  - Filtering with SQL-like syntax (e.g., `%name%`)
  - Pagination with limits and offsets vs token-based pagination to allow for asynchronous fetching
  - Filtering/limiting by an array of identifiers
  - Specifying which columns to return to reduce payload size

---

## XML-RPC Examples Missing from REST v2
Below is a list of specific features from various applications that relied on the various XML-RPC services that are not yet implemented,
under-implemented, or have significant limitations in REST v2.

### 1. Contact Count (`DataService.count` - Any Table)
**XML-RPC Capability:**
- `data()->count('Contact', ['Id' => '%'])` - Returns total contact count in a single call

**Current REST v2 Solution:**
- ❌ No endpoint to retrieve total contact count without pagination

**Workaround:** Paginate through all contacts to count them

**Impact:** Performance degradation, unnecessary API calls

---

### 2. Tag Application Dates (`DataService.query` - ContactGroupAssign Table)
**XML-RPC Capability:**
- `data()->query('ContactGroupAssign', ...)` with `['Contact.Id' => $contactId]`
- Returns `DateCreated` for all tags applied to a single contact in **one call**

**Current REST v2 Solution:**
- ❌ No endpoint to retrieve tag application dates directly
  - **Workaround:** Must make 1 call to get contact tags, then 1 call per each of contact's tags to "List Tagged Contacts" endpoint for each tag. **N+1 Problem**
- Additional problem: ❌ Cannot filter "List Tagged Contacts" by contact ID

**Impact:** Severe performance degradation (1 call → N+1 calls where N = number of tags)

---

### 3. Bulk Email Engagement Data (`DataService.query` - EmailAddStatus Table)
**XML-RPC Capability:**
- `data()->query('EmailAddStatus', ...)` - Returns bulk engagement data including:
  - Opt status (`Type`)
  - Last email sent date (`LastSentDate`)
  - Last open date (`LastOpenDate`)
  - Last click date (`LastClickDate`)
  - Date created (`DateCreated`)

**Current REST v2 Solution:**
- ⚠️ "Retrieve an Email Address Status" endpoint only returns opt status for **single contact lookups**
- ❌ Missing fields: `LastClickDate`, `LastOpenDate`, `LastSentDate`, `DateCreated`

**Workaround:** ❌ None

**Impact:** (*Assuming endpoint can be modified to return needed data*) Severe performance degradation (1 call per contact, instead of 1 call per 1,000 contacts)

---

### 4. Custom Field Group Names (`DataService.query` - DataFormGroup Table)
**XML-RPC Capability:**
- `data()->query('DataFormGroup', ...)` - Returns custom field group metadata:
  - `Id`
  - `TabId`
  - `Name` (group name)

**Current REST v2 Solution:**
- Contact model endpoint returns custom fields data with `group_id`
- ❌ No way to retrieve the **name** of the custom field group
- Can only see numeric group ID

**Workaround:** ❌ None

**Impact:** Cannot display human-readable group names in UI

---

### 5. Tag Search with LIKE (`DataService.query` - ContactGroup Table)
**XML-RPC Capability:**
- `data()->query('ContactGroup', ...)` with `['GroupName' => '%search%']`
- LIKE searching to find tags containing a word or phrase

- Retrieving tags endpoint only supports exact name matches when filtering

**Workaround:** Must paginate through all tags and manually filter client-side

**Impact:** Reduced search functionality for tag discovery

---

### 6. Fetch Specific Tags by IDs (`DataService.query` - ContactGroup Table - IN Query)
**XML-RPC Capability:**
- `data()->query('ContactGroup', ...)` with `['Id' => [1, 5, 10, 25]]`
- IN searching to fetch only specific tags by ID array

**Current REST v2 Solution:**
- Retrieving tags endpoint only supports limited fields to filter by

**Workaround:** Must paginate through all tags and manually filter client-side

**Impact:** Inefficient data retrieval when only specific tags are needed

---

### 7. Tag Contact Count (`DataService.count` - ContactGroupAssign Table)
**XML-RPC Capability:**
- `data()->count('ContactGroupAssign', ['GroupId' => $tagId])`
- Returns count of contacts with a specific tag in one call

**Current REST v2 Solution:**
- ❌ No endpoint to count contacts for a tag

**Workaround:** Must paginate through all contacts for a specific tag and count them

**Impact:** Performance degradation for tag analytics

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

