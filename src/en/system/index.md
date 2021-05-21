# Overview

The Kiebitz system consists of several components that can be used independently of each other and without central coordination. Together they form a federated, decentralized system for appointment scheduling.

The backend consists of three different services:

* **Appointment servers** store priority tokens of users in lists of catchment areas, pass them on to providers, and mediate encrypted communication between providers and users.
* **Storage servers** optionally store encrypted user and provider settings.
* **Notification server** optionally sends notifications to users (e.g. via e-mail)

In addition, the system has several client applications:

* The **user application** allows initialization of contact tracking for users.
* The **operator application** allows the collection of visit data by operators and the transmission of the data to one or more **operator servers**.
* The **scanner application** allows the collection of visit data on mobile devices. It exchanges data with the **operator application via the operator** **API**.
* The **health department application** allows the request of contact data by health departments, the retrieval of returned data as well as the decryption and further processing of this data.

## System concept

The design of the components aims to obtain a system that is **redundant**, **decentralized**, **federated**, **resilient**, **secure**, and **privacy-friendly**. This is achieved through several aspects:

### Decentralisation

In the Kiebitz system, there is basically no central office that controls the operation of the entire system. All server types can be operated independently. The functional integration takes place via a trust approach within the framework of a "Web of Trust".

