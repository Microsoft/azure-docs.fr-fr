---
title: Élément d’interface utilisateur EditableGrid
description: Décrit l’élément d’interface utilisateur Microsoft.Common.EditableGrid pour le portail Azure. Permet aux utilisateurs de collecter des entrées tabulaires.
author: tfitzmac
ms.topic: conceptual
ms.date: 08/24/2020
ms.author: tomfitz
ms.openlocfilehash: 04f86883a75110985d1cbe050fe3fd3e0582986a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88893613"
---
# <a name="microsoftcommoneditablegrid-ui-element"></a>Élément d’interface utilisateur Microsoft.Common.EditableGrid

Contrôle permettant de collecter les entrées tabulaires. Tous les champs de la grille sont modifiables et le nombre de lignes peut varier.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

:::image type="content" source="./media/managed-application-elements/microsoft-common-editablegrid.png" alt-text="Microsoft.Common.EditableGrid":::

## <a name="schema"></a>schéma

```json
{
  "name": "people",
  "type": "Microsoft.Common.EditableGrid",
  "ariaLabel": "Enter information per person",
  "label": "People",
  "constraints": {
    "width": "Full",
    "rows": {
      "count": {
        "min": 1,
        "max": 10
      }
    },
    "columns": [
      {
        "id": "colName",
        "header": "Name",
        "width": "1fr",
        "element": {
          "type": "Microsoft.Common.TextBox",
          "placeholder": "Full name",
          "constraints": {
            "required": true,
            "validations": [
              {
                "isValid": "[startsWith(last(take(steps('grid').people, $rowIndex)).colName, 'contoso')]",
                "message": "Must start with 'contoso'."
              },
              {
                "regex": "^[a-z0-9A-Z]{1,30}$",
                "message": "Only alphanumeric characters are allowed, and the value must be 1-30 characters long."
              }
            ]
          }
        }
      },
      {
        "id": "colGender",
        "header": "Gender",
        "width": "1fr",
        "element": {
          "name": "dropDown1",
          "type": "Microsoft.Common.DropDown",
          "placeholder": "Select a gender...",
          "constraints": {
            "allowedValues": [
              {
                "label": "Female",
                "value": "female"
              },
              {
                "label": "Male",
                "value": "male"
              },
              {
                "label": "Other",
                "value": "other"
              }
            ],
            "required": true
          }
        }
      },
      {
        "id": "colContactPreference",
        "header": "Contact preference",
        "width": "1fr",
        "element": {
          "type": "Microsoft.Common.OptionsGroup",
          "constraints": {
            "allowedValues": [
              {
                "label": "Email",
                "value": "email"
              },
              {
                "label": "Text",
                "value": "text"
              }
            ],
            "required": true
          }
        }
      }
    ]
  }
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
{
  "colName": "contoso",
  "colGender": "female",
  "colContactPreference": "email"
}
```

## <a name="remarks"></a>Notes

- Les seuls contrôles valides dans le tableau de colonnes sont [TextBox](microsoft-common-textbox.md), [OptionsGroup](microsoft-common-optionsgroup.md) et [DropDown](microsoft-common-dropdown.md).
- La variable `$rowIndex` n’est valide que dans les expressions contenues dans les enfants des colonnes de la grille. Il s’agit d’un entier qui représente l’index de ligne relatif de l’élément et le nombre commence à un et s’incrémente d’une unité. Comme indiqué dans la section `"columns":` du schéma, `$rowIndex` est utilisé pour la validation.
- Lorsque les validations sont effectuées à l’aide de la variable `$rowIndex`, il est possible d’extraire la valeur de la ligne actuelle en combinant les commandes `last()` et `take()`.

  Par exemple :

  `last(take(<reference_to_grid>, $rowIndex))`

- La propriété `label` n’apparaît pas dans le contrôle, mais s’affiche dans le résumé de l’onglet final.
- La propriété `ariaLabel` est l’étiquette d’accessibilité de la grille. Spécifiez un texte utile pour les utilisateurs qui utilisent des lecteurs d’écran.
- La propriété `constraints.width` est utilisée pour définir la largeur globale de la grille. Les options sont _Full_ (Complète), _Medium_ (Moyenne), _Small_ (Petite). La valeur par défaut est _Full_ (Complète).
- La propriété `width` sur les enfants des colonnes détermine la largeur de la colonne. Les largeurs sont spécifiées à l’aide d’unités fractionnaires telles que _3fr_. L’espace total alloué aux colonnes est proportionnel à leurs unités. Si aucune largeur de colonne n’est spécifiée, la valeur par défaut est _1fr_.

## <a name="next-steps"></a>Étapes suivantes

- Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
- Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
