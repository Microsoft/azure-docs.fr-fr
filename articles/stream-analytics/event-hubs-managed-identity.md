---
title: Accès à Event Hubs à partir d’un travail Azure Stream Analytics à l’aide des identités managées (préversion)
description: Cet article explique comment utiliser les identités managées afin d’authentifier un travail Azure Stream Analytics sur l’entrée et la sortie Azure Event Hubs.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 11/30/2020
ms.openlocfilehash: c65833e5bf581c6326bf9362c7e5fc00a985d301
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96354868"
---
# <a name="use-managed-identities-to-access-event-hubfrom-an-azure-stream-analytics-job-preview"></a>Accès à Event Hubs à partir d’un travail Azure Stream Analytics à l’aide des identités managées (préversion)

Azure Stream Analytics prend en charge l’authentification d’identité managée pour l’entrée et la sortie Azure Event Hubs. Les identités managées permettent d’éviter les limitations des méthodes d’authentification basée sur l’utilisateur, par exemple la réauthentification obligatoire après un changement de mot de passe ou l’expiration du jeton d’utilisateur tous les 90 jours. En l'absence d'authentification manuelle, vos déploiements Stream Analytics peuvent être entièrement automatisés.  

Une identité managée est une application managée inscrite auprès d’Azure Active Directory qui représente un travail Stream Analytics. L’application gérée est utilisée pour l’authentification auprès d’une ressource ciblée, y compris les ressources Event Hubs qui se trouvent derrière un pare-feu ou un réseau virtuel. Pour savoir comment contourner les pare-feu, consultez [Autorisation de l’accès aux espaces de noms Azure Event Hubs via des points de terminaison privés](../event-hubs/private-link-service.md#trusted-microsoft-services).

Cet article montre comment activer une identité managée pour l’entrée ou la sortie Event Hubs d’un travail Stream Analytics par le biais du Portail Azure. Pour pouvoir activer l’identité managée, vous devez disposer d’un travail Stream Analytics et d’une ressource Event Hubs.

## <a name="create-a-managedidentity"></a>Création d’une identité managée  

Commencez par créer une identité managée pour votre tâche Azure Stream Analytics.  

1. Sur le Portail Azure, ouvrez votre travail Azure Stream Analytics.  

1. Dans le menu de navigation gauche, sélectionnez  **Identité managée**  sous  *Configurer*. Cochez ensuite la case située à côté de l’option  **Utiliser l’identité managée affectée par le système** , puis sélectionnez  **Enregistrer**.

   :::image type="content" source="media/event-hubs-managed-identity/system-assigned-managed-identity.png" alt-text="Identité managée affectée par le système":::  

1. Un principal de service est créé pour l’identité du travail Stream Analytics dans Azure Active Directory. Le cycle de vie de la nouvelle identité est géré par Azure. Quand le travail Stream Analytics est supprimé, l’identité associée (autrement dit, le principal de service) est également automatiquement supprimée par Azure.  

   Lorsque vous enregistrez la configuration, l’ID objet (l’OID) du principal de service s’affiche en tant qu’ID de principal, comme ci-dessous :  

   :::image type="content" source="media/event-hubs-managed-identity/principal-id.png" alt-text="ID du principal":::

   Le principal de service a le même nom que le travail Stream Analytics. Par exemple, si le nom de votre travail est  `MyASAJob`, le nom du principal de service est également  `MyASAJob`.  

## <a name="grant-the-stream-analytics-job-permissionsto-access-the-event-hub"></a>Octroi de l’autorisation d’accéder au hub d’événements pour le travail Stream Analytics

Pour que le travail Stream Analytics puisse accéder à votre hub d’événements à l’aide de l’identité managée, il faut que le principal de service que vous avez créé dispose d’autorisations spéciales sur le hub d’événements.

1. Accédez à **Contrôle d’accès (IAM)** dans votre hub d’événements.

1. Sélectionnez **+ Ajouter**, puis **Ajouter une attribution de rôle**.

1. Sur la page **Ajouter une attribution de rôle**, entrez les options suivantes :

   |Paramètre|Valeur|
   |---------|-----|
   |Role|Propriétaire de données Azure Event Hubs|
   |Attribuer l’accès à|Utilisateur, groupe ou principal de service|
   |Sélectionnez|Entrez le nom de votre travail Stream Analytics|

   :::image type="content" source="media/event-hubs-managed-identity/add-role-assignment.png" alt-text="Ajouter une attribution de rôle":::

1. Sélectionnez **Enregistrer** et attendez environ une minute, le temps que les modifications se propagent.

Vous pouvez également accorder ce rôle au niveau de l’espace de noms du hub d’événements, qui propagera naturellement les autorisations à tous les hubs d’événements créés dessous. Autrement dit, tous les hubs d’événements qui se trouvent sous un espace de noms peuvent être utilisés comme ressources d’authentification d’identité managée dans un travail Stream Analytics.

## <a name="create-anevent-hub-input-or-output"></a>Création d’une entrée ou d’une sortie Event Hubs  

Maintenant que votre identité managée est configurée, vous pouvez ajouter la ressource Event Hubs comme entrée ou sortie de votre travail Stream Analytics.  

### <a name="add-the-event-hub-as-an-input"></a>Ajout du hub d’événements comme entrée 

1. Accédez à votre travail Stream Analytics, puis à la page **Entrées** sous **Topologie des travaux**.

1. Cliquez sur **+ Ajouter une entrée Stream > Hub d’événements**. Dans la fenêtre Propriétés de l’entrée, recherchez et sélectionnez votre hub d’événements, puis sélectionnez **Identité managée** dans le menu déroulant *Mode d’authentification*.

1. Renseignez le reste des propriétés, puis sélectionnez **Enregistrer**.

### <a name="add-the-event-hub-as-an-output"></a>Ajout du hub d’événements comme sortie

1. Accédez à votre travail Stream Analytics, puis à la page **Sorties** sous **Topologie des travaux**.

1. Sélectionnez **Ajouter > Hub d’événements**. Dans la fenêtre Propriétés de la sortie, recherchez et sélectionnez votre hub d’événements, puis sélectionnez **Identité managée** dans le menu déroulant *Mode d’authentification*.

1. Renseignez le reste des propriétés, puis sélectionnez **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

* [Event Hubs à partir d’Azure Stream Analytics](event-hubs-output.md)
* [Diffuser en continu des données depuis Event Hubs](stream-analytics-define-inputs.md#stream-data-from-event-hubs)