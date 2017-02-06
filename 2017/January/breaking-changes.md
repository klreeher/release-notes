[All Releases](../../README.md) / [2017](../README.md) / [January](README.md) / breaking changes release

# API Breaking Changes Release Candidate Notes 

Planned to be released to Staging environment TBD. 

This public staging environment will be available for 30 days. Production data will be copied down to the Staging environment once, on TBD. In Staging, all webhooks will be disabled initially. Please update your webhooks and integrations in Staging to point somewhere other than Production ASAP.

The planned release to Production is TBD. _This date is subject to change_

## New Features
- Payments have a new boolean field, `Accepted`. Only users with the new `ProcessPayments` role will be able to create or update payments with `Accepted` set to `true`, and create Payment Transactions. 
- Order submit logic will validate `Payment.Accepted=true` and orders without an accepted payment will fail with **ERROR CODE**.
- Previously, any admin user could impersonate any buyer user. Going forward, **MORE DETAILS HERE**.
- Due to refactoring around our password hash algorithm, and since we do not store users' passwords ourselves, but simply a hash of the password, **users will need to reset their passwords through the normal password reset process before they can log into the OrderCloud devcenter or any OrderCloud apps**.
- Added roles that control who can list or edit shipments. Now users with `ShipmentAdmin` *or* `OrderAdmin` can create or edit shipments. Users with `ShipmentReader` or `OrderReader` can get/list shipments.
- You can now filter product lists based on `CategoryID`. 
- An order that requires approval can now be sent back to the submitting user by the approver user for editing and re-submission. **MORE DETAILS HERE**
- We are changing the route to register an anonymous user (previously called "Create From Temp User") to `PUT` `v1/me/temp`. This will help make our Swagger spec more flexible.
- Any buyer user can now list shipments for their own orders in a User Perspective route, `me/shipments`. No more need for elevated permissions!
- We are disallowing several user-level assignments. These type of assignments tend to be a short-sighted solution that result in a lot of copying when new users are added. You will no longer be able to assign the following at the user-level:
    + Products
    + Categories
    + Promotions
    + Cost Centers
    + Message Config
**If there are existing user-level assignments that you need for any of the above, convert them to something else before the production release date**.

- We have added a sub-object to Approvals to list Approving User information.
Example:
>`{
"ApprovalRuleID": null,
"ApprovingGroupID": null,
"Status": "Pending",
"DateCreated": null,
"DateCompleted": null,
"ApprovingUser:
{ "ApproverID": null, "ApproverFirstName": null, "ApproverLastName": null, "ApproverUserName": null, "ApproverEmail": null }
"Comments": null
}`
- We have also moved approval comments out of the URL query string and into the request body. There is a maximum length of 2000 characters.


### Inventory Revamp
- Inventory is now a sub-object on the Product model.

#### Summary of Inventory Object Changes:

|            Old Inventory Object           |    New Product.Inventory Sub-Object    |
|-------------------------------------------|----------------------------------------|
| Product.InventoryEnabled                  | Product.Inventory.Enabled              |
| Product.InventoryNotificationPoint        | Product.Inventory.NotificationPoint    |
| Product.VarientLevelInventory             | Product.Inventory.VariantLevelTracking |
| Product.AllowOrderExceedInventory         | Product.Inventory.OrderCanExceed       |
| Product.InventoryVisible                  | *gone!*                                |
| `/products/:id/inventory`                 | *gone!*                                |
| Inventory.ID                              | *gone!*                                |
| Inventory.Name                            | *gone!*                                |
| number posted to `/product/:id/inventory` | Product.Inventory.Quantity             |
| Inventory.Available                       | Product.Inventory.AvailableQuantity    |
| Inventory.Reserved                        | Product.Inventory.ReservedQuantity     |
| Inventory.LastUpdated                     | Product.Inventory.LastUpdated          |
| `/products/:id/inventory`                 | *gone!*                                |

#### Summary of Inventory Action Changes:

|                  Action                 |  Quantity | ReservedQuantity | Available Quantity |
|-----------------------------------------|-----------|------------------|--------------------|
| Add Item To Order                       | No Change | Increases        | Decreases          |
| Submit, Submit for Approval, or Approve | No Change | No Change        | No Change          |
| Ship Item                               | Decreases | Decreases        | No Change          |
| Cancel or Decline Shipped Order         | No Change | Decreases        | Increases          |
| Remove Unshipped Line Item              | No Change | Decreases        | Increases          |
| Manually Increase Inventory             | Increases | No Change        | Increases          |

- **Quantity** is writeable and represents the total amount physically warehoused by the seller.
- **ReservedQuantity** is read-only and represents the total unshipped amount on live orders (submitted or unsubmitted).
- **AvailableQuantity** is read-only and is the difference Quantity and ReservedQuantity. It is usually the number you want to display to the buyer, and is the number checked when enforcing that orders cannot exceed inventory.

### Shipment Changes
- We have removed Shipment.Items, and shipped items are instead retrieved or updated via new endpoints, much like line items.
- BuyerID has been removed from routes, meaning you can list shipments across multiple buyers. 
- Shipment IDs are now seller-unique.
- Many of the new fields on Shipment and ShipmentItem are derived from LineItems, to increase performance on lookup.

|                    New Shipment Object                    |                     Notes                     |
|-----------------------------------------------------------|-----------------------------------------------|
| Shipment.ShippingAccount                                  | writeable, default derived from LineItems [1] |
| Shipment.ShippingAddressID                                | writeable, default derived from LineItems [1] |
| Shipment.ShippingAddressID                                | writeable, default derived from LineItems [1] |
| Shipment.ShipFromAddressID                                | read-only, derived from LineItems             |
| Shipment.ShippingAddress                                  | read-only, derived from LineItems             |
| Shipment.ShipFromAddress                                  | read-only, derived from LineItems             |
| ShipmentItem.UnitPrice                                    | read-only, derived from LineItems             |
| ShipmentItem.CostCenter                                   | read-only, derived from LineItems             |
| ShipmentItem.DateNeeded                                   | read-only, derived from LineItems             |
| ShipmentItem.Product                                      | read-only, derived from LineItems             |
| ShipmentItem.Specs                                        | read-only, derived from LineItems             |
| ShipmentItem.xp                                           | read-only, derived from LineItems             |
| `GET` `v1/shipments`                                      |                                               |
| `GET` `v1/shipments/{id}`                                 |                                               |
| `GET` `v1/shipments/{id}/items                          ` |                                               |
| `GET` `v1/shipments/{id}/items/{orderID}/{lineItemID}   ` |                                               |
| `POST` `v1/shipments/{id}/items                         ` |                                               |
| `PATCH` `v1/shipments/{id}/items/{orderID}/{lineItemID}`  |                                               |

[1] Updating these on the Shipment will update the underlying LineItem as well.

### Simplified Product and Category Assignments

|            New Properties           |                   Notes                    |
|-------------------------------------|--------------------------------------------|
| Catalog.Active                      |                                            |
| CatalogAssignment.ViewAllCategories |                                            |
| CatalogAssignment.ViewAllProducts   |                                            |
|-------------------------------------|--------------------------------------------|
| CategoryAssignment.Visible          | Nullable, inherited from parent or catalog |
| CategoryAssignment.ViewAllProducts  | Nullable, inherited from parent or catalog |
|-------------------------------------|--------------------------------------------|
| Product.DefaultPriceScheduleId      | Optional, but encouraged.                    |

- For a Buyer User to see a Product in the User Perspective (`GET` `/me/products`), the following must *all* be true:
    * `Product.Active` = `true`
    * Catalog exists where:
        + `Catalog.Active` = `true`
        + `Buyer` is assigned via `CatalogAssignment`
        + `Product` is assigned via `ProductCatalogAssignment`
- For a Buyer User to see a Product in the User Perspective (`GET` `/me/products`), *one* of the following must be true:
    * `CatalogAssignment.ViewAllProducts` = `true`, **OR**
    * Category exists in Catalog where:
        + `Active` = `true`
        + Product assigned via `CategoryProductAssignment`
        + `CategoryAssignment` exists where:
            - `Buyer`, `User`, or `Group` matches the user
            - `ViewAllProducts` = true for first non-null setting up the tree
        + `ProductAssignment` exists for Buyer, User, or any Group the user belongs to. (`PriceScheduleID` not required)



## Bug Fixes


## Client Libraries
- Save Security Profile assignment parameter order changes
- We've updated our terminology around claims vs roles to consistently use roles. You may need to check your token requests and make sure that you've updated your code to use roles.
- `MesageSenders` is now correctly spelled as `MessageSenders` in the SDK.