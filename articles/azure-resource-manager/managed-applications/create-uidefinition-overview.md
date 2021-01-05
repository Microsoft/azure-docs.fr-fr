---
title: Fichier CreateUiDefinition.json pour le volet du portail
description: Explique comment créer des définitions d’interface utilisateur pour le portail Azure. Utilisé lors de la définition d’applications managées Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: tomfitz
ms.openlocfilehash: 327fa1d7eb73d8e65bb4f81c1dff0fe2bec2913b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89319563"
---
# <a name="createuidefinitionjson-for-azure-managed-applications-create-experience"></a>CreateUiDefinition.json pour une expérience de création d’applications managées Azure

Ce document présente les concepts de base du fichier **createUiDefinition.json**. Le portail Azure utilise ce fichier pour définir l’interface utilisateur lors de la création d’une application managée.

Le modèle est le suivant :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
    "handler": "Microsoft.Azure.CreateUIDef",
    "version": "0.1.2-preview",
    "parameters": {
        "config": {
            "isWizard": false,
            "basics": { }
        },
        "basics": [ ],
        "steps": [ ],
        "outputs": { },
        "resourceTypes": [ ]
    }
}
```

Une fonction `CreateUiDefinition` contient toujours trois propriétés :

* gestionnaire
* version
* parameters

Le gestionnaire doit toujours être `Microsoft.Azure.CreateUIDef` et la version prise en charge la plus récente est `0.1.2-preview`.

Le schéma de la propriété des paramètres dépend de la version et du gestionnaire spécifiés. Pour les applications managées, les propriétés prises en charge sont `config`, `basics`, `steps` et `outputs`. Vous utilisez `config` uniquement lorsque vous devez remplacer le comportement par défaut de l’étape `basics`. Les propriétés basics et steps contiennent des [éléments](create-uidefinition-elements.md), comme des zones de texte et des listes déroulantes, à afficher dans le portail Azure. La propriété « outputs » est utilisée pour mapper les valeurs de sortie des éléments spécifiés aux paramètres du modèle Resource Manager.

L’inclusion de `$schema` est recommandée, mais facultative. Si la valeur de `version` est spécifiée, celle-ci doit correspondre à la version figurant dans l’`$schema` URI.

Vous pouvez utiliser un éditeur JSON pour créer votre fichier createUiDefinition, puis la tester dans le [sandbox (bac à sable) createUiDefinition](https://portal.azure.com/?feature.customPortal=false&#blade/Microsoft_Azure_CreateUIDef/SandboxBlade) pour en afficher un aperçu. Pour plus d’informations sur le bac à sable, consultez [Tester votre interface de portail pour les Applications managées Azure](test-createuidefinition.md).

## <a name="config"></a>Config

La propriété `config` est facultative. Utilisez-la pour remplacer le comportement par défaut de l’étape de base, ou pour définir votre interface en tant qu’Assistant pas à pas. Si `config` est utilisée, il s’agit de la première propriété de la section `parameters` du fichier **createUiDefinition.json**. L’exemple suivant présente les propriétés disponibles.

```json
"config": {
    "isWizard": false,
    "basics": {
        "description": "Customized description with **markdown**, see [more](https://www.microsoft.com).",
        "subscription": {
            "constraints": {
                "validations": [
                    {
                        "isValid": "[expression for checking]",
                        "message": "Please select a valid subscription."
                    },
                    {
                        "permission": "<Resource Provider>/<Action>",
                        "message": "Must have correct permission to complete this step."
                    }
                ]
            },
            "resourceProviders": [
                "<Resource Provider>"
            ]
        },
        "resourceGroup": {
            "constraints": {
                "validations": [
                    {
                        "isValid": "[expression for checking]",
                        "message": "Please select a valid resource group."
                    }
                ]
            },
            "allowExisting": true
        },
        "location": {
            "label": "Custom label for location",
            "toolTip": "provide a useful tooltip",
            "resourceTypes": [
                "Microsoft.Compute/virtualMachines"
            ],
            "allowedValues": [
                "eastus",
                "westus2"
            ],
            "visible": true
        }
    }
},
```

### <a name="wizard"></a>Assistant

La propriété `isWizard` vous permet de demander une validation réussie de chaque étape avant de passer à l’étape suivante. Lorsque la propriété `isWizard` n’est pas spécifiée, la valeur par défaut est **false** et la validation pas à pas n’est pas nécessaire.

Lorsque `isWizard` est activé, affectez la valeur **true**, l’onglet **de base** est disponible et tous les autres onglets sont désactivés. Quand le bouton **Suivant** est sélectionné, l’icône de l’onglet indique si la validation d’un onglet a réussi ou échoué. Une fois les champs obligatoires d’un onglet remplis et validés, le bouton **Suivant** permet de naviguer jusqu’à l’onglet suivant. Lorsque tous les onglets sont validés, vous pouvez accéder à la page **Vérifier et créer** et sélectionner le bouton **Créer** pour commencer le déploiement.

:::image type="content" source="./media/create-uidefinition-overview/tab-wizard.png" alt-text="Assistant d’onglet":::

### <a name="override-basics"></a>Notions de base des remplacements

La configuration de base vous permet de personnaliser les étapes de base.

Pour `description`, indiquez une chaîne avec démarques qui décrit votre ressource. Le format sur plusieurs lignes et les liens sont pris en charge.

Les éléments `subscription` et `resourceGroup` vous permettent de spécifier des validations supplémentaires. La syntaxe permettant de spécifier les validations est identique à la validation personnalisée pour la [zone de texte](microsoft-common-textbox.md). Vous pouvez également spécifier des validations `permission` sur l’abonnement ou le groupe de ressources.  

Le contrôle d’abonnement accepte une liste d’espaces de noms de fournisseurs de ressources. Par exemple, vous pouvez spécifier **Microsoft.Compute**. Il affiche un message d’erreur lorsque l’utilisateur sélectionne un abonnement qui ne prend pas en charge le fournisseur de ressources. L’erreur se produit lorsque le fournisseur de ressources n’est pas inscrit sur cet abonnement et que l’utilisateur n’est pas autorisé à l’y inscrire.  

Le contrôle de groupe de ressources a une option pour `allowExisting`. Lorsqu’elle est définie sur `true`, les utilisateurs peuvent sélectionner des groupes de ressources qui ont déjà des ressources. Cet indicateur s’applique surtout aux modèles de solutions, où le comportement par défaut oblige les utilisateurs à sélectionner un groupe de ressources nouveau ou vide. Dans la plupart des autres scénarios, il n’est pas nécessaire de spécifier cette propriété.  

Pour `location`, spécifiez les propriétés du contrôle d’emplacement que vous souhaitez remplacer. Toutes les propriétés non remplacées sont définies sur leurs valeurs par défaut. `resourceTypes` accepte un tableau de chaînes contenant des noms complets de types de ressources. Les options d’emplacement sont limitées aux seules régions qui prennent en charge les types de ressources.  `allowedValues` accepte un tableau de chaînes de région. Seules ces régions s’affichent dans la liste déroulante. Vous pouvez définir à la fois `allowedValues` et  `resourceTypes`. Le résultat est l’intersection des deux listes. Enfin, la propriété `visible` peut être utilisée pour désactiver complètement ou sous certaines conditions la liste déroulante des emplacements.  

## <a name="basics"></a>Concepts de base

L’étape **Basics** est la première étape générée lorsque le portail Azure analyse le fichier. Par défaut, l’étape Basics permet aux utilisateurs de choisir l’abonnement, le groupe de ressources et l’emplacement du déploiement.

:::image type="content" source="./media/create-uidefinition-overview/basics.png" alt-text="Assistant d’onglet":::

Vous pouvez ajouter d’autres éléments dans cette section. Lorsque cela est possible, ajoutez des éléments qui interrogent les paramètres de déploiement, comme le nom d’un cluster ou des informations d’identification administrateur.

L’exemple suivant montre une zone de texte qui a été ajoutée aux éléments par défaut.

```json
"basics": [
    {
        "name": "textBox1",
        "type": "Microsoft.Common.TextBox",
        "label": "Textbox on basics",
        "defaultValue": "my text value",
        "toolTip": "",
        "visible": true
    }
]
```

## <a name="steps"></a>Étapes

La propriété Étapes contient zéro ou plusieurs étapes supplémentaires à afficher après les étapes de base. Chaque étape contient un ou plusieurs éléments. Vous pouvez ajouter des étapes par rôle ou niveau de l’application déployée. Par exemple, ajoutez une étape pour les entrées destinées aux nœuds principaux et une étape pour les nœuds Worker à un cluster.

```json
"steps": [
    {
        "name": "demoConfig",
        "label": "Configuration settings",
        "elements": [
          ui-elements-needed-to-create-the-instance
        ]
    }
]
```

## <a name="outputs"></a>Sorties

Le portail Azure utilise la propriété `outputs` pour mettre en correspondance des éléments issus de `basics` et `steps` avec les paramètres du modèle de déploiement Azure Resource Manager. Les clés de ce dictionnaire sont les noms des paramètres du modèle et les valeurs sont les propriétés des objets de sortie issues des éléments référencés.

Pour définir le nom de ressource d’application managée, vous devez inclure une valeur nommée `applicationResourceName` dans la propriété des sorties. Si vous ne définissez pas cette valeur, l’application affecte un GUID au nom. Vous pouvez inclure une zone de texte dans l’interface utilisateur afin de demander un nom à l’utilisateur.

```json
"outputs": {
    "vmName": "[steps('appSettings').vmName]",
    "trialOrProduction": "[steps('appSettings').trialOrProd]",
    "userName": "[steps('vmCredentials').adminUsername]",
    "pwd": "[steps('vmCredentials').vmPwd.password]",
    "applicationResourceName": "[steps('appSettings').vmName]"
}
```

## <a name="resource-types"></a>Types de ressources

Pour filtrer les emplacements disponibles afin d’obtenir uniquement ceux qui prennent en charge les types de ressources à déployer, fournissez un tableau des types de ressources. Si vous fournissez plusieurs types de ressources, seuls les emplacements qui prennent en charge tous les types de ressources sont retournés. Cette propriété est facultative.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
    "handler": "Microsoft.Azure.CreateUIDef",
    "version": "0.1.2-preview",
    "parameters": {
        "resourceTypes": ["Microsoft.Compute/disks"],
        "basics": [
          ...
```  

## <a name="functions"></a>Fonctions

CreateUiDefinition propose des [fonctions](create-uidefinition-functions.md) permettant de travailler avec les entrées et les sorties des éléments, ainsi que des fonctionnalités telles que des logiques conditionnelles. Ces fonctions ont une syntaxe et des fonctionnalités semblables aux fonctions de modèle d’Azure Resource Manager.

## <a name="next-steps"></a>Étapes suivantes

Le fichier createUiDefinition.json proprement dit a un schéma simple. Sa profondeur réelle provient de tous les éléments et fonctions pris en charge. Ces éléments sont décrits plus en détail dans :

- [Éléments](create-uidefinition-elements.md)
- [Fonctions](create-uidefinition-functions.md)

Un schéma JSON actuel pour createUiDefinition est disponible ici : `https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json`.

Pour obtenir un exemple de fichier d’interface utilisateur, voir [createUiDefinition.json](https://github.com/Azure/azure-managedapp-samples/blob/master/Managed%20Application%20Sample%20Packages/201-managed-app-using-existing-vnet/createUiDefinition.json).
