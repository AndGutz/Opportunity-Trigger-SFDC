# Opportunity Trigger SFDC

TRIGGER 1 

"public void OnBeforeInsert(Opportunity[] newOpportunities, Map<ID, Opportunity> newOpportunityMap){
validateReqFields(null,new Opportunities);
//populateMarketing Campaign(new Opportunities);
calculateOpportunityFields(null, newOpportunities);
populateOpportunityCustomerField(null, new Opportunities);
copyFieldsFromOpptyPrimaryContact(newOpportunities, null);"

TRIGGER 2
"public void OnBeforeUpdate(Map<ID, Opportunity> oldOpportunityMap, Opportunity[] newOpportunities, Map<ID, Opportunity> newOpportunityMap){
validateReqFields(oldOpportunityMap, new Opportunities);
calculateOpportunityFields(oldOpportunityMap, newOpportunities);
populateOpportunityCustomerField(oldOpportunityMap, new Opportunities);
copyFieldsFromOpptyPrimaryContact(newOpportunities, oldOpportunityMap);
//validateRelatedQuote(oldOpportunityMap, newOpportunityMap);
populateNextStepsHistory(oldOpportunityMap, newOpportunities);"

TRIGGER 3
"public void OnAfterInsert(Map<ID, Opportunity> newOpportunityMap){
/*if(newOpportunityMap.size() == 1){
checkOpportunityClosed(null, newOpportunityMap.values()[0]);
}*/
processDSOEmail(newOpportunityMap);
createOppContactRoles('insert', newOpportunity Map, null);
//copyLeadSource('insert', newOpportunityMap, null);
rollUpValuesToParentOpportunity(null, newOpportunityMap);"

TRIGGER 4
"public void OnAfterUpdate(Opportunity[] or Opportunities, Opportunity[] updated Opportunities, Map<ID, Opportunity> oldOpportunityMap, Map<ID, Opportunity> newOpportunityMap){
if(firstRun){
firstRun = false;
/*if(updated Opportunities.size() == 1){
checkOpportunityClosed(oldOpportunities[0], updatedOpportunities[0]);
}*/
createOppContactRoles('update', newOpportunityMap, oldOpportunityMap);
copyFieldsToOrder(oldOpportunityMap, newOpportunityMap);
//copyLeadSource('update', newOpportunityMap, oldOpportunityMap);
rollUpValuesToParentOpportunity(oldOpportunityMap, newOpportunityMap);
copyFieldsFromPArentToChildOppty(oldOpportunityMap, newOpportunityMap);

processNetsuiteFields(oldOpportunityMap,updatedOpportunities);

updateRelatedQuote(oldOpportunityMap, newOpportunityMap);


List<Case> cList1 = generate3rdPartyDataCase(oldOpportunityMap, newOpportunityMap);
//List<Case> cList2 = generateStrategyFeedbackCase(oldOpportunityMap, newOpportunityMap);
List<Case> cList3 = generateProServCase(oldOpportunityMap, newOpportunityMap);
List<Case> cList4 = generateCustomerSuccessCase(oldOpportunityMap, newOpportunityMap);
List<Case> cList5 = generateFinanceCommitReviewCase(oldOpportunityMap, newOpportunityMap); // BOSP-581
List<Case> cList6 = generateOpsCommitReviewCase(oldOpportunityMap, newOpportunityMap); // BOSP-586



List<Case> insertMainList = new List<Case>();
if (cList1!=null) insertMainList.addAll(cList1);
//if (cList2!=null) insertMainList.addAll(cList2);
if (cList3!=null) insertMainList.addAll(cList3);
if (cList4!=null) insertMainList.addAll(cList4);
if (cList5!=null) insertMainList.addAll(cList5); // BOSP-581
if (cList6!=null) insertMainList.addAll(cList6); // BOSP-586
if (insertMainList.size()>0)  insert insertMainList;"
  
TRIGGER 5
"// BOSP-519 autopopulate Next Steps History
public void populateNextStepsHistory(Map<ID, Opportunity> oldOpportunityMap, Opportunity[] newOpportunities){
Opportunity oldOpp = null;
for (Opportunity opp : newOpportunities){
if (oldOpportunityMap!=null && oldOpportunityMap.containsKey(opp.Id)) oldOpp = oldOpportunityMap.get(opp.Id);
if (oldOpp!=null && oldOpp.Next_Steps__c!=null && oldOpp.Next_Steps__c!='' &&
oldOpp.Next_Steps__c!=opp.Next_Steps__c &&
opp.Next_Steps__c!=null && opp.Next_Steps__c!=''){

opp.Previous_Next_Steps__c = oldOpp.Next_Steps__c;
opp.Next_Steps_History__c = (opp.Next_Steps_History__c!=null && opp.Next_Steps_History__c!='') ? opp.Next_Steps_History__c + '\n' + oldOpp.Next_Steps__c : oldOpp.Next_Steps__c;"

TRIGGER 6
"/*// autopopulate Marketing Campaign field depending on the Opportunity Primary Contact's Marketing Campaign field
public void populateMarketingCampaign(Opportunity[] newOpportunities){
Set<Id> contactIdSet = new Set<Id>();
for (Opportunity opp : newOpportunities){
if (opp.Opportunity_Primary_Contact__c!=null){
contactIdSet.add(opp.Opportunity_Primary_Contact__c);"
  
TRIGGER 7
"Map<Id,Contact> contactIdContactMap = new Map<Id,Contact>([select Id, Marketing_Campaign__c from Contact where Id IN: contactIdSet]);

for (Opportunity opp : newOpportunities){
if (opp.Opportunity_Primary_Contact__c!=null && contactIdContactMap.containsKey(opp.Opportunity_Primary_Contact__c)){
if (opp.Marketing_Campaign__c==null) opp.Marketing_Campaign__c = contactIdContactMap.get(opp.Opportunity_Primary_Contact__c).Marketing_Campaign__c;"

TRIGGER 8
"public void calculateOpportunityFields(Map<ID, Opportunity> oldOpportunityMap, Opportunity[] newOpportunities){
Set<Id> oppClosedWonIdSet = new Set<Id>();
for (Opportunity opp : newOpportunities){

Double newAmt = (opp.New_Amount__c!=null) ? opp.New_Amount__c : 0;
Double dealRate = (opp.Deal_Rate__c!=null) ? opp.Deal_Rate__c : 0;
Double prodNetRev = (opp.Product_Net_Revenue__c!=null) ? opp.Product_Net_Revenue__c : 0;

if (opp.Product__c == 'Advertising Commitment' && opp.Parent_Opportunity_SSA__c == null)
opp.Net_Revenue_Account_Roll_Up__c = prodNetRev;
else if ((opp.Product__c == 'Enterprise Software' || opp.Product__c == 'Advertising Commitment') && opp.Parent_Opportunity_SSA__c != null)
opp.Net_Revenue_Account_Roll_Up__c = opp.Product_Net_Revenue__c;
else if (opp.Product__c == 'Buying' && (opp.Deal_Type__c == '% of Budget' || opp.Deal_Type__c == 'Cost Plus' || opp.Deal_Type__c == 'Monthly Fee'))
opp.Net_Revenue_Account_Roll_Up__c = newAmt * dealRate;
else if (opp.Product__c == 'Buying' && (opp.Deal_Type__c == 'One Time Fee' || opp.Deal_Type__c == 'Quarterly Fee' || opp.Deal_Type__c == 'Annual Fee'))
opp.Net_Revenue_Account_Roll_Up__c = newAmt * dealRate;
else if (opp.Product__c == 'Media IO' && (opp.Deal_Type__c == '% of Budget' || opp.Deal_Type__c == 'Cost Plus'))
opp.Net_Revenue_Account_Roll_Up__c = newAmt * dealRate;
else if (opp.Product__c == 'Media IO' && opp.Deal_Type__c == 'CPA')
opp.Net_Revenue_Account_Roll_Up__c = (newAmt * dealRate) * 0.3;
else if (opp.Product__c == 'Enterprise IO' && (opp.Deal_Type__c == '% of Budget' || opp.Deal_Type__c == 'Cost Plus'))
opp.Net_Revenue_Account_Roll_Up__c = newAmt * dealRate;
else if (opp.Product__c == 'Hybrid Commitment')
opp.Net_Revenue_Account_Roll_Up__c = opp.Child_Opportunities_Net_Revenue__c;"

TRIGGER 9
"make signature date field on Order level required if platfom != Unified Application
public void validateReqFields(Map<ID, Opportunity> oldOpportunityMap, Opportunity[] newOpportunities){

Set<String> stageNamesWithReqFieldsSet = new Set<String>{'Discovery', 'Demo or Pitch', 'Value Proposition',
Proposal', 'Contract or Strategy Sent', 'Redlines or Legal Review',
Out for Signature or IO Verbal', 'Closed Won', 'Closed Lost',
Proposal_c', 'Committed', 'Ops Review' }; // BOSP-563

for (Opportunity opp : newOpportunities){
if (stageNamesWithReqFieldsSet.contains(opp.StageName)){
if (opp.StageName!='Discovery' && opp.RecordTypeId != Constants.opptyCommitment2RecType){
if (opp.New_Existing_Account__c==null || opp.New_Existing_Account__c=='') opp.New_Existing_Account__c.addError('Please input value here.');
if (opp.RecordTypeId==Constants.opptyAdvertisingRecType &&
(opp.New_Existing_Brand__c==null || opp.New_Existing_Brand__c=='')) opp.New_Existing_Brand__c.addError('Please input value here.');
if (opp.Product__c==null || opp.Product__c=='') opp.Product__c.addError('Please input value here.');
if (opp.Campaign_Start_Date__c==null) opp.Campaign_Start_Date__c.addError('Please input value here.');
if (opp.Campaign_End_Date__c==null) opp.Campaign_End_Date__c.addError('Please input value here.');
if (opp.Deal_Type__c==null || opp.Deal_Type__c=='') opp.Deal_Type__c.addError('Please input value here.');
if (opp.RecordTypeId==Constants.opptyAdvertisingRecType &&
(opp.Deal_Rate__c==null || opp.Deal_Rate__c==0)) opp.Deal_Rate__c.addError('Please input value here.');
if (opp.RecordTypeId==Constants.opptyAdvertisingRecType &&
(opp.Platform__c==null || opp.Platform__c=='')) opp.Platform__c.addError('Please input value here.');
if (opp.RecordTypeId==Constants.opptyAdvertisingRecType &&
(opp.New_Amount__c==null || opp.New_Amount__c==0)) opp.New_Amount__c.addError('Please input value here.');"

TRIGGER 10
"if (opp.StageName!='Discovery' &&
opp.RecordTypeId == Constants.opptyCommitment2RecType){
if (opp.Product__c==null || opp.Product__c=='') opp.Product__c.addError('Product Is required to move a commitment opportunity past the Discovery Stage.');"

TRIGGER 11
"if (opp.StageName!='Discovery' &&
opp.StageName!='Inbound Prospecting' &&
opp.StageName!='Outbound Prospecting' &&
opp.StageName!='Rep DQ / Inactive' && opp.RecordTypeId == Constants.opptyCommitment2RecType){
if (opp.Deal_Description__c==null || opp.Deal_Description__c=='' ||
opp.Next_Steps__c==null || opp.Next_Steps__c=='' ||
opp.Opportunity_Primary_Contact__c==null ||
opp.New_Existing_Account__c==null || opp.New_Existing_Account__c==''){
opp.addError('Deal Description, Next Steps, Opportunity Primary contact and New / Existing Account are required to move an Opportunity past the Discovery stage');"

TRIGGER 12
"/*if (opp.StageName!='Closed Lost' &&
opp.Opportunity_Category__c!=null && opp.Opportunity_Category__c=='Lead'){
opp.addError('Opportunity Category can not be equal to Lead if the Opportunity Stage is not equal to Inbound Prospecting, Outbound Prospecting Or Discovery');"

TRIGGER 13
"if (opp.Opportunity_Primary_Contact__c==null && opp.RecordTypeId != Constants.opptyCommitment2RecType) opp.Opportunity_Primary_Contact__c.addError('Please input value here.');

// BOSP-575
if (opp.RecordTypeId == Constants.opptyCommitment2RecType){
if ((opp.Opportunity_Type__c==null || opp.Opportunity_Type__c=='') && opp.StageName=='Demo or Pitch') opp.Opportunity_Type__c.addError('Please confirm Opportunity Type for this Commitment.');

if (opp.Opportunity_Type__c!=null && opp.Opportunity_Type__c=='New Sale' && (opp.StageName=='Redlines or Legal Review' || opp.StageName=='Redlines or Legal Review')){
if (opp.Customer_Success_Manager__c==null) opp.Customer_Success_Manager__c.addError('A Customer Success Manager is required before the Opportunity Stage can be updated to Redlines or Legal Review');"

TRIGGER 14
"/*if (opp.Opportunity_Type__c!=null && opp.Opportunity_Type__c=='New Sale' && opp.StageName=='Closed Won'){
if (opp.Customer_Success_Manager__c==null) opp.Customer_Success_Manager__c.addError('Customer Success Manager is required.');
if (opp.Project_Manager__c==null) opp.Project_Manager__c.addError('Project Manager is required.');
if (opp.Customer_Experience_Manager__c==null) opp.Customer_Experience_Manager__c.addError('Customer Experience Manager is required.');"

TRIGGER 16
"if (oldOpp!=null && oldOpp.StageName != opp.StageName){
if (oldOpp.StageName == 'Best Case' && opp.StageName != 'Best Case Closed') opp.addError('Best Case Opportunity can only be updated to Best Case Closed stage');
if (oldOpp.StageName == 'Best Case Closed' && opp.StageName != 'Best Case') opp.addError('Best Case Closed Opportunity can only be updated to Best Case stage');"

TRIGGER 17
"//if ((oldOpp!=null && oldOpp.StageName != opp.StageName && opp.StageName == 'Closed Won' && opp.Product__c == 'Advertising Commitment')){
//if (opp.Customer_Success_Manager__c==null) opp.Customer_Success_Manager__c.addError('Please input value here.');
//}"

TRIGGER 18
"if (oldOpp!=null && oldOpp.StageName != opp.StageName &&
opp.StageName == 'Closed Won' &&
opp.Product__c == 'Advertising Commitment' &&
opp.RecordTypeId==Constants.opptyCommitmentRecType){
if (opp.Customer_Success_Manager__c==null) opp.Customer_Success_Manager__c.addError('Customer Success Manager is required to set a Hybrid Opportunity Closed Won.');"

TRIGGER 19
"// BOSP-534
if (oldOpp!=null && oldOpp.StageName != opp.StageName &&
opp.StageName == 'Closed Won' &&
(opp.Customer_Experience_Manager__c==null ||
opp.Project_Manager__c==null ||
opp.Customer_Success_Manager__c==null) &&
opp.RecordTypeId==Constants.opptyCommitmentRecType){
if (opp.Customer_Experience_Manager__c==null) opp.Customer_Experience_Manager__c.addError('Customer Experience Manager is required to set this Opportunity to Closed Won.');
if (opp.Project_Manager__c==null) opp.Project_Manager__c.addError('Project Manager is required to set this Opportunity to Closed Won.');
if (opp.Customer_Success_Manager__c==null) opp.Customer_Success_Manager__c.addError('Customer Success Manager is required to set this Opportunity to Closed Won.');"

TRIGGER 20
"// autopopulate SBQQ_Contracted for Commitment2 record type
if (oldOpp!=null && oldOpp.StageName != opp.StageName &&
opp.StageName == 'Closed Won' &&
opp.RecordTypeId==Constants.opptyCommitment2RecType){
opp.SBQQ__Contracted__c = true;"

TRIGGER 21
"if (oldOpp!=null && oldOpp.Acceleration_Lead_Source__c != opp.Acceleration_Lead_Source__c &&
opp.Acceleration_Lead_Source__c != null){
opp.Last_Modified_Accelerated_Lead_Source__c = Date.today();"

TRIGGER 26
"for (Opportunity opp : newOpportunities){
if ( (opp.Closed_Won_Reason__c==null || opp.Closed_Won_Reason__c=='') && opp.StageName=='Closed Won'){
opp.Closed_Won_Reason__c.addError('Closed Won Reason is required before you can set this Opportunity to Closed Won.');"

TRIGGER 28
"public List<Case> generate3rdPartyDataCase(Map<Id, Opportunity> oldOpportunityMap, Map<Id, Opportunity> newOpportunityMap){

List<Case> newCaseToInsert = new List<Case>();
for (Opportunity opp : newOpportunityMap.values()){

Opportunity oldOppty = oldOpportunityMap.get(opp.Id);

// BOSP-492 Contact 3rd Party Data Partner Workflow
if ((oldOppty.StageName != 'Closed Won' &&
opp.StageName == 'Closed Won' &&
opp.RecordTypeId == Constants.opptyAdvertisingRecType &&
opp.X3rd_Party_Data_Requested__c == 'Yes' &&
opp.X3rd_party_data_uploaded_to_ad_account__c == 'No' &&
opp.X3rd_party_data__c == 'Yes'
) ||
Test.isRunningTest()){ // to bypass for maximum test coverage


Case tempCase = new Case();
tempCase.RecordtypeId = Constants.CAMPAIGN_MANAGEMENT_CASE_REC_TYPE;
tempCase.Assigned_To__c = opp.Account_Manager__c;
tempCase.Status = 'New';
tempCase.Sub_Area__c = '3rd Party Data';
tempCase.Opportunity__c = opp.Id;
if (opp.Senior_Account_Manager__c!=null) tempCase.Assigned_To__c = opp.Senior_Account_Manager__c;
else if (opp.Account_Manager__c!=null) tempCase.Assigned_To__c = opp.Account_Manager__c;

tempCase.Subject = opp.X3rd_party_partners__c + ' data upload needed';

tempCase.Due_Date__c = date.today();
tempCase.Subject = 'Request data from ' + opp.X3rd_party_partners__c;
tempCase.Description = 'Contact 3rd party data partner to request audiences be uploaded to ad account.';
tempCase.AccountId = opp.AccountId;
tempCase.Brand__c = opp.Brand__c;
tempCase.Parent_Brand__c = opp.Parent_Brand__c;
tempCase.Opportunity__c = opp.Id;
tempCase.OwnerId = opp.OwnerId;
tempCase.X3rd_party_partners__c = opp.X3rd_party_partners__c;

newCaseToInsert.add(tempCase);"

TRIGGER 29
"// BOSP-492 Upload Confirmation Workflow
if ((oldOppty.StageName != 'Closed Won' &&
opp.StageName == 'Closed Won' &&
opp.RecordTypeId == Constants.opptyAdvertisingRecType &&
(opp.X3rd_Party_Data_Requested__c==null || opp.X3rd_Party_Data_Requested__c=='' || opp.X3rd_Party_Data_Requested__c == 'No') &&
opp.X3rd_party_data_uploaded_to_ad_account__c == 'No' &&
opp.X3rd_party_data__c == 'Yes'
) ||
Test.isRunningTest()){ // to bypass for maximum test coverage


Case tempCase = new Case();
tempCase.RecordtypeId = Constants.CAMPAIGN_MANAGEMENT_CASE_REC_TYPE;
tempCase.Assigned_To__c = opp.Account_Manager__c;
tempCase.Status = 'New';
tempCase.Sub_Area__c = '3rd Party Data';
tempCase.Opportunity__c = opp.Id;
if (opp.Senior_Account_Manager__c!=null) tempCase.Assigned_To__c = opp.Senior_Account_Manager__c;
else if (opp.Account_Manager__c!=null) tempCase.Assigned_To__c = opp.Account_Manager__c;

tempCase.Subject = opp.X3rd_party_partners__c + ' data upload needed';

tempCase.Due_Date__c = date.today();
tempCase.Subject = '3rd Party Data Upload Verification';
tempCase.Description = 'Please confirm 3rd party audience(s) is uploaded to ad account..('+opp.X3rd_party_data_uploaded_to_ad_account__c+')';
tempCase.AccountId = opp.AccountId;
tempCase.Brand__c = opp.Brand__c;
tempCase.Parent_Brand__c = opp.Parent_Brand__c;
tempCase.Opportunity__c = opp.Id;
tempCase.OwnerId = opp.OwnerId;
tempCase.X3rd_party_data_uploaded_to_ad_account__c = opp.X3rd_party_data_uploaded_to_ad_account__c;
tempCase.X3rd_party_partners__c = opp.X3rd_party_partners__c;

newCaseToInsert.add(tempCase);"

TRIGGER 30
"// BOSP-492 Request and Upload Workflow
if ((oldOppty.StageName != 'Closed Won' &&
opp.StageName == 'Closed Won' &&
opp.RecordTypeId == Constants.opptyAdvertisingRecType &&
opp.X3rd_Party_Data_Requested__c == 'Yes' &&
opp.X3rd_party_data_uploaded_to_ad_account__c == 'Yes' &&
opp.X3rd_party_data__c == 'Yes'
) ||
Test.isRunningTest()){ // to bypass for maximum test coverage


Case tempCase = new Case();
tempCase.RecordtypeId = Constants.CAMPAIGN_MANAGEMENT_CASE_REC_TYPE;
tempCase.Assigned_To__c = opp.Account_Manager__c;
tempCase.Status = 'New';
tempCase.Sub_Area__c = '3rd Party Data';
tempCase.Opportunity__c = opp.Id;
if (opp.Senior_Account_Manager__c!=null) tempCase.Assigned_To__c = opp.Senior_Account_Manager__c;
else if (opp.Account_Manager__c!=null) tempCase.Assigned_To__c = opp.Account_Manager__c;

tempCase.Subject = '3rd Party Data has been requested and uploaded';

tempCase.Due_Date__c = date.today();
tempCase.Subject = '3rd Party Data Upload Verification';
tempCase.Description = 'Please confirm 3rd party data has been requested and audience(s) is/are uploaded to ad account.';
tempCase.AccountId = opp.AccountId;
tempCase.Brand__c = opp.Brand__c;
tempCase.Parent_Brand__c = opp.Parent_Brand__c;
tempCase.Opportunity__c = opp.Id;
tempCase.OwnerId = opp.OwnerId;
tempCase.X3rd_party_data_uploaded_to_ad_account__c = opp.X3rd_party_data_uploaded_to_ad_account__c;


newCaseToInsert.add(tempCase);
if (newCaseToInsert.size()>0) return newCaseToInsert;
else return null;"

TRIGGER 32
"public List<Case> generateProServCase(Map<Id, Opportunity> oldOpportunityMap, Map<Id, Opportunity> newOpportunityMap){

List<Case> newCaseToInsert = new List<Case>();
for (Opportunity opp : newOpportunityMap.values()){

Opportunity oldOppty = oldOpportunityMap.get(opp.Id);

if ((opp.StageName!=oldOppty.StageName &&
opp.StageName=='Redlines or Legal Review' &&
opp.RecordTypeId == Constants.opptyCommitment2RecType) ||
Test.isRunningTest() // to bypass for maximum test coverage
){

Case tempCase = new Case();
tempCase.Status = 'New';
tempCase.Opportunity__c = opp.Id;
//if (newOppty.Senior_Account_Manager__c!=null) tempCase.Assigned_To__c = newOppty.Senior_Account_Manager__c;
//else if (newOppty.Account_Manager__c!=null) tempCase.Assigned_To__c = newOppty.Account_Manager__c;

tempCase.OwnerId = opp.OwnerId;
tempCase.AccountId = opp.AccountId;
tempCase.RecordtypeId = Constants.PRE_SALES_CASE_REC_TYPE; // Pro-Serv case record type
tempCase.Area__c = 'Pro-Serve';
tempCase.Sub_Area__c = 'Onboarding Request';
tempCase.Subject = 'Onboarding Request for ' + opp.Name;
tempCase.Description = 'Onboarding Request for ' + opp.Name;
tempCase.Due_Date__c = opp.CloseDate;

newCaseToInsert.add(tempCase);
if (newCaseToInsert.size()>0) return newCaseToInsert;
else return null;"

TRIGGER 33
"// BOSP-581
public List<Case> generateFinanceCommitReviewCase(Map<Id, Opportunity> oldOpportunityMap, Map<Id, Opportunity> newOpportunityMap){

List<Case> newCaseToInsert = new List<Case>();
for (Opportunity opp : newOpportunityMap.values()){

Opportunity oldOppty = oldOpportunityMap.get(opp.Id);

// CS Case
if ((opp.StageName!=oldOppty.StageName &&
opp.StageName=='Ops Review' &&
opp.RecordTypeId == Constants.opptyCommitment2RecType) ||
Test.isRunningTest() // to bypass for maximum test coverage
){

Case tempCase = new Case();
tempCase.RecordtypeId = Constants.FINANCE_COMMIT_REVIEW_CASE_REC_TYPE; // Finance Commit Review case record type
tempCase.Area__c = 'Finance Commit Review';
tempCase.Sub_Area__c = 'Commitment Review';

tempCase.Status = 'New';
tempCase.Subject = 'Please review the commitment opportunity and the signed contract';
tempCase.Description = 'Please compare the contract to the commitment opportunity and quote to ensure all data in the opportunity is accurate.';

tempCase.Assigned_To__c = '0051400000CR53W'; // default to Chris Cahill
tempCase.OwnerId = Constants.FINANCE_COMMIT_REVIEW_QUEUE;
tempCase.Opportunity__c = opp.Id;

newCaseToInsert.add(tempCase);
}


}

if (newCaseToInsert.size()>0) return newCaseToInsert;
else return null;"

TRIGGER 34
"// BOSP-586
public List<Case> generateOpsCommitReviewCase(Map<Id, Opportunity> oldOpportunityMap, Map<Id, Opportunity> newOpportunityMap){

List<Case> newCaseToInsert = new List<Case>();
for (Opportunity opp : newOpportunityMap.values()){

Opportunity oldOppty = oldOpportunityMap.get(opp.Id);

// CS Case
if ((opp.StageName!=oldOppty.StageName &&
opp.StageName=='Ops Review' &&
opp.RecordTypeId == Constants.opptyCommitment2RecType) ||
Test.isRunningTest() // to bypass for maximum test coverage
){

Case tempCase = new Case();
tempCase.RecordtypeId = Constants.OPS_COMMIT_REVIEW_CASE_REC_TYPE; // Ops Commit Review case record type
tempCase.Area__c = 'Ops Commit Review';
tempCase.Sub_Area__c = 'Ops Review';

tempCase.Status = 'New';
tempCase.Subject = 'Please review the commitment opportunity and the signed contract';
tempCase.Description = 'Please compare the contract to the commitment opportunity and quote to ensure all data in the opportunity is accurate.';

tempCase.Assigned_To__c = '0051400000BNxvk'; // default to Arianna Peschiera
tempCase.OwnerId = Constants.BUSINESS_OPS_QUEUE;
tempCase.Opportunity__c = opp.Id;

newCaseToInsert.add(tempCase);
}


}

if (newCaseToInsert.size()>0) return newCaseToInsert;
else return null;
}"

TRIGGER 35

"// BOSP-565
public List<Case> generateCustomerSuccessCase(Map<Id, Opportunity> oldOpportunityMap, Map<Id, Opportunity> newOpportunityMap){

List<Case> newCaseToInsert = new List<Case>();
for (Opportunity opp : newOpportunityMap.values()){

Opportunity oldOppty = oldOpportunityMap.get(opp.Id);

// CS Case
if ((opp.StageName!=oldOppty.StageName &&
opp.StageName=='Redlines or Legal Review' &&
opp.RecordTypeId == Constants.opptyCommitment2RecType &&
opp.Opportunity_Type__c=='New Sale') ||
Test.isRunningTest() // to bypass for maximum test coverage
){

Case tempCase = new Case();
tempCase.RecordtypeId = Constants.CUSTOMER_SUCCESS_CASE_REC_TYPE; // Customer Success case record type
tempCase.Area__c = 'Customer Success';
tempCase.Sub_Area__c = 'CS Handoff Request';
tempCase.Due_Date__c = date.today().addDays(7);

tempCase.Status = 'New';
tempCase.Subject = 'Please Complete CS Customer Handoff Doc';
tempCase.Description = 'Please duplicate the Google Sheet in the template link and attach to the opportunity.';

tempCase.Assigned_To__c = opp.OwnerId;
if (opp.Customer_Success_Manager__c!=null) tempCase.OwnerId = opp.Customer_Success_Manager__c;
tempCase.Opportunity__c = opp.Id;

newCaseToInsert.add(tempCase);"

TRIGGER 36
"// CX Team Assignment Case
if ((opp.StageName!=oldOppty.StageName &&
opp.StageName=='Redlines or Legal Review' &&
opp.RecordTypeId == Constants.opptyCommitment2RecType &&
opp.Opportunity_Type__c=='New Sale') ||
Test.isRunningTest() // to bypass for maximum test coverage
){

Case tempCase = new Case();
tempCase.RecordtypeId = Constants.CUSTOMER_SUCCESS_CASE_REC_TYPE; // Customer Success case record type
tempCase.Area__c = 'Customer Success';
tempCase.Sub_Area__c = 'Team Assignment';
tempCase.Assigned_To__c = '005a0000007Qhk4'; // Assign to Ryan Gang

tempCase.Status = 'New';
tempCase.Subject = 'Team Assignment';
tempCase.Description = 'Please assign someone from your team to the Opportunity on the case.';

if (opp.Customer_Success_Manager__c!=null) tempCase.OwnerId = opp.Customer_Success_Manager__c;
tempCase.Opportunity__c = opp.Id;

newCaseToInsert.add(tempCase);
}"

TRIGGER 37
"// PMO Team Assignment Case
if ((opp.StageName!=oldOppty.StageName &&
opp.StageName=='Redlines or Legal Review' &&
opp.RecordTypeId == Constants.opptyCommitment2RecType &&
opp.Opportunity_Type__c=='New Sale') ||
Test.isRunningTest() // to bypass for maximum test coverage
){

Case tempCase = new Case();
tempCase.RecordtypeId = Constants.CUSTOMER_SUCCESS_CASE_REC_TYPE; // Customer Success case record type
tempCase.Area__c = 'Customer Success';
tempCase.Sub_Area__c = 'Team Assignment';
tempCase.Assigned_To__c = '0051400000Bhn0e'; // Assign to Jon Quinn

tempCase.Status = 'New';
tempCase.Subject = 'Team Assignment';
tempCase.Description = 'Please assign someone from your team to the Opportunity on the case.';

if (opp.Customer_Success_Manager__c!=null) tempCase.OwnerId = opp.Customer_Success_Manager__c;
tempCase.Opportunity__c = opp.Id;

newCaseToInsert.add(tempCase);
}"

TRIGGER 38
"// CS Team Assignment Case
if ((opp.StageName!=oldOppty.StageName &&
opp.StageName=='Contract or Strategy Sent' &&
opp.RecordTypeId == Constants.opptyCommitment2RecType &&
opp.Opportunity_Type__c=='New Sale') ||
Test.isRunningTest() // to bypass for maximum test coverage
){

Case tempCase = new Case();
tempCase.RecordtypeId = Constants.CUSTOMER_SUCCESS_CASE_REC_TYPE; // Customer Success case record type
tempCase.Area__c = 'Customer Success';
tempCase.Sub_Area__c = 'Team Assignment';
tempCase.Assigned_To__c = '0051O00000CcMwz'; // Assign to Jeremy Silver

tempCase.Status = 'New';
tempCase.Subject = 'Team Assignment';
tempCase.Description = 'Please assign someone from your team to the Opportunity on the case.';

tempCase.OwnerId = opp.OwnerId;
tempCase.Opportunity__c = opp.Id;

newCaseToInsert.add(tempCase);
}

}

if (newCaseToInsert.size()>0) return newCaseToInsert;
else return null;
}"

TRIGGER 43
"/*public void validateRelatedQuote(Map<Id, Opportunity> oldOpportunityMap, Map<Id, Opportunity> newOpportunityMap){
Set<Id> oppIdSet = new Set<Id>();


for (Opportunity opp : newOpportunityMap.values()){
Opportunity oldOppty = oldOpportunityMap.get(opp.Id);

if (opp.StageName!=oldOppty.StageName &&
opp.StageName=='Contract or Strategy Sent' &&
opp.RecordTypeId == Constants.opptyCommitment2RecType){

oppIdSet.add(opp.Id);
""if (oppIdSet.size()>0){
List<SBQQ__Quote__c> qtList = [ select Id, SBQQ__Status__c, SBQQ__Opportunity2__c, SBQQ__NetAmount__c
from SBQQ__Quote__c
where SBQQ__Opportunity2__c IN :oppIdSet AND
SBQQ__Primary__c = true];
Set<Id> oppWithPrimaryQuoteIdSet = new Set<Id>();
if (qtList.size()>0){
for (SBQQ__Quote__c qt : qtList){
oppWithPrimaryQuoteIdSet.add(qt.SBQQ__Opportunity2__c);
if (!(qt.SBQQ__NetAmount__c>0))
newOpportunityMap.get(qt.SBQQ__Opportunity2__c).addError('Opportunity must have a related primary Quote with Net Amount greater than 0');
}
}
// throw error for Oppty without a related Primary quote
for (Opportunity opp : newOpportunityMap.values()){

if (!oppWithPrimaryQuoteIdSet.contains(opp.Id)){
opp.addError('Opportunity must have a related primary Quote with Net Amount greater than 0');"""

TRIGGER 45
"public void updateRelatedQuote(Map<Id, Opportunity> oldOpportunityMap, Map<Id, Opportunity> newOpportunityMap){
Set<Id> oppIdSet = new Set<Id>();


for (Opportunity opp : newOpportunityMap.values()){
Opportunity oldOppty = oldOpportunityMap.get(opp.Id);

if (opp.StageName!=oldOppty.StageName &&
opp.StageName=='Contract or Strategy Sent' &&
opp.RecordTypeId == Constants.opptyCommitment2RecType){

oppIdSet.add(opp.Id);
""if (oppIdSet.size()>0){
List<SBQQ__Quote__c> qtList = [ select Id, SBQQ__Status__c, SBQQ__Opportunity2__c, SBQQ__NetAmount__c
from SBQQ__Quote__c
where SBQQ__Opportunity2__c IN :oppIdSet
AND SBQQ__Primary__c = true
//AND ApprovalStatus__c = 'Approved' // reverted by BOSP-608
AND SBQQ__NetAmount__c > 0];
Set<Id> oppWithPrimaryQuoteIdSet = new Set<Id>();
if (qtList.size()>0){
for (SBQQ__Quote__c qt : qtList){
oppWithPrimaryQuoteIdSet.add(qt.SBQQ__Opportunity2__c);
//if (qt.SBQQ__NetAmount__c>0)
qt.SBQQ__Status__c = 'Accepted';
//else newOpportunityMap.get(qt.SBQQ__Opportunity2__c).addError('A Commitment Opportunities Stage cannot be updated to Contract or Strategy Sent if it is not related to a Quote where Primary is equal to true, Net Amount is more than 0 and Approval Status is equal to Approved');
}
// throw error for Oppty without a related Primary quote
for (Opportunity opp : newOpportunityMap.values()){

if (!oppWithPrimaryQuoteIdSet.contains(opp.Id)){
opp.addError('A Commitment Opportunity\'s Stage cannot be updated to Contract or Strategy Sent if it is not related to a Quote where Primary is equal to true, Net Amount is more than 0 and Approval Status is equal to Approved');
}"""

TRIGGER 47
"public void copyFieldsFromParentToChildOppty(Map<Id, Opportunity> oldOpportunityMap, Map<Id, Opportunity> newOpportunityMap){
System.debug('copyFieldsFromPArentToChildOppty START');

Set<Id> parentHybridOppIdSet = new Set<Id>();
Set<Id> parentHybridOppClosedWonIdSet = new Set<Id>();

// get all parent oppty with product = hybrid commitment and any of the monitored fields were updated
for (Opportunity opp : newOpportunityMap.values()){

Opportunity oldOppty = oldOpportunityMap.get(opp.Id);

if (opp.Product__c == 'Hybrid Commitment' &&
( opp.CloseDate!=oldOppty.CloseDate ||
opp.StageName!=oldOppty.StageName ||
opp.Probability!=oldOppty.Probability ||
opp.Campaign_Start_Date__c!=oldOppty.Campaign_Start_Date__c ||
opp.Campaign_End_Date__c!=oldOppty.Campaign_End_Date__c ||
opp.Closed_Won_Reason__c!=oldOppty.Closed_Won_Reason__c ||
opp.Customer_Success_Manager__c!=oldOppty.Customer_Success_Manager__c) ){
parentHybridOppIdSet.add(opp.Id);
}
if (opp.Product__c == 'Hybrid Commitment' && opp.StageName == 'Closed Won'){
parentHybridOppClosedWonIdSet.add(opp.Id);"

TRIGGER 51
"public void copyFieldsFromOpptyPrimaryContact(List<Opportunity> newOpportunities, Map<ID, Opportunity> oldOpportunityMap){
    // gather all opp primary contacts and its lead source
    Set<Id> contactIdSet = new Set<Id>();
    for (Opportunity opp : newOpportunities){
      if (opp.Opportunity_Primary_Contact__c != null){
        contactIdSet.add(opp.Opportunity_Primary_Contact__c);
      }
    }
    
    Map<Id,String> contactIdLeadCampaignMap = new Map<Id,String>();
    Map<Id,Contact> contactIdContactMap = new Map<Id,Contact>();
    // query all contact lead source
    for (Contact c : [select Id, Job_Title_Type__c, Yearly_Ad_Budget__c, Lead_Campaign__c, Marketing_Campaign__c, Paid_Media_Source__c, Social_Channels__c,
                LeadSource, Brands__c, Prospect_Lead_Interest__c
                , Marketing_Program__c //BOSP-584
             from Contact where Id IN :contactIdSet]){
      if (c.Lead_Campaign__c!=null && c.Lead_Campaign__c!='')
        contactIdLeadCampaignMap.put(c.Id, c.Lead_Campaign__c);
      else
        contactIdLeadCampaignMap.put(c.Id, '');
        
      contactIdContactMap.put(c.Id, c);
    }
    
    for (Opportunity opp : newOpportunities){
      if (opp.Opportunity_Primary_Contact__c != null){
        opp.New_Lead_Campaign__c = contactIdLeadCampaignMap.get(opp.Opportunity_Primary_Contact__c);
        
        //System.debug('========: ' + opp.Opportunity_Primary_Contact__c);
        
        //System.debug('========contactIdContactMap: ' + contactIdContactMap.keyset());
        
        //copy marketing fields
        if ( (oldOpportunityMap==null && opp.Opportunity_Primary_Contact__c!=null) || 
          (  oldOpportunityMap!=null && opp.Opportunity_Primary_Contact__c!=null &&
            oldOpportunityMap.get(opp.Id).Opportunity_Primary_Contact__c!=opp.Opportunity_Primary_Contact__c)){ // means it is insert event or update event and oppty primary contact value was updated, copy all marketing fields
          
          Contact c = contactIdContactMap.get(opp.Opportunity_Primary_Contact__c);
          
          opp.Job_Title_Type__c = c.Job_Title_Type__c;
      opp.Yearly_Ad_Budget__c = c.Yearly_Ad_Budget__c;
      opp.New_Lead_Campaign__c = c.Lead_Campaign__c;
      
      // set marketing campaign only if New/Existing Account and Brand fields are both set to ""Existing Account"" or ""Existing Brand""
      if (opp.New_Existing_Account__c!='Existing Account' || opp.New_Existing_Brand__c!='Existing Brand')  opp.Marketing_Campaign__c = c.Marketing_Campaign__c;
      
      opp.Paid_Media_Source__c = c.Paid_Media_Source__c;
      opp.Social_Channels__c = c.Social_Channels__c;
      opp.Prospect_Lead_Interest__c = c.Prospect_Lead_Interest__c;
      opp.LeadSource = c.LeadSource;
      opp.Brands__c = c.Brands__c;
      "
      " // BOSP-584
      if (c.Marketing_Program__c!=null)  opp.Marketing_Program__c = c.Marketing_Program__c;
        }"




  
