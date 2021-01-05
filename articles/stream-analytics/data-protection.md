---
title: Protection des données dans Azure Stream Analytics
description: Cet article explique comment chiffrer vos données privées utilisées par une tâche Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/03/2020
ms.openlocfilehash: 4436289d544de057acef132117346ac53c20b5a7
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576494"
---
# <a name="data-protection-in-azure-stream-analytics"></a>Protection des données dans Azure Stream Analytics 

Azure Stream Analytics est une plateforme en tant que service entièrement gérée, qui vous permet de générer des pipelines d’analyse en temps réel. Le gros du travail, par exemple, l’approvisionnement du cluster, la mise à l’échelle des nœuds pour prendre en charge votre utilisation et la gestion des points de contrôle internes sont gérés en coulisses.

## <a name="private-data-assets-that-are-stored"></a>Ressources de données privées stockées

Azure Stream Analytics conserve les métadonnées et les données suivantes pour exécuter : 

* Définition des requêtes et leur configuration associée  

* Fonctions ou agrégats définis par l’utilisateur  

* Points de contrôle nécessaires au runtime Stream Analytics

* Captures instantanées de données de référence 

* Détails de connexion des ressources utilisées par votre tâche Stream Analytics

Pour vous aider à répondre à vos obligations de conformité dans n’importe quel secteur ou environnement réglementé, apprenez-en davantage sur les [offres de conformité de Microsoft](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942). 

## <a name="in-region-data-residency"></a>Résidence des données dans la région
Azure Stream Analytics stocke les données client et les autres métadonnées décrites ci-dessus. Les données client sont stockées par Azure Stream Analytics dans une seule région par défaut, si bien que ce service répond automatiquement aux exigences en matière de résidence des données de la région, y compris celles spécifiées dans le [Centre de gestion de la confidentialité](https://azuredatacentermap.azurewebsites.net/).
En outre, vous pouvez choisir de stocker toutes les ressources de données (données client et autres métadonnées) relatives à votre tâche Stream Analytics dans une région unique en les chiffrant dans un compte de stockage de votre choix.

## <a name="encrypt-your-data"></a>Chiffrer vos données

Stream Analytics utilise automatiquement les normes de chiffrement les plus sophistiquées dans l’ensemble de son infrastructure pour chiffrer et sécuriser vos données. Vous pouvez tout simplement faire confiance à Stream Analytics pour stocker vos données en toute sécurité. Vous n’avez donc pas à vous soucier de la gestion de l’infrastructure.

Si vous souhaitez utiliser des clés gérées par le client pour chiffrer vos données, vous pouvez utiliser votre propre compte de stockage (à usage général v1 ou v2) pour stocker toutes les ressources de données privées requises par le runtime Stream Analytics. Votre compte de stockage peut être chiffré en fonction des besoins. Aucune de vos ressources de données privées n’est stockée de façon permanente par l’infrastructure Stream Analytics. 

Vous devez configurer ce paramètre au moment de la création de la tâche Stream Analytics, et ne pouvez pas le modifier pendant le cycle de vie de la tâche. Il n’est pas recommandé de modifier ou supprimer le stockage qu’utilise votre Stream Analytics. Si vous supprimez votre compte de stockage, vous supprimez définitivement toutes les ressources de données privées, ce qui entraînera l’échec de votre tâche. 

Une mise à jour ou une rotation de clés sur votre compte de stockage n’est pas possible à l’aide du portail Stream Analytics. Vous pouvez mettre à jour les clés à l’aide des API REST.


### <a name="configure-storage-account-for-private-data"></a>Configurer un compte de stockage pour des données privées 

Chiffrez votre compte de stockage pour sécuriser toutes vos données et choisissez explicitement l’emplacement de vos données privées. 

Pour vous aider à répondre à vos obligations de conformité dans n’importe quel secteur ou environnement réglementé, apprenez-en davantage sur les [offres de conformité de Microsoft](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942). 

Utilisez les étapes suivantes pour configurer votre compte de stockage pour des ressources de données privées. Cette configuration s’effectue à partir de votre tâche Stream Analytics, non à partir de votre compte de stockage.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez **Créer une ressource** dans le coin supérieur gauche du portail Azure. 

1. Dans la liste des résultats, sélectionnez  **Analytics** > **Tâche Stream Analytics** . 

1. Renseignez la page de la tâche Stream Analytics avec les détails nécessaires, tels que le nom, la région et l’échelle. 

1. Activez la case à cocher *Sécuriser tous les actifs de données privés nécessaires à ce travail dans mon compte de stockage*.

1. Sélectionnez un compte de stockage dans votre abonnement. Notez qu’il n’est pas possible de modifier ce paramètre pendant le cycle de vie de la tâche. Vous ne pouvez pas non plus ajouter cette option une fois la tâche créée.

1. Pour vous authentifier avec une chaîne de connexion, sélectionnez **Chaîne de connexion** dans la liste déroulante Mode d’authentification. La clé du compte de stockage est automatiquement renseignée depuis votre abonnement.

   ![Paramètres du compte de stockage de données privées](./media/data-protection/storage-account-create.png)

1. Pour vous authentifier avec une identité managée (préversion), sélectionnez **Identité managée** dans la liste déroulante Mode d’authentification. Si vous optez pour l’identité managée, vous devez ajouter votre tâche Stream Analytics à la liste de contrôle d’accès du compte de stockage. Si vous n’accordez pas d’accès à votre travail, celui-ci ne pourra effectuer aucune opération. Pour plus d’informations sur l’octroi d’accès, consultez [Utiliser Azure RBAC pour attribuer à une identité managée un accès à une autre ressource](../active-directory/managed-identities-azure-resources/howto-assign-access-portal.md#use-azure-rbac-to-assign-a-managed-identity-access-to-another-resource).

   :::image type="content" source="media/data-protection/storage-account-create-msi.png" alt-text="Paramètres de compte de stockage de données privées avec authentification par identité managée":::

## <a name="private-data-assets-that-are-stored-by-stream-analytics"></a>Ressources de données privées stockées par Stream Analytics

Toutes les données privées requises que Stream Analytics doit conserver son stockées dans votre compte de stockage. Voici quelques exemples de ressources de données privées : 

* Requêtes que vous avez créées et configurations associées  

* Fonctions définies par l’utilisateur 

* Points de contrôle nécessaires au runtime Stream Analytics

* Captures instantanées de données de référence 

Les détails de connexion de vos ressources, que votre tâche Stream Analytics utilise, sont également stockés. Chiffrez votre compte de stockage pour sécuriser toutes vos données. 

Pour vous aider à répondre à vos obligations de conformité dans n’importe quel secteur ou environnement réglementé, apprenez-en davantage sur les [offres de conformité de Microsoft](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942). 

## <a name="enables-data-residency"></a>Active la résidence des données 
Vous pouvez utiliser cette fonctionnalité pour appliquer les exigences en matière de résidence des données que vous pouvez avoir en fournissant un compte de stockage en conséquence.

## <a name="known-issues"></a>Problèmes connus
Il existe un problème connu où un travail qui utilise une clé gérée par le client connaît des échecs lors de l’utilisation d’une identité managée pour s’authentifier auprès de toutes les entrées ou sorties. 

## <a name="next-steps"></a>Étapes suivantes

* [Création d’un compte de stockage Azure](../storage/common/storage-account-create.md)
* [Comprendre les entrées d’Azure Stream Analytics](stream-analytics-add-inputs.md)
* [Concepts de point de contrôle et de relecture dans les tâches Azure Stream Analytics](stream-analytics-concepts-checkpoint-replay.md)
* [Utilisation de données de référence pour effectuer des recherches dans Stream Analytics](stream-analytics-use-reference-data.md)
