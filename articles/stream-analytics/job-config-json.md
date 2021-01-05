---
title: Champs JobConfig.json Azure Stream Analytics
description: Cet article liste les champs pris en charge pour le fichier JobConfig.json Azure Stream Analytics utilisé pour créer des tâches dans Visual Studio Code.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 02/14/2020
ms.openlocfilehash: bb1a1a0b9f3a5996af56ff2c4e2de91d78a6c260
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93129335"
---
# <a name="azure-stream-analytics-jobconfigjson-fields"></a>Champs JobConfig.json Azure Stream Analytics

Les champs suivants sont pris en charge dans le fichier *JobConfig.json* utilisé pour [créer une tâche Azure Stream Analytics avec Visual Studio Code](quick-create-visual-studio-code.md).

```json
{
    "DataLocale": "string",
    "OutputErrorPolicy": "string",
    "EventsLateArrivalMaxDelayInSeconds": "integer",
    "EventsOutOfOrderMaxDelayInSeconds": "integer",
    "EventsOutOfOrderPolicy": "string",
    "StreamingUnits": "integer",
    "CompatibilityLevel": "string",
    "UseSystemAssignedIdentity": "boolean",
    "GlobalStorage": {
        "AccountName": "string",
        "AccountKey": "string",
    },
    "DataSourceCredentialDomain": "string",
    "ScriptType": "string",
    "Tags": {}
}
```

|Name|Type|Obligatoire|Valeur|
|----|----|--------|-----|
|DataLocale|string|Non|Paramètres régionaux des données de la tâche Stream Analytics. La valeur doit être le nom d’une culture .NET prise en charge. La valeur par défaut est « en-US » si aucune n’est spécifiée.|
|OutputErrorPolicy|string|Non|Indique la stratégie à appliquer aux événements qui arrivent à la sortie et qui ne peuvent pas être écrits dans le stockage externe en raison d’une constitution inappropriée (valeurs de colonne manquantes ou avec une taille ou un type incorrect). - Arrêter ou Supprimer|
|EventsLateArrivalMaxDelayInSeconds|entier|Non|Délai maximal acceptable en secondes pendant lequel les événements arrivant en retard peuvent être inclus. La plage prise en charge est comprise entre -1 et 1814399 (20.23:59:59 jours) et -1 est utilisé pour spécifier d’attendre indéfiniment. Si la propriété est absente, elle est interprétée comme ayant la valeur -1.|
|EventsOutOfOrderMaxDelayInSeconds|entier|Non|Délai maximal acceptable en secondes pendant lequel les événements en désordre peuvent être ajustés pour être restaurés dans l’ordre.|
|EventsOutOfOrderPolicy|string|Non|Indique la stratégie à appliquer aux événements qui arrivent en désordre dans le flux d’événements d’entrée. - Ajuster ou Supprimer|
|StreamingUnits|entier|Oui|Indique le nombre d’unités de streaming que la tâche de streaming utilise.|
|CompatibilityLevel|string|Non|Contrôle certains comportements d’exécution de la tâche de streaming. - Les valeurs acceptables sont « 1.0 », « 1.1 », « 1.2 »|
|UseSystemAssignedIdentity|boolean|Non|Affectez la valeur true pour permettre à cette tâche de communiquer en son nom avec d’autres services Azure à l’aide d’une identité Azure Active Directory managée.|
|GlobalStorage.AccountName|string|Non|Un compte de stockage général est utilisé pour stocker le contenu relatif à votre tâche Stream Analytics, comme les instantanés de données de référence SQL.|
|GlobalStorage.AccountKey|string|Non|Clé correspondante pour le compte de stockage général.|
|DataSourceCredentialDomain|string|Non|Propriété réservée pour le stockage local des informations d’identification.|
|ScriptType|string|Oui|Propriété réservée pour indiquer le type de ce fichier source. La valeur acceptable est « JobConfig » pour JobConfig.json.|
|Balises|Paires clé-valeur JSON|Non|Les étiquettes sont des paires nom/valeur qui vous permettent de catégoriser les ressources et d’afficher une facturation centralisée en appliquant la même étiquette à plusieurs ressources et groupes de ressources. Les noms d’étiquette ne sont pas sensibles à la casse, alors que les valeurs d’étiquette le sont.|

## <a name="next-steps"></a>Étapes suivantes

* [Créer une tâche Azure Stream Analytics dans Visual Studio Code](quick-create-visual-studio-code.md)
* [Tester des requêtes Stream Analytics localement avec des exemples de données à l’aide de Visual Studio Code](visual-studio-code-local-run.md)
* [Tester des requêtes Stream Analytics localement par rapport à une entrée de stream en direct à l’aide de Visual Studio Code](visual-studio-code-local-run-live-input.md)
*[Déployer une tâche Azure Stream Analytics à l’aide du package npm CI/CD](./cicd-overview.md)