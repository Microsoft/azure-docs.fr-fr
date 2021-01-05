---
title: Informations sur la structure d'une définition d'initiative
description: Explique comment les définitions d'initiative de stratégie permettent de regrouper les définitions de stratégie à des fins de déploiement sur les ressources Azure de votre organisation.
ms.date: 10/07/2020
ms.topic: conceptual
ms.openlocfilehash: 8f9c6146e1dde5b5a7f6595c61638319de60a82d
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91876173"
---
# <a name="azure-policy-initiative-definition-structure"></a>Structure d'une définition d'initiative Azure Policy

Les initiatives vous permettent de regrouper en un seul élément plusieurs définitions de stratégies associées pour simplifier les affectations et la gestion. Par exemple, vous pouvez regrouper des définitions de stratégies de balisage associées en une même initiative. Au lieu d’attribuer chaque stratégie individuellement, vous appliquez l’initiative.

Pour créer une définition d'initiative de stratégie, vous devez utiliser JSON. La définition d'initiative de stratégie contient des éléments pour les propriétés suivantes :

- le nom d’affichage
- description
- metadata
- parameters
- définitions de stratégie
- groupes de stratégies (cette propriété fait partie de la fonctionnalité [Conformité réglementaire (préversion)](./regulatory-compliance.md))

L’exemple suivant montre comment créer une initiative pour gérer deux balises : `costCenter` et `productName`. Il utilise deux stratégies intégrées pour appliquer la valeur de balise par défaut.

```json
{
    "properties": {
        "displayName": "Billing Tags Policy",
        "policyType": "Custom",
        "description": "Specify cost Center tag and product name tag",
        "metadata": {
            "version": "1.0.0",
            "category": "Tags"
        },
        "parameters": {
            "costCenterValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for Cost Center tag"
                },
                "defaultValue": "DefaultCostCenter"
            },
            "productNameValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for product Name tag"
                },
                "defaultValue": "DefaultProduct"
            }
        },
        "policyDefinitions": [{
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            }
        ]
    }
}
```

Les modèles et les éléments intégrés Azure Policy sont disponibles sous [Exemples Azure Policy](../samples/index.md).

## <a name="metadata"></a>Métadonnées

La propriété facultative `metadata` stocke les informations relatives à la définition d'initiative de stratégie.
Les clients peuvent définir toutes les propriétés et valeurs utiles à leur organisation dans `metadata`. Cependant, certaines propriétés _communes_ sont utilisées par Azure Policy et dans les éléments intégrés.

### <a name="common-metadata-properties"></a>Propriétés de métadonnées communes

- `version` (chaîne) : assure le suivi des informations relatives à la version du contenu d'une définition d'initiative de stratégie.
- `category` (chaîne) : détermine sous quelle catégorie du portail Azure la définition de stratégie apparaît.

  > [!NOTE]
  > Pour une initiative [Conformité réglementaire](./regulatory-compliance.md), la `category` doit être **Conformité réglementaire**.

- `preview` (booléen) : indicateur true ou false permettant de déterminer si la définition d'initiative de stratégie est en _préversion_.
- `deprecated` (booléen) : indicateur true ou false permettant de déterminer si la définition d'initiative de stratégie a été marquée comme _déconseillée_.

> [!NOTE]
> Le service Azure Policy utilise les propriétés `version`, `preview` et `deprecated` pour transmettre le niveau de changement à la définition ou à initiative et à l’état d’une stratégie intégrée. Le format de `version` est le suivant : `{Major}.{Minor}.{Patch}`. Les états spécifiques, tels que _déprécié_ ou _préversion_, sont ajoutés à la propriété `version` ou à toute autre propriété en tant que valeur **booléenne**. Pour plus d'informations sur la façon dont les versions d'Azure Policy sont intégrées, consultez [Contrôle des versions des éléments intégrés](https://github.com/Azure/azure-policy/blob/master/built-in-policies/README.md).

## <a name="parameters"></a>Paramètres

Les paramètres permettent de simplifier la gestion des stratégies en réduisant le nombre de définitions de stratégies. Considérez les paramètres comme les champs d’un formulaire : `name`, `address`, `city`, `state`. Ces paramètres restent toujours les mêmes ; toutefois, leurs valeurs changent en fonction de la personne qui remplit le formulaire.
Les paramètres fonctionnent de la même manière lors de l'élaboration des initiatives de stratégie. En incluant des paramètres dans une définition d'initiative de stratégie, vous pouvez réutiliser ce paramètre dans les stratégies incluses.

> [!NOTE]
> Une fois qu’une initiative est attribuée, il n’est plus possible de modifier les paramètres au niveau de l’initative. Pour cette raison, il est recommandé de définir une **defaultValue** lors de la définition du paramètre.

### <a name="parameter-properties"></a>Propriétés du paramètre

Un paramètre possède les propriétés suivantes, qui sont utilisées dans la définition d'initiative de stratégie :

- `name`: Nom de votre paramètre. Utilisé par la fonction de déploiement `parameters` dans le cadre de la règle de stratégie. Pour plus d’informations, consultez [Utilisation d’une valeur de paramètre](#passing-a-parameter-value-to-a-policy-definition).
- `type`: Détermine si le paramètre est une **chaîne**, un **tableau**, un **objet**, **booléen**, **entier**, **flottant**, ou **DateHeure**.
- `metadata`: Définit les sous-propriétés utilisées principalement par le portail Azure pour afficher des informations conviviales :
  - `description`: Explication du rôle du paramètre. Utilisable pour fournir des exemples de valeurs acceptables.
  - `displayName`: Nom convivial du paramètre visible dans le portail.
  - `strongType`: (Facultatif) Utilisé lors de l’affectation de la définition de stratégie via le portail. Fournit une liste prenant en compte le contexte. Pour plus d’informations, voir [strongType](#strongtype).
- `defaultValue`: (Facultatif) Définit la valeur du paramètre dans une affectation si aucune valeur n’est fournie.
- `allowedValues`: (Facultatif) Fournit le tableau des valeurs que le paramètre accepte pendant l’attribution.

À titre d'exemple, vous pouvez définir une définition d'initiative de stratégie de manière à limiter l'emplacement des ressources dans les différentes définitions de stratégie incluses. Le paramètre **allowedLocations** pourrait s'appliquer à cette définition d'initiative de stratégie. Le paramètre est alors disponible pour chaque définition de stratégie incluse et défini lors de l'attribution de l'initiative de stratégie.

```json
"parameters": {
    "init_allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": [ "westus2" ],
        "allowedValues": [
            "eastus2",
            "westus2",
            "westus"
        ]
    }
}
```

### <a name="passing-a-parameter-value-to-a-policy-definition"></a>Transmettre une valeur de paramètre à une définition de stratégie

Vous déclarez les paramètres d'initiative que vous transmettez à telle ou telle définition de stratégie incluse dans le tableau [policyDefinitions](#policy-definitions) de la définition d'initiative. Bien que le nom du paramètre puisse être le même, l'utilisation dans les initiatives de noms différents de ceux utilisés dans les définitions de stratégie simplifie la lisibilité du code.

Par exemple, le paramètre d'initiative **init_allowedLocations** défini précédemment peut être transmis à plusieurs définitions de stratégie incluses et à leurs paramètres, **sql_locations** et **vm_locations**, comme suit :

```json
"policyDefinitions": [
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/0ec8fc28-d5b7-4603-8fec-39044f00a92b",
        "policyDefinitionReferenceId": "allowedLocationsSQL",
        "parameters": {
            "sql_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    },
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/aa09bd0f-aa5f-4343-b6ab-a33a6a6304f3",
        "policyDefinitionReferenceId": "allowedLocationsVMs",
        "parameters": {
            "vm_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    }
]
```

Cet exemple fait référence au paramètre **init_allowedLocations** qui a été démontré dans les [propriétés du paramètre](#parameter-properties).

### <a name="strongtype"></a>strongType

Dans la propriété `metadata`, vous pouvez utiliser **strongType** pour fournir une liste à choix multiple des options dans le portail Azure. **strongType** peut être un _type de ressource_ pris en charge ou une valeur autorisée. Pour déterminer si un _type de ressource_ est valide pour **strongType**, utilisez la commande [Get-AzResourceProvider](/powershell/module/az.resources/get-azresourceprovider).

Certains types de ressources non retournés par la commande **Get-AzResourceProvider** sont pris en charge. Ces types de ressources sont les suivants :

- `Microsoft.RecoveryServices/vaults/backupPolicies`

En dehors du type de ressource, les valeurs autorisées pour **strongType** sont les suivantes :

- `location`
- `resourceTypes`
- `storageSkus`
- `vmSKUs`
- `existingResourceGroups`

## <a name="policy-definitions"></a>Définitions de stratégies

La partie `policyDefinitions` de la définition d'initiative est un _tableau_ dont les définitions de stratégie existantes sont incluses dans l'initiative. Comme mentionné dans [Transmettre une valeur de paramètre à une définition de stratégie](#passing-a-parameter-value-to-a-policy-definition), cette propriété permet de transmettre les [paramètres d'initiative](#parameters) à la définition de stratégie.

### <a name="policy-definition-properties"></a>Propriétés de la définition de stratégie

Chaque élément du _tableau_ qui représente une définition de stratégie comporte les propriétés suivantes :

- `policyDefinitionId` (chaîne) : ID de la définition de stratégie personnalisée ou intégrée à inclure.
- `policyDefinitionReferenceId` (chaîne) : nom court de la définition de stratégie incluse.
- `parameters`: (facultatif) paires nom/valeur permettant de transmettre un paramètre d'initiative à la définition de stratégie incluse en tant que propriété dans cette définition de stratégie. Pour plus d’informations, consultez [Paramètres](#parameters).
- `groupNames` (tableau de chaînes) : (facultatif) groupe dont la définition de stratégie est membre. Pour plus d'informations, consultez [Groupes de stratégies](#policy-definition-groups).

Exemple de `policyDefinitions` comportant deux définitions de stratégie incluses qui reçoivent chacune le même paramètre d'initiative :

```json
"policyDefinitions": [
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/0ec8fc28-d5b7-4603-8fec-39044f00a92b",
        "policyDefinitionReferenceId": "allowedLocationsSQL",
        "parameters": {
            "sql_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    },
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/aa09bd0f-aa5f-4343-b6ab-a33a6a6304f3",
        "policyDefinitionReferenceId": "allowedLocationsVMs",
        "parameters": {
            "vm_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    }
]
```

## <a name="policy-definition-groups"></a>Groupes de définition de stratégie

Les définitions de stratégie dans une définition d’initiative peuvent être regroupées et classées par catégorie. La fonctionnalité [Conformité réglementaire](./regulatory-compliance.md) (préversion) d’Azure Policy utilise cette propriété pour regrouper les définitions dans des **contrôles** et des **domaines de conformité**. Ces informations sont définies dans la propriété `policyDefinitionGroups` _tableau_. Des détails de regroupement supplémentaires sont disponibles dans un objet **policyMetadata** créé par Microsoft. Pour plus d'informations, consultez [Objets de métadonnées](#metadata-objects).

### <a name="policy-definition-groups-parameters"></a>Paramètres des groupes de définitions de stratégie

Chaque élément du _tableau_ de `policyDefinitionGroups` doit disposer des deux propriétés suivantes :

- `name` (chaîne) \[obligatoire\] : Nom court du **groupe**. Dans Conformité réglementaire, le **contrôle**. La valeur de cette propriété est utilisée par `groupNames` dans `policyDefinitions`.
- `category` (chaîne) : Hiérarchie à laquelle le groupe appartient. Dans Conformité réglementaire, le **domaine de conformité** du contrôle.
- `displayName` (chaîne) : Nom convivial du **groupe** ou du **contrôle**. Utilisé par le portail.
- `description` (chaîne) : Description de ce que couvre le **groupe** ou le **contrôle**.
- `additionalMetadataId` (chaîne) : emplacement de l'objet [policyMetadata](#metadata-objects) contenant des détails supplémentaires sur le **contrôle** et le **domaine de conformité**.

  > [!NOTE]
  > Les clients peuvent pointer vers un objet [policyMetadata](#metadata-objects) existant. Toutefois, ces objets sont _en lecture seule_ et uniquement créés par Microsoft.

Exemple de propriété `policyDefinitionGroups` de la définition d'initiative intégrée NIST :

```json
"policyDefinitionGroups": [
    {
        "name": "NIST_SP_800-53_R4_AC-1",
        "additionalMetadataId": "/providers/Microsoft.PolicyInsights/policyMetadata/NIST_SP_800-53_R4_AC-1"
    }
]
```

### <a name="metadata-objects"></a>Objets de métadonnées

Les éléments intégrés créés par Microsoft pour la fonctionnalité Conformité réglementaire contiennent des informations supplémentaires sur chaque contrôle.
Ces informations sont :

- affichées sur le portail Azure dans la présentation d'un **contrôle** sur une initiative Conformité réglementaire ;
- disponibles via l'API REST (voir le `Microsoft.PolicyInsights`fournisseur de ressources et le [groupe d'opérations policyMetadata](/rest/api/policy-insights/policymetadata/getresource)) ;
- disponibles via Azure CLI (voir la commande [az policy metadata](/cli/azure/policy/metadata)).

> [!IMPORTANT]
> Les objets de métadonnées de la fonctionnalité Conformité réglementaire sont _en lecture seule_ et ne peuvent pas être créés par les clients.

Les métadonnées d'un regroupement de stratégies comportent les informations suivantes dans le nœud `properties` :

- `metadataId`: **ID de contrôle** auquel le regroupement se rapporte.
- `category` (obligatoire) : **domaine de conformité** auquel appartient le **contrôle**.
- `title` (obligatoire) : nom convivial de l'**ID du contrôle**.
- `owner` (obligatoire) : identifie qui est responsable du contrôle dans Azure : _Client_, _Microsoft_, _Partagé_.
- `description`: informations supplémentaires sur le contrôle.
- `requirements`: détails sur la responsabilité de l'implémentation du contrôle.
- `additionalContentUrl`: lien d'accès à des informations supplémentaires sur le contrôle. Cette propriété est généralement un lien vers la section de la documentation qui couvre ce contrôle dans la norme de conformité.

Vous trouverez ci-dessous un exemple de l'objet **policyMetadata**. Cet exemple de métadonnées appartient au contrôle _NIST SP 800-53 R4 AC-1_.

```json
{
  "properties": {
    "metadataId": "NIST SP 800-53 R4 AC-1",
    "category": "Access Control",
    "title": "Access Control Policy and Procedures",
    "owner": "Shared",
    "description": "**The organization:**    \na. Develops, documents, and disseminates to [Assignment: organization-defined personnel or roles]:  \n1. An access control policy that addresses purpose, scope, roles, responsibilities, management commitment, coordination among organizational entities, and compliance; and  \n2. Procedures to facilitate the implementation of the access control policy and associated access controls; and  \n  
\nb. Reviews and updates the current:  \n1. Access control policy [Assignment: organization-defined frequency]; and  \n2. Access control procedures [Assignment: organization-defined frequency].",
    "requirements": "**a.**  The customer is responsible for developing, documenting, and disseminating access control policies and procedures. The customer access control policies and procedures address access to all customer-deployed resources and customer system access (e.g., access to customer-deployed virtual machines, access to customer-built applications).  \n**b.**  The customer is responsible for reviewing and updating access control policies and procedures in accordance with FedRAMP requirements.",
    "additionalContentUrl": "https://nvd.nist.gov/800-53/Rev4/control/AC-1"
  },
  "id": "/providers/Microsoft.PolicyInsights/policyMetadata/NIST_SP_800-53_R4_AC-1",
  "name": "NIST_SP_800-53_R4_AC-1",
  "type": "Microsoft.PolicyInsights/policyMetadata"
}
```

## <a name="next-steps"></a>Étapes suivantes

- Consultez la [structure d'une définition](./definition-structure.md)
- Consultez des exemples à la page [Exemples Azure Policy](../samples/index.md).
- Consultez la page [Compréhension des effets de Policy](effects.md).
- Découvrez comment [créer des stratégies par programmation](../how-to/programmatically-create.md).
- Découvrez comment [obtenir des données de conformité](../how-to/get-compliance-data.md).
- Découvrez comment [corriger des ressources non conformes](../how-to/remediate-resources.md).
- Pour en savoir plus sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).
