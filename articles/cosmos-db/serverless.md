---
title: Offre serverless basée sur la consommation dans Azure Cosmos DB
description: En savoir plus sur l’offre serverless basée sur la consommation d’Azure Cosmos DB.
author: ThomasWeiss
ms.author: thweiss
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 11/25/2020
ms.openlocfilehash: cabc243c6ba74217873b5b0a5fa51a7cb410512e
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96170828"
---
# <a name="azure-cosmos-db-serverless-preview"></a>Azure Cosmos DB serverless (préversion)
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

> [!IMPORTANT]
> Azure Cosmos DB serverless est actuellement en préversion. Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Cosmos DB serverless vous permet d’utiliser votre compte Azure Cosmos sur la base de la consommation ; dans ce cas, vous êtes facturé uniquement pour les unités de requête consommées par vos opérations de base de données et le stockage consommé par vos données. L’utilisation d’Azure Cosmos DB en mode serverless n’implique pas de frais minimum.

> [!IMPORTANT] 
> Avez-vous des commentaires sur les solutions serverless ? Nous attendons vos remarques ! N’hésitez pas à envoyer un message à l’équipe Azure Cosmos DB serverless : [azurecosmosdbserverless@service.microsoft.com](mailto:azurecosmosdbserverless@service.microsoft.com).

Lors de l’utilisation d’Azure Cosmos DB, chaque opération de base de données a un coût exprimé en [unités de requête](request-units.md). La façon dont vous êtes facturé pour ce coût dépend du type de compte Azure Cosmos que vous utilisez :

- Dans le mode [débit approvisionné](set-throughput.md), vous devez effectuer une validation sur une certaine quantité de débit (exprimée en unités de requête par seconde) approvisionnée sur vos bases de données et conteneurs. Le coût de vos opérations de base de données est ensuite déduit du nombre d’unités de requête disponibles chaque seconde. À la fin de votre période de facturation, vous êtes facturé pour la quantité de débit que vous avez approvisionnée.
- En mode serverless, vous n’avez pas besoin d’approvisionner un débit lors de la création de conteneurs dans votre compte Azure Cosmos. À la fin de votre période de facturation, vous êtes facturé pour le nombre d’unités de requête consommées par vos opérations de base de données.

## <a name="use-cases"></a>Cas d’usage

Azure Cosmos DB serverless est adapté aux scénarios où vous vous attendez à :

- **Trafic faible, intermittent et imprévisible** : Car l’approvisionnement de capacité dans de telles situations n’est pas obligatoire et peut être un coût prohibitif
- **Des performances modérées** : Car les conteneurs serverless ont des [caractéristiques de performances spécifiques](#performance)

Pour ces raisons, Azure Cosmos DB serverless doit être pris en compte pour les types de charges de travail suivants :

- Développement
- Test
- Prototypage
- Preuve de concept
- Application non critique avec un trafic léger

Pour plus d’informations sur la façon de choisir l’offre qui correspond le mieux à votre cas d’utilisation, reportez-vous à l’article [Choisir entre le débit provisionné et le serverless](throughput-serverless.md).

## <a name="using-serverless-resources"></a>Utilisation de ressources serverless

Le serverless est un nouveau type de compte Azure Cosmos, ce qui signifie que vous devez choisir entre un **débit approvisionné** et le **serverless** lors de la création d’un compte. Vous devez créer un nouveau compte serverless pour commencer à utiliser le serverless. Pendant la préversion, le seul moyen pris en charge de créer un nouveau compte serverless consiste à [utiliser le portail Azure](create-cosmosdb-resources-portal.md). La migration de comptes existants vers/à partir du mode serverless n’est pas prise en charge actuellement.

Tout conteneur créé dans un compte serverless est un conteneur serverless. Les conteneurs serverless exposent les mêmes fonctionnalités que les conteneurs créés en mode de débit approvisionné, ce qui vous permet de lire, d’écrire et d’interroger vos données exactement de la même façon. Toutefois, les comptes et les conteneurs serverless ont également des caractéristiques spécifiques :

> [!IMPORTANT]
> Certaines de ces limitations peuvent être atténuées ou supprimées lorsque le serverless sera mis à la disposition générale, et **vos commentaires** nous aideront à décider ! Contactez-nous pour partager vos remarques sur votre expérience serverless : [azurecosmosdbserverless@service.microsoft.com](mailto:azurecosmosdbserverless@service.microsoft.com).

- Un compte serverless ne peut s’exécuter que dans une seule région Azure. Il n’est pas possible d’ajouter des régions Azure supplémentaires à un compte serverless après l’avoir créé.
- Il n’est pas possible d’activer la [fonctionnalité en préversion Synapse Link](synapse-link.md) sur un compte serverless.
- L’approvisionnement de débit n’est pas requis sur les conteneurs serverless ; les instructions suivantes sont donc applicables :
    - Vous ne pouvez pas passer de débit lors de la création d’un conteneur serverless ; faire cela retourne une erreur.
    - Vous ne pouvez pas lire ou mettre à jour le débit sur un conteneur serverless ; faire cela retourne une erreur.
    - Vous ne pouvez pas créer une base de données de débit partagé dans un compte serverless ; faire cela renvoie une erreur.
- Les conteneurs serverless peuvent stocker un maximum de 50 Go de données et d’index.

## <a name="monitoring-your-consumption"></a>Surveillance de votre consommation

Si vous avez déjà utilisé Azure Cosmos DB en mode de débit approvisionné, vous constaterez que le serverless est plus économique lorsque votre trafic ne justifie pas la capacité approvisionnée. Le compromis est que vos coûts deviendront moins prévisibles, car vous êtes facturé en fonction du nombre de requêtes traitées par votre base de données. Pour cette raison, il est important de garder un œil sur votre consommation actuelle.

Lorsque vous parcourez le volet **Métriques** de votre compte, vous trouverez un graphique nommé **Unités de requête consommées** sous l’onglet **Vue d’ensemble**. Ce graphique montre le nombre d’unités de requête consommées par votre compte :

:::image type="content" source="./media/serverless/request-units-consumed.png" alt-text="Graphique présentant les unités de requête consommées" border="false":::

Vous pouvez trouver le même graphique lorsque vous utilisez Azure Monitor, comme décrit [ici](monitor-request-unit-usage.md). Notez qu’Azure Monitor vous permet de configurer des [alertes](../azure-monitor/platform/alerts-metric-overview.md), qui peuvent être utilisées pour vous avertir lorsque la consommation d’unités de requête a passé un certain seuil.

## <a name="performance"></a><a id="performance"></a>Niveau de performance

Les ressources serverless offrent des caractéristiques de performances spécifiques qui diffèrent de celles fournies par les ressources de débit approvisionné :

- **Disponibilité** : Une fois l’offre serverless mise à la disposition générale, la disponibilité des conteneurs serverless est couverte par un Contrat de niveau de service (SLA) de 99,9 % lorsque les zones de disponibilité (redondance de zone) ne sont pas utilisées. Le contrat SLA est de 99,99 % lors de l’utilisation de zones de disponibilité.
- **Latence** : Une fois que l’offre serverless sera en disponibilité générale, la latence des conteneurs serverless est couverte par un objectif de niveau de service (SLO) de 10 millisecondes ou moins pour les lectures de point et de 30 millisecondes ou moins pour les écritures. Une opération de lecture de point consiste à extraire un seul élément par son ID et sa valeur de clé de partition.
- **Prise en charge de l’augmentation du trafic** : Une fois que l’offre serverless sera en disponibilité générale, la charge des conteneurs serverless est couverte par un objectif de niveau de service (SLO) de 95 %. Cela signifie que la charge maximale peut être atteinte au moins 95 % du temps.

> [!NOTE]
> Comme n’importe quelle préversion d’Azure, Azure Cosmos DB serverless est exclu des contrats de niveau de service (SLA). Les caractéristiques de performances mentionnées ci-dessus sont fournies en tant qu’aperçu de ce que cette offre offrira lors de sa disponibilité générale.

## <a name="next-steps"></a>Étapes suivantes

Prenez en main le serverless avec les articles suivants :

- [Unités de requête dans Azure Cosmos DB](request-units.md)
- [Choisir entre le débit provisionné et le serverless](throughput-serverless.md)
- [Modèle de prix dans Azure Cosmos DB](how-pricing-works.md)
