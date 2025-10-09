# ASM - Seed Data Creation 

## Product Setup

Refer to the below doc to insert PCM and ASM related design data. (excludes targeting)

[Script for Sample Product Data Setup](scripts/SampleProductDataSetup.md)

## Targeting Setup

Refer to the below doc to insert the Targeting data.

[Targeting Setup Script](scripts/TargetingSetupScript.md)

## Rules Framework

* [Setup Steps: [User] Rules Framework Universal Configuration Data](documents/UserRulesFrameworkUniversalConfigurationData.md)
* [Setup Steps: [User] Rules Framework Decision Table Configuration Data](documents/UserRulesFrameworkDecisionTableConfigurationData.md)


## Pricing Procedure

* Sample Pricing Procedure which has all the steps related to Media : [MediaPricingProcedure_package.zip](zip-files/MediaPricingProcedure_package.zip)
This can be deployed from Workbench.

* Before deploying, unzip the package and open the procedure definition.

* Make sure the Context definition mentioned in the procedure is present in the target org. If not, update the definition with the right context definition name.

* Make sure the lookup tables are already present in the target org and update the definition to have the correct LookUpId properly set with the ID of the lookup tables in the target org.

* In the selected Pricing Recipe, add the Lookup table under Price Adjustment Matrix section.  

* The above pricing procedure has a reference to lookup table Priority Based Adjustments. If you do not have this in your org, use this zip to deploy the Lookup table and activate. [Priority_based_adj_DM.zip](zip-files/Priority_based_adj_DM.zip)


## Recording for reference:

[▶️ Click here to view the demo video](recordings/recording-1.mov)


## Set Up Advertising Inventory Management Sample Data

This section describes the steps and prerequisites for setting up the sample data for Advertising Inventory Management.

### Prerequisites
Before setting up the data, make sure that key administrative records and picklist values are correctly configured in your org.
### Required Records
To set the dimensions, make sure that these records are created in your org:
* Products
* Media Channels
* Ad Space Specifications
* Locations
* Unit of Measure

If the records aren't available in the org, you’ll see this error:
No <entity> records found for the provided IDs. Skipping AdSpace insertion.

### Required Junction Objects
The necessary junction object records must exist in the org and be correctly linked to the corresponding dimensional records. The setup script depends on these mappings to properly relate the new Ad Space records to the provided inputs. For example, the Ad Space Spec Product junction object links Product records with Ad Space Specification records.

### Required Dynamic Picklist Values
Make sure that the required picklist values are present in the object fields:

<table>
  <tr>
    <th align="left">Object</th>
    <th align="left">Field</th>
    <th align="left">Required Values</th>
  </tr>
  <tr>
    <td><b>Media Channel</b></td>
    <td>Media Type</td>
    <td>
      <ul>
        <li>Digital</li>
        <li>Print</li>
        <li>In Store</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td><b>Ad Space Creative Size Type</b></td>
    <td>Appearance Order</td>
    <td>
      <ul><li>Primary</li></ul>
    </td>
  </tr>
  <tr>
    <td><b>Location</b></td>
    <td>Location Type</td>
    <td>
      <ul><li>Store</li></ul>
    </td>
  </tr>
  <tr>
    <td><b>Unit Of Measure</b></td>
    <td>Type</td>
    <td>
      <ul>
        <li>Fixed</li>
        <li>SOV</li>
        <li>Impressions</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td rowspan="2"><b>Ad Space</b></td>
    <td>Ineligible Industry</td>
    <td>
      <ul>
        <li>Insurance</li>
        <li>Healthcare</li>
        <li>Education</li>
        <li>Agriculture</li>
        <li>Banking</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>Media Type</td>
    <td>
      <ul>
        <li>Digital</li>
        <li>Print</li>
        <li>In Store</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td><b>AdQuoteLine</b></td>
    <td>Pricing Model</td>
    <td>
      <ul><li>CPM</li></ul>
    </td>
  </tr>
  <tr>
    <td><b>Ad Availability View Config</b></td>
    <td>Configuration Type</td>
    <td>
      <ul>
        <li>Template</li>
        <li>Colour Scheme</li>
        <li>Screen</li>
        <li>Tab</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td><b>Ad Space Capacity</b></td>
    <td>Capacity Duration Type</td>
    <td>
      <ul>
        <li>Daily</li>
        <li>Weekly</li>
        <li>Monthly</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td><b>Ad Space Capacity Allocation</b></td>
    <td>Allocation Status</td>
    <td>
      <ul>
        <li>Pitched</li>
        <li>On Hold</li>
        <li>Released</li>
        <li>Planned</li>
        <li>Planned and On Hold</li>
        <li>Booked</li>
      </ul>
    </td>
  </tr>
</table>

### Create Apex Class Definitions
The procedure uses two Apex class definitions to set up  the data. 

Create two Apex class definitions to configure sample data in your org. Copy the contents of the [InventoryManagementSetup](scripts/InventoryManagementSetup.md) and the [AdSpaceCapacityBatch](scripts/AdSpaceCapacityBatch.md) files to those  Apex class definitions.

*   [Script for AdSpace Capacity Batch Setup](scripts/AdSpaceCapacityBatch.md)
*   [Script for Sample Inventory Management Setup](scripts/InventoryManagementSetup.md)

Now follow these steps: 
    [Steps for executing Inventory Management Scripts](scripts/InventoryManagementExecuteSteps.md)

