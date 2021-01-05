---
title: Modèle de tarification d’Azure Cosmos DB
description: Cet article explique le modèle de tarification d’Azure Cosmos DB et comment il simplifie la planification de coûts.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/19/2020
ms.openlocfilehash: 978a2909202e01d8c58ba9253bfe0b3996b72d56
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93100879"
---
# <a name="pricing-model-in-azure-cosmos-db"></a>Modèle de tarification dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Le modèle de tarification d’Azure Cosmos DB simplifie la gestion et la planification des coûts. Avec Azure Cosmos DB, vous payez pour les opérations que vous effectuez sur la base de données et pour le stockage utilisé par vos données.

- **Opérations de base de données**  : la façon dont vous êtes facturé pour vos opérations de base de données dépend du type de compte Azure Cosmos que vous utilisez.

  - **Débit approvisionné**  : le [débit approvisionné](set-throughput.md) (également appelé débit réservé) fournit des performances élevées quelle que soit l’échelle. Vous spécifiez le débit dont vous avez besoin en [unités de requête](request-units.md) par seconde (RU/s) et Azure Cosmos DB dédie les ressources nécessaires pour fournir le débit configuré. Vous pouvez [approvisionner le débit sur une base de données ou sur un conteneur](set-throughput.md). En fonction des besoins de votre charge de travail, vous pouvez augmenter ou diminuer le débit à tout moment ou utiliser la [mise à l’échelle automatique](provision-throughput-autoscale.md) (bien qu’un débit minimum soit requis sur une base de données ou un conteneur pour garantir les contrats SLA). Vous payez à l’heure le débit provisionné maximal pour une heure donnée.

   > [!NOTE]
   > Comme le modèle de débit approvisionné consacre des ressources à votre conteneur ou à votre base de données, le débit approvisionné vous est facturé même si vous n’exécutez aucune charge de travail.

  - **Serverless**  : en mode [serverless](serverless.md), vous n’avez pas besoin d’approvisionner un débit lors de la création de ressources dans votre compte Azure Cosmos. À la fin de votre période de facturation, vous êtes facturé pour la quantité d’unités de requête consommées par vos opérations de base de données.

- **Stockage** : un tarif fixe vous est facturé pour la quantité totale de stockage (exprimée en Go) que vos données et index ont utilisée pendant une heure donnée. Le stockage est facturé en fonction de la consommation, de sorte que vous n’avez pas à réserver de stockage à l’avance. Vous ne payez que le stockage que vous consommez.

Le modèle de tarification dans Azure Cosmos DB est cohérent sur toutes les API. Pour plus d’informations, consultez la [page de tarification d’Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/), [Comprendre vos factures Azure Cosmos DB](understand-your-bill.md) et [En quoi le modèle de tarification Azure Cosmos DB est rentable pour les clients](total-cost-ownership.md).

Si vous déployez votre compte Azure Cosmos DB dans une région non gouvernementale aux États-Unis, il y a un prix minimal du débit sur base de données et sur conteneur est mode de débit approvisionné. Il n’existe pas de prix minimal en mode serverless. Le tarif varie en fonction de la région. Pour connaître les dernières informations tarifaires, consultez la [page des tarifs Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/).

## <a name="try-azure-cosmos-db-for-free"></a>Essayez gratuitement Azure Cosmos DB

Azure Cosmos DB offre de nombreuses possibilités d’essai gratuit aux développeurs. Ces options sont les suivantes :

* **Niveau gratuit d’Azure Cosmos DB**  : Le niveau gratuit d’Azure Cosmos DB facilite le démarrage, le développement et le test de vos applications, voire l’exécution de petites charges de travail de production, gratuitement. Quand le niveau gratuit est activé sur un compte, vous obtenez gratuitement 400 RU/s et 5 Go de stockage pendant toute la durée de vie du compte. Vous pouvez avoir un seul compte Azure Cosmos DB de niveau gratuit par abonnement Azure et vous devez le choisir à la création du compte. Pour commencer, [créez un nouveau compte dans le portail Azure avec l’option tarifaire gratuite activée](create-cosmosdb-resources-portal.md) ou utilisez un modèle [ARM](./manage-with-templates.md#free-tier).

* **Compte gratuit Azure**  : Azure propose un [niveau gratuit](https://azure.microsoft.com/free/) avec 200 $ de crédit Azure pendant les 30 premiers jours et une quantité limitée de services gratuits pendant 12 mois. Pour plus d’informations, consultez la page [Compte Azure gratuit](../cost-management-billing/manage/avoid-charges-free-account.md). L’essai d’Azure Cosmos DB fait partie du compte gratuit Azure. Spécialement pour Azure Cosmos DB, ce compte gratuit offre 5 Go de stockage et 400 RU/s de débit approvisionné pour toute l’année.

* **Essayez gratuitement Azure Cosmos DB**  : Azure Cosmos DB propose une expérience à durée limitée pour essayer Azure Cosmos DB pour les comptes gratuits. Vous pouvez créer un compte Azure Cosmos DB, créer une base de données et des collections et exécuter un exemple d’application en utilisant les didacticiels et guides de démarrage rapide. Vous pouvez exécuter l’exemple d’application sans abonnement à un compte Azure ou avec votre carte de crédit. [Essai gratuit d'Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) offre Azure Cosmos DB pendant un mois, avec la possibilité de renouveler votre compte autant de fois que vous le souhaitez.

* **Émulateur Azure Cosmos DB**  : l’émulateur Azure Cosmos DB fournit un environnement local qui émule le service Azure Cosmos DB à des fins de développement. L’émulateur est disponible gratuitement et simule avec une grande fidélité le service cloud. L’émulateur Azure Cosmos DB vous permet de développer et de tester vos applications localement, sans créer d’abonnement Azure et sans frais. Vous pouvez développer vos applications en local à l’aide de l’émulateur avant de passer en production. Une fois que vous êtes satisfait du fonctionnement de l’application dans l’émulateur, vous pouvez commencer à utiliser le compte Azure Cosmos DB dans le cloud et faire des économies. Pour plus d’informations sur l’émulateur, consultez l’article [Utilisation d’Azure Cosmos DB pour le développement et le test](local-emulator.md) pour plus d’informations.

## <a name="pricing-with-reserved-capacity"></a>Tarification avec la capacité réservée

La [capacité de réserve](cosmos-db-reserved-capacity.md) Azure Cosmos DB vous permet de faire des économies lorsque vous utilisez le mode de débit approvisionné grâce au prépaiement des ressources Azure Cosmos DB sur un an ou trois ans. Vous pouvez considérablement réduire vos coûts avec les engagements à l’avance sur un ou trois ans et économiser entre 20 et 65 % par rapport à la tarification standard. La capacité réservée d’Azure Cosmos DB réservée permet de réduire les coûts en prépayant le débit approvisionné (RU/s) sur une période de un à trois ans, et vous bénéficiez d’une remise sur le débit approvisionné. 

La capacité réservée permet de bénéficier d’une remise, sans affecter l’état d’exécution de vos ressources Azure Cosmos DB. La capacité réservée est disponible pour toutes les API, ce qui comprend MongoDB, Cassandra, SQL, Gremlin et Table Azure, dans toutes les régions du monde. Vous retrouverez plus d’informations sur la capacité réservée dans l’article [Prépayer des ressources Azure Cosmos DB avec une capacité réservée](cosmos-db-reserved-capacity.md) et sur l’achat de capacité réservée dans le [portail Azure](https://portal.azure.com/).

## <a name="next-steps"></a>Étapes suivantes

Vous retrouverez plus d’informations sur l’optimisation des coûts pour vos ressources Azure Cosmos DB dans les articles suivants :

* En savoir plus sur l’[optimisation pour le développement et le test](optimize-dev-test.md)
* En savoir plus sur les [factures Azure Cosmos DB](understand-your-bill.md)
* En savoir plus sur l’[optimisation du coût du débit](optimize-cost-throughput.md)
* En savoir plus sur l’[optimisation du coût de stockage](optimize-cost-storage.md)
* En savoir plus sur l’[optimisation du coût des lectures et écritures](optimize-cost-reads-writes.md)
* En savoir plus sur l’[optimisation du coût des requêtes](./optimize-cost-reads-writes.md)
* En savoir plus sur l’[optimisation du coût des comptes Cosmos sur plusieurs régions](optimize-cost-regions.md)
* En savoir plus sur la [capacité de réserve Azure Cosmos DB](cosmos-db-reserved-capacity.md)
* En savoir plus sur l'[émulateur Azure Cosmos DB](local-emulator.md)