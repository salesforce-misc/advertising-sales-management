# [User] Rules Framework Decision Table Configuration Data

### Abbreviations

* DT : Decision Table

### Note

* These decision tables will be used to override the UI configuration data present in AdAvailabilityViewConfig entity.
    * Any customizations in tabs, row actions etc., should be done accordingly in both places.
* They are required to be set up mandatorily for honoring certain rules around bundles, non-media products & amend media plans.

## Setup Steps

1. Open Salesforce Bench Press application for your target org.
2. Go to Metadata → Deploy.
3. Upload this zip file [ASM Rules DT](../zip-files/ASM_Rules_DT.zip)
4. Select Rollback On Error & Single Package parameters.
5. Click on deploy.
6. Once deployment succeeds, navigate to Lookup Tables in App Launcher.
7. Now, we can upload data to each DT via a CSV file using the Upload CSV button in the Table tab, located next to the Details tab.
    1. [This](../csv-files/256-ASMOOTBDecisionTableRules.xlsx) sheet contains the OOTB DT rules.
        1. There are 5 sheets & each sheet name is exactly same as the corresponding developer name of deployed DT’s in step 3.
        2. Download each of them as .csv file. I have attached the CSV’s here also in a zip.
            1. Use this zip file for .csv files [Rules Framework DT CSV Data](../zip-files/Rules_Framework_DT_CSV_Data.zip)
8. Post data upload is complete, click on refresh button for each DT.
9. Now we will be able to see impact of DT rules in UI features.

## Decision Table Details

### 1. Header Level Decision Table
1. This table will specify the exact DT to be selected for the side panel and grid data based on a specific quote-level header data.
2. DT’s can be reused for different set of header level input combinations. Also there is a particular limit on total number of possible DT’s in an org, so we need to be mindful of that as well.
3. Below is the data source for different input columns
    1. DealType → DealType field in AdOpportunity
    2. QuoteStatus → Status field in Quote
    3. QuoteActionType → OriginalActionType field in Quote
4. It will not support any custom input column via metadata service.
5. User can reuse same field config or grid data DT across multiple permutations.
6. Exact match will be done on this table from our metadata service. Providing generic column data for certain columns like Any, All is not supported in 256.
7. Input column values will be case sensitive & output column values will be case insensitive.
8. Below table contains some sample data.

| DealType     | QuoteStatus | QuoteActionType | FieldConfigurationDecisionTableAPIName      | GridDecisionTableAPIName           |
|--------------|-------------|------------------|----------------------------------------------|------------------------------------|
| Direct-sales | Draft       |                  | FieldConfig_Direct_DT                        | Grid_Direct_DT                     |
| Direct-sales | Draft       | Amend           | FieldConfig_Direct_Rejected_Amend_DT        | Grid_Direct_Rejected_Amend_DT      |

### 2. Grid Level Decision Table
1. This table will specify what are the applicable actions in grid rows & applicable side panel tabs.
2. Below is the data source for different input columns
    1. QuoteLineItemActionType → QuoteAction.Type field in QuoteLineItem
    2. ProductFamily → Family field in Product2
    3. DerivedLineItemTag → Not a specific field, its value is being derived from product. More details available in [DerivedLineItemTag Column Details](#derivedlineitemtag-column-details)
    4. ProductClassification → BasedOn field in Product2
3. Output columns contain comma separated applicable row action & sidepanel tab keys.
4. It will not support any custom input column via metadata service.
5. We will try to find match for two kind of input conditions in runtime.
    1. One will be where we will match the data exactly as it is in the input columns.
    2. Another will be where DerivedLineItemTag columns would have exact value but rest of the columns will be marked as ALL. This can be used to define conditions at a more generic level.
    3. If both kind of rows are defined, exact matched row will take precedence.
6. Input column values will be case sensitive & output column values will be case insensitive.
7. Below table contains some sample data.

| QuoteLineItemActionType | ProductFamily | DerivedLineItemTag          | ProductClassification | ApplicableActions | ApplicableTabs                                                 |
|-------------------------|---------------|-----------------------------|------------------------|------------------|----------------------------------------------------------------|
|                         | Digital       | Static Simple Product       | Digital Bann           | edit, clone, delete | lineitemdetails, attributes, creatives, targeting, delivery   |
| Amend                   | Digital       | Configurable Simple Product | Digital Audio          | edit, clone, delete | lineitemdetails, attributes, creatives, targeting, delivery   |


### 3. Field Configuration Level Decision Table
1. This table will specify what are the read only, required & hidden fields in various side panel tabs, modals etc.
2. Below is the data source for different input columns
    1. QuoteLineItemActionType → QuoteAction.Type field in QuoteLineItem
    2. ProductFamily → Family field in Product2
    3. DerivedProductType → Not a specific field, its value is being derived from product. More details available in [DerivedLineItemTag Column Details](#derivedlineitemtag-column-details)
    4. ProductClassification → BasedOn field in Product2
    5. ConfigurationKey → Not a specific field, its value is determined internally based on different UI tabs, screens
3. Output column contains comma separated values in form of {entityName}.{fieldDevName}.
    1. Spanning fields should always remain read only & not be editable or required. As PST doesn’t support spanning fields while performing CUD operations on final save button.
4. It will not support any custom input column via metadata service.
5. We will try to find match for two kind of input conditions in runtime.
    1. One will be where we will match the data exactly as it is in the input columns.
    2. Another will be where DerivedLineItemTag & ConfigurationKey would still have exact values but rest of the columns will be marked as ALL. This can be used to define conditions at a more generic level.
    3. If both kind of rows are defined, first one will take precedence.
6. Input column values will be case sensitive & output column values will be case insensitive.
7. Below table contains some sample data.

| QuoteLineItemActionType | ProductFamily             | DerivedLineItemTag              | ProductClassification | ConfigurationKey          | ReadOnlyFields                                                                                  | RequiredFields | HiddenFields                        |
|--------------------------|---------------------------|----------------------------------|------------------------|----------------------------|--------------------------------------------------------------------------------------------------|----------------|--------------------------------------|
| Digital                  | Configurable Simple Product | Digital Non Media              |                        | sidepanel-lineitemdetails | AdQuoteLine.PricingModel, QuoteLineItem.Quantity, AdQuoteLine.AdRequestedQuantity              |                | QuoteLineItem.Product2.Name         |
| ALL                      | ALL                       | Configurable Child Simple Product | ALL                  | sidepanel-lineitemdetails |                                                                                                  |                | QuoteLineItem.Product2.Name         |

## DerivedLineItemTag Column Details

* This column’s value is not coming from any entity field.
* OOTB it will contain certain fixed values which are derived internally on basis of product structure.
* Only 2 levels of bundles are supported in 256.
* It can contain following list of values (Support distinct values for 2 levels of bundles) -
    * Static Simple Product
    * Static Bundled Product
    * Static Child Simple Product
    * Static Child Bundled Product
    * Static Grandchild Simple Product
    * Configurable Simple Product
    * Configurable Bundled Product
    * Configurable Child Simple Product
    * Configurable Child Bundled Product
    * Configurable Grandchild Simple Product
* All grandchilds & further nested childs will fall in either Configurable Standalone Grandchild or Static Standalone Grandchild category.
* Static and Configurable products are PCM concept. If Configure During Sale field is set to Alllowed, then it is a configurable product else static product.
* User can override its data source from some custom or OOTB field available on AdQLI by specifying an additional configuration in AdAvailabilityViewConfig entity.
    * Create a new record in AdAvailabilityViewConfig entity.
        * Provide any relevant record name.
        * Provide ConfigurationType as General Configuration.
        * Provide ConfigurationKey as overrideDerivedProductTypeColumnDataSource.
        * Provide ConfigurationValue as field API name. Note that this field should be present on AdQuoteLine.

## ConfigurationKey Column Details

* This column’s value is not coming from any entity field.
* It’s purpose is to uniquely define a rule for a corresponding UI screen or tab.
    * All relevant screens, tabs have their fixed unique configuration key that needs to be used.
* As the key name suggests they are applicable for specific media grid sidepanel tabs.

#### System Fields

* These fields should not be removed from universe configuration data as well as should not be overriden as hidden in DT as they are required on UI for certain functionalities. Currently only these are marked as system fields in lineitemdetails tab configuration data. This validation is available in metadata service.
  * QuoteLineItem → Quantity
  * AdQuoteLine → Budget
    * Both of these fields are also being utilized in budget functionality in lineitem details tab. So user should not change the default readOnly or editable behaviour as being shipped OOTB. Budget should be readOnly & Quantity should be editable.
