# Script for Inventory Management Setup

```
/**
 * @description InventoryManagementSetup - A utility class for managing AdSpace inventory setup
 * across different media types (Digital, Print, In Store). This class provides methods to
 * create AdSpace records with their associated capacities and configurations.
 *
 * @version 1.0
 * @since 2025
 */
public class InventoryManagementSetup {
    
    // ===========================================
    // CONSTANTS AND STATIC VARIABLES
    // ===========================================
    
    /** Map to store AdSpace name to ID mappings for quick lookup */
    public static Map<String, Id> adSpaceNameToIdMap = new Map<String, Id>();
    
    public static final String MEDIATYPE_DIGITAL = 'Digital';
    public static final String MEDIATYPE_PRINT = 'Print';
    public static final String MEDIATYPE_INSTORE = 'In Store';
    
    // ===========================================
    // PUBLIC METHODS
    // ===========================================
    
    /**
     * @description Clears all AdSpace related data from the system
     * @param sure Boolean flag to confirm data deletion (safety mechanism)
     * 
     * This method performs a complete cleanup of:
     * - AdSpaceCapacityAllocation records (limited to 9000)
     * - AdSpaceCapacity records
     * - AdSpace records
     * 
     * @example
     * InventoryManagementSetup.clearData(true); // Confirms deletion
     * InventoryManagementSetup.clearData(false); // Shows warning message
     */
    public static void clearData(Boolean sure) {
        if (sure) {
            System.debug('=== ALERT ===');
            System.debug('=== ALL Inventory AdSpace data is being deleted ===');
            try {
                // Delete in reverse dependency order to avoid foreign key constraints
                delete [SELECT Id FROM AdSpaceCapacityAllocation LIMIT 8000];
                delete [SELECT Id FROM AdSpaceCapacity];
                delete [SELECT Id FROM AdSpace];
                System.debug('Cleanup complete.');
            } catch (DMLException e) {
                System.debug('Error clearing data: ' + e.getMessage());
            }
        } else {
            System.debug('Looks like you are not sure for clearing data. If sure, execute: InventoryManagementSetup.clearData(true)');
        }
    }
    
    /**
     * @description Inserts AdAvailabilityViewConfig records into the system.
     * Records are inserted only if a record with the same ConfigurationKey does not already exist.
     */
    public static void insertAdAvailabilityViewConfigs() {
        System.debug('Starting AdAvailabilityViewConfig insertion process...');

        List<AdAvailabilityViewConfig> configsToCreate = new List<AdAvailabilityViewConfig>();

        // RECORD 1: Template Config
        String jsonValue1 = '{"defaultCalendarType":"digital","digital":{"productGroupTypeValue":"Digital","productGroupType":"productFamily","durationType":"Weekly","label":"Digital","unitType":"Impressions","adSpaceDimensions":[{"label":"Media Channel","objectDeveloperName":"AdSpace","fieldDeveloperName":"MediaChannel.Name","isVisible":true},{"label":"Product","objectDeveloperName":"AdSpace","fieldDeveloperName":"Product.Name","isVisible":true},{"label":"Ad Space","objectDeveloperName":"AdSpace","fieldDeveloperName":"AdSpaceSpecification.Name","isVisible":true},{"label":"Targeting Segment Value","objectDeveloperName":"AdSpace","fieldDeveloperName":"AdTargetSegmentValue.Name","isVisible":true}],"adSpaceCapacityFields":[{"label":"Booked","objectDeveloperName":"AdSpaceCapacity","fieldDeveloperName":"BookedCount","isVisible":true},{"label":"Reserved","objectDeveloperName":"AdSpaceCapacity","fieldDeveloperName":"ReservedCount","isVisible":true},{"label":"Pitched","objectDeveloperName":"AdSpaceCapacity","fieldDeveloperName":"PitchedCount","isVisible":true}]},"Print":{"productGroupTypeValue":"Print","productGroupType":"productFamily","durationType":"Weekly","unitType":"Fixed Slots","adSpaceDimensions":[{"label":"Platform","objectDeveloperName":"AdSpace","fieldDeveloperName":"MediaChannel.Name","isVisible":true},{"label":"Product","objectDeveloperName":"AdSpace","fieldDeveloperName":"Product.Name","isVisible":true},{"label":"Ad Space","objectDeveloperName":"AdSpace","fieldDeveloperName":"AdSpaceSpecification.Name","isVisible":true}],"adSpaceCapacityFields":[{"label":"Booked","objectDeveloperName":"AdSpaceCapacity","fieldDeveloperName":"BookedCount","isVisible":true},{"label":"Reserved","objectDeveloperName":"AdSpaceCapacity","fieldDeveloperName":"ReservedCount","isVisible":true},{"label":"Pitched","objectDeveloperName":"AdSpaceCapacity","fieldDeveloperName":"PitchedCount","isVisible":true}]},"In Store":{"productGroupTypeValue":"In Store","productGroupType":"productFamily","durationType":"Weekly","unitType":"Fixed Slots","adSpaceDimensions":[{"label":"Store Name","objectDeveloperName":"AdSpace","fieldDeveloperName":"Location.Name","isVisible":true},{"label":"Product","objectDeveloperName":"AdSpace","fieldDeveloperName":"Product.Name","isVisible":true},{"label":"Ad Space","objectDeveloperName":"AdSpace","fieldDeveloperName":"AdSpaceSpecification.Name","isVisible":true}],"adSpaceCapacityFields":[{"label":"Booked","objectDeveloperName":"AdSpaceCapacity","fieldDeveloperName":"BookedCount","isVisible":true},{"label":"Reserved","objectDeveloperName":"AdSpaceCapacity","fieldDeveloperName":"ReservedCount","isVisible":true},{"label":"Pitched","objectDeveloperName":"AdSpaceCapacity","fieldDeveloperName":"PitchedCount","isVisible":true}]}}';
        configsToCreate.add(new AdAvailabilityViewConfig(
            Name = 'Advertising Inventory Calendar Template',
            PivotOn = null,
            MediaType = null,
            ConfigurationType = 'Template',
            IsActive = true,
            ConfigurationKey = 'advertising-inventory-calendar-template',
            ConfigurationValue = jsonValue1
        ));
        
        // RECORD 2: Cell Quick Actions Config
        String jsonValue2 = '{"actions":[{"label":"Pitch Inventory Slot","key":"pitchSlot","flowApiName":"media_inventory__PitchInventorySlot","isVisible":true, "isBulkAction":true},{"label":"Reserve Inventory Slot","key":"reserveSlot","flowApiName":"media_inventory__ReserveInventorySlots","isVisible":true, "isBulkAction":true},{"label":"View Details","key":"viewSlotDetails","isVisible":true}]}';
        configsToCreate.add(new AdAvailabilityViewConfig(
            Name = 'Advertising Inventory Calendar Cell Quick Actions',
            PivotOn = null,
            MediaType = 'Digital',
            ConfigurationType = 'Screen',
            IsActive = true,
            ConfigurationKey = 'advertising-inventory-calendar-cell-actions',
            ConfigurationValue = jsonValue2
        ));
        
        // RECORD 3: Demand menu options
        String jsonValue3 = '{"default":{"fields":[{"entityName":"AdSpaceCapacityAllocation","fieldApiName":"Quantity","label":"Quantity","isVisible":true},{"entityName":"AdSpaceCapacityAllocation","fieldApiName":"OfferPrice","label":"Offer Price","isVisible":true},{"entityName":"AdSpaceCapacityAllocation","fieldApiName":"StartDate","label":"Start Date","isVisible":true},{"entityName":"AdSpaceCapacityAllocation","fieldApiName":"EndDate","label":"End Date","isVisible":true},{"entityName":"AdSpaceCapacityAllocation","fieldApiName":"AdQuote.Name","isVisible":false},{"entityName":"AdSpaceCapacityAllocation","fieldApiName":"Opportunity.Name","label":"Opportunity","isVisible":true},{"entityName":"AdSpaceCapacityAllocation","fieldApiName":"AdSpaceCapacity.AdSpace.Product.Name","label":"Ad Product","isVisible":true},{"entityName":"AdSpaceCapacityAllocation","fieldApiName":"AdSpaceCapacity.AdSpace.MediaChannel.Name","label":"Media Channel","isVisible":true}],"actions":[]},"pitched":{"actions":[{"label":"Reserve Inventory Slot","key":"reserveSlot","flowApiName":"media_inventory__ReserveInventorySlots","isVisible":true},{"label":"Add to Media Plan","key":"addToMediaPlan","flowApiName":"media_inventory__AddSlotsToMediaPlan","isVisible":true}]},"on hold":{"actions":[{"label":"Release Allocation","key":"releaseAllocation","flowApiName":"media_inventory__ReleaseInventorySlots","isVisible":true},{"label":"Extend Reservation","key":"extendReservation","flowApiName":"media_inventory__ExtendSlotRsvDate","isVisible":true},{"label":"Add to Media Plan","key":"addToMediaPlan","flowApiName":"media_inventory__AddSlotsToMediaPlan","isVisible":true}]}}';
        configsToCreate.add(new AdAvailabilityViewConfig(
            Name = 'Inventory Calendar Demand Details',
            PivotOn = null,
            MediaType = null,
            ConfigurationType = 'Tab',
            IsActive = true,
            ConfigurationKey = 'advertising-inventory-calendar-demand-details',
            ConfigurationValue = jsonValue3
        ));
        
        // RECORD 4: Colour Scheme
        String jsonValue4 = '[{"id":"lciQt","label":"Available","startRange":0,"endRange":22,"color":"#2E844A","recordType":"Default","active":true,"key":"Available"},{"id":"4OQHJ","label":"Partially Available","startRange":23,"endRange":70,"color":"#FE9339","recordType":"Default","active":true,"key":"Partially Available"},{"id":"zKGMD","label":"Booked","startRange":71,"endRange":100,"color":"#BA0517","recordType":"Default","active":true,"key":"Booked"},{"id":"Sxe6b","label":"Overbooked","startRange":101,"endRange":300,"color":"#8E030F","recordType":"Default","active":true,"key":"Overbooked"}]';
        configsToCreate.add(new AdAvailabilityViewConfig(
            Name = 'Availability Status',
            PivotOn = null,
            MediaType = null,
            ConfigurationType = 'Colour Scheme',
            IsActive = true,
            ConfigurationKey = 'Status Colors',
            ConfigurationValue = jsonValue4
        ));
                
        Set<String> existingConfigKeys = new Set<String>();
        for (AdAvailabilityViewConfig config : [SELECT Id, Name, ConfigurationKey FROM AdAvailabilityViewConfig WHERE ConfigurationKey IN :new Set<String>(new List<String>{'advertising-inventory-calendar-template', 'advertising-inventory-calendar-cell-actions', 'advertising-inventory-calendar-demand-action', 'Status Colors'})]) {
            existingConfigKeys.add(config.ConfigurationKey);
        }

        List<AdAvailabilityViewConfig> configsToInsert = new List<AdAvailabilityViewConfig>();
        for (AdAvailabilityViewConfig config : configsToCreate) {
            if (!existingConfigKeys.contains(config.ConfigurationKey)) {
                configsToInsert.add(config);
            } else {
                System.debug('AdAvailabilityViewConfig with ConfigurationKey ' + config.ConfigurationKey + ' already exists, skipping insertion.');
            }
        }

        if (!configsToInsert.isEmpty()) {
            try {
                insert configsToInsert;
                System.debug('Successfully inserted ' + configsToInsert.size() + ' new AdAvailabilityViewConfig records.');
            } catch (DMLException e) {
                System.debug('Error inserting AdAvailabilityViewConfig records: ' + e.getMessage());
            }
        } else {
            System.debug('No new AdAvailabilityViewConfig records to insert.');
        }
        System.debug('AdAvailabilityViewConfig insertion process completed.');
    }
    
    /**
     * @description Main entry point for AdSpace insertion based on media type
     * @param productIds List of Product2 IDs to create AdSpaces for
     * @param dimensions Map containing dimension data for AdSpace creation
     *                  Expected keys: 'MediaChannel', 'AdSpaceSpecification', 'TargetingSegmentValues', 'UnitOfMeasure'
     * 
     * This method determines the media type from the Product2 Family field and delegates
     * to the appropriate media-specific insertion method.
     * 
     * @example
     * List<Id> productIds = new List<Id>{'01tWs000009ERCrIAO'};
     * Map<String,List<String>> dimensions = new Map<String,List<String>>();
     * dimensions.put('MediaChannel', new List<String>{'CNBC Home Page'});
     * dimensions.put('AdSpaceSpecification', new List<String>{'CNBC Skyscraper'});
     * dimensions.put('TargetingSegmentValues', new List<String>{'USA'});
     * dimensions.put('UnitOfMeasure', new List<String>{'Impressions'});
     * InventoryManagementSetup.insertAdSpaces(productIds, dimensions);
     */
    public static void insertAdSpaces(List<Id> productIds, Map<String, List<String>> dimensions) {
        // 1. Fetch the Product family (Media Type), assuming the family is same throughout productIds
        String mediatype;
        List<Product2> products = [SELECT Name, Family FROM Product2 WHERE Id IN :productIds];
        
        if (!products.isEmpty()) {
            mediatype = products.get(0).Family;
        } else {
            System.debug('No products found for the given IDs.');
            mediatype = null;
        }
        
        // Route to appropriate media type handler
        switch on mediatype {
            when 'Digital' {
                insertDigitalAdSpaces(productIds, dimensions);
            }
            when 'Print' {
                insertPrintAdSpaces(productIds, dimensions);
            }
            when 'In Store' {
                insertInStoreAdSpaces(productIds, dimensions);
            }
            when else {
                // TODO: Custom Mediatype - Create your own insertion logic
                System.debug('## User needs to implement custom Ad Space insertion logic for the ' + mediatype + ' mediatype.');
            }
        }
    }
    
    /**
     * @description Creates AdSpace records for Digital media type
     * @param productIds List of Product2 IDs to create AdSpaces for
     * @param dimensions Map containing dimension data for AdSpace creation
     * 
     * This method handles the complete workflow for Digital AdSpace creation:
     * 1. Validates and fetches admin data (MediaChannels, AdSpaceSpecs, Creatives, etc.)
     * 2. Fetches junction objects (AdSpaceSpecProduct, AdSpaceCreativeSizeType)
     * 3. Creates AdSpace records with all required relationships
     * 
     * Expected dimensions:
     * - MediaChannel: List of MediaChannel names
     * - AdSpaceSpecification: List of AdSpaceSpecification names
     * - TargetingSegmentValues: List of targeting segment values
     * - AdCreativeSizeType: (Optional) List of creative size type names
     */
    public static void insertDigitalAdSpaces(List<Id> productIds, Map<String, List<String>> dimensions) {
        System.debug('Starting AdSpace insertion process for Digital mediatype...');

        // Input validation
        if (productIds == null || productIds.isEmpty()) {
            System.debug('No Product2 IDs provided for AdSpace insertion. Skipping.');
            return;
        }
        
        List<AdSpace> adSpacesToInsert = new List<AdSpace>();
        Map<Id, Product2> products = new Map<Id, Product2>([SELECT Id, Name FROM Product2 WHERE Id IN :productIds]);
        
        if (products.isEmpty()) {
            System.debug('No Product2 records found for the provided IDs. Skipping AdSpace insertion.');
            return;
        }

        String mediatype = 'Digital';
        Map<Id, Id> adSpecToMediaChannelMap = new Map<Id, Id>();

        /*
         * PART A: FETCH AND VALIDATE ADMIN DATA
         */
        
        // Step 1.1: Fetch and validate Media Channels
        List<String> mediaChannelNames = dimensions.get('MediaChannel');
        List<MediaChannel> mediaChannels = validateAndExtractMediaChannels(mediaChannelNames, mediatype);
        if (mediaChannels == null || mediaChannels.size() == 0) {
            System.debug('No valid Media Channel records found for the provided input. Skipping AdSpace insertion.');
            return;
        }
        
        List<Id> mediaChannelIds = new List<Id>();
        for (MediaChannel mChannel : mediaChannels) {
            mediaChannelIds.add(mChannel.Id);
        }
        
        // Step 1.2: Fetch and validate AdSpace Specifications
        List<String> adSpecNames = dimensions.get('AdSpaceSpecification');
        List<AdSpaceSpecification> adSpaceSpecs = validateAndExtractAdSpaceSpecs(adSpecNames, mediaChannelIds);
        if (adSpaceSpecs == null || adSpaceSpecs.size() == 0) {
            System.debug('No valid Ad Space Specification records found for the provided input. Skipping AdSpace insertion.');
            return;
        }

        List<String> adSpecsIds = new List<String>();
        for (AdSpaceSpecification aSpec : adSpaceSpecs) {
            adSpecsIds.add((String)aSpec.Id);
            adSpecToMediaChannelMap.put((String)aSpec.Id, aSpec.MediaChannelId);
        }
        
        /* Optional Steps: If user wants to add new dimensions, say AdCreativeSizeType, he can do so
         * by passing the dimension and making sure the linked junction objects exist.
         * For example, in case of AdCreativeSizeType, make sure AdSpaceCreativeSizType object records 
         * exist and are linked to the correct AdSpaceSpecification records.
         * 
         * // Step 1.3: Fetch and validate Creative Size Types (optional)
         * List<String> creativeNames = dimensions.get('AdCreativeSizeType');
         * List<AdCreativeSizeType> adCreatives = validateAndExtractAdCreatives(creativeNames, mediatype);
         *  List<String> adCreativeIds = new List<String>();
         *  for (AdCreativeSizeType aCreative : adCreatives) {
         *      adCreativeIds.add((String)aCreative.Id);
         *  }
         *  
         *  // Step 1.4: Fetch all Locations for random assignment
         *  List<String> locationNames = dimensions.get('Location');
         *  List<Schema.Location> allLocations = [SELECT Id FROM Location where Name in :locationNames];
         *  List<Id> allLocationIds = new List<Id>();
         *  for (Schema.Location loc : allLocations) {
         *      allLocationIds.add(loc.Id);
         *  }
         */

        // Step 1.5: Fetch Unit of Measures records
        List<String> uomNames = dimensions.get('UnitOfMeasure');
        List<UnitOfMeasure> uoms = [SELECT Id FROM UnitOfMeasure where Name in :uomNames];
        List<Id> allUomIds = new List<Id>();
        for (UnitOfMeasure uom : uoms) {
            allUomIds.add(uom.Id);
        }

        // Step 1.6: Fetch and validate Targeting Segment Value records
        List<String> segmentNames = dimensions.get('TargetingSegmentValues');
        List<AdTargetSegmentValue> segmentValueIds = [SELECT Id, TargetCategorySegment.TargetCategoryId 
                                                     FROM AdTargetSegmentValue 
                                                     WHERE NAME IN :segmentNames];
        List<Id> allSegmentValueIds = new List<Id>();
        Map<String, String> segmentToCategoryMap = new Map<String, String>();
        for (AdTargetSegmentValue atsv : segmentValueIds) {
            allSegmentValueIds.add(atsv.Id);
            segmentToCategoryMap.put(atsv.Id, atsv.TargetCategorySegment.TargetCategoryId);
        }

        /*
         * PART B: FETCH JUNCTION OBJECTS
         */
        
        // Step 2.1: Fetch AdSpaceSpecProduct junction records
        List<AdSpaceSpecProduct> adSpaceProducts = [SELECT Id, AdSpaceSpecificationId, ProductId, 
                                                           AdSpaceSpecification.Name, Product.Name 
                                                   FROM AdSpaceSpecProduct 
                                                   WHERE AdSpaceSpecificationId IN :adSpecsIds 
                                                   AND ProductId IN :productIds];
        
        
        
        /* // Step 2.2: Fetch AdSpaceCreativeSizeType junction records (if creatives provided)
         * 
         * List<AdSpaceCreativeSizeType> adSpaceCreatives = new List<AdSpaceCreativeSizeType>();
         * if (adCreativeIds != null && !adCreativeIds.isEmpty()) {
         *    System.debug('## User has provided the creative Ids: ' + adCreativeIds);
         *   adSpaceCreatives = [SELECT Id, AdSpaceSpecificationId, AdSpaceSpecification.Name, AdCreativeSizeTypeId 
         *                      FROM AdSpaceCreativeSizeType 
         *                      WHERE AdSpaceSpecificationId IN :adSpecsIds 
         *                      AND AdCreativeSizeTypeId IN :adCreativeIds];
         * 
         *  }
         */
        
        // Initialize supporting data structures
        Map<Id, UnitOfMeasure> uomById = new Map<Id, UnitOfMeasure>([SELECT Id, Name FROM UnitOfMeasure WHERE Id IN :allUomIds]);
        List<String> ineligibleIndustries = new List<String>{'Insurance', 'Healthcare', 'Education', 'Agriculture', 'Banking'};
        Map<String, Integer> adSpaceCountersByProductAdSpaceSpec = new Map<String, Integer>();
        
        // Clear and initialize counters
        adSpaceCountersByProductAdSpaceSpec.clear();
        Map<String, AdSpace> existingAdSpaces = new Map<String, AdSpace>();
        for (AdSpace ads : [SELECT Id, Name FROM AdSpace]) {
            existingAdSpaces.put(ads.Name, ads);
            adSpaceNameToIdMap.put(ads.Name, ads.Id);
        }

        /*
         * PART C: CREATE ADSPACE RECORDS
         */
        System.debug('## Trying to insert records...');
        // Process each product and create AdSpace combinations
        for (Id prodId : products.keySet()) {
            String productName = products.get(prodId).Name;
            for (AdSpaceSpecProduct adSpaceProduct : adSpaceProducts) {
                if (adSpaceProduct.AdSpaceSpecificationId != null && adSpaceProduct.ProductId != null) {
                    // Skip if this junction record doesn't belong to current product
                    if (adSpaceProduct.ProductId != prodId) {
                        continue;
                    }
                    
                    String adSpaceSpecName = adSpaceProduct.AdSpaceSpecification.Name;
                    String counterKey = prodId + '_' + adSpaceProduct.AdSpaceSpecificationId;
                    Integer currentCounter = adSpaceCountersByProductAdSpaceSpec.get(counterKey);
                    if (currentCounter == null) {
                        currentCounter = 0;
                    }
                    // Create AdSpace for each combination of UOM and Segment Value
                    for (Id uomId : allUomIds) {
                        for (Id segmentValueId : allSegmentValueIds) {
                            currentCounter++; 
                            adSpaceCountersByProductAdSpaceSpec.put(counterKey, currentCounter); 
                            String adSpaceName = productName + ' ' + adSpaceSpecName + ' ' + currentCounter; 
                            
                            // Skip if AdSpace already exists
                            if (!existingAdSpaces.containsKey(adSpaceName)) {
                                // Generate random values for variety
                                Integer randomIndex = Math.mod(Math.abs(Crypto.getRandomInteger()), ineligibleIndustries.size()); 
                                String randomIndustry = ineligibleIndustries.get(randomIndex);
                                
                                // Additional validation check
                                if (adSpaceProduct.AdSpaceSpecificationId == null) {
                                    System.debug('Skipping AdSpace creation for junction record ' + productName + ' due to missing AdSpaceSpecificationId or ProductId.');
                                    continue;
                                }
                                
                                // Create new AdSpace record
                                AdSpace newAdSpace = new AdSpace(
                                    Name = adSpaceName,
                                    ProductId = prodId,
                                    AdSpaceSpecificationId = adSpaceProduct.AdSpaceSpecificationId,
                                    MediaChannelId = adSpecToMediaChannelMap.get(adSpaceProduct.AdSpaceSpecificationId),
                                    AdTargetSegmentValueId = segmentValueId,
                                    AdTargetCategoryId = segmentToCategoryMap.get(segmentValueId),
                                    MediaType = mediatype,
                                    UnitOfMeasureId = uomId,
                                    IneligibleIndustry = randomIndustry
                                );
                                adSpacesToInsert.add(newAdSpace);
                            } else {
                                System.debug('AdSpace already exists, skipping: ' + adSpaceName);
                            }
                        }
                    }
                } else {
                    System.debug('Skipping AdSpace creation for junction record ' + productName + ' due to missing AdSpaceSpecificationId or ProductId.');
                }                
            }
        }

        // Insert all created AdSpace records
        if (!adSpacesToInsert.isEmpty()) {
            try {
                System.debug('Trying to insert ' + adSpacesToInsert.size() + ' new AdSpace records.');
                insert adSpacesToInsert;
                System.debug('Successfully inserted ' + adSpacesToInsert.size() + ' new AdSpace records.');
                
                // Update the name-to-ID mapping
                for (AdSpace ads : adSpacesToInsert) {
                    adSpaceNameToIdMap.put(ads.Name, ads.Id);
                }
            } catch (DMLException e) {
                System.debug('Error inserting AdSpace records: ' + e.getMessage());
            }
        } else {
            System.debug('No new AdSpace records to insert.');
        }
        
        System.debug('AdSpace insertion process completed.');
    }
    
    /**
     * @description Creates AdSpace records for Print media type
     * @param productIds List of Product2 IDs to create AdSpaces for
     * @param dimensions Map containing dimension data for AdSpace creation
     * 
     * This method handles Print-specific AdSpace creation including:
     * - Media Print Issues (specific to print media)
     * - Print-specific junction objects
     * 
     * Expected dimensions:
     * - MediaChannel: List of MediaChannel names
     * - MediaPrintIssue: List of MediaPrintIssue names
     * - AdSpaceSpecification: List of AdSpaceSpecification names
     */
    public static void insertPrintAdSpaces(List<Id> productIds, Map<String, List<String>> dimensions) {
        System.debug('Starting AdSpace insertion process for Print mediatype...');

        // Input validation
        if (productIds == null || productIds.isEmpty()) {
            System.debug('No Product2 IDs provided for AdSpace insertion. Skipping.');
            return;
        }
        
        Map<Id, Product2> products = new Map<Id, Product2>([SELECT Id, Name FROM Product2 WHERE Id IN :productIds]);
        if (products.isEmpty()) {
            System.debug('No Product2 records found for the provided IDs. Skipping AdSpace insertion.');
            return;
        }
        
        String mediatype = 'Print';
        Map<Id, Id> adSpecToMediaChannelMap = new Map<Id, Id>();
        List<AdSpace> adSpacesToInsert = new List<AdSpace>();

        // Step 1.1: Fetch and validate Media Channels
        List<String> mediaChannelNames = dimensions.get('MediaChannel');
        List<MediaChannel> mediaChannels = validateAndExtractMediaChannels(mediaChannelNames, mediatype);
        if (mediaChannels == null || mediaChannels.size() == 0) {
            System.debug('No valid Media Channel records found for the provided input. Skipping AdSpace insertion.');
            return;
        }
        
        List<Id> mediaChannelIds = new List<Id>();
        for (MediaChannel mChannel : mediaChannels) {
            mediaChannelIds.add(mChannel.Id);
        }
        
        /*// Step 1.2: Fetch and validate Media Print Issues
        List<String> printIssueNames = dimensions.get('MediaPrintIssue');
        List<MediaPrintIssue> mediaPrintIssues = validateAndExtractMediaPrintIssues(printIssueNames, mediaChannelIds);
        List<Id> mediaPrintIssueIds = new List<Id>();
        for (MediaPrintIssue mpi : mediaPrintIssues) {
            mediaPrintIssueIds.add(mpi.Id);
        }*/
        
        // Step 1.3: Fetch and validate AdSpace Specifications
        List<String> adSpecNames = dimensions.get('AdSpaceSpecification');
        List<AdSpaceSpecification> adSpaceSpecs = validateAndExtractAdSpaceSpecs(adSpecNames, mediaChannelIds);
        if (adSpaceSpecs == null || adSpaceSpecs.size() == 0) {
            System.debug('No valid Ad Space Specification records found for the provided input. Skipping AdSpace insertion.');
            return;
        }
        
        List<String> adSpecsIds = new List<String>();
        for (AdSpaceSpecification aSpec : adSpaceSpecs) {
            adSpecsIds.add((String)aSpec.Id);
            adSpecToMediaChannelMap.put((String)aSpec.Id, aSpec.MediaChannelId);
        }
        
        // Step 1.4: Fetch all Unit of Measures
        List<String> uomNames = dimensions.get('UnitOfMeasure');
        List<UnitOfMeasure> uoms = [SELECT Id FROM UnitOfMeasure where Name in :uomNames];
        List<Id> allUomIds = new List<Id>();
        for (UnitOfMeasure uom : uoms) {
            allUomIds.add(uom.Id);
        }
        
        /*
         * PART B: FETCH JUNCTION OBJECTS
         */
        
        // Step 2.1: Fetch AdSpaceSpecProduct junction records
        List<AdSpaceSpecProduct> adSpaceProducts = [SELECT Id, AdSpaceSpecificationId, ProductId, 
                                                           AdSpaceSpecification.Name, Product.Name 
                                                   FROM AdSpaceSpecProduct 
                                                   WHERE AdSpaceSpecificationId IN :adSpecsIds 
                                                   AND ProductId IN :productIds];
        
        // Step 2.2: Fetch AdSpecMediaPrintIssue junction records
        /*List<AdSpecMediaPrintIssue> adSpecPrintIssues = [SELECT Id, AdSpaceSpecificationId, AdSpaceSpecification.Name, 
                                                                MediaPrintIssue.Id, MediaPrintIssue.Name 
                                                        FROM AdSpecMediaPrintIssue 
                                                        WHERE AdSpaceSpecificationId IN :adSpecsIds 
                                                        AND MediaPrintIssueId IN :mediaPrintIssueIds];*/

        // Initialize supporting data structures
        Map<Id, UnitOfMeasure> uomById = new Map<Id, UnitOfMeasure>([SELECT Id, Name FROM UnitOfMeasure WHERE Id IN :allUomIds]);
        List<String> ineligibleIndustries = new List<String>{'Insurance', 'Healthcare', 'Education', 'Agriculture', 'Banking'};
        Map<String, Integer> adSpaceCountersByProductAdSpaceSpec = new Map<String, Integer>();
        
        // Clear and initialize counters
        adSpaceCountersByProductAdSpaceSpec.clear();
        Map<String, AdSpace> existingAdSpaces = new Map<String, AdSpace>();
        for (AdSpace ads : [SELECT Id, Name FROM AdSpace]) {
            existingAdSpaces.put(ads.Name, ads);
            adSpaceNameToIdMap.put(ads.Name, ads.Id);
        }
        
        /*
         * PART C: CREATE ADSPACE RECORDS
         */
        
        // Process each product and create AdSpace combinations
        for (Id prodId : products.keySet()) {
            String productName = products.get(prodId).Name;
            
            for (AdSpaceSpecProduct adSpaceProduct : adSpaceProducts) {
                if (adSpaceProduct.AdSpaceSpecificationId != null && adSpaceProduct.ProductId != null) {
                    // Skip if this junction record doesn't belong to current product
                    if (adSpaceProduct.ProductId != prodId) {
                        continue;
                    }
                    
                    String adSpaceSpecName = adSpaceProduct.AdSpaceSpecification.Name;
                    String counterKey = prodId + '_' + adSpaceProduct.AdSpaceSpecificationId;
                    Integer currentCounter = adSpaceCountersByProductAdSpaceSpec.get(counterKey);
                    if (currentCounter == null) {
                        currentCounter = 0;
                    }
                    
                    // Create AdSpace for each UOM (Print doesn't use segment values)
                    for (Id uomId : allUomIds) {
                        currentCounter++; 
                        adSpaceCountersByProductAdSpaceSpec.put(counterKey, currentCounter); 
                        String adSpaceName = productName + ' ' + adSpaceSpecName + ' ' + currentCounter; 
                        
                        // Skip if AdSpace already exists
                        if (!existingAdSpaces.containsKey(adSpaceName)) {
                            // Generate random industry for variety
                            Integer randomIndex = Math.mod(Math.abs(Crypto.getRandomInteger()), ineligibleIndustries.size()); 
                            String randomIndustry = ineligibleIndustries.get(randomIndex);
                            
                            // Additional validation check
                            if (adSpaceProduct.AdSpaceSpecificationId == null) {
                                System.debug('Skipping AdSpace creation for junction record ' + productName + ' due to missing AdSpaceSpecificationId or ProductId.');
                                continue;
                            }
                            
                            // Create new AdSpace record with Print-specific fields
                            AdSpace newAdSpace = new AdSpace(
                                Name = adSpaceName,
                                ProductId = prodId,
                                AdSpaceSpecificationId = adSpaceProduct.AdSpaceSpecificationId,
                                MediaChannelId = adSpecToMediaChannelMap.get(adSpaceProduct.AdSpaceSpecificationId),
                                MediaType = mediatype,
                                UnitOfMeasureId = uomId,
                                IneligibleIndustry = randomIndustry
                            );
                            adSpacesToInsert.add(newAdSpace);
                        } else {
                            System.debug('AdSpace already exists, skipping: ' + adSpaceName);
                        }
                    }
                }
            }
        }
        
        // Insert all created AdSpace records
        if (!adSpacesToInsert.isEmpty()) {
            try {
                System.debug('Trying to insert ' + adSpacesToInsert.size() + ' new AdSpace records.');
                insert adSpacesToInsert;
                System.debug('Successfully inserted ' + adSpacesToInsert.size() + ' new AdSpace records.');
                
                // Update the name-to-ID mapping
                for (AdSpace ads : adSpacesToInsert) {
                    adSpaceNameToIdMap.put(ads.Name, ads.Id);
                }
            } catch (DMLException e) {
                System.debug('Error inserting AdSpace records: ' + e.getMessage());
            }
        } else {
            System.debug('No new AdSpace records to insert.');
        }
        
        System.debug('AdSpace insertion process completed.');
    }
    
    /**
     * @description Placeholder method for In Store AdSpace creation
     * @param productIds List of Product2 IDs to create AdSpaces for
     * @param dimensions Map containing dimension data for AdSpace creation
     * 
     * TODO: Implement In Store specific AdSpace creation logic
     */
    public static void insertInStoreAdSpaces(List<Id> productIds, Map<String, List<String>> dimensions) {
        System.debug('Starting AdSpace insertion process for In-Store mediatype...');

        // Input validation
        if (productIds == null || productIds.isEmpty()) {
            System.debug('No Product2 IDs provided for AdSpace insertion. Skipping.');
            return;
        }
        
        Map<Id, Product2> products = new Map<Id, Product2>([SELECT Id, Name FROM Product2 WHERE Id IN :productIds]);
        if (products.isEmpty()) {
            System.debug('No Product2 records found for the provided IDs. Skipping AdSpace insertion.');
            return;
        }
        
        String mediatype = 'In Store';
        List<AdSpace> adSpacesToInsert = new List<AdSpace>();

        // Step 1.1: Fetch and validate AdSpace Specifications
        List<String> adSpecNames = dimensions.get('AdSpaceSpecification');
        List<AdSpaceSpecification> adSpaceSpecs = [SELECT ID, Name 
                                                  FROM AdSpaceSpecification 
                                                  WHERE Name IN :adSpecNames 
                                                  AND IsActive = true];
        if (adSpaceSpecs == null || adSpaceSpecs.size() == 0) {
            System.debug('No valid Ad Space Specification records found for the provided input. Skipping AdSpace insertion.');
            return;
        }
        
        List<String> adSpecsIds = new List<String>();
        for (AdSpaceSpecification aSpec : adSpaceSpecs) {
            adSpecsIds.add((String)aSpec.Id);
        }
        
        System.debug('## AdSpaceIds: ' + adSpecsIds);
        
        // Step 1.2: Fetch and validate Locations
        List<String> locationNames = dimensions.get('Location');
        List<Schema.Location> locations = [SELECT ID
                                           FROM Location 
                                           WHERE Name IN :locationNames];
        if (locations == null || locations.size() == 0) {
            System.debug('No valid Location records found for the provided input. Skipping AdSpace insertion.');
            return;
        }
        
        List<String> locationIds = new List<String>();
        for (Schema.Location loc : locations) {
            locationIds.add((String)loc.Id);
        }
        System.debug('## locationIds: ' + locationIds);
        
        // Step 1.3: Fetch all Unit of Measures
        List<String> uomNames = dimensions.get('UnitOfMeasure');
        List<UnitOfMeasure> allUoms = [SELECT Id FROM UnitOfMeasure where Name in :uomNames];
        List<Id> allUomIds = new List<Id>();
        for (UnitOfMeasure uom : allUoms) {
            allUomIds.add(uom.Id);
        }
        System.debug('## allUomIds: ' + allUomIds);
        
        /*
         * PART B: FETCH JUNCTION OBJECTS
         */
        
        // Step 2.1: Fetch AdSpaceSpecProduct junction records
        List<AdSpaceSpecProduct> adSpaceProducts = [SELECT Id, AdSpaceSpecificationId, ProductId, 
                                                           AdSpaceSpecification.Name, Product.Name 
                                                   FROM AdSpaceSpecProduct 
                                                   WHERE AdSpaceSpecificationId IN :adSpecsIds 
                                                   AND ProductId IN :productIds];
        System.debug('## adSpaceProducts: ' + adSpaceProducts);
        
        // Initialize supporting data structures
        Map<Id, UnitOfMeasure> uomById = new Map<Id, UnitOfMeasure>([SELECT Id, Name FROM UnitOfMeasure WHERE Id IN :allUomIds]);
        List<String> ineligibleIndustries = new List<String>{'Insurance', 'Healthcare', 'Education', 'Agriculture', 'Banking'};
        Map<String, Integer> adSpaceCountersByProductAdSpaceSpec = new Map<String, Integer>();
        
        // Clear and initialize counters
        adSpaceCountersByProductAdSpaceSpec.clear();
        Map<String, AdSpace> existingAdSpaces = new Map<String, AdSpace>();
        for (AdSpace ads : [SELECT Id, Name FROM AdSpace]) {
            existingAdSpaces.put(ads.Name, ads);
            adSpaceNameToIdMap.put(ads.Name, ads.Id);
        }
        
        /*
         * PART C: CREATE ADSPACE RECORDS
         */
        
        // Process each product and create AdSpace combinations
        for (Id prodId : products.keySet()) {
            String productName = products.get(prodId).Name;
            for (AdSpaceSpecProduct adSpaceProduct : adSpaceProducts) {
                if (adSpaceProduct.AdSpaceSpecificationId != null && adSpaceProduct.ProductId != null) {
                    // Skip if this junction record doesn't belong to current product
                    if (adSpaceProduct.ProductId != prodId) {
                        continue;
                    }
                    
                    String adSpaceSpecName = adSpaceProduct.AdSpaceSpecification.Name;
                    String counterKey = prodId + '_' + adSpaceProduct.AdSpaceSpecificationId;
                    Integer currentCounter = adSpaceCountersByProductAdSpaceSpec.get(counterKey);
                    if (currentCounter == null) {
                        currentCounter = 0;
                    }
                    // Create AdSpace for each Location permutation
                    for(Id locId : locationIds) {
                        // Create AdSpace for each UOM permutation
                         for (Id uomId : allUomIds) {
                            currentCounter++; 
                            adSpaceCountersByProductAdSpaceSpec.put(counterKey, currentCounter); 
                            String adSpaceName = productName + ' ' + adSpaceSpecName + ' ' + currentCounter; 
                            
                            // Skip if AdSpace already exists
                            if (!existingAdSpaces.containsKey(adSpaceName)) {
                                // Generate random industry for variety
                                Integer randomIndex = Math.mod(Math.abs(Crypto.getRandomInteger()), ineligibleIndustries.size()); 
                                String randomIndustry = ineligibleIndustries.get(randomIndex);
                                
                                // Additional validation check
                                if (adSpaceProduct.AdSpaceSpecificationId == null) {
                                    System.debug('Skipping AdSpace creation for junction record ' + productName + ' due to missing AdSpaceSpecificationId or ProductId.');
                                    continue;
                                }
                                
                                // Create new AdSpace record with Print-specific fields
                                AdSpace newAdSpace = new AdSpace(
                                    Name = adSpaceName,
                                    ProductId = prodId,
                                    AdSpaceSpecificationId = adSpaceProduct.AdSpaceSpecificationId,
                                    MediaType = mediatype,
                                    UnitOfMeasureId = uomId,
                                    IneligibleIndustry = randomIndustry,
                                    LocationId = locId
                                );
                                adSpacesToInsert.add(newAdSpace);
                            } else {
                                System.debug('AdSpace already exists, skipping: ' + adSpaceName);
                            }
                        }   
                    }                    
                }
            }
        }
        
        // Insert all created AdSpace records
        if (!adSpacesToInsert.isEmpty()) {
            try {
                System.debug('Trying to insert ' + adSpacesToInsert.size() + ' new AdSpace records.');
                insert adSpacesToInsert;
                System.debug('Successfully inserted ' + adSpacesToInsert.size() + ' new AdSpace records.');
                
                // Update the name-to-ID mapping
                for (AdSpace ads : adSpacesToInsert) {
                    adSpaceNameToIdMap.put(ads.Name, ads.Id);
                }
            } catch (DMLException e) {
                System.debug('Error inserting AdSpace records: ' + e.getMessage());
            }
        } else {
            System.debug('No new AdSpace records to insert.');
        }
        
        System.debug('AdSpace insertion process completed.');
    }
    
    /**
     * @description Initiates batch processing for AdSpaceCapacity creation
     * @param dailyCount Number of daily capacity records to create (default: 90)
     * @param weeklyCount Number of weekly capacity records to create (default: 52)
     * @param monthlyCount Number of monthly capacity records to create (default: 12)
     * @param mediaType Media Type of the Adspaces to create capacity records of
     * 
     * This method creates capacity records for all existing AdSpaces using a batch process
     * to handle large volumes efficiently.
     * 
     * @example
     * InventoryManagementSetup.insertAdSpaceCapacities(90, 52, 12, 'Print');
     */
    public static void insertAdSpaceCapacities(Integer dailyCount, Integer weeklyCount, Integer monthlyCount, String mediaType, List<Id> productIds) {
        System.debug('Initiating AdSpaceCapacity batch insertion process...');
        
        // Fetch all existing AdSpaces
        List<AdSpace> adSpaces = [SELECT Id FROM AdSpace where Product.Family =:mediaType AND ProductId IN :productIds];
        List<Id> adSpaceIdsToProcess = new List<Id>();
        for (AdSpace aSpace : adSpaces) {
            adSpaceIdsToProcess.add(aSpace.Id);
        }
        
        // Set default values if not provided
        if (dailyCount == null) dailyCount = 90;
        if (weeklyCount == null) weeklyCount = 52;
        if (monthlyCount == null) monthlyCount = 12;
        
        // Execute batch job
        System.debug('## AdSpaces: ' + adSpaceIdsToProcess);
        AdSpaceCapacityBatch capacityBatch = new AdSpaceCapacityBatch(
            dailyCount,
            weeklyCount,
            monthlyCount,
            adSpaceIdsToProcess
        );
        Database.executeBatch(capacityBatch, 25);
        System.debug('AdSpaceCapacity batch job initiated.');
    }
    
    // ===========================================
    // PRIVATE HELPER METHODS
    // ===========================================
    
    /**
     * @description Validates and extracts MediaChannel records based on provided names and media type
     * @param mediaChannelNames List of MediaChannel names to validate
     * @param mediatype Media type to filter by
     * @return List<MediaChannel> Valid MediaChannel records
     * 
     * This method ensures that all provided MediaChannel names exist, are active,
     * and belong to the specified media type.
     */
    private static List<MediaChannel> validateAndExtractMediaChannels(List<String> mediaChannelNames, String mediatype) {
        // Query MediaChannels with validation criteria
        List<MediaChannel> mediaChannelList = [SELECT Id 
                                              FROM MediaChannel 
                                              WHERE Name IN :mediaChannelNames 
                                              AND IsActive = true 
                                              AND MediaType = :mediatype];
        
        // Validate that all requested channels were found
        if (mediaChannelList != null && mediaChannelList.size() == mediaChannelNames.size()) {
            return mediaChannelList;
        } else if (mediaChannelList != null && mediaChannelList.size() != mediaChannelNames.size()) {
            System.debug('AdSpaces cannot be created for one or more Media Channels due to config issue.');
            return mediaChannelList;
        } else {
            return new List<MediaChannel>();
        }
    }
    
    /**
     * @description Validates and extracts AdSpaceSpecification records based on provided names and media channels
     * @param adSpecNames List of AdSpaceSpecification names to validate
     * @param mediaChannelIds List of valid MediaChannel IDs to filter by
     * @return List<AdSpaceSpecification> Valid AdSpaceSpecification records
     * 
     * This method ensures that all provided AdSpaceSpecification names exist, are active,
     * and belong to the specified media channels.
     */
    private static List<AdSpaceSpecification> validateAndExtractAdSpaceSpecs(List<String> adSpecNames, List<Id> mediaChannelIds) {
        List<AdSpaceSpecification> adSpaceSpecs = [SELECT ID, Name, Product2Id, MediaChannelId 
                                                  FROM AdSpaceSpecification 
                                                  WHERE Name IN :adSpecNames 
                                                  AND IsActive = true 
                                                  AND MediaChannelId IN :mediaChannelIds];

        // Validate that all requested specs were found
        if (adSpaceSpecs != null && adSpaceSpecs.size() == adSpecNames.size()) {
            return adSpaceSpecs;
        } else if (adSpaceSpecs != null && adSpaceSpecs.size() != adSpecNames.size()) {
            System.debug('AdSpaces cannot be created for one or more Ad Space Specs due to config issue.');
            return adSpaceSpecs;
        } else {
            return new List<AdSpaceSpecification>();
        }
    }
    
    /**
     * @description Validates and extracts AdCreativeSizeType records based on provided names and media type
     * @param adCreatives List of AdCreativeSizeType names to validate (can be null/empty)
     * @param mediatype Media type to filter by
     * @return List<AdCreativeSizeType> Valid AdCreativeSizeType records
     * 
     * This method ensures that all provided AdCreativeSizeType names exist and belong to the specified media type.
     * Returns empty list if no creatives provided.
     */
    private static List<AdCreativeSizeType> validateAndExtractAdCreatives(List<String> adCreatives, String mediatype) {
        // Return empty list if no creatives provided
        if (adCreatives == null || adCreatives.isEmpty()) {
            return new List<AdCreativeSizeType>();
        }
        
        // Query AdCreativeSizeType with media type validation
        String searchPattern = '%' + mediatype + '%';
        List<AdCreativeSizeType> adCreativeList = [SELECT Id 
                                                  FROM AdCreativeSizeType 
                                                  WHERE Name IN :adCreatives 
                                                  AND MediaType LIKE :searchPattern];
        
        // Validate that all requested creatives were found
        if (adCreativeList != null && adCreativeList.size() == adCreatives.size()) {
            return adCreativeList;
        } else if (adCreativeList != null && adCreativeList.size() != adCreatives.size()) {
            System.debug('AdSpaces cannot be created for one or more Ad Creatives due to config issue.');
            return adCreativeList;
        } else {
            return new List<AdCreativeSizeType>();
        }
    }
    
    /**
     * @description Validates and extracts MediaPrintIssue records based on provided names and media channels
     * @param printIssueNames List of MediaPrintIssue names to validate
     * @param mediaChannelIds List of valid MediaChannel IDs to filter by
     * @return List<MediaPrintIssue> Valid MediaPrintIssue records
     * 
     * This method ensures that all provided MediaPrintIssue names exist and belong to the specified media channels.
     */
    private static List<MediaPrintIssue> validateAndExtractMediaPrintIssues(List<String> printIssueNames, List<Id> mediaChannelIds) {
        List<MediaPrintIssue> printIssues = [SELECT Id, MediaChannelId 
                                            FROM MediaPrintIssue 
                                            WHERE Name IN :printIssueNames 
                                            AND MediaChannelId IN :mediaChannelIds];
        
        // Validate that all requested print issues were found
        if (printIssues != null && printIssues.size() == printIssueNames.size()) {
            return printIssues;
        } else if (printIssues != null && printIssues.size() != printIssueNames.size()) {
            System.debug('AdSpaces cannot be created for one or more Media Print Issues due to config issue.');
            return printIssues;
        } else {
            return new List<MediaPrintIssue>();
        }
    }
}

```