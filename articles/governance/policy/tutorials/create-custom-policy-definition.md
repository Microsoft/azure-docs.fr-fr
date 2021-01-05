---
title: 'Tutoriel : Créer une définition de stratégie personnalisée'
description: Dans ce tutoriel, vous créez une définition de stratégie personnalisée pour Azure Policy afin d’appliquer des règles métier personnalisées sur vos ressources Azure.
ms.date: 10/05/2020
ms.topic: tutorial
ms.openlocfilehash: 24058a2c8428d306c5e53a73393b0d98785831cf
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91876292"
---
# <a name="tutorial-create-a-custom-policy-definition"></a>Tutoriel : Créer une définition de stratégie personnalisée

Une définition de stratégie personnalisée permet aux clients de définir leurs propres règles d’utilisation d’Azure. Ces règles appliquent souvent :

- Des mesures de sécurité
- la gestion des coûts ;
- Des règles propres à l’organisation (par exemple, de nommage ou d’emplacements)

Quel que soit l’objectif de la création d’une stratégie personnalisée, les étapes de définition de la nouvelle stratégie personnalisée sont les mêmes.

Avant de créer une stratégie personnalisée, consultez les [exemples de stratégie](../samples/index.md) pour voir si une stratégie correspond déjà à vos besoins.

Voici les étapes de création d’une stratégie personnalisée :

> [!div class="checklist"]
> - Identifier vos exigences métier
> - Associer chaque exigence à une propriété de ressource Azure
> - Associer la propriété à un alias
> - Déterminer l’effet à utiliser
> - Composer la définition de stratégie

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="identify-requirements"></a>Identifier les exigences

Avant de créer la définition de stratégie, vous devez bien définir l’intention de la stratégie. Dans ce tutoriel, nous utilisons une exigence de sécurité d’entreprise courante pour illustrer les étapes à suivre :

- Chaque compte de stockage doit être activé pour HTTPS
- Chaque compte de stockage doit être désactivé pour HTTP

Vos exigences doivent identifier clairement les deux états de ressource « être » et « ne pas être ».

Nous avons défini l’état attendu de la ressource, mais nous n’avons pas encore défini ce que nous voulons faire avec les ressources non conformes. Azure Policy prend en charge un certain nombre d’[effets](../concepts/effects.md). Dans ce tutoriel, nous définissons l’exigence métier de sorte à empêcher la création des ressources qui sont non conformes aux règles métier. Pour atteindre cet objectif, nous utilisons l’effet [Refuser](../concepts/effects.md#deny). Nous voulons aussi pouvoir suspendre la stratégie pour des attributions spécifiques. Par conséquent, nous utilisons l’effet [Désactivé](../concepts/effects.md#disabled) en le définissant comme [paramètre](../concepts/definition-structure.md#parameters) dans la définition de stratégie.

## <a name="determine-resource-properties"></a>Déterminer les propriétés de ressource

D’après les besoins métier, la ressource Azure à auditer avec Azure Policy est un compte de stockage. Toutefois, nous ne savons pas quelles sont les propriétés à utiliser dans la définition de stratégie. Azure Policy évalue la représentation JSON de la ressource ; nous devons donc déterminer les propriétés qui sont disponibles sur cette ressource.

Il existe de nombreuses façons de déterminer les propriétés d’une ressource Azure. Dans ce tutoriel, nous les examinons toutes :

- Extension Azure Policy pour VS Code
- Modèles Azure Resource Manager (modèles ARM)
  - Exporter une ressource existante
  - Expérience de création
  - Modèles de démarrage rapide (GitHub)
  - Documentation de référence de modèle
- Azure Resource Explorer

### <a name="view-resources-in-vs-code-extension"></a>Afficher des ressources dans l’extension VS Code

L’[extension VS Code](../how-to/extension-for-vscode.md#search-for-and-view-resources) peut être utilisée pour parcourir des ressources dans votre environnement et voir les propriétés Resource Manager sur chaque ressource.

### <a name="arm-templates"></a>Modèles ARM

Il existe plusieurs façons d’examiner un [modèle ARM](../../../azure-resource-manager/templates/template-tutorial-use-template-reference.md) qui inclut la propriété à gérer.

#### <a name="existing-resource-in-the-portal"></a>Ressource existante dans le portail

Pour rechercher des propriétés, le plus simple est d’examiner une ressource existante du même type. Vous pouvez aussi comparer les ressources déjà configurées avec le paramètre que vous voulez appliquer.
Consultez la page **Exporter le modèle** (sous **Paramètres**) dans le portail Azure pour cette ressource spécifique.

> [!WARNING]
> Le modèle ARM exporté par le portail Azure ne peut pas être directement relié à la propriété `deployment` pour un modèle ARM dans une définition de stratégie [deployIfNotExists](../concepts/effects.md#deployifnotexists).

:::image type="content" source="../media/create-custom-policy-definition/export-template.png" alt-text="Capture d’écran de la page Exporter le modèle sur une ressource existante dans le portail Azure." border="false":::

Si la ressource est un compte de stockage, vous voyez un modèle semblable à cet exemple :

```json
...
"resources": [{
    "comments": "Generalized from resource: '/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount'.",
    "type": "Microsoft.Storage/storageAccounts",
    "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
    },
    "kind": "Storage",
    "name": "[parameters('storageAccounts_mystorageaccount_name')]",
    "apiVersion": "2018-07-01",
    "location": "westus",
    "tags": {
        "ms-resource-usage": "azure-cloud-shell"
    },
    "scale": null,
    "properties": {
        "networkAcls": {
            "bypass": "AzureServices",
            "virtualNetworkRules": [],
            "ipRules": [],
            "defaultAction": "Allow"
        },
        "supportsHttpsTrafficOnly": false,
        "encryption": {
            "services": {
                "file": {
                    "enabled": true
                },
                "blob": {
                    "enabled": true
                }
            },
            "keySource": "Microsoft.Storage"
        }
    },
    "dependsOn": []
}]
...
```

Sous **propriétés**, une valeur nommée **supportsHttpsTrafficOnly** est définie sur **false**. Cette propriété a les caractéristique de celle que nous recherchons. De plus, le **type** de la ressource est **Microsoft.Storage/storageAccounts**. Le type nous permet de limiter la stratégie aux ressources de ce type uniquement.

#### <a name="create-a-resource-in-the-portal"></a>Créer une ressource dans le portail

Dans le portail, vous pouvez aussi utiliser l’expérience de création de ressource. Quand vous créez un compte de stockage dans le portail, sous l’onglet **Avancé**, vous avez l’option **Transfert de sécurité obligatoire**. Cette propriété a des options _Désactivé_ et _Activé_. L’icône d’informations confirme que cette option est probablement la propriété qui nous intéresse. Toutefois, le portail n’indique pas le nom de la propriété dans cet écran.

Sous l’onglet **Examiner + créer**, un lien en bas de la page vous permet de **Télécharger un modèle pour l’automatisation**. Sélectionnez le lien pour ouvrir le modèle qui crée la ressource que nous avons configurée. Dans notre exemple, nous voyons deux informations essentielles :

```json
...
"supportsHttpsTrafficOnly": {
    "type": "bool"
}
...
"properties": {
    "accessTier": "[parameters('accessTier')]",
    "supportsHttpsTrafficOnly": "[parameters('supportsHttpsTrafficOnly')]"
}
...
```

Ces informations nous indiquent le type de propriété et confirment que **supportsHttpsTrafficOnly** est la propriété que nous recherchons.

#### <a name="quickstart-templates-on-github"></a>Modèles de démarrage rapide sur GitHub

Les [modèles de démarrage rapide Azure](https://github.com/Azure/azure-quickstart-templates) sur GitHub ont des centaines de modèles ARM destinés à différentes ressources. Ces modèles peuvent être un excellent moyen de trouver la propriété de ressource que vous recherchez. Certaines propriétés peuvent sembler être celles que vous cherchez, mais contrôlent quelque chose de différent.

#### <a name="resource-reference-docs"></a>Documentation de référence de la ressource

Pour confirmer que **supportsHttpsTrafficOnly** est la propriété appropriée, consultez la référence du modèle ARM pour la [ressource de compte de stockage](/azure/templates/microsoft.storage/2018-07-01/storageaccounts) sur le fournisseur de stockage. L’objet de propriétés a une liste des paramètres valides. Sélectionnez le lien [StorageAccountPropertiesCreateParameters-object](/azure/templates/microsoft.storage/2018-07-01/storageaccounts#storageaccountpropertiescreateparameters-object) pour afficher un tableau des propriétés acceptables. **supportsHttpsTrafficOnly** est présente et la description correspond à ce que nous recherchons pour répondre à nos exigences métier.

### <a name="azure-resource-explorer"></a>Azure Resource Explorer

Un autre moyen d’explorer vos ressources Azure est d’utiliser [Azure Resource Explorer](https://resources.azure.com) (préversion). Comme cet outil utilise le contexte de votre abonnement, vous devez vous authentifier sur le site web avec vos informations d’identification Azure. Une fois authentifié, vous pouvez parcourir les fournisseurs, les abonnements, les groupes de ressources et les ressources.

Recherchez une ressource de compte de stockage et examinez ses propriétés. Nous voyons ici la propriété **supportsHttpsTrafficOnly**. Sous l’onglet **Documentation**, nous voyons que la description de la propriété correspond à ce que nous avons trouvé précédemment dans la documentation de référence.

## <a name="find-the-property-alias"></a>Rechercher l’alias de propriété

Nous avons identifié la propriété de la ressource, mais nous devons mapper cette propriété à un [alias](../concepts/definition-structure.md#aliases).

Il existe de nombreuses façons de déterminer les alias d’une ressource Azure. Dans ce tutoriel, nous les examinons toutes :

- Extension Azure Policy pour VS Code
- Azure CLI
- Azure PowerShell
- Azure Resource Graph

### <a name="get-aliases-in-vs-code-extension"></a>Obtenir des alias dans l’extension VS Code

L’extension Azure Policy pour l’extension VS Code facilite la navigation dans vos ressources et la [découverte d’alias](../how-to/extension-for-vscode.md#discover-aliases-for-resource-properties).

> [!NOTE]
> L’extension VS Code expose uniquement les propriétés du mode Resource Manager, et n’affiche pas les propriétés du [mode Fournisseur de ressources](../concepts/definition-structure.md#mode).

### <a name="azure-cli"></a>Azure CLI

Dans Azure CLI, le groupe de commandes `az provider` est utilisé pour rechercher des alias de ressource. Nous filtrons sur l’espace de noms **Microsoft.Storage** d’après les détails que nous avons obtenus précédemment sur la ressource Azure.

```azurecli-interactive
# Login first with az login if not using Cloud Shell

# Get Azure Policy aliases for type Microsoft.Storage
az provider show --namespace Microsoft.Storage --expand "resourceTypes/aliases" --query "resourceTypes[].aliases[].name"
```

Dans les résultats, nous voyons un alias pris en charge par les comptes de stockage, nommé **supportsHttpsTrafficOnly**. L’existence de cet alias signifie que nous pouvons écrire la stratégie pour appliquer nos exigences métier !

### <a name="azure-powershell"></a>Azure PowerShell

Dans Azure PowerShell, l’applet de commande `Get-AzPolicyAlias` est utilisé pour rechercher des alias de ressource. Nous filtrons sur l’espace de noms **Microsoft.Storage** d’après les détails que nous avons obtenus précédemment sur la ressource Azure.

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Use Get-AzPolicyAlias to list aliases for Microsoft.Storage
(Get-AzPolicyAlias -NamespaceMatch 'Microsoft.Storage').Aliases
```

Comme avec Azure CLI, les résultats montrent un alias pris en charge par les comptes de stockage, nommé **supportsHttpsTrafficOnly**.

### <a name="azure-resource-graph"></a>Azure Resource Graph

[Azure Resource Graph](../../resource-graph/overview.md) est un service qui fournit une autre méthode pour rechercher les propriétés des ressources Azure. Voici un exemple de requête permettant d’examiner un seul compte de stockage avec Resource Graph :

```kusto
Resources
| where type=~'microsoft.storage/storageaccounts'
| limit 1
```

```azurecli-interactive
az graph query -q "Resources | where type=~'microsoft.storage/storageaccounts' | limit 1"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type=~'microsoft.storage/storageaccounts' | limit 1"
```

Les résultats sont similaires à ce que nous avons vu dans les modèles ARM et dans Azure Resource Explorer. Cependant, les résultats Azure Resource Graph peuvent également inclure des détails d’[alias](../concepts/definition-structure.md#aliases) en _projetant_ le tableau d’_alias_ :

```kusto
Resources
| where type=~'microsoft.storage/storageaccounts'
| limit 1
| project aliases
```

```azurecli-interactive
az graph query -q "Resources | where type=~'microsoft.storage/storageaccounts' | limit 1 | project aliases"
```

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type=~'microsoft.storage/storageaccounts' | limit 1 | project aliases"
```

Voici un exemple de sortie de compte de stockage pour les alias :

```json
"aliases": {
    "Microsoft.Storage/storageAccounts/accessTier": null,
    "Microsoft.Storage/storageAccounts/accountType": "Standard_LRS",
    "Microsoft.Storage/storageAccounts/enableBlobEncryption": true,
    "Microsoft.Storage/storageAccounts/enableFileEncryption": true,
    "Microsoft.Storage/storageAccounts/encryption": {
        "keySource": "Microsoft.Storage",
        "services": {
            "blob": {
                "enabled": true,
                "lastEnabledTime": "2018-06-04T17:59:14.4970000Z"
            },
            "file": {
                "enabled": true,
                "lastEnabledTime": "2018-06-04T17:59:14.4970000Z"
            }
        }
    },
    "Microsoft.Storage/storageAccounts/encryption.keySource": "Microsoft.Storage",
    "Microsoft.Storage/storageAccounts/encryption.keyvaultproperties.keyname": null,
    "Microsoft.Storage/storageAccounts/encryption.keyvaultproperties.keyvaulturi": null,
    "Microsoft.Storage/storageAccounts/encryption.keyvaultproperties.keyversion": null,
    "Microsoft.Storage/storageAccounts/encryption.services": {
        "blob": {
            "enabled": true,
            "lastEnabledTime": "2018-06-04T17:59:14.4970000Z"
        },
        "file": {
            "enabled": true,
            "lastEnabledTime": "2018-06-04T17:59:14.4970000Z"
        }
    },
    "Microsoft.Storage/storageAccounts/encryption.services.blob": {
        "enabled": true,
        "lastEnabledTime": "2018-06-04T17:59:14.4970000Z"
    },
    "Microsoft.Storage/storageAccounts/encryption.services.blob.enabled": true,
    "Microsoft.Storage/storageAccounts/encryption.services.file": {
        "enabled": true,
        "lastEnabledTime": "2018-06-04T17:59:14.4970000Z"
    },
    "Microsoft.Storage/storageAccounts/encryption.services.file.enabled": true,
    "Microsoft.Storage/storageAccounts/networkAcls": {
        "bypass": "AzureServices",
        "defaultAction": "Allow",
        "ipRules": [],
        "virtualNetworkRules": []
    },
    "Microsoft.Storage/storageAccounts/networkAcls.bypass": "AzureServices",
    "Microsoft.Storage/storageAccounts/networkAcls.defaultAction": "Allow",
    "Microsoft.Storage/storageAccounts/networkAcls.ipRules": [],
    "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]": [],
    "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].action": [],
    "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].value": [],
    "Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules": [],
    "Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*]": [],
    "Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*].action": [],
    "Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*].id": [],
    "Microsoft.Storage/storageAccounts/networkAcls.virtualNetworkRules[*].state": [],
    "Microsoft.Storage/storageAccounts/primaryEndpoints": {
        "blob": "https://mystorageaccount.blob.core.windows.net/",
        "file": "https://mystorageaccount.file.core.windows.net/",
        "queue": "https://mystorageaccount.queue.core.windows.net/",
        "table": "https://mystorageaccount.table.core.windows.net/"
    },
    "Microsoft.Storage/storageAccounts/primaryEndpoints.blob": "https://mystorageaccount.blob.core.windows.net/",
    "Microsoft.Storage/storageAccounts/primaryEndpoints.file": "https://mystorageaccount.file.core.windows.net/",
    "Microsoft.Storage/storageAccounts/primaryEndpoints.queue": "https://mystorageaccount.queue.core.windows.net/",
    "Microsoft.Storage/storageAccounts/primaryEndpoints.table": "https://mystorageaccount.table.core.windows.net/",
    "Microsoft.Storage/storageAccounts/primaryEndpoints.web": null,
    "Microsoft.Storage/storageAccounts/primaryLocation": "eastus2",
    "Microsoft.Storage/storageAccounts/provisioningState": "Succeeded",
    "Microsoft.Storage/storageAccounts/sku.name": "Standard_LRS",
    "Microsoft.Storage/storageAccounts/sku.tier": "Standard",
    "Microsoft.Storage/storageAccounts/statusOfPrimary": "available",
    "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly": false
}
```

Azure Resource Graph peut être utilisé par le biais de [Cloud Shell](https://shell.azure.com) et devient alors un moyen simple et rapide pour explorer les propriétés de vos ressources.

## <a name="determine-the-effect-to-use"></a>Déterminer l’effet à utiliser

Choisir ce que vous allez faire de vos ressources non conformes est presque aussi important que choisir ce que vous devez évaluer en premier lieu. Chaque réponse possible à une ressource non conforme est appelée un [effet](../concepts/effects.md). L’effet contrôle si la ressource non conforme est enregistrée, bloquée, a des données ajoutées ou est associée à un déploiement pour la remettre dans un état conforme.

Dans notre exemple, nous voulons l’effet Refuser pour que les ressources non conformes ne soient pas créées dans notre environnement Azure. Avant de définir l’effet Refuser, commencez par l’auditer pour déterminer l’impact de la stratégie. Pour simplifier le changement d’effet par attribution, vous pouvez paramétrer l’effet. Consultez les [paramètres](#parameters) ci-dessous pour plus d’informations sur la procédure à suivre.

## <a name="compose-the-definition"></a>Composer la définition

Nous avons maintenant les détails et l’alias de la propriété correspondant à ce que nous voulons gérer. Ensuite, nous composons la règle de stratégie proprement dite. Si vous n’êtes pas encore familiarisé avec le langage de stratégie, consultez la [structure de définition de stratégie](../concepts/definition-structure.md) pour savoir comment structurer la définition de stratégie. Voici un modèle vide de définition de stratégie :

```json
{
    "properties": {
        "displayName": "<displayName>",
        "description": "<description>",
        "mode": "<mode>",
        "parameters": {
                <parameters>
        },
        "policyRule": {
            "if": {
                <rule>
            },
            "then": {
                "effect": "<effect>"
            }
        }
    }
}
```

### <a name="metadata"></a>Métadonnées

Les trois premiers composants sont des métadonnées de stratégie. Ces composants sont faciles à définir puisque nous connaissons l’objectif de la règle. Le [mode](../concepts/definition-structure.md#mode) concerne essentiellement les étiquettes et l’emplacement de la ressource. Comme nous n’avons pas besoin de limiter l’évaluation aux ressources qui prennent en charge les étiquettes, nous utilisons la valeur _Tout_ pour le **mode**.

```json
"displayName": "Deny storage accounts not using only HTTPS",
"description": "Deny storage accounts not using only HTTPS. Checks the supportsHttpsTrafficOnly property on StorageAccounts.",
"mode": "all",
```

### <a name="parameters"></a>Paramètres

Nous n’avons pas utilisé de paramètre pour changer l’évaluation, mais nous devons utiliser un paramètre afin de nous autoriser à changer l’**effet** pour la résolution des problèmes. Nous définissons un paramètre **effectType** et le limitons à **Refuser** et **Désactivé** uniquement. Ces deux options correspondent à nos exigences métier. Le bloc de paramètres terminé ressemble à cet exemple :

```json
"parameters": {
    "effectType": {
        "type": "string",
        "defaultValue": "Deny",
        "allowedValues": [
            "Deny",
            "Disabled"
        ],
        "metadata": {
            "displayName": "Effect",
            "description": "Enable or disable the execution of the policy"
        }
    }
},
```

### <a name="policy-rule"></a>Règle de stratégie

La dernière étape de création de notre définition de stratégie personnalisée est la composition de la [règle de stratégie](../concepts/definition-structure.md#policy-rule). Nous avons identifié deux instructions à tester :

- Le **type** du compte de stockage est **Microsoft.Storage/storageAccounts**
- La propriété **supportsHttpsTrafficOnly** du compte de stockage n’a pas la valeur **true**

Comme ces instructions doivent avoir la valeur true, nous utilisons l’[opérateur logique](../concepts/definition-structure.md#logical-operators) **allOf**. Nous passons le paramètre **effectType** à l’effet au lieu de faire une déclaration statique. Notre règle terminée ressemble à cet exemple :

```json
"if": {
    "allOf": [
        {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
        },
        {
            "field": "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly",
            "notEquals": "true"
        }
    ]
},
"then": {
    "effect": "[parameters('effectType')]"
}
```

### <a name="completed-definition"></a>Définition terminée

Voici notre définition terminée avec les trois parties de la stratégie définies :

```json
{
    "properties": {
        "displayName": "Deny storage accounts not using only HTTPS",
        "description": "Deny storage accounts not using only HTTPS. Checks the supportsHttpsTrafficOnly property on StorageAccounts.",
        "mode": "all",
        "parameters": {
            "effectType": {
                "type": "string",
                "defaultValue": "Deny",
                "allowedValues": [
                    "Deny",
                    "Disabled"
                ],
                "metadata": {
                    "displayName": "Effect",
                    "description": "Enable or disable the execution of the policy"
                }
            }
        },
        "policyRule": {
            "if": {
                "allOf": [
                    {
                        "field": "type",
                        "equals": "Microsoft.Storage/storageAccounts"
                    },
                    {
                        "field": "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly",
                        "notEquals": "true"
                    }
                ]
            },
            "then": {
                "effect": "[parameters('effectType')]"
            }
        }
    }
}
```

La définition terminée peut être utilisée pour créer une stratégie. Le portail et chaque SDK (Azure CLI, Azure PowerShell et API REST) acceptent la définition de différentes façons, vous devez donc passer en revue les commandes de chacun d’eux pour vérifier comment les utiliser. Attribuez-la ensuite, à l’aide de l’effet paramétré, aux ressources appropriées pour gérer la sécurité de vos comptes de stockage.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez fini d’utiliser les ressources de ce tutoriel, effectuez les étapes suivantes pour supprimer les affectations ou définitions créées ci-dessus :

1. Sélectionnez **Définitions** (ou **Affectations** si vous essayez de supprimer une affectation) sous **Création** dans la partie gauche de la page Azure Policy.

1. Recherchez la nouvelle définition d’initiative ou de stratégie (ou affectation) à supprimer.

1. Cliquez avec le bouton droit sur la liste ou cliquez sur le bouton de sélection en fin de définition (ou d’affectation), puis sélectionnez **Supprimer la définition** (ou **Supprimer l’affectation**).

## <a name="review"></a>Révision

Dans ce didacticiel, vous avez effectué avec succès les tâches suivantes :

> [!div class="checklist"]
> - Identifier vos exigences métier
> - Associer chaque exigence à une propriété de ressource Azure
> - Associer la propriété à un alias
> - Déterminer l’effet à utiliser
> - Composer la définition de stratégie

## <a name="next-steps"></a>Étapes suivantes

Ensuite, utilisez votre définition de stratégie personnalisée pour créer et attribuer une stratégie :

> [!div class="nextstepaction"]
> [Créer et attribuer une définition de stratégie](../how-to/programmatically-create.md#create-and-assign-a-policy-definition)
