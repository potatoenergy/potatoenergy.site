---
ShowBreadCrumbs: false
ShowPostNavLinks: false
canonicalURL: "https://potatoenergy.ru/en/privacy"
comments: false
description: "Data processing in Potato Energy home project"
disableAnchoredHeadings: true
disableShare: true
hideFooter: false
hideSummary: true
hidemeta: true
robotsNoIndex: false
searchHidden: true
showtoc: false
slug: "en/privacy"
title: "Privacy Policy"
tocopen: false
---

# Privacy Policy

## 1. General provisions

1.1. This Privacy Policy defines the procedure for processing and protecting personal data of users of Potato Energy services hosted on the domains `*.potatoenergy.ru `.

1.2. Potato Energy is a self-hosted home project provided in an "as is" state. The project is not a commercial service and is intended for personal use and a limited number of users.

1.3. Kirill Plotnikov is the operator of personal data.

1.4. By using the Potato Energy services, the user agrees to the processing of personal data in accordance with this Policy and Federal Law No. 152-FZ dated 27.07.2006 "On Personal Data".

## 2. The composition of personal data being processed

2.1. The following categories of data may be processed as part of the use of the services:

| Data category       | Sources of collection                                  | Purpose of processing                 |
| ------------------- | ------------------------------------------------------ | ------------------------------------- |
| Login / nickname    | Services with authorization                            | User identification                   |
| Password hash       | Authelia, services with authorization                  | Authentication                        |
| Email address       | Authelia, Nextcloud, Mastodon, Open-WebUI, Grafana     | Notifications, Access restoration     |
| IP address          | Traefik logs/Nginx                                     | Ensuring security, preventing attacks |
| Files and documents | Nextcloud, Mastodon, Open-WebUI, Drawpile              | Storage on user request               |
| Chat messages       | Nextcloud, Mastodon, Open-WebUI, Game servers, Discord | Communication between users           |
| Game progress       | Minecraft, Terraria, Mindustry                         | Saving the state of the game world    |

2.2. Special categories of personal data are not processed.

## 3. Data Storage

3.1. The main data is stored locally on the Potato Energy server located on the territory of the Russian Federation.

3.2. Backup is performed on the same server. If necessary, the data is encrypted.

3.3. Integration with third-party services (GitHub, Discord, Steam, etc.) is carried out at the user's choice. Data processing in such services is regulated by their own privacy policies.

## 4. Data Protection measures

4.1. The following technical measures are applied to protect personal data:

- Encryption of connections via TLS (Let's Encrypt certificates);
- Storing passwords exclusively in hashed form;
- Restriction of access to administrative interfaces via Authelia with two-factor authentication support;
- Automatic deletion of access logs after 90 days;
- The absence of third-party trackers, advertising systems and analytical services.

4.2. Organizational security measures include restriction of access to data based on the principle of minimum privileges and regular security monitoring.

## 5. Data transmission

5.1. Personal data is not transferred to third parties for commercial purposes.

5.2. Data transfer is possible only in the following cases::

- At the explicit request of the user (account or file export);
- At the request of authorized state bodies of the Russian Federation within the framework of the current legislation;
- As part of the operation of federated services (Nextcloud, Mastodon), public data can be accessed by other instances via the ActivityPub protocol.

## 6. Deleting data

6.1. The User has the right to request the deletion of his account or personal data at any time.

6.2. The request is sent to the operator via e-mail or messengers specified in Section 9 of this Policy.

6.3. Data deletion is carried out within 30 days from the date of confirmation of the request.

## 7. Using cookies

7.1. The Services use only technical cookies necessary to maintain the authorization session and save interface preferences.

7.2. Third-party cookies, trackers and analytics systems are not used.

## 8. Rights of the personal data subject

8.1. In accordance with Federal Law No. 152-FZ, the user has the right to:

- Receive information about the processing of your personal data;
- Request clarification, blocking or destruction of personal data;
- Request the export of personal data in a machine-readable format;
- Revoke consent to the processing of personal data.

8.2. In order to exercise the rights provided for in clause 8.1, the user sends a request to the operator using the contacts specified in Section 9.

## 9. Limitations and Disclaimers of Guarantees

9.1. The Project is provided "as is" without any express or implied guarantees.

9.2. The Operator does not guarantee:

- Uninterrupted and error-free operation of the services;
- Complete data security in case of force majeure circumstances;
- Compatibility of services with all devices and software.

9.3. The User is recommended to backup important data independently.

## 10. Final provisions

10.1. This Policy applies to all services on the domains `*.potatoenergy.ru `, run by Kirill Plotnikov.

10.2. The Policy has been drafted taking into account the requirements of Federal Law No. 152-FZ dated 27.07.2006 "On Personal Data".

10.3. Questions related to the processing of personal data are sent to:

- Email address: ponfertato@potatoenergy.ru
- Telegram: @ponfertato

10.4. The Operator reserves the right to make changes to this Policy. The current version is published at `/privacy'.
