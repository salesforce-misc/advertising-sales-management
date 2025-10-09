# Script for Ad Space Capacity Batch Class

```
public class AdSpaceCapacityBatch implements Database.Batchable<SObject>, Database.Stateful {
public Integer dailyCount;
public Integer weeklyCount;
public Integer monthlyCount;
private List<Id> adSpaceIds;
private List<Id> insertedCapacityIds;

    /**
     * @description Constructor for the AdSpaceCapacityBatch.
     * @param dailyRecCount The number of daily capacity records to create per AdSpace.
     * @param weeklyRecCount The number of weekly capacity records to create per AdSpace.
     * @param monthlyRecCount The number of monthly capacity records to create per AdSpace.
     * @param adSpaceIdsToProcess A list of AdSpace IDs that this batch should process.
     */
    public AdSpaceCapacityBatch(Integer dailyRecCount, Integer weeklyRecCount, Integer monthlyRecCount,
                                List<Id> adSpaceIdsToProcess) {
        this.dailyCount = dailyRecCount;
        this.weeklyCount = weeklyRecCount;
        this.monthlyCount = monthlyRecCount;
        this.adSpaceIds = adSpaceIdsToProcess; 
        this.insertedCapacityIds = new List<Id>();
    }

    /**
     * @description The start method collects the records or objects to be passed to the execute method.
     * @param BC The BatchableContext for the current batch job.
     * @return A Database.QueryLocator or an Iterable of SObjects.
     */
    public Database.QueryLocator start(Database.BatchableContext BC) {
        System.debug('AdSpaceCapacityBatch Start method executing. Querying AdSpace records using passed IDs.');
        if (adSpaceIds == null || adSpaceIds.isEmpty()) {
            System.debug('WARNING: No AdSpace IDs found in batch instance. Batch will process no records.');
            //Idiomatic way to gracefully handle the return of the start method.
            return Database.getQueryLocator('SELECT Id FROM AdSpace WHERE Id = null'); 
        }
        return Database.getQueryLocator([SELECT Id, Name FROM AdSpace WHERE Id IN :adSpaceIds]);
    }

    /**
     * @description The execute method processes each chunk of records.
     * @param BC The BatchableContext for the current batch job.
     * @param scope A list of AdSpace records returned by the start method.
     */
    public void execute(Database.BatchableContext BC, List<AdSpace> scope) {
        System.debug('AdSpaceCapacityBatch Execute method executing. Processing ' + scope.size() + ' AdSpace records in this chunk.');
        List<AdSpaceCapacity> adSpaceCapacitiesToInsert = new List<AdSpaceCapacity>();
        List<String> ineligibleIndustries = new List<String>{'Insurance', 'Healthcare', 'Education', 'Agriculture', 'Banking'}; 
            
        for (AdSpace adSpace : scope) {
            // Daily records with spanning
            Date currentDate = Date.today();
            for (Integer i = 0; i < dailyCount; i++) {
                Integer bookedCount = 0, reservedCount = 0, pitchedCount = 0;
                Integer availableCapacity = 0, totalCapacity = 0;
                Integer span;
        
                Double scenarioPicker = Math.random();
        
                if (scenarioPicker < 0.2) { // 20% chance for a single, zero-value cell
                    i++;  // Just move to next cell since this will be an N/A cell
                    continue;
                } else { // 80% chance for a cell with capacity that can span
                    // Determine a random span from 1 to 3 days
                    span = Math.mod(Math.abs(Crypto.getRandomInteger()), 3) + 1;
                    
                    // Generate random capacity
                    availableCapacity = (Math.mod(Math.abs(Crypto.getRandomInteger()), 100) + 1) * 100;
                    totalCapacity = availableCapacity;
                }
        
                // Check if the span would go beyond the total number of records to create
                if (i + span > dailyCount) {
                    span = dailyCount - i;
                }
                
                Date startDate = currentDate.addDays(i);
                Date endDate = startDate.addDays(span - 1); // Calculate EndDate based on span
        
                AdSpaceCapacity newCapacity = new AdSpaceCapacity(
                    AdSpaceId = adSpace.Id,
                    StartDate = startDate,
                    EndDate = endDate,
                    CapacityDurationType = 'Daily',
                    IsActive = TRUE,
                    TotalCapacity = totalCapacity,
                    BookedCount = bookedCount,
                    ReservedCount = reservedCount,
                    PitchedCount = pitchedCount,
                    AvailableCapacity = availableCapacity
                );
                adSpaceCapacitiesToInsert.add(newCapacity);
                
                // IMPORTANT: Advance the loop counter by the span to avoid creating overlapping records
                i += span - 1;
            }
        
            // Weekly records with corrected Monday calculation
            Datetime nowGmt = System.now();
            TimeZone userTz = UserInfo.getTimeZone();
            String userTimezoneId = userTz.getID();
            String dateStringInUserTz = nowGmt.format('yyyy-MM-dd', userTimezoneId);
            Date today = Date.valueOf(dateStringInUserTz);
            
            Date weekStartDate = today.toStartofWeek(); // Sunday 28
            Integer dayOfWeek = weekStartDate.daysBetween(today); // 3
            Integer daysToSubtract = Math.mod(dayOfWeek + 6, 7);
            Date mondayOfCurrentWeek = today.addDays(-daysToSubtract); // 29

            for (Integer i = 0; i < weeklyCount; i++) {
                Integer bookedCount = 0, reservedCount = 0, pitchedCount = 0;
                Integer availableCapacity = 0, totalCapacity = 0;
                Integer span;
            
                Double scenarioPicker = Math.random();
            
                if (scenarioPicker < 0.2) { // 20% chance for a single, zero-value cell
                    i++; // Just move to next cell since this will be an N/A cell
                    continue;
                } else { // 80% chance for a cell with capacity that can span
                    // Determine a random span from 1 to 2 weeks
                    span = Math.mod(Math.abs(Crypto.getRandomInteger()), 2) + 1;
            
                    availableCapacity = (Math.mod(Math.abs(Crypto.getRandomInteger()), 100) + 1) * 100;
                    totalCapacity = availableCapacity;
                }
                
                if (i + span > weeklyCount) {
                    span = weeklyCount - i;
                }
            
                Date startDate = mondayOfCurrentWeek.addDays(i * 7);
                Date endDate = startDate.addDays((span * 7) - 1);
            
                AdSpaceCapacity newCapacity = new AdSpaceCapacity(
                    AdSpaceId = adSpace.Id,
                    StartDate = startDate,
                    EndDate = endDate,
                    CapacityDurationType = 'Weekly',
                    IsActive = TRUE,
                    TotalCapacity = totalCapacity,
                    BookedCount = bookedCount,
                    ReservedCount = reservedCount,
                    PitchedCount = pitchedCount,
                    AvailableCapacity = availableCapacity
                );
                adSpaceCapacitiesToInsert.add(newCapacity);
            
                // Advance the loop counter
                i += span - 1;
            }
    
            // Monthly records with spanning
            Date firstDayOfCurrentMonth = Date.today().toStartOfMonth();
            for (Integer i = 0; i < monthlyCount; i++) {
                Integer bookedCount = 0, reservedCount = 0, pitchedCount = 0;
                Integer availableCapacity = 0, totalCapacity = 0;
                Integer span;
        
                Double scenarioPicker = Math.random();
        
                if (scenarioPicker < 0.2) { // 20% chance for a single, zero-value cell
                    i++;  // Just move to next cell since this will be an N/A cell
                    continue;
                } else { // 80% chance for a cell with capacity that can span
                    // Determine a random span from 1 to 3 months
                    span = Math.mod(Math.abs(Crypto.getRandomInteger()), 3) + 1;
        
                    availableCapacity = (Math.mod(Math.abs(Crypto.getRandomInteger()), 100) + 1) * 100;
                    totalCapacity = availableCapacity;
                }
        
                if (i + span > monthlyCount) {
                    span = monthlyCount - i;
                }
                
                Date startDate = firstDayOfCurrentMonth.addMonths(i);
                Date endDate = startDate.addMonths(span).addDays(-1); // End date is the last day of the final month
        
                AdSpaceCapacity newCapacity = new AdSpaceCapacity(
                    AdSpaceId = adSpace.Id,
                    StartDate = startDate,
                    EndDate = endDate,
                    CapacityDurationType = 'Monthly',
                    IsActive = TRUE,
                    TotalCapacity = totalCapacity,
                    BookedCount = bookedCount,
                    ReservedCount = reservedCount,
                    PitchedCount = pitchedCount,
                    AvailableCapacity = availableCapacity
                );
                adSpaceCapacitiesToInsert.add(newCapacity);
                
                // Advance the loop counter
                i += span - 1;
            }
        }

        if (!adSpaceCapacitiesToInsert.isEmpty()) {
            try {
                insert adSpaceCapacitiesToInsert;
                System.debug('Inserted ' + adSpaceCapacitiesToInsert.size() + ' AdSpaceCapacity records in this batch chunk.');
                for (AdSpaceCapacity ac : adSpaceCapacitiesToInsert) {
                    this.insertedCapacityIds.add(ac.Id);
                }
            } catch (DMLException e) {
                System.debug('Error inserting AdSpaceCapacity records in batch: ' + e.getMessage());
            }
        } else {
            System.debug('No AdSpaceCapacity records to insert in this batch chunk.');
        }
    }

    /**
     * @description The finish method executes once all batches are processed.
     * It triggers the AdSpaceCapacityAllocationBatch if capacity records were inserted.
     * @param BC The BatchableContext for the current batch job.
     */
    public void finish(Database.BatchableContext BC) {
        System.debug('AdSpaceCapacityBatch Finish method executing.');
        AsyncApexJob job = [SELECT Id, Status, NumberOfErrors, JobItemsProcessed, TotalJobItems, CreatedBy.Email
                            FROM AsyncApexJob WHERE Id = :BC.getJobId()];
        System.debug('Batch job ' + job.Id + ' finished with status: ' + job.Status + '. ' +
                     job.JobItemsProcessed + ' items processed with ' + job.NumberOfErrors + ' errors.');
    }
}
```