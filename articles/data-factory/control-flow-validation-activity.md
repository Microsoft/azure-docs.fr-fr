---
title: Activité Validation dans Azure Data Factory
description: L’activité Validation interrompt l’exécution du pipeline jusqu’à ce qu’elle valide le jeu de données attaché avec certains critères spécifiés par l’utilisateur.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 03/25/2019
ms.openlocfilehash: 63f4a7df76fc0480a2b0cdf2de13ff5e85aa93ef
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96485822"
---
# <a name="validation-activity-in-azure-data-factory"></a>Activité Validation dans Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Vous pouvez utiliser une Validation dans un pipeline pour garantir que l’exécution du pipeline se poursuit uniquement une fois qu’elle a confirmé que la référence du jeu de données attaché existe, qu’elle remplit les critères spécifiés ou que le délai d’attente a été atteint.


## <a name="syntax"></a>Syntaxe

```json

{
    "name": "Validation_Activity",
    "type": "Validation",
    "typeProperties": {
        "dataset": {
            "referenceName": "Storage_File",
            "type": "DatasetReference"
        },
        "timeout": "7.00:00:00",
        "sleep": 10,
        "minimumSize": 20
    }
},
{
    "name": "Validation_Activity_Folder",
    "type": "Validation",
    "typeProperties": {
        "dataset": {
            "referenceName": "Storage_Folder",
            "type": "DatasetReference"
        },
        "timeout": "7.00:00:00",
        "sleep": 10,
        "childItems": true
    }
}

```


## <a name="type-properties"></a>Propriétés type

Propriété | Description | Valeurs autorisées | Obligatoire
-------- | ----------- | -------------- | --------
name | Nom de l’activité « Validation » | String | Oui |
type | Doit être défini sur **Validation**. | String | Oui |
dataset | L’activité bloquera l’exécution jusqu’à ce qu’elle ait confirmé que cette référence de jeu de données existe et qu’elle remplit les critères spécifiés, ou que le délai d’attente ait été atteint. Le jeu de données fourni doit prendre en charge la propriété « MinimumSize » ou « ChildItems ». | Référence de jeu de données | Oui |
délai d'expiration | Spécifie le délai d’expiration d’exécution de l’activité. Si aucune valeur n’est spécifiée, la valeur par défaut est sept jours (« 7:00:00:00 »). Le format est j.hh:mm:ss | String | Non |
sleep | Délai en secondes entre les tentatives de validation. Si aucune valeur n’est spécifiée, la valeur par défaut est 10 secondes. | Integer | Non |
childItems | Vérifie si le dossier a des éléments enfants. Peut être défini sur -true : vérifie que le dossier existe et qu’il a des éléments. Bloque jusqu’à ce qu’au moins un élément se trouve dans le dossier ou que la valeur de délai d’attente soit atteinte. -false : vérifie que le dossier existe et qu’il est vide. Bloque jusqu’à ce que le dossier soit vide ou que la valeur de délai d’attente soit atteinte. Si aucune valeur n’est spécifiée, l’activité bloquera jusqu’à ce que le dossier existe ou que le délai d’attente soit atteint. | Boolean | Non |
minimumSize | Taille minimale d’un fichier en octets. Si aucune valeur n’est spécifiée, la valeur par défaut est de 0 octet | Integer | Non |


## <a name="next-steps"></a>Étapes suivantes
Consultez les autres activités de flux de contrôle prises en charge par Data Factory :

- [Activité IfCondition](control-flow-if-condition-activity.md)
- [Activité d’exécution du pipeline](control-flow-execute-pipeline-activity.md)
- [Pour chaque activité](control-flow-for-each-activity.md)
- [Activité d’obtention des métadonnées](control-flow-get-metadata-activity.md)
- [Activité de recherche](control-flow-lookup-activity.md)
- [Activité Web](control-flow-web-activity.md)
- [Activité jusqu’à](control-flow-until-activity.md)
