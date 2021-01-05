---
title: Sortie Azure Synapse Analytics à partir d’Azure Stream Analytics
description: Cet article décrit Azure Synapse en tant que sortie pour Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 65752321f94bfa1403839889ead284d46b5f7191
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97740060"
---
# <a name="azure-synapse-analytics-output-from-azure-stream-analytics"></a>Sortie Azure Synapse Analytics à partir d’Azure Stream Analytics

[Azure Synapse Analytics](https://azure.microsoft.com/services/synapse-analytics) est un service d’analytique illimité qui regroupe l’entreposage des données d’entreprise et l’analytique de Big Data. 

Les tâches Azure Stream Analytics peuvent définir une sortie vers une table de pool SQL dédié dans Azure Stream Analytics et traiter des débits jusqu’à 200 Mo/s. Cela permet de prendre en charge les besoins les plus exigeants en matière d'analytique en temps réel et de traitement des données de chemin réactif pour des charges de travail telles que la création de rapports et de tableaux de bord.  

La table de pool SQL dédié doit être présente pour vous permettre de l'ajouter en tant qu'entrée à votre tâche Stream Analytics. Le schéma de table doit correspondre aux champs et aux types dans la sortie de votre travail. 

Pour utiliser Azure Synapse en tant que sortie, assurez-vous que le compte de stockage est configuré. Accédez aux paramètres du compte de stockage pour configurer le compte de stockage. Seuls les types de compte de stockage prenant en charge les tables sont autorisés : Usage général V2 et Usage général V1 Sélectionnez le niveau Standard uniquement. Le niveau Premium n’est pas pris en charge.

## <a name="output-configuration"></a>Configuration de la sortie

Le tableau suivant répertorie les noms de propriétés et leur description pour la création d’une sortie Azure Synapse Analytics.

|Nom de la propriété|Description|
|-|-|
|Alias de sortie |Nom convivial utilisé dans les requêtes pour diriger la sortie de requête vers cette base de données. |
|Base de données |Nom du pool SQL dédié où vous envoyez votre sortie. |
|Nom du serveur |Nom du serveur Azure Synapse.  |
|Nom d’utilisateur |Nom de l’utilisateur qui a accès en écriture à la base de données. Stream Analytics prend uniquement en charge l’authentification SQL. |
|Mot de passe |Mot de passe de connexion à la base de données. |
|Table de charge de travail  | Nom de la table dans laquelle la sortie sera écrite. Le nom de la table respecte la casse. Le schéma de cette table doit correspondre exactement au nombre de champs et aux types que votre sortie de travail génère.|

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser les identités managées pour accéder à Azure SQL Database ou Azure Synapse Analytics à partir d’une tâche Azure Stream Analytics (préversion)](sql-database-output-managed-identity.md)
* [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)