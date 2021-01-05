---
title: Créer un point de terminaison privé managé à connecter aux résultats de votre source de données
description: Cet article explique comment créer un point de terminaison privé managé pour vos sources de données à partir d’un espace de travail Azure synapse.
author: RonyMSFT
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 04/15/2020
ms.author: ronytho
ms.reviewer: jrasnick
ms.openlocfilehash: e0309b4c96b2ae25eb568e390717ba76cfd84fa5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461319"
---
# <a name="create-a-managed-private-endpoint-to-your-data-source"></a>Créer un point de terminaison privé managé pour votre source de données

Cet article explique comment créer un point de terminaison privé managé pour votre sources de données dans Azure. Pour plus d’informations, consultez [Points de terminaison privés managés](./synapse-workspace-managed-private-endpoints.md).

## <a name="step-1-open-your-azure-synapse-workspace-in-azure-portal"></a>Étape 1 : Ouvrir votre espace de travail Azure Synapse dans le portail Azure

Vous pouvez créer un point de terminaison privé managé pour votre source de données à partir d’Azure Synapse Studio. Sélectionnez l’onglet **Vue d’ensemble** dans Portail Azure et sélectionnez **Ouvrir** sur la carte Ouvrir Synapse Studio dans la section Démarrage.

## <a name="step-2-navigate-to-the-managed-virtual-networks-tab-in-synapse-studio"></a>Étape 2 : Accéder à l’onglet Réseaux virtuels gérés dans Synapse Studio

Dans Azure Synapse Studio, dans le volet de navigation gauche, sélectionnez l’onglet **Gérer**. Sélectionnez **Points de terminaison privés gérés** puis sélectionnez **+ Nouveau**.
![Créer un point de terminaison privé managé](./media/how-to-create-managed-private-endpoints/managed-private-endpoint-2.png)

## <a name="step-3-select-the-data-source-type"></a>Étape 3 : Sélectionner le type de source de données

Sélectionnez le type de source de données. Dans ce cas, la source de données cible est un compte ADLS Gen2. Sélectionnez **Continuer**.
![Sélectionner un type de source de données cible](./media/how-to-create-managed-private-endpoints/managed-private-endpoint-3.png)

## <a name="step-4-enter-information-about-the-data-source"></a>Étape 4 : Entrer des informations sur la source de données

Dans la fenêtre suivante, entrez les informations relatives à la source de données. Dans cet exemple, nous créons un point de terminaison privé managé pour un compte ADLS Gen2. Entrez un **Nom** pour le point de terminaison privé managé. Fournissez un **Abonnement Azure** et un **Nom de compte de stockage**. Sélectionnez **Create** (Créer).
![Entrer les détails de la source de données cible](./media/how-to-create-managed-private-endpoints/managed-private-endpoint-4.png)

## <a name="step-5-verify-that-your-managed-private-endpoint-was-successfully-created"></a>Étape 5 : Vérifier que votre point de terminaison privé managé a été correctement créé

Une fois la demande envoyée, son état s’affiche. Pour vérifier que la création de votre point de terminaison privé managé a été correctement créée, vérifiez son *État d’approvisionnement*. Il se peut que vous deviez attendre 1 minute et sélectionner **Actualiser** pour mettre à jour l’état d’approvisionnement. Vous pouvez voir que le point de terminaison privé managé pour le compte ADLS Gen2 a été correctement créé.

Vous pouvez également voir que l’*État d’approbation* est *En attente*. Le propriétaire de la ressource cible peut approuver ou refuser la demande de connexion du point de terminaison privé. Si le propriétaire approuve la demande de connexion du point de terminaison privé, un lien privé est établi. En cas de refus, aucun lien privé n’est établi.
![État de la demande de création de point de terminaison privé managé](./media/how-to-create-managed-private-endpoints/managed-private-endpoint-5.png)

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [Points de terminaison privés managés](./synapse-workspace-managed-private-endpoints.md)