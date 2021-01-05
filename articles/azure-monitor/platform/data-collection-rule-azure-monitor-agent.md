---
title: Configurer la collecte de données pour l’agent Azure Monitor (version préliminaire)
description: Explique comment créer une règle de collecte de données pour collecter des données à partir de machines virtuelles à l’aide de l’agent Azure Monitor.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/19/2020
ms.openlocfilehash: 94c926c555a4bc96ac3c6fbe773650e16554bcf2
ms.sourcegitcommit: 5ae2f32951474ae9e46c0d46f104eda95f7c5a06
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95315700"
---
# <a name="configure-data-collection-for-the-azure-monitor-agent-preview"></a>Configurer la collecte de données pour l’agent Azure Monitor (version préliminaire)

Les règles de collecte de données (DCR) définissent les données entrantes dans Azure Monitor et spécifient l’emplacement où elles doivent être envoyées. Cet article explique comment créer une règle de collecte de données pour collecter des données à partir d’ordinateurs virtuels à l’aide de l’agent Azure Monitor.

Pour obtenir une description complète des règles de collecte des données, consultez [Règles de collecte de données dans Azure Monitor (version préliminaire)](data-collection-rule-overview.md).

> [!NOTE]
> Cet article explique comment configurer des données pour des machines virtuelles avec l’agent Azure Monitor qui est actuellement en version préliminaire. Pour obtenir une description des agents mis à la disposition générale et la façon de les utiliser pour collecter des données, consultez [Vue d’ensemble des agents Azure Monitor](agents-overview.md).

## <a name="data-collection-rule-associations"></a>Associations de règles de collecte de données

Pour appliquer une DCR à une machine virtuelle, vous devez créer une association pour la machine virtuelle. Une machine virtuelle peut avoir une association à plusieurs DCR, et une DCR peut avoir plusieurs machines virtuelles associées. Cela vous permet de définir un ensemble de DCR, chacune correspondant à une exigence particulière, et de les appliquer uniquement aux machines virtuelles appropriées. 

Par exemple, prenons un environnement avec un ensemble de machines virtuelles exécutant une application métier et d’autres exécutant SQL Server. Vous pouvez avoir une règle de collecte de données par défaut qui s’applique à toutes les machines virtuelles et des règles de collecte de données distinctes qui collectent des données spécifiquement pour l’application métier et pour SQL Server. Les associations des machines virtuelles aux règles de collecte de données ressemblent au diagramme suivant.

![Le diagramme montre les machines virtuelles hébergeant des applications métier et les instances SQL Server associées aux règles de collecte des données nommées central-i t-default et lob-app pour les applications métier, et central-i t-default et s q l pour SQL Server.](media/data-collection-rule-azure-monitor-agent/associations.png)



## <a name="create-rule-and-association-in-azure-portal"></a>Créer une règle et une association dans le portail Azure

Vous pouvez utiliser le Portail Azure pour créer une règle de collecte de données et associer des machines virtuelles de votre abonnement à cette règle. L’agent Azure Monitor est automatiquement installé et une identité managée est créée pour toutes les machines virtuelles sur lesquelles il n’est pas déjà installé.

Dans le menu **Azure Monitor** du Portail Azure, sélectionnez **Règles de collecte des données** dans la section **Paramètres**. Cliquez sur **Ajouter** pour ajouter une nouvelle règle de collecte de données et une attribution.

[![Règles de collecte des données](media/azure-monitor-agent/data-collection-rules.png)](media/azure-monitor-agent/data-collection-rules.png#lightbox)

Cliquez sur **Ajouter** pour créer une règle et un ensemble d’associations. Fournissez un **Nom de règle** et spécifiez un **Abonnement** et un **Groupe de ressources**. Spécifie l’emplacement où la DCR sera créée. Les machines virtuelles et leurs associations peuvent se trouver dans n’importe quel abonnement ou groupe de ressources dans le locataire.

[![Notions de base des règles de collecte des données](media/azure-monitor-agent/data-collection-rule-basics.png)](media/azure-monitor-agent/data-collection-rule-basics.png#lightbox)

Sous l’onglet **Machines virtuelles**, ajoutez des machines virtuelles auxquelles la règle de collecte de données doit être appliquée. Les machines virtuelles Azure et les serveurs avec Azure Arc dans votre environnement doivent être listés. L’agent Azure Monitor sera installé sur les machines virtuelles sur lesquelles il n’est pas déjà installé.

[![Règles de collecte des données - Machines virtuelles](media/azure-monitor-agent/data-collection-rule-virtual-machines.png)](media/azure-monitor-agent/data-collection-rule-virtual-machines.png#lightbox)

Sous l’onglet **Collecter et livrer**, cliquez sur **Ajouter une source de données** pour ajouter une source de données et un ensemble de destinations. Sélectionnez un **Type de source de données**, et les détails correspondants à sélectionner s’affichent. Pour les compteurs de performances, vous pouvez sélectionner un jeu d’objets prédéfini et leur taux d’échantillonnage. Pour les événements, vous pouvez sélectionner parmi un ensemble de journaux ou d’installations ainsi que le niveau de gravité. 

[![Source de données - De base](media/azure-monitor-agent/data-collection-rule-data-source-basic.png)](media/azure-monitor-agent/data-collection-rule-data-source-basic.png#lightbox)


Pour spécifier d’autres journaux et compteurs de performances, sélectionnez **Personnalisé**. Vous pouvez ensuite spécifier un [XPath](https://www.w3schools.com/xml/xpath_syntax.asp) pour toutes les valeurs spécifiques à collecter. Pour des exemples, consultez [Exemple de DCR](data-collection-rule-overview.md#sample-data-collection-rule).

[![Source de données - Personnalisé](media/azure-monitor-agent/data-collection-rule-data-source-custom.png)](media/azure-monitor-agent/data-collection-rule-data-source-custom.png#lightbox)

Sous l’onglet **Destination**, ajoutez une ou plusieurs destinations pour la source de données. Les sources de données d’événement Windows et Syslog peuvent uniquement envoyer aux Journaux Azure Monitor. Les compteurs de performance peuvent envoyer à la fois aux Métriques Azure Monitor et aux Journaux Azure Monitor.

[![Destination](media/azure-monitor-agent/data-collection-rule-destination.png)](media/azure-monitor-agent/data-collection-rule-destination.png#lightbox)

Cliquez sur **Ajouter une source de données**, puis sur **Vérifier + créer** pour passer en revue les détails de la règle de collecte des données et l’association avec l’ensemble de machines virtuelles. Cliquez sur **Créer** pour la créer.

> [!NOTE]
> Une fois que la règle de collecte des données et les associations ont été créées, il peut falloir jusqu’à 5 minutes pour que les données soient envoyées aux destinations.


## <a name="create-rule-and-association-using-rest-api"></a>Créer une règle et une association à l’aide de l’API REST

Suivez les étapes ci-dessous pour créer une règle de collecte de données et des associations à l’aide de l’API REST.

1. Créez manuellement le fichier de règle de collecte de données à l’aide du format JSON présenté dans [Exemple de règle de collecte de données](data-collection-rule-overview.md#sample-data-collection-rule).

2. Créez la règle à l’aide de l’[API REST](/rest/api/monitor/datacollectionrules/create#examples).

3. Créez une association pour chaque machine virtuelle à la règle de collecte de données à l’aide de l’[API REST](/rest/api/monitor/datacollectionruleassociations/create#examples).


## <a name="create-association-using-resource-manager-template"></a>Créer une association à l’aide d’un modèle Resource Manager

Vous ne pouvez pas créer une règle de collecte de données à l’aide d’un modèle Resource Manager, mais vous pouvez créer une association entre une machine virtuelle Azure ou un serveur avec Azure Arc à l’aide d’un modèle Resource Manager. Pour des exemples de modèles, consultez [Exemples de modèle Resource Manager pour les règles de collecte de données dans Azure Monitor](../samples/resource-manager-data-collection-rules.md).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur l’[agent Azure Monitor](azure-monitor-agent-overview.md).
- En savoir plus sur les [règles de collecte de données](data-collection-rule-overview.md).
