# APIs

Das Kibitz Backend-System besteht aus mehreren Diensten, die jeweils eigenständige JSON-RPC APIs zur Verfügung stellen:

* Eine **Appointments-API**, die verschlüsselte Kontaktdaten speichert, Hashes publiziert und Besuchsdaten entgegennimmt.
* Eine **Storage-API**, die (optional) verschlüsselte Einstellungen von Nutzern und Anbietern speichert.
* Eine **Notifications-API**, die optional Benachrichtigungen an Nutzer verschickt.

Die folgenden Abschnitte beschreiben die von den APIs angebotenen Methoden.

## Appointments-API

Die Appointments-API verfügt sowohl über unauthentifizierte als auch authentifizierte Endpunkte.

### Nutzer-Endpunkte

Diese Endpunkte sind lediglich für Nutzer vorgesehen.

* `getToken(hash bytes<32>) -> (enum<OK, ERR>, {OK: {token bytes<32>, signature bytes<32>}, ERR: Error})`: Liefert ein für den gegebenen `hash` signiertes Prioritätstoken zurück.
* `submitToken(token bytes<32>, encryptedData bytes<10,1024>, queueId bytes<32>) -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Reiht ein Prioritätstoken zusammen mit verschlüsselten Daten in eine gegebene Einzugsliste ein.
* `retractToken(token bytes<32>, queueId bytes<32>) -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Zieht ein Prioritätstoken zurück.

### Nutzer-Anbieter Endpunkte

Diese Endpunkte sind für Anbieter und Nutzer vorgesehen.

* `getMessages(id bytes<32>) -> (enum<OK, ERR>, {OK: list<EncryptedMessage>, ERR: Error})`: Liefert die unter einer gegebenen `id` für den Nutzer gespeicherten Nachrichten zurück.
* `deleteMessages(id bytes<32>) -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Löscht alle unter der gegebenen `id` vorliegenden Nachrichten.
* `postMessage(id bytes<32>, message EncryptedMessage) -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Speichert eine Nachricht unter der gegebenen `id`. Lediglich Anbieter können Nachrichten unter eine bis dato unbekannten `id` speichern.

### Anbieter-Endpunkte

Die folgenden Endpunkte sind für Anbieter vorgesehen und nur durch diese authentifiziert nutzbar.

* `storeProviderData(data ProviderData) -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Speichert Anbieterdaten zur Verifikation durch einen Vermittler.
* `deleteProviderData()  -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Löscht Anbieterdaten.
* `markTokenAsUsed(token bytes<32>)`: Markiert ein gegebenes Prioritätstoken als genutzt. Es kann dann nicht mehr genutzt werden.

### Vermittler-Endpunkte

Die folgenden Endpunkte sind für Vermittler vorgesehen und nur durch diese authentifiziert nutzbar.

* `storeVerifiedProviderData(data SignedProviderData)  -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Speichert verifizierte, signierte Anbieterdaten.
* `deleteVerifiedProviderData(id bytes<32>)  -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Löscht verifizierte Anbieterdaten.
* `storeQueueData(data SignedQueueData)  -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Speichert signierte Daten zu einer Einzugsliste.
* `deleteQueueData(id bytes<32>)  -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Löscht Daten zu einer Einzugsliste.

## Storage-API

Die Storage-API verfügt nur über unauthentifizierte Endpunkte.

* `storeSettings(id bytes<32>, data bytes<1,200kB>) -> enum<OK, ERR>`: Erlaubt das Speichern von verschlüsselten Nutzer-Einstellungen im Rahmen der Initialisierung.
* `getSettings(id bytes<32>) -> (enum<OK, ERR>, {OK: bytes<1,8192>, ERR: Error})`: Erlaubt das Abrufen von verschlüsselten Benutzer-Einstellungen.

## Notifications-API

Die Notifications-API verfügt nur über authentifizierte Endpunkte.

* `notify(recipient EncryptedRecipient, messageType string) -> (enum<OK, ERR>, {OK: nil, ERR: Error})`: Erlaubt die Speicherung von Besuchsdaten durch einen Betreiber.
