---
title: Vue d’ensemble de la définition de la vue
description: Décrit le concept de création d'une définition de vue pour les applications managées Azure.
ms.topic: conceptual
ms.author: lazinnat
author: lazinnat
ms.date: 06/12/2019
ms.openlocfilehash: bff846b4b64778d5e40ea7f08f88faf3dde81d9e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91371607"
---
# <a name="view-definition-artifact-in-azure-managed-applications"></a>Artefact de définition de vue dans les applications managées Azure

La définition de vue est un artefact facultatif dans les applications managées Azure. Elle permet de personnaliser la page de présentation et d'ajouter plus de vues telles que Métriques et Ressources personnalisées.

Cet article présente l'artefact de définition de vue et ses fonctionnalités.

## <a name="view-definition-artifact"></a>Afficher un artefact de définition

L’artefact de définition de vue doit être nommé **viewDefinition.json** et placé au même niveau que **createUiDefinition.json** et **mainTemplate.json** dans le package .zip qui crée une définition d’application managée. Pour savoir comment créer le package .zip et publier une définition d'application managée, consultez [Publier une définition d’application managée](publish-service-catalog-app.md)

## <a name="view-definition-schema"></a>Schéma de définition de vue

Le fichier **viewDefinition.json** présente une seule propriété `views` de niveau supérieur, qui est un tableau de vues. Chaque vue s'affiche dans l'interface utilisateur de l'application managée sous forme d'élément de menu distinct dans la table des matières. Chaque vue possède une propriété `kind` qui définit le type de vue. Elle doit être définie sur l’une des valeurs suivantes : [Vue d’ensemble](#overview), [Métriques](#metrics), [Ressources personnalisées](#custom-resources), [Associations](#associations). Pour plus d’informations, consultez le [schéma JSON actuel pour viewDefinition.json](https://schema.management.azure.com/schemas/viewdefinition/0.0.1-preview/ViewDefinition.json#).

Exemple de schéma JSON pour la définition de vue :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/viewdefinition/0.0.1-preview/ViewDefinition.json#",
    "contentVersion": "0.0.0.1",
    "views": [
        {
            "kind": "Overview",
            "properties": {
                "header": "Welcome to your Azure Managed Application",
                "description": "This managed application is for demo purposes only.",
                "commands": [
                    {
                        "displayName": "Test Action",
                        "path": "testAction"
                    }
                ]
            }
        },
        {
            "kind": "Metrics",
            "properties": {
                "displayName": "This is my metrics view",
                "version": "1.0.0",
                "charts": [
                    {
                        "displayName": "Sample chart",
                        "chartType": "Bar",
                        "metrics": [
                            {
                                "name": "Availability",
                                "aggregationType": "avg",
                                "resourceTagFilter": [ "tag1" ],
                                "resourceType": "Microsoft.Storage/storageAccounts",
                                "namespace": "Microsoft.Storage/storageAccounts"
                            }
                        ]
                    }
                ]
            }
        },
        {
            "kind": "CustomResources",
            "properties": {
                "displayName": "Test custom resource type",
                "version": "1.0.0",
                "resourceType": "testCustomResource",
                "createUIDefinition": { },
                "commands": [
                    {
                        "displayName": "Custom Context Action",
                        "path": "testCustomResource/testContextAction",
                        "icon": "Stop",
                        "createUIDefinition": { }
                    }
                ],
                "columns": [
                    {"key": "name", "displayName": "Name"},
                    {"key": "properties.myProperty1", "displayName": "Property 1"},
                    {"key": "properties.myProperty2", "displayName": "Property 2", "optional": true}
                ]
            }
        },
        {
            "kind": "Associations",
            "properties": {
                "displayName": "Test association resource type",
                "version": "1.0.0",
                "targetResourceType": "Microsoft.Compute/virtualMachines",
                "createUIDefinition": { }
            }
        }
    ]
}
```

## <a name="overview"></a>Vue d’ensemble

`"kind": "Overview"`

Lorsque vous fournissez cette vue dans **viewDefinition.json**, elle remplace la page Vue d'ensemble par défaut dans votre application managée.

```json
{
    "kind": "Overview",
    "properties": {
        "header": "Welcome to your Azure Managed Application",
        "description": "This managed application is for demo purposes only.",
        "commands": [
            {
                "displayName": "Test Action",
                "path": "testAction"
            }
        ]
    }
}
```

|Propriété|Obligatoire|Description|
|---------|---------|---------|
|en-tête|Non|En-tête de la page Vue d'ensemble.|
|description|Non|Description de votre application managée.|
|commandes|Non|Pour le tableau de boutons de barre d’outils supplémentaires de la page Vue d’ensemble, consultez [commandes](#commands).|

![La capture d’écran montre la vue d’ensemble d’une application managée avec un contrôle d’action de test pour exécuter une application de démonstration.](./media/view-definition/overview.png)

## <a name="metrics"></a>Mesures

`"kind": "Metrics"`

La vue Métriques vous permet de collecter et d'agréger les données de vos ressources d’application managée dans [Métriques Azure Monitor](../../azure-monitor/platform/data-platform-metrics.md).

```json
{
    "kind": "Metrics",
    "properties": {
        "displayName": "This is my metrics view",
        "version": "1.0.0",
        "charts": [
            {
                "displayName": "Sample chart",
                "chartType": "Bar",
                "metrics": [
                    {
                        "name": "Availability",
                        "aggregationType": "avg",
                        "resourceTagFilter": [ "tag1" ],
                        "resourceType": "Microsoft.Storage/storageAccounts",
                        "namespace": "Microsoft.Storage/storageAccounts"
                    }
                ]
            }
        ]
    }
}
```

|Propriété|Obligatoire|Description|
|---------|---------|---------|
|displayName|Non|Titre affiché de la vue.|
|version|Non|Version de la plateforme utilisée pour afficher la vue.|
|graphiques|Oui|Tableau de graphiques de la page de métriques.|

### <a name="chart"></a>Graphique

|Propriété|Obligatoire|Description|
|---------|---------|---------|
|displayName|Oui|Titre affiché du graphique.|
|chartType|Non|Visualisation à utiliser pour ce graphique. Graphique en courbes utilisé par défaut. Types de graphiques pris en charge : `Bar, Line, Area, Scatter`.|
|Mesures|Oui|Tableau de métriques à tracer sur ce graphique. Pour en savoir plus sur les métriques prises en charge dans le portail Azure, consultez [Métriques prises en charge avec Azure Monitor](../../azure-monitor/platform/metrics-supported.md)|

### <a name="metric"></a>Métrique

|Propriété|Obligatoire|Description|
|---------|---------|---------|
|name|Oui|Nom de la mesure.|
|aggregationType|Oui|Type d’agrégation à utiliser pour cette métrique. Types d’agrégation pris en charge : `none, sum, min, max, avg, unique, percentile, count`|
|espace de noms|Non|Informations supplémentaires à utiliser pour déterminer le fournisseur de métriques qui convient.|
|resourceTagFilter|Non|Tableau de balises de ressource (seront séparées par le mot `or`) pour lesquelles les métriques s'affichent. S’applique en plus du filtre de type de ressource.|
|resourceType|Oui|Type de ressource pour laquelle les métriques s'affichent.|

![Capture d’écran montrant une page de surveillance des métriques pour une application managée.](./media/view-definition/metrics.png)

## <a name="custom-resources"></a>Ressources personnalisées

`"kind": "CustomResources"`

Vous pouvez définir plusieurs vues de ce type. Chaque vue représente un type de ressource personnalisé **unique** dans le fournisseur personnalisé que vous avez défini dans **mainTemplate.json**. Pour une introduction aux fournisseurs personnalisés, consultez [Présentation de la préversion d’Azure Custom Providers](../custom-providers/overview.md).

Dans cette vue, vous pouvez effectuer des opérations GET, PUT, DELETE et POST pour votre type de ressource personnalisé. Les opérations POST peuvent correspondre à des actions personnalisées globales ou actions personnalisées dans le contexte de votre type de ressource personnalisé.

```json
{
    "kind": "CustomResources",
    "properties": {
        "displayName": "Test custom resource type",
        "version": "1.0.0",
        "resourceType": "testCustomResource",
        "icon": "Polychromatic.ResourceList",
        "createUIDefinition": { },
        "commands": [
            {
                "displayName": "Custom Context Action",
                "path": "testCustomResource/testContextAction",
                "icon": "Stop",
                "createUIDefinition": { },
            }
        ],
        "columns": [
            {"key": "name", "displayName": "Name"},
            {"key": "properties.myProperty1", "displayName": "Property 1"},
            {"key": "properties.myProperty2", "displayName": "Property 2", "optional": true}
        ]
    }
}
```

|Propriété|Obligatoire|Description|
|---------|---------|---------|
|displayName|Oui|Titre affiché de la vue. Le titre doit être **unique** pour chaque vue CustomResources de votre fichier **viewDefinition.json**.|
|version|Non|Version de la plateforme utilisée pour afficher la vue.|
|resourceType|Oui|Type de ressource personnalisé. Doit correspondre à un type de ressource personnalisé **unique** de votre fournisseur personnalisé.|
|icon|Non|Icône de l’affichage. La liste des exemples d’icônes est définie dans [Schéma JSON](https://schema.management.azure.com/schemas/viewdefinition/0.0.1-preview/ViewDefinition.json#).|
|createUIDefinition|Non|Créez le schéma de définition d’interface utilisateur pour créer une commande de ressource personnalisée. Pour une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md)|
|commandes|Non|Pour le tableau de boutons de barre d’outils supplémentaires de la page Ressources personnalisées, consultez [commandes](#commands).|
|colonnes|Non|Tableau de colonnes de la ressource personnalisée. S'il n'est pas défini, la colonne `name` s'affiche par défaut. La colonne doit contenir `"key"` et `"displayName"`. Pour la clé, indiquez la clé de la propriété à afficher dans une vue. Si elle est imbriquée, utilisez un point en tant que séparateur, par exemple, `"key": "name"` ou `"key": "properties.property1"`. Pour le nom d’affichage, indiquez le nom complet de la propriété à afficher dans une vue. Vous pouvez également fournir une propriété `"optional"`. Lorsqu'elle est définie sur true, la colonne est masquée dans une vue par défaut.|

![Capture d’écran montrant une page de ressources pour le test du type de ressource personnalisée et l’action de contexte personnalisé de contrôle.](./media/view-definition/customresources.png)

## <a name="commands"></a>Commandes

Les commandes correspondent à un tableau de boutons de barre d’outils supplémentaires qui s'affichent sur la page. Chaque commande représente une action POST de votre fournisseur personnalisé Azure défini dans **mainTemplate.json**. Pour une présentation des fournisseurs personnalisés, consultez [Vue d'ensemble des fournisseurs personnalisés Azure](../custom-providers/overview.md).

```json
{
    "commands": [
        {
            "displayName": "Start Test Action",
            "path": "testAction",
            "icon": "Start",
            "createUIDefinition": { }
        },
    ]
}
```

|Propriété|Obligatoire|Description|
|---------|---------|---------|
|displayName|Oui|Nom affiché du bouton de commande.|
|path|Oui|Nom de l'action du fournisseur personnalisé. L’action doit être définie dans **mainTemplate.json**.|
|icon|Non|Icône du bouton de commande. La liste des exemples d’icônes est définie dans [Schéma JSON](https://schema.management.azure.com/schemas/viewdefinition/0.0.1-preview/ViewDefinition.json#).|
|createUIDefinition|Non|Créez le schéma de définition d’interface utilisateur pour la commande. Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).|

## <a name="associations"></a>Associations

`"kind": "Associations"`

Vous pouvez définir plusieurs vues de ce type. Cette vue vous permet de lier des ressources existantes à l’application managée par le biais du fournisseur personnalisé que vous avez défini dans **mainTemplate.json**. Pour une introduction aux fournisseurs personnalisés, consultez [Présentation de la préversion d’Azure Custom Providers](../custom-providers/overview.md).

Dans cette vue, vous pouvez étendre les ressources Azure existantes en fonction du `targetResourceType`. Lorsqu’une ressource est sélectionnée, elle crée une requête d’intégration au fournisseur personnalisé **public**, qui peut appliquer un effet secondaire à la ressource. 

```json
{
    "kind": "Associations",
    "properties": {
        "displayName": "Test association resource type",
        "version": "1.0.0",
        "targetResourceType": "Microsoft.Compute/virtualMachines",
        "createUIDefinition": { }
    }
}
```

|Propriété|Obligatoire|Description|
|---------|---------|---------|
|displayName|Oui|Titre affiché de la vue. Le titre doit être **unique** pour chaque vue Associations de votre fichier **viewDefinition.json**.|
|version|Non|Version de la plateforme utilisée pour afficher la vue.|
|targetResourceType|Oui|Le type de ressource cible. Il s’agit du type de ressource qui sera affiché pour l’intégration des ressources.|
|createUIDefinition|Non|Créez le schéma de définition d’interface utilisateur pour créer une commande de ressource d’association. Pour une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md)|

## <a name="looking-for-help"></a>Besoin d’aide

Si vous avez des questions sur Applications managées Azure, n’hésitez pas à vous rendre sur le site [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-managedapps). Avant de la publier, vérifiez si votre réponse a déjà été posée et a déjà reçu une réponse. Ajoutez le mot clé `azure-managedapps` pour obtenir une réponse rapide !

## <a name="next-steps"></a>Étapes suivantes

- Pour voir une présentation des applications gérées, consultez [Vue d’ensemble des applications gérées Azure](overview.md).
- Pour une présentation des fournisseurs personnalisés, consultez [Vue d'ensemble des fournisseurs personnalisés Azure](../custom-providers/overview.md).
- Pour créer une application managée Azure avec des fournisseurs personnalisés Azure, consultez le tutoriel [ : Créer une application managée avec des actions des fournisseurs et des types de ressources personnalisés](tutorial-create-managed-app-with-custom-provider.md)
