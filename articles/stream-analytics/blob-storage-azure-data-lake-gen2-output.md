---
title: Sortie de stockage d’objets blob et Azure Data Lake Gen2 à partir d’Azure Stream Analytics
description: Cet article décrit le stockage d’objets blob et Azure Data Lake Gen2 comme sortie pour Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 17fc2db8c51c06601852037c38b9cfc2096c811f
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109753"
---
# <a name="blob-storage-and-azure-data-lake-gen2-output-from-azure-stream-analytics"></a>Sortie de stockage d’objets blob et Azure Data Lake Gen2 à partir d’Azure Stream Analytics

Data Lake Storage Gen2 fait du stockage Azure la base pour créer des dépôts Data Lake d’entreprise sur Azure. Conçu dès le départ pour traiter plusieurs téraoctets d’informations tout en assurant des centaines de gigaoctets de débit, Data Lake Storage Gen2 vous permet de facilement gérer d’importants volumes de données. Une caractéristique fondamentale de Data Lake Storage Gen2 est l’ajout d’un espace de noms hiérarchique au stockage Blob.

Le stockage d’objets blob Azure offre une solution peu coûteuse et évolutive pour stocker de grandes quantités de données non structurées dans le cloud. Pour une introduction au stockage d’objets blob et à son utilisation, consultez [Démarrage rapide : Charger, télécharger et répertorier des objets blob à l’aide du portail Microsoft Azure](../storage/blobs/storage-quickstart-blobs-portal.md).

## <a name="output-configuration"></a>Configuration de la sortie

Le tableau suivant répertorie les noms de propriétés et leur description pour la création d’une sortie d’objet blob ou ADLS Gen2.

| Nom de la propriété       | Description                                                                      |
| ------------------- | ---------------------------------------------------------------------------------|
| Alias de sortie        | Nom convivial utilisé dans les requêtes pour diriger la sortie de requête vers ce stockage d’objets blob. |
| Compte de stockage     | Nom du compte de stockage où vous envoyez votre sortie.               |
| Clé du compte de stockage | Clé secrète associée au compte de stockage.                              |
| Conteneur de stockage   | Regroupement logique des objets blob stockés dans le service BLOB Azure. Lorsque vous téléchargez un objet blob dans le service d'objets Blob, vous devez spécifier un conteneur pour cet objet blob. |
| Modèle de chemin d’accès | facultatif. Modèle de chemin d’accès au fichier utilisé pour écrire vos objets blob dans le conteneur spécifié. <br /><br /> Dans le modèle de chemin d’accès, vous pouvez choisir d’utiliser une ou plusieurs instances des variables de date et d’heure pour spécifier la fréquence d’écriture des objets blob : <br /> {date}, {time} <br /><br />Vous pouvez utiliser le partitionnement d’objet blob personnalisé afin de spécifier un nom personnalisé {field} de vos données d’événement aux objets blob de partition. Le nom du champ est alphanumérique et peut inclure des espaces, des traits d’union et des traits de soulignement. Voici les restrictions qui s’appliquent aux champs personnalisés : <ul><li>Les noms de champs ne sont pas sensibles à la casse. Par exemple, le service ne peut pas faire la différence entre la colonne « ID » et la colonne « id ».</li><li>Les champs imbriqués ne sont pas autorisés. Utilisez plutôt un alias dans la requête du travail pour « aplatir » le champ.</li><li>Des expressions ne peuvent pas être utilisées comme nom de champ.</li></ul> <br />Cette fonctionnalité permet également d’utiliser des configurations de spécificateur de format de date/heure personnalisé dans le chemin d’accès. Les formats de date et d’heure personnalisés doivent être spécifiés un par un et délimités par le mot clé {datetime:\<specifier>}. Les entrées autorisées pour \<specifier> sont yyyy, MM, M, dd, d, HH, H, mm, m, ss ou s. Le mot-clé {datetime:\<specifier>} peut être utilisé plusieurs fois dans le chemin pour former des configurations de date/d’heure personnalisées. <br /><br />Exemples : <ul><li>Exemple 1 : cluster1/logs/{date}/{time}</li><li>Exemple 2 : cluster1/logs/{date}</li><li>Exemple 3 : cluster1/{client_id}/{date}/{time}</li><li>Exemple 4 : cluster1/{datetime:ss}/{myField}, où la requête est : SELECT data.myField AS myField FROM Input;</li><li>Exemple 5 : cluster1/année = {datetime:yyyy} / mois = {datetime:MM} / jour = {datetime:dd}</ul><br />L’horodatage de la structure de dossiers créée suit l’heure UTC et pas l’heure locale. [System.Timestamp](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-time-handling#choose-the-best-starting-time) est l’heure utilisée pour tout partitionnement basé sur le temps.<br /><br />La convention suivante est utilisée pour l’attribution de noms : <br /><br />{Modèle de préfixe de chemin d’accès}/Code_hachage_schéma_Numéro_Guid.extension<br /><br /> Ici, le GUID représente l’identificateur unique affecté à un writer interne créé pour écrire dans un fichier bob. Le nombre représente l’index du bloc d’objets blob. <br /><br /> Exemples de fichier de sortie :<ul><li>Myoutput/20170901/00/45434_gguid_1.csv</li>  <li>Myoutput/20170901/01/45434_gguid_1.csv</li></ul> <br />Pour plus d’informations sur cette fonctionnalité, consultez [Partitionnement personnalisé de sortie BLOB dans Azure Stream Analytics](stream-analytics-custom-path-patterns-blob-storage-output.md). |
| Format de la date | facultatif. Si le jeton de la date est utilisé dans le chemin d’accès du préfixe, vous pouvez sélectionner le format de date dans lequel vos fichiers sont organisés. Exemple : AAAA/MM/JJ |
| Format de l’heure | facultatif. Si le jeton de l’heure est utilisé dans le chemin d’accès du préfixe, spécifiez le format d’heure dans lequel vos fichiers sont organisés. Actuellement, la seule valeur possible est HH. |
| Format de sérialisation de l’événement | Format de sérialisation pour les données de sortie. JSON, CSV, Avro et Parquet sont pris en charge. |
|Nombre minimal de lignes |Le nombre minimum de lignes par lot. Pour Parquet, chaque lot crée un nouveau fichier. La valeur par défaut actuelle est de 2 000 lignes et la valeur maximum autorisée est de 10 000 lignes.|
|Durée maximale |Le délai d’attente maximum par lot. À l’issue de cette période, le lot est écrit dans la sortie même si l’exigence de lignes minimum n’est pas remplie. La valeur par défaut actuelle est de 1 minute et la valeur maximum autorisée est de 2 heures. Si la sortie de votre objet BLOB a une fréquence de modèle de chemin d’accès, le délai d’attente ne peut pas être supérieur à l’intervalle de temps de la partition.|
| Encodage    | Si vous utilisez le format CSV ou JSON, vous devez spécifier un encodage. UTF-8 est le seul format de codage actuellement pris en charge. |
| Délimiteur   | Applicable uniquement pour la sérialisation CSV. Stream Analytics prend en charge un certain nombre de délimiteurs communs pour sérialiser des données CSV. Valeurs prises en charge : virgule, point-virgule, espace, tabulation et barre verticale. |
| Format      | Applicable uniquement pour la sérialisation JSON. L’expression **Séparé par une ligne** indique que la sortie est mise en forme de sorte que tous les objets JSON soient séparés par une nouvelle ligne. Si vous sélectionnez **Séparé par une ligne**, le JSON est lu un objet à la fois. Le contenu entier seul ne serait pas un JSON valide. Le terme **Tableau** indique que la sortie est mise en forme en tant que tableau d’objets JSON. Ce tableau se ferme uniquement lorsque le travail s’arrête ou que Stream Analytics est passé à la période suivante. En règle générale, il est préférable d’utiliser du code JSON séparé par des lignes, car il ne requiert aucun traitement spécial pendant que le fichier de sortie est écrit. |

## <a name="blob-output-files"></a>Fichiers de sortie d’objets blob

Lorsque vous utilisez le stockage d’objets blob en tant que sortie, un fichier est créé dans l’objet blob dans les cas suivants :

* si le fichier actuel dépasse le nombre maximal de blocs autorisés (50 000 actuellement) Vous pouvez atteindre cette limite sans atteindre la taille maximale d’objet blob autorisée. Par exemple, si le taux de sortie est élevé, vous voyez apparaître davantage d’octets par bloc, et la taille de fichier est supérieure. S’il est faible, chaque bloc comporte moins de données et cette taille est inférieure.) ;
* si le schéma est modifié dans la sortie, et que le format de cette dernière requiert un schéma fixe (CSV et Avro) ;
* si un travail est relancé, de façon externe par un utilisateur ou en interne à des fins de maintenance système ou de récupération sur erreur ;
* si la requête est entièrement partitionnée et qu’un fichier est créé pour chaque partition de sortie ;
* si un fichier ou un conteneur du compte de stockage est supprimé par l’utilisateur ;
* si la sortie fait l’objet d’un partitionnement temporel via le modèle de préfixe de chemin d’accès et qu’un nouvel objet blob est utilisé lorsque la requête passe à l’heure suivante ;
* si la sortie est partitionnée par un champ personnalisé et qu’un nouvel objet blob est créé par clé de partition s’il n’y en a pas ;
* si la sortie est partitionnée par un champ personnalisé où la cardinalité des clés de partition est supérieure à 8 000 et qu’un nouvel objet blob est créé par clé de partition.

## <a name="partitioning"></a>Partitionnement

Pour une clé de partition, utilisez les jetons {date} et {time} de vos champs d’événement dans le modèle de chemin. Choisissez le format de date, par exemple, YYYY/MM/DD, DD/MM/YYYY ou MM-DD-YYYY. HH est utilisé pour le format de l’heure. La sortie d’objet blob peut être partitionnée par un seul attribut d’événement personnalisé {fieldname} ou par {datetime:\<specifier>}. Le nombre d’enregistreurs de sortie suit le partitionnement d’entrée de [requêtes entièrement parallélisables](stream-analytics-scale-jobs.md).

## <a name="output-batch-size"></a>Taille de lot de sortie

Pour obtenir la taille maximale du message, consultez [Limites de stockage Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#storage-limits). La taille maximale des blocs d’objets blob est de 4 Mo et le nombre maximal des blobs d’objets blob est de 50 000. |

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)
* [Démarrage rapide : Créer un travail Azure Stream Analytics avec l’interface de ligne de commande Azure](quick-create-azure-cli.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics à l’aide d’un modèle Resource Manager](quick-create-azure-resource-manager.md)
* [Démarrage rapide : Créer un travail Stream Analytics à l’aide d’Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics à l’aide de Visual Studio](stream-analytics-quick-create-vs.md)
* [Démarrage rapide : Créer une tâche Azure Stream Analytics dans Visual Studio Code](quick-create-visual-studio-code.md)
