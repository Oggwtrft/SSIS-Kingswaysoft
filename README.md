# SSIS-Kingswaysoft for Dynamics CE
Package examples SSIS Kingswaysoft for data migration --> D365 CE

###### The main thing, that should be done is: Target System should have only core for migration users, business unit, transaction currency, etc. The less the better, to avoid using text lookups while migration.
**Always map createdon field on overriddencreatedon. And all records with new GUID using text-lookup feature.**

## USERS
Before Users migration update business unit primary field (name), it can be done manually or you can update it via ID replacement of Source BU.
The user could be migrated and records could be assigned without buying any licenses if while migration the Salesperson role would be assigned.
### Users migration steps:
1. Check createdby field, start migration with those, who created others
2. Double-check for any createdby lookup missings and migrate the rest 
#### In order the user is already in a system:
1. Migrate user from source with native ID (create a duplicate, but with correct guid)
2. Delete user in Active Directory (permanently)
3. Change domainname (username) of deactivated record
4. Create the same user in Active Directory and assign licenses, it automatically will be mapped by the domainname (username) 
Create and assign custom role to let the creator of the record set correct ‘createdon’, etc fields. 
#### Note
**“createdby” field could be set only when creating the record, keep in mind that it is not possible to delete a User, check the lookup before migration.**

## GENERAL RESTRICTIONS

* “createdon” field cannot be set directly, destination “overriddencreatedon” should be set equal to source “createdon” to save the correct date (the User, set on createdby field must have privilege to change this field);
* “createdby” cannot be modified and with User lookup requires User to be licensed and have specific role;
* “ownerid” doesn’t require User licenses, can be modified;
* “modifiedby” can be modified, for User has the same requirements as “createdby”;
* for entities, containing recursive fields (e.g.: contact, with “parentcontact”), just ignore such field and then update, fetching only recordid and target lookup field.

Sometimes it is long-running process to fetch Source system (in most cases because the filter is complex) in this case the solution is to take all linked entities and use Left Outer Join to take the records you need.
The process of selecting would be running on your local machine except online environment,  config ‘Task Data Flow’ properly to reach more speed. Use “Properties” section to set the right Batch and Buffer size.
Example: For plain entities like  “Queue” or “Case” we can set Batch 20.000 and Buffer 40Mb, if you want to count the records — use 30-50k Batch and AutoBuffer or <40Mb Buffer. For attachments and other big-size entities set <500 Batch and >40Mb Buffer.
The challenge is to config the Task in such a way that Fetch Source (Source Batch) and writing data on your local machine (Task Batch and Buffer) would have taken a period of time that would be equal to writing in Destination System (Batch and Multithreading). If the batch will be too big — the time-out Error will arise, and if too small — the  system will block the request and slow-down the migration process.


Some entities like “Dashboards” or “Personal Views” must be migrated using Impersonalization .
In such cases “Foreach loop container” should be used.
*while impersonalization in “Personal Views” and “Dashboards” it is not possible to change the ownership and map the View to another User.
