---
title: Configurer des notifications et des modèles de messages électroniques
titleSuffix: Azure API Management
description: Apprenez à configurer des notifications et des modèles de messages électroniques dans Gestion des API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/10/2020
ms.author: apimpm
ms.openlocfilehash: 786a9e26003a7afb98307e0bd7fae94c42a2f00d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84690300"
---
# <a name="how-to-configure-notifications-and-email-templates-in-azure-api-management"></a>Configuration des notifications et des modèles de messages électroniques dans Gestion des API Azure

Gestion des API Azure permet de configurer les notifications pour des événements spécifiques et de configurer les modèles de courrier électronique utilisés pour communiquer avec les administrateurs et les développeurs de l’instance Gestion des API. Cet article montre comment configurer des notifications pour les événements disponibles. Il offre également un aperçu de la configuration des modèles d’e-mail utilisés pour ces événements.

## <a name="prerequisites"></a>Conditions préalables requises

Si vous ne disposez pas d’une instance de service Gestion des API, effectuez le démarrage rapide suivant : [Créer une instance Gestion des API Azure](get-started-create-service-instance.md).

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="configure-notifications"></a><a name="publisher-notifications"> </a>Configurer les notifications

1.  Sélectionnez votre instance **GESTION DES API**.
2.  Cliquez sur **Notifications** pour afficher les notifications disponibles.

    ![Notifications de l’éditeur][api-management-publisher-notifications]

    La liste suivante répertorie les événements pour lesquels il est possible de configurer des notifications.

    -   **Demandes d'abonnement (approbation nécessaire)** : les destinataires du message et les utilisateurs spécifiés reçoivent une notification pour les demandes d'abonnement aux produits API nécessitant une approbation.
    -   **Nouveaux abonnements** : les destinataires du message et les utilisateurs spécifiés reçoivent une notification concernant les nouveaux abonnements aux produits API.
    -   **Demandes de la galerie d'applications** : les destinataires du message et les utilisateurs spécifiés reçoivent une notification lorsque de nouvelles applications sont proposées dans la galerie d'applications.
    -   **CCI** : les destinataires du message et les utilisateurs spécifiés reçoivent une copie cachée de tous les messages envoyés aux développeurs.
    -   **Nouveau problème ou commentaire** : les destinataires du message et les utilisateurs spécifiés reçoivent une notification lorsqu'un problème ou un commentaire est envoyé sur le portail des développeurs.
    -   **Message de fermeture de compte** : les destinataires du message et les utilisateurs spécifiés reçoivent une notification lorsqu'un compte est fermé.
    -   **Limite du quota d'abonnements bientôt atteint** : les destinataires du message et les utilisateurs spécifiés reçoivent une notification lorsque l'utilisation de l'abonnement approche le quota.

        > [!NOTE]
        > Les notifications sont déclenchées par la stratégie [quota by subscription](api-management-access-restriction-policies.md#SetUsageQuota) uniquement. La stratégie [Quota by key](api-management-access-restriction-policies.md#SetUsageQuotaByKey) ne génère pas de notifications.

    Pour chaque événement, vous pouvez spécifier les destinataires du message via la zone de texte d'adresse. Vous pouvez également sélectionner les utilisateurs dans une liste.

3.  Pour spécifier les adresses à notifier, entrez-les dans la zone de texte d'adresse. Si vous devez entrer plusieurs adresses, séparez-les par des virgules.

    ![Notification recipients][api-management-email-addresses]

4.  Cliquez sur **Ajouter**.

## <a name="configure-notification-templates"></a><a name="email-templates"> </a>Configurer des modèles de notification

Gestion des API fournit des modèles de notification pour les e-mails envoyés dans le cadre de l’administration et de l’utilisation du service. Les modèles suivants sont fournis.

-   Demande d'ajout à la galerie d'applications approuvée
-   Lettre d'adieu au développeur
-   Notification de la limite du quota d'abonnement bientôt atteinte
-   Invitation de l'utilisateur
-   Nouveau commentaire ajouté à un problème
-   Nouveau problème reçu
-   Nouvel abonnement activé
-   Confirmation du renouvellement de l'abonnement
-   Refus de la demande d'abonnement
-   Réception de la demande d'abonnement

Ces modèles peuvent être modifiés comme vous le souhaitez.

Pour afficher et configurer les modèles d’e-mail pour votre instance Gestion des API, cliquez sur **Modèles de notification**.

![Email templates][api-management-email-templates]

Pour chaque modèle de message, l'objet est au format texte et le corps au format HTML. Chaque élément peut être personnalisé.

![Email template editor][api-management-email-template]

La liste **Paramètres** contient une liste de paramètres qui, lorsqu'ils sont insérés dans l'objet ou dans le corps du message, sont remplacés par une valeur définie lorsque le message est envoyé. Pour insérer un paramètre, placez le curseur là où vous voulez insérer le paramètre, puis cliquez sur la flèche à gauche du nom du paramètre.

> [!NOTE]
> Les paramètres ne sont pas remplacés par les valeurs réelles lors de l’aperçu ou du test.

Pour enregistrer les changements apportés au modèle d’e-mail, cliquez sur **Enregistrer**. Pour annuler les changements, cliquez sur **Abandonner**.

[api-management-management-console]: ./media/api-management-howto-configure-notifications/api-management-management-console.png
[api-management-publisher-notifications]: ./media/api-management-howto-configure-notifications/api-management-publisher-notifications.png
[api-management-email-addresses]: ./media/api-management-howto-configure-notifications/api-management-email-addresses.png
[api-management-email-templates]: ./media/api-management-howto-configure-notifications/api-management-email-templates.png
[api-management-email-templates-list]: ./media/api-management-howto-configure-notifications/api-management-email-templates-list.png
[api-management-email-template]: ./media/api-management-howto-configure-notifications/api-management-email-template.png
[configure publisher notifications]: #publisher-notifications
[configure email templates]: #email-templates
[how to create and use groups]: api-management-howto-create-groups.md
[how to associate groups with developers]: api-management-howto-create-groups.md#associate-group-developer
[get started with azure api management]: get-started-create-service-instance.md
[create an api management service instance]: get-started-create-service-instance.md
