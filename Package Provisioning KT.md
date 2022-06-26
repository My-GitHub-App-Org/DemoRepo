# Package Provisioning KT

## Problem Statement

- FS is a platform on top of which you can build healthcare applications. e.g. PA
- Why FS was selected as the core platform and why we didn't go for the `CAP` model offered by the BTP?
  - FS was inspired from `FHIR`
  - To support the interoperability between different healthcare applications
  - Advantages:
    - `FHIR` has a well predefined set of healthcare entities like `Patient`. So we can say it is model driven
    - It is possible to extend/constrain the existing entities, build other entities on top of the existing ones

### Identifying FHIR resources

- What is the difference between `id` and `identifier` for a resource?

  - <details>
      <summary><b>Expand me!</b></summary>
      <p>
        the .id is controlled by the local server. As a resource is coped from server to server, it will change. it's basically the internal primary key for the object, and it's entirely controlled by the FHIR server itself (or, more precisely, by the interaction between the client and server). So it's not a portable identifier.<br />
        But almost all the resources correspond to (somewhat) real world entities that also are recorded in other systems, and that are assigned portable identifiers that are used across multiple systems to track the entity. These identifiers are constant as a resource (or other forms of representation of the real world entity) are copied around and moved from place to place. Some identifiers are assigned by external (government) agencies. Identifiers includes things like Patient MRNs, Provider Numbers etc. Often, because of distributed record processing, entities have many identifiers to carry, and there's a whole business in arbitrating between them.
      </p>
    </details>

- `http://<core-svc>/fhir/Patient/123/_history/2`

  - At a given point in time it is possible to maintain multiple versions of the patient resource behind the logical id

- What is unique about a resource for the FHIR server?

  - `Server + resource type + id + version`

- What are `canonical resources`?

  - If you consider the above given patient resource with the `logical id` it is specific/unique for only a particular FHIR server
  - e.g. `https://hl7.org/fhir/StructureDefinition/Patient`
  - `Canonical resources` are unique across different FHIR servers
  - `Conformance` and `terminology` resources are by default `canonical`
  - It is possible to define additional `canonical` resources
    - e.g. We have defined business configuration as a canonical resource
  - How to uniquely identify `canonical` resources
    - Using the `URL + Business Version`

### Identifying FHIR profiles

- At a given point of time only one version of the FHIR resource will be active in a given server
  - For a patient there can be multiple profiles with multiple versions
  - This is one fundamental difference between `canonical` resources and the normal resources. For `canonical` resources the business version is maintained by the author, whereas for the normal resources the version is maintained by the FHIR server
- <img src="/Users/I353767/Documents/SAP Health/Fhir Packages/identifying_fhir_profiles.png" alt="image-20211027051751701" style="zoom:50%;" />
- Here we can see a chain of reference between different FHIR profiles
  - If you change version of one profile, you have to update in all the resources which was pointing to it earlier

### Version Complexity

- <img src="/Users/I353767/Library/Application Support/typora-user-images/image-20211027053350369.png" alt="image-20211027053350369" style="zoom: 25%;" />
- e.g. At this point in time PA has patient profile 4.0 is being used as an active version
  - In the next release we would like to add a flag for the patient to determine whether it is an inpatient or outpatient. We should update the profile in this case. When you are updating the profile the business version should be incremented
- Problems:
  - FHIR content author: how to manage the different versions?
  - How the system understands the version complexities at runtime?
  - How to deliver the version updates to the customers and partners without any disruption?

## FHIR Packages

- How to solve the above mentioned version complexities?
  - Don't version individual resources, version the set
- Advantages:
  - Maintanence
    - No need to version individual resources
    - No cascading profile updates
    - No version hell
  - Distribution
    - Helps for smoother delivery
  - Communication
    - A unique canonical name for packages
      - Clear and simple way to communicate update using package versions
  - Eco system
    - Aligned with FHIR packaging methodology
- <img src="/Users/I353767/Library/Application Support/typora-user-images/image-20211027055957789.png" alt="image-20211027055957789" style="zoom:50%;" />
  - FHIR package is a folder containing collection of `canonical` resources
  - Guide line is one json file per resource
- As a first offering from FS we are providing packages containing the `canonical` resources
  - It makes more sense to create the `transactional` resources in the customer system itself
- Data pachages
  - Contains set of `transactional` resources
  - One use case might be for demo content loading
- Going forward if there is a new feature that is to be delivered from FS or PA, the expectation is that corresponding FHIR package version is already updated in the tenant

### SAP Patient Accounting

- SAP Patient Accounting consists of many components like FS, BECS, Approuter, UI application- RAB
- The boundary condition which is taken into consideration is that the architecture is model driven
  - The code alone can't work, it requires some model data (content)
- Application is build in a way so that it will work with different meta models
  - Since its a health care application the model which we are using is the FHIR model
- Different layers of content that we are having:
  - hl7 fhir r4 content (basic content from hl7 4.0.0)
  - sap fhir r4 content (resources needed for auth, rule, plugin svcs)
  - sap fhir r4 pm content (application agnostic content - content common for PA, PM)
  - sap fhir r4 pac content
  - sap fhir r4 pac de content
- In the current architectural setup we have some gaps in delivering the content

### Notes

- What is Package2ResourceMapping table

  - [link](https://github.wdf.sap.corp/sap-health/fhir-package-server/blob/docs/designdoc/docs/FHIR_Package_Provisioning_Listener.md#package2resourcemapping-table-)
  - We will update this table as part of PP for canonical resources
  - For new resources we will add the entry
  - For existing resources we will add a new entry and in the `LastLogicalID` & `LastMetaVersion` columns we will add last the values of the last active canonical resource

- Docs: [link](https://github.wdf.sap.corp/sap-health/fhir-package-server/blob/docs/designdoc/docs/FHIR_Package_Provisioning_Listener.md#package2resourcemapping-table-)

- Provisioning scenario

  - | Package name & Version | Contents    |
    | ---------------------- | ----------- |
    | P1V1                   | SAPPatient1 |
    | P1V2                   | SAPPatient2 |

    Even if we deprovision P1V1 we should be able to create patients with meta.profile of SAPPatient1

- <img src="/Users/I353767/Documents/SAP Health/Fhir Packages/images/provisioning.png" alt="image-20211029164900404" style="zoom: 33%;" />

- Where we use the ValueSet

  - If we have a ValueSet with a url, version and code as "male|female"
  - This will be added in the Patient SD snapshot for field gender with the ValueSet url and version, so that when patient instances is created we can store the values "male|female"

- If we are deprovisioning one package we will not delete any tables or columns created with provisioning of the same (soft delete)

- Form core-svc while deprovisioning we will remove the entry from the inline table, and in meta-svc we will remove the entry from the respective tables (e.g. removing SAPPatient from SD table)

- What is happening in current CUD scenario

- POST P1 with meta.version 123 (meta.version is used to fetch the records using _history)

  - | ID   | Meta.version | ValidTill | isDeleted |
    | ---- | ------------ | --------- | --------- |
    | 101  | 123          | Null      | false     |

- PUT P1 will update the meta.version to 124

  - | ID   | Meta.version | ValidTill  | isDeleted |
    | ---- | ------------ | ---------- | --------- |
    | 101  | 123          | 12-12-2020 | false     |
    | 101  | 124          | Null       | false     |

- DELETE P1

  - | ID   | Meta.version | ValidTill  | isDeleted |
    | ---- | ------------ | ---------- | --------- |
    | 101  | 123          | 12-12-2020 | false     |
    | 101  | 124          | 15-12-2020 | false     |
    | 101  | 124          | 15-12-2020 | true      |

- GET P1

  - Will return null

## Open Questions

- `http://<core-svc>/fhir/Patient/123/_history/2`
  - What is the meaning of having multiple versions of the patient resource instance?
- What you mean by multiple active version for a `canonical` resource?
  - Why cant we insert new `structure definition` for a patient into the table
- What is model driven approach?
- How `profile` and `canonical` resources are linked?
  - `Canonical` referencing happens via meta.profile
  - `target profile` in referenced based profiling
- Referencing within the package and referencing across the package?
- How come lexical ordering is coming into picture?
- How SAPPatient profile become non compatible with its target profile
- Where is the inline-snapshot of StructureDefinition stored in conformance service?
- Inside the conformance event handler we are reading the inline-snapshot by id. Will it be unique like the url and version combination?

## Closed Questions

- PackageProvisioningEventHandlerAnnotation is a @Component, So spring takes care of initialising it
  - Is there any post bean processor?
    - Yes. HandleRegistry class does some of the post bean processing logic
- While sending the response from `bounded-context` to PPS, there are some extra properties added (like status). It is not present in the PackageContext POJO class
  - As a response we are sending PackageResponseContext class object (which is prepared using the PackageContext object), which has a mergePackageResponseContext method which can be used by the services to add the service specific properties as a response
- Is the same rollback method invoked when PPS raise even to rollback?
  - Currently this part is not implemented. Once the rollback event is received, it should read all the handlers from the registry and invoke the rollback method
- What is the cache clearing based on ff?
  - This is for the older implementation before provisioning. On deploy-svc app startup we load the hdb artefacts for creating the tables from the object store to the cache (memory space, not the disk space).
  - If the packaging ff is on before every method in deploy-svc we are clearing the cache
- How to onboard multiple hl7 instances together?
  - using the command mentioned here: https://github.wdf.sap.corp/sap-health/fhir-package-server/blob/docs/designdoc/docs/fhir-packages.md
- What is the use of AbstractPackageHandler class?
  - To keep the common implementation that will be re-used in multiple handlers in one place (eg. SDhandler & SPhandler re-uses the same logic)
- Why are we adding the DDLAdminUser (for core)
  - [fhir-server-framework/fhir-server-package/docs/package-eventing/event-processing at master · sap-health/fhir-server-framework](https://github.wdf.sap.corp/sap-health/fhir-server-framework/tree/master/fhir-server-package/docs/package-eventing/event-processing#existing-eventing-flow)
  - Let's say we have SAPPatient profile as a json as part of a new package provisioning. And SAPPatient contains a new field, so the PATIENT_EXTENSION table should be updated to add the new column. For this we need to fire the DDL queries, we require the DDLAdmin user for this purpose
- While off boarding the hl7 instance and error status is returned while provisioning, PPS will raise the same rollback events?
  - Yes. It will be the same rollback method which services invoke if there is error while provisioning
- Inside ConformanceEventHandler why there is callDeploySvc() inside handleTableDDL()?
  - If we update a custom resource, and the event reached ConformanceEventHandler the inline table will already have the entry for the custom resource. But still we have to call the deploy svc and update/recreate the table, since it is not a profile its the update of the same resource 


## Open Points

- While invoking rollback, for `core-svc` we also have to delete the DDLUSER
- If event publishing fails when the status is sent from FS -> PPS, we don't have a resilience mechanism
  - If there is error while publishing the success status from core to PPS, if the publishing fails then rollback will not happen for core, it will happen for only the services which are onboarded before core
- In the Package2ResourceMapping table
- Multiple Canonical resources can be active at the same time, so what is bring back to life for entries in the Package2ResourceMapping table?
- Why the packageContext object is not passed to validateMessage()?
  - Validation should happen before creating the PackageContext
- Do we need to insert SP again for rule?
- Why are we doing profile validation in case of create resource ? JSON walker will only return valid resource object based on the profile
- If the identifier is added inside the session.create() then what is the identifier added in the service layer?
- What happens to the entries in resource table in case of an update?
- Why did we made use of reference table instead of the foreign key concept?

## References

### Grooming

- https://sap-my.sharepoint.com/:v:/r/personal/prasanna_prabhu01_sap_com/Documents/Recordings/[Team%20Grooming]_%20Sprint%2018-20210817_104640-Meeting%20Recording.mp4?csf=1&web=1&e=75gc5q
- https://sap-my.sharepoint.com/:v:/r/personal/suraj_choudhary_sap_com/Documents/Recordings/Grooming%20Condt..-20210817_140358-Meeting%20Recording.mp4?csf=1&web=1&e=lf4ApT
- https://sap-my.sharepoint.com/:v:/p/prasanna_prabhu01/Eacs9iR9_lJGngKRMe6c-uQBvaroTK0xvoXsGitbXAZd6A

### KT Recording

- https://sap.sharepoint.com/:v:/r/teams/SAPHealth_All/Shared%20Documents/team-BLR2/package-provisioning/Package%20Provisioning%20KT%20-%201-20211013_160521-Meeting%20Recording.mp4?csf=1&web=1&e=09JEHc
- https://sap.sharepoint.com/:v:/r/teams/SAPHealth_All/Shared%20Documents/team-BLR2/package-provisioning/Package%20Provisioning%20KT%20-%202-20211014_160148-Meeting%20Recording.mp4?csf=1&web=1&e=qbd3Oq
- https://sap.sharepoint.com/:v:/r/teams/SAPHealth_All/Shared%20Documents/team-BLR2/package-provisioning/PackageCategory%20!=%20FS-20211027_160254-Meeting%20Recording.mp4?csf=1&web=1&e=Xy8AXQ
- https://sap-my.sharepoint.com/personal/nikesh_t_t_sap_com/_layouts/15/onedrive.aspx?id=%2Fpersonal%2Fnikesh%5Ft%5Ft%5Fsap%5Fcom%2FDocuments%2FRecordings%2FSync%20%5F%20changes%20done%20wrt%20PackageCategory%20%21%3D%20FS%2D20211027%5F160254%2DMeeting%20Recording%2Emp4&parent=%2Fpersonal%2Fnikesh%5Ft%5Ft%5Fsap%5Fcom%2FDocuments%2FRecordings
- [Onboarding of application packages](https://sap.sharepoint.com/teams/SAPHealth_All/Shared%20Documents/Forms/AllItems.aspx?id=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021%2FPatient%20Accounting%20Product%20Review%2D20211214%5F130202%2DMeeting%20Recording%2Emp4&parent=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021)
- [Multiple instance handling](https://sap.sharepoint.com/teams/SAPHealth_All/Shared%20Documents/Forms/AllItems.aspx?FolderCTID=0x0120004CFD61B8133F994ABC61C1010EB3736C&id=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021%2FPatient%20Accounting%20Product%20Review%2D20211130%5F130154%2DMeeting%20Recording%2Emp4&parent=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021)`
- [FS Instance Creation is fhir r4 package](https://sap.sharepoint.com/teams/SAPHealth_All/Shared%20Documents/Forms/AllItems.aspx?FolderCTID=0x0120004CFD61B8133F994ABC61C1010EB3736C&id=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021%2FPatient%20Accounting%20Product%20Review%2D20211116%5F130131%2DMeeting%20Recording%2Emp4&parent=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021)
- [FS Instance Creation with fhir r4 package](https://sap.sharepoint.com/teams/SAPHealth_All/Shared%20Documents/Forms/AllItems.aspx?FolderCTID=0x0120004CFD61B8133F994ABC61C1010EB3736C&id=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021%2FPatient%20Accounting%20Product%20Review%2D20211102%5F083309%20%2DMeeting%20Recording%2Emp4&parent=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021)
- [FHIR Package Registry Command](https://sap.sharepoint.com/teams/SAPHealth_All/Shared%20Documents/Forms/AllItems.aspx?FolderCTID=0x0120004CFD61B8133F994ABC61C1010EB3736C&id=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021%2FPatient%20Accounting%20Product%20Review%2D20210921%5F130156%2DMeeting%20Recording%2Emp4&parent=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021)
- [FHIR Package Overview](https://sap.sharepoint.com/teams/SAPHealth_All/Shared%20Documents/Forms/AllItems.aspx?FolderCTID=0x0120004CFD61B8133F994ABC61C1010EB3736C&id=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021%2FPatient%20Accounting%20Product%20Review%2D20210907%5F093251%2DMeeting%20Recording%2Emp4&parent=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021)
- [Package Registry Commands](https://sap.sharepoint.com/teams/SAPHealth_All/Shared%20Documents/Forms/AllItems.aspx?id=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021%2FPatient%20Accounting%20Product%20Review%2D20210921%5F130156%2DMeeting%20Recording%2Emp4&parent=%2Fteams%2FSAPHealth%5FAll%2FShared%20Documents%2Fteam%2DUnit%2FRecordings%2FProduct%20Review%2F2021)
- Architect doc : https://saphealthengineering.cfapps.eu10.hana.ondemand.com/Architecture/definition/foundation_services/content-packages/
- Recording on FHIR packages from Shankar : https://sap-my.sharepoint.com/:v:/p/vandana_tadigadapa/EbFNNjN_IvZItyHpofYws1MBsp9jruA3dCExnM6fR7Wy7g
- 
- Closed PRs
  - https://github.wdf.sap.corp/sap-health/fhir-server-framework/pull/552
  - https://github.wdf.sap.corp/sap-health/fhir-server-framework/pull/597
  - https://github.wdf.sap.corp/sap-health/fhir-server-framework/pull/634
  - https://github.wdf.sap.corp/sap-health/fhir-server-framework/pull/693
  - https://github.wdf.sap.corp/sap-health/fhir-server-framework/pull/708
  - https://github.wdf.sap.corp/sap-health/fhir-meta-svc/pull/728

### HL7

- Terminology
  - https://www.youtube.com/watch?v=3ursz6nyDh4

### Docs

- https://github.wdf.sap.corp/sap-health/fhir-package-server/blob/docs/updatedPackage2Resource/docs/FHIR_Package_Provisioning_Listener.md
- [fhir-package-server/fhir-packages.md at docs/designdoc · sap-health/fhir-package-server](https://github.wdf.sap.corp/sap-health/fhir-package-server/blob/docs/designdoc/docs/fhir-packages.md)
- [Merged profiles](https://github.wdf.sap.corp/sap-health/fhir-server-framework/blob/master/fhir-server-persistence/docs/extensibility/implementation.md)
- 





























## Planned Items

- Write operations to bounded cache and master tables & P2RMtable should happen in single transaction
  - Package provisioning is asyc
  - No performance overhead
  - Bundle Transaction will not insert data into inline table
  - Understanding the current flow, How I am able to rollback ?
  - Is the default profile is reset in core db?
- Cache the db artifacts and alter scripts in a central place
  - Issue with running all sql scripts in one shot
    - Scenario profile 1 - with primitive datetime field (birth_datetime)
    - profile 2 - with primitive datetime field (birth_datetime)
    - If we run this sql statement generator, sql statement generator will generate 2 seperate alter scripts for both profiles
      - delta check will be missed
      - If we try to create same column twice we will get JDBC exception
- Only create the ddl user if there is alter scripts
  - 2 use-cases 
    - running sql scripts
    - for HANA Usage (Hana sequence creation)
    - In the current flow DDL user is created as part of onboarding
      - Package provisioning for category = FS is similar to onboarding
  - Understand the current flow
  - Is the DDL connection open
- First execute DML then execute DDL
- Testing with package containing SD of extension, SD of custom resource, SD of Profile referencing the extension, SP of extension, terminology resources 
- Utilise the utility class for common logic 

## Current Implementation

- For a bounded-context write operations to the master tables are happening in single transaction
  - We are able to rollback in case of an exception
- Write operations to bounded cache will not be referencing to any transaction-id hence the changes will not be reverted
- All of the files in the package are processed one by one

### SD Handler

#### Conformance

- Conditional create to the master table

#### Core

- Existing ConformanceEventHandler is called
- Execution flow
  - Load the inline-bundle from meta using the id from conformance
  - Store in the bounded cache table (API internally takes care of the table-map generation & terminology-binding generation internally)
  - Check SD is present in the inline-table
    - Yes
      - Check SD is a new resource or new profile (baseDefinition == "DomainResource")
        - New resource
          - Generate HDB artifact for the resource bundle and call deploy-svc
        - New profile
          - Generate alter scripts and execute
    - No
      - Generate HDB artifact for the resource bundle and call deploy-svc

#### Other Bounded-Contexts

- Load the inline-bundle from meta using the id from conformance
- Store in the bounded cache table

### SP Handler

#### Conformance

- Store in the master table
- Store resource in bounded cache table

#### Other Bounded-contexts

- Store resource in bounded cache table

### Terminology Resource Handler

- Conditional create to the master table

## Proposed solutions

### Caching DB Artifacts for Core-svc

#### Solution 1

- Cache DB artifacts in the StructureDefinition handler
- Call deploy-svc end-point which accepts a list of Multipart files

##### Changes Required

- Inside StructureDefinitionHandler
  - Load the SD inline bundle from conformance
  - Check whether the SD is a new resource by checking bounded cache table
  - Update the bounded cache table
  - Generate and cache DB artifacts
  - Prepare the payload and call deploy-svc in batches

##### Pros

- Performance improvement as compared to processing each file one by one

##### Cons

- If there are multiple file as part of a package payload will be heavy 
  - Call deploy-svc endpoint in partition (use batch size)
  - New resources will be comparatively less in a package 

#### Solution 2

- Identify the list of new resource SD ids from core-svc & cache the ids
- Call deploy-svc with a list of SD ids
- Deploy-svc will load the SD inline-bundle from conformance

##### Changes Required

- Inside StructureDefinitionHandler
  - Load the SD inline bundle from meta svc
  - Check whether the SD is a new resource by checking bounded cache table
  - Update the bounded cache table
  - Cache the ids of new resource
  - Prepare the payload and call deploy-svc
- Inside deploy-svc
  - Create new end-point for accepting list of SD ids and carryout the processing
  - Load the SD inline bundle from conformance using the ids
  - Generate the DB artifacts
  - Continue the current existing flow to deploy the content

##### Pros

- Payload to deploy-svc will be light weight

##### Cons

- Need to load the SD inline bundle twice (core-svc & deploy-svc)
- Against the micro service architecture
- Deploy-svc role is to only deploy the artefacts in the db

### Rollback of Write Operations

#### Solution 1

- Since transaction-id is maintained in the resource master tables we will be able to do session.revert()
- Transaction-id is not maintained in the bounded-cache table
  - Transaction-id is maintained only for transactional data, like resource master table entries
  - Bounded-cache table stores the metadata
- For updated to bounded cache table do not perform a transaction.commit()
  - We don't require any read operations as part of package provisioning
  - Will there be a scenario where there are multiple versions of same resource present in a single package?
    - Need to check
- Session.save() which is happening at the end of execution will internally call the transaction.commit() which will persist the data in the bounded cache table

##### Changes required

- Remove the transaction.commit() from current implementation of persist into cache

##### Pros

- No changes to the session.create()

##### Cons

- Intermediate transaction.commit() will happen for resource master table insert operations which will persist the data temporarily
  - No performance impact

##### Open Questions

- Is there any impact of setting the transaction dirty=false, which is happening inside the transaction.commit()
  - We are not using this flag (dirty) in our session operations

#### Solution 2

- Modify the session.create() api to avoid intermediate transaction.commit() operations
- Same set of changes of to persist into cache method as above

##### Changes required

- Enhancement of session.create() api
- Persist into cache same as above

##### Pros

- Avoiding intermediate transaction.commit() operations

##### Cons

- Code changes for session.create() api
- Persisted transaction concept will not work
- We have implemented this approach to handle persisted transaction

##### Open Questions

- Is there any impact of avoiding the intermediate transaction.commit()?

#### Solution 3

- In case of rollback delete the entry from bounded-cache table based on the package context & version
- Make the last profile version as default with the help of Package2ResourceMapping table

##### Changes required

- Create query to delete entry from bounded cache table based on the package context & version
- Create query to get the last active profile from Package2ResourceMapping table
- Extract the url and version from the result and update the bounded cache table to make the the last version as default

##### Pros

- Existing apis can be reused to persist into cache without any modification

##### Cons

- Creation of 2 extra queries and execution 



ddl auto commit -> false

data migration will get affected 

- data migration is a topic which is currently under development, where we support change of data type of a column, first we perform the ddl then migrate the existing data in that table to this format
- If we don't commit the ddl this will not work

drop column -> sql alter scripts

drop table -> deploy-svc

- get the table name from hdb file and prepare the drop column query

Data package vs meta package

- both will be separate packages

Non-canonical data will be present in a package?

Merged profiles

- we can not perform a search based on the package context and version, because the entries will be of per resource type
- we have to store as a separate row with package information





































## Before Session.revert()

### CodeSystem

![image-20211130151703969](/Users/I353767/Library/Application Support/typora-user-images/image-20211130151703969.png)

### CodeSystem-ConceptMap

![image-20211130152042607](/Users/I353767/Library/Application Support/typora-user-images/image-20211130152042607.png)

## After Session.revert

## CodeSystem

![image-20211130152240537](/Users/I353767/Library/Application Support/typora-user-images/image-20211130152240537.png)



































SD0 -> Custom Resource

SD1 -> extension

SD2 -> Profile

SD3 -> Profile2



- Cache Resourcetype -> hdb file

- Call to deploy-svc 2 times
- Need to test with deploy-svc





Observations:

- We will be able to do the revert of master table inserts via the session.revert()
  - All the operations are done under the same transaction
  - TransactionExtension post commit logic will be clearing the entries from the master table based on the transactionID

Action Items:

- Avoid Transaction.commit() from BoundedCache UPSERT
  - Confirm the BoundedCache UPSERT is only called from ConformanceEventHandler
  - Create a separate PR for this change
  - Regression testing of all the existing scenarios with this change
- Test deploy-svc call with the following set of changes (HDB files) to understand the flow
  - ResourceType R1
    - Profile 1 -> having extension 1
    - Profile 2 -> having extension 2
  - Is the existing column for extension 1 is getting deleted and new column for extension 2 is getting created
  - Yes, table is getting recreated dropping the columns corresponding to profile 1
- Caching of DB Artifacts
  - Maintain a cache of ResourceType -> HDB file
    - If structure definition for the same resource type is present in the cache, do deploy-svc call for the first entry and clear the record from the cache. For the second structure definition generate the SQL alter scripts
- Check kind = resource is happening in HDB file generation and alter script generation
  - If the type is primitive or complex we are returning from the conformance event handler without further processing





hl7-test (before)

-----------------------

Conformance -> 25

Core -> 202

auth -> 18

Plugin -> 16

hl7-test (after-master)

----------

Core -> 214































Core

--------

![image-20211209150845676](/Users/I353767/Library/Application Support/typora-user-images/image-20211209150845676.png)

![image-20211209152041843](/Users/I353767/Library/Application Support/typora-user-images/image-20211209152041843.png)

![image-20211209152124557](/Users/I353767/Library/Application Support/typora-user-images/image-20211209152124557.png)



![image-20211209152149883](/Users/I353767/Library/Application Support/typora-user-images/image-20211209152149883.png)

![image-20211209152317087](/Users/I353767/Library/Application Support/typora-user-images/image-20211209152317087.png)

![image-20211209152422220](/Users/I353767/Library/Application Support/typora-user-images/image-20211209152422220.png)



![image-20211209152544638](/Users/I353767/Library/Application Support/typora-user-images/image-20211209152544638.png)

![image-20211209152633692](/Users/I353767/Library/Application Support/typora-user-images/image-20211209152633692.png)



![image-20211209152824502](/Users/I353767/Library/Application Support/typora-user-images/image-20211209152824502.png)

![image-20211209152900581](/Users/I353767/Library/Application Support/typora-user-images/image-20211209152900581.png)



Testing

--------------

- [x] Take the latest changes from FSF master to the commit changes PR
- [x] Write a duplicate method in the inline bundle update for conformance handler
- [x] Test a sample custom resource creation
- [x] Test all svc postman collections
  - [ ] ![img](https://saphealthblr1.jaas-gcp.cloud.sap.corp/static/75056865/images/16x16/document_add.png) [05TransactionProcessingTests.12 01- Bundle with custom resource that doesn't exist, status 400](https://saphealthblr1.jaas-gcp.cloud.sap.corp/view/all/job/run-postman/job/master/3482/testReport/(root)/05TransactionProcessingTests/12_01__Bundle_with_custom_resource_that_doesn_t_exist__status_400/)



Extensibility_New_Implementation

--------

- https://github.wdf.sap.corp/sap-health/fhir-server-framework/blob/master/fhir-server-persistence/docs/extensibility/Extensibility_New_Implementation.md#extension-scenarios
- How can we allow 0..* to 0..0, 0..1, ?
  - The data will be stored as comma separated, how to make that to single value?

Extension_within_extension

-------------

- [ ] What do you mean by field level extensions?
- [ ] What does the cardinalities in the child extension vs wrapper extension means?
  - Check the payload in the doc
- [ ] What is the Extension.extension inside the wrapper extension? what is its slicing?
  - Check the payload in the doc
- [ ] What does extension.url 1..1 means?
- [ ] What is `FHIR_ACCOUNT_DATATYPE` table? what is its usage?
  - [ ] Table Map?





























### FHIR_Package_Provisioning_Listener.md

- IsExisting?
- MergedContent?
- revert of sequence?
- All internal reference to dependent resources would be via `identifier`?
- When should the P2R mapping table get updated?
- how much of the deprovisioning listener is completed
- meta.version and history
- 2 approaches
- Execute batch together
- use case of isDefault
- In a new package only SP is present































## Sync on Package 2 Resource Mapping Table

### Package 2 Resource Mapping Table - Canonical Resource 

| Nos  | `Package name` | `Package Version` | `Canonical URL`                              | `Canonical Version` | `ResourceType`      | `IsExisting` | `LogicalID` | `CurrentMetaVersion` | `LastMetaVersion` |
| ---- | -------------- | ----------------- | -------------------------------------------- | ------------------- | ------------------- | ------------ | ----------- | -------------------- | ----------------- |
| 1    | sap-fhir-hl7   | 1.0.0             | /fhir/ValueSet-s1                            | 1.0.0               | ValueSet            | false        | -           |                      | -                 |
| 2    | sap-fhir-hl7   | 1.0.1             | /fhir/ValueSet-s1                            | 1.0.1               | ValueSet            | false        | -           |                      | -                 |
| 3    | hl7-fhir-org   | 4.0.0             | /fhir/StructureDefinition/Patient            | 4.0.0               | StructureDefinition | false        | -           |                      | -                 |
| 4    | sap-fhir-hl7   | 1.0.0             | /fhir/StructureDefinition/SAP_Patient        | 1.0.0               | StructureDefinition | false        | -           |                      | -                 |
| 5    | sap-fhir-hl7   | 1.0.1             | /fhir/StructureDefinition/SAP_Patient        | 1.0.1               | StructureDefinition | false        | -           |                      | -                 |
| 6    | sap-pm-r4      | 1.0.0             | /fhir/StructureDefinition/ActivityAllocation | 1.0.0               | StructureDefinition | false        | -           |                      | -                 |
| 7    | sap-pm-r4      | 1.0.1             | /fhir/StructureDefinition/ActivityAllocation | 1.0.0               | StructureDefinition | true         |             |                      |                   |

### BoundedCache Table - Before Provisioning Package sap-fhir-hl7 1.0.1

| `Nos` | `Package name` | `Package Version` | `Canonical URL`                       | `Canonical Version` | `LastActive` | `ResourceType`      | `Target Resource Type` | `Table Map` | `BLOB( Profile )`             | `BLOB (SP)`                |
| ----- | -------------- | ----------------- | ------------------------------------- | ------------------- | ------------ | ------------------- | ---------------------- | ----------- | ----------------------------- | -------------------------- |
| 1     | hl7-fhir-org   | 4.0.0             | /fhir/StructureDefinition/Patient     | 4.0.0               |              | StructureDefinition | Patient                | value1      | Value3                        | Value5                     |
| 2     | sap-fhir-hl7   | 1.0.0             | /fhir/StructureDefinition/SAP_Patient | 1.0.0               |              | StructureDefinition | Patient                | value2      | Value4                        | Value6                     |
| 3     | sap-fhir-hl7   | 1.0.0             | Merge-Content-Patient                 | -                   |              | StructureDefinition | Patient                | Value9      | Union of Patient & SAPPatient | SP of Patient & SAPPatient |
| 4     | sap-fhir-hl7   | 1.0.0             | /fhir/CapabilityStatement-core        | 1.0.0               |              | CapabilityStatement | CapabilityStatement    | -           | value10                       | -                          |
| 5     | hl7-fhir-org   | 4.0.0             | /fhir/CapabilityStatement-rule        | 1.0.0               |              | CapabilityStatement | CapabilityStatement    | -           | value12                       | -                          |

### BoundedCache Table - After Provisioning Package sap-fhir-hl7 1.0.1

| `Nos` | `Package name` | `Package Version` | `Canonical URL`                       | `Canonical Version` | `LastActive` | `ResourceType`      | `Target Resource Type` | `Table Map` | `BLOB( Profile )`             | `BLOB (SP)`                |
| ----- | -------------- | ----------------- | ------------------------------------- | ------------------- | ------------ | ------------------- | ---------------------- | ----------- | ----------------------------- | -------------------------- |
| 1     | hl7-fhir-org   | 4.0.0             | /fhir/StructureDefinition/Patient     | 4.0.0               |              | StructureDefinition | Patient                | value1      | Value3                        | Value5                     |
| 2     | sap-fhir-hl7   | 1.0.0             | /fhir/StructureDefinition/SAP_Patient | 1.0.0               |              | StructureDefinition | Patient                | value2      | Value4                        | Value6                     |
| 3     | sap-fhir-hl7   | 1.0.1             | /fhir/StructureDefinition/SAP_Patient | 1.0.1               |              | StructureDefinition | Patient                | value8      | Union of Patient & SAPPatient | SP of Patient & SAPPatient |
| 4     | sap-fhir-hl7   | 1.0.0             | Merge-Content-Patient                 | -                   | true         | StructureDefinition | Patient                | Value9      | Union of Patient & SAPPatient | SP of Patient & SAPPatient |
| 5     | sap-fhir-hl7   | 1.0.1             | Merge-Content-Patient                 | -                   |              | StructureDefinition | Patient                | Value7      | Union of Patient & SAPPatient | SP of Patient & SAPPatient |
| 6     | sap-fhir-hl7   | 1.0.0             | /fhir/CapabilityStatement-core        | 1.0.0               | true         | CapabilityStatement | CapabilityStatement    | -           | value10                       | -                          |
| 7     | sap-fhir-hl7   | 1.0.1             | /fhir/CapabilityStatement-core        | 1.0.0               |              | CapabilityStatement | CapabilityStatement    | -           | value11                       | -                          |
| 8     | hl7-fhir-org   | 4.0.0             | /fhir/CapabilityStatement-rule        | 1.0.0               |              | CapabilityStatement | CapabilityStatement    | -           | value12                       | -                          |

### Package 2 Resource Mapping Table - Data Package

| Nos  | `Package name` | `Package Version` | `Canonical URL` | `Canonical Version` | `ResourceType` | `IsExisting` | `CurrentLogicalID` | `CurrentMetaVersion` | `LastMetaVersion` |
| ---- | -------------- | ----------------- | --------------- | ------------------- | -------------- | ------------ | ------------------ | -------------------- | ----------------- |
| 1    | sap-data-pakg1 | 1.0.0             | -               | -                   | Patient        | false        | P1                 | V1                   |                   |
| 2    | sap-data-pakg1 | 1.0.0             | -               | -                   | Patient        | false        | P2                 | V1                   | -                 |

- Store the entries in the `P2RM table` only after we process the package entries
  - We need to populate the `LogicalID` in the table, only after the conditional create we will get the `LogicalID`
- What is the significance of `IsExisting` column
  - This column will help to identify whether the resource was already existing in the system
    - Suppose we have 2 packages as shown in the above table. Package 1 name: `sap-pm-r4` package 1 version: `1.0.0` & Package 2 name: `sap-pm-r4` package 2 version: `1.0.1`. In the version `1.0.1` only the `SAPPatient`SD got modified (assumption). So the package `1.0.1` will contain all the entries in the `1.0.0` + the modified `SAPPatient` SD. This is because if we only add the delta content in the successor packages it will be difficult to track. So here as shown in the above table entry for `ActivityAllocation`is repeated (means the same resource is present again without any `url` & `version` change). So system will perform a conditional create and finds the canonical resource was already existing, so it will update the `P2RM table` with a row for this resource and set the `IsExisting` column to true (by default it is false).
  - Open points
    - How to determine the resource is already existing or not?
      - For canonical resources, the conditional create is already returning a POJO class as a response (ResponseContext). Enhance this class to maintain a property for already existing resource
      - For SD & SP the `BoundedCache` table should be updated. How to identify the already existing entries here?
        - Since we are using the `UPSERT` query it will always create a new entry, since the `package name` & `package version` will be different each time
        - Can we somehow make use of the int[] array returned from the `JDBC` `executeBatch()` operation to determine whether it is a conditional create
        - Enhance the `BoundedCache` API to perform a conditional create. This can be performed by having a check on the `canonical url` & `canonical version`
    - Approach 2
      - No need to maintain the column `IsExisting` column in `P2RM table`
      - Duplicate entries should not be inserted to the table. But here we still need to perform the check on whether the resource is existing or not in both `master tables` and `BoundedCache` table
- How to perform the delete package?
  - Read the entries corresponds to the same `package name` & `package version` from the `P2RM` table.
  - Delete from `master table`
    - Perform the hard delete (destroy -> logical id, resourceType, meta.version) - if meta.version is not present delete the active one
    - Create a new api for this one
    - Only for the master table entries the columns logical id and current meta version will get updated
  - Delete from `BoundedCache` table
    - Delete (package name , package version , canonical url , canonical version ) -> check whether these many parameters should be passed
    - Check in Cache with canonical URL-Version if exists ( skip irrespective which package it belong )
  - For `data package`
    - Use the `LogicalID` from `P2RM table` for identifying and deleting the entries from the `master table`
    - Make the `LastMetaVersion` as active -> what does this mean?
- Consider a package where only SPs are present. Whether all of the required processing has happened properly?
- Execute all of the prepared statements for the `BoundedCache table` update together (for performance improvement)
  - In the conditional create only if the resource is not existing create the query and add to batch
- Integration with merge content logic
  - Should check with Lallu
- Merge profile
  - Check the above table `BoundedCache Table - After Provisioning Package sap-fhir-hl7 1.0.1`
    - Here the `Table Map` will contain the merged what? and will help during which operation (create, update, search)?
    - `Profile` will contain the combined extensions -> this value is loaded from the conformance and dumped in the column. This will help is case of the profile validation. For the entries other than the `Merge-Content-Patient` (this will remain as a constant string value) only this column will be useful from out of the three. For the operations where the `meta.version` is specified we can use this profile for the validation
    - `SP` we are storing SPs at resource type level and not at the individual profile level. So only one query is required to get all the SPs for a given resource type -> query the `Merge-Content-Patient`
    - If a new profile for patient comes delete the `Merge-Content-Patient` marked as `LastActive` `true`. Make the current active(one with `LastActive` `false`) as `LastActive` `true`. Create a new `Merge-Content-Patient` and make it active
    - In case of a delete operation for a resource, delete the current active `Merge-Content-Patient`. Make the `LastActive` `Merge-Content-Patient` as active
- Ability to support `InlineProfile` and `BoundedCache` table
  - FF1 -> Package provisioning
  - FF2 -> Merge profile
  - Support for IS_DEFAULT is mandatory ( for time being till the dependency on is_default is removed ) -> irrespective of the FF
  - FF1 & FF2 both are active -> use of merge content . IS_DEFAULT is removed
  - FF1 is active -> use of IS_DEFAULT is present
  - Support only FF1 for time being, till the merge content is in place

### References

https://devblogs.microsoft.com/azure-sql/the-insert-if-not-exists-challenge-a-solution/

https://sqlperformance.com/2012/12/t-sql-queries/left-anti-semi-join

https://cc.davelozinski.com/sql/fastest-way-to-insert-new-records-where-one-doesnt-already-exist

https://stackoverflow.com/questions/1361340/how-can-i-do-insert-if-not-exists-in-mysql

https://www.delftstack.com/howto/mysql/mysql-insert-if-not-exists/

### Open Points

- We can have the following two approaches to determine the is existing entries

| Approach 1: Check the P2RM table for the url & version       | Approach 2: Check the Master & BoundedCache table for the url & version |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Single point of reference: We can check only the P2RM table for existing meta data resources, canonical resources and the transactional resource | Multiple tables needs to be referred: Should check the master table & Bounded cache table separately |
| 2 db calls to P2RM table are required:<br /><ul><li>Before inserting data to check the entry is present</li><li>After inserting the data to update the logical id</li></ul><br />(In order to avoid the two db calls we can store the identifier in the P2RM table instead of logical id, during delete fetch the resource based on the identifier) -> we can not use the identifier because for most of the entries the identifier is not passed (its not a mandatory one). And When the system generates it keep the logical id and the identifier the same, if we explicitly create a UUID and append to the json node, then the logical id will not be the same. We can't use this approach since in commit with transaction id the records will get persisted, and it is mandatory to executeUpdate() to get the updated row count | only 1 db call to P2RM table is required: We can cache the insert statements (containing the logical id) |
| In case the record already exists no db calls to the Master & BoundedCache table will be made | One db call to the master and BoundedCache table to check whether the entry is already existing |
| Need to create new api for insert if not exists in the P2RM table interface | Need to create new api for for insert if not exists in the BoundedCache interface |
| No need to enhance session api                               | Need to enhance the session api to return `Is_Existing` flag in case of conditional create |

- INSERT if NOT EXISTS

  - ```sql
    INSERT INTO "com.sap.health::FHIR_BOUNDED_CACHE" (
      "PACKAGE_NAME", "PACKAGE_VERSION", "ID", "URL", "VERSION"
    )
    SELECT 'sap.pm.r4', '2.0.0', 'SAPPatient', 'http://sap.com/healthcare/PatActg/fhir/StructureDefinition/SAPPatient2', '4.0.0' 
    FROM "DUMMY"
    WHERE NOT EXISTS (
    	SELECT * from "com.sap.health::FHIR_BOUNDED_CACHE"
    	WHERE "URL"='http://sap.com/healthcare/PatActg/fhir/StructureDefinition/SAPPatient2' AND 		         "VERSION"='4.0.0'
    );
    ```

- Should we remove entries from P2RM table after de-provisioning?
  - Yes
  
- How to handle hierarchical delete package for merge content?
  - We support only one level of failure handling
  
- Before deleting how to perform the referential integrity
  - No need since we will be deleting all the entries which was part of the package



| Table     | Query                                                        |
| --------- | ------------------------------------------------------------ |
| Child     | "DELETE FROM %s WHERE \"INT_PARENT_ID\" = ? AND \"INT_VERSION_ID\" IN ( "     + "SELECT \"INT_VERSION_ID\" FROM %s WHERE \"INT_ID\" = ? AND \"INT_TRANSACTION_ID\" = ? )" |
| Reference | "DELETE FROM %s WHERE \"INT_PARENT_ID\" = ? AND \"INT_VERSION_ID\" IN ( "     + "SELECT \"INT_VERSION_ID\" FROM %s WHERE \"INT_ID\" = ? AND \"INT_TRANSACTION_ID\" = ? )" |
| Master    | "DELETE FROM %s WHERE \"INT_TRANSACTION_ID\" = ?"            |
| Resource  | "DELETE FROM %s WHERE \"INT_TRANSACTION_ID\" = ? AND \"INT_ID\" = ?" |

| Table     | Query                                                        |
| --------- | ------------------------------------------------------------ |
| Child     | "DELETE FROM %s WHERE \"INT_PARENT_ID\" = ? AND \"INT_VERSION_ID\" = ? " |
| Reference | "DELETE FROM %s WHERE \"INT_PARENT_ID\" = ? AND \"INT_VERSION_ID\" = ? " |
| Master    | "DELETE FROM %s WHERE \"INT_ID\" = ? AND \"INT_VERSION_ID\" = ? " |
| Resource  | "DELETE FROM %s WHERE \"INT_ID\" = ? AND \"INT_VERSION_ID" = ?" |



- Why are we keeping same values in `versionId` and `meta.versionId`?

- Why didn't we use `validTill` in case of delete operation (what was the use of `isDeleted` column)

- Why this check -> 

- ```java
  if (!StaticTables.getTechnicalTables().contains(type)) {
      parentSql += " and " + TechnicalFields.INT_TRANSACTION_ID.toString() + " = ?";
  }
  ```

| Table    | Query                                                        |
| -------- | ------------------------------------------------------------ |
| Resource | UPDATE "com.sap.health::FHIR_SAP_HEALTH_RESOURCE" SET "INT_VALID_TILL" = ? WHERE "INT_PARENT_ID" = ? and "INT_VALID_TILL" IS NULL and "INT_TRANSACTION_ID" = ? |
| Master   | UPDATE "com.sap.health::FHIR_PATIENT" SET "INT_VALID_TILL" = ? WHERE "INT_ID" = ? and "INT_VALID_TILL" IS NULL and "INT_TRANSACTION_ID" = ? |
|          |                                                              |
|          |                                                              |



- Create and destroy the resource in different transactions
- Create and destroy the resource in the same transaction
- When we have the canonical validator what is the use of conditional create ?
  - When processing SD foe core canonical validator for SD will not happen

- What is 2 packages have same SP but for different resources





- How to populate SD in sap.fhir.r4 in package registry?
- Will DDL connection.commit() will commit the normal connection changes as well?
  - No




- Meta capability statement regeneration is calling session.save() -> session is not reused

- need to remove the unneccesory columns

  - instead fill null values

- change the upser to insert in p2rm table api

  - done

- during rollback meta-svc should rollback the capability statement

- How to handle capability statement in package rollback?

- Confirm with nikita -> SP is not inserted to bounded-cache table in conformance

- Onboarding service seems to register the tenant with type PaaS, for packaging -> how it is getting registered ? [here](https://github.wdf.sap.corp/sap-health/fhir-onboarding-svc/blob/63064d7067f6270d331d9fb54b22fadb00ba829c/svc/src/main/java/com/sap/health/fhir/onboarding/services/provision/PackageOrchestratorService.java#L178)

- Should SD event handled in other svcs like plugin

- snapshot in Inline snapshot is not getting generated for SAPCustomResource1 in meta-svc

- Capability statement is not updated in core-svc

- After de-provisioning setting the is_default to the previous value

- Enabling package name & package version in bounded cache queries

- provisioning queues are exclusive (provisioning-svc)

- If a hl7 instance is created with a `package name` & `package version` how will we provision another package

  

- Different event listener for de-provisioning event / Same listener with two topic subscriptions

| Same listener                                                | Different listener                    |
| ------------------------------------------------------------ | ------------------------------------- |
| Same Queue                                                   | Different queues                      |
| Code reuse (if check for topic name)                         | Need to create another listener       |
| Code changes in EMS library to populate the topic name in the event properties | No code changes in EMS library        |
| Bounded context will have 2 consumers                        | Bounded context will have 3 consumers |
| Parallel processing depends on the number of service instances | Parallel processing                   |



Conformance

------------

P2MTable

![image-20220220184040582](/Users/I353767/Library/Application Support/typora-user-images/image-20220220184040582.png)

StructureDefinition Table

SQL

```sql
SELECT
	"INT_ID",
	"INT_VERSION_ID",
	"INT_VALID_FROM",
	"URL",
	"VERSION",
	"NAME",
	"KIND",
	"TYPE"
FROM "4D4B980521F04489A519EDCB1C13A388"."com.sap.health::FHIR_STRUCTUREDEFINITION"
WHERE "INT_ID" in ('account-city',
'custom-resource-encounter',
'custom-resource-chargeItem',
'sap-patient-birthTime',
'CustomResource603606',
'CustomResource',
'SAPCustomResource1',
'SAPPatient',
'SAPCustomResource2',
'SAPAccount');
```

![image-20220221104517286](/Users/I353767/Library/Application Support/typora-user-images/image-20220221104517286.png)



SearchParameter

```sql
SELECT TOP 1000
	"INT_ID",
	"INT_VERSION_ID",
	"INT_VALID_FROM",
	"URL",
	"VERSION",
	"NAME"
FROM "4D4B980521F04489A519EDCB1C13A388"."com.sap.health::FHIR_SEARCHPARAMETER"
where "INT_ID" in ('birthtime-sp-rvn17',
'encounter-sp-2seaj')
```

![image-20220220201600686](/Users/I353767/Library/Application Support/typora-user-images/image-20220220201600686.png)

ValueSet

```sql
SELECT TOP 1000 
	"INT_ID",
	"INT_VALID_FROM",
	"URL",
	"VERSION",
	"STATUS"
FROM "4D4B980521F04489A519EDCB1C13A388"."com.sap.health::FHIR_VALUESET"
WHERE "INT_ID" = 'f79938a3-c856-485e-825f-24a49748b451';
```

![image-20220220202248398](/Users/I353767/Library/Application Support/typora-user-images/image-20220220202248398.png)



Core

--------

P2RMTable

```sql
SELECT TOP 1000
	"PACKAGE_NAME",
	"PACKAGE_VERSION",
	"CANONICAL_URL",
	"CANONICAL_VERSION",
	"LOGICAL_ID",
	"CURRENT_META_VERSION",
	"RESOURCE_TYPE",
	"IS_EXISTING"
FROM "3BA7198D63AF43BAB92588BCE222D2CA"."com.sap.health::FHIR_SAP_HEALTH_PACKAGE_RESOURCE"
```

![image-20220220234410923](/Users/I353767/Library/Application Support/typora-user-images/image-20220220234410923.png)

ValueSet

![image-20220220235233554](/Users/I353767/Library/Application Support/typora-user-images/image-20220220235233554.png)

Bounded_Cache

```sql
SELECT
	"PACKAGE_NAME",
	"PACKAGE_VERSION",
	"ID",
	"URL",
	"RESOURCE_TYPE",
	"TARGET_RESOURCE_TYPE",
	"CREATED_TIME",
	"CONTENT",
	"TABLEMAP",
	"TERMINOLOGY_BINDING",
	"SEARCH_PARAMETERS",
	"IS_LAST_ACTIVE"
FROM "3BA7198D63AF43BAB92588BCE222D2CA"."com.sap.health::FHIR_BOUNDED_CACHE"
where "URL" in ('https://coophir.cfapps.sap.hana.ondemand.com/fhir/R4/StructureDefinition/CustomResource603606',
'http://sap.com/healthcare/PatActg/fhir/StructureDefinition/CustomResource',
'http://sap.com/healthcare/PatActg/fhir/StructureDefinition/SAPCustomResource1',
'http://sap.com/healthcare/PatActg/fhir/StructureDefinition/SAPPatient',
'http://sap.com/healthcare/PatActg/fhir/StructureDefinition/SAPCustomResource2',
'http://sap.com/healthcare/PatActg/fhir/StructureDefinition/SAPAccount')
```



![image-20220221101821986](/Users/I353767/Library/Application Support/typora-user-images/image-20220221101821986.png)

- birthtime-sp is not inserted in patient

SearchParameter

- while deleting not getting all the resources in the baseDefinition

Device

![image-20220221103356719](/Users/I353767/Library/Application Support/typora-user-images/image-20220221103356719.png)

![image-20220224173712972](/Users/I353767/Library/Application Support/typora-user-images/image-20220224173712972.png)

![image-20220321234420715](/Users/I353767/Library/Application Support/typora-user-images/image-20220321234420715.png)

## Sync on FHIR Package - Plugin Part

- https://github.wdf.sap.corp/sap-health/fhir-package-server/blob/master/docs/fhir-packages.md#key-design-decisions-open-questions-and-challenges-top
- Fhir Content(Packages)
  - Bring new extension on FS specific extension point 
  - Bring new ExtensionPoint & specific extension(s) 
- Since creation of extnpt and ext are Operations – bring it as bundle – separate entry point 
- CS to be updated accordingly
- In case we go with only packages : then update core-cache with the extensions which core is responsible for, so that remote calls can avoided for discover (TBD)
- Same instance two different end points to be avoided 
- Check :
  - Lifecycle of plugin regn opns – packages in pic, partner content : No Usecase as part of package content (AI : )
    - Default – sap provided
    - Partner : security issue
- Tasks :
  - Plugin-svc
    - Document : packages way to register extnPT and extn , CS update – document the flow as well (AI : )
    - Registration opn to support via bundle : all valdns needs to be handled even if it is via bundle : separate entry point (TBD) like 2 
- consider operation extension ...which might come from application team . like 1

## Design Discussion on Order of Processing of Bundle Resources

![image-20220323130114354](/Users/I353767/Library/Application Support/typora-user-images/image-20220323130114354.png)





![image-20220325153134162](/Users/I353767/Library/Application Support/typora-user-images/image-20220325153134162.png)



![image-20220325153617652](/Users/I353767/Library/Application Support/typora-user-images/image-20220325153617652.png)





https://github.wdf.sap.corp/sap-health/pa-content/blob/task/SPAC-4075-application-packaging/package/package.json#L9



## Sync - Package

- Sushi ?
- Simplifier package --> fhir cache ?
- final 4.0.0 version ?

| resourceType          | context                                          |
| --------------------- | ------------------------------------------------ |
| CodeSystem            | "core", "conformance", "authorization", "plugin" |
| ValueSet              | "core", "conformance", "authorization", "plugin" |
| GroupPolicy           | "authorization"                                  |
| RolePermission        | "authorization"                                  |
| OperationDefinition   | "conformance"                                    |
| StructureMap          | "conformance"                                    |
| CompartmentDefinition | "conformance"                                    |
| MessageDefinition     | "conformance"                                    |
| ImplementationGuide   | "conformance"                                    |
| GraphDefinition       | "conformance"                                    |
| RuleSet               | "core"                                           |





- Need to enhance the `DefaultResourceType` once the `CapabilityStatement` validator is implemented --> insert to `BoundedCache`
- `CompartmentDefinition` , `MessageDefinition`
- `OperationDefinition` --> handle in the current PR

BC -> iden -> Org -> iden



SD kind == "logical"

in case of meta ...

- do profile validation against SAPStrutureDefinition ... as this is custom resource and it should conform to SAPStructureDefinition
- meta svc will not add the entry for this in CS for the said svc

in case of core

- dump into BoundedContext of core
- update the CS for core in bounded cache







![image-20220428130243900](/Users/I353767/Library/Application Support/typora-user-images/image-20220428130243900.png)	































## Validation

https://github.wdf.sap.corp/sap-health/fhir-server-framework/compare/task/package_validation

https://github.wdf.sap.corp/sap-health/fhir-core-svc/compare/task/package_validation

https://github.wdf.sap.corp/sap-health/fhir-authorization-svc/compare/task/package_validation































![image-20220502151830448](/Users/I353767/Library/Application Support/typora-user-images/image-20220502151830448.png)

- Sense of ownership 
  - the content should be modified by the owner MyActivityAllocation
  - Producer and consumer
  - no shared ownership
  - dependency tree
- why package name and version is required
  - in a customer instance the whole set of package got deployed
    - to check in this instance which are all the artifacts got deployed
    - artifact is coming from which package and who is the owner
    - which version of that package has brought the data
    - enables organisation also
- How to transport the package from provider to the consumer
  - partner is also a provider
    - FS + localized partner is building solution for customer
    - PA is global 
      - once released what are the changes needs to be done german specific version can be developed
      - Hospitals needs to run the application based on their local contry specific regulations
        - partners are the experts so they will build on top of the PA
- FHIR Package provisioning
  - validation --> write (create/update)
