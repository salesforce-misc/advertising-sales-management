# [User] Rules Framework Universal Configuration Data



## AdAvailabilityViewConfig Records Structure

* Universal configuration data for rules framework will be present in AdAvailabilityViewConfig entity.
* ConfigurationKey & ConfigurationType field value should not be changed in default records.
* ConfigurationValue field structure & parameter keys should not be changed for corresponding configurations
    * While adding custom configuration data, ensure that it aligns with the existing JSON structure.
* IsActive field denotes whether that particular configuration is active or not.
* The MediaType field specifies the type of line items to which this configuration applies.
    * In some generic configurations that apply regardless of media type, this field value will not be relevant.
* PivotOn field is not relevant for configuration rules.


## Setup Steps

* Ensure FLS is enabled for Ad Availability View Configuration entity & its fields.
* Ensure that following picklist values are available in ConfigurationType field within Ad Availability View Configuration entity. If they are not present, add them manually in org.
    * General Configuration
    * Screen
    * Tab
    * Modal
    * Summary
* Ensure following picklist values are available in MediaType field within Ad Availability View Configuration entity. If they are not present, add them manually in org.
    * Digital
* Navigate to Media Cloud Settings page in org setup.
* Turn the Media Plan and Proposal Configuration toggle on.
* It will handle the creation of all required configuration records.


## Default Records

### 1) Configuration For Grid Row Actions

* This configuration controls the media grid row actions to be displayed on UI.

| Name               | PivotOn | MediaType | ConfigurationType | IsActive | ConfigurationKey | ConfigurationValue  |
|--------------------|---------|-----------|--------------------|----------|-------------------|----------------------|
| Media Grid Row Actions | Null    | Digital   | Screen             | TRUE     | grid-rowactions   | Row Actions JSON     |

* Row Actions JSON
  <pre>{ "actions": [ { "label": "Edit", "key": "edit" }, { "label": "Clone", "key": "clone" }, { "label": "Delete", "key": "delete" } ] }</pre>
    * “label” denotes the value that will be shown on UI.
    * “key” denotes the unique key for that action.
        * edit, clone & delete keys are reserved for their respective OOTB functionality.
    * Custom row actions can be added similarly & when user clicks on that row actions, we emit a event in the message channel with the relevant row data. The user can subscribe to it & perform custom actions.


### 2) Configuration For Side Panel Tabs Metadata

* This configuration controls the available tabs in media grid’s side panel.

| Name                   | PivotOn | MediaType | ConfigurationType | IsActive | ConfigurationKey | ConfigurationValue |
|------------------------|---------|-----------|--------------------|----------|-------------------|--------------------|
| Side Panel Tabs Metadata | Null    | Digital   | Screen             | TRUE     | sidepanel         | Tabs Metadata JSON |

* Tabs Metadata JSON

<pre>
  {
    "tabsMetadata": [
      {
          "label": "Line Item",
          "isVisible": true,
          "flowApiName": null,
          "key": "lineitemdetails",
          "isSelected": true
      },
      {
          "label": "Targeting",
          "isVisible": true,
          "flowApiName": null,
          "key": "targeting",
          "isSelected": false
      },
      {
          "label": "Delivery",
          "isVisible": true,
          "flowApiName": null,
          "key": "delivery",
          "isSelected": false
      },
      {
          "label": "Attribute",
          "isVisible": true,
          "key": "attributes",
          "flowApiName": null,
          "isSelected": false
      },
      {
          "label": "Ad spaces and Creatives",
          "isVisible": true,
          "flowApiName": null,
          "key": "creatives",
          "isSelected": false
      },
      {
          "label": "Bundle Items",
          "isVisible": true,
          "key": "bundles",
          "flowApiName": null,
          "isSelected": false
      }
    ]
  }
</pre>

* “label” denotes the tab label to be shown on UI.
* “isVisible” denotes whether that tab is visible or not.
* “isSelected” denotes which tab is selected by default when the side panel opens. Only one tab should be marked isSelected as true & rest all should be false.
* “key” denotes the unique key for that tab.
    * lineitemdetails, targeting, delivery, attributes, creatives & bundles keys are reserved for OOTB tabs.
* To add a custom tab, “flowApiName” is mandatory and must contain the developer name of the flow, as a corresponding flow is required for the tab’s functionality.
    * Key for custom tabs should be prefixed with tab keyword.


### 3) Configuration For Side Panel LineItemDetails Tab Data

* This configuration controls the available sections & fields in LineItemDetails tab.
* Only AdQuoteLine & QuoteLineItem entity fields are supported.
* Spanning fields like for example Product2.Family can only be used in read only mode. Editable spanning fields are not supported.
* All editable fields should be mapped in the context definition linked to default pricing procedure.
    * Map them under QuoteEntitiesMapping.
    * Note the context attribute name to which the field has been mapped, as it will be needed in the configuration JSON data. Mapped attribute name is required for editable fields.
* System fields :
    * System fields should not be removed from universe configuration data as well as should not be overridden as hidden in Decision Table as they are required on UI for certain functionalities. Currently only these are marked as system fields in lineitemdetails tab configuration data. This validation is available in metadata service. Below are the two system fields -
      * QuoteLineItem → Quantity
      * AdQuoteLine → Budget
        * Both of these fields are also being utilized in budget functionality in lineItem details tab. So user should not change the default readOnly or editable behaviour as being shipped OOTB. Budget should be readOnly & Quantity should be editable.

| Name                 | PivotOn | MediaType | ConfigurationType | IsActive | ConfigurationKey        | ConfigurationValue        |
|----------------------|---------|-----------|--------------------|----------|--------------------------|----------------------------|
| LineItemDetails Tab Data | Null    | Digital   | Tab                | TRUE     | sidepanel-lineitemdetails | LineItemDetails Tab JSON   |

* LineItemDetails Tab JSON
<pre>
{
  "tabsData": {
    "content": [
      {
        "section": {
          "label": "Details",
          "isVisible": true,
          "key": "Product",
          "isCollapsed": false,
          "fields": [
            {
              "entityName": "QuoteLineItem",
              "fieldApiName": "LineNumber",
              "isReadOnly": true,
              "datatype": "String",
              "label": "Line Number"
            },
            {
              "entityName": "QuoteLineItem",
              "fieldApiName": "Product2.Family",
              "isReadOnly": true,
              "datatype": "String",
              "label": "Product Category"
            },
            {
              "entityName": "QuoteLineItem",
              "fieldApiName": "Product2.Name",
              "isReadOnly": true,
              "datatype": "String",
              "label": "Product Name"
            },
            {
              "entityName": "QuoteLineItem",
              "fieldApiName": "Description",
              "isReadOnly": false,
              "datatype": "String",
              "label": "Description",
              "mappedContextAttributeName": "SalesTrxnItemDescription"
            },
            {
              "entityName": "AdQuoteLine",
              "fieldApiName": "AdRequestedStartDate",
              "isReadOnly": false,
              "isRequired": true,
              "datatype": "DateTime",
              "label": "Start Date",
              "mappedContextAttributeName": "AdRequestedStartDate"
            },
            {
              "entityName": "AdQuoteLine",
              "fieldApiName": "AdRequestedEndDate",
              "isReadOnly": false,
              "isRequired": false,
              "datatype": "DateTime",
              "label": "End Date",
              "mappedContextAttributeName": "AdRequestedEndDate"
            },
            {
              "entityName": "AdQuoteLine",
              "fieldApiName": "AvailableAdServerUnits",
              "isReadOnly": true,
              "datatype": "Double",
              "label": "Available Units"
            },
            {
              "entityName": "AdQuoteLine",
              "fieldApiName": "TotalAdServerUnits",
              "isReadOnly": true,
              "datatype": "Double",
              "label": "Total Units"
            }
          ]
        }
      },
      {
        "section": {
          "label": "Price",
          "isVisible": true,
          "key": "Pricing",
          "isCollapsed": true,
          "fields": [
            {
              "entityName": "QuoteLineItem",
              "fieldApiName": "UnitPrice",
              "isReadOnly": true,
              "isRequired": false,
              "datatype": "Currency",
              "label": "Base Rate"
            },
            {
              "entityName": "AdQuoteLine",
              "fieldApiName": "RevisedPrice",
              "isReadOnly": false,
              "isRequired": false,
              "datatype": "Currency",
              "label": "Revised Price",
              "mappedContextAttributeName": "RevisedPrice"
            },
            {
              "entityName": "AdQuoteLine",
              "fieldApiName": "Budget",
              "isReadOnly": true,
              "isRequired": false,
              "datatype": "Currency",
              "label": "Calculated Budget",
              "mappedContextAttributeName": "Budget"
            },
            {
              "entityName": "QuoteLineItem",
              "fieldApiName": "Quantity",
              "isReadOnly": false,
              "isRequired": false,
              "datatype": "Double",
              "label": "Units",
              "mappedContextAttributeName": "Quantity"
            },
            {
              "entityName": "AdQuoteLine",
              "fieldApiName": "AdServerFees",
              "isReadOnly": false,
              "isRequired": false,
              "datatype": "Currency",
              "label": "Ad Server Fees",
              "mappedContextAttributeName": "AdServerFees"
            },
            {
              "entityName": "QuoteLineItem",
              "fieldApiName": "DiscountAmount",
              "isReadOnly": false,
              "isRequired": false,
              "datatype": "Currency",
              "label": "Discount Amount",
              "mappedContextAttributeName": "DiscountAmount"
            },
            {
              "entityName": "QuoteLineItem",
              "fieldApiName": "NetUnitPrice",
              "isReadOnly": true,
              "isRequired": true,
              "datatype": "Currency",
              "label": "Offer Price"
            },
            {
              "entityName": "AdQuoteLine",
              "fieldApiName": "SponsorshipType",
              "isReadOnly": false,
              "isRequired": false,
              "datatype": "Picklist",
              "label": "Booking Type",
              "mappedContextAttributeName": "SponsorshipType"
            }
          ]
        }
      }
    ]
  }
}
</pre>

* “section” refers to the accordion on tab & it contain certain section info & fields that are present in it. Below are the details of keys present within it.
    * “label” contains the section’s label.
    * “isVisible” denotes whether that section is visible or not.
    * “isCollapsed” denotes whether that section will be by default collapsed or not.
    * “key” contains the unique key for sections.
    * “fields” contains the list of fields that are configured in that section. Below are the details of keys present within it.
        * “entityName” contains the developer name of entity.
        * “fieldApiName” contains the developer name of field.
        * “isReadOnly” denotes whether this field is read only or editable.
        * “isRequired” denotes whether this field is required to be populated.
        * “datatype” denotes field’s data type.
        * “label” denotes the displayed field label.
        * “mappedContextAttributeName” is the mapped context attribute name in QuoteEntitiesMapping within context linked to default pricing procedure. It is required for editable fields.
* Custom sections & fields can be added in a similar manner.

### 4) Configuration For Create/Edit Media Plan Modal Data

* This configuration controls the fields and sections available in the Create Media Plan Modal and Edit Media Plan Moal.
* The modal consists of multiple sections, each containing a set of fields.
* Fields are fetched from Opportunity, AdOpportunity, AdQuote, and Quote entities.  Only fields from AdQuote and Quote should be editable, while those from Opportunity and AdOpportunity must remain read-only.
* Some fields are read-only while others are editable.
* The modal includes validations for required fields.

| Name            | PivotOn | MediaType | ConfigurationType | IsActive | ConfigurationKey | ConfigurationValue       |
|------------------|---------|-----------|--------------------|----------|--------------------|---------------------------|
| Modal Metadata   | Null    | Null      | Modal              | TRUE     | mediaplanmodal     | media plan modal JSON     |

* media plan modal JSON
<pre>
{
  "modalData": {
    "content": [
      {
        "section": {
          "label": "Campaign Details",
          "isVisible": true,
          "key": "Campaign-Details",
          "sequence": 1,
          "fields": [
            {
              "objectApiName": "AdOpportunity",
              "fieldApiName": "CampaignStartDate",
              "isReadOnly": true,
              "isRequired": false,
              "sequence": 3,
              "label": "Campaign Start Date"
            },
            {
              "objectApiName": "AdOpportunity",
              "fieldApiName": "CampaignEndDate",
              "isReadOnly": true,
              "isRequired": false,
              "sequence": 4,
              "label": "Campaign End Date"
            },
            {
              "objectApiName": "Opportunity",
              "fieldApiName": "Name",
              "isReadOnly": true,
              "isRequired": false,
              "sequence": 1,
              "label": "Name"
            },
            {
              "objectApiName": "Opportunity",
              "fieldApiName": "Amount",
              "isReadOnly": true,
              "isRequired": false,
              "sequence": 2,
              "label": "Campaign Budget"
            }
          ]
        }
      },
      {
        "section": {
          "label": "Media Plan Details",
          "isVisible": true,
          "key": "Media-Plan-Details",
          "sequence": 2,
          "fields": [
            {
              "objectApiName": "AdOpportunity",
              "fieldApiName": "AgencyId",
              "isReadOnly": true,
              "isRequired": false,
              "sequence": 2,
              "label": "Agency"
            },
            {
              "objectApiName": "AdOpportunity",
              "fieldApiName": "DealType",
              "isReadOnly": true,
              "isRequired": false,
              "sequence": 3,
              "label": "Campaign Type"
            },
            {
              "objectApiName": "AdQuote",
              "fieldApiName": "Budget",
              "isReadOnly": false,
              "isRequired": true,
              "sequence": 6,
              "label": "Budget"
            },
            {
              "objectApiName": "AdQuote",
              "fieldApiName": "StartDate",
              "isReadOnly": false,
              "isRequired": true,
              "sequence": 7,
              "label": "Start Date"
            },
            {
              "objectApiName": "AdQuote",
              "fieldApiName": "EndDate",
              "isReadOnly": false,
              "isRequired": false,
              "sequence": 9,
              "label": "End Date"
            },
            {
              "objectApiName": "AdQuote",
              "fieldApiName": "MediaType",
              "isReadOnly": false,
              "isRequired": true,
              "sequence": 9,
              "label": "Media Type"
            },
            {
              "objectApiName": "Quote",
              "fieldApiName": "Name",
              "isReadOnly": false,
              "isRequired": true,
              "sequence": 5,
              "label": "Plan Name"
            },
            {
              "objectApiName": "Quote",
              "fieldApiName": "Description",
              "isReadOnly": false,
              "isRequired": false,
              "sequence": 10,
              "label": "Description"
            }
          ]
        }
      }
    ]
  }
}
</pre>
* “section” refers to the accordion on modal & it contain certain section info & fields that are present in it. Below are the details of keys present within it.
    * “label” contains the section’s label.
    * “isVisible” denotes whether that section is visible or not.
    * “isCollapsed” denotes whether that section will be by default collapsed or not.
    * “key” contains the unique key for sections.
    * “fields” contains the list of fields that are configured in that section. Below are the details of keys present within it.
        * “objectApiName” contains the developer name of entity.
        * “fieldApiName” contains the developer name of field.
        * “isReadOnly” denotes whether this field is read only or editable.
        * “isRequired” denotes whether this field is required to be populated.
        * “datatype” denotes field’s data type.
        * “label” denotes the displayed field label.


### 5) Configuration For Media Plan Summary Screen Data

* This configuration defines the fields shown in the Media Plan Summary screen.
* Summary data includes general information of media plan and line items.
* Supports Opportunity, AdOpportunity, Quote , AdQuote and Account entities.

| Name             | PivotOn | MediaType | ConfigurationType | IsActive | ConfigurationKey | ConfigurationValue        |
|------------------|---------|-----------|--------------------|----------|-------------------|----------------------------|
| Summary Metadata | Null    | Null      | Summary            | TRUE     | summary           | Summary columns JSON       |


* Summary columns JSON
<pre>
{
  "summaryData": [
    {
      "summaryColumns": [
        {
          "entityName": "AdOpportunity",
          "fieldApiName": "DealType"
        },
        {
          "entityName": "Quote",
          "fieldApiName": "Status"
        },
        {
          "entityName": "AdQuote",
          "fieldApiName": "StartDate"
        },
        {
          "entityName": "AdQuote",
          "fieldApiName": "EndDate"
        },
        {
          "entityName": "Opportunity",
          "fieldApiName": "StageName"
        }
      ]
    },
    {
      "kpiColumns": [
        {
          "entityName": "AdQuote",
          "fieldApiName": "Budget"
        }
      ]
    }
  ]
}
</pre>
* Summary columns represent the fields displayed in the bottom section of the summary component, providing general information about the media plan. KPI columns include budget-related fields. The summary component will only display the first 3 fields of kpi columns, as a limit of three fields is enforced.
    * “entityName” denotes the the developer name of entityI.
    * “fieldApiName” contains the developer name of field.


### 6) Configuration For Rules Header DT

* This configuration denotes which Header DT will be plugged in within Rules Framework.
* ConfigurationValue contain the DT API name.

| Name                | PivotOn | MediaType | ConfigurationType     | IsActive | ConfigurationKey             | ConfigurationValue           |
|---------------------|---------|-----------|------------------------|----------|-------------------------------|-------------------------------|
| Rules Header DT Config | Null    | Null      | General Configuration | TRUE     | metadataRulesHeaderDTApiName | ASM

## Proposal Config

### 1. Configuration For Proposal Grid Row Action


* This configuration controls the media grid row actions to be displayed on UI.

| Name                      | PivotOn | MediaType | ConfigurationType | IsActive | ConfigurationKey          | ConfigurationValue    |
|---------------------------|---------|-----------|--------------------|----------|----------------------------|------------------------|
| Proposal Grid Row Actions | Null

* Row Actions JSON
<pre>
{
  "actions": [
    {
      "label": "Delete",
      "name": "delete",
      "showAction": true
    },
    {
      "label": "Show Products in Bundle",
      "name": "show_bundle_item",
      "showAction": true
    },
    {
      "label": "Custom flow",
      "name": "custom_flow",
      "flowApiName": "custom_Flow_Api_Names",
      "showAction": false
    }
  ]
}
</pre>

* “label” denotes the value that will be shown on UI.
* “name” denotes the unique key for that action.
* “showAction” denotes whether the action will be visible on the UI or not.
* Custom row actions can be added similarly & when user clicks on that row actions, open up a Screen flow in a Modal.

### 2. Configuration For Proposal Grid Columns Level Config

* This configuration controls the columns on the Proposal Grid.

| Name                    | PivotOn | MediaType | ConfigurationType | IsActive | ConfigurationKey         | ConfigurationValue   |
|-------------------------|---------|-----------|--------------------|----------|---------------------------|----------------------|
| Proposal Grid Row Actions | Null    | Digital   | Screen             | TRUE     | proposal-grid-rowactions  | Grid Column JSON     |

* Grid Column JSON
<pre> 
  {
  "proposal": {
    "content": {
      "name": "Product Grid",
      "key": "Product-Grid",
      "allocations": {
        "productAllocation": {
          "columns": [
            {
              "entityName": "Product2",
              "fieldApiName": "Name",
              "datatype": "STRING",
              "label": "Product Name",
              "editable": false,
              "showOnGrid": true,
              "width": 200
            },
            {
              "entityName": "Product2",
              "fieldApiName": "Description",
              "datatype": "STRING",
              "label": "Product Description",
              "editable": false,
              "showOnGrid": true,
              "width": 270
            },
            {
              "entityName": "QuoteLineItem",
              "fieldApiName": "UnitPrice",
              "datatype": "CURRENCY",
              "label": "Unit Price",
              "editable": false,
              "showOnGrid": true,
              "width": 180
            },
            {
              "entityName": "AdQuoteLine",
              "fieldApiName": "PricingModel",
              "datatype": "PICKLIST",
              "label": "Pricing Model",
              "editable": true,
              "showOnGrid": true,
              "width": 180
            },
            {
              "entityName": "QuoteLineItem",
              "fieldApiName": "Quantity",
              "datatype": "NUMBER",
              "label": "Units",
              "editable": true,
              "showOnGrid": true,
              "width": 180
            },
            {
              "entityName": "QuoteLineItem",
              "fieldApiName": "TotalPrice",
              "datatype": "CURRENCY",
              "label": "Total Price",
              "editable": false,
              "showOnGrid": true,
              "width": 180
            }
          ]
        },
        "budgetAllocation": {
          "columns": [
            {
              "entityName": "Product2",
              "fieldApiName": "Name",
              "datatype": "STRING",
              "label": "Product Name",
              "editable": false,
              "showOnGrid": true,
              "width": 230
            },
            {
              "entityName": "Product2",
              "fieldApiName": "Description",
              "datatype": "STRING",
              "label": "Product Description",
              "editable": false,
              "showOnGrid": true,
              "width": 200
            },
            {
              "entityName": "AdQuoteLine",
              "fieldApiName": "Budget",
              "datatype": "CURRENCY",
              "label": "Budget",
              "editable": true,
              "showOnGrid": true,
              "width": 200
            },
            {
              "entityName": "",
              "fieldApiName": "Split",
              "datatype": "PERCENT",
              "label": "Split (%)",
              "editable": false,
              "showOnGrid": true,
              "width": 240
            }
          ]
        }
      }
    }
  }
}
</pre>
* “allocations” denotes two different allocations (Budget Allocation and Product Allocation).
* “columns” denotes the visible columns under each allocations. Each columns will have following properties:
    * “entityName” contains the developer name of entity.
    * “fieldApiName” contains the developer name of field.
    * “datatype” denotes field’s data type.
    * “label” denotes the displayed Column label.
    * “editable” denotes whether this field is read only or editable.
    * “showOnGrid” denotes whether the column is visible on the grid or not.
    * “width” denotes the fixed width of the column.
* Edit-ability is supported on all the AdQuoteLine field and QuoteLineItem Quantity field.
* Date Type field is not supported.

