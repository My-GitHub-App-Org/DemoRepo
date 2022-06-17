# Eventing Solution 4
test
## Open Points test
test
- In EMS FITs currently not added tags for the ems credentials
- Test the SaaS FITs
- How the ems credentials are getting filled while running in the pipeline?
- Are we whitelisting the dynamic namespace for reuse ems instances?
  - yes. for prod we are overriding the namespace to be default/sap.health.fs/-test
- future scope
- mind map
- How the wrong mtaext is getting deployed test updated

Hello All,

Please find the EMS library changes for event publishing and consumption from the `provider event bus` here: https://github.wdf.sap.corp/sap-health/fhir-server-framework/pull/620

The following changes should be made to test the eventing scenario for PaaS tenants:

- Update the master fs EMS client (`sap-health-ems`) with this service tag: `sap-health-ems-fs`
- Bind the master fs EMS client (`sap-health-ems`) with the event producer `bounded-context` (e.g. core-svc/provisioning-svc)
- Bind the master fs EMS client (`sap-health-ems`) with the event consumer `bounded-context` (e.g. core-svc/provisioning-svc)
- Define the event listener in the consumer `bounded-context` with the type as `SapHealthEventListener.EventListenerType.PROVIDER_LISTENER` 
  - e.g.

```java
@SapHealthEventListener(name = "valid-listener", type = SapHealthEventListener.EventListenerType.PROVIDER_LISTENER, topics = {
        @Topic("/core/valid") })
public class ValidMessageListener implements ISapHealthEventReceiver {
```

- Use the `ISapHealthEventProducer.sendToProvider` api in the event producer `bounded-context` to publish the message
  - e.g.

```java
sapHealthEventProducer.sendToProvider("core/valid", SAPEvent, props, 50000, scpTenant);
```

## SaaS Testing

- How is the team 4 subaccount setup
  - Where is the FS deployed
  - Is there a PA application subscribed
  - How to execute postman requests
- deploy the FS to dev-team 4 dev space
- SAP PA (blr) application is subscribed from dev in the space saas-blr
  - ![image-20211001171524077](/Users/I353767/Documents/SAP Health/Eventing solution 4/image-20211001171524077.png)
- unsubscribe and resubscribe the application
- Note: just publishing app: SAP Health Sample FS F
- The deployed application for this subscription is in dev-ui (sub acc) -> blr (space)
  - Here update the EMS client binding in billing-engine-controller-svc
- For accessing the subscription UI need to assign the role collection FSDevRoleCollection
  - ![image-20211001171207618](/Users/I353767/Documents/SAP Health/Eventing solution 4/saas-ui-role-collection.png)
  - Here the Review_Automated_Billing_Viewer will be deleted while unsubscribing
    - Should be assigned again to view the UI
  - test

## Testing

- BECS having old FSF version and no service tag
  - Update the FS in dev-team-4 -> dev space
  - Perform the patient create and check the published topic for name space -> is BECS able to consume
  - Adopt the sendToPrivider for core-svc and check the published topic for namespace -> -> is BECS able to consume
- BECS having new FSF and a service tag
  - Update BECS in dev-ui -> blr
  - Re-subscription in saas-ui
  - Assign the missing role
  - Perform the patient create and check the published topic for name space -> is BECS able to consume
  - Adopt the sendToPrivider for core-svc and check the published topic for namespace -> -> is BECS able to consume
