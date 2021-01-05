---
title: Vue d’ensemble – Automatiser le déploiement pour le service Azure Logic Apps
description: Apprenez-en davantage sur l’utilisation de modèles Resource Manager afin d’automatiser le déploiement pour le service Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: article
ms.date: 11/06/2020
ms.openlocfilehash: 4070f373175f3497156ced011a57e2ed7bd6e770
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009770"
---
# <a name="overview-automate-deployment-for-azure-logic-apps-by-using-azure-resource-manager-templates"></a>Présentation : Automatiser le déploiement pour le service Azure Logic Apps à l’aide de modèles Resource Manager

Lorsque vous êtes prêt à automatiser la création et le déploiement de votre application logique, vous pouvez développer la définition de flux de travail sous-jacente de votre application logique en un [modèle Azure Resource Manager](../azure-resource-manager/management/overview.md). Ce modèle définit l’infrastructure, les ressources, les paramètres et d’autres informations pour l’approvisionnement et le déploiement de votre application logique. En définissant des paramètres pour des valeurs qui varient lors du déploiement, opération également appelée *paramétrage*, vous pouvez déployer de façon répétée et cohérente des applications logiques en fonction de différents besoins de déploiement.

Par exemple, si vous déployez dans des environnements à des fins de développement, de test et de production, vous utiliserez probablement des chaînes de connexion différentes pour chaque environnement. Vous pouvez déclarer des paramètres de modèle qui acceptent différentes chaînes de connexion, puis stocker celles-ci dans un [fichier de paramètres](../azure-resource-manager/templates/parameter-files.md) séparé. De cette façon, vous pouvez modifier ces valeurs sans avoir à mettre à jour et à redéployer le modèle. Pour les scénarios où vous avez des valeurs de paramètre sensibles ou qui doivent être sécurisées, telles que des mots de passe et des secrets, vous pouvez les stocker dans [Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md) et faire en sorte que votre fichier de paramètres récupère ces valeurs. Cependant, dans ces scénarios, vous devez effectuer un redéploiement pour récupérer les valeurs actuelles.

La présente vue d’ensemble décrit les attributs d’un modèle de Resource Manager incluant une définition de flux de travail d’application logique. Tant le modèle que votre définition de flux de travail utilisent la syntaxe JSON, mais il existe des différences, car la définition de flux de travail suit également le [schéma du Langage de définition du flux de travail](../logic-apps/logic-apps-workflow-definition-language.md). Par exemple, les expressions de modèle et les expressions de définition de flux de travail diffèrent par la manière dont elles [font référence aux paramètres](#parameter-references) et par les valeurs qu’elles peuvent accepter.

> [!TIP]
> La manière la plus simple d’obtenir un modèle d’application logique paramétré valide prêt pour le déploiement consiste à utiliser Visual Studio (version Community Edition gratuite ou version ultérieure) et les outils Azure Logic Apps pour Visual Studio. Vous pouvez ensuite [créer votre application logique dans Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md) ou [rechercher et télécharger une application logique existante d’Azure dans Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md).
>
> Vous pouvez également créer des modèles d’application logique en utilisant [Azure PowerShell avec le module LogicAppTemplate](../logic-apps/logic-apps-create-azure-resource-manager-templates.md#azure-powershell).

L’exemple d’application logique présenté dans cette rubrique utilise un [déclencheur Outlook Office 365](/connectors/office365/) qui s’active lors de l’arrivée d’un nouvel e-mail, ainsi qu’une [action de Stockage Blob Azure](/connectors/azureblob/) qui crée un blob pour le corps de l’e-mail et charge ce blob dans un conteneur de stockage Azure. Les exemples montrent également comment paramétrer des valeurs qui varient lors du déploiement.

Pour plus d’informations sur les modèles Resource Manager, voir les rubriques suivantes :

* [Structure et syntaxe du modèle Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)
* [Meilleures pratiques relatives aux modèles Azure Resource Manager](../azure-resource-manager/templates/template-best-practices.md)
* [Développer des modèles Azure Resource Manager pour la cohérence du cloud](../azure-resource-manager/templates/templates-cloud-consistency.md)

Pour plus d’informations sur les ressources de modèle spécifiques pour les applications logiques, les comptes d’intégration, les artefacts de compte d’intégration et les environnements de service d’intégration, consultez [Types de ressources Microsoft.Logic](/azure/templates/microsoft.logic/allversions).

Pour des exemples de modèles d’applications logiques, voir :

* le [modèle complet](#full-example-template) utilisé pour les exemples de cette rubrique ;
* l’[exemple de modèle d’application logique de démarrage rapide](https://github.com/Azure/azure-quickstart-templates/blob/master/101-logic-app-create) dans GitHub.

Pour l’API REST Logic Apps, commencez par la [Vue d’ensemble de l’API REST Azure Logic Apps](/rest/api/logic).

<a name="template-structure"></a>

## <a name="template-structure"></a>Structure du modèle

Au niveau supérieur, un modèle Resource Manager suit cette structure entièrement décrite dans la rubrique [Structure et syntaxe du modèle Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md) :

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {},
   "variables": {},
   "functions": [],
   "resources": [],
   "outputs": {}
}
```

Pour un modèle d’application logique, vous travaillez principalement avec les objets de modèle suivants :

| Attribut | Description |
|-----------|-------------|
| `parameters` | Déclare les [ paramètres de modèle ](../azure-resource-manager/templates/template-syntax.md#parameters) pour l’acceptation des valeurs à utiliser lors de la création et de la personnalisation des ressources pour un déploiement vers Azure. Par exemple, ces paramètres acceptent les valeurs du nom et de l’emplacement de votre application logique, des connexions et des autres ressources nécessaires au déploiement. Vous pouvez stocker ces valeurs de paramètre dans un [fichier de paramètres](#template-parameter-files) décrit plus loin dans cette rubrique. Pour plus de détails, voir [Paramètres – Structure et syntaxe de modèle Resource Manager](../azure-resource-manager/templates/template-syntax.md#parameters). |
| `resources` | Définit les [ressources](../azure-resource-manager/templates/template-syntax.md#resources) à créer ou mettre à jour, ainsi qu’à déployer vers un groupe de ressources Azure, telles que votre application logique, vos connexions, vos comptes de stockage Azure, etc. Pour des détails généraux, voir [Ressources – Structure et syntaxe de modèle Resource Manager](../azure-resource-manager/templates/template-syntax.md#resources). |
||||

Votre modèle d’application logique utilise le format de nom de fichier suivant :

**<*nom-application-logique*>. Json**

> [!IMPORTANT]
> La syntaxe du modèle respectant la casse, veillez à utiliser une casse cohérente. 

<a name="template-parameters"></a>

## <a name="template-parameters"></a>Paramètres de modèle

Un modèle d’application logique comporte plusieurs objets `parameters` existant à différents niveaux et exécutant différentes fonctions. Par exemple, au niveau supérieur, vous pouvez déclarer des [paramètres de modèle](../azure-resource-manager/templates/template-syntax.md#parameters) pour les valeurs à accepter et à utiliser lors de la création et du déploiement de ressources dans Azure, par exemple :

* Votre application logique
* Connexions que votre logique utilise pour accéder à d’autres services et systèmes via des [Connecteurs managés](../connectors/apis-list.md)
* Autres ressources dont votre application logique a besoin pour le déploiement

  Par exemple, si votre application logique utilise un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) pour les scénarios interentreprises (B2B), l’objet `parameters` de niveau supérieur du modèle déclare le paramètre qui accepte l’ID de ressource pour ce compte d’intégration.

Voici la structure et la syntaxe générales d’une définition de paramètre, qui sont décrites en détail dans [Paramètres – Structure et syntaxe de modèle Resource Manager](../azure-resource-manager/templates/template-syntax.md#parameters) :

```json
"<parameter-name>": {
   "type": "<parameter-type>",
   "defaultValue": <default-parameter-value>,
   <other-parameter-attributes>,
   "metadata": {
      "description": "<parameter-description>"
   }
},
```

Cet exemple présente uniquement les paramètres de modèle applicables aux valeurs utilisées pour créer et déployer ces ressources dans Azure :

* Nom et emplacement de votre application logique
* ID à utiliser pour un compte d’intégration lié à l’application logique

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "LogicAppName": {
         "type": "string",
         "minLength": 1,
         "maxLength": 80,
         "defaultValue": "MyLogicApp",
         "metadata": {
            "description": "The resource name for the logic app"
         }
      },
      "LogicAppLocation": {
         "type": "string",
         "minLength": 1,
         "defaultValue": "[resourceGroup().location]",
         "metadata": {
            "description": "The resource location for the logic app"
         }
      },
      "LogicAppIntegrationAccount": {
         "type":"string",
         "minLength": 1,
         "defaultValue": "/subscriptions/<Azure-subscription-ID>/resourceGroups/fabrikam-integration-account-rg/providers/Microsoft.Logic/integrationAccounts/fabrikam-integration-account",
         "metadata": {
            "description": "The ID to use for the integration account"
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [],
   "outputs": {}
}
```

À l’exception des paramètres qui traitent des valeurs sensibles ou qui doivent être sécurisées, tels que des noms d’utilisateur, mots de passe ou secrets, tous ces paramètres incluent des attributs `defaultValue`, même si, dans certains cas, les valeurs par défaut sont vides. Les valeurs de déploiement à utiliser pour ces paramètres de modèle sont fournies par l’exemple de [fichier de paramètres](#template-parameter-files) décrit plus loin dans cette rubrique.

Pour plus d’informations sur la sécurisation des paramètres de modèle, consultez les rubriques suivantes :

* [Recommandations de sécurité pour les paramètres de modèle](../azure-resource-manager/templates/template-best-practices.md#parameters)
* [Améliorer la sécurité pour les paramètres de modèle](../logic-apps/logic-apps-securing-a-logic-app.md#secure-parameters-deployment-template)
* [Transmettre des valeurs de paramètre sécurisées avec Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md)

D’autres objets de modèle référencent souvent des paramètres de modèle afin de pouvoir utiliser les valeurs transmises par des paramètres de modèle, par exemple :

* L’[objet de ressources de votre modèle](#template-resources) décrit plus loin dans cette rubrique définit chaque ressource dans Azure, que vous pouvez créer et déployer, telle que la [définition de ressource de votre application logique](#logic-app-resource-definition). Ces ressources utilisent souvent des valeurs de paramètre de modèle, telles que le nom et l’emplacement de votre application logique, ainsi que des informations de connexion.

* À un niveau plus profond dans la définition de ressource de votre application logique, l’[objet de paramètres de votre définition de flux de travail](#workflow-definition-parameters) déclare des paramètres pour les valeurs à utiliser lors de l’exécution de votre application logique. Par exemple, vous pouvez déclarer des paramètres de définition de flux de travail pour le nom d’utilisateur et le mot de passe qu’un déclencheur HTTP utilise pour l’authentification. Pour spécifier les valeurs pour des paramètres de définition de flux de travail, utilisez l’objet `parameters` qui est *en dehors* de votre définition de flux de travail mais toujours *à l’intérieur* de la définition de ressource de votre application logique. Dans cet objet `parameters` externe, vous pouvez référencer des paramètres de modèle précédemment déclarés, qui peuvent accepter des valeurs lors du déploiement à partir d’un fichier de paramètres.

Lors du référencement des paramètres, les expressions et fonctions de modèle utilisent une syntaxe différente et se comportent différemment des expressions et fonctions de définition de flux de travail. Pour plus d’informations sur ces différences, voir [Références aux paramètres](#parameter-references) plus loin dans cette rubrique.

<a name="best-practices-template-parameters"></a>

## <a name="best-practices---template-parameters"></a>Meilleures pratiques – Paramètres de modèle

Voici quelques meilleures pratiques pour la définition de paramètres :

* Déclarez des paramètres uniquement pour des valeurs qui varient, en fonction de vos besoins de déploiement. Ne déclarez pas de paramètres pour des valeurs qui ne changent pas en fonction des différentes exigences de déploiement.

* Incluez l’attribut `defaultValue`, qui peut spécifier des valeurs vides, pour tous les paramètres, à l’exception des valeurs sensibles ou qui doivent être sécurisées. Utilisez toujours des paramètres sécurisés pour les noms d’utilisateur, mots de passe et secrets. Pour masquer ou protéger les valeurs de paramètres sensibles, suivez les instructions des rubriques suivantes :

  * [Recommandations de sécurité pour les paramètres de modèle](../azure-resource-manager/templates/template-best-practices.md#parameters)

  * [Améliorer la sécurité pour les paramètres de modèle](../logic-apps/logic-apps-securing-a-logic-app.md#secure-parameters-deployment-template)

  * [Transmettre des valeurs de paramètre sécurisées avec Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md)

* Pour différencier les noms de paramètre de modèle des noms de paramètre de définition de flux de travail, vous pouvez utiliser des noms de paramètre de modèle descriptifs, par exemple : `TemplateFabrikamPassword`

Pour d’autres meilleures pratiques en lien avec les modèles, voir [Meilleures pratiques pour les paramètres de modèle](../azure-resource-manager/templates/template-best-practices.md#parameters).

<a name="template-parameter-files"></a>

## <a name="template-parameters-file"></a>Fichier de paramètres de modèle

Pour fournir les valeurs des paramètres de modèle, stockez ces valeurs dans un [fichier de paramètres](../azure-resource-manager/templates/parameter-files.md). De cette façon, vous pouvez utiliser différents fichiers de paramètres en fonction de vos besoins de déploiement. Voici le format de nom de fichier à utiliser :

* Nom de fichier de modèle d’application logique : **<*nom-application-logique*>.json**
* Nom de fichier de paramètres : **<*nom-application-logique*>.parameters.json**

Voici la structure interne du fichier de paramètres, qui inclut une référence de coffre de clés pour [transmettre une valeur de paramètre sécurisée avec Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md) :

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
   "contentVersion": "1.0.0.0",
   // Template parameter values
   "parameters": {
      "<parameter-name-1>": {
         "value": "<parameter-value>"
      },
      "<parameter-name-2>": {
         "value": "<parameter-value>"
      },
      "<secured-parameter-name>": {
         "reference": {
            "keyVault": {
               "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/Microsoft.KeyVault/vaults/<key-vault-name>",
            },
            "secretName: "<secret-name>"
         }
      },
      <other-parameter-values>
   }
}
```

Cet exemple de fichier de paramètres spécifie les valeurs des paramètres de modèle déclarés précédemment dans cette rubrique :

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
   "contentVersion": "1.0.0.0",
   // Template parameter values
   "parameters": {
      "LogicAppName": {
         "value": "Email-Processor-Logic-App"
      },
      "LogicAppLocation": {
         "value": "westeurope"
      }
   }
}
```

<a name="template-resources"></a>

## <a name="template-resources"></a>Ressources de modèle

Votre modèle inclut un objet `resources` qui est un tableau contenant des définitions de chaque ressource à créer et à déployer dans Azure, telles que la [définition de ressource de votre application logique](#logic-app-resource-definition), les [définitions de ressources de connexion](#connection-resource-definitions), ainsi que toutes les autres ressources nécessaires au déploiement de votre application logique.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "variables": {},
   "functions": [],
   "resources": [
      {
         <logic-app-resource-definition>
      },
      // Start connection resource definitions
      {
         <connection-resource-definition-1>
      },
      {
         <connection-resource-definition-2>
      }
   ],
   "outputs": {}
}
```

> [!NOTE]
> Des modèles pouvant inclure des définitions de ressources pour plusieurs applications logiques, assurez-vous que toutes vos ressources d’application logique spécifient le même groupe de ressources Azure. Lorsque vous déployez le modèle sur un groupe de ressources Azure à l’aide de Visual Studio, vous êtes invité à indiquer l’application logique que vous voulez ouvrir. Par ailleurs, votre projet de groupe de ressources Azure pouvant contenir plusieurs modèles, veillez à sélectionner le fichier de paramètres approprié lorsque vous y êtes invité.

<a name="view-resource-definitions"></a>

### <a name="view-resource-definitions"></a>Afficher les définitions de ressources

Pour passer en revue les définitions de toutes les ressources d’un groupe de ressources Azure, [téléchargez votre application logique dans Visual Studio à partir d’Azure](../logic-apps/manage-logic-apps-with-visual-studio.md), qui est le moyen le plus simple de créer un modèle d’application logique paramétré valide et dans l’ensemble prêt à être déployé.

Pour des informations générales sur les ressources de modèle et leurs attributs, voir les rubriques suivantes :

* [Ressources – Structure et syntaxe de modèle Resource Manager](../azure-resource-manager/templates/template-syntax.md#resources)
* [Meilleures pratiques pour les ressources de modèle](../azure-resource-manager/templates/template-best-practices.md#resources)

<a name="logic-app-resource-definition"></a>

### <a name="logic-app-resource-definition"></a>Définition de ressource d’application logique

La [définition de ressource de flux de travail dans un modèle](/azure/templates/microsoft.logic/workflows) de votre application logique commence par l’objet `properties` qui contient les informations suivantes :

* État de votre application logique lors du déploiement
* ID de tout compte d’intégration utilisé par votre application logique
* Définition de flux de travail de votre application logique
* Objet `parameters` qui définit les valeurs à utiliser lors de l’exécution
* Autres informations de ressources concernant votre application logique, telles que le nom, le type, l’emplacement, les paramètres de configuration du runtime, etc.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "variables": {},
   "functions": [],
   "resources": [
      {
         // Start logic app resource definition
         "properties": {
            "state": "<Enabled or Disabled>",
            "integrationAccount": {
               "id": "[parameters('LogicAppIntegrationAccount')]" // Template parameter reference
            },
            "definition": {<workflow-definition>},
            "parameters": {<workflow-definition-parameter-values>},
            "accessControl": {},
            "runtimeConfiguration": {}
         },
         "name": "[parameters('LogicAppName')]", // Template parameter reference
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]", // Template parameter reference
         "tags": {
           "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "dependsOn": [
         ]
      }
      // End logic app resource definition
   ],
   "outputs": {}
}
```

Voici les attributs spécifiques de la définition de ressource de votre application logique :

| Attribut | Obligatoire | Type | Description |
|-----------|----------|------|-------------|
| `state` | Oui | String | État de votre application logique lors du déploiement, où `Enabled` signifie que votre application logique est active et `Disabled` signifie qu’elle est inactive. Par exemple, si vous n’êtes pas prêt à mettre en ligne votre application logique mais souhaitez déployer une version brouillon, vous pouvez utiliser l’option `Disabled`. |
| `integrationAccount` | Non | Object | Si votre application logique utilise un compte d’intégration qui stocke des artefacts pour des scénarios interentreprises (B2B), cet objet inclut l’attribut `id` qui spécifie l’ID du compte d’intégration. |
| `definition` | Oui | Object | Définition de flux de travail sous-jacent de votre application logique, qui est l’objet qui s’affiche en mode Code et qui est décrit en détail dans la rubrique [Référence de schéma pour le langage de définition de flux de travail](../logic-apps/logic-apps-workflow-definition-language.md). Dans cette définition de flux de travail, l’objet `parameters` déclare des paramètres pour les valeurs à utiliser lors de l’exécution de l’application logique. Pour plus d’informations, voir [Définition et paramètres de flux de travail](#workflow-definition-parameters). <p><p>Pour afficher les attributs dans la définition de flux de travail de votre application logique, passez du « mode Création » au « mode Code » dans le portail Azure ou Visual Studio, ou en vous servant d’un outil tel qu’[Azure Resource Explorer](https://resources.azure.com). |
| `parameters` | Non | Object | [Valeurs de paramètre de définition de flux de travail](#workflow-definition-parameters) à utiliser lors de l’exécution d’une application logique. Les définitions de paramètre de ces valeurs apparaissent dans l’[objet de paramètres de votre définition de flux de travail​​](#workflow-definition-parameters). De plus, si votre application logique utilise des [connecteurs managés](../connectors/apis-list.md) pour accéder à d’autres services et systèmes, cet objet inclut un objet `$connections` qui définit les valeurs de connexion à utiliser lors de l’exécution. |
| `accessControl` | Non | Object | Utilisé pour spécifier des attributs de sécurité pour votre application logique, tels qu’une restriction d’accès IP aux déclencheurs de demandes ou aux entrées et sorties de l’historique d’exécution. Pour plus d’informations, voir [Accès sécurisé aux applications logiques](../logic-apps/logic-apps-securing-a-logic-app.md). |
| `runtimeConfiguration` | Non | Object | Pour spécifier des propriétés `operationOptions` qui contrôlent le comportement de votre application logique au moment de l’exécution. Par exemple, vous pouvez exécuter votre application logique en [mode de débit élevé](../logic-apps/logic-apps-limits-and-config.md#run-high-throughput-mode). |
|||||

Pour plus d’informations sur les définitions de ressources pour ces objets Logic Apps, consultez [Types de ressources Microsoft.Logic](/azure/templates/microsoft.logic/allversions) :

* [Définition de ressource de flux de travail](/azure/templates/microsoft.logic/workflows)
* [Définition de ressource d’environnement de service d’intégration](/azure/templates/microsoft.logic/integrationserviceenvironments)
* [Définition de ressource d’API gérée d’environnement de service d’intégration](/azure/templates/microsoft.logic/integrationserviceenvironments/managedapis)

* [Définition de ressource de compte d’intégration](/azure/templates/microsoft.logic/integrationaccounts)

* Artefacts de compte d’intégration :

  * [Définition de ressource de contrat](/azure/templates/microsoft.logic/integrationaccounts/agreements)

  * [Définition de ressource d’assembly](/azure/templates/microsoft.logic/integrationaccounts/assemblies)

  * [Définition de ressource de configuration de lot](/azure/templates/microsoft.logic/integrationaccounts/batchconfigurations)

  * [Définition de ressource de certificat](/azure/templates/microsoft.logic/integrationaccounts/certificates)

  * [Définition de ressource de carte](/azure/templates/microsoft.logic/integrationaccounts/maps)

  * [Définition de ressource de partenaire](/azure/templates/microsoft.logic/integrationaccounts/partners)

  * [Définition de ressource de schéma](/azure/templates/microsoft.logic/integrationaccounts/schemas)

  * [Définition de ressource de session](/azure/templates/microsoft.logic/integrationaccounts/sessions)

<a name="workflow-definition-parameters"></a>

## <a name="workflow-definition-and-parameters"></a>Définition et paramètres de flux de travail

La définition de flux de travail de votre application logique apparaît dans l’objet `definition` qui apparaît dans l’objet `properties` dans la définition de ressource de votre application logique. Cet objet `definition` est celui qui s’affiche en mode Code et qui est décrit en détail dans la rubrique [Référence de schéma pour le langage de définition de flux de travail](../logic-apps/logic-apps-workflow-definition-language.md). Votre définition de flux de travail inclut un objet de déclaration `parameters` interne dans lequel vous pouvez définir de nouveaux paramètres ou modifier des paramètres existants pour les valeurs utilisées par votre définition de flux de travail au moment de l’exécution. Vous pouvez ensuite référencer ces paramètres dans le déclencheur ou des actions dans votre flux de travail. Par défaut, cet objet `parameters` est vide, à moins que votre application logique crée des connexions à d’autres services et systèmes via des [connecteurs managés](../connectors/apis-list.md).

Pour définir les valeurs pour des paramètres de définition de flux de travail, utilisez l’objet `parameters` qui est *en dehors* de votre définition de flux de travail mais toujours *à l’intérieur* de la définition de ressource de votre application logique. Dans cet objet `parameters` externe, vous pouvez ensuite référencer vos paramètres de modèle précédemment déclarés, qui peuvent accepter des valeurs lors du déploiement à partir d’un fichier de paramètres.

> [!TIP]
>
> En tant que meilleure pratique, ne référencez pas des paramètres de modèle qui sont évalués lors du déploiement, directement à partir de la définition de flux de travail. Au lieu de cela, déclarez un paramètre de définition de flux de travail que vous pouvez ensuite définir dans l’objet `parameters` qui est *en dehors* de votre définition de flux de travail mais toujours *à l’intérieur* de la définition de ressource de votre application logique. Pour plus d’informations, voir [Références aux paramètres](#parameter-references).

Cette syntaxe indique où vous pouvez déclarer des paramètres aux niveaux de définition de modèle et de flux de travail, ainsi que où vous pouvez définir ces valeurs de paramètre en référençant les paramètres de modèle et de définition de flux de travail :

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "<template-parameter-name>": {
         "type": "<parameter-type>",
         "defaultValue": "<parameter-default-value>",
         "metadata": {
            "description": "<parameter-description>"
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         // Start logic app resource definition
         "properties": {
            <other-logic-app-resource-properties>,
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {<action-definitions>},
               // Workflow definition parameters
               "parameters": {
                  "<workflow-definition-parameter-name>": {
                     "type": "<parameter-type>",
                     "defaultValue": "<parameter-default-value>",
                     "metadata": {
                        "description": "<parameter-description>"
                     }
                  }
               },
               "triggers": {
                  "<trigger-name>": {
                     "type": "<trigger-type>",
                     "inputs": {
                         // Workflow definition parameter reference
                         "<attribute-name>": "@parameters('<workflow-definition-parameter-name')"
                     }
                  }
               },
               <...>
            },
            // Workflow definition parameter value
            "parameters": {
               "<workflow-definition-parameter-name>": { 
                  "value": "[parameters('<template-parameter-name>')]"
               }
            },
            "accessControl": {}
         },
         <other-logic-app-resource-definition-attributes>
      }
      // End logic app resource definition
   ],
   "outputs": {}
}
```

<a name="secure-workflow-definition-parameters"></a>

### <a name="secure-workflow-definition-parameters"></a>Paramètres de définition du flux de travail sécurisé

Pour un paramètre de définition de flux de travail qui traite des informations sensibles, des mots de passe, des clés d’accès ou des secrets lors de l’exécution, déclarez ou modifiez le paramètre pour utiliser le type de paramètre `securestring` ou `secureobject`. Vous pouvez référencer ce paramètre dans votre définition de flux de travail. Au niveau supérieur du modèle, déclarez un paramètre du même type pour gérer ces informations lors du déploiement.

Pour définir la valeur pour le paramètre de définition de flux de travail, utilisez l’objet `parameters` qui est *en dehors* de votre définition de flux de travail mais toujours *à l’intérieur* de la définition de ressource de votre application logique pour référencer le paramètre de modèle. Enfin, pour transmettre la valeur à votre paramètre de modèle lors du déploiement, stockez-la dans [Vault Key Azure](../azure-resource-manager/templates/key-vault-parameter.md) et référencez ce coffre de clés dans le [fichier de paramètres](#template-parameter-files) utilisé par votre modèle lors du déploiement.

Cet exemple de modèle montre comment accomplir ces tâches en définissant des paramètres sécurisés lorsque cela est nécessaire pour pouvoir stocker leurs valeurs dans Azure Key Vault :

* Déclarez des paramètres sécurisés pour les valeurs utilisées pour authentifier l’accès.
* Utilisez ces valeurs aux niveaux de définition de modèle et de flux de travail.
* Fournissez ces valeurs à l’aide d’un fichier de paramètres.

**Modèle**

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      <previously-defined-template-parameters>,
      // Additional template parameters for passing values to use in workflow definition
      "TemplateAuthenticationType": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "The type of authentication used for the Fabrikam portal"
         }
      },
      "TemplateFabrikamPassword": {
         "type": "securestring",
         "metadata": {
            "description": "The password for the Fabrikam portal"
         }
      },
      "TemplateFabrikamUserName": {
         "type": "securestring",
         "metadata": {
            "description": "The username for the Fabrikam portal"
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         // Start logic app resource definition
         "properties": {
            <other-logic-app-resource-properties>,
            // Start workflow definition
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {<action-definitions>},
               // Workflow definition parameters
               "parameters": {
                  "authenticationType": {
                     "type": "string",
                     "defaultValue": "",
                     "metadata": {
                        "description": "The type of authentication used for the Fabrikam portal"
                     }
                  },
                  "fabrikamPassword": {
                     "type": "securestring",
                     "metadata": {
                        "description": "The password for the Fabrikam portal"
                     }
                  },
                  "fabrikamUserName": {
                     "type": "securestring",
                     "metadata": {
                        "description": "The username for the Fabrikam portal"
                     }
                  }
               },
               "triggers": {
                  "HTTP": {
                     "inputs": {
                        "authentication": {
                           // Reference workflow definition parameters
                           "password": "@parameters('fabrikamPassword')",
                           "type": "@parameters('authenticationType')",
                           "username": "@parameters('fabrikamUserName')"
                        }
                     },
                     "recurrence": {<...>},
                     "type": "Http"
                  }
               },
               <...>
            },
            // End workflow definition
            // Start workflow definition parameter values
            "parameters": {
               "authenticationType": "[parameters('TemplateAuthenticationType')]", // Template parameter reference
               "fabrikamPassword": "[parameters('TemplateFabrikamPassword')]", // Template parameter reference
               "fabrikamUserName": "[parameters('TemplateFabrikamUserName')]" // Template parameter reference
            },
            "accessControl": {}
         },
         <other-logic-app-resource-attributes>
      }
      // End logic app resource definition
   ],
   "outputs": {}
}
```

**Fichier de paramètres**

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
   "contentVersion": "1.0.0.0",
   // Template parameter values
   "parameters": {
      <previously-defined-template-parameter-values>,
     "TemplateAuthenticationType": {
        "value": "Basic"
     },
     "TemplateFabrikamPassword": {
        "reference": {
           "keyVault": {
              "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/Microsoft.KeyVault/vaults/fabrikam-key-vault"
           },
           "secretName": "FabrikamPassword"
        }
     },
     "TemplateFabrikamUserName": {
        "reference": {
           "keyVault": {
              "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/Microsoft.KeyVault/vaults/fabrikam-key-vault"
           },
           "secretName": "FabrikamUserName"
        }
     }
   }
}
```

<a name="best-practices-workflow-definition-parameters"></a>

## <a name="best-practices---workflow-definition-parameters"></a>Meilleures pratiques – Paramètres de définition de flux de travail

Pour vous assurer que le Concepteur d’application logique peut afficher correctement les paramètres de définition de flux de travail, suivez les meilleures pratiques suivantes :

* Incluez l’attribut `defaultValue`, qui peut spécifier des valeurs vides, pour tous les paramètres, à l’exception des valeurs sensibles ou qui doivent être sécurisées.

* Utilisez toujours des paramètres sécurisés pour les noms d’utilisateur, mots de passe et secrets. Pour masquer ou protéger les valeurs de paramètres sensibles, suivez les instructions des rubriques suivantes :

  * [Recommandations de sécurité pour les paramètres d’action](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters)

  * [Recommandations de sécurité pour les paramètres dans les définitions de flux de travail](../logic-apps/logic-apps-securing-a-logic-app.md#secure-parameters-workflow)

  * [Transmettre des valeurs de paramètre sécurisées avec Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md)

Pour plus d’informations sur les paramètres de définition de flux de travail, voir [Paramètres – Langage de définition du flux de travail](../logic-apps/logic-apps-workflow-definition-language.md#parameters).

<a name="connection-resource-definitions"></a>

## <a name="connection-resource-definitions"></a>Définitions de ressources de connexion

Lorsque votre application logique crée et utilise des connexions à d’autres services et système à l’aide de [connecteurs managés](../connectors/apis-list.md), l’objet `resources` de votre modèle contient les définitions de ressources pour ces connexions. Bien que vous ayez créé des connexions à partir d’une application logique, les connexions sont des ressources Azure distinctes avec leurs propres définitions de ressource. Pour passer en revue ces définitions de ressources de connexion, [téléchargez votre application logique dans Visual Studio à partir d’Azure](../logic-apps/manage-logic-apps-with-visual-studio.md), qui est le moyen le plus simple de créer un modèle d’application logique paramétré valide et dans l’ensemble prêt à être déployé.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "variables": {},
   "functions": [],
   "resources": [
      {
         <logic-app-resource-definition>
      },
      // Start connection resource definitions
      {
         <connection-resource-definition-1>
      },
      {
         <connection-resource-definition-2>
      }
   ],
   "outputs": {}
}
```

Les définitions de ressources de connexion font référence aux paramètres de niveau supérieur du modèle pour leurs valeurs, ce qui signifie que vous pouvez fournir ces valeurs lors du déploiement à l’aide d’un fichier de paramètres. Assurez-vous que les connexions utilisent les mêmes groupe de ressources et emplacement Azure que votre application logique.

Voici un exemple de définition de ressource pour une connexion Outlook Office 365 et les paramètres de modèle correspondants :

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "LogicAppName": {<parameter-definition>},
      "LogicAppLocation": {<parameter-definition>},
      "office365_1_Connection_Name": {
         "type": "string",
         "defaultValue": "office365",
         "metadata": {
            "description": "The resource name for the Office 365 Outlook connection"
         }
      },
      "office365_1_Connection_DisplayName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "The display name for the Office 365 Outlook connection"
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         <logic-app-resource-definition>
      },
      // Office 365 Outlook API connection resource definition
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         // Template parameter reference for connection name
         "name": "[parameters('office365_1_Connection_Name')]",
         // Template parameter reference for connection resource location. Must match logic app location.
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
               // Connector ID
               "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]"
            },
            // Template parameter reference for connection display name
            "displayName": "[parameters('office365_1_Connection_DisplayName')]"
         }
      }
   ],
   "outputs": {}
}
```

La définition de ressource de votre application logique fonctionne également avec des définitions de ressources de connexion des manières suivantes :

* Dans votre définition de flux de travail, l’objet `parameters` déclare un paramètre `$connections` pour les valeurs de connexion à utiliser lors de l’exécution de l’application logique. De plus, le déclencheur ou l’action qui créent une connexion utilisent les valeurs correspondantes que ce paramètre `$connections` transmet.

* *En dehors* de votre définition de flux de travail mais toujours *à l’intérieur* de la définition de ressource de votre application logique, un autre objet `parameters` définit les valeurs à utiliser lors de l’exécution pour le paramètre `$connections` en faisant référence aux paramètres de modèle correspondants. Ces valeurs utilisent des expressions de modèle pour référencer des ressources qui stockent en toute sécurité les métadonnées pour les connexions dans votre application logique.

  Par exemple, les métadonnées peuvent inclure des chaînes de connexion et jetons d’accès que vous pouvez stocker dans [Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md). Pour transmettre ces valeurs à vos paramètres de modèle, vous référencez ce coffre de clés dans le [fichier de paramètres](#template-parameter-files) utilisé par votre modèle lors du déploiement. Pour plus d’informations sur les différences de référencement de paramètres, voir [Références aux paramètres](#parameter-references) plus loin dans cette rubrique.

  Lorsque vous ouvrez la définition de flux de travail de votre application logique en mode Code dans le portail Azure ou Visual Studio, l’objet `$connections` apparaît en dehors de votre définition de flux de travail, mais au même niveau. Cette commande en mode Code facilite le référencement de ces paramètres lorsque vous mettez à jour manuellement la définition de flux de travail :

  ```json
  {
     "$connections": {<workflow-definition-parameter-connection-values-runtime},
     "definition": {<workflow-definition>}
  }
  ```

* La définition de ressource de votre application logique contient un objet `dependsOn` qui spécifie les dépendances des connexions utilisées par votre application logique.

Chaque connexion que vous créez a un nom unique dans Azure. Lorsque vous créez plusieurs connexions au même service ou système, un nombre est ajouté à chaque nom de connexion, qui est incrémenté à chaque nouvelle connexion créée, par exemple, `office365`, `office365-1`, etc.

Cet exemple montre les interactions entre la définition de ressource de votre application logique et une définition de ressource de connexion pour Office 365 Outlook :

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "LogicAppName": {<parameter-definition>},
      "LogicAppLocation": {<parameter-definition>},
      "office365_1_Connection_Name": {<parameter-definition>},
      "office365_1_Connection_DisplayName": {<parameter-definition>}
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         // Start logic app resource definition
         "properties": {
            <...>,
            "definition": {
               <...>,
               "parameters": {
                  // Workflow definition "$connections" parameter
                  "$connections": {
                     "defaultValue": {},
                     "type": "Object"
                  }
               },
               <...>
            },
            "parameters": {
               // Workflow definition "$connections" parameter values to use at runtime
               "$connections": {
                  "value": {
                     "office365": {
                        // Template parameter references
                        "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]",
                        "connectionId": "[resourceId('Microsoft.Web/connections', parameters('office365_1_Connection_Name'))]",
                        "connectionName": "[parameters('office365_1_Connection_Name')]"
                     }
                  }
               }
            },
            <other-logic-app-resource-information>,
            "dependsOn": [
               "[resourceId('Microsoft.Web/connections', parameters('office365_1_Connection_Name'))]"
            ]
         }
         // End logic app resource definition
      },
      // Office 365 Outlook API connection resource definition
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         // Template parameter reference for connection name
         "name": "[parameters('office365_1_Connection_Name')]",
         // Template parameter reference for connection resource location. Must match logic app location.
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
               // Connector ID
               "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]"
            },
            // Template parameter reference for connection display name
            "displayName": "[parameters('office365_1_Connection_DisplayName')]"
         }
      }
   ],
   "outputs": {}
}
```

<a name="secure-connection-parameters"></a>

### <a name="secure-connection-parameters"></a>Paramètres de connexion sécurisée

Pour un paramètre de connexion qui traite des informations sensibles, des mots de passe, des clés d’accès ou des secrets, la définition de ressource de la connexion inclut un objet `parameterValues` spécifiant ces valeurs dans un format de paire nom-valeur. Pour masquer ces informations, vous pouvez déclarer ou modifier les paramètres de modèle pour ces valeurs en utilisant les types de paramètres `securestring` ou `secureobject`. Vous pouvez ensuite stocker ces informations dans [ Azure Key Vault ](../azure-resource-manager/templates/key-vault-parameter.md). Pour transmettre ces valeurs à vos paramètres de modèle, vous référencez ce coffre de clés dans le [fichier de paramètres](#template-parameter-files) utilisé par votre modèle lors du déploiement.

Voici un exemple fournissant le nom du compte et la clé d’accès pour une connexion de Stockage Blob Azure :

**Fichier de paramètres**

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
   "contentVersion": "1.0.0.0",
   // Template parameter values
   "parameters": {
      "LogicAppName": {
         "value": "Email-Processor-Logic-App"
      },
      "LogicAppLocation": {
         "value": "westeurope"
      },
      "azureblob_1_Connection_Name": {
         "value": "Fabrikam-Azure-Blob-Storage-Connection"
      },
      "azureblob_1_Connection_DisplayName": {
         "value": "Fabrikam-Storage"
      },
      "azureblob_1_accountName": {
         "value": "Fabrikam-Storage-Account"
      },
      "azureblob_1_accessKey": {
         "reference": {
            "keyVault": {
               "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/Microsoft.KeyVault/vaults/fabrikam-key-vault"
            },
            "secretName": "FabrikamStorageKey"
         }
      }
   }
}
```

**Modèle**

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "LogicAppName": {<parameter-definition>},
      "LogicAppLocation": {<parameter-definition>},
      "azureblob_1_Connection_Name": {<parameter-definition>},
      "azureblob_1_Connection_DisplayName": {<parameter-definition>},
      "azureblob_1_accountName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "Name of the storage account the connector should use."
         }
      },
      "azureblob_1_accessKey": {
         "type": "secureobject",
         "metadata": {
            "description": "Specify a valid primary/secondary storage account access key."
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         "properties": {
            "state": "Disabled",
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {
                  // Azure Blob Storage action
                  "Create_blob": {
                     "type": "ApiConnection",
                     "inputs": {
                        "host": {
                           "connection": {
                              // Workflow definition parameter reference for values to use at runtime
                              "name": "@parameters('$connections')['azureblob']['connectionId']"
                           }
                        },
                     },
                     "method": "post",
                     "body": "@triggerBody()?['Body']",
                     "path": "/datasets/default/files",
                     "queries": {
                        "folderPath": "/emails",
                        "name": "@triggerBody()?['Subject']",
                        "queryParametersSingleEncoded": true
                     },
                     "runAfter": {},
                     "runtimeConfiguration": {
                        "contentTransfer": {
                           "transferMode": "Chunked"
                        }
                     }
                  }
               },
               "parameters": {
                  // Workflow definition parameter for values to use at runtime
                  "$connections": {
                     "defaultValue": {},
                     "type": "Object"
                  }
               },
               "triggers": {<trigger-definition>},
               "contentVersion": "1.0.0.0",
               "outputs": {}
            },
            "parameters": {
               "$connections": {
                  "value": {
                     // Template parameter references for values to use at runtime
                     "azureblob": {
                        "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'azureblob')]",
                        "connectionId": "[resourceId('Microsoft.Web/connections', parameters('azureblob_1_Connection_Name'))]",
                        "connectionName": "[parameters('azureblob_1_Connection_Name')]"
                    }
                  }
               }
            },
            "name": "[parameters('LogicAppName')]",
            "type": "Microsoft.Logic/workflows",
            "location": "[parameters('LogicAppLocation')]",
            "tags": {
               "displayName": "LogicApp"
            },
            "apiVersion": "2016-06-01",
            // Template parameter reference for value to use at deployment
            "dependsOn": [
               "[resourceId('Microsoft.Web/connections', parameters('azureblob_1_Connection_Name'))]"
            ]
         }
      },
      // Azure Blob Storage API connection resource definition
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         "name": "[parameters('azureblob_1_Connection_Name')]",
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
               "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'azureblob')]"
            },
            "displayName": "[parameters('azureblob_1_Connection_DisplayName')]",
            // Template parameter reference for values to use at deployment
            "parameterValues": {
               "accountName": "[parameters('azureblob_1_accountName')]",
               "accessKey": "[parameters('azureblob_1_accessKey')]"
            }
         }
      }
   ],
   "outputs": {}
}
```

<a name="authenticate-connections"></a>

### <a name="authenticate-connections"></a>Authentifiez les connexions

Après le déploiement, votre application logique fonctionne de bout en bout avec des paramètres valides. Toutefois, vous devez encore autoriser les connexions OAuth à générer des jetons d’accès valides pour l’[authentification de vos informations d’identification](../active-directory/develop/authentication-vs-authorization.md). Pour plus d’informations, voir [Autoriser des connexions OAuth](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md#authorize-oauth-connections).

Certaines connexions prennent en charge l’utilisation d’un [principal de service](../active-directory/develop/app-objects-and-service-principals.md) Azure Active Directory (Azure AD) pour autoriser les connexions d’une application logique [inscrite dans Azure AD](../active-directory/develop/quickstart-register-app.md). Par exemple, cette définition de ressource de connexion Azure Data Lake montre comment référencer les paramètres de modèle qui gèrent les informations du principal de service, et comment le modèle déclare ces paramètres :

**Définition de ressource de connexion**

```json
{
   <other-template-objects>
   "type": "MICROSOFT.WEB/CONNECTIONS",
   "apiVersion": "2016-06-01",
   "name": "[parameters('azuredatalake_1_Connection_Name')]",
   "location": "[parameters('LogicAppLocation')]",
   "properties": {
      "api": {
         "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', 'resourceGroup().location', '/managedApis/', 'azuredatalake')]"
      },
      "displayName": "[parameters('azuredatalake_1_Connection_DisplayName')]",
      "parameterValues": {
         "token:clientId": "[parameters('azuredatalake_1_token:clientId')]",
         "token:clientSecret": "[parameters('azuredatalake_1_token:clientSecret')]",
         "token:TenantId": "[parameters('azuredatalake_1_token:TenantId')]",
         "token:grantType": "[parameters('azuredatalake_1_token:grantType')]"
      }
   }
}
```

| Attribut | Description |
|-----------|-------------|
| `token:clientId` | Application ou ID client associés à votre principal de service |
| `token:clientSecret` | Valeur de clé associée à votre principal de service |
| `token:TenantId` | ID de répertoire de votre locataire Azure AD |
| `token:grantType` | Type d’autorisation demandée, qui doit être `client_credentials`. Pour plus d’informations, voir [Plateforme d’identités Microsoft et flux d’informations d’identification du client OAuth 2.0](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md). |
|||

**Définitions de paramètre de modèle**

L’objet `parameters` de niveau supérieur du modèle déclare ces paramètres pour l’exemple de connexion :

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      "azuredatalake_1_Connection_Name": {
        "type": "string",
        "defaultValue": "azuredatalake"
      },
      "azuredatalake_1_Connection_DisplayName": {
        "type": "string",
        "defaultValue": "<connection-name>"
      },
      "azuredatalake_1_token:clientId": {
        "type": "securestring",
        "metadata": {
          "description": "Client (or Application) ID of the Azure Active Directory application."
        }
      },
      "azuredatalake_1_token:clientSecret": {
        "type": "securestring",
        "metadata": {
          "description": "Client secret of the Azure Active Directory application."
        }
      },
      "azuredatalake_1_token:TenantId": {
        "type": "securestring",
        "metadata": {
          "description": "The tenant ID of for the Azure Active Directory application."
        }
      },
      "azuredatalake_1_token:resourceUri": {
        "type": "string",
        "metadata": {
          "description": "The resource you are requesting authorization to use."
        }
      },
      "azuredatalake_1_token:grantType": {
        "type": "string",
        "metadata": {
          "description": "Grant type"
        },
        "defaultValue": "client_credentials",
        "allowedValues": [
          "client_credentials"
        ]
      },
      // Other template parameters
   }
   // Other template objects
}
```

Pour plus d’informations sur l’utilisation des principaux de service, voir les rubriques suivantes :

* [Création d’un principal de service à l’aide du portail Azure](../active-directory/develop/howto-create-service-principal-portal.md)
* [Créer un principal de service Azure à l’aide d’Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps)
* [Créer un principal de service avec un certificat à l’aide d’Azure PowerShell](../active-directory/develop/howto-authenticate-service-principal-powershell.md)

<a name="parameter-references"></a>

## <a name="references-to-parameters"></a>Références aux paramètres

Pour référencer des paramètres de modèle, vous pouvez utiliser des expressions de modèle avec des [fonctions de modèle](../azure-resource-manager/templates/template-functions.md) qui sont évaluées lors du déploiement. Les expressions de modèle utilisent des crochets (**[]**) :

`"<attribute-name>": "[parameters('<template-parameter-name>')]"`

Pour référencer des paramètres de définition de flux de travail, vous utilisez des [expressions et fonctions du Langage de définition du flux de travail](../logic-apps/workflow-definition-language-functions-reference.md), qui sont évaluées au moment de l’exécution. Vous remarquerez peut-être que certaines fonctions de modèle et fonctions de définition de flux de travail portent le même nom. Les expressions de définition de flux de travail commencent par le symbole d’arobase ( **@** ) :

`"<attribute-name>": "@parameters('<workflow-definition-parameter-name>')"`

Vous pouvez transmettre des valeurs de paramètre de modèle à votre définition de flux de travail pour que votre application logique puisse les utiliser au moment de l’exécution. Cependant, évitez d’utiliser des paramètres, des expressions et une syntaxe de modèle dans la définition de votre flux de travail, car le Concepteur d’application logique ne prend pas en charge les éléments de modèle. Par ailleurs, une syntaxe et des expressions de modèle peuvent compliquer votre code en raison de différences entre les moments où les expressions sont évaluées.

Suivez plutôt ces étapes générales pour déclarer et référencer les paramètres de définition de flux de travail à utiliser lors de l’exécution, déclarer et référencer les paramètres de modèle à utiliser lors du déploiement, et spécifier les valeurs à transmettre lors du déploiement à l’aide d’un fichier de paramètres. Pour plus de détails, voir la section [Définition et paramètres de flux de travail](#workflow-definition-parameters) plus haut dans cette rubrique.

1. Créez votre modèle et déclarez les paramètres de modèle pour les valeurs à accepter et à utiliser lors du déploiement.

1. Dans votre définition de flux de travail, déclarez les paramètres des valeurs à accepter et à utiliser lors de l’exécution. Vous pouvez ensuite référencer ces valeurs dans votre définition de flux de travail.

1. Dans l’objet `parameters` qui est *en dehors de* votre définition de flux de travail mais toujours *à l’intérieur de* la définition de ressource de votre application logique, définissez les valeurs des paramètres de définition de votre flux de travail en référençant les paramètres de modèle correspondants. De cette façon, vous pouvez transmettre les valeurs de paramètre de modèle dans les paramètres de définition de votre flux de travail.

1. Dans le fichier de paramètres, spécifiez les valeurs de votre modèle à utiliser lors du déploiement.

<a name="full-example-template"></a>

## <a name="full-example-template"></a>Exemple de modèle complet

Voici l’exemple de modèle paramétré utilisé dans les exemples de cette rubrique :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
   "parameters": {
      "LogicAppName": {
         "type": "string",
         "minLength": 1,
         "maxLength": 80,
         "defaultValue": "MyLogicApp",
         "metadata": {
            "description": "The resource name to use for the logic app"
         }
      },
      "LogicAppLocation": {
         "type": "string",
         "minLength": 1,
         "defaultValue": "[resourceGroup().location]",
         "metadata": {
            "description": "The resource location to use for the logic app"
         }
      },
      "office365_1_Connection_Name": {
         "type": "string",
         "defaultValue": "office365",
         "metadata": {
            "description": "The resource name to use for the Office 365 Outlook connection"
         }
      },
      "office365_1_Connection_DisplayName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "The display name to use for the Office 365 Outlook connection"
         }
      },
      "azureblob_1_Connection_Name": {
         "type": "string",
         "defaultValue": "azureblob",
         "metadata": {
            "description": "The resource name to use for the Azure Blob storage account connection"
         }
      },
      "azureblob_1_Connection_DisplayName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "Name of the storage account the connector should use."
         },

      },
      "azureblob_1_accountName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "Name of the storage account the connector should use."
         }
      },
      "azureblob_1_accessKey": {
         "type": "securestring",
         "metadata": {
            "description": "Specify a valid primary/secondary storage account access key."
         }
      },
      "LogicAppIntegrationAccount": {
         "type":"string",
         "minLength": 1,
         "defaultValue": "/subscriptions/<Azure-subscription-ID>/resourceGroups/fabrikam-integration-account-rg/providers/Microsoft.Logic/integrationAccounts/fabrikam-integration-account",
         "metadata": {
            "description": "The ID to use for the integration account"
         }
      }
   },
   "variables": {},
   "resources": [
      {
         "properties": {
            "state": "Disabled",
            "integrationAccount": {
              "id": "[parameters('LogicAppIntegrationAccount')]"
            },
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {
                  "Create_blob": {
                     "type": "ApiConnection",
                     "inputs": {
                        "host": {
                           "connection": {
                              "name": "@parameters('$connections')['azureblob']['connectionId']"
                           }
                        },
                     },
                     "method": "post",
                     "body": "@triggerBody()?['Body']",
                     "path": "/datasets/default/files",
                     "queries": {
                        "folderPath": "/emails",
                        "name": "@triggerBody()?['Subject']",
                        "queryParametersSingleEncoded": true
                     },
                     "runAfter": {},
                     "runtimeConfiguration": {
                        "contentTransfer": {
                           "transferMode": "Chunked"
                        }
                     }
                  }
               },
               "parameters": {
                  "$connections": {
                     "defaultValue": {},
                     "type": "Object"
                  }
               },
               "triggers": {
                  "When_a_new_email_arrives": {
                     "type": "ApiConnection",
                     "inputs": {
                        "host": {
                           "connection": {
                              "name": "@parameters('$connections')['office365']['connectionId']"
                           }
                        },
                        "method": "get",
                        "path": "/Mail/OnNewEmail",
                        "queries": {
                           "folderPath": "Inbox",
                           "importance": "Any",
                           "fetchOnlyWithAttachment": false,
                           "includeAttachments": false
                        }
                     },
                     "recurrence": {
                        "frequency": "Day",
                        "interval": 1
                     },
                     "splitOn": "@triggerBody()?['value']"
                  }
               },
               "contentVersion": "1.0.0.0",
               "outputs": {}
            },
            "parameters": {
               "$connections": {
                  "value": {
                     "azureblob": {
                        "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'azureblob')]",
                        "connectionId": "[resourceId('Microsoft.Web/connections', parameters('azureblob_1_Connection_Name'))]",
                        "connectionName": "[parameters('azureblob_1_Connection_Name')]"
                     },
                     "office365": {
                        "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]",
                        "connectionId": "[resourceId('Microsoft.Web/connections', parameters('office365_1_Connection_Name'))]",
                        "connectionName": "[parameters('office365_1_Connection_Name')]"
                     }
                  }
               }
            },
            "accessControl": {}
         },
         "name": "[parameters('LogicAppName')]",
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]",
         "tags": {
            "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "dependsOn": [
            "[resourceId('Microsoft.Web/connections', parameters('azureblob_1_Connection_Name'))]",
            "[resourceId('Microsoft.Web/connections', parameters('office365_1_Connection_Name'))]"
         ]
      },
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         "name": "[parameters('office365_1_Connection_Name')]",
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
                "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]"
            },
            "displayName": "[parameters('office365_1_Connection_DisplayName')]"
         }
      },
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         "name": "[parameters('azureblob_1_Connection_Name')]",
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
               "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'azureblob')]"
            },
            "displayName": "[parameters('azureblob_1_Connection_DisplayName')]",
            "parameterValues": {
               "accountName": "[parameters('azureblob_1_accountName')]",
               "accessKey": "[parameters('azureblob_1_accessKey')]"
            }
         }
      }
   ],
   "outputs": {}
}
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer des modèles d’application logique](../logic-apps/logic-apps-create-azure-resource-manager-templates.md)
