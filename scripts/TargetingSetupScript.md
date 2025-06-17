# Targeting Setup Script

Create a new apex class and copy the below code.

```
public class GenerateTargetingSeedData {
    private static Map<String, String> oldNewSegId = new Map<String, String>();
    private static Map<String, String> nameToCategoryIdMap = new Map<String, String>();
    private static Map<String, String> nameToSegmentIdMap = new Map<String, String>();
    private static Map<String, String> nameToValueIdMap = new Map<String, String>();

    public class CategoryList {
        public List<Category> categoryList;
    }
    
    public class Category {
        public String Code;
        public String Description;
        public Integer DisplaySequence;
        public String Id;
        public Boolean IsActive;
        public Boolean IsAvailableForSelfService;
        public String MediaType;
        public String Name;
        public String ParentAdTargetCategoryId;
    }
    
    public class SegmentList {
        public List<Segment> SegmentList;
    }
    
    public class Segment {
        public String Code;
        public String DataType;
        public String DependentCategorySegmentCode;
        public String Description;
        public Integer DisplaySequence;
        public String DisplayType;
        public Boolean IsActive;
        public Boolean IsAvailableForSelfService;
        public String MediaType;
        public String Name;
        public String ProductId;
    }
    
    public class ValueList {
        public List<Value> ValueList;
    }
    
    public class Value {
        public String SegmentCode;
        public String ServerIdentifier;
        public String Label;
        public String RefServerIdentifierId;
    }
    
    public class AvailMapping {
        public List<AvailData> availList;
    }
    
    public class AvailData {
        public String CategoryCode;
        public String SegmentCode;
    }
    
    public static String suffix = '';
    
    public static void clearData(Boolean sure) {
        if (sure) {
            System.debug('=== ALERT ===');
            System.debug('=== ALL Targeting data is deleted ===');
            delete [SELECT Id FROM AdAvailableTargetSegmentValue];
            delete [SELECT Id FROM AdAvailableTargetSegment];
            delete [SELECT Id FROM AdTargetSegmentValue];
            delete [SELECT Id FROM AdTargetCategorySegment];
            delete [SELECT Id FROM AdTargetCategory];
        } else {
            System.debug('looks like you are not sure for clearing data. if sure, execute : GenerateTargetingSeedData.clearData(true)');
        }
    }
    
    public static void processCategoryCSV(String fileName) {
        try {
            ContentVersion cv = [SELECT Id, Title, VersionData FROM ContentVersion WHERE Title = :fileName ORDER BY CreatedDate DESC LIMIT 1];
            String fileContent = cv.VersionData.toString();
            List<Map<String, Object>> categoryDataList = parseCategoryCSV(fileContent);
            createCategories(categoryDataList);
        } catch (Exception e) {
            System.debug('Error processing CSV file: ' + e.getMessage());
        }
    }
    
    private static List<Map<String, Object>> parseCategoryCSV(String csvContent) {
        List<Map<String, Object>> categoryDataList = new List<Map<String, Object>>();
        List<String> lines = csvContent.split('\n');
        
        if (lines.size() < 2) return categoryDataList;
        
        
        String delimiter = lines[0].contains(';') ? ';' : ',';
        
        List<String> headers = lines[0].split(delimiter);
        
        for (Integer i = 1; i < lines.size(); i++) {
            List<String> values = lines[i].split(delimiter);
            
            
            while (values.size() < headers.size()) {
                values.add('');
            }
            
            
            if (values.size() != headers.size()) {
                System.debug('Skipping row ' + i + ' due to mismatched columns. Headers: ' + headers.size() + ', Values: ' + values.size());
                continue;
            }
            
            Map<String, Object> categoryData = new Map<String, Object>();
            for (Integer j = 0; j < headers.size(); j++) {
                categoryData.put(headers[j].trim(), values[j].trim());
            }
            
            try {
                categoryData.put('DisplaySequence', Integer.valueOf((String)categoryData.get('DisplaySequence')));
            } catch (Exception e) {
                System.debug('Error parsing DisplaySequence in row ' + i + ': ' + e.getMessage());
                continue;
            }
            
            categoryData.put('IsActive', ((String) categoryData.get('IsActive')).toLowerCase() == 'true');
            categoryData.put('IsAvailableForSelfService', ((String) categoryData.get('IsAvailableForSelfService')).toLowerCase() == 'true');
            categoryDataList.add(categoryData);
        }
        
        return categoryDataList;
    }
    
    
    public static void createCategories(List<Map<String, Object>> categoryDataList) {
        List<String> processedCategoryIds = new List<String>();
        
        for (Map<String, Object> categoryData : categoryDataList) {
            
            if(((String)categoryData.get('Id')).length()==15 ||    ((String)categoryData.get('Id')).length()==18 )
                
                try {
                    String insertedId = insertCategory(
                        (String) categoryData.get('Code'),
                        (String) categoryData.get('Description'),
                        (Integer) categoryData.get('DisplaySequence'),
                        (Boolean) categoryData.get('IsActive'),
                        (Boolean) categoryData.get('IsAvailableForSelfService'),
                        (String) categoryData.get('MediaType'),
                        (String) categoryData.get('Name'),
                        (String) categoryData.get('ParentAdTargetCategoryId')
                    );
                    
                    if (insertedId != null) {
                        processedCategoryIds.add(insertedId);
                        nameToCategoryIdMap.put((String) categoryData.get('Name') + suffix, insertedId);
                    }
                } catch (Exception e) {
                    System.debug('Error processing category (insert/update): ' + e.getMessage());
                }
        }
        
        System.debug('Processed ' + processedCategoryIds.size() + ' categories successfully.');
    }
    
    public static void processSegmentCSV(String fileName) {
        try {
            ContentVersion cv = [SELECT Id, Title, VersionData FROM ContentVersion WHERE Title = :fileName ORDER BY CreatedDate DESC LIMIT 1];
            String fileContent = cv.VersionData.toString();
            List<Map<String, Object>> segmentDataList = parseSegmentCSV(fileContent);
            createSegments(segmentDataList);
        } catch (Exception e) {
            System.debug('Error processing Segment CSV file: ' + e.getMessage());
        }
    }
    
    private static List<Map<String, Object>> parseSegmentCSV(String csvContent) {
        List<Map<String, Object>> segmentDataList = new List<Map<String, Object>>();
        List<String> lines = csvContent.split('\n');
        
        if (lines.size() < 2) return segmentDataList;
        
        
        String delimiter = lines[0].contains(';') ? ';' : ',';
        
        List<String> headers = lines[0].split(delimiter);
        
        for (Integer i = 1; i < lines.size(); i++) {
            List<String> values = lines[i].split(delimiter);
            
            
            while (values.size() < headers.size()) {
                values.add('');
            }
            
            
            if (values.size() != headers.size()) {
                System.debug('Skipping row ' + i + ' due to mismatched columns. Headers: ' + headers.size() + ', Values: ' + values.size());
                continue;
            }
            
            Map<String, Object> segmentData = new Map<String, Object>();
            for (Integer j = 0; j < headers.size(); j++) {
                segmentData.put(headers[j].trim(), values[j].trim());
            }
            
            try {
                segmentData.put('DisplaySequence', Integer.valueOf((String)segmentData.get('DisplaySequence')));
            } catch (Exception e) {
                System.debug('Error parsing DisplaySequence: ' + e.getMessage());
                continue;
            }
            
            segmentData.put('IsActive', ((String) segmentData.get('IsActive')).toLowerCase() == 'true');
            segmentData.put('IsAvailableForSelfService', ((String) segmentData.get('IsAvailableForSelfService')).toLowerCase() == 'true');
            segmentDataList.add(segmentData);
        }
        
        return segmentDataList;
    }
    
    
    public static void createSegments(List<Map<String, Object>> segmentDataList) {
        for (Map<String, Object> segmentData : segmentDataList) {
            try {
                String segId = (String) segmentData.get('Id');
                if (segId != null && (segId.length() == 15 || segId.length() == 18)) {
                    updateSegment(
                        segId,
                        (String) segmentData.get('Code'),
                        (String) segmentData.get('DataType'),
                        (String) segmentData.get('Description'),
                        (Integer) segmentData.get('DisplaySequence'),
                        (String) segmentData.get('DisplayType'),
                        (Boolean) segmentData.get('IsActive'),
                        (Boolean) segmentData.get('IsAvailableForSelfService'),
                        (String) segmentData.get('MediaType'),
                        (String) segmentData.get('Name'),
                        (String) segmentData.get('ProductId')
                    );
                    oldNewSegId.put(segId, segId); 
                } else {
                    String newSegmentId = insertSegment(
                        (String) segmentData.get('Code'),
                        (String) segmentData.get('DataType'),
                        (String) segmentData.get('Description'),
                        (Integer) segmentData.get('DisplaySequence'),
                        (String) segmentData.get('DisplayType'),
                        (Boolean) segmentData.get('IsActive'),
                        (Boolean) segmentData.get('IsAvailableForSelfService'),
                        (String) segmentData.get('MediaType'),
                        (String) segmentData.get('Name'),
                        (String) segmentData.get('ProductId')
                    );
                    if (newSegmentId != null) {
                        oldNewSegId.put((String) segmentData.get('Code'), newSegmentId);
                        nameToSegmentIdMap.put((String) segmentData.get('Name')+suffix , newSegmentId);
                    }
                }
            } catch (Exception e) {
                System.debug('Error inserting/updating segment: ' + e.getMessage());
            }
        }
        
        for (Map<String, Object> segmentData : segmentDataList) {
            try {
                if (segmentData.containsKey('DependentCategorySegmentId') && segmentData.get('DependentCategorySegmentId') != null) {
                    String newId = oldNewSegId.get((String) segmentData.get('Code'));
                    String depId = oldNewSegId.get((String) segmentData.get('DependentCategorySegmentId'));
                    
                    if (newId != null && depId != null) {
                        updateDependencyIdForSegment(newId, depId);
                    }
                }
            } catch (Exception e) {
                System.debug('Error updating dependency for segment: ' + e.getMessage());
            }
        }
        
        System.debug('Inserted/Updated ' + oldNewSegId.size() + ' segments successfully.');
    }
    
    public static void processValues(List<Map<String, Object>> valueDataList) {
        createValues(valueDataList, oldNewSegId);
    }
    
    public static void processValueCSV(String fileName) {
        try {
            ContentVersion cv = [SELECT Id, Title, VersionData FROM ContentVersion WHERE Title = :fileName ORDER BY CreatedDate DESC LIMIT 1];
            String fileContent = cv.VersionData.toString();
            List<Map<String, Object>> valueDataList = parseValueCSV(fileContent);
            processValues(valueDataList);
        } catch (Exception e) {
            System.debug('Error processing Value CSV file: ' + e.getMessage());
        }
    }
    
    private static List<Map<String, Object>> parseValueCSV(String csvContent) {
        List<Map<String, Object>> valueDataList = new List<Map<String, Object>>();
        List<String> lines = csvContent.split('\n');
        
        if (lines.size() < 2) return valueDataList;
        
        
        String delimiter = lines[0].contains(';') ? ';' : ',';
        
        List<String> headers = lines[0].split(delimiter);
        
        for (Integer i = 1; i < lines.size(); i++) {
            List<String> values = lines[i].split(delimiter);
            
            
            while (values.size() < headers.size()) {
                values.add('');
            }
            
            if (values.size() != headers.size()) {
                System.debug('Skipping row ' + i + ' due to mismatched columns. Headers: ' + headers.size() + ', Values: ' + values.size());
                continue;
            }
            
            Map<String, Object> valueData = new Map<String, Object>();
            for (Integer j = 0; j < headers.size(); j++) {
                valueData.put(headers[j].trim(), values[j].trim());
            }
            
            valueDataList.add(valueData);
        }
        
        return valueDataList;
    }
    
    public static void createValues(List<Map<String, Object>> valueDataList, Map<String, String> oldNewSegId) {
        Map<String, String> serverIdentifierAndValueIdMap = new Map<String, String>();
        List<AdTargetSegmentValue> valuesToInsert = new List<AdTargetSegmentValue>();
        
        for (Map<String, Object> valueData : valueDataList) {
            try {
                String originalSegmentId = (String) valueData.get('SegmentCode');
                String resolvedSegmentId = oldNewSegId.get(originalSegmentId);
                String valueId = (String) valueData.get('Id');
                
                if (originalSegmentId != null && (originalSegmentId.length() == 15 || originalSegmentId.length() == 18)) {
                    updateValue(
                        valueId,
                        (String) valueData.get('Label'),
                        resolvedSegmentId != null ? resolvedSegmentId : originalSegmentId,
                        (String) valueData.get('ServerIdentifier')
                    );
                } else {
                    if (resolvedSegmentId != null) {
                        AdTargetSegmentValue value = populateSegmentValue(
                            (String) valueData.get('Label'),
                            resolvedSegmentId,
                            (String) valueData.get('ServerIdentifier')
                        );
                        valuesToInsert.add(value);
                    }
                }  
                
            } catch (Exception e) {
                System.debug('Error creating value: ' + e.getMessage());
            }
        }        
        try {
            if (!valuesToInsert.isEmpty()) {
                insert valuesToInsert;
                for (AdTargetSegmentValue v : valuesToInsert) {
                    serverIdentifierAndValueIdMap.put(v.ServerIdentifier, v.Id);
                    nameToValueIdMap.put((String) v.Name + suffix, v.Id);
                }
                System.debug('Inserted ' + valuesToInsert.size() + ' values successfully.');
            }
        } catch (Exception e) {
            System.debug('Error during insert/update of values: ' + e.getMessage());
        }
        
        for (Map<String, Object> valueData : valueDataList) {
            try {
                if (valueData.containsKey('RefServerIdentifierId') && valueData.get('RefServerIdentifierId') != null) {
                    String newId = serverIdentifierAndValueIdMap.get((String) valueData.get('ServerIdentifier') + suffix);
                    String refId = serverIdentifierAndValueIdMap.get((String) valueData.get('RefServerIdentifierId') + suffix);
                    
                    if (newId != null && refId != null) {
                        updateDependencyIdForValue(newId, refId);
                    }
                }
            } catch (Exception e) {
                System.debug('Error updating dependency for value: ' + e.getMessage());
            }
        }
        
        System.debug('Processed ' + serverIdentifierAndValueIdMap.size() + ' value dependencies successfully.');
    }
    
    public static void processAvailCSV(String fileName) {
        try {
            ContentVersion cv = [
                SELECT Id, Title, VersionData 
                FROM ContentVersion 
                WHERE Title = :fileName 
                ORDER BY CreatedDate DESC 
                LIMIT 1
            ];
            String fileContent = cv.VersionData.toString();
            List<Map<String, Object>> availDataList = parseAvailCSV(fileContent);
            Boolean buildValue=false;
            for (Map<String, Object> row : availDataList) {
                if (row.size() > 2) {
                    buildValue = true;
                    break;
                }
            }
            
            buildAvailable(availDataList,buildValue);
        } catch (Exception e) {
            System.debug('Error processing Avail CSV file: ' + e.getMessage());
        }
    }
    
    private static List<Map<String, String>> parseAvailCSV(String csvContent) {
        List<Map<String, String>> availDataList = new List<Map<String, String>>();
        List<String> lines = csvContent.split('\n');
        
        if (lines.size() < 2) return availDataList;
        
        String delimiter = lines[0].contains(';') ? ';' : ',';
        List<String> headers = lines[0].split(delimiter);
        
        for (Integer i = 1; i < lines.size(); i++) {
            List<String> values = lines[i].split(delimiter);
            
            while (values.size() < headers.size()) {
                values.add('');
            }
            
            if (values.size() != headers.size()) continue;
            
            Map<String, String> availData = new Map<String, String>();
            for (Integer j = 0; j < headers.size(); j++) {
                availData.put(headers[j].trim(), values[j].trim());
            }
            
            availDataList.add(availData);
        }
        
        return availDataList;
    }
    
    
    
    public static void buildAvailable(List<Map<String, Object>> availDataList,Boolean buildValue) {
        for (Map<String, Object> availData : availDataList) {
            try {
                if(buildValue){
                    InsertAvail((String) availData.get('CategoryCode'), (String) availData.get('SegmentCode'), (String) availData.get('Value'), (String) availData.get('isIncluded')) ;
                }
                else
                {
                    InsertAvail((String) availData.get('CategoryCode'), (String) availData.get('SegmentCode'));
                }
                
            } catch (Exception e) {
                System.debug('Error inserting avail mapping: ' + e.getMessage());
            }
        }
    }
    
    private static String insertCategory(String code, String description, Integer displaySequence, Boolean isActive, Boolean isAvailableForSelfService, String mediaType, String name, String parentAdTargetCategoryId) {
        AdTargetCategory c = new AdTargetCategory();
        c.Code = code + suffix;
        c.Description = description;
        c.DisplaySequence = displaySequence;
        c.IsActive = isActive;
        c.IsAvailableForSelfService = isAvailableForSelfService;
        c.MediaType = mediaType;
        c.Name = name;
        c.ParentAdTargetCategoryId = (parentAdTargetCategoryId == '' || parentAdTargetCategoryId == 'null') ? null : parentAdTargetCategoryId;
        insert c;
        return c.Id;
    }
    
    private static String insertSegment(String code, String dataType, String description, Integer displaySequence, String displayType, Boolean isActive, Boolean isAvailableForSelfService, String mediaType, String name, String productId) {
        AdTargetCategorySegment s = new AdTargetCategorySegment();
        s.Code = code + suffix;
        s.DataType = dataType;
        s.Description = description;
        s.DisplaySequence = displaySequence;
        s.DisplayType = displayType;
        s.IsActive = isActive;
        s.IsAvailableForSelfService = isAvailableForSelfService;
        s.MediaType = mediaType;
        s.Name = name;
        s.ProductId = (productId == '' || productId == 'null') ? null : productId;
        insert s;
        return s.Id;
    }
    
    private static void updateDependencyIdForSegment(String newId, String depId) {
        AdTargetCategorySegment seg = [SELECT Id, DependentCategorySegmentId FROM AdTargetCategorySegment WHERE Id = :newId];
        seg.DependentCategorySegmentId = depId;
        update seg;
    }
    
    private static AdTargetSegmentValue populateSegmentValue(String label, String newSegId, String serverIdentifier) {
        AdTargetSegmentValue v = new AdTargetSegmentValue();
        v.Name = label;
        v.TargetCategorySegmentId = newSegId;
        v.ServerIdentifier = serverIdentifier + suffix;
        return v;
    }
    
    private static void updateDependencyIdForValue(String newId, String depId) {
        AdTargetSegmentValue val = [SELECT Id, DependantSegmentValueId FROM AdTargetSegmentValue WHERE Id = :newId];
        val.DependantSegmentValueId = depId;
        update val;
    }
    
    private static void InsertAvail(String categoryCode, String segmentCode) {
        AdAvailableTargetSegment a = new AdAvailableTargetSegment();
        categoryCode += suffix;
        segmentCode += suffix;
        AdTargetCategory c = [SELECT Id FROM AdTargetCategory WHERE code = :categoryCode];
        AdTargetCategorySegment s = [SELECT Id FROM AdTargetCategorySegment WHERE code = :segmentCode];
        a.AdTargetCategoryId = c.Id;
        a.AdTargetCategorySegmentId = s.Id;
        insert a;
        
        List<AdTargetSegmentValue> vs = [SELECT Id FROM AdTargetSegmentValue WHERE TargetCategorySegmentId = :s.Id];
        List<AdAvailableTargetSegmentValue> avs = new List<AdAvailableTargetSegmentValue>();
        for (AdTargetSegmentValue v : vs) {
            AdAvailableTargetSegmentValue av = new AdAvailableTargetSegmentValue();
            av.AdAvailableTargetSegmentId = a.Id;
            av.AdTargetSegmentValueId = v.Id;
            av.IsIncluded = true;
            avs.add(av);
        }
        insert avs;
    }
    
    
    private static void InsertAvail(String categoryCode, String segmentCode, String value, String isIncluded) {
        AdAvailableTargetSegment a = new AdAvailableTargetSegment();
        categoryCode += suffix;
        segmentCode += suffix;
        value+=suffix;
        AdTargetCategory c = [SELECT Id FROM AdTargetCategory WHERE code = :categoryCode];
        AdTargetCategorySegment s = [SELECT Id FROM AdTargetCategorySegment WHERE code = :segmentCode];
        a.AdTargetCategoryId = c.Id;
        a.AdTargetCategorySegmentId = s.Id;
        insert a;
        AdTargetSegmentValue vs = [SELECT Id FROM AdTargetSegmentValue WHERE TargetCategorySegmentId = :s.Id AND Name =:value];
        AdAvailableTargetSegmentValue av = new AdAvailableTargetSegmentValue();
        av.AdAvailableTargetSegmentId = a.Id;
        av.AdTargetSegmentValueId = vs.Id;
        av.IsIncluded = (isIncluded != null && isIncluded.toLowerCase() == 'true') ? true : false;
        insert av;
    }
    
    private static void updateSegment(String segmentId, String code, String dataType, String description, Integer displaySequence, String displayType, Boolean isActive, Boolean isAvailableForSelfService, String mediaType, String name, String productId) {
        AdTargetCategorySegment s = [SELECT Id, Code, DataType, Description, DisplaySequence, DisplayType, IsActive, IsAvailableForSelfService, MediaType, Name, ProductId FROM AdTargetCategorySegment WHERE Id = :segmentId];
        s.Code = code + suffix;
        s.DataType = dataType;
        s.Description = description;
        s.DisplaySequence = displaySequence;
        s.DisplayType = displayType;
        s.IsActive = isActive;
        s.IsAvailableForSelfService = isAvailableForSelfService;
        s.MediaType = mediaType;
        s.Name = name;
        s.ProductId = (productId == '' || productId == 'null') ? null : productId;
        update s;
    }
    
    private static void updateValue(String valueId, String label, String segmentId, String serverIdentifier) {
        AdTargetSegmentValue existingValue = [SELECT Id, Name,TargetCategorySegmentId , ServerIdentifier FROM AdTargetSegmentValue WHERE Id = :valueId];
        existingValue.Name = label + suffix;
        existingValue.TargetCategorySegmentId = segmentId;
        existingValue.ServerIdentifier = serverIdentifier;
        update existingValue;
    }
    
    public static void processProductTargetingCSV(String fileName) {
        try {
               ContentVersion cv = [SELECT Id, Title, VersionData FROM ContentVersion WHERE Title = :fileName ORDER BY CreatedDate DESC LIMIT 1];
            String fileContent = cv.VersionData.toString();
            List<Map<String, String>> rawData = parseProductTargetingCSV(fileContent);
            
            
            // Step 1: Gather unique product names
            Set<String> productNames = new Set<String>();
            for (Map<String, String> row : rawData) {
                productNames.add(row.get('ProductName'));
            }
            
            // Step 2: Query Product2 to get Ids
            Map<String, Id> productNameToIdMap = new Map<String, Id>();
            for (Product2 p : [SELECT Id, Name FROM Product2 WHERE Name IN :productNames]) {
                productNameToIdMap.put(p.Name, p.Id);
            }
            
            // Step 3: Final resolved data with all IDs
            List<Map<String, Id>> resolvedRecords = new List<Map<String, Id>>();
            for (Map<String, String> row : rawData) {
                String prodName = row.get('ProductName');
                String catName = row.get('TargetCategoryName');
                String segName = row.get('TargetSegmentName');
                String valName = row.get('TargetSegmentValueName');
                
                if (productNameToIdMap.containsKey(prodName)
                    && nameToCategoryIdMap.containsKey(catName)
                    && nameToSegmentIdMap.containsKey(segName)
                    && nameToValueIdMap.containsKey(valName)) {
                        
                        Map<String, Id> resolved = new Map<String, Id>{
                            'ProductId' => productNameToIdMap.get(prodName),
                                'TargetCategoryId' => nameToCategoryIdMap.get(catName),
                                'TargetSegmentId' => nameToSegmentIdMap.get(segName),
                                'TargetSegmentValueId' => nameToValueIdMap.get(valName)
                                };
                                    resolvedRecords.add(resolved);
                    } else {
                        System.debug('Missing mapping for row: ' + row);
                    }
            }
            
            System.debug('Resolved product-targeting rows: ' + resolvedRecords.size());
                        
            List<AdProductDefaultTargetValue> defaultTargetValueRecords = new List<AdProductDefaultTargetValue>();
            for (Map<String, Id> record : resolvedRecords) {
                defaultTargetValueRecords.add(new AdProductDefaultTargetValue(
                    AdTargetCategoryId = record.get('TargetCategoryId'),
                    TargetCategorySegmentId = record.get('TargetSegmentId'),
                    TargetCategorySegmentValueId = record.get('TargetSegmentValueId'),
                    ProductId = record.get('ProductId')
                    
                ));
            }
            insert defaultTargetValueRecords;
            
            System.debug('Inserted ' + defaultTargetValueRecords.size() + ' AdProductDefaultTargetValue records.');
            
            
        } catch (Exception e) {
            System.debug('Error processing CSV file: ' + e.getMessage());
        }
    }
    
    
    private static List<Map<String, String>> parseProductTargetingCSV(String csvContent) {
        List<Map<String, String>> parsedList = new List<Map<String, String>>();
        List<String> lines = csvContent.split('\n');
        if (lines.size() < 2) return parsedList;
        
        String delimiter = lines[0].contains(';') ? ';' : ',';
        List<String> headers = lines[0].split(delimiter);
        
        for (Integer i = 1; i < lines.size(); i++) {
            List<String> values = lines[i].split(delimiter);
            while (values.size() < headers.size()) values.add('');
            
            if (values.size() != headers.size()) {
                System.debug('Skipping row ' + i + ' due to mismatched columns.');
                continue;
            }
            
            Map<String, String> row = new Map<String, String>();
            for (Integer j = 0; j < headers.size(); j++) {
                row.put(headers[j].trim(), values[j].trim());
            }
            parsedList.add(row);
        }   
        return parsedList;
    }
}
```

### Upload the below listed CSVs as Files in the org.  You can add/remove the rows of these CSVs.

[AdTargetCategory.csv](../csv-files/AdTargetCategory.csv)

[AdTargetSegment.csv](../csv-files/AdTargetSegment.csv)

[AdTargetSegmentValues.csv](../csv-files/AdTargetSegmentValues.csv)

[BuildAvailable.csv](../csv-files/BuildAvailable.csv)

[DefaultTargeting.csv](../csv-files/DefaultTargeting.csv)


### In the Developer console / Benchpress, run the below code snippets. Make sure to follow the sequence.

#### To insert Ad Target Category
```
GenerateTargetingSeedData.processCategoryCSV('AdTargetCategory');
```

#### To insert Ad Target Category Segment
```
GenerateTargetingSeedData.processSegmentCSV('AdTargetSegment');
```

#### To insert Ad Target Segment Values
```
GenerateTargetingSeedData.processValueCSV('AdTargetSegmentValues');
```

#### To build Ad Available Target Segment and Ad Available Target Segment Values
```
GenerateTargetingSeedData.processAvailCSV('BuildAvailable');
```

#### To insert Default Targeting for Products 
```
GenerateTargetingSeedData.processProductTargetingCSV('DefaultTargeting');
```
### Demo Recording

[▶️ Click here to view the demo video](../recordings/recording-2.mov)

