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
