---
title: Power BI à partir d’Azure Stream Analytics
description: Cet article explique comment sortir des données d’Azure Stream Analytics vers Power BI.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: cc48d26c217f52dfa52ac6cd3d7f18a806c93927
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97740978"
---
# <a name="power-bi-output-from-azure-stream-analytics"></a>Power BI à partir d’Azure Stream Analytics

Vous pouvez utiliser [Power BI](https://powerbi.microsoft.com/) comme sortie d’une tâche Stream Analytics pour fournir une expérience de visualisation riche des résultats d’analyse. Vous pouvez utiliser cette fonctionnalité pour les tableaux de bord opérationnels, la génération de rapports et la création de rapports pilotés par des métriques.

La sortie Power BI de Stream Analytics n’est pas disponible dans les régions Azure - Chine 21Vianet et Azure - Allemagne (T-Systems International).

## <a name="output-configuration"></a>Configuration de la sortie

Le tableau suivant répertorie les noms de propriétés et leur description pour configurer votre sortie Power BI.

| Nom de la propriété | Description |
| --- | --- |
| Alias de sortie |Fournit un nom convivial qui est utilisé dans les requêtes pour diriger la sortie de requête vers cette sortie Power BI. |
| Espace de travail de groupe |Pour permettre le partage de données avec d’autres utilisateurs de Power BI, vous pouvez sélectionner des groupes à l’intérieur de votre compte Power BI ou choisir **Mon espace de travail** si vous ne souhaitez pas écrire dans un groupe. La mise à jour d’un groupe existant nécessite le renouvellement de l’authentification Power BI. |
| Nom du jeu de données |Fournissez un nom de jeu de données qui doit être utilisé pour la sortie Power BI. |
| Nom de la table |Fournissez un nom de table sous le jeu de données de la sortie Power BI. Actuellement, une sortie Power BI des travaux Stream Analytics ne peut avoir qu’une table dans un jeu de données. |
| Autoriser la connexion | Vous devez autoriser la connexion avec Power BI pour configurer vos paramètres de sortie. Après avoir accordé à cette sortie l’accès à votre tableau de bord Power BI, vous pouvez le révoquer en modifiant le mot de passe du compte d’utilisateur, en supprimant la sortie du travail ou en effaçant le travail Stream Analytics. | 

Pour une vue d’ensemble de la configuration d’un tableau de bord et d’une sortie Power BI, consultez le didacticiel sur [Azure Stream Analytics et Power BI](stream-analytics-power-bi-dashboard.md).

> [!NOTE]
> Ne créez pas explicitement le jeu de données et la table dans le tableau de bord Power BI. Le jeu de données et la table seront automatiquement remplis au démarrage du travail et lorsque celui-ci commence à extraire des sorties dans Power BI. Si la requête de travail ne génère aucun résultat, le jeu de données et la table ne sont pas créés. De même, si Power BI dispose déjà d’un jeu de données et d’une table portant le même nom que celui saisi dans le travail Stream Analytics, alors ces données sont écrasées.
>

### <a name="create-a-schema"></a>Créer un schéma

Azure Stream Analytics crée un jeu de données et un schéma de table Power BI pour l’utilisateur s’il n’en existe pas encore. Dans tous les autres cas, la table est mise à jour de façon à inclure les nouvelles valeurs. Actuellement, seule une table peut exister dans un jeu de données. 

Power BI utilise la stratégie de rétention FIFO (premier entré, premier sorti). Les données sont collectées dans une table jusqu’à ce qu’elle atteigne 200 000 lignes.

> [!NOTE]
> Nous vous déconseillons d’utiliser plusieurs sorties pour écrire dans le même jeu de données, car cela peut entraîner plusieurs problèmes. Chaque sortie tente de créer le jeu de données Power BI indépendamment, ce qui peut entraîner l’affichage de plusieurs jeux de données portant le même nom. En outre, si les sorties n’ont pas de schémas cohérents, le jeu de données modifie le schéma sur chaque écriture, ce qui génère un trop grand nombre de demandes de modification de schéma. Même si ces problèmes sont évités, les sorties multiples seront moins performantes qu’une seule sortie fusionnée.

### <a name="convert-a-data-type-from-stream-analytics-to-power-bi"></a>Convertir un type de données Stream Analytics vers Power BI

Azure Stream Analytics met à jour le modèle de données dynamiquement lors de l’exécution si le schéma de sortie est modifié. L’intégralité des modifications de nom de colonne, modifications de type de colonne et ajouts ou suppressions de colonnes sont suivis.

Ce tableau décrit les conversions des [types de données Stream Analytics](/stream-analytics-query/data-types-azure-stream-analytics) vers les [types Entity Data Model (EDM)](/dotnet/framework/data/adonet/entity-data-model) de Power BI si un jeu de données et une table Power BI n’existent pas.

De Stream Analytics | Vers Power BI
-----|-----
bigint | Int64
nvarchar(max) | String
DATETIME | Datetime
float | Double
Tableau d’enregistrements | Type chaîne, valeur constante « IRecord » ou « IArray »

### <a name="update-the-schema"></a>Mettre à jour le schéma

Steam Analytics déduit le schéma de modèle de données sur la base du premier ensemble d’événements de la sortie. Plus tard, si nécessaire, le schéma de modèle de données est mis à jour pour prendre en compte les événements entrants qui peuvent ne pas correspondre au schéma d’origine.

Évitez d’utiliser la requête `SELECT *` pour empêcher la mise à jour dynamique du schéma entre les lignes. En plus de l’impact potentiel sur les performances, le temps nécessaire pour la création des résultats devient impossible à déterminer. Sélectionnez les champs exacts qui doivent être présentés dans le tableau de bord Power BI. En outre, les valeurs de données doivent être compatibles avec le type de données choisi.

Précédent/actuel | Int64 | String | Datetime | Double
-----------------|-------|--------|----------|-------
Int64 | Int64 | String | String | Double
Double | Double | String | String | Double
String | String | String | String | String 
Datetime | String | String |  Datetime | String

## <a name="output-batch-size"></a>Taille de lot de sortie

Pour la taille du lot de sortie, consultez [Limites d’API REST de Power BI](/power-bi/developer/automation/api-rest-api-limitations).

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser une identité managée pour authentifier votre travail Azure Stream Analytics vers Power BI (préversion)](powerbi-output-managed-identity.md)
* [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)