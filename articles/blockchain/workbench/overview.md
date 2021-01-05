---
title: Vue d’ensemble d’Azure Blockchain Workbench (préversion)
description: Vue d’ensemble d’Azure Blockchain Workbench (préversion) et de ses fonctionnalités.
ms.date: 05/22/2020
ms.topic: overview
ms.reviewer: brendal
ms.openlocfilehash: fbd6be3907dbd10b003d065dfb14031a0e378478
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87003143"
---
# <a name="what-is-azure-blockchain-workbench"></a>Qu’est-ce qu’Azure Blockchain Workbench ?

La préversion d’Azure Blockchain Workbench est une collection de services et de fonctionnalités Azure conçus pour vous aider à créer et à déployer des applications blockchain afin de partager des processus métier et des données avec d’autres organisations. Azure Blockchain Workbench fournit la génération de modèles automatique d’infrastructure pour générer des applications blockchain permettant aux développeurs de se concentrer sur la création d’une logique métier et de contrats intelligents. Il facilite aussi la création d’applications blockchain en intégrant plusieurs services et fonctionnalités Azure pour vous aider à automatiser les tâches de développement courantes.

[!INCLUDE [Preview note](./includes/preview.md)]

## <a name="create-blockchain-applications"></a>Créer des applications blockchain

Avec Blockchain Workbench, vous pouvez définir des applications blockchain en utilisant la configuration et l’écriture de code de contrat intelligent. Vous pouvez accélérer le développement d’applications blockchain et vous concentrer sur la définition de votre contrat et l’écriture de logique métier au lieu de la génération de modèles automatique et la configuration des services pris en charge.

## <a name="manage-applications-and-users"></a>Gérer les applications et les utilisateurs

Azure Blockchain Workbench fournit une application web et des API REST pour gérer les utilisateurs et les applications blockchain. Les administrateurs de Blockchain Workbench peuvent gérer l’accès à l’application et affecter des rôles d’application à vos utilisateurs. Les utilisateurs d’Azure AD sont automatiquement mappés aux membres dans l’application.

## <a name="integrate-blockchain-with-applications"></a>Intégrer le blockchain avec des applications

Vous pouvez utiliser les API REST Blockchain Workbench et les API basées sur message pour l’intégration avec des systèmes existants. Les API fournissent une interface pour permettre le remplacement ou l’utilisation de plusieurs technologies de comptabilité distribuées, d’offres de stockage et de base de données.

Blockchain Workbench peut transformer les messages envoyés à son API basée sur les messages pour générer des transactions dans un format attendu par l’API native de cette blockchain.  Workbench peut signer et acheminer les transactions vers le blockchain approprié. 

Workbench remet automatiquement des événements à Service Bus et Event Grid pour envoyer des messages aux consommateurs en aval. Les développeurs peuvent effectuer l’intégration avec l’un de ces systèmes de messagerie pour diriger les transactions et examiner les résultats.

## <a name="deploy-a-blockchain-network"></a>Déployer un réseau blockchain

Azure Blockchain Workbench simplifie la configuration du consortium du réseau blockchain en tant que solution préconfigurée avec un modèle de solution Azure Resource Manager. Le modèle fournit un déploiement simplifié qui déploie tous les composants nécessaires à l’exécution d’un consortium. Aujourd'hui, Blockchain Workbench prend en charge Ethereum.

## <a name="use-active-directory"></a>Utiliser Active Directory

Avec les protocoles blockchain existants, les identités du blockchain sont représentées sous forme d’une adresse sur le réseau. Azure Blockchain Workbench élimine l’identité du blockchain en l’associant à une identité Active Directory, simplifiant ainsi la génération d’applications d’entreprise avec des identités Active Directory.

## <a name="synchronize-on-chain-data-with-off-chain-storage"></a>Synchroniser les données on-chain avec le stockage off-chain

Azure Blockchain Workbench facilite l’analyse de données et d’événements du blockchain en synchronisant automatiquement les données sur le blockchain avec le stockage off-chain. Au lieu d’extraire les données directement à partir du blockchain, vous pouvez interroger des systèmes de base de données off-chain, comme SQL Server. Les utilisateurs finaux qui effectuent des tâches d'analyse des données ne doivent pas nécessairement posséder de connaissances en matière de blockchains.

## <a name="support-and-feedback"></a>Support et commentaires

Pour vous tenir informé sur Azure Blockchain, visitez le [blog Azure Blockchain](https://azure.microsoft.com/blog/topics/blockchain/). Vous y trouverez les toutes dernières offres du service Blockchain ainsi que diverses informations de l’équipe technique d’Azure Blockchain.

Pour faire des commentaires sur le produit ou suggérer de nouvelles fonctionnalités, postez votre idée ou votez pour une autre idée sur le [forum de commentaires Azure pour Blockchain](https://aka.ms/blockchainuservoice).

### <a name="community-support"></a>Support de la communauté pour les objets blob

Communiquez avec les ingénieurs Microsoft et les experts de la communauté Azure Blockchain.

* [Page de questions Microsoft Q&A sur Azure Blockchain Workbench](/answers/topics/azure-blockchain-workbench.html)
* [Microsoft Tech Community](https://techcommunity.microsoft.com/t5/Blockchain/bd-p/AzureBlockchain)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-blockchain-workbench)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Architecture Azure Blockchain Workbench](architecture.md)
