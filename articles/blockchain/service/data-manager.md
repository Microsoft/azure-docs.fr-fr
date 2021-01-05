---
title: Qu’est-ce que Blockchain Data Manager pour Azure Blockchain Service ?
description: Blockchain Data Manager capture, transforme et fournit des données blockchain à des rubriques Event Grid.
ms.date: 11/04/2019
ms.topic: conceptual
ms.reviewer: chroyal
ms.openlocfilehash: 78c93880007259267b26cf53e93de722be1c7323
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77209441"
---
# <a name="what-is-blockchain-data-manager-for-azure-blockchain-service"></a>Qu’est-ce que Blockchain Data Manager pour Azure Blockchain Service ?

Blockchain Data Manager capture, transforme et fournit des données de transaction Azure Blockchain Service aux rubriques Azure Event Grid proposant une intégration de registre blockchain évolutive et fiable aux services Azure.

Dans la plupart des scénarios blockchain d’entreprise, un registre blockchain constitue une partie d’une solution. Par exemple, pour transférer une ressource d’une entité à une autre, vous avez besoin d’un mécanisme pour l’envoi de la transaction. Vous avez ensuite besoin d’un autre mécanisme pour lire les données du registre afin de garantir que la transaction a été effectuée et acceptée. Par la suite, les modifications apportées à l’état sont intégrées à votre solution de bout en bout. Dans cet exemple, si vous écrivez un contrat intelligent pour transférer des ressources, vous pouvez utiliser Blockchain Data Manager pour intégrer des magasins de données et applications hors chaîne. Pour l’exemple de transfert de ressources, lorsqu’une ressource est transférée sur le blockchain, les modifications d’état des événements et de la propriété sont fournies par Blockchain Data Manager via Event Grid. Vous pouvez ensuite utiliser plusieurs gestionnaires d’événements pour Event Grid, afin de stocker des données blockchain hors chaîne ou réagir aux modifications d’état en temps réel.

Blockchain Data Manager exécute trois fonctions principales : capture, transformation et fourniture.

![Fonctions de Blockchain Data Manager](./media/data-manager/functions.png)

## <a name="capture"></a>Capture

Chaque instance de Blockchain Data Manager se connecte à un nœud de transaction de membre Azure Blockchain Service. Seuls les utilisateurs ayant accès au nœud de transaction peuvent créer une connexion garantissant un contrôle d’accès approprié aux données client. Une instance de Blockchain Data Manager capture de manière fiable tous les blocs bruts et toutes les données de transaction brutes à partir du nœud de transaction. En outre, elle peut être mise à l’échelle pour prendre en charge des charges de travail d’entreprise.

## <a name="transform"></a>Transformer

Vous pouvez utiliser Blockchain Data Manager pour décoder l’état des événements et de la propriété en configurant des applications de contrat intelligent dans Blockchain Data Manager. Pour ajouter un contrat intelligent, vous fournissez l’ABI et le bytecode du contrat. Blockchain Data Manager utilise les artefacts du contrat intelligent pour décoder et détecter les adresses du contrat. Après avoir ajouté l’application blockchain à l’instance, Blockchain Data Manager détecte dynamiquement l’adresse du contrat intelligent lorsque le contrat intelligent est déployé dans le consortium et envoie l’état de propriété et d’événement décodé aux destinations configurées.

## <a name="deliver"></a>Remettre

Blockchain Data Manager prend en charge plusieurs connexions sortantes de rubrique Event Grid pour une instance de Data Manager Blockchain donnée. Vous pouvez envoyer des données blockchain à une ou plusieurs destinations. À l’aide de Blockchain Data Manager, vous pouvez créer une solution de publication de données évolutive basée sur des événements pour tout déploiement blockchain.

## <a name="configuration-options"></a>Options de configuration

Vous pouvez configurer Blockchain Data Manager pour répondre aux besoins de votre solution. Vous pouvez par exemple approvisionner :

* Une seule instance de Blockchain Data Manager pour un membre d’Azure Blockchain Service.
* Une instance de Blockchain Data Manager par nœud de transaction Azure Blockchain Service. Par exemple, les nœuds de transaction privés peuvent avoir leur propre instance de Blockchain Data Manager pour assurer la confidentialité.
* Une instance de Blockchain Data Manager peut prendre en charge plusieurs connexions de sortie. Une instance de Blockchain Data Manager peut être utilisée pour gérer tous les points d’intégration de la publication de données pour un membre Azure Blockchain Service.

## <a name="next-steps"></a>Étapes suivantes

Essayez de [créer une instance de Blockchain Data Manager](data-manager-portal.md) pour un membre d’Azure Blockchain Service.
