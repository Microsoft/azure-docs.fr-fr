---
title: Vue d’ensemble du développement d’Azure Blockchain Service
description: Introduction au développement de solutions sur Azure Blockchain Service.
ms.date: 03/26/2020
ms.topic: conceptual
ms.reviewer: janders
ms.openlocfilehash: 4a98e5ea025894303bc5c77bba0c6154a08315f2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80348382"
---
# <a name="azure-blockchain-service-development-overview"></a>Vue d’ensemble du développement d’Azure Blockchain Service

Avec Azure Blockchain Service, vous pouvez créer des réseaux blockchain de consortium permettant l’exécution de scénarios d’entreprise comme le suivi des biens, les jetons numériques, la fidélité et les récompenses, finances de la chaîne d’approvisionnement et la provenance. Les sections suivantes présentent le développement Azure Blockchain Service pour l’implémentation de solutions blockchain d’entreprise.

## <a name="connecting-to-azure-blockchain-service"></a>Connexion à Azure Blockchain Service

Il existe différents types de clients pour les réseaux blockchain, y compris les nœuds complets, les nœuds lumineux et les clients distants. Azure Blockchain Service génère un réseau blockchain qui inclut des nœuds. Vous pouvez utiliser différents clients comme passerelle vers Azure Blockchain Service pour le développement de solutions blockchain. Azure Blockchain Service offre l’authentification de base ou une clé d’accès comme point de terminaison de développement. Les éléments suivants sont des clients vous permettant de vous connecter.

### <a name="visual-studio-code"></a>Visual Studio Code

Vous pouvez vous connecter à des membres de consortium au moyen de l’extension Visual Studio Code Azure Blockchain Development Kit. Une fois connecté à un consortium, vous pouvez compiler, générer et déployer des contrats intelligents sur un membre de consortium Azure Blockchain Service.

Pour développer des solutions de blockchain d’entreprise sophistiquées, une infrastructure de développement est nécessaire pour se connecter aux différents réseaux blockchain et gérer le cycle de vie d’un contrat intelligent. La plupart des projets interagissent avec au moins deux nœuds de blockchain. Les développeurs utilisent un blockchain local pendant le développement. Quand l’application est prête pour le test ou la publication, le développeur la déploie sur un réseau blockchain. Par exemple, le principal réseau Ethereum public ou Azure Blockchain Service. Azure Blockchain Development Kit pour l’extension Ethereum dans Visual Studio Code utilise Truffle. Truffle est une infrastructure de développement blockchain populaire pour écrire, compiler, déployer et tester des applications décentralisées sur des blockchains Ethereum. Vous pouvez également considérer Truffle comme une infrastructure qui tente d’intégrer en toute transparence le développement de contrats intelligents et le développement web traditionnel.

Pour plus d’informations, consultez [Démarrage rapide : Utiliser Visual Studio Code pour se connecter à un réseau du consortium Azure Blockchain Service](connect-vscode.md).

### <a name="metamask"></a>MetaMask

MetaMask est un portefeuille basé sur un navigateur (client distant), un client RPC et un explorateur de contrat de base. Contrairement aux autres portefeuilles de navigateur, MetaMask injecte une instance web3 dans le contexte JavaScript du navigateur, agissant comme un client RPC qui se connecte à un large éventail de blockchains Ethereum (*mainnet*, *Ropsten testnet*, *Kovan testnet*, nœud RPC local, etc..). Vous pouvez configurer un RPC personnalisé facilement pour qu’il se connecter à Azure Blockchain Service et démarrer le développement de blockchain à l’aide de Remix.

Pour plus d’informations, consultez [Démarrage rapide : Utiliser MetaMask pour vous connecter et déployer un contrat intelligent](connect-metamask.md)

### <a name="geth"></a>Geth

Geth est l’interface de ligne de commande pour l’exécution d’un nœud Ethereum complet implémenté dans Go. Vous n’avez pas besoin de l’exécution complète de nœud, mais vous pouvez lancer sa console interactive qui fournit un environnement d’exécution JavaScript exposant une API JavaScript pour interagir avec Azure Blockchain Service.

Pour plus d’informations, consultez [Démarrage rapide : Utiliser Geth pour effectuer une jonction à un nœud de transaction Azure Blockchain Service](connect-geth.md).

## <a name="ethereum-quorum-private-transactions"></a>Transactions privées Ethereum Quorum

Quorum est un protocole de registre distribué basé sur Ethereum avec une confidentialité des transactions et des contrats ainsi que de nouveaux mécanismes de consensus. Les améliorations clés sur Go-Ethereum sont les suivantes :

* **Confidentialité** - Quorum prend en charge les transactions et les contrats privés via la séparation de l’état public et privé et utilise des échanges de messages chiffrés en peer-to-peer pour un transfert direct de données privées aux participants du réseau.
* **Autres mécanismes de consensus** - Aucun consensus de preuve de travail ou de preuve de participation n’est nécessaire pour un réseau autorisé. Quorum offre plusieurs mécanismes de consensus qui sont conçus pour les chaînes de consortium telles que RAFTING et IBFT.  Azure Blockchain Service utilise le mécanisme de consensus IBFT.
* **Permissions de paires** - Les autorisations de nœud et de paire à l’aide de contrats intelligents garantissent que seuls les tiers connus peuvent se connecter au réseau.
* **Performances plus élevées** - Quorum offre de meilleures performances que Geth publique.

## <a name="block-explorers"></a>Explorateurs de bloc

Les explorateurs de bloc sont des navigateurs de blockchain en ligne qui affichent le contenu des blocs individuels, les données des adresses des transactions et l’historique. Les informations de blocs de base sont disponibles via Azure Monitor dans Azure Blockchain Service. Toutefois, si vous avez besoin d’informations plus détaillées au cours du développement, les explorateurs de blocs peuvent être utiles.  Les explorateurs de bloc suivants fonctionnent avec Azure Blockchain Service :

* [Epirus Azure Blockchain Service Explorer](https://azuremarketplace.microsoft.com/marketplace/apps/blk-technologies.azure-blockchain-explorer-template?tab=Overview) de Web3 Labs
* [BlockScout](https://github.com/Azure-Samples/blockchain/blob/master/ledger/template/ethereum-on-azure/technology-samples/blockscout/README.md)

Vous pouvez également créer votre propre explorateur de bloc à l’aide de Blockchain Data Manager et d’Azure Cosmos DB ; consultez [Tutoriel : Utiliser Blockchain Data Manager pour envoyer des données à Azure Cosmos DB](data-manager-cosmosdb.md).

## <a name="tps-measurement"></a>Mesure TPS

Comme une blockchain est utilisée dans plus de scénarios d’entreprise, la vitesse en transactions par seconde (TPS) est importante pour éviter les goulots d’étranglement et le manque d’efficacité des systèmes. Des taux de transaction élevés peuvent être difficiles à gérer au sein d’une blockchain décentralisée. Une mesure précise des TPS peut-être affectée par différents facteurs tels que le thread du serveur, la taille de la file d’attente des transactions, la latence du réseau et la sécurité. Si vous avez besoin de mesurer la vitesse en TPS au cours du développement, [ChainHammer](https://github.com/drandreaskrueger/chainhammer) est un outil open source populaire.

## <a name="next-steps"></a>Étapes suivantes

Essayez un guide de démarrage consistant à utiliser Azure Blockchain Development Kit pour Ethereum afin d’effectuer une jonction à un consortium dans Azure Blockchain Service.

> [!div class="nextstepaction"]
> [Utiliser Visual Studio Code pour se connecter à Azure Blockchain Service](connect-vscode.md)