---
title: Processus de demande et notifications - Gestion des droits d’utilisation Azure AD
description: Découvrez le processus de demande d’un package d’accès, et les envois de notifications par e-mail dans la gestion des droits d’utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: mamtakumar
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: compliance
ms.date: 12/23/2020
ms.author: barclayn
ms.reviewer: mamkumar
ms.collection: M365-identity-device-management
ms.openlocfilehash: bb84eae2db01180865db54250598f45c7b2ddca0
ms.sourcegitcommit: 6e2d37afd50ec5ee148f98f2325943bafb2f4993
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97746233"
---
# <a name="request-process-and-email-notifications-in-azure-ad-entitlement-management"></a>Processus de demande et notifications par e-mail dans la gestion des droits d’utilisation Azure AD

Lorsqu’un utilisateur envoie une demande à un package d’accès, un processus démarre pour remettre cette demande. La gestion des droits d’utilisation Azure AD envoie des notifications par e-mail aux approbateurs et demandeurs lorsque des événements clés se produisent pendant le processus. Cet article décrit le processus de demande et les notifications par e-mail qui sont envoyées.

## <a name="request-process"></a>Processus de demande

Un utilisateur qui a besoin d’accéder à un package d’accès peut soumettre une demande d’accès. Selon la configuration de la stratégie, la demande peut nécessiter une approbation. Lorsqu’une demande est approuvée, un processus commence à attribuer à l’utilisateur l’accès à chaque ressource dans le package d’accès. Le diagramme suivant montre une vue d’ensemble du processus et des différents états :

![Diagramme du processus d’approbation](./media/entitlement-management-process/request-process.png)

| State | Description |
| --- | --- |
| Envoyée | L’utilisateur soumet une demande. |
| Approbation en attente | Si la stratégie d’un package d’accès exige une approbation, la demande passe à l’approbation en attente. |
| Expiré | Si aucun approbateur ne valide la demande dans le délai d’attente de la demande d’approbation, la demande expire. Pour réessayer, l’utilisateur doit soumettre de nouveau sa demande. |
| Refusée | L’approbateur refuse une demande. |
| Approved | L’approbateur approuve une demande. |
| Remise | L’accès à toutes les ressources dans le package d’accès n’a **pas** été affecté à l’utilisateur. S’il s’agit d’un utilisateur externe, il n’a peut-être pas encore accédé au répertoire de ressources. Il se peut aussi qu’il n’ait pas accepté l’invite de consentement. |
| Delivered (Livrée) | L’utilisateur a reçu l’accès à toutes les ressources dans le package d’accès. |
| Accès étendu | Si des extensions sont autorisées dans la stratégie, l’utilisateur a étendu l’attribution. |
| Accès expiré | L’accès de l’utilisateur au package d’accès a expiré. Pour obtenir de nouveau l’accès, l’utilisateur doit soumettre une demande. |

## <a name="email-notifications"></a>Notifications par e-mail

Si vous êtes un approbateur, vous recevez des notifications par e-mail lorsque vous avez besoin d’approuver une demande d’accès. Vous recevez également des notifications lorsqu’une demande d’accès est terminée. Si vous êtes demandeur, des notifications par e-mail indiquant l’état de votre demande vous sont également envoyées.

Les diagrammes suivants montrent quand ces notifications par e-mail sont envoyées aux approbateurs ou au demandeur. Reportez-vous au [tableau des notifications par e-mail](entitlement-management-process.md#email-notifications-table) pour trouver le numéro correspondant dans les notifications par e-mail affichées dans les diagrammes.

### <a name="first-approvers-and-alternate-approvers"></a>Premiers approbateurs et approbateurs de substitution
Le diagramme suivant illustre l’expérience des premiers approbateurs et des approbateurs de substitution, ainsi que les notifications par e-mail qu’ils reçoivent pendant le processus de demande :

![Déroulement du processus pour les premiers approbateurs et les approbateurs de substitution](./media/entitlement-management-process/first-approvers-and-alternate-with-escalation-flow.png)

### <a name="requestors"></a>Demandeurs
Le diagramme suivant illustre l’expérience des demandeurs et les notifications par e-mail qu’ils reçoivent pendant le processus de demande :

![Déroulement du processus du demandeur](./media/entitlement-management-process/requestor-approval-request-flow.png)

### <a name="2-stage-approval"></a>Approbation en deux phases
Le diagramme suivant illustre l’expérience des approbateurs de la phase 1 et de la phase 2 et les notifications par e-mail qu’ils reçoivent pendant le processus de demande :

![Diagramme du processus d’approbation en deux phases](./media/entitlement-management-process/2stage-approval-with-request-timeout-flow.png)

### <a name="email-notifications-table"></a>Tableau de notifications par e-mail
Le tableau suivant fournit plus de détails sur chacune de ces notifications par e-mail. Pour gérer ces e-mails, vous pouvez utiliser des règles. Par exemple, dans Outlook, vous pouvez créer des règles pour déplacer les e-mails vers un dossier si l’objet contient des mots de ce tableau :

| # | Objet de l’e-mail | Moment de l’envoi | Envoyé à |
| --- | --- | --- | --- |
| 1 | Action demandée : Approuver ou refuser une demande transférée d’ici le *[date]* | Cet e-mail sera envoyé aux approbateurs de substitution de la phase 1 (une fois la demande remontée) pour leur demander d’agir. | Approbateurs de substitution de la phase 1 |
| 2 | Action demandée : Approuver ou refuser la demande d’ici le *[date]* | Cet e-mail sera envoyé au premier approbateur, si l’escalade est désactivée, pour lui demander d’agir. | Premier approbateur |
| 3 | Rappel : Approuver ou refuser la demande avant le *[date]* pour *[demandeur]* | Cet e-mail de rappel sera envoyé au premier approbateur, si l’escalade est désactivée. L’e-mail lui demande d’agir s’il ne l’a pas fait. | Premier approbateur |
| 4 | Approuver ou refuser la demande avant *[heure]* le *[date]* | Cet e-mail sera envoyé au premier approbateur (si l’escalade est activée) pour lui demander d’agir. | Premier approbateur |
| 5 | Rappel d’action requise : Approuver ou refuser la demande avant le *[date]* pour *[demandeur]* | Cet e-mail de rappel sera envoyé au premier approbateur, si l’escalade est activée. L’e-mail lui demande d’agir s’il ne l’a pas fait. | Premier approbateur |
| 6 | La demande a expiré pour *[package d’accès]* | Cet e-mail sera envoyé au premier approbateur et aux approbateurs de substitution de la phase 1 après l’expiration de la demande. | Premier approbateur, approbateurs de substitution de la phase 1 |
| 7 | Demande approuvée pour *[demandeur]* à *[package d’accès]* | Cet e-mail est envoyé au premier approbateur et aux approbateurs de substitution de phase 1 à la fin d’une demande. | Premier approbateur, approbateurs de substitution de la phase 1 |
| 8 | Demande approuvée pour *[demandeur]* à *[package d’accès]* | Cet e-mail est envoyé au premier approbateur et aux approbateurs de substitution de la phase 1 d’une demande en deux phases lorsque la phase 1 est approuvée. | Premier approbateur, approbateurs de substitution de la phase 1 |
| 9 | Demande refusée pour *[package d’accès]* | Cet e-mail est envoyé au demandeur lorsque sa demande est refusée. | Demandeur |
| 10 | Votre demande a expiré pour *[package d’accès]* | Cet e-mail est envoyé au demandeur à la fin d’une demande en une ou deux phases. L’e-mail informe le demandeur que la demande a expiré. | Demandeur |
| 11 | Action demandée : Approuver ou refuser la demande d’ici le *[date]* | Cet e-mail sera envoyé au deuxième approbateur, si l’escalade est désactivée, pour lui demander d’agir. | Deuxième approbateur |
| 12 | Rappel d’action requise : Approuver ou refuser la demande d’ici le *[date]* | Cet e-mail de rappel sera envoyé au deuxième approbateur, si l’escalade est désactivée. La notification lui demande d’agir s’il ne l’a pas encore fait. | Deuxième approbateur |
| 13 | Action demandée : Approuver ou refuser la demande avant le *[date]* pour *[demandeur]* | Cet e-mail sera envoyé au deuxième approbateur, si l’escalade est activée, pour lui demander d’agir. | Deuxième approbateur |
| 14 | Rappel d’action requise : Approuver ou refuser la demande avant le *[date]* pour *[demandeur]* | Cet e-mail de rappel sera envoyé au deuxième approbateur, si l’escalade est activée. La notification lui demande d’agir s’il ne l’a pas encore fait. | Deuxième approbateur |
| 15 | Action demandée : Approuver ou refuser une demande transférée d’ici le *[date]* | Cet e-mail sera envoyé aux approbateurs de substitution de la phase 2, si l’escalade est activée, pour leur demander d’agir. | Approbateurs de substitution de la phase 2 |
| 16 | Demande approuvée pour *[demandeur]* à *[package d’accès]* | Cet e-mail est envoyé au deuxième approbateur et aux approbateurs de substitution de la phase 2 lors de l’approbation de la demande. | Deuxième approbateur, approbateurs de substitution de la phase 2 |
| 17 | Une demande a expiré pour *[access_package]* | Cet e-mail sera envoyé au deuxième approbateur ou aux approbateurs de substitution, après l’expiration de la demande. | Deuxième approbateur, approbateurs de substitution de la phase 2 |
| 18 | Vous avez maintenant accès à *[package d’accès]* | Cet e-mail est envoyé aux utilisateurs finaux pour leur indiquer qu’ils peuvent commencer à utiliser leur accès. | Demandeur |
| 19 | Extension d’accès pour *[package d’accès]* avant le *[date]* | Cet e-mail est envoyé aux utilisateurs finaux avant l’expiration de leur accès. | Demandeur |
| 20 | L’accès est terminé pour *[package d’accès]* | Cet e-mail est envoyé aux utilisateurs finaux après l’expiration de leur accès. | Demandeur |

### <a name="access-request-emails"></a>E-mails notifiant la demande d’accès

Lorsqu’un demandeur envoie une demande d’accès à un package d’accès qui est configuré pour demander l’approbation, tous les approbateurs ajoutés à la stratégie reçoivent une notification par e-mail avec les détails de la demande. Les détails de l’e-mail incluent : le nom de l’organisation du demandeur, la justification commerciale et les dates de début et de fin d’accès demandées (le cas échéant). Les détails incluront également le moment où la demande a été envoyée et où la demande expirera.

L’e-mail comporte un lien, sur lequel les approbateurs peuvent cliquer pour accéder à Mon Accès et approuver ou refuser la demande d’accès. Voici un exemple de notification par e-mail qui est envoyé au premier ou au deuxième approbateur (si l’approbation en deux phases est activée) pour effectuer une demande d’accès :

![E-mail concernant l’approbation de la demande pour un package d’accès](./media/entitlement-management-shared/approver-request-email.png)

Les approbateurs peuvent également recevoir un e-mail de rappel. L’e-mail demande à l’approbateur de prendre une décision quant à la demande. Voici un exemple d’e-mail de notification que l’approbateur reçoit pour lui rappeler d’agir :

![E-mail de rappel notifiant la demande d’accès](./media/entitlement-management-process/approver-access-request-reminder-email.png)

### <a name="alternate-approvers-request-emails"></a>E-mails de demande adressés aux approbateurs de substitution

Si le paramètre Approbateurs de substitution est activé et que la demande est toujours en attente, l’e-mail est transféré. Les approbateurs de substitution reçoivent un e-mail pour approuver ou refuser la demande. Vous pouvez activer des approbateurs de substitution pour les phases 1 et 2. Voici un exemple d’e-mail de notification que les approbateurs de substitution reçoivent :

![E-mail de demande adressé aux approbateurs de substitution](./media/entitlement-management-process/alternate-approver-email-fwd-request.png)

Les approbateurs principaux et de substitution peuvent approuver ou refuser la demande.

### <a name="approved-or-denied-emails"></a>E-mails notifiant l’approbation ou le refus

 Quand un approbateur reçoit une demande d’accès présentée par un demandeur, il peut approuver ou refuser la demande d’accès. L’approbateur doit ajouter une justification professionnelle motivant sa décision. Voici un exemple d’e-mail envoyé aux approbateurs principaux et de substitution après l’approbation d’une demande :

![E-mail notifiant l’approbation de la demande pour un package d’accès](./media/entitlement-management-process/approver-request-email-approved.png)

Lorsqu’une demande d’accès est approuvée et que son accès est configuré, une notification par e-mail est envoyée au demandeur pour lui indiquer qu’il a à présent accès au package d’accès. Voici un exemple de notification par e-mail qui est envoyé à un demandeur lorsque l’accès à un package d’accès lui est accordé :

![E-mail notifiant l’approbation de la demande pour une demande d’accès](./media/entitlement-management-process/requestor-email-approved.png)

Lorsqu’une demande d’accès est refusée, une notification par e-mail est envoyée au demandeur. Voici un exemple de notification par e-mail qui est envoyé à un demandeur lorsque sa demande d’accès est refusée :

![E-mail de refus de la demande du demandeur](./media/entitlement-management-process/requestor-email-denied.png)

### <a name="2-stage-approval-access-request-emails"></a>E-mails de demande d’accès pour une approbation en deux phases

Si l’approbation en deux phases est activée, au moins deux approbateurs doivent approuver la demande, un à chaque étape, avant que le demandeur ne puisse recevoir l’accès.

Au cours de la phase 1, le premier approbateur reçoit l’e-mail de demande d’accès et prend une décision. S’il approuve la demande, tous les premier approbateurs et les approbateurs de substitution de la phase 1 (si l’escalade est activée) recevront une notification indiquant que la phase 1 est terminée. Voici un exemple d’e-mail de notification envoyé lorsque la phase 1 est terminée :

![E-mail de demande d’accès en deux phases](./media/entitlement-management-process/approver-request-email-2stage.png)

Une fois que le premier approbateur ou les approbateurs de substitution ont approuvé la demande à la phase 1, la phase 2 commence. Au cours de la phase 2, le deuxième approbateur recevra l’e-mail de notification de demande d’accès. Lorsque le deuxième approbateur ou les approbateurs de substitution de la phase 2 (si l’escalade est activée) ont décidé d’approuver ou de refuser la demande, des e-mails de notification sont envoyés au premier et au deuxième approbateurs, ainsi qu’à tous les approbateurs de substitution des phases 1 et 2, ainsi qu’au demandeur.

### <a name="expired-access-request-emails"></a>E-mails notifiant la demande d’accès expirée

Les demandes d’accès peuvent expirer si aucun approbateur n’a approuvé ou refusé la demande. 

Lorsque la demande atteint sa date d’expiration, elle expire et ne peut plus être approuvée ou refusée par les approbateurs. Voici un exemple d’e-mail de notification envoyé à tous les premiers et deuxièmes approbateurs (si l’approbateur en deux phases est activée) et à tous les approbateurs  de substitution :

![E-mail notifiant la demande d’accès expirée pour les approbateurs](./media/entitlement-management-process/approver-request-email-expired.png)

Une notification par e-mail est également envoyée au demandeur pour l’informer que sa demande d’accès a expiré et qu’il doit soumettre sa demande à nouveau. Le diagramme suivant illustre l’expérience du demandeur et les notifications par e-mail qu’il reçoit lorsqu’il demande d’étendre l’accès :

![Flux de processus d’accès étendu par le demandeur](./media/entitlement-management-process/requestor-expiration-request-flow.png) 

Voici un exemple de notification par e-mail qui est envoyée à un demandeur lorsque sa demande d’accès a expiré :

![E-mail notifiant la demande d’accès expirée pour le demandeur](./media/entitlement-management-process/requestor-email-request-expired.png)

## <a name="next-steps"></a>Étapes suivantes

- [Demander l’accès à un package d’accès](entitlement-management-request-access.md)
- [Approuver ou refuser les demandes d’accès](entitlement-management-request-approve.md)
