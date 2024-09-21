`back-connectivity/clients/touropapi/operation_post.go`

## `postInternal`
### Context Setup and Validation

* The function `postInternal` takes in a `context.Context` object (`ctx`) and uses it to retrieve the user's email.
* It also takes in a `mongo.Client` object (`client`), database name (`db`), collection name (`collection`), and raw input JSON data (`rawInputObj`).
* The function checks if the `_id` field is present in the `objBeforeHook` map, which is created by unmarshaling the raw input JSON data using `bson.UnmarshalExtJSON`.
* If the `_id` field is not present, it calls the `validate` function to validate the input data.

### Pre-Update Hooks for Tourop Collections

* The function checks if the database name and collection name match specific Tourop collections (PropertyReferenceCollection, JoiningInstructionCollection, ActivityCollection, PlaceCollection, PropertyCollection, SaleCollection).
* If a match is found, it calls a corresponding pre-update hook function to modify the raw input JSON data before updating the MongoDB collection.
* The pre-update hook functions are:
	+ `touropPropertyReferencePreUpdateHook`
	+ `touropJoiningInstructionPreUpdateHook`
	+ `touropActivityPreUpdateHook`
	+ `touropPlacePreUpdateHook`
	+ `touropPropertyPreUpdateHook`
	+ `postTouropSalePreHook`

### Document Update and Insert Logic

* The function checks if the `_id` field is present in the `obj` map. If it is, it proceeds to update the document.
* It creates a `set` and `unset` BSON maps to store fields that need to be updated or unset.
* For each key-value pair in the `obj` map (excluding certain fields like `_id`, `_createdAtField`, `_createdByField`, `_lastUpdatedAtField`, and `_lastUpdatedByField`), it checks if the value is nil. If so, it adds the key to the `unset` map with a value of "". Otherwise, it adds the key-value pair to the `set` map.
* It then sets the values for `_lastUpdatedAtField` and `_lastUpdatedByField` based on the current timestamp and user email.
* A `patch` BSON map is created by combining the `set` and `unset` maps. If both are empty, it returns a FindOne result using the provided client, context, database, collection, and object ID.
* Otherwise, it calls the `FindOneAndUpdate` method on the MongoDB collection to update the document with the provided patch. It sets upsert to true to allow creation of a new document if one does not exist.
* If an error occurs during the update operation, it returns nil along with the wrapped error message.
* If the update operation is successful and no documents were found (i.e., ErrNoDocuments), it sets `isCreation` to true and proceeds to create a notification event for creation instead of update.
* It creates a new Tourop notify object with the action set to UPDATE or CREATE depending on whether a document was created or updated. If an error occurs while publishing the notification event, it logs the error.

### Document Insert Logic

* If the `_id` field is not present in the `obj` map, it generates a new ID using the `generateId` function.
* It then sets the values for `_createdAtField`, `_createdByField`, `_lastUpdatedAtField`, and `_lastUpdatedByField` based on the current timestamp and user email.
* It creates a notification event with the action set to CREATE.
* If an error occurs while inserting the document, it returns nil along with the wrapped error message. Otherwise, it returns the result of the InsertOne operation.

### Notification and Hook Logic

* The function creates a Tourop notify object with the action set to DUPLICATE if a duplicate ID was provided. Otherwise, it sets the action to CREATE.
* It publishes the notification event using the `PublishEvent` method from the Tourop package. If an error occurs while publishing the notification, it logs the error.

### Post-Update Hook Logic

* The function checks if the database and collection match specific values (Tourop entities).
* For each matching entity, it calls a corresponding post-patch hook function:
	+ `postPatchContractHook`: Called for contracts in the Tourop database.
	+ `postPatchTransferContractHook`: Called for transfer contracts in the Tourop database.
	+ `touropPropertyReferencePostUpdateHook`: Called for property references in the Tourop database.
	+ `postTouropSalePostHook`: Called for sales in the Tourop database.

### Return New Object

* The function returns the newly updated object from the FindOne operation, along with a nil error.

## `touropPropertyReferencePreUpdateHook`

### Property Reference Pre-Update Hook

* The function `touropPropertyReferencePreUpdateHook` is a pre-update hook for property references in the Tourop database.
* It takes in a `context.Context` object (`ctx`) and a `json.RawMessage` representing the raw input object.

### Data Unmarshalling

* The function unmarshals the raw input JSON into a `tourop_entities.PropertyReference` struct using `bson.UnmarshalExtJSON`.
* If an error occurs during unmarshalling, it returns a nil response with an error message.

### Building Places With Parent Field

* The function builds a list of places with parent fields by recursively traversing the place hierarchy.
* It starts from the given property reference's place ID and iteratively fetches its parent places until it reaches the top-level resort.
* If any errors occur during this process, it returns a nil response with an error message.

### Updating Property Reference

* The function updates the property reference by setting its resort name, place name, and places with parent fields based on the built list.
* It then calls the `UpsertPropertyReferenceHook` method from the Tourop service to update the property reference in the database.
* If an error occurs during this process, it returns a nil response with an error message.

### Returning Updated JSON

* The function checks if any updates were made to the property reference. If so, it returns the updated JSON representation of the property reference using `bson.MarshalExtJSON`.
* Otherwise, it returns the original raw input JSON.
