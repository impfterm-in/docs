# APIs

The Kibitz backend system consists of several services, each providing independent JSON-RPC APIs:

* An **Appointments API** that stores encrypted contact data, publishes hashes, and accepts visit data.
* A **storage API** that (optionally) stores encrypted user and provider settings.
* A **Notifications API** that optionally sends notifications to users.

The following sections describe the methods offered by the APIs.

## Appointments API

The Appointments API has both unauthenticated and authenticated endpoints.

### User endpoints

These endpoints are intended for users only.

* `getToken(hash bytes<32>) -> (enum<OK, ERR>, {OK: {token bytes<32>, signature bytes<32>}, ERR: Error})`: Returns a priority token signed for the given `hash`.
* `submitToken(token bytes<32>, encryptedData bytes<10,1024>, queueId bytes<32>) -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Enqueues a priority token along with encrypted data into a given move-in list.
* `retractToken(token bytes<32>, queueId bytes<32>) -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Withdraws a priority token.

### User-Provider Endpoints

These endpoints are intended for providers and users.

* `getMessages(id bytes<32>) -> (enum<OK, ERR>, {OK: list<EncryptedMessage>, ERR: Error})`: Returns the messages stored under a given `id` for the user.
* `deleteMessages(id bytes<32>) -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Deletes all messages present at the given `id`.
* `postMessage(id bytes<32>, message EncryptedMessage) -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Saves a message under the given `id`. Only providers can save messages under a previously unknown `id`.

### Provider endpoints

The following endpoints are intended for providers and can only be used authenticated by them.

* `storeProviderData(data ProviderData) -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Stores provider data for verification by an intermediary.
* `deleteProviderData()  -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Deletes provider data.
* `markTokenAsUsed(token bytes<32>)`: Marks a given priority token as used. It can then no longer be used.

### Intermediary endpoints

The following endpoints are intended for intermediaries and can only be used authenticated by them.

* `storeVerifiedProviderData(data SignedProviderData)  -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Stores verified, signed provider data.
* `deleteVerifiedProviderData(id bytes<32>)  -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Deletes verified provider data.
* `storeQueueData(data SignedQueueData)  -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Stores signed data for a move-in list.
* `deleteQueueData(id bytes<32>)  -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Deletes data for a move-in list.

## Storage API

The Storage API only has unauthenticated endpoints.

* `storeSettings(id bytes<32>, data bytes<1,200kB>) -> enum<OK, ERR>`: Allows encrypted user settings to be saved as part of the initialization process.
* `getSettings(id bytes<32>) -> (enum<OK, ERR>, {OK: bytes<1,8192>, ERR: Error})`: Allows retrieval of encrypted user settings.

## Notifications API

The Notifications API only has authenticated endpoints.

* `notify(recipient EncryptedRecipient, messageType string) -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Allows the storage of visit data by an operator.

