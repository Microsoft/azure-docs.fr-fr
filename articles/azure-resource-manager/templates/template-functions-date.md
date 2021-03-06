---
title: Fonctions de modèle - date
description: Décrit les fonctions à utiliser dans un modèle Azure Resource Manager (ARM) pour travailler avec des dates.
ms.topic: conceptual
ms.date: 05/11/2021
ms.openlocfilehash: c6bf3adca5dde4947e2c22dd8468f1b045f77120
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/10/2021
ms.locfileid: "111959700"
---
# <a name="date-functions-for-arm-templates"></a>Fonctions de date pour les modèles ARM

Resource Manager fournit les fonctions ci-dessous pour vous permettre d’utiliser des dates dans votre modèle Azure Resource Manager (ARM) :

* [dateTimeAdd](#datetimeadd)
* [utcNow](#utcnow)

## <a name="datetimeadd"></a>dateTimeAdd

`dateTimeAdd(base, duration, [format])`

Ajoute une durée à une valeur de base. Le format ISO 8601 est attendu.

### <a name="parameters"></a>Paramètres

| Paramètre | Obligatoire | Type | Description |
|:--- |:--- |:--- |:--- |
| base | Oui | string | Valeur datetime de début pour l’ajout. Utilisez le [format d’horodatage ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). |
| duration | Oui | string | Valeur de temps à ajouter à la base. Il peut s’agir d’une valeur négative. Utilisez le [format de durée ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Durations). |
| format | Non | string | Format de sortie pour le résultat de date et d’heure. S’il n’est pas fourni, le format de la valeur de base est utilisé. Utilisez des [chaînes de format standard](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou des [chaînes de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). |

### <a name="return-value"></a>Valeur retournée

Valeur datetime qui résulte de l’ajout de la valeur de durée à la valeur de base.

### <a name="examples"></a>Exemples

L’exemple de modèle suivant montre différentes façons d’ajouter des valeurs de temps.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "baseTime": {
      "type": "string",
      "defaultValue": "[utcNow('u')]"
    }
  },
  "variables": {
    "add3Years": "[dateTimeAdd(parameters('baseTime'), 'P3Y')]",
    "subtract9Days": "[dateTimeAdd(parameters('baseTime'), '-P9D')]",
    "add1Hour": "[dateTimeAdd(parameters('baseTime'), 'PT1H')]"
  },
  "resources": [],
  "outputs": {
    "add3YearsOutput": {
      "value": "[variables('add3Years')]",
      "type": "string"
    },
    "subtract9DaysOutput": {
      "value": "[variables('subtract9Days')]",
      "type": "string"
    },
    "add1HourOutput": {
      "value": "[variables('add1Hour')]",
      "type": "string"
    },
  }
}
```

Lorsque le modèle précédent est déployé avec la valeur de temps de base `2020-04-07 14:53:14Z`, la sortie est la suivante :

| Nom | Type | Valeur |
| ---- | ---- | ----- |
| add3YearsOutput | String | 7/4/2023 14:53:14 |
| subtract9DaysOutput | String | 29/3/2020 14:53:14 |
| add1HourOutput | String | 7/4/2023 15:53:14 |

L’exemple de modèle suivant montre comment définir l’heure de début d’une planification Automation.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "omsAutomationAccountName": {
      "type": "string",
      "defaultValue": "demoAutomation",
      "metadata": {
        "description": "Use an existing Automation account."
      }
    },
    "scheduleName": {
      "type": "string",
      "defaultValue": "demoSchedule1",
      "metadata": {
        "description": "Name of the new schedule."
      }
    },
    "baseTime": {
      "type": "string",
      "defaultValue": "[utcNow('u')]",
      "metadata": {
        "description": "Schedule will start one hour from this time."
      }
    }
  },
  "variables": {
    "startTime": "[dateTimeAdd(parameters('baseTime'), 'PT1H')]"
  },
  "resources": [
    ...
    {
      "type": "Microsoft.Automation/automationAccounts/schedules",
      "apiVersion": "2015-10-31",
      "name": "[concat(parameters('omsAutomationAccountName'), '/', parameters('scheduleName'))]",

      "properties": {
        "description": "Demo Scheduler",
        "startTime": "[variables('startTime')]",
        "interval": 1,
        "frequency": "Hour"
      }
    }
  ],
  "outputs": {
  }
}
```

## <a name="utcnow"></a>utcNow

`utcNow(format)`

Retourne la valeur de date/heure (UTC) actuelle au format spécifié. Si aucun format n’est indiqué, le format utilisé est ISO 8601 (`yyyyMMddTHHmmssZ`). **Cette fonction peut uniquement être utilisée dans la valeur par défaut d’un paramètre.**

### <a name="parameters"></a>Paramètres

| Paramètre | Obligatoire | Type | Description |
|:--- |:--- |:--- |:--- |
| format |Non |string |Valeur encodée de l’URI à convertir en une chaîne. Utilisez des [chaînes de format standard](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou des [chaînes de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). |

### <a name="remarks"></a>Notes

Vous pouvez uniquement utiliser cette fonction dans une expression pour la valeur par défaut d’un paramètre. Son utilisation partout ailleurs dans un modèle retourne une erreur. La fonction n’est pas autorisée dans d’autres parties du modèle, car elle retourne une valeur différente chaque fois qu’elle est appelée. Le déploiement du même modèle avec les mêmes paramètres ne produit pas forcément les mêmes résultats.

Si vous choisissez l’[option de restauration en cas d’erreur](rollback-on-error.md) d’un déploiement antérieur ayant abouti et que celui-ci comprend un paramètre qui utilise utcNow, le paramètre n’est pas réévalué. Au lieu de cela, la valeur du paramètre du déploiement précédent est automatiquement réutilisée dans le déploiement de la restauration.

Soyez prudent quand vous redéployez un modèle basé sur la fonction utcNow pour une valeur par défaut. Si vous effectuez un tel déploiement sans fournir de valeur pour le paramètre, la fonction est réévaluée. Si vous souhaitez mettre à jour une ressource existante au lieu d’en créer une autre, passez la valeur du paramètre qui était utilisée dans le déploiement précédent.

### <a name="return-value"></a>Valeur retournée

Valeur de date/heure UTC actuelle.

### <a name="examples"></a>Exemples

L’exemple de modèle suivant montre des formats différents pour la valeur de date/heure.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "utcValue": {
      "type": "string",
      "defaultValue": "[utcNow()]"
    },
    "utcShortValue": {
      "type": "string",
      "defaultValue": "[utcNow('d')]"
    },
    "utcCustomValue": {
      "type": "string",
      "defaultValue": "[utcNow('M d')]"
    }
  },
  "resources": [
  ],
  "outputs": {
    "utcOutput": {
      "type": "string",
      "value": "[parameters('utcValue')]"
    },
    "utcShortOutput": {
      "type": "string",
      "value": "[parameters('utcShortValue')]"
    },
    "utcCustomOutput": {
      "type": "string",
      "value": "[parameters('utcCustomValue')]"
    }
  }
}
```

La sortie de l’exemple précédent varie pour chaque déploiement, mais elle sera semblable à celle-ci :

| Nom | Type | Valeur |
| ---- | ---- | ----- |
| utcOutput | string | 20190305T175318Z |
| utcShortOutput | string | 03/05/2019 |
| utcCustomOutput | string | 3 5 |

L’exemple suivant montre comment utiliser une valeur de la fonction pour définir une valeur de balise.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "utcShort": {
      "type": "string",
      "defaultValue": "[utcNow('d')]"
    },
    "rgName": {
      "type": "string"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2020-10-01",
      "name": "[parameters('rgName')]",
      "location": "westeurope",
      "tags": {
        "createdDate": "[parameters('utcShort')]"
      },
      "properties": {}
    }
  ],
  "outputs": {
    "utcShortOutput": {
      "type": "string",
      "value": "[parameters('utcShort')]"
    }
  }
}
```

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir une description des sections d’un modèle ARM, consultez [Comprendre la structure et la syntaxe des modèles ARM](./syntax.md).