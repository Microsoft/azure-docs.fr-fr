---
title: Estimer les coûts à l’aide du planificateur de capacité Azure Cosmos DB
description: Le planificateur de capacité Azure Cosmos DB vous permet d’estimer le débit (unités de requête/seconde) requis et le coût de votre charge de travail. Cet article explique comment utiliser la nouvelle version du planificateur de capacité pour estimer le débit et le coût requis.
author: deborahc
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 07/30/2019
ms.author: dech
ms.openlocfilehash: 017aeaaa24e8ac2d493d0de81a7abb655bae5611
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93342077"
---
# <a name="estimate-rus-using-the-azure-cosmos-db-capacity-planner"></a>Estimer le nombre d’unités de requête/seconde à l’aide du planificateur de capacité Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Pour optimiser les coûts et les performances, il est essentiel de configurer vos bases de données et conteneurs Azure Cosmos avec la quantité appropriée de débit provisionné, ou d’[unités de requête par seconde](request-units.md), pour votre charge de travail. Cet article explique comment utiliser le [planificateur de capacité](https://cosmos.azure.com/capacitycalculator/) Azure Cosmos DB pour estimer le nombre requis d’unités de requête par seconde et le coût de votre charge de travail. 

## <a name="how-to-estimate-throughput-and-cost-with-azure-cosmos-db-capacity-planner"></a>Estimer le débit et le coût avec le planificateur de capacité Azure Cosmos DB

Le planificateur de capacité peut être utilisé dans deux modes.

|**Mode**  |**Description**  |
|---------|---------|
|De base|Fournit des estimations de coût et d’unités de requête/seconde rapides et de haut niveau. Ce mode utilise les paramètres Azure Cosmos DB par défaut pour la stratégie d’indexation, la cohérence et d’autres paramètres. <br/><br/>Utilisez le mode De base pour une estimation rapide et de haut niveau lorsque vous évaluez une charge de travail potentielle à exécuter sur Azure Cosmos DB.|
|Avancé|Fournit une estimation plus détaillée des unités de requête par seconde et des coûts, avec la possibilité de paramétrer des paramètres supplémentaires, tels que la stratégie d’indexation, le niveau de cohérence et d’autres paramètres qui affectent le coût et le débit. <br/><br/>Utilisez le mode Avancé lorsque vous estimez les unités de requête par seconde d’un nouveau projet ou que vous souhaitez obtenir une estimation plus détaillée. |


## <a name="estimate-provisioned-throughput-and-cost-using-basic-mode"></a>Estimer le débit provisionné et le coût à l’aide du mode De base
Pour obtenir une estimation rapide de votre charge de travail à l’aide du mode De base,accédez au [planificateur de capacité](https://cosmos.azure.com/capacitycalculator/). Entrez les paramètres suivants en fonction de votre charge de travail : 

|**Input**  |**Description**  |
|---------|---------|
|Nombre de régions|Azure Cosmos DB est disponible dans toutes les régions Azure. Sélectionnez le nombre de régions requises pour votre charge de travail. Vous pouvez associer un nombre quelconque de régions à votre compte Cosmos. Pour plus d’informations, consultez [Distribution de données mondiale avec Azure Cosmos DB](distribute-data-globally.md).|
|Écritures multirégions|Si vous activez les [écritures dans plusieurs régions](distribute-data-globally.md#key-benefits-of-global-distribution), votre application peut lire et écrire dans n’importe quelle région Azure. Si vous désactivez les écritures dans plusieurs régions, votre application peut écrire des données dans une seule région. <br/><br/> Activez les écritures dans plusieurs régions si vous prévoyez d’avoir une charge de travail active-active qui nécessite des écritures à faible latence dans différentes régions. Par exemple, une charge de travail IoT qui écrit des données dans la base de données à des volumes élevés dans différentes régions. <br/><br/> Les écritures dans plusieurs régions garantissent une disponibilité en lecture et en écriture de 99,999 %. Les écritures dans plusieurs régions requièrent davantage de débit par rapport aux écritures dans une seule région. Pour plus d’informations, découvrez [en quoi les unités de requêtes se distinguent entre les écritures dans une région et les écritures dans plusieurs régions](optimize-cost-regions.md).|
|Total des données stockées (par région)|Estimation du nombre total de données stockées en Go dans une seule région.|
|Taille de l’élément|Taille estimée de l’élément de données (par exemple un document), comprise entre 1 Ko et 2 Mo. |
|Lectures/seconde par région|Nombre de lectures attendues par seconde. |
|Écritures/seconde par région|Nombre d’écritures attendues par seconde. |

Une fois que vous avez rempli les informations requises, sélectionnez **Calculer**. L’onglet **Estimation du coût** affiche le coût total du stockage et le débit provisionné. Vous pouvez développer le lien **Afficher les détails** sous cet onglet pour obtenir la répartition du débit requis pour les demandes de lecture et d’écriture. Chaque fois que vous modifiez la valeur d’un champ, sélectionnez **Calculer** pour recalculer le coût estimé. 

:::image type="content" source="./media/estimate-ru-with-capacity-planner/basic-mode.png" alt-text="Mode De base du planificateur de capacité":::

## <a name="estimate-provisioned-throughput-and-cost-using-advanced-mode"></a>Estimer le débit provisionné et le coût à l’aide du mode Avancé

Le mode Avancé vous permet de fournir des paramètres supplémentaires qui ont un impact sur l’estimation des unités de requête/seconde. Pour utiliser cette option, accédez au [planificateur de capacité](https://cosmos.azure.com/capacitycalculator/) et connectez-vous à l’outil avec un compte que vous utilisez pour Azure. L’option de connexion est disponible dans l’angle droit. 

Une fois que vous êtes connecté, vous pouvez voir des champs supplémentaires par rapport aux champs du mode De base. Entrez les paramètres supplémentaires en fonction de votre charge de travail. 

|**Input**  |**Description**  |
|---------|---------|
|API|Azure Cosmos DB est un service multi-modèle et multi-API. Pour les nouvelles charges de travail, sélectionnez API SQL (Core). |
|Nombre de régions|Azure Cosmos DB est disponible dans toutes les régions Azure. Sélectionnez le nombre de régions requises pour votre charge de travail. Vous pouvez associer un nombre quelconque de régions à votre compte Cosmos. Pour plus d’informations, consultez [Distribution de données mondiale avec Azure Cosmos DB](distribute-data-globally.md).|
|Écritures multirégions|Si vous activez les [écritures dans plusieurs régions](distribute-data-globally.md#key-benefits-of-global-distribution), votre application peut lire et écrire dans n’importe quelle région Azure. Si vous désactivez les écritures dans plusieurs régions, votre application peut écrire des données dans une seule région. <br/><br/> Activez les écritures dans plusieurs régions si vous prévoyez d’avoir une charge de travail active-active qui nécessite des écritures à faible latence dans différentes régions. Par exemple, une charge de travail IoT qui écrit des données dans la base de données à des volumes élevés dans différentes régions. <br/><br/> Les écritures dans plusieurs régions garantissent une disponibilité en lecture et en écriture de 99,999 %. Les écritures dans plusieurs régions requièrent davantage de débit par rapport aux écritures dans une seule région. Pour plus d’informations, découvrez [en quoi les unités de requêtes se distinguent entre les écritures dans une région et les écritures dans plusieurs régions](optimize-cost-regions.md).|
|Cohérence par défaut|Azure Cosmos DB prend en charge 5 niveaux de cohérence, afin de permettre aux développeurs d’avoir un compromis entre cohérence, disponibilité et latence. Pour en savoir plus, consultez l’article [Niveaux de cohérence](consistency-levels.md). <br/><br/> Par défaut, Azure Cosmos DB utilise la cohérence de session, qui garantit la capacité de lire vos propres écritures dans une session. <br/><br/> Le choix du niveau Obsolescence limitée ou Cohérence forte nécessite le double d’unités de requête/seconde requis pour les lectures, par rapport aux niveaux Session, Préfixe cohérent et Cohérence éventuelle. Une cohérence forte avec les écritures dans plusieurs régions n’est pas prise en charge et prend automatiquement par défaut les écritures dans une seule région avec une cohérence forte. |
|Stratégie d’indexation|Par défaut, Azure Cosmos DB [indexe toutes les propriétés](index-policy.md) de tous les éléments pour des requêtes flexibles et efficaces (en respectant la stratégie d’indexation **automatique** ). <br/><br/> Si vous choisissez **désactivé** , aucune des propriétés n’est indexée. Cela entraîne un coût d’unité de requête inférieur pour les écritures. Choisissez de **désactiver** la stratégie si vous prévoyez uniquement des [lectures de point](/dotnet/api/microsoft.azure.cosmos.container.readitemasync?preserve-view=true&view=azure-dotnet) (recherches de valeurs de clés) et/ou des écritures, et aucune requête. <br/><br/> La stratégie d’indexation personnalisée vous permet d’inclure ou d’exclure des propriétés spécifiques de l’index pour réduire le débit d’écriture et le stockage. Pour plus d’informations, consultez les articles sur la [stratégie d’indexation](index-overview.md) et les [exemples de stratégies d’indexation](how-to-manage-indexing-policy.md#indexing-policy-examples).|
|Total des données stockées (par région)|Estimation du nombre total de données stockées en Go dans une seule région.|
|Mode Charge de travail|Sélectionnez **Stable** si votre volume de charge de travail est constant. <br/><br/> Sélectionnez **Variable** si le volume de votre charge de travail change au fil du temps.  Par exemple, pendant un jour ou un mois spécifique. <br/><br/> Les paramètres suivants sont disponibles si vous choisissez l’option de charge de travail variable :<ul><li>Percentage of time at peak (Pourcentage de temps lors des pics) : Pourcentage de temps dans un mois où votre charge de travail requiert un débit maximal (le plus élevé). <br/><br/> Par exemple, si vous avez une charge de travail qui a une activité élevée pendant les heures de bureau de 9 h 00 à 18 h 00, le pourcentage de temps à atteindre est le suivant : 45 heures lors du pic/730 heures/mois = ~ 6 %.<br/><br/></li><li>Reads/sec per region at peak (Lectures/s par région lors des pics) : nombre de lectures attendues par seconde lors d’un pic.</li><li>Writes/sec per region at peak (Écritures/s par région lors des pics) : nombre d’écritures attendues par seconde lors des pics.</li><li>Reads/sec per region off peak (Lectures/s par région pendant les heures creuses) : nombre de lectures attendues par seconde pendant les heures creuses.</li><li>Writes/sec per region off peak (Écritures/s par région pendant les heures creuses) : nombre d’écritures attendues par seconde pendant les heures creuses.</li></ul>Avec les intervalles prenant en compte les heures creuses et les pics, vous pouvez optimiser vos coûts en [mettant à l’échelle votre débit provisionné par programmation](set-throughput.md#update-throughput-on-a-database-or-a-container) en conséquence.|
|Taille de l’élément|Taille de l’élément de données (par exemple un document), comprise entre 1 Ko et 2 Mo. <br/><br/>Vous pouvez également **charger un exemple de document (JSON)** pour obtenir une estimation plus précise.<br/><br/>Si votre charge de travail comporte plusieurs types d’éléments (avec un contenu JSON différent) dans le même conteneur, vous pouvez télécharger plusieurs documents JSON et récupérer l’estimation. Utilisez le bouton **Add new item (Ajouter un nouvel élément)** pour ajouter plusieurs exemples de documents JSON.|

Vous pouvez également utiliser le bouton **Save Estimate (Enregistrer l’estimation)** pour télécharger un fichier .csv contenant l’estimation actuelle. 

:::image type="content" source="./media/estimate-ru-with-capacity-planner/advanced-mode.png" alt-text="Mode Avancé du planificateur de capacité":::

Les prix indiqués dans le planificateur de capacité Azure Cosmos DB sont des estimations basées sur les tarifs publics pour le débit et le stockage. Tous les prix sont affichés en dollars américains. Reportez -vous à la [page de tarification Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/) pour afficher tous les tarifs par région.  

## <a name="estimating-throughput-for-queries"></a>Estimation du débit des requêtes

La calculatrice de capacité Azure Cosmos prend en compte des lectures de points (une lecture d’un seul élément, par exemple un document, par ID et valeur de clé de partition) et des écritures pour la charge de travail. Pour estimer le débit nécessaire pour les requêtes, exécutez votre requête sur un jeu de données représentatif dans un conteneur Cosmos et [obtenez le tarif appliqué aux unités de requête](find-request-unit-charge.md). Multipliez le tarif des unités de requête par le nombre de requêtes que vous prévoyez d’exécuter par seconde pour connaître la quantité totale d’unités de requête requise. 

Par exemple, si votre charge de travail requiert une requête ``SELECT * FROM c WHERE c.id = 'Alice'`` qui est exécutée 100 fois par seconde, et que la facturation d’unités de requête de la requête est de 10 unités de requête, vous avez besoin de 100 requêtes/s * 10 unités de requête/requête = 1 000 unités de requête/s au total pour traiter ces requêtes. Ajoutez ces unités de requête/s aux unités de requête/s nécessaires à toute lecture ou écriture se produisant dans votre charge de travail.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur le [modèle de tarification Azure Cosmos DB](how-pricing-works.md).
* Créez un [compte, une base de données et un conteneur Cosmos](create-cosmosdb-resources-portal.md).
* Découvrez comment [optimiser le coût du débit provisionné](optimize-cost-throughput.md).
* Découvrez comment [optimiser le coût avec une capacité réservée](cosmos-db-reserved-capacity.md).