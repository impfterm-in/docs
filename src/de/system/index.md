# Überblick

Das Kiebitz System besteht aus mehreren Komponenten, die unabhängig voneinander und ohne zentrale Koordination eingesetzt werden können. Gemeinsam bilden sie ein föderiertes, dezentrales System zur Terminvermittlung.

Das Backend besteht aus drei verschiedenen Diensten:

* **Appointment-Server** speichern Prioritätstoken von Nutzern in Listen von Einzugsgebieten, geben diese an Anbieter weiter und vermitteln die verschlüsselte Kommunikation zwischen Anbietern und Nutzern.
* **Storage-Server** speichern optional verschlüsselte Einstellungen von Nutzern und Anbietern.
* **Notifications-Server** senden optional Benachrichtungen an Nutzer (z.B. per E-Mail)

Zudem verfügt das System über mehrere Client-Anwendungen:

* Die **Nutzer-Anwendung** erlaubt die Initialisierung der Kontaktnachverfolgung für Nutzer.
* Die **Betreiber-Anwendung** erlaubt die Erfassung von Besuchsdaten durch Betreiber sowie die Übermittlung der Daten an einen oder mehrere **Betreiber-Server**.
* Die **Scanner-Anwendung** erlaubt die Erfassung von Besuchsdaten auf mobilen Endgeräten. Sie tausch über die **Operator-API** mit der **Betreiber-Anwendung** Daten aus.
* Die **Gesundheitsamt-Anwendung** erlaubt die Anfrage von Kontaktdaten durch Gesundheitsämter, den Abruf zurückgelieferter Daten sowie die Entschlüsselung und Weiterverarbeitung dieser Daten.

## Systemkonzept

Die Gestaltung der Komponenten zielt darauf ab, ein System zu erhalten das **redundant**, **dezentral**, **föderiert**, **resilient**, **sicher** und **privatsphäre-freundlich** ist. Dies wird über mehrere Aspekte erreicht:

### Dezentralität

Es gibt im Kiebitz System prinzipiell keine zentrale Stelle, die den Betrieb des Gesamtsystems kontrolliert. Alle Server-Typen können unabhängig betrieben werden. Die funktionale Einbindung erfolgt über einen Vertrauensansatz im Rahmen eines "Web of Trust".
