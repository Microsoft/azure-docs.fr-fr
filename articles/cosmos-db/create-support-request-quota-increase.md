---
title: Comment demander une augmentation du quota pour les ressources Azure Cosmos DB
description: Découvrez comment demander une augmentation de quota pour les ressources Azure Cosmos DB. Vous allez également apprendre à activer un abonnement pour accéder à une région.
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.topic: how-to
ms.date: 07/17/2020
ms.openlocfilehash: e7ec71220b75647e789508c760e50957b3b497fa
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93090033"
---
# <a name="how-to-request-quota-increase-for-azure-cosmos-db-resources"></a>Comment demander une augmentation du quota pour les ressources Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Les ressources d’Azure Cosmos DB ont des [quotas/limites par défaut](concepts-limits.md). Toutefois, il peut arriver que votre charge de travail nécessite un quota plus important que la valeur par défaut. Dans ce cas, vous devez contacter l’équipe Azure Cosmos DB pour demander une augmentation du quota. Cet article explique comment demander une augmentation de quota pour les ressources Azure Cosmos DB. Vous allez également apprendre à activer un abonnement pour accéder à une région.

## <a name="create-a-new-support-request"></a>Créer une demande de support

Pour demander une augmentation du quota, vous devez créer une nouvelle demande de support avec les détails de votre charge de travail. L’équipe Azure Cosmos DB évaluera ensuite votre demande et l’approuvera. Suivez les étapes ci-dessous pour créer une demande de support à partir du portail Azure :

1. Connectez-vous au portail Azure.

1. Dans le menu de gauche, sélectionnez **Aide et support** , puis sélectionnez **Nouvelle demande de support**.

1. Sous l’onglet **Informations de base** , renseignez les informations suivantes :

   * Pour **Type de problème** , sélectionnez **Limites du service et des abonnements (quotas)**
   * Pour l’option **Abonnement** , sélectionnez l’abonnement dont vous voulez augmenter le quota.
   * Dans **Type de quota** , sélectionnez **Cosmos DB**.

   :::image type="content" source="./media/create-support-request-quota-increase/create-quota-increase-request.png" alt-text="Créer une demande de support Cosmos DB pour augmenter le quota":::

1. Sous l’onglet **Détails** , entrez les détails correspondant à votre demande de quota. Les informations fournies dans cet onglet permettent de mieux évaluer votre problème et d’aider l’ingénieur du support à résoudre le problème.

1. Remplissez les informations suivantes dans ce formulaire :

   * **Description**  : Fournissez une brève description de votre demande, telle que votre charge de travail et pourquoi les valeurs par défaut ne sont pas suffisantes. En fonction du type de ressource pour lequel vous souhaitez augmenter le quota, il est obligatoire de fournir les détails suivants dans le champ **Description**  :

     **Demandes de régions** Si votre demande correspond à l’ajout d’une région à la liste verte, veillez à fournir les valeurs suivantes :

        * Nom de la région
        * Identifiant d’abonnement

     **Demandes de limite de débit** Si votre demande correspond à une augmentation de quota pour le débit, assurez-vous de fournir les valeurs suivantes :

        * Nom de la base de données
        * Identifiant d’abonnement
        * Nouvelle limite de débit

     **Demandes de limite de compte de base de données** Si votre demande correspond à une augmentation de quota pour le nombre de comptes de base de données dans un abonnement, assurez-vous de fournir les valeurs suivantes :

       * Identifiant d’abonnement
       * Nouvelle limite de comptes de base de données

   * **Chargement de fichiers**  : Chargez les fichiers de diagnostic ou tout autre fichier qui vous semble pertinent pour la demande de support. Pour en savoir plus sur les instructions de chargement de fichiers, consultez l’article du [support Azure]( ../azure-portal/supportability/how-to-manage-azure-support-request.md#upload-files).

   * **Gravité** : Choisissez l’un des niveaux de gravité disponibles en fonction de l’impact sur l’activité.

   * **Moyen de communication préféré**  : Vous pouvez choisir d’être contacté par **e-mail** ou par **téléphone**.

1. Remplissez les autres informations du formulaire, telles que votre disponibilité, la langue du support, vos coordonnées, votre adresse e-mail et votre numéro de téléphone.

1. Sélectionnez **Suivant : Vérifier + créer**. Vérifiez les informations fournies et sélectionnez **Créer** pour créer une demande de support.

L’équipe de support Azure Cosmos DB évaluera votre demande et vous recontactera dans les 24 heures.

## <a name="next-steps"></a>Étapes suivantes

* Consultez les [quotas de service par défaut d’Azure Cosmos DB](concepts-limits.md).
