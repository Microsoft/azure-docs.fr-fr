---
title: Détails de la structure des définitions de stratégies
description: Décrit comment les définitions de stratégie permettent d’établir des conventions pour les ressources Azure dans votre organisation.
ms.date: 10/22/2020
ms.topic: conceptual
ms.openlocfilehash: 5f9a110247d4ec93c8f3fb95fc9ed61eb6806787
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93305152"
---
# <a name="azure-policy-definition-structure"></a>Structure de définition Azure Policy

Azure Policy établit des conventions pour les ressources. Les définitions de stratégie décrivent les [conditions](#conditions) de la conformité des ressources et l’effet à exécuter si une condition est remplie. Une condition compare un [champ](#fields) ou une [valeur](#value) de propriété de ressource à une valeur requise. Les champs de propriétés de ressources sont accessibles à l’aide d’[alias](#aliases). Quand un champ de propriété de ressource est un tableau, un [alias de tableau](#understanding-the--alias) spécial peut être utilisé pour sélectionner les valeurs de tous les membres du tableau et appliquer à chacune d’elles une condition.
Apprenez-en davantage sur les [conditions](#conditions).

En définissant des conventions, vous pouvez contrôler les coûts et gérer plus facilement vos ressources. Par exemple, vous pouvez spécifier que seuls certains types de machines virtuelles sont autorisés. Vous pouvez aussi exiger que les ressources soient marquées avec une balise particulière. Les ressources enfants héritent des attributions de stratégie. Si une attribution de stratégie est appliquée à un groupe de ressources, elle s’applique à toutes les ressources appartenant à ce groupe de ressources.

Le schéma de la définition de stratégie _policyRule_ se trouve ici : [https://schema.management.azure.com/schemas/2019-09-01/policyDefinition.json](https://schema.management.azure.com/schemas/2019-09-01/policyDefinition.json)

Vous devez utiliser JSON pour créer une définition de stratégie. La définition de stratégie contient des éléments pour :

- le nom d’affichage
- description
- mode
- metadata
- parameters
- la règle de stratégie
  - évaluation logique
  - effet

Par exemple, le code JSON suivant illustre une stratégie qui limite les emplacements où les ressources sont déployées :

```json
{
    "properties": {
        "displayName": "Allowed locations",
        "description": "This policy enables you to restrict the locations your organization can specify when deploying resources.",
        "mode": "Indexed",
        "metadata": {
            "version": "1.0.0",
            "category": "Locations"
        },
        "parameters": {
            "allowedLocations": {
                "type": "array",
                "metadata": {
                    "description": "The list of locations that can be specified when deploying resources",
                    "strongType": "location",
                    "displayName": "Allowed locations"
                },
                "defaultValue": [ "westus2" ]
            }
        },
        "policyRule": {
            "if": {
                "not": {
                    "field": "location",
                    "in": "[parameters('allowedLocations')]"
                }
            },
            "then": {
                "effect": "deny"
            }
        }
    }
}
```

Les modèles et les éléments intégrés Azure Policy sont disponibles sous [Exemples Azure Policy](../samples/index.md).

## <a name="display-name-and-description"></a>Nom d’affichage et description

**displayName** et **description** permettent de distinguer la définition de stratégie et de préciser son contexte d’utilisation. **displayName** a une longueur maximale de _128_ caractères et **description** a une longueur maximale de _512_ caractères.

> [!NOTE]
> Lors de la création ou de la mise à jour d’une définition de stratégie, **id** , **type** et **name** sont définis par des propriétés externes non comprises dans le code JSON et qui ne sont pas nécessaires au fichier JSON. La récupération de la définition de stratégie via le SDK permet de retourner les propriétés **id** , **type** et **name** dans le code JSON, mais chacune d’elles correspond à des informations en lecture seule qui sont relatives à la définition de stratégie.

## <a name="type"></a>Type

Bien que la propriété du **type** ne puisse pas être définie, trois valeurs sont retournées par le Kit de développement logiciel (SDK) et visibles dans le portail :

- `Builtin`: Ces définitions de stratégie sont fournies et gérées par Microsoft.
- `Custom`: Toutes les définitions de stratégie créées par les clients ont cette valeur.
- `Static`: Indique une définition de stratégie de [conformité réglementaire](./regulatory-compliance.md) avec la **propriété** de Microsoft. Les résultats de conformité pour ces définitions de stratégie sont les résultats des audits tiers sur l’infrastructure Microsoft. Sur le portail Azure, cette valeur est parfois affichée comme étant **managée par Microsoft**. Pour plus d’informations, consultez [Responsabilité partagée dans le cloud](../../../security/fundamentals/shared-responsibility.md).

## <a name="mode"></a>Mode

Le **Mode** est configuré selon que la stratégie cible une propriété Azure Resource Manager ou une propriété de fournisseur de ressources.

### <a name="resource-manager-modes"></a>Modes Resource Manager

Le **mode** détermine les types de ressources à évaluer pour une définition de stratégie. Les modes pris en charge sont les suivants :

- `all` : évaluer les groupes de ressources, les abonnements et tous les types de ressources
- `indexed` : évaluer uniquement les types de ressources qui prennent en charge les balises et l’emplacement

Par exemple, la ressource `Microsoft.Network/routeTables` prend en charge les étiquettes et l’emplacement, et elle est évaluée dans les deux modes. En revanche, la ressource `Microsoft.Network/routeTables/routes` ne peut pas être étiquetée et n’est pas évaluée en mode `Indexed`.

Nous vous recommandons de définir **mode** sur `all` dans tous les cas. Toutes les définitions de stratégie créées via le portail utilisent le mode `all`. Si vous utilisez PowerShell ou Azure CLI, vous pouvez spécifier le paramètre **mode** manuellement. Si la définition de stratégie ne comporte pas de valeur **mode** , elle prend la valeur par défaut `all` dans Azure PowerShell et `null` dans Azure CLI. Le mode `null` a le même effet que `indexed`, à savoir assurer une compatibilité descendante.

Il est recommandé (quoique non obligatoire) d’utiliser `indexed` pour créer des stratégies qui appliquent des balises ou des emplacements, car cela empêche les ressources qui ne prennent pas en charge les balises et les emplacements de s’afficher comme non conformes dans les résultats de conformité. Les **groupes de ressources** et les **abonnements** font figure d’exception. Les définitions de stratégie qui appliquent des emplacements ou des balises à un groupe de ressources ou un abonnement doivent définir le **mode** sur `all` et cibler spécifiquement le type `Microsoft.Resources/subscriptions/resourceGroups` ou `Microsoft.Resources/subscriptions`. Pour obtenir un exemple, consultez [Modèle : Balises – Exemple 1](../samples/pattern-tags.md). Pour obtenir la liste des ressources qui prennent en charge les étiquettes, consultez [Prise en charge des étiquettes pour les ressources Azure](../../../azure-resource-manager/management/tag-support.md).

### <a name="resource-provider-modes"></a>Modes Fournisseur de ressources

Le mode Fournisseur de ressources suivant est entièrement pris en charge :

- `Microsoft.Kubernetes.Data` pour la gestion de vos clusters Kubernetes sur ou hors Azure. Les définitions utilisant ce mode Fournisseur de ressources utilisent les effects _audit_ , _deny_ et _disabled_. L’effet [EnforceOPAConstraint](./effects.md#enforceopaconstraint) est _déconseillé_.

Les modes Fournisseur de ressources suivants sont actuellement pris en charge en **préversion**  :

- `Microsoft.ContainerService.Data` pour la gestion des règles d’admission de contrôleur sur [Azure Kubernetes Service](../../../aks/intro-kubernetes.md). Les définitions utilisant ce mode Fournisseur de ressources **doivent** utiliser l’effet [EnforceRegoPolicy](./effects.md#enforceregopolicy). Ce mode est _déconseillé_.
- `Microsoft.KeyVault.Data` pour la gestion des coffres et des certificats dans [Azure Key Vault](../../../key-vault/general/overview.md). Pour plus d’informations sur ces définitions de stratégie, consultez [Intégrer Azure Key Vault à Azure Policy](../../../key-vault/general/azure-policy.md).

> [!NOTE]
> Les modes Fournisseur de ressources prennent uniquement en charge les définitions de stratégie intégrées et non les [exemptions](./exemption-structure.md).

## <a name="metadata"></a>Métadonnées

La propriété facultative `metadata` stocke les informations relatives à la définition de stratégie. Les clients peuvent définir toutes les propriétés et valeurs utiles à leur organisation dans `metadata`. Cependant, certaines propriétés _communes_ sont utilisées par Azure Policy et dans les éléments intégrés.

### <a name="common-metadata-properties"></a>Propriétés de métadonnées communes

- `version` (chaîne) : Effectue le suivi des détails sur la version du contenu d’une définition de stratégie.
- `category` (chaîne) : détermine sous quelle catégorie du portail Azure la définition de stratégie apparaît.
- `preview` (booléen) : indicateur true ou false permettant de déterminer si la définition de stratégie est en _préversion_.
- `deprecated` (booléen) : indicateur true ou false permettant de déterminer si la définition de stratégie a été marquée comme _déconseillée_.

> [!NOTE]
> Le service Azure Policy utilise les propriétés `version`, `preview` et `deprecated` pour transmettre le niveau de changement à la définition ou à initiative et à l’état d’une stratégie intégrée. Le format de `version` est le suivant : `{Major}.{Minor}.{Patch}`. Les états spécifiques, tels que _déprécié_ ou _préversion_ , sont ajoutés à la propriété `version` ou à toute autre propriété en tant que valeur **booléenne**. Pour plus d’informations sur la façon dont les versions d’Azure Policy sont intégrées, consultez [Contrôle des versions des éléments intégrés](https://github.com/Azure/azure-policy/blob/master/built-in-policies/README.md).

## <a name="parameters"></a>Paramètres

Les paramètres permettent de simplifier la gestion des stratégies en réduisant le nombre de définitions de stratégies. Considérez les paramètres comme les champs d’un formulaire : `name`, `address`, `city`, `state`. Ces paramètres restent toujours les mêmes ; toutefois, leurs valeurs changent en fonction de la personne qui remplit le formulaire.
Les paramètres fonctionnent de manière identique durant la création de stratégies. En incluant des paramètres dans une définition de stratégie, vous pouvez réutiliser cette stratégie pour différents scénarios avec des valeurs différentes.

> [!NOTE]
> Des paramètres peuvent être ajoutés à une définition existante et attribuée. Le nouveau paramètre doit inclure la propriété **defaultValue**. Cela empêche les affectations de stratégie ou d’initiative déjà existantes d’être indirectement invalidées.

### <a name="parameter-properties"></a>Propriétés du paramètre

Un paramètre possède les propriétés suivantes qui sont utilisées dans la définition de la stratégie :

- `name`: Nom de votre paramètre. Utilisé par la fonction de déploiement `parameters` dans le cadre de la règle de stratégie. Pour plus d’informations, consultez [Utilisation d’une valeur de paramètre](#using-a-parameter-value).
- `type`: Détermine si le paramètre est une **chaîne** , un **tableau** , un **objet** , **booléen** , **entier** , **flottant** , ou **DateHeure**.
- `metadata`: Définit les sous-propriétés utilisées principalement par le portail Azure pour afficher des informations conviviales :
  - `description`: Explication du rôle du paramètre. Utilisable pour fournir des exemples de valeurs acceptables.
  - `displayName`: Nom convivial du paramètre visible dans le portail.
  - `strongType`: (Facultatif) Utilisé lors de l’affectation de la définition de stratégie via le portail. Fournit une liste prenant en compte le contexte. Pour plus d’informations, voir [strongType](#strongtype).
  - `assignPermissions`: (Facultatif) Définissez l’option sur _True_ pour que le portail Azure crée des attributions de rôles lors de l’attribution de stratégie. Cette propriété est utile si vous souhaitez attribuer des autorisations en dehors de l’étendue d’attribution. Il existe une attribution de rôle par définition de rôle dans la stratégie (ou par définition de rôle dans toutes les stratégies dans l’initiative). La valeur du paramètre doit être une ressource ou une étendue valide.
- `defaultValue`: (Facultatif) Définit la valeur du paramètre dans une affectation si aucune valeur n’est fournie.
  Obligatoire lors de la mise à jour d’une définition de stratégie existante qui est affectée.
- `allowedValues`: (Facultatif) Fournit le tableau des valeurs que le paramètre accepte pendant l’attribution.

Par exemple, vous pouvez définir une définition de stratégie qui limite les emplacements sur lesquels les ressources peuvent être déployées. Le paramètre **allowedLocations** pourrait s’appliquer à cette définition de stratégie. Ce paramètre serait utilisé par chaque affectation de la définition de la stratégie pour limiter les valeurs acceptées. L’utilisation de **strongType** permet d’améliorer l’expérience lors de l’affectation via le portail :

```json
"parameters": {
    "allowedLocations": {
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

### <a name="using-a-parameter-value"></a>Utiliser une valeur de paramètre

Dans la règle de stratégie, vous référencez des paramètres avec la syntaxe de fonction `parameters` suivante :

```json
{
    "field": "location",
    "in": "[parameters('allowedLocations')]"
}
```

Cet exemple fait référence au paramètre **allowedLocations** autorisé qui a été démontré dans les [propriétés du paramètre](#parameter-properties).

### <a name="strongtype"></a>strongType

Dans la propriété `metadata`, vous pouvez utiliser **strongType** pour fournir une liste à choix multiple des options dans le portail Azure. **strongType** peut être un _type de ressource_ pris en charge ou une valeur autorisée. Pour déterminer si un _type de ressource_ est valide pour **strongType** , utilisez la commande [Get-AzResourceProvider](/powershell/module/az.resources/get-azresourceprovider). Le format d’un _type de ressource_ **strongType** est `<Resource Provider>/<Resource Type>`. Par exemple : `Microsoft.Network/virtualNetworks/subnets`.

Certains _types de ressources_ non retournés par la commande **AzResourceProvider** sont pris en charge. Ces types sont les suivants :

- `Microsoft.RecoveryServices/vaults/backupPolicies`

Les valeurs non _type de ressource_  autorisées pour **strongType** sont les suivantes :

- `location`
- `resourceTypes`
- `storageSkus`
- `vmSKUs`
- `existingResourceGroups`

## <a name="definition-location"></a>Emplacement de la définition

Lors de la création d’une initiative ou d’une stratégie, il est important de spécifier l’emplacement de la définition, qui doit être un groupe d’administration ou un abonnement. L’emplacement détermine l’étendue à laquelle l’initiative ou la stratégie peut être affectée. Les ressources doivent être des membres directs ou des enfants dans la hiérarchie de l’emplacement de la définition à cibler pour l’affectation.

Si l’emplacement de la définition est l’un ou l’autre élément suivant :

- **Abonnement** : seules les ressources au sein de cet abonnement peuvent être affectées à la définition de la stratégie.
- **Groupe d’administration** : seules les ressources au sein des groupes d’administration enfants et des abonnements enfants peuvent être affectées à la définition de la stratégie. Si vous voulez appliquer la définition de stratégie à plusieurs abonnements, l’emplacement doit correspondre à un groupe d’administration comportant chaque abonnement.

Pour plus d’informations, consultez [Comprendre l’étendue d’Azure Policy](./scope.md#definition-location).

## <a name="policy-rule"></a>Règle de stratégie

La règle de stratégie se compose de blocs **if** et **then**. Dans le bloc **if** , vous définissez une ou plusieurs conditions qui spécifient à quel moment la stratégie est mise en œuvre. Vous pouvez appliquer des opérateurs logiques à ces conditions pour définir avec précision le scénario d’une stratégie.

Dans le bloc **then** , vous définissez l’effet qui se produit lorsque les conditions de **si** sont remplies.

```json
{
    "if": {
        <condition> | <logical operator>
    },
    "then": {
        "effect": "deny | audit | modify | append | auditIfNotExists | deployIfNotExists | disabled"
    }
}
```

### <a name="logical-operators"></a>Opérateurs logiques

Les opérateurs logiques pris en charge sont les suivants :

- `"not": {condition  or operator}`
- `"allOf": [{condition or operator},{condition or operator}]`
- `"anyOf": [{condition or operator},{condition or operator}]`

La syntaxe **not** inverse le résultat de la condition. La syntaxe **allOf** (semblable à l’opération logique **And** ) nécessite que toutes les conditions soient remplies. La syntaxe **anyOf** (semblable à l’opération logique **Of** ) nécessite qu’au moins une des conditions soit remplie.

Vous pouvez imbriquer des opérateurs logiques. L’exemple suivant illustre une opération **not** imbriquée dans une opération **allOf**.

```json
"if": {
    "allOf": [{
            "not": {
                "field": "tags",
                "containsKey": "application"
            }
        },
        {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
        }
    ]
},
```

### <a name="conditions"></a>Conditions

Une condition évalue si un **champ** ou un accesseur de **valeur** répond à certains critères. Les conditions prises en charge sont les suivantes :

- `"equals": "stringValue"`
- `"notEquals": "stringValue"`
- `"like": "stringValue"`
- `"notLike": "stringValue"`
- `"match": "stringValue"`
- `"matchInsensitively": "stringValue"`
- `"notMatch": "stringValue"`
- `"notMatchInsensitively": "stringValue"`
- `"contains": "stringValue"`
- `"notContains": "stringValue"`
- `"in": ["stringValue1","stringValue2"]`
- `"notIn": ["stringValue1","stringValue2"]`
- `"containsKey": "keyName"`
- `"notContainsKey": "keyName"`
- `"less": "dateValue"` | `"less": "stringValue"` | `"less": intValue`
- `"lessOrEquals": "dateValue"` | `"lessOrEquals": "stringValue"` | `"lessOrEquals": intValue`
- `"greater": "dateValue"` | `"greater": "stringValue"` | `"greater": intValue`
- `"greaterOrEquals": "dateValue"` | `"greaterOrEquals": "stringValue"` |
  `"greaterOrEquals": intValue`
- `"exists": "bool"`

Pour **less** , **lessOrEquals** , **greater** et **greaterOrEquals** , si le type de propriété ne correspond pas au type de condition, une erreur est générée. Les comparaisons de chaînes sont effectuées à l’aide de `InvariantCultureIgnoreCase`.

Avec les conditions **like** et **notLike** , un caractère générique `*` est indiqué dans la valeur.
Celle-ci ne doit pas en comporter plus d’un (`*`).

Si vous utilisez les conditions **match** et **notMatch** , entrez `#` pour trouver un chiffre, `?` pour une lettre, `.` pour un caractère et tout autre caractère pour représenter ce caractère réel. **match** et **notMatch** sont sensibles à la casse. Cependant, toutes les autres conditions qui évaluent une _stringValue_ ne sont pas sensibles à la casse. Des alternatives non sensibles à la casse sont disponibles dans **matchInsensitively** et **notMatchInsensitively**.

Dans une valeur de champ de tableau à **alias \[\*\]** , chaque élément du tableau est évalué individuellement avec un opérateur logique **and** entre les éléments. Pour plus d’informations, consultez [Références aux propriétés des ressources de tableau](../how-to/author-policies-for-arrays.md#referencing-array-resource-properties).

### <a name="fields"></a>Champs

Les conditions sont formées à partir de champs. Un champ correspond à des propriétés de la charge utile de la demande de ressource et décrit l’état de la ressource.

Les champs suivants sont pris en charge :

- `name`
- `fullName`
  - Retourne le nom complet de la ressource, soit le nom de la ressource précédé du nom de toutes les ressources parentes (par exemple, « monServeur/maBaseDeDonnées »).
- `kind`
- `type`
- `location`
  - Utilisez **global** pour les ressources indépendantes de l’emplacement.
- `id`
  - Retourne l’ID de la ressource qui est évaluée.
  - Exemple : `/subscriptions/06be863d-0996-4d56-be22-384767287aa2/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myVault`
- `identity.type`
  - Renvoie le type d'[identité managée](../../../active-directory/managed-identities-azure-resources/overview.md) activé sur la ressource.
- `tags`
- `tags['<tagName>']`
  - Cette syntaxe en crochet prend en charge les noms de balise contenant des signes de ponctuation tels qu’un trait d’union, un point ou un espace.
  - Où **\<tagName\>** est le nom de l’étiquette pour laquelle vérifier la condition.
  - Exemples : `tags['Acct.CostCenter']` où **Acct.CostCenter** est le nom de l’étiquette.
- `tags['''<tagName>''']`
  - Cette syntaxe en crochet prend en charge les noms de balise contenant des apostrophes en appliquant une séquence d’échappement entre apostrophes doubles.
  - Où **« \<tagName\> »** est le nom de l’étiquette pour laquelle vérifier la condition.
  - Exemple : `tags['''My.Apostrophe.Tag''']` où **'My.Apostrophe.Tag** est le nom de la balise.
- alias de propriété : pour en obtenir la liste, consultez [Alias](#aliases).

> [!NOTE]
> `tags.<tagName>`, `tags[tagName]` et `tags[tag.with.dots]` sont toujours des manières acceptables de déclarer un champ de balises. Toutefois, les expressions préférées sont celles répertoriées ci-dessus.

#### <a name="use-tags-with-parameters"></a>Utiliser des balises avec des paramètres

Une valeur de paramètre peut être passée à un champ de balise. Le passage d’un paramètre à un champ de balise augmente la flexibilité de la définition de stratégie lors de l’attribution de stratégie.

Dans l’exemple suivant, `concat` est utilisé pour créer une recherche dans le champ de balises pour la balise nommée avec la valeur du paramètre **tagName**. Si cette balise n’existe pas, l’effet **Modifier** est utilisé pour ajouter la balise à l’aide de la valeur de la même balise nommée définie sur le groupe de ressources parent des ressources auditées à l’aide de la fonction de recherche `resourcegroup()`.

```json
{
    "if": {
        "field": "[concat('tags[', parameters('tagName'), ']')]",
        "exists": "false"
    },
    "then": {
        "effect": "modify",
        "details": {
            "operations": [{
                "operation": "add",
                "field": "[concat('tags[', parameters('tagName'), ']')]",
                "value": "[resourcegroup().tags[parameters('tagName')]]"
            }],
            "roleDefinitionIds": [
                "/providers/microsoft.authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
            ]
        }
    }
}
```

### <a name="value"></a>Valeur

Les conditions peuvent également être formées à l’aide de **valeur**. **valeur** vérifie les conditions selon les [paramètres](#parameters), les [fonctions de modèle supportées](#policy-functions) ou des littéraux. **valeur** est associée à n’importe quelle [condition](#conditions) prise en charge.

> [!WARNING]
> Si le résultat d’une _fonction de modèle_ est une erreur, la stratégie d’évaluation échoue. Une évaluation ayant échoué correspond à un **refus** implicite. Pour plus d’informations, consultez [Éviter les défaillances des modèles](#avoiding-template-failures). Définissez la propriété [enforcementMode](./assignment-structure.md#enforcement-mode) sur **DoNotEnforce** pour empêcher l’impact d’une évaluation qui a échoué sur des ressources nouvelles ou mises à jour lors du test et de la validation d’une nouvelle définition de stratégie.

#### <a name="value-examples"></a>Exemples de valeur

Cet exemple de règle de stratégie utilise **valeur** pour comparer le résultat de la fonction `resourceGroup()` et la propriété **nom** renvoyée à une condition **like** de `*netrg`. La règle refuse toute ressource qui n’est pas du **type** `Microsoft.Network/*` dans tout groupe de ressources dont le nom se termine par `*netrg`.

```json
{
    "if": {
        "allOf": [{
                "value": "[resourceGroup().name]",
                "like": "*netrg"
            },
            {
                "field": "type",
                "notLike": "Microsoft.Network/*"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

Cet exemple de règle de stratégie utilise **value** pour vérifier si le résultat de plusieurs fonctions imbriquées **est égal à** `true`. La règle refuse toute ressource qui n’a pas au moins trois balises.

```json
{
    "mode": "indexed",
    "policyRule": {
        "if": {
            "value": "[less(length(field('tags')), 3)]",
            "equals": "true"
        },
        "then": {
            "effect": "deny"
        }
    }
}
```

#### <a name="avoiding-template-failures"></a>Éviter les défaillances des modèles

L’utilisation de _fonctions de modèle_ dans **value** (valeur) autorise de nombreuses fonctions imbriquées complexes. Si le résultat d’une _fonction de modèle_ est une erreur, la stratégie d’évaluation échoue. Une évaluation ayant échoué correspond à un **refus** implicite. Voici un exemple d’une **valeur** qui échoue dans certains scénarios :

```json
{
    "policyRule": {
        "if": {
            "value": "[substring(field('name'), 0, 3)]",
            "equals": "abc"
        },
        "then": {
            "effect": "audit"
        }
    }
}
```

L’exemple de règle de stratégie ci-dessus utilise [substring()](../../../azure-resource-manager/templates/template-functions-string.md#substring) pour comparer les trois premiers caractères du **nom** avec **abc**. Si le **nom** a moins de 3 caractères, la fonction `substring()` génère une erreur. Cette erreur fait que la stratégie produit un effet **deny (refuser)** .

Au lieu de cela, utilisez la fonction [if()](../../../azure-resource-manager/templates/template-functions-logical.md#if) pour vérifier si les 3 premiers caractères du **nom** sont égaux à **abc** pour éviter qu’un **nom** contenant moins de  3caractères entraîne une erreur :

```json
{
    "policyRule": {
        "if": {
            "value": "[if(greaterOrEquals(length(field('name')), 3), substring(field('name'), 0, 3), 'not starting with abc')]",
            "equals": "abc"
        },
        "then": {
            "effect": "audit"
        }
    }
}
```

Avec la règle de stratégie révisée, `if()` vérifie la longueur du **nom** avant d’essayer d’obtenir une `substring()` sur une valeur avec moins de 3 caractères. Si le **nom** est trop court, la valeur « ne commence pas par abc » est retournée à la place et comparée à **abc**. Une ressource avec un nom court qui ne commence pas par **abc** fait toujours échouer la règle de stratégie, mais ne provoque plus d’erreur lors de l’évaluation.

### <a name="count"></a>Count

Les conditions qui comptent le nombre de membres d’un tableau dans la charge utile de la ressource satisfaisant une expression de condition peuvent être formées à l’aide d’une expression **count**. Les scénarios courants vérifient si « au moins un des », « un seul des », « tous les » ou « aucun des » membres du tableau remplissent la condition. **count** évalue chaque membre du tableau [\[\*\] alias](#understanding-the--alias) à la recherche d’une expression de condition, et additionne les résultats _true_ , qui sont ensuite comparés à l’opérateur d’expression. Les expressions **count** peuvent être ajoutées jusqu’à 3 fois à une même définition **policyRule**.

La structure de l’expression **count** est :

```json
{
    "count": {
        "field": "<[*] alias>",
        "where": {
            /* condition expression */
        }
    },
    "<condition>": "<compare the count of true condition expression array members to this value>"
}
```

Les propriétés suivantes sont utilisées avec **count**  :

- **count.field** (obligatoire) : contient le chemin du tableau et doit être un alias de tableau. Si le tableau est manquant, l’expression est évaluée à _false_ sans tenir compte de l’expression de condition.
- **count.where** (facultatif) : l’expression de condition pour évaluer individuellement chaque membre du tableau [alias \[\*\]](#understanding-the--alias) de **count.field**. Si cette propriété n’est pas fournie, tous les membres du tableau avec le chemin « field » sont évalués à _true_. Toute [condition](../concepts/definition-structure.md#conditions) peut être utilisée à l’intérieur de cette propriété.
  Il est possible d’utiliser des [opérateurs logiques](#logical-operators) à l’intérieur de cette propriété pour créer des exigences d’évaluation complexes.
- **\<condition\>** (obligatoire) : la valeur est comparée au nombre d’éléments qui ont satisfait l’expression de condition **count.where**. Une [condition](../concepts/definition-structure.md#conditions) numérique doit être utilisée.

Pour plus d’informations sur l’utilisation des propriétés de tableau dans Azure Policy, notamment une explication détaillée de l’évaluation de l’expression count, consultez [Références aux propriétés des ressources de tableau](../how-to/author-policies-for-arrays.md#referencing-array-resource-properties).

#### <a name="count-examples"></a>Exemples de comptage

Exemple 1 : Vérifier si un tableau est vide

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]"
    },
    "equals": 0
}
```

Exemple 2 : Rechercher un seul membre du tableau qui satisfait l’expression de condition

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].description",
            "equals": "My unique description"
        }
    },
    "equals": 1
}
```

Exemple 3 : Rechercher au moins un membre du tableau qui satisfait l’expression de condition

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].description",
            "equals": "My common description"
        }
    },
    "greaterOrEquals": 1
}
```

Exemple 4 : Vérifier que tous les membres du tableau d’objets satisfont l’expression de condition

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].description",
            "equals": "description"
        }
    },
    "equals": "[length(field('Microsoft.Network/networkSecurityGroups/securityRules[*]'))]"
}
```

Exemple 5 : Vérifier qu’au moins un membre du tableau correspond à plusieurs propriétés dans l’expression de condition

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
            "allOf": [
                {
                    "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].direction",
                    "equals": "Inbound"
                },
                {
                    "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].access",
                    "equals": "Allow"
                },
                {
                    "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].destinationPortRange",
                    "equals": "3389"
                }
            ]
        }
    },
    "greater": 0
}
```

Exemple 6 : Utilisez la fonction `field()` à l’intérieur des conditions `where` pour accéder à la valeur littérale du membre du tableau évalué. Cette condition permet de vérifier qu’il n’existe aucune règle de sécurité dont la valeur _priority_ soit paire.

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
          "value": "[mod(first(field('Microsoft.Network/networkSecurityGroups/securityRules[*].priority')), 2)]",
          "equals": 0
        }
    },
    "greater": 0
}
```

### <a name="effect"></a>Résultat

Azure Policy prend en charge les types d’effet suivants :

- **append** : ajoute l’ensemble de champs défini à la requête.
- **audit** : génère un événement d’avertissement dans le journal d’activité, mais ne fait pas échouer la requête.
- **AuditIfNotExists**  : génère un événement d’avertissement dans le journal d’activité si une ressource connexe n’existe pas
- **deny** : génère un événement dans le journal d’activité et fait échouer la requête.
- **DeployIfNotExists**  : déploie une ressource connexe si elle n’existe pas déjà
- **disabled**  : n’évalue pas la conformité des ressources à la règle de stratégie.
- **Modify**  : ajoute, met à jour ou supprime les étiquettes définies dans une ressource
- **EnforceOPAConstraint** (déconseillé) : configure le contrôleur des admissions Open Policy Agent avec Gatekeeper v3 pour les clusters Kubernetes automanagés sur Azure
- **EnforceRegoPolicy** (déconseillé) : configure le contrôleur des admissions Open Policy Agent avec Gatekeeper v2 dans Azure Kubernetes Service

Pour plus d’informations sur chaque effet, l’ordre d’évaluation, les propriétés et des exemples, consultez [Présentation des effets Azure Policy](effects.md).

### <a name="policy-functions"></a>Fonctions de stratégie

Toutes les [fonctions de modèle Resource Manager](../../../azure-resource-manager/templates/template-functions.md) peuvent être utilisées dans une règle de stratégie, à l’exception des fonctions et fonctions définies par l’utilisateur suivantes :

- copyIndex()
- deployment()
- list*
- newGuid()
- pickZones()
- providers()
- reference()
- resourceId()
- variables()

> [!NOTE]
> Ces fonctions sont toujours disponibles dans la partie `details.deployment.properties.template` du déploiement de modèle dans une définition de stratégie **deployIfNotExists**.

La fonction suivante est utilisable dans une règle de stratégie, mais diffère de l’utilisation dans un modèle Resource Manager :

- `utcNow()` : contrairement à un modèle Resource Manager, cette propriété peut être utilisée en dehors de _defaultValue_.
  - Retourne une chaîne qui est définie sur la date et l’heure actuelles au format de date/heure universel ISO 8601 `yyyy-MM-ddTHH:mm:ss.fffffffZ`.

Les fonctions suivantes sont disponibles uniquement dans les règles de stratégie :

- `addDays(dateTime, numberOfDaysToAdd)`
  - **dateTime**  : [obligatoire] chaîne – chaîne au format DateHeure universel ISO 8601 « yyyy-MM-ddTHH:mm:ss.FFFFFFFZ »
  - **numberOfDaysToAdd**  : [obligatoire] nombre entier - nombre de jours à ajouter
- `field(fieldName)`
  - **fieldName**  : [Obligatoire] chaîne - Nom du [champ](#fields) à récupérer
  - Retourne la valeur de ce champ à partir de la ressource en cours d’évaluation par la condition If.
  - `field` est principalement utilisé avec **AuditIfNotExists** et **DeployIfNotExists** pour faire référence aux champs actuellement évalués de la ressource. Vous pouvez en voir une illustration dans [l’exemple DeployIfNotExists](effects.md#deployifnotexists-example).
- `requestContext().apiVersion`
  - Retourne la version d’API de la requête qui a déclenché l’évaluation de la stratégie (par exemple : `2019-09-01`).
    Cette valeur est la version d’API qui a été utilisée dans la requête PUT/PATCH pour les évaluations relatives à la création/mise à jour de ressources. La dernière version de l’API est toujours utilisée lors de l’évaluation de la conformité sur des ressources existantes.
- `policy()`
  - Retourne les informations suivantes sur la stratégie en cours d’évaluation. Les propriétés sont accessibles à partir de l’objet retourné (exemple : `[policy().assignmentId]`).
  
  ```json
  {
    "assignmentId": "/subscriptions/ad404ddd-36a5-4ea8-b3e3-681e77487a63/providers/Microsoft.Authorization/policyAssignments/myAssignment",
    "definitionId": "/providers/Microsoft.Authorization/policyDefinitions/34c877ad-507e-4c82-993e-3452a6e0ad3c",
    "setDefinitionId": "/providers/Microsoft.Authorization/policySetDefinitions/42a694ed-f65e-42b2-aa9e-8052e9740a92",
    "definitionReferenceId": "StorageAccountNetworkACLs"
  }
  ```


- `ipRangeContains(range, targetRange)`
    - **range**  : Chaîne [obligatoire] ; chaîne spécifiant une plage d’adresses IP.
    - **targetRange**  : Chaîne [obligatoire] ; chaîne spécifiant une plage d’adresses IP.

    Retourne une valeur indiquant si la plage d’adresses IP donnée contient la plage d’adresses IP cible. Les plages vides ou la combinaison entre familles d’adresses IP ne sont pas autorisées et entraînent l’échec de l’évaluation.

    Formats pris en charge :
    - Adresse IP unique (exemples : `10.0.0.0`, `2001:0DB8::3:FFFE`)
    - Plage CIDR (exemples : `10.0.0.0/24`, `2001:0DB8::/110`)
    - Plage définie par les adresses IP de début et de fin (exemples : `192.168.0.1-192.168.0.9`, `2001:0DB8::-2001:0DB8::3:FFFF`)


#### <a name="policy-function-example"></a>Exemple de fonction de stratégie

Cet exemple de règle de stratégie utilise la fonction de ressource `resourceGroup` pour obtenir la propriété **name** , combinée au tableau `concat` et à la fonction d’objet, pour créer une condition `like` selon laquelle le nom de ressource commence par le nom du groupe de ressources.

```json
{
    "if": {
        "not": {
            "field": "name",
            "like": "[concat(resourceGroup().name,'*')]"
        }
    },
    "then": {
        "effect": "deny"
    }
}
```

## <a name="aliases"></a>Alias

Les alias de propriété permettent d’accéder aux propriétés spécifiques d’un type de ressource. Les alias permettent de restreindre les valeurs ou les conditions autorisées pour la propriété d’une ressource. Chaque alias correspond aux chemins des différentes versions d’API d’un type de ressource donné. Lors de l’évaluation de la stratégie, le moteur de stratégie obtient le chemin de la propriété de cette version de l’API.

La liste des alias augmente toujours. Pour trouver les alias actuellement pris en charge par Azure Policy, utilisez l’une des méthodes suivantes :

- Extension Azure Policy pour Visual Studio Code (recommandée)

  Utilisez l’[extension Azure Policy pour Visual Studio Code](../how-to/extension-for-vscode.md) afin d’afficher et de découvrir les alias des propriétés de ressources.

  :::image type="content" source="../media/extension-for-vscode/extension-hover-shows-property-alias.png" alt-text="Capture d’écran de l’extension Azure Policy pour Visual Studio Code permettant de survoler une propriété afin d’afficher les noms d’alias." border="false":::

- Azure Resource Graph

  Utilisez l’opérateur `project` pour afficher l’ **alias** d’une ressource.

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

- Azure PowerShell

  ```azurepowershell-interactive
  # Login first with Connect-AzAccount if not using Cloud Shell

  # Use Get-AzPolicyAlias to list available providers
  Get-AzPolicyAlias -ListAvailable

  # Use Get-AzPolicyAlias to list aliases for a Namespace (such as Azure Compute -- Microsoft.Compute)
  (Get-AzPolicyAlias -NamespaceMatch 'compute').Aliases
  ```

  > [!NOTE]
  > Pour rechercher des alias qui peuvent être utilisés avec l’effet [modify](./effects.md#modify), utilisez la commande suivante dans Azure PowerShell **4.6.0** ou version ultérieure :
  >
  > ```azurepowershell-interactive
  > Get-AzPolicyAlias | Select-Object -ExpandProperty 'Aliases' | Where-Object { $_.DefaultMetadata.Attributes -eq 'Modifiable' }
  > ```

- Azure CLI

  ```azurecli-interactive
  # Login first with az login if not using Cloud Shell

  # List namespaces
  az provider list --query [*].namespace

  # Get Azure Policy aliases for a specific Namespace (such as Azure Compute -- Microsoft.Compute)
  az provider show --namespace Microsoft.Compute --expand "resourceTypes/aliases" --query "resourceTypes[].aliases[].name"
  ```

- API REST/ARMClient

  ```http
  GET https://management.azure.com/providers/?api-version=2019-10-01&$expand=resourceTypes/aliases
  ```

### <a name="understanding-the--alias"></a>Comprendre l’alias [*]

Plusieurs des alias disponibles ont une version qui s’affiche sous la forme d’un nom « normal » et une autre avec **\[\*\]** qui lui est attaché. Par exemple :

- `Microsoft.Storage/storageAccounts/networkAcls.ipRules`
- `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]`

L’alias « normal » représente le champ sous la forme d’une valeur unique. Ce champ est réservé aux scénarios de comparaison de correspondance exacte, lorsque l’ensemble de valeurs entier doit être exactement tel que défini, ni plus ni moins.

L’alias **\[\*\]** représente une collection de valeurs sélectionnées à partir des éléments d’une propriété de ressource de tableau. Par exemple :

| Alias | Valeurs sélectionnées |
|:---|:---|
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]` | Éléments du tableau `ipRules`. |
| `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].action` | Valeurs de la propriété `action` des différents éléments du tableau `ipRules`. |

Utilisés dans une condition de [champ](#fields), les alias de tableau permettent de comparer chaque élément de tableau à une valeur cible. Lorsqu’ils sont employés avec une expression [count](#count), il est possible d’effectuer les opérations suivantes :

- Vérifier la taille d’un tableau
- Vérifier combien d’éléments du tableau (tous, certains ou aucun) répondent à une condition complexe
- Vérifier si exactement ***n*** éléments du tableau répondent à une condition complexe

Pour plus d’informations et d’exemples, consultez [Références aux propriétés des ressources de tableau](../how-to/author-policies-for-arrays.md#referencing-array-resource-properties).

## <a name="next-steps"></a>Étapes suivantes

- Voir [Structure d’une définition d’initiative](./initiative-definition-structure.md)
- Consultez des exemples à la page [Exemples Azure Policy](../samples/index.md).
- Consultez la page [Compréhension des effets de Policy](effects.md).
- Découvrez comment [créer des stratégies par programmation](../how-to/programmatically-create.md).
- Découvrez comment [obtenir des données de conformité](../how-to/get-compliance-data.md).
- Découvrez comment [corriger des ressources non conformes](../how-to/remediate-resources.md).
- Pour en savoir plus sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).
