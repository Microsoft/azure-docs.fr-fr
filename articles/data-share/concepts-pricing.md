---
title: Présentation de la tarification d'Azure Data Share
description: Découvrez sur quoi repose la tarification d'Azure Data Share
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: conceptual
ms.date: 08/11/2020
ms.openlocfilehash: 1c2c58e206a70a3801c712df3b5582409712d3c8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88136608"
---
# <a name="understand-azure-data-share-pricing"></a>Présentation de la tarification d'Azure Data Share

Pour le partage basé sur une capture instantanée, Azure Data Share facture la capture instantanée du jeu de données et l'exécution de la capture instantanée. Cet article explique comment estimer la capture instantanée du jeu de données et l'exécution de la capture instantanée à l'aide des informations de l'historique de la capture instantanée. Actuellement, le fournisseur de données est facturé pour la capture instantanée du jeu de données et l'exécution de la capture instantanée.

## <a name="dataset-snapshot"></a>Capture instantanée du jeu de données

La capture instantanée du jeu de données est l'opération qui consiste à déplacer le jeu de données de sa source vers sa destination. Lorsqu'une capture instantanée est effectuée pour un partage, la capture instantanée de chaque jeu de données du partage est une opération de capture instantanée de jeu de données. Suivez les étapes ci-dessous pour afficher la liste des captures instantanées de jeu de données. 

1. Sur le portail Azure, accédez à votre ressource Data Share.

1. Sélectionnez un partage dans Partage envoyé ou Partage reçu.

1. Cliquez sur l'onglet **Historique**.

1. Cliquez sur l'Heure de début d'une capture instantanée.
 
    ![Historique des instantanés](./media/concepts/concepts-pricing/pricing-snapshot-history.png "Historique des instantanés") 

1. Affichez la liste des opérations de capture instantanée de jeu de données. Chaque ligne correspond à une opération de capture instantanée de jeu de données. Dans cet exemple, il y a deux captures instantanées de jeu de données.

    ![Opération de capture instantanée de jeu de données](./media/concepts/concepts-pricing/pricing-dataset-snapshot.png "Opération de capture instantanée de jeu de données")

## <a name="snapshot-execution"></a>Exécution de la capture instantanée

L'exécution de la capture instantanée inclut les ressources nécessaires pour déplacer un jeu de données de la source vers la destination. Pour chaque opération de capture instantanée de jeu de données, l'exécution de la capture instantanée est calculée comme suit : nombre de vCores multiplié par la durée de l'instantané. Les frais sont calculés à la minute près et arrondis au chiffre supérieur. Le nombre de vCores est sélectionné de manière dynamique en fonction de la paire source-cible et du modèle de données. Suivez les étapes ci-dessous pour afficher l'heure de début de la capture instantanée, l'heure de fin et les vCores utilisés pour une opération de capture instantanée de jeu de données.

1. Sur le portail Azure, accédez à votre ressource Data Share.

1. Sélectionnez un partage dans Partage envoyé ou Partage reçu.

1. Cliquez sur l'onglet **Historique**.

1. Cliquez sur l'Heure de début d'une capture instantanée.

    ![Historique des instantanés](./media/concepts/concepts-pricing/pricing-snapshot-history.png "Historique des instantanés") 

1. Cliquez sur l'État d'une opération de capture instantanée de jeu de données.

    ![État de la capture instantanée de jeu de données](./media/concepts/concepts-pricing/pricing-snapshot-status.png "État de la capture instantanée du jeu de données")

1. Affichez l'heure de début, l'heure de fin et les vCores utilisés pour cette opération de capture instantanée de jeu de données. 

    ![Exécution de la capture instantanée](./media/concepts/concepts-pricing/pricing-snapshot-execution.png "Exécution de la capture instantanée")

## <a name="next-steps"></a>Étapes suivantes

- Procurez-vous les dernières informations relatives à la tarification - [Tarification d'Azure Data share](https://azure.microsoft.com/pricing/details/data-share/)
- Utilisez la calculatrice de prix Azure pour estimer le coût - [Calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/)
