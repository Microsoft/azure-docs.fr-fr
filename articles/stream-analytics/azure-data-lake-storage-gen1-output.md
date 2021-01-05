---
title: Sortie Azure Data Lake Storage Gen 1 à partir d’Azure Stream Analytics
description: Cet article décrit Azure Data Lake Storage Gen 1 comme option de sortie pour Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 0445126046a6f4140dd68b2969221b196b8bef8b
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97740451"
---
# <a name="azure-data-lake-storage-gen-1-output-from-azure-stream-analytics"></a>Sortie Azure Data Lake Storage Gen 1 à partir d’Azure Stream Analytics

Stream Analytics prend en charge les sorties [Azure Data Lake Storage Gen 1](../data-lake-store/data-lake-store-overview.md). Azure Data Lake Storage est un référentiel d’entreprise à très grande échelle pour les charges de travail d’analytique du Big Data. Vous pouvez utiliser Data Lake Storage pour stocker des données de toute taille, de tout type et de toute vitesse d’ingestion en vue d’une analyse opérationnelle et exploratoire. Stream Analytics doit être autorisé à accéder à Data Lake Storage.

La sortie Azure Data Lake Storage de Stream Analytics n’est pas disponible dans les régions Azure – Chine 21Vianet et Azure – Allemagne (T-Systems International).

## <a name="output-configuration"></a>Configuration de la sortie

Le tableau suivant répertorie les noms de propriétés et leur description pour configurer votre sortie Data Lake Storage Gen 1.

| Nom de la propriété | Description |
| --- | --- |
| Alias de sortie | Nom convivial utilisé dans les requêtes pour diriger la sortie de la requête vers Data Lake Store. |
| Abonnement | Abonnement qui contient votre compte Azure Data Lake Storage. |
| Nom du compte | Nom du compte Data Lake Store où vous envoyez votre sortie. Vous accédez à la liste déroulante des comptes Data Lake Store disponibles dans votre abonnement. |
| Modèle de préfixe de chemin d’accès | Chemin de fichier utilisé pour écrire vos fichiers dans le compte Data Lake Store spécifié. Vous pouvez spécifier une ou plusieurs instances des variables {date} et {time} :<br /><ul><li>Exemple 1 : dossier1/journaux/{date}/{heure}</li><li>Exemple 2 : dossier1/journaux/{date}</li></ul><br />L’horodatage de la structure de dossiers créée suit l’heure UTC et pas l’heure locale.<br /><br />Si le modèle de chemin d’accès du fichier ne se termine pas par le caractère (/), le dernier modèle du chemin d’accès du fichier est traité comme préfixe de nom de fichier. <br /><br />De nouveaux fichiers sont créés dans les cas de figure suivants :<ul><li>modification du schéma de sortie ;</li><li>redémarrage externe ou interne d’un travail.</li></ul> |
| Format de la date | facultatif. Si le jeton de la date est utilisé dans le chemin d’accès du préfixe, vous pouvez sélectionner le format de date dans lequel vos fichiers sont organisés. Exemple : AAAA/MM/JJ |
|Format de l’heure | facultatif. Si le jeton de l’heure est utilisé dans le chemin d’accès du préfixe, spécifiez le format d’heure dans lequel vos fichiers sont organisés. Actuellement, la seule valeur possible est HH. |
| Format de sérialisation de l’événement | Format de sérialisation pour les données de sortie. JSON, CSV et Avro sont pris en charge.|
| Encodage | Si vous utilisez le format CSV ou JSON, vous devez spécifier un encodage. UTF-8 est le seul format de codage actuellement pris en charge.|
| Délimiteur | Applicable uniquement pour la sérialisation CSV. Stream Analytics prend en charge un certain nombre de délimiteurs communs pour sérialiser des données CSV. Valeurs prises en charge : virgule, point-virgule, espace, tabulation et barre verticale.|
| Format | Applicable uniquement pour la sérialisation JSON. L’expression **Séparé par une ligne** indique que la sortie est mise en forme de sorte que tous les objets JSON soient séparés par une nouvelle ligne. Si vous sélectionnez **Séparé par une ligne**, le JSON est lu un objet à la fois. Le contenu entier seul ne serait pas un JSON valide.  Le terme **Tableau** indique que la sortie est mise en forme en tant que tableau d’objets JSON. Ce tableau se ferme uniquement lorsque le travail s’arrête ou que Stream Analytics est passé à la période suivante. En règle générale, il est préférable d’utiliser du code JSON séparé par des lignes, car il ne requiert aucun traitement spécial pendant que le fichier de sortie est écrit.|
| Mode d'authentification | Vous pouvez autoriser l’accès à votre compte Data Lake Storage en utilisant une [identité managée](stream-analytics-managed-identities-adls.md) (préversion) ou un jeton d’utilisateur. Une fois l’accès accordé, vous pouvez le révoquer en modifiant le mot de passe du compte d’utilisateur, en supprimant la sortie Data Lake Storage de ce travail ou en effaçant le travail Stream Analytics. |

## <a name="partitioning"></a>Partitionnement

Pour la clé de partition, utilisez les jetons {date} et {time} dans le modèle de préfixe du chemin d’accès. Choisissez un format de date, par exemple, YYYY/MM/DD, DD/MM/YYYY ou MM-DD-YYYY. Utilisez HH pour le format de l’heure. Le nombre d’enregistreurs de sortie suit le partitionnement d’entrée de [requêtes entièrement parallélisables](stream-analytics-scale-jobs.md).

## <a name="output-batch-size"></a>Taille de lot de sortie

Pour obtenir la taille maximale du message, consultez [Limites de Data Lake Storage](../azure-resource-manager/management/azure-subscription-service-limits.md#data-lake-storage-limits). Pour optimiser la taille du lot, utilisez jusqu’à 4 Mo par opération d’écriture.

## <a name="next-steps"></a>Étapes suivantes

* [Authentifier Stream Analytics pour Azure Data Lake Storage Gen1 à l’aide d’identités managées (préversion)](stream-analytics-managed-identities-adls.md)
* [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)