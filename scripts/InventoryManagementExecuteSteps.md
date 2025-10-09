# Step to execute

### Add the Ad Availability View Config Records
The setup needs the Ad Availability View Config records to be available in your org. These records include the Inventory Calendar Template, Quick Actions, Demand Details, and Availability Status color schemes.

To add the records, run this Apex code:
```
InventoryManagementSetup.insertAdAvailabilityViewConfigs();
```

### Run the Apex Class Definitions
When you run the Apex class definitions, the system adds the Ad Space and Ad Space Capacity records for the Digital Media type, based on the dimensions you set.
To run the Apex class definitions, run this code:


```
// InventoryManagementSetup.clearData(true); // Keep commented if you don't want to clear all data

/*
The below example inserts AdSpace and Capacity records for a Digital Product 'Digital 4K Video'
The product is linked with Media Channel: 'CNN Home Page', AdSpaceSpecifications: 'CNN Skyscraper' and 'CNN Leaderboard',
UnitOfMeasure: 'Impressions' and Targeting Values: 'Family Income < 50k', 'Family Income between 50k - 100k'
*/
// Replace with actual Product2 IDs from your org
List<Id> productIds = new List<Id>{
'01tWs000009ERCrIAO'
};
Map<String,List<String>> dimensions = new Map<String,List<String>>();

// Define the dimensions based on your media type
// Eg. Dimensions for Digital Mediatype: Creates AdSpaces on the basis of MediaChannel, AdSpaceSpecification, UnitOfMeasure and Targeting data
dimensions.put('MediaChannel', new List<String>{'CNN Home Page'});
dimensions.put('AdSpaceSpecification', new List<String>{'CNN Skyscraper', 'CNN Leaderboard'});
dimensions.put('TargetingSegmentValues', new List<String>{'Family Income < 50k', 'Family Income between 50k - 100k'});
dimensions.put('UnitOfMeasure', new List<String>{'Impressions'});

// Insert AdSpace records
InventoryManagementSetup.insertAdSpaces(productIds, dimensions);
// Insert AdSpaceCapacity records
// You can provide custom counts or null for defaults (90 daily, 52 weekly, 12 monthly) and the media type
InventoryManagementSetup.insertAdSpaceCapacities(25, 20, 10, 'Digital', productIds);
// >> The above snippet creates 25 daily, 20 weekly and 10 monthly duration type capacity records for the digital products with ProductIds mentioned in :productIds



// Similarly for other media types:
/* Eg: Dimensions for Print media type. Creates AdSpaces on the basis of MediaChannel, AdSpaceSpecification andMedia Print Issues
* Assumption: Admin data (MediaChannel, AdSpaceSpecifications and MediPrintIsuue) and
* Junction data (AdSpecMediaPrintIssue, AdSpaceSpecProduct) records exist.
*/
/*
List<Id> productIds = new List<Id>{'01txx0000006iSGAAY'};
Map<String,List<String>> dimensions = new Map<String,List<String>>();

dimensions.put('MediaChannel', new List<String>{'AutoCar'});
dimensions.put('AdSpaceSpecification', new List<String>{'AutoCar FrontoPage', 'AutoCar BackCover'});
dimensions.put('UnitOfMeasure', new List<String>{'Fixed Slots'});

InventoryManagementSetup.insertAdSpaces(productIds, dimensions);
InventoryManagementSetup.insertAdSpaceCapacities(5, 10, 15, 'Print', productIds);
// >> The above snippet creates 5 daily, 10 weekly and 15 monthly duration type capacity records for the print products with ProductIds mentioned in :productIds
*/

// Eg: Dimensions for In Store media type. Creates AdSpaces on the basis of Location and AdSpaceSpecification combinations.
/*
List<Id> productIds = new List<Id>{'01txx0000006iSWxyY'};
Map<String,List<String>> dimensions = new Map<String,List<String>>();
dimensions.put('Location', new List<String>{'Nairobi', 'Munich'});
dimensions.put('AdSpaceSpecification', new List<String>{'60 Second Audio', '90 Second Audio'});

InventoryManagementSetup.insertAdSpaces(productIds, dimensions);
InventoryManagementSetup.insertAdSpaceCapacities(25, 25, 10, 'In Store', productIds);
// >> The above snippet creates 25 daily, 20 weekly and 10 monthly duration type capacity records for the 'In Store' products with ProductIds mentioned in :productIds
*/
```

### Configure Filters
Use filters to refine and narrow down your inventory data. To configure filters, see [Configure Filters on Advertising Inventory Calendar](https://help.salesforce.com/s/articleView?id=ind.media_ad_inventory_management_configure_filters.htm&type=5).