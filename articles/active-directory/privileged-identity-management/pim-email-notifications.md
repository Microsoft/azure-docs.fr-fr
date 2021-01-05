---
title: Notifications par courrier électronique dans PIM - Azure Active Directory | Microsoft Docs
description: Explique ce que sont les notifications par e-mail dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: pim
ms.date: 04/28/2020
ms.author: curtand
ms.reviewer: hanki
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: fe57a44a5a6fa9a631604d92419fd8f5ebcce50a
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93394728"
---
# <a name="email-notifications-in-pim"></a>Notifications par e-mail dans PIM

Privileged Identity Management (PIM) vous informe quand des événements importants se produisent dans votre organisation Azure Active Directory (Azure AD), par exemple l’attribution ou l’activation d’un rôle. Pour vous tenir informé, Privileged Identity Management envoie des notifications par e-mail qui sont adressées à vous et à d’autres participants. Ces e-mails peuvent également inclure des liens vers des tâches appropriées, comme l’activation ou le renouvellement d’un rôle. Cet article décrit à quoi ressemblent ces e-mails, quand ils sont envoyés et qui les reçoit.

## <a name="sender-email-address-and-subject-line"></a>Adresse e-mail de l’expéditeur et ligne Objet

L’adresse de l’expéditeur des e-mails envoyés par Privileged Identity Management pour les rôles de ressources Azure AD et Azure est la suivante :

- Adresse de messagerie : **azure-noreply\@microsoft.com**
- Nom complet : Microsoft Azure

Ces e-mails incluent un préfixe **PIM** dans la ligne Objet. Voici un exemple :

- PIM : Le rôle Lecteur de sauvegarde a été définitivement attribué à Alain Charon

## <a name="notifications-for-azure-ad-roles"></a>Notifications pour les rôles Azure AD

Privileged Identity Management envoie des e-mails quand les événements suivants se produisent pour des rôles Azure AD :

- Quand l’activation d’un rôle privilégié est en attente d’approbation
- Quand une demande d’activation de rôle privilégié est approuvée
- Quand Azure AD Privileged Identity Management est activé

Les destinataires de ces e-mails pour les rôles Azure AD varient selon votre rôle, l’événement et le paramètre de notification :

| Utilisateur | Une activation d’un rôle est en attente d’approbation | Une demande d’activation de rôle est terminée | PIM est activé |
| --- | --- | --- | --- |
| Administrateur de rôle privilégié</br>(Activé/éligible) | Oui</br>(uniquement si aucun approbateur explicite n’est spécifié) | Oui* | Oui |
| Security Administrator</br>(Activé/éligible) | Non  | Oui* | Oui |
| Administrateur général</br>(Activé/éligible) | Non  | Oui* | Oui |

\* Si le paramètre [**Notifications**](pim-how-to-change-default-settings.md#notifications) a la valeur **Activer**.

L’exemple suivant montre un exemple d’e-mail envoyé quand un utilisateur active un rôle Azure AD pour l’organisation fictive Contoso.

![Nouvel e-mail Privileged Identity Management pour les rôles Azure AD](./media/pim-email-notifications/email-directory-new.png)

### <a name="weekly-privileged-identity-management-digest-email-for-azure-ad-roles"></a>E-mail Privileged Identity Management de synthèse hebdomadaire pour les rôles Azure AD

Un e-mail Privileged Identity Management récapitulatif pour les rôles Azure AD est envoyé chaque semaine aux administrateurs de rôles privilégiés, administrateurs de sécurité et administrateurs généraux qui ont activé Privileged Identity Management. Cet e-mail hebdomadaire fournit un instantané des activités Privileged Identity Management pour la semaine, ainsi que les attributions de rôles privilégiés. Il est uniquement disponible pour les organisations Azure AD sur le cloud public. Voici un exemple d’e-mail :

![E-mail Privileged Identity Management de synthèse hebdomadaire pour les rôles Azure AD](./media/pim-email-notifications/email-directory-weekly.png)

Cet e-mail comprend quatre vignettes :

| Vignette | Description |
| --- | --- |
| **Utilisateurs activés** | Nombre de fois que les utilisateurs ont activé leur rôle éligible au sein de l’organisation. |
| **Utilisateurs devenus permanents** | Nombre de fois que les utilisateurs avec une attribution éligible sont devenus permanents. |
| **Attributions de rôle dans Privileged Identity Management** | Nombre de fois que les utilisateurs se voient attribuer un rôle éligible dans Privileged Identity Management. |
| **Attributions de rôles en dehors de PIM** | Nombre de fois que les utilisateurs se voient attribuer un rôle permanent en dehors de Privileged Identity Management (dans Azure AD). |

La **vue d’ensemble des rôles principaux** liste les cinq rôles principaux de votre organisation en fonction du nombre total d’administrateurs permanents et éligibles pour chaque rôle. Le lien **Entreprendre une action** ouvre l’[Assistant PIM](pim-security-wizard.md) dans lequel vous pouvez convertir des administrateurs permanents en administrateurs éligibles par lots.

## <a name="email-timing-for-activation-approvals"></a>Chronologie des e-mails pour les approbations d’activation

Lorsque des utilisateurs activent leur rôle et que le paramètre de rôle nécessite une approbation, les approbateurs reçoivent deux e-mails pour chaque approbation :

- Demande d’approbation ou de refus de la demande d’activation de l’utilisateur (envoyée par le moteur d’approbation des demandes)
- La demande de l’utilisateur est approuvée (envoyée par le moteur d’approbation des demandes)

En outre, les administrateurs généraux et les administrateurs de rôle privilégié reçoivent un e-mail pour chaque approbation :

- Le rôle de l’utilisateur est activé (envoyé par Privileged Identity Management)

Les deux premiers e-mails envoyés par le moteur d’approbation des demandes peuvent être retardés. Actuellement, 90 % des e-mails prennent trois à dix minutes, mais pour 1 % des clients, ce délai peut s’avérer plus long, jusqu’à quinze minutes.

Si une demande d’approbation est approuvée dans le portail Azure avant l’envoi du premier e-mail, le premier e-mail n’est plus déclenché et les autres approbateurs ne sont pas avertis par e-mail de la demande d’approbation. Cela peut donner l’impression qu’ils n’ont pas reçu d’e-mail, mais c’est le comportement attendu.

## <a name="pim-emails-for-azure-resource-roles"></a>E-mails PIM pour les rôles de ressources Azure

Privileged Identity Management envoie des e-mails aux propriétaires et administrateurs de l’accès utilisateur quand les événements suivants se produisent pour les rôles de ressources Azure :

- Quand une attribution de rôle est en attente d’approbation
- Quand un rôle est attribué
- Quand un rôle va bientôt expirer
- Quand un rôle peut être étendu
- Quand un rôle est renouvelé par un utilisateur final
- Quand une demande d’activation de rôle est terminée

Privileged Identity Management envoie des e-mails aux utilisateurs finals quand les événements suivants se produisent pour les rôles de ressources Azure :

- Quand un rôle est attribué à l’utilisateur
- Quand le rôle d’un utilisateur a expiré
- Quand le rôle d’un utilisateur est étendu
- Quand la demande d’activation de rôle d’un utilisateur est terminée

L’exemple suivant montre un exemple d’e-mail envoyé quand un utilisateur se voit attribuer un rôle de ressource Azure pour l’organisation fictive Contoso.

![Nouvel e-mail Privileged Identity Management pour les rôles de ressources Azure](./media/pim-email-notifications/email-resources-new.png)

## <a name="next-steps"></a>Étapes suivantes

- [Configurer les paramètres des rôles Azure AD dans Privileged Identity Management](pim-how-to-change-default-settings.md)
- [Approuver ou rejeter des requêtes de rôles Azure AD dans Privileged Identity Management](azure-ad-pim-approval-workflow.md)
