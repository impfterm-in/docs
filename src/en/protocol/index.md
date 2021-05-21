# Protocol & System Design

This document describes a decentralized, privacy-friendly protocol & system for mediating appointments between two parties. It is intended to be used, among other things, for privacy-friendly mediation of vaccination appointments in the context of the Covid-19 pandemic.

## Stakeholders

There are the following players in the system:

* **Users** book appointments with **providers** (e.g. doctors).
* **Providers** make appointments available to **users**.
* **Mediators** create trust between the actors in the system.
* **Backend operators** run the IT infrastructure needed for communication between the players.

## Requirements

The privacy and security of users and providers should be protected as much as possible. Intermediaries and backend operators should not be able to obtain information about users or providers. They should also not be able to track whether appointments have been made between specific providers and users, or reveal details of these appointments. Intermediaries and backend operators should nevertheless be able to detect and prevent misuse of the system (if necessary with the support of users and providers). 

The system should also be as robust as possible against abuse by users and providers. It should be prevented that rogue providers can contact users via the system or view data from them, that users can resell appointments obtained via the system (arbitrage/scalping), or that actors can impair the functionality of the system for legitimate users or providers through spamming or other techniques. Compromise of individual system components or actors should have minimal impact on the privacy or security guarantees of the system.

## System components

The system consists of the following components:

* **User Frontend** : A web/desktop/mobile application used by users to register and book appointments. 
* **Provider Front End** : A web/desktop/mobile application used by providers to register and schedule appointments.
* **Intermediary Front End** : A web/desktop/mobile application used by intermediaries to verify providers, monitor the system, and curb abuse.
* **Data & authentication backends** : An API-based backend system that stores encrypted data of the above actors, authenticates providers and cryptographically signs specific data of users or providers. If necessary, backend services are split into several services/APIs for this purpose.

In the following, the term "app" is used to refer to the applications of individual players; this does not necessarily mean a mobile app.

## Protocol `v0.1`

The following sections describe the protocol for appointment switching and the system components used for this purpose (cryptographic operations and data flows are not described in full detail).

### Notes

For simplicity, the document refers to encryption using asymmetric (ECDH) key pairs. This refers in each case to a key derivation based on a (temporary or permanent) ECDH key pair in combination with a symmetric encryption method (e.g. AES-GCM). When talking about public or private keys for the encryption of data in the context of bidirectional communication (e.g. between providers and users), it is further implied that these keys are modified via a suitable key derivation procedure (e.g. a Diffie-Hellman ratchet) for each exchanged message.

### Abstract

Appointments are arranged in several predominantly automated and asynchronous steps:

* Intermediaries are registered by the backend operator.
* Providers register and request verification through intermediaries.
* Intermediaries verify providers and grant them access to the system.
* Users register and submit (anonymous) appointment requests for a given catchment area.
* Providers retrieve appointment requests and create appointment offers for specific users.
* Users select dates from the offers and book them.
* Providers verify user data and confirm booked appointments.
* Users and providers may change or cancel booked appointments.

All communication between providers and users is encrypted end-to-end, the backend system stores the encrypted data without assignment to specific providers or users and can therefore hardly derive metadata about them. If external systems (e-mail, SMS) are used for communication or verification, this is only done on an ad hoc basis with the cooperation of providers; the corresponding contact data (e-mail addresses, telephone numbers) are only accessible to users themselves and to the operators of external systems for specific purposes and on an ad hoc basis.

### 1. mediator initialization

Intermediaries act as a certificate authority (CA) in the system and create an ECDSA signature key pair and an associated certificate $C_\mathrm{auth}$, which is stored as a trust anchor (root certificate) in the backend. Furthermore, they create ECDH & ECDSA key pairs $(K_\mathrm{auth}^\mathrm{enc,priv}, K_\mathrm{auth}^\mathrm{enc,pub}), (K_\mathrm{auth}^\mathrm{sign,priv}, K_\mathrm{auth}^\mathrm{sign,pub})$, whose signed public keys are stored in the backend.

The agent app creates a random Salt value (32 bytes of random data) for each of the relevant catchment areas (e.g. based on postcodes) and saves the list of catchment areas with relevant data and the associated Salt values in the backend. This can be called up publicly from there. For each catchment area, the mediator app also creates an ECDH key pair $(K_\mathrm{r_i}^\mathrm{enc,priv}, K_\mathrm{r_i}^\mathrm{enc,pub})$. The private key $K_\mathrm{r_i}^\mathrm{enc,priv}$ is encrypted by the backend with the public key of relevant providers (see below) during provider initialization and stored for them in the backend. The public key is published together with the catchment area data.

### 2. backend initialization

Backends have a randomly generated key (32 bytes of random data) with the aid of which priority tokens are derived using a suitable key derivation procedure (e.g. HKDF), which are used in the further process for priority determination. This key must be kept secret in order to ensure the fairness of the scheduling process. Backends also have signature key pairs $(K_\mathrm{be}^\mathrm{sign,priv}, K_\mathrm{be}^\mathrm{sign,pub})$, which are used to sign priority tokens.

### 3. provider initialization

Providers first authenticate themselves to the backend and thereby gain access to the provider app. There, they enter relevant data about their practice as well as information they would like to display to users when scheduling appointments. The provider app generates an ECDH key pair $(K_\mathrm{op_i}^\mathrm{enc,priv},K_\mathrm{op_i}^\mathrm{enc,pub})$, encrypts the entered data with the public provider key and uploads it to the backend for verification, linked to a random ID (32 bytes of random data). Intermediaries retrieve the data from there, decrypt and verify it, and sign successfully verified data with their private signing key $K_\mathrm{auth}^\mathrm{sign,priv}$. Additionally, they sign the public key $K_\mathrm{op_i}^\mathrm{enc,pub}$ of the provider. The mediator app again encrypts this data with the provider key and stores it under the given ID in the backend. The signed provider public key is additionally published via the backend, and the associated provider data is encrypted with a randomly generated symmetric key from the provider app and also stored there (this allows intermediaries to control providers later). The provider app downloads the signed data from the backend and decrypts it.

Providers now create appointments in their app, which are saved locally in the app. This completes the initialization on the provider side. Ideally, the app must be opened continuously, or at least regularly, for appointment scheduling.

### 4. user initialization

Users open the user app, which can be used without authentication. They enter relevant personal data (name, address, e-mail and telephone number, if applicable). The user app generates an ECDH key pair $(K_\mathrm{u_i}^\mathrm{enc,priv},K_\mathrm{u_i}^\mathrm{enc,pub})$ as well as a nonce value (32 bytes of random data), links this with the specified data and creates a hash value $H_i$ from it. This value is sent to the authentication backend, which generates a priority token $P_i$ (32-byte random data) and signs it together with the hash value $H_i$ from the backend: $S_i^{PH} = \mathrm{sign}((H_i, P_i),K_\mathrm{be}^\mathrm{sign,priv})$. The signature links the priority token with the data provided by users (without knowing it) and thus prevents users from reselling the token.

Users now select a catchment area and create an anonymous appointment request for it. If necessary, this is provided with additional information (e.g. desired vaccine). The data of the appointment request is encrypted by the user app together with a randomly generated ID (32-byte random data) with the public key of the catchment area and uploaded to the backend together with the priority token. Here, the backend ensures that priority tokens cannot be uploaded multiple times. This completes the user initialization.

### 5. create a time slot offer

The appointment requests uploaded by users in catchment areas can be sorted by the backend according to the priority tokens specified. This creates a dynamic, prioritized list of users for a given catchment area. Provider apps request a small number of user requests from each of these lists. They can decrypt these using the private key of the catchment area (which is provided to them in encrypted form via the backend). They check the appointment request against available capacity, create appointment offers, encrypt them along with the signed practice data using the user's public key, and store the data in the backend under the ID provided by the user app. The data is additionally signed with the private signature key. Each time slot is assigned a random ID.

The provider app signals the specific intent to the backend before creating an appointment offer. The backend associates the requests with a priority token and removes this token from the priority list after a given number of received offers (temporarily at first). This prevents users who are not responsive from receiving a very large number of appointment offers, while users with lower priority tokens do not receive any offers.

Optionally, the app generates a notification for the user using an external service (e.g. in the form of an email notification). This requires the user to provide such contact details in encrypted form in the appointment request (see below).

The provider app can assign appointments multiple times in order to achieve a sufficient occupancy of appointments. User apps can dynamically detect appointments that have already been allocated and, if necessary, request a new appointment allocation if all offers have already been allocated. The optimal overbooking rate depends on various factors and must be dynamically optimized in the system.

### 6. accept appointment offer (user)

The user app regularly checks whether data is available in the backend under the ID created in step 4 (the user may also receive a notification about this via an external service). If so, the app retrieves this data and decrypts it with the private user key. The user app then checks the signatures (provider & agent signature) of all offers and removes invalid offers. The valid dates are displayed to users for selection. They select a suitable one from the dates. The user app now encrypts the user data specified during initialization and the signed priority token with the public key of the provider and uploads the encrypted data with the ID specified in the offer to the backend. A check of the IDs still available can be carried out in advance in the backend (since time slots can be sent to several users in parallel).

### 7. confirm time slot offer (supplier)

The provider app regularly checks whether data is available in the backend under the IDs created in step 5. If so, the app retrieves this data and decrypts it with the private key generated in step 5. Then, the app checks whether the specified priority token matches the originally specified token. Additionally, it checks whether the signed hash value matches the hash value of the user data calculated by the app. If necessary, providers can still manually check the specified user data or automatically request verification via an external service (see below). The app then creates an appointment confirmation, encrypts it with the user's public key, and stores it in the backend under the previous ID. Optionally, the app generates a notification for the user using an external service (e.g. in the form of an email notification).

This concludes the appointment process.

### 8. cancel or change appointments

Users can cancel appointments via the app by sending an encrypted request to the provider under the known ID. The provider app can then generate a new appointment offer. In principle, other communication options are also conceivable via this encrypted channel; the functionality can be easily adapted to the use case for this purpose.

### Extensions

The following extensions to the protocol are conceivable and can increase usability and security.

#### Storing key material

Since users, providers and intermediaries each require key material in order to use the system, it is important to store this material in a fail-safe manner. This can be done, for example, by storing it as a file or via encrypted storage in a backend. In the latter case, a randomly generated token or user-selected password can be used to encrypt the data locally before uploading. The password/token can be written down by users or stored in the form of a bookmark. If a user-selected password is used, a randomly generated ID should also be chosen to uniquely identify the data. In the case of a random token, a password as well as an ID can be generated from it using a key derivation. 

#### Use of external services for verification & notification of users

The verification of data (e.g. e-mail addresses or telephone numbers) can be an effective instrument to combat misuse. However, such verification is not privacy-friendly and is often time-consuming and costly. It should therefore only be done on an ad hoc basis and as late in the process as possible. Notification of users via external services (e.g. via e-mail) is also often useful, but for the same reasons should also only be carried out on an ad hoc basis.

To enable such services, system components can be created which, for example, implement e-mail or SMS transmission or reception. These should only be able to be used on an occasion-related and authenticated basis. For this purpose, the services can each receive their own ECDH key pairs whose public keys are made known to the user app. User data such as e-mail addresses or telephone numbers can then be encrypted in the user app with the public key of the respective service and added to an appointment request. Providers can then use the encrypted data and an additional authentication token to request verification or notification via the corresponding external service. This service can decrypt the user data with the private key and generate a corresponding e-mail or SMS, for example. This ensures that providers do not have access to the decrypted data at any time and that this data can only be made available to external services on an ad hoc basis and with the cooperation of users. These services can be operated independently of the rest of the system. Since verification only takes place on an ad hoc basis and, if necessary, after an initial check of the data by providers, the volume of external services used can be greatly reduced, which in turn reduces costs and effort accordingly. The potential for misuse is also significantly reduced by the purely authenticated usability of the services.

Contact data for use in external services can also be encrypted multiple times to increase the purpose limitation. For example, e-mail addresses or telephone numbers can be additionally encrypted with the public key of the catchment area in order to be decryptable only for providers from this area.

#### Design of catchment areas

Catchment areas can be structured not only geographically but also according to other criteria. For example, it is conceivable to define separate catchment areas for individual vaccines or age groups. This can improve appointment finding and reduce the number of data records that have to be viewed by providers. Providers can then also be assigned to several catchment areas if necessary (e.g. if they offer vaccinations with several vaccines).

#### Recording and combating abuse

From both the user and the provider side, there are several ways to abuse the system:

* Actors may be able to register as providers in the system and send appointment offers to users. If users accept these offers and make their data available to the actors, they may be misused.
* Actors can generate a large number of priority tokens and use them to compromise the functionality of the system. They can also try to resell priority tokens.
* Actors may try to overwhelm the backend system by issuing a large number of queries.

To prevent these abuse scenarios, the system should provide countermeasures. For example, users should be able to report providers registered in the system that they do not consider reputable. Intermediaries can then remove such providers from the system by depublishing their public signature key, user apps will then automatically remove all appointment offers of these providers when loading the offers. However, such a "reporting" function can also be misused, so it should also be subject to restrictions and implement appropriate security measures.

Several countermeasures are conceivable to prevent the system from being overwhelmed by a high number of abusively created priority tokens. On the one hand, the overbooking rate of appointments can be adjusted to the number of abusively created tokens. In addition, individual user data can be verified. For this purpose, data such as an e-mail address or a telephone number are verified with the help of an external service (see above). This service then generates a signature for the specified data, which can additionally be linked to context information such as a priority token. Providers can thus sort out user requests that do not have verified contact data or require the user to verify the data again. Again, external systems can be used for this purpose. This can increase the hurdle for manipulation of the system by malicious users.

To avoid overwhelming the system with a high number of backend requests, the backend must be designed to withstand high request volumes. Since authentication of users is generally not desirable (since this results in a large number of possibilities for metadata analysis), it must be ensured that users can only make requests with context information if possible. This is possible in the given system because providers can be authenticated more easily and they create ID values on their own initiative, which users can then use to send data to the backend. Users can thus only store data in the backend knowing these values, and the storage of data under these IDs can be limited via simple mechanisms. Only for the storage of encrypted settings/user data this is not possible, but this backend system can be operated separately from the rest of the system and can easily be designed for large data volumes due to the simple nature of storage.

#### Decentralized & federated operation

In principle, all components of the system can be operated redundantly and federated. Apps can communicate with multiple backends and retrieve data from them or send data to them. Likewise, several intermediaries can use one backend to register providers and catchment areas. Thus, it is conceivable to assign the registration and monitoring of providers to regionally specific actors. Similarly, separate backends can be operated for different regions, or multiple backends can be redundantly combined into a more fail-safe system. Apps can also be distributed decentrally and independently of the backend, so that they can also be adapted to regional conditions, for example.
