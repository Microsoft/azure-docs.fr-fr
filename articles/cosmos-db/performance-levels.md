---
title: Niveaux de performances Azure Cosmos DB retirés
description: Découvrez les niveaux de performances S1, S2 et S3 qui étaient disponibles dans Azure Cosmos DB.
author: SnehaGunda
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 06/04/2018
ms.author: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 4cb07d9d19d85cd8dff9a52eeeb7e173b60f4d6d
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93080768"
---
# <a name="retiring-the-s1-s2-and-s3-performance-levels"></a>Mise hors service des niveaux de performances S1, S2 et S3
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

> [!IMPORTANT] 
> Les niveaux de performances S1, S2 et S3 abordés dans cet article vont être mis hors service et ne sont plus disponibles pour les nouveaux comptes Azure Cosmos DB.

Cet article fournit une vue d’ensemble des niveaux de performances S1, S2 et S3, et explique comment les collections qui les utilisent peuvent être migrées vers des collections à partition unique. Après avoir lu cet article, vous serez en mesure de répondre aux questions suivantes :

- [Pourquoi les niveaux de performances S1, S2 et S3 vont-ils être mis hors service ?](#why-retired)
- [En quoi les collections à partition unique et les collections partitionnées se différencient-elles des niveaux de performances S1, S2 et S3 ?](#compare)
- [Comment assurer un accès ininterrompu à mes données ?](#uninterrupted-access)
- [Qu’est-ce qui va changer au niveau de ma collection suite à la migration ?](#collection-change)
- [Qu’est-ce qui va changer au niveau de ma facturation suite à la migration vers des collections à partition unique ?](#billing-change)
- [Que se passe-t-il si j’ai besoin de plus de 20 Go de stockage ?](#more-storage-needed)
- [Puis-je modifier les niveaux de performances entre S1, S2 et S3 avant la migration planifiée ?](#change-before)
- [Comment procéder à la migration des niveaux de performances S1, S2 et S3 vers des collections à partition unique par moi-même ?](#migrate-diy)
- [Quelles sont les conséquences pour moi en tant que client Contrat Entreprise ?](#ea-customer)

<a name="why-retired"></a>

## <a name="why-are-the-s1-s2-and-s3-performance-levels-being-retired"></a>Pourquoi les niveaux de performances S1, S2 et S3 vont-ils être mis hors service ?

Les niveaux de performances S1, S2 et S3 n’offrent pas la même flexibilité qu’Azure Cosmos DB standard. Avec les niveaux de performances S1, S2 et S3, le débit et la capacité de stockage étaient prédéfinis et n’offraient aucune élasticité. Azure Cosmos DB offre désormais la possibilité de personnaliser votre débit et votre stockage, ce qui vous garantit une plus grande flexibilité vis-à-vis de l’évolution de vos besoins.

<a name="compare"></a>

## <a name="how-do-single-partition-collections-and-partitioned-collections-compare-to-the-s1-s2-s3-performance-levels"></a>En quoi les collections à partition unique et les collections partitionnées se différencient-elles des niveaux de performances S1, S2 et S3 ?

Le tableau suivant compare les options de débit et de stockage disponibles dans les collections à partition unique, les collections partitionnées et les niveaux de performances S1, S2 et S3. Voici un exemple pour la région USA Est 2 :

| Nom du quota  |Collection partitionnée|Collection à partition unique|S1|S2|S3|
|---|---|---|---|---|---|
|Débit maximal|Illimité|10 000 RU/s|250 RU/s|1 000 RU/s|2 500 RU/s|
|Débit minimal|2 500 RU/s|400 RU/s|250 RU/s|1 000 RU/s|2 500 RU/s|
|Stockage maximal|Illimité|20 Go|20 Go|20 Go|20 Go|
|Prix (mensuel)|Débit : 6 USD / 100 RU/s<br><br>Stockage : 0,25 USD/Go|Débit : 6 USD / 100 RU/s<br><br>Stockage : 0,25 USD/Go|25 USD|50 USD|100 USD|

Vous êtes un client Contrat Entreprise ? Si oui, voir [Quelles sont les conséquences pour moi en tant que client Contrat Entreprise ?](#ea-customer)

<a name="uninterrupted-access"></a>

## <a name="what-do-i-need-to-do-to-ensure-uninterrupted-access-to-my-data"></a>Comment assurer un accès ininterrompu à mes données ?

Si vous avez une collection S1, S2 ou S3, vous devez migrer la collection vers une collection à partition unique par programmation [en utilisant le SDK .NET](#migrate-diy). 

<a name="collection-change"></a>

## <a name="how-will-my-collection-change-after-the-migration"></a>Qu’est-ce qui va changer au niveau de ma collection suite à la migration ?

Si vous avez une collection S1, vous pouvez la migrer vers une collection à partition unique avec un débit de 400 RU/s, ce qui correspond au débit le plus bas disponible pour les collections à partition unique. Toutefois, le coût d’un débit de 400 RU/s dans une collection à partition unique est approximativement identique à celui que vous payez pour votre collection S1 dotée d’un débit de 250 RU/s : vous ne payez pas pour les 150 RU/s disponibles supplémentaires.

Si vous avez une collection S1, vous pouvez la migrer vers une collection à partition unique avec 1 000 RU/s. Vous n’observerez aucun changement de niveau de débit.

Si vous avez une collection S3, vous pouvez la migrer vers une collection à partition unique avec 2 500 RU/s. Vous n’observerez aucun changement de niveau de débit.

Dans chacun de ces cas, après avoir migré la collection, vous serez en mesure de personnaliser votre niveau de débit, ou de le mettre à l’échelle selon vos besoins afin de fournir un accès à faible latence à vos utilisateurs. 

<a name="billing-change"></a>

## <a name="how-will-my-billing-change-after-i-migrated-to-the-single-partition-collections"></a>Qu’est-ce qui va changer au niveau de ma facturation suite à la migration vers des collections à partition unique ?

Supposons que vous avez 10 collections S1, d’une capacité de stockage de 1 Go chacune, dans la région USA Est et que vous migrez ces 10 collections S1 vers 10 collections à partition unique dotées d’un débit de 400 RU/s (niveau minimal). Votre facture se présentera comme suit si vous conservez les 10 collections à partition unique pendant un mois complet :

:::image type="content" source="./media/performance-levels/s1-vs-standard-pricing.png" alt-text="Comparaison entre la tarification de 10 collections S1 et de 10 collections utilisant la tarification d’une collection à partition unique" border="false":::

<a name="more-storage-needed"></a>

## <a name="what-if-i-need-more-than-20-gb-of-storage"></a>Que se passe-t-il si j’ai besoin de plus de 20 Go de stockage ?

Que vous disposiez d’une collection avec un niveau de performances S1, S2 ou S3 ou d’une collection à partition unique disposant de 20 Go de stockage disponible, vous pouvez utiliser l’outil de migration de données Azure Cosmos DB pour migrer vos données vers une collection partitionnée bénéficiant d’un stockage quasi illimité. Pour plus d’informations sur les avantages d’une collection partitionnée, voir [Partitionnement et mise à l’échelle dans Azure Cosmos DB](partitioning-overview.md). 

<a name="change-before"></a>

## <a name="can-i-change-between-the-s1-s2-and-s3-performance-levels-before-the-planned-migration"></a>Puis-je modifier les niveaux de performances entre S1, S2 et S3 avant la migration planifiée ?

Seuls les comptes existants avec des niveaux de performances S1, S2 et S3 peuvent être changés et peuvent modifier les niveaux de performances par programmation [via le SDK .NET](#migrate-diy). Si vous remplacez une collection S1, S2 ou S3 par une collection à partition unique, vous ne pouvez plus revenir aux niveaux de performances S1, S2 ou S3.

<a name="migrate-diy"></a>

## <a name="how-do-i-migrate-from-the-s1-s2-s3-performance-levels-to-single-partition-collections-on-my-own"></a>Comment procéder à la migration des niveaux de performances S1, S2 et S3 vers des collections à partition unique par moi-même ?

Vous pouvez effectuer la migration des niveaux de performance S1, S2 et S3 vers des collections à partition unique par programmation [via le SDK .NET](#migrate-diy). Vous pouvez effectuer cette opération vous-même avant la migration planifiée pour tirer parti des options de débit flexibles disponibles pour les collections à partition unique.

### <a name="migrate-to-single-partition-collections-by-using-the-net-sdk"></a>Migrer vers des collections à partition unique avec le SDK .NET

Cette section couvre uniquement la modification du niveau de performances d’une collection à l’aide de [l’API .NET SQL](sql-api-sdk-dotnet.md), mais le processus est similaire pour nos autres SDK.

Voici un extrait de code permettant de remplacer le débit de la collection par 5 000 unités de requête par seconde :
    
```csharp
    //Fetch the resource to be updated
    Offer offer = client.CreateOfferQuery()
                      .Where(r => r.ResourceLink == collection.SelfLink)    
                      .AsEnumerable()
                      .SingleOrDefault();

    // Set the throughput to 5000 request units per second
    offer = new OfferV2(offer, 5000);

    //Now persist these changes to the database by replacing the original resource
    await client.ReplaceOfferAsync(offer);
```

Visitez [MSDN](/dotnet/api/microsoft.azure.documents.client.documentclient) pour afficher des exemples supplémentaires et en savoir plus sur nos méthodes d’offre :

* [**ReadOfferAsync**](/dotnet/api/microsoft.azure.documents.client.documentclient.readofferasync)
* [**ReadOffersFeedAsync**](/dotnet/api/microsoft.azure.documents.client.documentclient.readoffersfeedasync)
* [**ReplaceOfferAsync**](/dotnet/api/microsoft.azure.documents.client.documentclient.replaceofferasync)
* [**CreateOfferQuery**](/previous-versions/azure/dn975114(v=azure.100))

<a name="ea-customer"></a>

## <a name="how-am-i-impacted-if-im-an-ea-customer"></a>Quelles sont les conséquences pour moi en tant que client Contrat Entreprise ?

Les clients Contrat Entreprise bénéficieront d’une protection en matière de tarification jusqu’à la fin de leur contrat actuel.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur la tarification et la gestion des données avec Azure Cosmos DB, explorez les ressources suivantes :

1.  [Partitioning data in Cosmos DB (Partitionnement des données dans Cosmos DB)](partitioning-overview.md). Découvrez la différence entre les conteneurs à partition unique et les conteneurs partitionnés, et bénéficiez de conseils concernant l’implémentation d’une stratégie de partitionnement pour une mise à l’échelle en toute transparence.
2.  [Tarification Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/). Apprenez-en davantage sur le coût de l’approvisionnement du débit et de la consommation du stockage.
3.  [Unités de requête](request-units.md). Découvrez la consommation de débit pour les différents types d’opérations, telles que les opérations de lecture, d’écriture et de requête.