---
title: Réutiliser les modèles entre les clouds
description: Développez des modèles Azure Resource Manager qui fonctionnent de manière identique pour des environnements cloud différents. Créez ou mettez à jour des modèles existants pour Azure Stack.
author: marcvaneijk
ms.topic: conceptual
ms.date: 12/09/2018
ms.author: mavane
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: ea010a625c3e3cd6228513299d878733bf3775ce
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92744759"
---
# <a name="develop-arm-templates-for-cloud-consistency"></a>Développer des modèles ARM pour la cohérence du cloud

[!INCLUDE [requires-azurerm](../../../includes/requires-azurerm.md)]

La cohérence est l’un des principaux avantages d’Azure. Les investissements de développement pour un emplacement sont réutilisables pour un autre. Un modèle Azure Resource Manager (ARM) rend vos déploiements cohérents et reproductibles entre différents environnements, y compris Azure global, les clouds souverains Azure et Azure Stack. Toutefois, pour réutiliser des modèles dans différents clouds, vous devez prendre en compte les dépendances spécifiques au cloud, comme l’explique ce guide.

Microsoft propose des services cloud intelligents adaptés à l’entreprise dans de nombreux emplacements, notamment :

* La plateforme Azure globale est prise en charge par un réseau croissant de centres de données gérés par Microsoft dans différentes régions partout dans le monde.
* Clouds souverains isolés comme Azure Germany, Azure Government et Azure China 21Vianet. Les clouds souverains fournissent une plateforme cohérente avec la plupart des fonctionnalités auxquelles les clients Azure globaux ont accès.
* Azure Stack, une plateforme cloud hybride permettant de créer des services Azure à partir du centre de données de votre organisation. Les entreprises peuvent configurer Azure Stack dans leurs propres centres de données ou consommer des Services Azure auprès de fournisseurs de services, en exécutant Azure Stack sur leurs installations (parfois appelées régions hébergées).

Au cœur de tous ces clouds, Azure Resource Manager fournit une API qui permet à de nombreuses interfaces utilisateur de communiquer avec la plateforme Azure. Cette API vous offre des fonctionnalités puissantes d’infrastructure en tant que code. Tous les types de ressource disponibles sur la plateforme cloud Azure peuvent être déployés et configurés avec Azure Resource Manager. Avec un seul modèle, vous pouvez déployer et configurer votre application complète pour un état final opérationnel.

![Environnements Azure](./media/templates-cloud-consistency/environments.png)

La cohérence d’Azure global, les clouds souverains, les clouds hébergés, et un cloud dans votre centre de données vous permettent de tirer parti d’Azure Resource Manager. Vous pouvez réutiliser vos investissements de développement dans ces clouds lorsque vous configurez un déploiement de ressources basé sur un modèle.

Toutefois, même si les clouds hybrides, hébergés, souverains et globaux fournissent des services cohérents, les clouds ne sont pas tous identiques. Par conséquent, vous pouvez créer un modèle avec des dépendances de fonctionnalités disponibles uniquement dans un cloud spécifique.

Le reste de ce guide décrit les domaines à prendre en compte lors de la planification du développement de nouveaux modèles ou de la mise à jour de modèles existants ARM pour Azure Stack. En général, votre checklist doit comprendre les éléments suivants :

* Vérifiez que les fonctions, les points de terminaison, les services et les autres ressources de votre modèle sont disponibles dans les emplacements de déploiement cible.
* Stockez les modèles imbriqués et les artefacts de configuration dans des emplacements accessibles, pour en assurer l’accès à travers les clouds.
* Utilisez des références dynamiques à la place d’éléments et de liens de codage en dur.
* Vérifiez que les paramètres de modèle que vous utilisez fonctionnent dans les clouds cibles.
* Vérifiez que les propriétés propres aux ressources sont disponibles pour les clouds cibles.

Pour une présentation des modèles ARM, consultez [Déploiement de modèle](overview.md).

## <a name="ensure-template-functions-work"></a>Assurer le fonctionnement des fonctions des modèles

La syntaxe de base du modèle ARM est JSON. Les modèles utilisent un sur-ensemble de JSON, étendant la syntaxe avec des expressions et des fonctions. Le processeur de langage de modèle est fréquemment mis à jour pour prendre en charge les fonctions de modèle supplémentaires. Pour obtenir une explication détaillée des fonctions de modèle disponibles, consultez [Fonctions des modèles ARM](template-functions.md).

Les nouvelles fonctions de modèle intégrées à Azure Resource Manager ne sont pas immédiatement disponibles dans les clouds souverains ou dans Azure Stack. Pour déployer un modèle avec succès, toutes les fonctions référencées dans le modèle doivent être disponibles dans le cloud cible.

Les fonctionnalités d’Azure Resource Manager sont toujours introduites à Azure global en premier. Vous pouvez utiliser le script PowerShell suivant pour vérifier si les fonctions de modèle nouvellement introduites sont également disponibles dans Azure Stack :

1. Clonez le référentiel GitHub : [https://github.com/marcvaneijk/arm-template-functions](https://github.com/marcvaneijk/arm-template-functions).

1. Une fois que vous avez un clone local du référentiel, connectez-vous au Azure Resource Manager de la destination avec PowerShell.

1. Importez le module psm1 et exécutez l’applet de commande Test-AzureRmTemplateFunctions :

   ```powershell
   # Import the module
   Import-module <path to local clone>\AzTemplateFunctions.psm1

   # Execute the Test-AzureRmTemplateFunctions cmdlet
   Test-AzureRmTemplateFunctions -path <path to local clone>
   ```

Le script déploie plusieurs modèles réduits, chacun contenant seulement des fonctions de modèle uniques. La sortie du script signale les fonctions de modèle prises en charge et non disponibles.

## <a name="working-with-linked-artifacts"></a>Utilisation des artefacts liés

Un modèle peut contenir des références aux artefacts liés et une ressource de déploiement qui établit un lien vers un autre modèle. Les modèles liés (également appelés modèles imbriqués) sont récupérés par Resource Manager au moment de l’exécution. Un modèle peut également contenir des références à des artefacts pour les extensions de machine virtuelle. Ces artefacts sont récupérés par l’extension de machine virtuelle en cours d’exécution à l’intérieur de la machine virtuelle pour configurer l’extension de machine virtuelle pendant le déploiement du modèle.

Les sections suivantes décrivent les considérations relatives à la cohérence du cloud lors du développement de modèles qui comprennent des artefacts externes au modèle de déploiement principal.

### <a name="use-nested-templates-across-regions"></a>Utiliser des modèles imbriqués dans différentes régions

Les modèles peuvent être décomposés en petits modèles ayant chacun un but spécifique, et peuvent être réutilisés dans des scénarios de déploiement. Pour exécuter un déploiement, vous devez spécifier un modèle unique appelé modèle principal. Il spécifie les ressources à déployer, telles que les réseaux virtuels, les machines virtuelles et les applications web. Le modèle principal peut également contenir un lien vers un autre modèle, ce qui signifie que vous pouvez imbriquer des modèles. De même, un modèle imbriqué peut contenir des liens vers d’autres modèles. Vous pouvez imbriquer jusqu’à cinq niveaux de profondeur.

Le code suivant explique comment le paramètre templateLink fait référence à un modèle imbriqué :

```json
"resources": [
  {
     "type": "Microsoft.Resources/deployments",
     "apiVersion": "2017-05-10",
     "name": "linkedTemplate",
     "properties": {
       "mode": "incremental",
       "templateLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/vNet.json",
          "contentVersion":"1.0.0.0"
       }
     }
  }
]
```

Azure Resource Manager évalue le modèle principal au moment de l’exécution, puis extrait et évalue chaque modèle imbriqué. Une fois tous les modèles imbriqués récupérés, le modèle est aplati et un traitement supplémentaire est lancé.

### <a name="make-linked-templates-accessible-across-clouds"></a>Rendre les modèles liés accessibles dans différents clouds

Déterminez où et comment stocker les modèles liés que vous utilisez. Au moment de l’exécution, Azure Resource Manager récupère les modèles liés, et nécessite donc un accès direct à ces modèles. Une pratique courante consiste à utiliser GitHub pour stocker les modèles imbriqués. Un référentiel GitHub peut contenir des fichiers accessibles publiquement via une URL. Bien que cette technique fonctionne bien pour le cloud public et les clouds souverains, un environnement Azure Stack peut se trouver sur un réseau d’entreprise ou sur un emplacement distant déconnecté, sans accès Internet sortant. Dans ce cas, Azure Resource Manager ne réussit pas à récupérer les modèles imbriqués.

Une meilleure pratique relative aux déploiements entre clouds consiste à stocker vos modèles liés dans un emplacement accessible au cloud cible. Dans l’idéal, tous les artefacts de déploiement sont conservés et déployés à partir d’un pipeline d’intégration continue/de déploiement continu (CI/CD). Sinon, vous pouvez stocker les modèles imbriqués dans un conteneur de stockage d’objets blob, d’où Azure Resource Manager peut les récupérer.

Étant donné que le stockage d’objets blob de chaque cloud utilise un nom de domaine complet (FQDN) de point de terminaison différent, configurez le modèle avec l’emplacement des modèles liés avec deux paramètres. Les paramètres peuvent accepter une entrée utilisateur au moment du déploiement. Les modèles sont généralement créés et partagés par plusieurs personnes. Il est donc recommandé d’utiliser un nom standard pour ces paramètres. Les conventions d’affectation de noms permettent de réutiliser plus facilement les modèles entre les régions, les clouds et les auteurs.

Dans le code suivant, `_artifactsLocation` est utilisé pour pointer vers un emplacement unique contenant tous les artefacts liés au déploiement. Notez qu’une valeur est fournie par défaut. Au moment du déploiement, si aucune valeur d’entrée n’est spécifiée pour `_artifactsLocation`, la valeur par défaut est utilisée. Le `_artifactsLocationSasToken` est utilisé comme entrée pour le `sasToken`. La valeur par défaut doit être une chaîne vide pour les scénarios où le `_artifactsLocation` n’est pas sécurisé ; par exemple, un référentiel GitHub public.

```json
"parameters": {
  "_artifactsLocation": {
    "type": "string",
    "metadata": {
      "description": "The base URI where artifacts required by this template are located."
    },
    "defaultValue": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-custom-script-windows/"
  },
  "_artifactsLocationSasToken": {
    "type": "securestring",
    "metadata": {
      "description": "The sasToken required to access _artifactsLocation."
    },
    "defaultValue": ""
  }
}
```

Dans le modèle, les liens sont générés en combinant l’URI de base (à partir du paramètre `_artifactsLocation`) avec un chemin d’accès relatif à l’artefact et le `_artifactsLocationSasToken`. Le code suivant explique comment spécifier le lien vers le modèle imbriqué à l’aide de la fonction de modèle d’URI :

```json
"resources": [
  {
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2019-10-01",
    "name": "shared",
    "properties": {
      "mode": "Incremental",
      "templateLink": {
        "uri": "[uri(parameters('_artifactsLocation'), concat('nested/vnet.json', parameters('_artifactsLocationSasToken')))]",
        "contentVersion": "1.0.0.0"
      }
    }
  }
]
```

Avec cette approche, la valeur par défaut est utilisée pour le paramètre `_artifactsLocation`. Si les modèles liés doivent être récupérés à partir d’un autre emplacement, l’entrée du paramètre peut être utilisée au moment du déploiement pour remplacer la valeur par défaut. Aucune modification du modèle n’est nécessaire.

### <a name="use-_artifactslocation-instead-of-hardcoding-links"></a>Utiliser _artifactsLocation à la place de liens de codage en dur

Outre son utilisation pour les modèles imbriqués, l’URL du paramètre `_artifactsLocation` est utilisé comme base pour tous les artefacts associés d’un modèle de déploiement. Certaines extensions de machine virtuelle contiennent un lien vers un script stocké en dehors du modèle. Pour ces extensions, vous ne devez pas coder les liens en dur. Par exemple, les extensions de Script personnalisé et de DSC PowerShell peuvent établir un lien vers un script externe sur GitHub comme suit :

```json
"properties": {
  "publisher": "Microsoft.Compute",
  "type": "CustomScriptExtension",
  "typeHandlerVersion": "1.9",
  "autoUpgradeMinorVersion": true,
  "settings": {
    "fileUris": [
      "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
    ]
  }
}
```

Coder en dur les liens vers le script empêche potentiellement le modèle de se déployer avec succès vers un autre emplacement. Lors de la configuration de la ressource de machine virtuelle, l’agent de machine virtuelle en cours d’exécution à l’intérieur de la machine virtuelle lance un téléchargement de tous les scripts liés dans l’extension de machine virtuelle, puis stocke les scripts sur le disque local de la machine virtuelle. Cette approche fonctionne comme les liens du modèle imbriqué expliqués précédemment dans la section « Utiliser les modèles imbriqués dans différentes régions ».

Resource Manager récupère des modèles imbriqués au moment de l’exécution. Pour les extensions de machine virtuelle, la récupération des artefacts externes est effectuée par l’agent de machine virtuelle. Outre l’initiateur différent de la récupération de l’artefact, la solution dans la définition du modèle reste la même. Utilisez le paramètre _artifactsLocation avec une valeur par défaut du chemin d’accès de base où se trouvent tous les artefacts (y compris les scripts d’extension de machine virtuelle) et le paramètre `_artifactsLocationSasToken` pour l’entrée du sasToken.

```json
"parameters": {
  "_artifactsLocation": {
    "type": "string",
    "metadata": {
      "description": "The base URI where artifacts required by this template are located."
    },
    "defaultValue": "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/"
  },
  "_artifactsLocationSasToken": {
    "type": "securestring",
    "metadata": {
      "description": "The sasToken required to access _artifactsLocation."
    },
    "defaultValue": ""
  }
}
```

Pour construire l’URI absolu d’un artefact, la méthode recommandée est d’utiliser la fonction de modèle d’URI au lieu de la fonction de modèle concat. En remplaçant les liens codés en dur vers les scripts de l’extension de machine virtuelle par la fonction de modèle d’URI, cette fonctionnalité du modèle est configurée pour la cohérence du cloud.

```json
"properties": {
  "publisher": "Microsoft.Compute",
  "type": "CustomScriptExtension",
  "typeHandlerVersion": "1.9",
  "autoUpgradeMinorVersion": true,
  "settings": {
    "fileUris": [
      "[uri(parameters('_artifactsLocation'), concat('scripts/configure-music-app.ps1', parameters('_artifactsLocationSasToken')))]"
    ]
  }
}
```

Grâce à cette approche, tous les artefacts de déploiement, y compris les scripts de configuration, peuvent être stockés au même emplacement avec le modèle lui-même. Pour modifier l’emplacement de tous les liens, vous n’avez qu’à spécifier une URL de base différente pour les _paramètres artifactsLocation_ .

## <a name="factor-in-differing-regional-capabilities"></a>Tenir compte des différentes fonctionnalités régionales

Avec le développement agile et le flux continu de mises à jour et de nouveaux services intégrés à Azure, la disponibilité des services ou des mises à jour [peut varier selon les régions](https://azure.microsoft.com/regions/services/). Après un test interne rigoureux, les nouveaux services ou les mises à jour de services existants sont généralement introduits à un petit groupe de clients participant à un programme de validation. Après la validation du client réussie, les services ou les mises à jour sont mis à disposition dans un sous-ensemble de régions Azure, puis intégrés à d’autres régions, déployés vers les clouds souverains, puis éventuellement mis aussi à disposition des clients Azure Stack.

Sachant que leurs services disponibles peuvent varier dans les clouds et régions Azure, vous pouvez prendre des décisions proactives relatives à vos modèles. Examiner les fournisseurs de ressources disponibles pour un cloud constitue un bon point de départ. Un fournisseur de ressources vous indique l’ensemble des ressources et opérations disponibles pour un service Azure.

Un modèle déploie et configure des ressources. Un type de ressource est fourni par un fournisseur de ressources. Par exemple, le fournisseur de ressources de calcul (Microsoft.Compute) propose plusieurs types de ressources tels que virtualMachines et availabilitySets. Chaque fournisseur de ressources fournit à Azure Resource Manager une API définie par un contrat commun, permettant une expérience de création cohérente et unifiée sur tous les fournisseurs de ressources. Toutefois, un fournisseur de ressources disponible dans Azure global peut ne pas être disponible dans un cloud souverain ou une région Azure Stack.

![Fournisseurs de ressources](./media/templates-cloud-consistency/resource-providers.png)

Pour vérifier quels fournisseurs de ressources sont disponibles dans un cloud donné, exécutez le script suivant dans l’interface de ligne de commande Azure ([CLI](/cli/azure/install-azure-cli)) :

```azurecli-interactive
az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
```

Vous pouvez également utiliser l’applet de commande PowerShell suivante pour afficher les fournisseurs de ressources disponibles :

```azurepowershell-interactive
Get-AzureRmResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState
```

### <a name="verify-the-version-of-all-resource-types"></a>Vérifier la version de tous les types de ressources

Un ensemble de propriétés est commun à tous les types de ressources, mais chaque ressource possède également des propriétés spécifiques. De nouvelles fonctionnalités et propriétés associées sont parfois ajoutées aux types de ressources existants par le biais d’une nouvelle version d’API. Une ressource dans un modèle a sa propre propriété de version d’API : `apiVersion`. Ce contrôle de version garantit qu’une configuration de ressource existante dans un modèle n’est pas affectée par les modifications faites sur la plateforme.

Les nouvelles versions d’API intégrées aux types de ressources existants dans Azure global peuvent ne pas être immédiatement disponibles dans toutes les régions, clouds souverains ou Azure Stack. Pour afficher une liste des fournisseurs de ressources disponibles, des types de ressources, et des versions d’API pour un cloud, vous pouvez utiliser l’Explorateur de ressources dans le portail Azure. Recherchez l’Explorateur de ressources dans le menu Tous les services. Développez le nœud Fournisseurs dans l’Explorateur de ressources pour renvoyer tous les fournisseurs de ressources disponibles, leurs types de ressources et les versions d’API dans ce cloud.

Pour afficher la liste des versions d’API disponibles pour tous les types de ressources dans un cloud donné dans Azure CLI, exécutez le script suivant :

```azurecli-interactive
az provider list --query "[].{namespace:namespace, resourceType:resourceType[]}"
```

Vous pouvez également utiliser l’applet de commande PowerShell suivante :

```azurepowershell-interactive
Get-AzureRmResourceProvider | select-object ProviderNamespace -ExpandProperty ResourceTypes | ft ProviderNamespace, ResourceTypeName, ApiVersions
```

### <a name="refer-to-resource-locations-with-a-parameter"></a>Faire référence à des emplacements de ressources avec un paramètre

Un modèle est toujours déployé dans un groupe de ressources qui se trouve dans une région. En plus du déploiement, chaque ressource d’un modèle possède également une propriété d’emplacement qui vous permet de spécifier la région vers laquelle déployer. Pour développer votre modèle de cohérence de cloud, vous avez besoin de faire référence de manière dynamique à des emplacements de ressources, car chaque compte Azure Stack peut contenir des noms d’emplacement uniques. Les ressources sont généralement déployées dans la même région que le groupe de ressources, mais pour prendre en charge des scénarios tels que la disponibilité des applications entre régions, il peut être utile de répartir les ressources entre les régions.

Bien que vous puissiez coder en dur les noms des régions lorsque vous spécifiez les propriétés de ressource dans un modèle, cette approche ne garantit pas que le modèle peut être déployé dans d’autres environnements Azure Stack, car le nom de la région n’y existe probablement pas.

Pour prendre en compte plusieurs régions, ajoutez un emplacement de paramètre d’entrée au modèle avec une valeur par défaut. La valeur par défaut est utilisée si aucune valeur n’est spécifiée au cours du déploiement.

La fonction de modèle `[resourceGroup()]` retourne un objet contenant les paires clé/valeur suivantes :

```json
{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}",
  "name": "{resourceGroupName}",
  "location": "{resourceGroupLocation}",
  "tags": {
  },
  "properties": {
    "provisioningState": "{status}"
  }
}
```

En référençant la clé d’emplacement de l’objet dans la valeur defaultValue du paramètre d’entrée, Azure Resource Manager, lors de l’exécution, remplacera la fonction de modèle `[resourceGroup().location]` par le nom de l’emplacement du groupe de ressources vers lequel le modèle est déployé.

```json
"parameters": {
  "location": {
    "type": "string",
    "metadata": {
      "description": "Location the resources will be deployed to."
    },
    "defaultValue": "[resourceGroup().location]"
  }
},
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2015-06-15",
    "name": "storageaccount1",
    "location": "[parameters('location')]",
    ...
```

Grâce à cette fonction de modèle, vous pouvez déployer votre modèle vers n’importe quel cloud sans même connaître les noms des régions à l’avance. En outre, l’emplacement d’une ressource spécifique dans le modèle peut être différent de l’emplacement du groupe de ressources. Dans ce cas, vous pouvez le configurer à l’aide des paramètres d’entrée supplémentaires de cette ressource spécifique, tandis que les autres ressources du même modèle utiliseront toujours le paramètre d’entrée d’emplacement initial.

### <a name="track-versions-using-api-profiles"></a>Suivre les versions à l’aide de profils d’API

Il peut être très difficile d’assurer le suivi de tous les fournisseurs de ressources disponibles et des versions d’API associées présents dans Azure Stack. Par exemple, au moment de la rédaction, la dernière version d’API pour **Microsoft.Compute/availabilitySets** dans Azure est `2018-04-01`, tandis que la version d’API disponible commune à Azure et à Azure Stack est `2016-03-30`. La version d’API commune pour **Microsoft.Storage/storageAccounts** partagée entre tous les emplacements Azure et Azure Stack est `2016-01-01`, tandis que la dernière version d’API dans Azure est `2018-02-01`.

C’est pour cette raison que Resource Manager a introduit le concept de profils d’API dans les modèles. Sans profil d’API, chaque ressource dans un modèle est configurée avec un élément `apiVersion` qui décrit la version d’API de cette ressource spécifique.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "metadata": {
          "description": "Location the resources will be deployed to."
      },
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2016-01-01",
      "name": "mystorageaccount",
      "location": "[parameters('location')]",
      "properties": {
        "accountType": "Standard_LRS"
      }
    },
    {
      "type": "Microsoft.Compute/availabilitySets",
      "apiVersion": "2016-03-30",
      "name": "myavailabilityset",
      "location": "[parameters('location')]",
      "properties": {
        "platformFaultDomainCount": 2,
        "platformUpdateDomainCount": 2
      }
    }
  ],
  "outputs": {}
}
```

Une version de profil d’API agit en tant qu’alias pour une seule version d’API, par type de ressource commun à Azure et à Azure Stack. Au lieu de spécifier une version d’API pour chaque ressource du modèle, vous n’avez qu’à spécifier la version de profil d’API dans un nouvel élément racine appelé `apiProfile` et omettre l’élément `apiVersion` pour les ressources individuelles.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "apiProfile": "2018–03-01-hybrid",
    "parameters": {
        "location": {
            "type": "string",
            "metadata": {
                "description": "Location the resources will be deployed to."
            },
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "mystorageaccount",
            "location": "[parameters('location')]",
            "properties": {
                "accountType": "Standard_LRS"
            }
        },
        {
            "type": "Microsoft.Compute/availabilitySets",
            "name": "myavailabilityset",
            "location": "[parameters('location')]",
            "properties": {
                "platformFaultDomainCount": 2,
                "platformUpdateDomainCount": 2
            }
        }
    ],
    "outputs": {}
}
```

Le profil d’API permet de s’assurer que les versions d’API sont disponibles entre les emplacements, vous n’êtes donc pas obligé de vérifier manuellement les apiVersions disponibles dans un emplacement spécifique. Pour vérifier que les versions d’API référencées par votre profil d’API sont présentes dans un environnement Azure Stack, les opérateurs Azure Stack doivent maintenir la solution à jour selon la stratégie de prise en charge. Si un système est obsolète depuis plus de six mois, il est considéré comme non conforme, et l’environnement doit être mis à jour.

Le profil d’API n’est pas un élément nécessaire dans un modèle. Même si vous ajoutez l’élément, il ne sera utilisé que pour les ressources pour lesquelles aucun `apiVersion` n’est spécifié. Cet élément permet des modifications progressives, mais ne nécessite aucune modification des modèles existants.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "apiProfile": "2018–03-01-hybrid",
    "parameters": {
        "location": {
            "type": "string",
            "metadata": {
                "description": "Location the resources will be deployed to."
            },
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2016-01-01",
            "name": "mystorageaccount",
            "location": "[parameters('location')]",
            "properties": {
                "accountType": "Standard_LRS"
            }
        },
        {
            "type": "Microsoft.Compute/availabilitySets",
            "name": "myavailabilityset",
            "location": "[parameters('location')]",
            "properties": {
                "platformFaultDomainCount": 2,
                "platformUpdateDomainCount": 2
            }
        }
    ],
    "outputs": {}
}
```

## <a name="check-endpoint-references"></a>Vérification des références de point de terminaison

Les ressources peuvent avoir des références à d’autres services sur la plateforme. Par exemple, une adresse IP publique peut avoir un nom DNS public affecté. Le cloud public, les clouds souverains et les solutions Azure Stack possèdent leurs propres espaces de noms de point de terminaison distincts. La plupart du temps, une ressource ne nécessite qu’un préfixe en guise d’entrée dans le modèle. Durant l’exécution, Azure Resource Manager y ajoute la valeur de point de terminaison. Certaines valeurs de point de terminaison doivent être spécifiées explicitement dans le modèle.

> [!NOTE]
> Pour développer des modèles de cohérence du cloud, ne codez pas les espaces de noms du point de terminaison en dur.

Les deux exemples suivants sont des espaces de noms de point de terminaison communs qui doivent être spécifiés explicitement lors de la création d’une ressource :

* Comptes de stockage (objets blob, file d’attente, table et fichier)
* Chaînes de connexion pour les bases de données et le cache Redis Azure

Les espaces de noms du point de terminaison peuvent également être utilisés dans la sortie d’un modèle en tant qu’informations pour l’utilisateur lorsque le déploiement est terminé. Voici quelques exemples courants :

* Comptes de stockage (objets blob, file d’attente, table et fichier)
* Chaînes de connexion (MySql, SQLServer, SQLAzure, Custom, NotificationHub, ServiceBus, EventHub, ApiHub, DocDb, RedisCache, PostgreSQL)
* Traffic Manager
* domainNameLabel d’une adresse IP publique
* Services cloud

De manière générale, évitez les points de terminaison codés en dur dans un modèle. La meilleure pratique consiste à utiliser la fonction de modèle de référence pour récupérer les points de terminaison de manière dynamique. Par exemple, le point de terminaison généralement codé en dur est l’espace de noms du point de terminaison des comptes de stockage. Chaque compte de stockage possède un nom de domaine complet (FQDN) unique, construit en concaténant le nom du compte de stockage avec l’espace de noms du point de terminaison. Un compte de stockage d’objets blob nommé mystorageaccount1 donne lieu à différents noms de domaine complets (FQDN) en fonction du cloud :

* **mystorageaccount1.blob.core.windows.net** s’il est créé dans le cloud Azure global.
* **mystorageaccount1.blob.core.chinacloudapi.cn** s’il est créé dans le cloud Azure Chine 21Vianet.

La fonction de modèle de référence suivante récupère l’espace de noms du point de terminaison du fournisseur de ressources de stockage :

```json
"diskUri":"[concat(reference(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))).primaryEndpoints.blob, 'container/myosdisk.vhd')]"
```

En remplaçant la valeur codée en dur du point de terminaison du compte de stockage par la fonction de modèle `reference`, vous pouvez utiliser le même modèle pour déployer avec succès sur différents environnements sans apporter de modification à la référence du point de terminaison.

### <a name="refer-to-existing-resources-by-unique-id"></a>Faire référence à des ressources existantes avec un ID unique

Vous pouvez également faire référence à une ressource existante à partir du même groupe de ressources ou d’un autre groupe de ressources, dans le même abonnement ou dans un autre, au sein du même client dans le même cloud. Pour récupérer les propriétés de ressources, vous devez utiliser l’identificateur unique pour la ressource elle-même. La fonction de modèle `resourceId` récupère l’ID unique d’une ressource telle que SQL Server, comme l’indique le code suivant :

```json
"outputs": {
  "resourceId":{
    "type": "string",
    "value": "[resourceId('otherResourceGroup', 'Microsoft.Sql/servers', parameters('serverName'))]"
  }
}
```

Vous pouvez ensuite utiliser la fonction `resourceId` dans la fonction de modèle `reference` pour récupérer les propriétés d’une base de données. L’objet retourné contient la propriété `fullyQualifiedDomainName` qui détient la valeur complète du point de terminaison. Cette valeur est récupérée au moment de l’exécution et fournit l’espace de noms du point de terminaison spécifique à l’environnement cloud. Pour définir la chaîne de connexion sans coder l’espace de noms du point de terminaison en dur, vous pouvez faire référence à la propriété de l’objet retourné directement dans la chaîne de connexion comme suit :

```json
"[concat('Server=tcp:', reference(resourceId('sql', 'Microsoft.Sql/servers', parameters('test')), '2015-05-01-preview').fullyQualifiedDomainName, ',1433;Initial Catalog=', parameters('database'),';User ID=', parameters('username'), ';Password=', parameters('pass'), ';Encrypt=True;')]"
```

## <a name="consider-resource-properties"></a>Prendre en compte les propriétés des ressources

Les ressources spécifiques des environnements Azure Stack possèdent des propriétés uniques que vous devez prendre en compte dans votre modèle.

### <a name="ensure-vm-images-are-available"></a>Vérifier que les images de machine virtuelle sont disponibles

Azure comporte une sélection étoffée d’images de machine virtuelle. Ces images sont créées et préparées pour le déploiement par Microsoft et ses partenaires. Les images constituent la base des machines virtuelles sur la plateforme. Toutefois, un modèle cohérent de cloud doit uniquement faire référence aux paramètres disponibles, en particulier, le serveur de publication, l’offre et la référence SKU des images de machine virtuelle disponibles pour Azure global, les clouds souverains Azure ou une solution Azure Stack.

Pour récupérer la liste des images de machine virtuelle disponibles dans un emplacement, exécutez la commande Azure CLI suivante :

```azurecli-interactive
az vm image list -all
```

Vous pouvez récupérer la même liste à l’aide de l’applet de commande Azure PowerShell [Get-AzureRmVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher) et spécifier l’emplacement de votre choix avec le paramètre `-Location`. Par exemple :

```azurepowershell-interactive
Get-AzureRmVMImagePublisher -Location "West Europe" | Get-AzureRmVMImageOffer | Get-AzureRmVMImageSku | Get-AzureRmVMImage
```

Cette commande prend quelques minutes pour retourner toutes les images disponibles dans la région Europe Ouest du cloud Azure global.

Si vous avez mis ces images de machine virtuelle à disposition dans Azure Stack, la totalité du stockage disponible doit être utilisée. Pour prendre en compte même la plus petite unité d’échelle, Azure Stack vous permet de sélectionner les images que vous souhaitez ajouter à un environnement.

L’exemple de code suivant propose une approche cohérente pour faire référence au serveur de publication, à l’offre et aux paramètres SKU dans vos modèles ARM :

```json
"storageProfile": {
    "imageReference": {
    "publisher": "MicrosoftWindowsServer",
    "offer": "WindowsServer",
    "sku": "2016-Datacenter",
    "version": "latest"
    }
}
```

### <a name="check-local-vm-sizes"></a>Vérification des tailles de machine virtuelle locale

Pour développer votre modèle de cohérence du cloud, assurez-vous que la taille de machine virtuelle de votre choix est disponible dans tous les environnements cibles. Les tailles de machine virtuelle sont un regroupement de fonctionnalités et de caractéristiques de performances. Certaines tailles de machine virtuelle varient selon le matériel sur lequel la machine virtuelle s’exécute. Par exemple, si vous souhaitez déployer une machine virtuelle optimisée pour le GPU, le matériel qui exécute l’hyperviseur doit disposer du matériel GPU.

Quand Microsoft introduit une nouvelle taille de machine virtuelle qui dispose de certaines dépendances matérielles, la taille de machine virtuelle est généralement d’abord disponible dans un petit sous-ensemble de régions dans le cloud Azure. Ultérieurement, elle est mise à disposition dans d’autres régions et clouds. Pour vous assurer que la taille de machine virtuelle existe dans chaque cloud vers lequel vous déployez, vous pouvez récupérer les tailles disponibles avec la commande Azure CLI suivante :

```azurecli-interactive
az vm list-sizes --location "West Europe"
```

Pour Azure PowerShell, utilisez :

```azurepowershell-interactive
Get-AzureRmVMSize -Location "West Europe"
```

Pour obtenir la liste complète des services actuellement disponibles, consultez [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?cdn=disable).

### <a name="check-use-of-azure-managed-disks-in-azure-stack"></a>Vérifier l’utilisation d’Azure Disques managés dans Azure Stack

Disques Managés gère le stockage pour un client Azure. Au lieu de créer explicitement un compte de stockage et de spécifier l’URI d’un disque dur virtuel (VHD), vous pouvez utiliser des disques managés pour exécuter implicitement ces actions lorsque vous déployez une machine virtuelle. Les disques managés améliorent la disponibilité en plaçant tous les disques de machines virtuelles dans le même groupe à haute disponibilité dans différentes unités de stockage. En outre, les disques durs virtuels (VHD) existants peuvent être convertis du stockage standard au stockage Premium avec beaucoup moins de temps d’arrêt.

Bien que les disques managés se trouvent sur la feuille de route Azure Stack, ils ne sont actuellement pas pris en charge. Tant qu’ils ne le sont pas, vous pouvez développer des modèles cohérents de cloud pour Azure Stack en spécifiant explicitement les disques durs virtuels à l’aide de l’élément `vhd` dans le modèle pour la ressource de machine virtuelle comme suit :

```json
"storageProfile": {
  "imageReference": {
    "publisher": "MicrosoftWindowsServer",
    "offer": "WindowsServer",
    "sku": "[parameters('windowsOSVersion')]",
    "version": "latest"
  },
  "osDisk": {
    "name": "osdisk",
    "vhd": {
      "uri": "[concat(reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2015-06-15').primaryEndpoints.blob, 'vhds/osdisk.vhd')]"
    },
    "caching": "ReadWrite",
    "createOption": "FromImage"
  }
}
```

En revanche, pour spécifier une configuration de disque managé dans un modèle, supprimez l’élément `vhd` de la configuration du disque.

```json
"storageProfile": {
  "imageReference": {
    "publisher": "MicrosoftWindowsServer",
    "offer": "WindowsServer",
    "sku": "[parameters('windowsOSVersion')]",
    "version": "latest"
  },
  "osDisk": {
    "caching": "ReadWrite",
    "createOption": "FromImage"
  }
}
```

Ces modifications s’appliquent également aux [disques de données](../../virtual-machines/using-managed-disks-template-deployments.md).

### <a name="verify-that-vm-extensions-are-available-in-azure-stack"></a>Vérifier que les extensions de machine virtuelle sont disponibles dans Azure Stack

Par souci de cohérence du cloud, vous devez tenir compte de l’utilisation des [extensions de machine virtuelle](../../virtual-machines/extensions/features-windows.md) pour configurer les ressources d’une machine virtuelle. Les extensions de machine virtuelle ne sont pas toutes disponibles dans Azure Stack. Un modèle peut spécifier les ressources dédiées à l’extension de machine virtuelle, en créant des dépendances et des conditions dans le modèle.

Par exemple, si vous souhaitez configurer une machine virtuelle exécutant Microsoft SQL Server, l’extension de machine virtuelle peut configurer SQL Server lors du déploiement du modèle. Il est important de savoir ce qui se passe si le modèle de déploiement contient également un serveur d’applications configuré pour créer une base de données sur la machine virtuelle exécutant SQL Server. Outre l’utilisation d’une extension de machine virtuelle pour les serveurs d’applications, vous pouvez configurer la dépendance du serveur d’applications sur le retour réussi de la ressource d’extension de machine virtuelle SQL Server. Cette approche garantit que la machine virtuelle exécutant SQL Server est configurée et disponible lorsque le serveur d’applications est invité à créer la base de données.

L’approche déclarative du modèle vous permet de définir l’état final des ressources et de leurs interdépendances, tandis que la plateforme prend en charge la logique requise pour les dépendances.

#### <a name="check-that-vm-extensions-are-available"></a>Vérifier que les extensions de machine virtuelle sont disponibles

Il existe de nombreux types d’extensions de machine virtuelle. Lorsque vous développez un modèle de cohérence du cloud, assurez-vous de n’utiliser que les extensions disponibles dans toutes les régions que le modèle cible.

Pour obtenir une liste des extensions de machine virtuelle disponibles pour une région spécifique (dans cet exemple, `myLocation`), exécutez la commande Azure CLI suivante :

```azurecli-interactive
az vm extension image list --location myLocation
```

Vous pouvez également exécuter l’applet de commande [Get-AzureRmVmImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher) PowerShell Azure et utiliser `-Location` pour spécifier l’emplacement de l’image de machine virtuelle. Par exemple :

```azurepowershell-interactive
Get-AzureRmVmImagePublisher -Location myLocation | Get-AzureRmVMExtensionImageType | Get-AzureRmVMExtensionImage | Select Type, Version
```

#### <a name="ensure-that-versions-are-available"></a>S’assurer que deux versions sont disponibles

Comme les extensions de machine virtuelle sont des ressources Resource Manager internes, elles ont leurs propres versions d’API. Comme l’indique le code suivant, le type d’extension de machine virtuelle est une ressource imbriquée dans le fournisseur de ressources Microsoft.Compute.

```json
{
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "apiVersion": "2015-06-15",
    "name": "myExtension",
    "location": "[parameters('location')]",
    ...
```

La version d’API de la ressource d’extension de machine virtuelle doit être présente dans tous les emplacements que vous envisagez de cibler avec votre modèle. La dépendance de l’emplacement fonctionne comme la disponibilité de la version d’API du fournisseur de ressources évoquée précédemment dans la section « Vérifier la version de tous les types de ressources ».

Pour obtenir une liste des versions d’API disponibles pour la ressource d’extension de machine virtuelle, utilisez l’applet de commande [Get-AzureRmResourceProvider](/powershell/module/az.resources/get-azresourceprovider) avec le fournisseur de ressources **Microsoft.Compute** , comme suit :

```azurepowershell-interactive
Get-AzureRmResourceProvider -ProviderNamespace "Microsoft.Compute" | Select-Object -ExpandProperty ResourceTypes | Select ResourceTypeName, Locations, ApiVersions | where {$_.ResourceTypeName -eq "virtualMachines/extensions"}
```

Vous pouvez également utiliser des extensions de machine virtuelle pour les jeux de mise à l’échelle de machine virtuelle. Les mêmes conditions d’emplacement s’appliquent. Pour développer votre modèle de cohérence du cloud, assurez-vous que les versions d’API sont disponibles dans tous les emplacements où vous envisagez de déployer le modèle. Pour récupérer les versions d’API de la ressource d’extension de machine virtuelle pour les jeux de mise à l’échelle, utilisez la même applet de commande que précédemment, mais spécifiez le type de ressource des jeux de mise à l’échelle de machine virtuelle comme suit :

```azurepowershell-interactive
Get-AzureRmResourceProvider -ProviderNamespace "Microsoft.Compute" | Select-Object -ExpandProperty ResourceTypes | Select ResourceTypeName, Locations, ApiVersions | where {$_.ResourceTypeName -eq "virtualMachineScaleSets/extensions"}
```

Chaque extension spécifique est gérée. Cette version est montrée dans la propriété `typeHandlerVersion` de l’extension de machine virtuelle. Assurez-vous que la version spécifiée dans l’élément `typeHandlerVersion` des extensions de machine virtuelle de votre modèle est disponible dans les emplacements où vous envisagez de déployer le modèle. Par exemple, le code suivant spécifie la version 1.7 :

```json
{
    "type": "extensions",
    "apiVersion": "2016-03-30",
    "name": "MyCustomScriptExtension",
    "location": "[parameters('location')]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]"
    ],
    "properties": {
        "publisher": "Microsoft.Compute",
        "type": "CustomScriptExtension",
        "typeHandlerVersion": "1.7",
        ...
```

Pour obtenir une liste des versions disponibles d’une extension de machine virtuelle spécifique, utilisez l’applet de commande [Get-AzureRmVMExtensionImage](/powershell/module/az.compute/get-azvmextensionimage). L’exemple suivant récupère les versions disponibles de l’extension de machine virtuelle PowerShell DSC (Configuration de l’état souhaité) à partir de **myLocation** :

```azurepowershell-interactive
Get-AzureRmVMExtensionImage -Location myLocation -PublisherName Microsoft.PowerShell -Type DSC | FT
```

Pour obtenir une liste des serveurs de publication, utilisez la commande [Get-AzureRmVmImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher). Pour demander un type, utilisez la commande [Get-AzureRmVMExtensionImageType](/powershell/module/az.compute/get-azvmextensionimagetype).

## <a name="tips-for-testing-and-automation"></a>Conseils relatifs aux tests et à l’automatisation

Il est difficile d’effectuer le suivi de tous les paramètres connexes, des fonctionnalités et des limitations pendant la création d’un modèle. L’approche courante consiste à développer et à tester des modèles par rapport à un seul cloud avant que d’autres emplacements soient ciblés. Toutefois, plus les tests sont effectués tôt dans le processus de création, moins votre équipe de développement aura de problèmes à résoudre et de code à réécrire. Les déploiements qui échouent en raison des dépendances de l’emplacement peuvent prendre beaucoup de temps à résoudre les problèmes. C’est pourquoi nous vous recommandons d’effectuer des tests automatisés le plus tôt possible dans le cycle de création. Au final, vous aurez besoin de moins de temps pour le développement et de moins de ressources, et vos artefacts cohérents de cloud seront encore plus utiles.

L’image suivante montre un exemple typique d’un processus de développement d’une équipe à l’aide d’un environnement de développement intégré (IDE). Différents types de tests sont exécutés à différents stages de la chronologie. Ici, deux développeurs travaillent sur la même solution, mais ce scénario s’applique autant à un développeur seul qu’à une grande équipe. Chaque développeur crée généralement une copie locale d’un référentiel central, ce qui permet à chacun de travailler sur la copie locale sans affecter les autres qui travaillent peut-être sur les mêmes fichiers.

![Diagramme représentant deux ensembles de tests unitaires et de tests d'intégration en parallèle dans l'IDE local et qui, au fil du processus de développement CI/CD, fusionnent pour se transformer en tests unitaires, puis en tests d'intégration, puis en test de déploiement, puis en déploiement.](./media/templates-cloud-consistency/workflow.png)

Prenez en compte les conseils suivants relatifs aux tests et à l’automatisation :

* Utilisez les outils de test. Par exemple, Visual Studio Code et Visual Studio comprennent IntelliSense et d’autres fonctionnalités qui peuvent vous aider à valider vos modèles.
* Pour améliorer la qualité du code pendant le développement sur l’IDE local, effectuez une analyse statique du code avec des tests unitaires et d’intégration.
* Pour une meilleure expérience lors du développement initial, les tests unitaires et d’intégration émettent uniquement un avertissement lorsqu’un problème est détecté, et se poursuivent. De cette façon, vous pouvez identifier les problèmes à corriger, et hiérarchiser l’ordre des modifications, ce qui également appelé déploiement piloté par les tests (TDD).
* N’oubliez pas que certains tests peuvent être effectués sans être connecté à Azure Resource Manager. D’autres, comme les tests de déploiement de modèle, nécessitent Resource Manager pour effectuer certaines actions qui ne peuvent pas être réalisées hors connexion.
* Tester d’un modèle de déploiement par rapport à l’API de validation n’équivaut pas à un déploiement réel. En outre, même si vous déployez un modèle à partir d’un fichier local, toutes les références à des modèles imbriqués dans le modèle sont directement récupérées par Resource Manager, et les artefacts référencés par les extensions de machine virtuelle sont récupérés par l’agent de machine virtuelle en cours d’exécution dans la machine virtuelle déployée.

## <a name="next-steps"></a>Étapes suivantes

* [Considérations relatives au modèle Azure Resource Manager](/azure-stack/user/azure-stack-develop-templates)
* [Meilleures pratiques relatives aux modèles ARM](template-syntax.md)
