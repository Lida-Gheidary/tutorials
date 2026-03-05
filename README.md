# Reading the Data Layer via Browser Console

This method uses the browser's **Console** to read the `digitalData` object directly on the page, instead of relying solely on AEP Tags "Data Elements" to discover and map available data.
---
## What This Method Does
The website exposes a JavaScript object (the **data layer**), often called `digitalData`, containing information about the page, user, products, etc.
- In **DevTools → Console**, type `digitalData` (or a deeper path like `digitalData.page.pageInfo.pageName`) to see current values
- Use those same paths in **AEP Data Collection (Tags)** — in Data Elements, XDM objects, and Rules — to send data to AEP / Analytics
- The Console is a **read-only window** to inspect what's already on the page; you copy correct paths into Tags configuration, you don't change data there
---
## Where in AEP / Tags You Use It
### Data Elements Tab
Create data elements that read from `digitalData`:
- **Type:** `JavaScript Variable` or `Custom Code`
- **Path:** `digitalData.page.pageInfo.pageName`
- 
### Extensions → Adobe Experience Platform Web SDK
Configure an **XDM Object** data element that uses those Data Elements to build the XDM payload.
### Rules Tab
In a `Page View` or `Custom Event` rule, set an action like **Send Event** (Web SDK) and attach your XDM Object data element.
### Optional: Datastream → Data Prep (AEP UI, not Tags)
Map incoming XDM fields to Analytics dimensions — e.g. `web.webPageDetails.name` becomes your **Page** dimension automatically.
> **Note:** You use the Console only to verify data and learn correct paths, then configure everything inside these tabs.
---
## Typical Tasks

### 1. Check That the Data Layer Exists
Open DevTools → Console and run:
```js
digitalData
```
If it returns an object (not `undefined`), the data layer is present.

### 2. Find the Exact Path to a Value
```js
digitalData.page.pageInfo
// Inspect keys like pageName, language, etc.
```
Copy the path (e.g. `digitalData.page.pageInfo.pageName`) to use in a Data Element.

### 3. Validate Values Before Building Tags
Change the page, refresh, and re-run:
```js
digitalData.page.pageInfo.pageName
```
This confirms the value updates correctly before you configure anything in Tags.

### 4. Use the Path in Tags
- **Data Element name:** `de_pageName`
- **Type:** JavaScript Variable
- **Path:** `digitalData.page.pageInfo.pageName`

In an XDM Object data element:
```
web.webPageDetails.name = %de_pageName%
```
In a Rule, use that XDM Object when calling `sendEvent`.
---
## Example: What You See in Console

Suppose the site has this data layer:

```js
window.digitalData = {
  page: {
    pageInfo: {
      pageName: "PLP - Women Shoes",
      pageType: "product-list",
      language: "en-GB"
    }
  },
  user: {
    profile: {
      profileID: "user_12345",
      loginStatus: "logged-in"
    }
  }
};
```

### Console Commands & Outputs

**Command:**
```js
digitalData
```
**Output:** expandable object showing `page`, `user`, etc.

---

**Command:**
```js
digitalData.page.pageInfo
```
**Output:**
```js
{
  pageName: "PLP - Women Shoes",
  pageType: "product-list",
  language: "en-GB"
}
```

---

**Command:**
```js
digitalData.page.pageInfo.pageName
```
**Output:**
```js
"PLP - Women Shoes"
```

---

**Full JSON dump** (handy for documentation and test design):
```js
JSON.stringify(window.digitalData, null, 2)
```
Returns a formatted JSON string of the entire data layer — useful for quickly documenting what's available and designing Data Elements and XDM mappings from it.

---

## When to Use These Console Checks

- **Designing the tracking spec** — confirm the site exposes required fields in `digitalData`
- **Creating or debugging Data Elements** — verify the path in Console first, then configure the element
- **Troubleshooting missing Analytics values:**
  1. Check if `digitalData` has the value in Console
  2. If **yes** → debug Tags / Rules / XDM mapping
  3. If **no** → it's a site/data layer issue, not a Tags issue




----------------------------- DigitalData completion
To get a comprehensive list of all available digitalData paths and values, systematically inspect across 10-15 key pages representing your site's main templates — this covers 95% of scenarios without overkill.

Pages to check (prioritize these)
1 homepage
3-5 product pages (different categories)
2 category/listing pages
1 search results page
1 cart page
1 checkout/thank-you page
1 login/register page
1 logged-in profile page
1 error/404 page
2 dynamic pages (e.g. PDP with variants, blog post)

Step-by-step process (30-60 min total)
Prep a Google Sheet or Notion table (columns: Page URL, Command, Path, Value, Notes/Issues).

For each page:
Visit → F12 → Console → clear console (Ctrl+L).
Run 4 core commands (copy-paste one by one):
text
digitalData
text
digitalData.page.pageInfo
text
digitalData.user.profile
text
digitalData.events
If events exist (e.g. add to cart), trigger the action and re-run digitalData.events.

Expand and document:
Right-click objects → Copy object (or use JSON.stringify(digitalData, null, 2)).
Paste into your sheet → note every key/value under the path (e.g. digitalData.page.pageInfo.pageName = "PLP - Shoes").
Flag missing data (e.g. digitalData.user = undefined).
Cross-check: Refresh page, navigate away/return, confirm values update correctly.
Total commands per page: 4-6 (core 4 + events + stringify if complex).

How to store the comprehensive list
Option 1: Google Sheet (best for sharing/team audits)
Page URL	Command/Path	Full Value	Available Keys	Issues
https://site.com/home	digitalData.page.pageInfo.pageName	"Homepage"	pageName, pageType, language	None
https://site.com/pdp/shoes	digitalData.product[0].productInfo.productName	"Nike Air Max"	productName, price, SKU	price missing
Option 2: Single JSON file (for dev handoff)

Run JSON.stringify(window.digitalData, null, 2) on each page → save as data-layer-[page-type].json.
Or combine into one master file with page metadata.

Option 3: Markdown/Notion (for docs)
text
## Homepage
**digitalData.page.pageInfo**
- pageName: "Homepage"
- pageType: "home"

**digitalData.user.profile**
- profileID: undefined
This becomes your tracking spec blueprint — directly copy paths to create Data Elements in AEP Tags (e.g. %digitalData.page.pageInfo.pageName%). Re-run monthly or post-site changes

--------------------------------
XDM, ExperienceEvent and schemas
XDM system overview (why standard schemas, field groups, tenant fields):
https://experienceleague.adobe.com/en/docs/experience-platform/xdm/home

​XDM ExperienceEvent class (event-centric schemas you’ll use for Web SDK hits):
https://experienceleague.adobe.com/en/docs/experience-platform/xdm/classes/experienceevent
​

Web SDK, datastreams and overall flow
Implement Adobe Experience Cloud with Web SDK tutorial (tags property, datastream, schema, AA + AEP + CJA through Edge):
https://experienceleague.adobe.com/en/docs/platform-learn/implement-web-sdk/overview
​

Web SDK overview (library, Edge Network, extension in Tags):
https://experienceleague.adobe.com/en/docs/target-dev/developer/client-side/aep/aep-web-sdk-overview

Mapping XDM/data to Adobe Analytics
XDM object field mapping to Adobe Analytics (automatic mapping via AA ExperienceEvent Template, contextData, processing rules):
https://experienceleague.adobe.com/en/docs/analytics/implementation/aep-edge/xdm-var-mapping
​

Data object field mapping to Adobe Analytics (how Edge prioritizes XDM vs data object when ingesting into AA):
https://experienceleague.adobe.com/en/docs/analytics/implementation/aep-edge/data-var-mapping
​

Mapping AA fields into XDM / CJA
Mapping fields for the Adobe Analytics source connector (AA → XDM mapping concepts, useful for thinking in reverse when you do AA vars → XDM):
https://experienceleague.adobe.com/en/docs/experience-platform/sources/connectors/adobe-applications/mapping/analytics
​

Conceptual XDM system support
XDM system overview on GitHub (same concepts, more technical detail about classes, field groups, tenant namespaces):
https://github.com/AdobeDocs/experience-platform.en/blob/main/help/xdm/home.md

Adobe Experience Platform Web SDK: Everything you need to know Acc
​

Various users still track digital behavior using Adobe Launch (tag management) and AppMeasurement. Data is sent server-side in a format tied to Adobe Analytics only.
The goal is to migrate to AEP Web SDK, where data is structured using XDM (Experience Data Model) — a standardized format that feeds Adobe Analytics today and enables Customer Journey Analytics (CJA) in the future. An edge/server-side configuration (Adobe-native or third-party) will also be part of the architecture.
The goal is to migrate this setup to Adobe Experience Platform Web SDK, where all data is structured using XDM (Experience Data Model) — a standardized format that feeds Adobe Analytics in the short term and enables Customer Journey Analytics (CJA) in the future. An edge/server-side configuration will also be part of the final architecture.

AEP Web SDK & XDM Schema Design — Phase 1
Phase 1 covers four areas:
Data Layer
Tags (AEP Tags / Launch)
XDM Schema Design
AEP Configuration


Task 1 — Audit Existing AA Variables
Review all conversion variables (eVars) and traffic variables (props) in the Adobe Analytics Report Suite Manager.
Record variable number, name, enabled/disabled status, and allocation setting
Note what each variable appears to track based on its name and description
Mark variables that are clearly disabled or legacy
Output: Variable inventory spreadsheet

Task 2 — Check Real Usage in Workspace
Validate which variables are actually collecting data by examining them in Analysis Workspace.
Drag each variable as a dimension against Page Views or Visits, last 90 days
Note whether values are populated or mostly "(not set)"
Record sample values to understand what kind of data each variable holds
Classify each variable: Active / Sparse / Legacy / Disabled
Output: Usage column added to the inventory spreadsheet

Task 3 — Group Variables by Business Domain
Organize all active variables into logical groups that will map to XDM field groups.
Assign each variable a category (e.g., Content, Search, User/Membership, Tags/Taxonomy, Navigation, Forms, Campaigns, Errors/Downloads)
Identify variables that belong together under the same schema field group
Flag variables that carry sensitive or identity-related data
Output: Grouped variable table

Task 4 — Review Tags Properties and Data Elements
Access AEP Data Collection and review the active Tag properties and their contents.
Identify which properties are actively maintained vs legacy (DTM-origin)
Determine which property/properties are the primary scope for this migration
Review the Data Elements list: note naming conventions and volume
Click into individual data elements to find the data element type and the exact source path (e.g., JavaScript variable, data layer path)
Cross-reference data elements with the AA variable inventory to find which data element feeds which eVar/prop
Review Rules briefly to understand when and how data is sent to AA today
Output: Summary of tag structure — active properties, data element sources, rule logic overview

Task 5 — Propose XDM Schema Mapping
Based on the variable groupings and data element sources, propose how the current setup maps to XDM.
For each variable group, identify the closest standard XDM field group from Adobe documentation
For variables with no standard match, propose a custom field under _tenant
Produce a mapping table:
| AA Variable                  | Group      | Data Element Source                 | XDM Field Path          | Standard or Custom |
| ---------------------------- | ---------- | ----------------------------------- | ----------------------- | ------------------ |
| eVarX – Page Name            | Navigation | JavaScript: digitalData.page.name   | web.webPageDetails.name | Standard           |
| eVarX – Internal Search Term | Search     | JavaScript: digitalData.search.term | web.search.query        | Standard           |
| eVarX – Tag X                | Tags       | JavaScript: digitalData.tags.X      | _tenant.tags.X          | Custom             |
Output: XDM mapping proposal table

Task 6 — Document AEP Configuration Logic
Describe (no build required) how the new architecture would work end to end.
Explain how the Web SDK sends XDM events to the Edge Network
Describe how the Datastream routes data to AA and to an AEP dataset
Note how XDM fields map back to AA variables via datastream rules, preserving existing reports
Note whether edge configuration is Adobe-native or involves a third-party endpoint
Output: Written configuration summary (1 page)

Task 7 — Compile and Review
Pull all outputs into one clean deliverable package.
Update this brief with any new findings
Mark all assumptions clearly
Write a short open questions list for the senior team
Prepare a brief verbal summary for manager review
Output: Final document package

| # | Deliverable                                              |
| - | -------------------------------------------------------- |
| 1 | Variable inventory spreadsheet (eVars + props)           |
| 2 | Usage audit (populated vs sparse)                        |
| 3 | Variable grouping by domain                              |
| 4 | Tag structure summary (properties, data elements, rules) |
| 5 | XDM mapping proposal table                               |
| 6 | AEP configuration notes                                  |
| 7 | Open questions and risks list                            |

Open Questions for Senior Team
Which Tag property/properties are in scope for this migration?
What is the exact data layer format and structure used on the client site?
Will Web SDK run in parallel with AppMeasurement during transition, or replace it directly?
Is the edge/server-side configuration Adobe-native or third-party?
Is the expected deliverable documentation only, or does it include a sandbox schema build?

1 Step-by-step:
Start with top 20-30 by "Most Recent" data volume (visible in Report Suite Manager). These are the important ones.
Use Page Views as metric (works for both props and eVars). Last 90 days.
Test one-by-one or small batches (5-10 at a time) — don't add all 100 rows. Too messy.
Prop vs eVar: Same process for both. Props are hit-level (good for page names), eVars persist (good for campaigns). Page Views works for both.
Classify:
Active = >10% of rows have data
Sparse = <10% populated
Legacy = no data last 90 days
Disabled = marked in Admin
Add a "Priority" column first: High/Med/Low based on "Most Recent" %, then test only High/Med. Low = "Suspected Legacy, not tested".

| Variable | Type | Name                 | Enabled | Allocation  | Usage (90d) | Populated % | Sample Values       | Priority | Status   |
| -------- | ---- | -------------------- | ------- | ----------- | ----------- | ----------- | ------------------- | -------- | -------- |
| eVarx    | eVar | Page Name            | Yes     | Most Recent | 45k rows    | 85%         | /article1, /home    | High     | Active   |
| prop x   | Prop | Internal Search Term | Yes     | Hit         | 2.1k rows   | 12%         | "EXAMPLE   ", "EXAMPLE" | Med      | Sparse   |
| eVarx    | eVar | Tag example                | No      | Most Recent | 0 rows      | 0%          | N/A                 | Low      | Disabled |
| propx    | Prop | Article Author       | Yes     | Hit         | 18k rows    | 92%         | "EXAMPLE"     | High     | Active   |


1) Allocation
Admin > Report Suites > [Your Report Suite Name] > Edit Settings
eVars: Go to "Conversion > Conversion Variables" → click variable number → Allocation column shows "Most Recent (Last)" or other.
Props: Go to "Traffic > Traffic Variables" → click variable number → Allocation shows "Hit" (always hit-level for props).
​
2) Priority Level
Report Suite Manager (same Admin screen):
Look at "Most Recent" column next to each variable (it's a data volume %).
High: >20% | Med: 5-20% | Low: <5% or 0%

3) Usage (90d) & Populated % & Status
Analysis Workspace > New Project:
Date range: Last 90 days
Drag variable → Rows, Page Views → Columns
Usage (90d): Total rows with data (e.g., 45k)

Populated %: (Rows with data / Total Page Views) * 100

Status:

Active: >10% populated

Sparse: 1-10%

Legacy: 0% last 90d

Disabled: From Admin
