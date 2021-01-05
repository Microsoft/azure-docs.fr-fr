---
title: Élément d'interface utilisateur TextBox
description: Décrit l’élément d’interface utilisateur Microsoft.Common.TextBox pour le Portail Azure. Permet d’ajouter du texte non mis en forme.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: c89bc434d9d67144a95b5c2f23e7664078fe7825
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87474306"
---
# <a name="microsoftcommontextbox-ui-element"></a>Élément d’interface utilisateur Microsoft.Common.TextBox

Contrôle pouvant être utilisé pour modifier du texte non mis en forme.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

![Microsoft.Common.TextBox](./media/managed-application-elements/microsoft-common-textbox.png)

## <a name="schema"></a>schéma

```json
{
    "name": "nameInstance",
    "type": "Microsoft.Common.TextBox",
    "label": "Name",
    "defaultValue": "contoso123",
    "toolTip": "Use only allowed characters",
    "placeholder": "",
    "constraints": {
        "required": true,
        "validations": [
            {
                "regex": "^[a-z0-9A-Z]{1,30}$",
                "message": "Only alphanumeric characters are allowed, and the value must be 1-30 characters long."
            },
            {
                "isValid": "[startsWith(steps('resourceConfig').nameInstance, 'contoso')]",
                "message": "Must start with 'contoso'."
            }
        ]
    },
    "visible": true
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
"contoso123"
```

## <a name="remarks"></a>Notes

- Si `constraints.required` est défini sur **true**, la zone de texte doit contenir une valeur pour permettre la réussite de la validation. La valeur par défaut est **false**.
- La propriété `validations` est un tableau dans lequel vous ajoutez des conditions pour vérifier la valeur fournie dans la zone de texte.
- La propriété `regex` est un modèle d’expression régulière JavaScript. S’il est spécifié, la valeur de la zone de texte doit correspondre au modèle pour permettre la réussite de la validation. La valeur par défaut est **null**.
- La propriété `isValid` contient une expression qui prend la valeur true ou false. Dans l’expression, vous définissez la condition qui détermine si la zone de texte est valide.
- La propriété `message` est une chaîne à afficher en cas d’échec de la validation de la valeur de la zone de texte.
- Il est possible de spécifier une valeur pour `regex` lorsque `required` est défini sur **false**. Dans ce scénario, aucune valeur n’est requise dans la zone de texte pour que la validation réussisse. Si elle est spécifiée, celle-ci doit correspondre au modèle d’expression régulière.
- La propriété `placeholder` est un texte d’aide qui disparaît lorsque l’utilisateur commence à modifier. Si les `placeholder` et `defaultValue` sont définis, le `defaultValue` est prioritaire et est affiché.

## <a name="example"></a>Exemple

L’exemple suivant utilise une zone de texte avec le contrôle [Microsoft.Solutions.ArmApiControl](microsoft-solutions-armapicontrol.md) pour vérifier la disponibilité d’un nom de ressource.

```json
"basics": [
    {
        "name": "nameApi",
        "type": "Microsoft.Solutions.ArmApiControl",
        "request": {
            "method": "POST",
            "path": "[concat(subscription().id, '/providers/Microsoft.Storage/checkNameAvailability?api-version=2019-06-01')]",
            "body": "[parse(concat('{\"name\": \"', basics('txtStorageName'), '\", \"type\": \"Microsoft.Storage/storageAccounts\"}'))]"
        }
    },
    {
        "name": "txtStorageName",
        "type": "Microsoft.Common.TextBox",
        "label": "Storage account name",
        "constraints": {
            "validations": [
                {
                    "isValid": "[not(equals(basics('nameApi').nameAvailable, false))]",
                    "message": "[concat('Name unavailable: ', basics('txtStorageName'))]"
                }
            ]
        }
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
* Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
