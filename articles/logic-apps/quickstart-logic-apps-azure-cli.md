---
title: Créer et gérer des applications logiques avec Azure CLI
description: Utilisez Azure CLI pour créer une application logique, puis gérez votre application logique à l’aide de commandes telles que list, show (get), update et delete.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm
ms.topic: quickstart
ms.custom: mvc, devx-track-azurecli, contperf-fy21q2
ms.date: 11/23/2020
ms.openlocfilehash: bc172fd1702addf8f4e34094452a779b09320a4d
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033372"
---
# <a name="quickstart-create-and-manage-logic-apps-using-the-azure-cli"></a>Démarrage rapide : Créer et gérer des applications logiques à l’aide d’Azure CLI

Ce guide de démarrage rapide montre comment créer et gérer des applications logiques à l’aide de l’extension [Logic Apps d’Azure CLI](/cli/azure/ext/logic/logic) (`az logic`). À partir de la ligne de commande, vous pouvez créer une application logique à l’aide du fichier JSON pour une définition de flux de travail d’application logique. Vous pouvez ensuite gérer votre application logique en exécutant des commandes telles que `list`, `show` (`get`), `update` et `delete` à partir de la ligne de commande.

> [!WARNING]
> L’extension Logic Apps d’Azure CLI, actuellement *expérimentale*, *ne bénéficie pas d’un support technique*. Utilisez-la CLI avec précaution, en particulier si vous choisissez de l’utiliser dans des environnements de production.

Si vous débutez avec Logic Apps, vous pouvez aussi apprendre à créer vos premières applications logiques [par le biais du portail Azure](quickstart-create-first-logic-app-workflow.md), [dans Visual Studio](quickstart-create-logic-apps-with-visual-studio.md) et [dans Visual Studio Code](quickstart-create-logic-apps-visual-studio-code.md).

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. Si vous n’avez pas d’abonnement Azure, [créez un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Azure CLI](/cli/azure/install-azure-cli) installé sur votre ordinateur local.
* [Extension Logic Apps d’Azure CLI](/cli/azure/azure-cli-extensions-list) installée sur votre ordinateur. Pour installer cette extension, utilisez la commande suivante : `az extension add --name logic`
* [Groupe de ressources Azure](#example---create-resource-group) dans lequel créer votre application logique.

### <a name="prerequisite-check"></a>Vérification du prérequis

Validez votre environnement avant de commencer :

* Connectez-vous au portail Azure et vérifiez que votre abonnement est actif en exécutant la commande `az login`.
* Vérifiez votre version d’Azure CLI dans un terminal ou une fenêtre de commande en exécutant `az --version`. Pour obtenir la version la plus récente, consultez les [notes de publication les plus récentes](/cli/azure/release-notes-azure-cli?tabs=azure-cli).
  * Si vous ne disposez pas de la dernière version, mettez à jour votre installation en suivant le [guide d’installation pour votre système d’exploitation ou votre plateforme](/cli/azure/install-azure-cli).

### <a name="example---create-resource-group"></a>Exemple – Créer un groupe de ressources

Si ne disposez pas encore d’un groupe de ressources pour votre application logique, créez-en un avec la commande `az group create`. Par exemple, la commande suivante crée un groupe de ressources nommé `testResourceGroup` à l’emplacement `westus`.

```azurecli-interactive

az group create --name testResourceGroup --location westus

```

Une fois votre groupe de ressources créé, la sortie affiche l’état `provisioningState` `Succeeded` :

```output

<...>
  "name": "testResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
<...>

```

## <a name="workflow-definition"></a>Définition de flux de travail

Avant de [créer une application logique](#create-logic-apps-from-cli) ou de [mettre à jour une application logique existante](#update-logic-apps-from-cli) à l’aide d’Azure CLI, vous avez besoin d’une définition de flux de travail pour votre application logique. Dans le portail Azure, vous pouvez afficher la définition de flux de travail sous-jacente de votre application logique au format JSON en basculant du **Mode Création** en **Mode Code**.

Lorsque vous exécutez les commandes pour créer ou mettre à jour votre application logique, votre définition de flux de travail est chargée en tant que paramètre obligatoire (`--definition`). Vous devez créer votre définition de flux de travail sous la forme d’un fichier JSON qui suit le [schéma de langage de définition de flux de travail](./logic-apps-workflow-definition-language.md).

## <a name="create-logic-apps-from-cli"></a>Créer des applications logiques à partir de CLI

Vous pouvez créer un flux de travail d’application logique à partir d’Azure CLI à l’aide de la commande [`az logic workflow create`](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create) avec un fichier JSON pour la définition.

```azurecli

az logic workflow create --definition
                         --location
                         --name
                         --resource-group
                         [--access-control]
                         [--endpoints-configuration]
                         [--integration-account]
                         [--integration-service-environment]
                         [--state {Completed, Deleted, Disabled, Enabled, NotSpecified, Suspended}]
                         [--tags]

```

Votre commande doit inclure les [paramètres obligatoires](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create-required-parameters) suivants :

| Paramètre | Valeur | Description |
| --------- | ----- | ----------- |
| Définition de flux de travail | `--definition` | Fichier JSON avec la [définition de flux de travail](#workflow-definition) de votre application logique. |
| Emplacement | `--location -l` | Région Azure dans laquelle se trouve votre application logique. |
| Nom | `--name -n` | Nom de votre application logique. Le nom peut contenir uniquement des lettres, des chiffres, des traits d’union (`-`), des traits de soulignement (`_`), des parenthèses (`()`), et des points (`.`). Le nom doit également être unique dans les régions. |
| Nom de groupe ressources | `--resource-group -g` | [Groupe de ressources Azure](../azure-resource-manager/management/overview.md) dans lequel vous souhaitez créer votre application logique. [Créez un groupe de ressources](#example---create-resource-group) avant de commencer si vous n’en avez pas déjà un pour votre application logique. |

Vous pouvez également inclure des [paramètres facultatifs](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create-optional-parameters) supplémentaires pour configurer les contrôles d’accès, les points de terminaison, le compte d’intégration, l’environnement du service d’intégration, l’état et les balises de ressource de votre application logique.

### <a name="example---create-logic-app"></a>Exemple – Créer une application logique

Dans cet exemple, un flux de travail nommé `testLogicApp` est créé dans le groupe de ressources `testResourceGroup` à l’emplacement `westus`. Le fichier JSON `testDefinition.json` contient la définition de flux de travail.

```azurecli-interactive

az logic workflow create --resource-group "testResourceGroup" --location "westus" --name "testLogicApp" --definition "testDefinition.json"

```

Une fois votre flux de travail correctement créé, l’interface CLI affiche le code JSON de votre nouvelle définition de flux de travail. Si la création de votre flux de travail échoue, consultez la [liste des erreurs possibles](#errors).

## <a name="update-logic-apps-from-cli"></a>Mettre à jour des applications logiques à partir de l’interface CLI

Vous pouvez également mettre à jour le flux de travail d’une application logique à partir d’Azure CLI à l’aide de la commande [`az logic workflow create`](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create).

Votre commande doit inclure les mêmes [les paramètres requis](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create-required-parameters) que lorsque vous [créez une application logique](#create-logic-apps-from-cli). Vous pouvez également ajouter les mêmes [paramètres facultatifs](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create-optional-parameters) que lors de la création d’une application logique.

```azurecli

az logic workflow create --definition
                         --location
                         --name
                         --resource-group
                         [--access-control]
                         [--endpoints-configuration]
                         [--integration-account]
                         [--integration-service-environment]
                         [--state {Completed, Deleted, Disabled, Enabled, NotSpecified, Suspended}]
                         [--tags]

```

### <a name="example---update-logic-app"></a>Exemple – Mettre à jour une application logique

Dans cet exemple, l’[exemple de flux de travail créé dans la section précédente](#example---create-logic-app) est mis à jour pour utiliser un autre fichier de définition JSON, `newTestDefinition.json`, et ajouter deux balises de ressource, `testTag1` et `testTag2`, avec des valeurs de description.

```azurecli-interactive

az logic workflow create --resource-group "testResourceGroup" --location "westus" --name "testLogicApp" --definition "newTestDefinition.json" --tags "testTag1=testTagValue1" "testTag2=testTagValue"

```

Une fois votre flux de travail mis à jour, l’interface CLI affiche la définition de flux de travail mise à jour de votre application logique. Si votre mise à jour échoue, consultez la [liste des erreurs possibles](#errors).

## <a name="delete-logic-apps-from-cli"></a>Supprimer des applications logiques de l’interface CLI

Vous pouvez supprimer le flux de travail d’une application logique d’Azure CLI à l’aide de la commande [`az logic workflow delete`](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-delete).

Votre commande doit inclure les [paramètres requis](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-delete-required-parameters) suivants :

| Paramètre | Valeur | Description |
| --------- | ----- | ----------- |
| Nom | `--name -n` | Nom de votre application logique. |
| Nom de groupe ressources | `-resource-group -g` | Groupe de ressources dans lequel se trouve votre application logique. |

Vous pouvez également inclure un [paramètre facultatif](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-delete-optional-parameters), `--yes -y`, pour ignorer les invites de confirmation.

```azurecli

az logic workflow delete --name
                         --resource-group
                         [--yes]

```

L’interface CLI vous invite ensuite à confirmer la suppression de votre application logique. Vous pouvez ignorer l’invite de confirmation en utilisant le paramètre facultatif `--yes -y` avec votre commande.

```azurecli

Are you sure you want to perform this operation? (y/n):

```

Vous pouvez confirmer la suppression d’une application logique en [répertoriant vos applications logiques dans l’interface CLI](#list-logic-apps-in-cli) ou en affichant vos applications logiques dans le portail Azure.

### <a name="example---delete-logic-app"></a>Exemple – Supprimer une application logique

Dans cet exemple, l’[exemple de flux de travail créé dans une section précédente](#example---create-logic-app) est supprimé.

```azurecli-interactive

az logic workflow delete --resource-group "testResourceGroup" --name "testLogicApp"

```

Une fois que vous avez répondu à l’invite de confirmation en tapant `y`, l’application logique est supprimée.

## <a name="show-logic-apps-in-cli"></a>Afficher des applications logiques dans l’interface CLI

Vous pouvez obtenir un flux de travail d’application logique spécifique à l’aide de la commande [`az logic workflow show`](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-show).

```azurecli

az logic workflow show --name
                       --resource-group

```

Votre commande doit inclure les [paramètres obligatoires](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-show-required-parameters) suivants.

| Paramètre | Valeur | Description |
| --------- | ----- | ----------- |
| Nom | `--name -n` | Nom de votre application logique. |
| Nom de groupe ressources | `--resource-group -g` | Nom du groupe de ressources dans lequel se trouve votre application logique. |

### <a name="example---get-logic-app"></a>Exemple – Obtenir une application logique

Dans cet exemple, l’application logique `testLogicApp` dans le groupe de ressources `testResourceGroup` est retournée avec des journaux complets pour le débogage.

```azurecli-interactive

az logic workflow show --resource-group "testResourceGroup" --name "testLogicApp" --debug

```

## <a name="list-logic-apps-in-cli"></a>Répertorier les applications logiques dans l’interface CLI

Vous pouvez répertorier vos applications logiques par abonnement à l’aide de la commande [`az logic workflow list`](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-list). Cette commande retourne le code JSON pour les flux de travail de vos applications logiques.

Vous pouvez filtrer vos résultats à l’aide des [paramètres facultatifs](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-list-optional-parameters) suivants :

| Paramètre | Valeur | Description |
| --------- | ----- | ----------- |
| Nom de groupe ressources | `--resource-group -g` | Nom du groupe de ressources sur lequel vous souhaitez filtrer les résultats. |
| Nombre d’éléments | `--top` | Nombre d’éléments inclus dans vos résultats. |
| Filtrer | `--filter` | Type de filtre que vous utilisez dans votre liste. Vous pouvez filtrer par état (`State`), déclencheur (`Trigger`) et identificateur de la ressource référencée (`ReferencedResourceId`). |

```azurecli

az logic workflow list [--filter]
                       [--resource-group]
                       [--top]

```

### <a name="example---list-logic-apps"></a>Exemple – Répertorier les applications logiques

Dans cet exemple, tous les flux de travail activés dans le groupe de ressources `testResourceGroup` sont retournés sous la forme d’une table ASCII.

```azurecli-interactive

az logic workflow list --resource-group "testResourceGroup" --filter "(State eq 'Enabled')" --output "table"

```

## <a name="errors"></a>Erreurs

L’erreur suivante indique que l’extension CLI d’Azure Logic Apps n’est pas installée. Suivez les étapes des conditions préalables pour [installer l’extension Logic Apps](#prerequisites) sur votre ordinateur.

```output

az: 'logic' is not in the 'az' command group. See 'az --help'. If the command is from an extension, please make sure the corresponding extension is installed. To learn more about extensions, please visit https://docs.microsoft.com/cli/azure/azure-cli-extensions-overview

```

L’erreur suivante peut indiquer que le chemin d’accès du fichier pour le chargement de la définition de votre flux de travail est incorrect.

```output

Expecting value: line 1 column 1 (char 0)

```

## <a name="global-parameters"></a>Paramètres globaux

Vous pouvez utiliser les paramètres Azure CLI globaux facultatifs suivants avec vos commandes `az logic` :

| Paramètre | Valeur | Description |
| --------- | ----- | ----------- |
| Format de sortie | `--output -o` | Modifiez le [format de sortie](/cli/azure/format-output-azure-cli) à partir du JSON par défaut. |
| Afficher uniquement les erreurs | `--only-show-errors` | Supprimer les avertissements et n’afficher que les erreurs. |
| Commentaires | `--verbose` | Afficher des journaux d’activité détaillés. |
| Débogage | `--debug` | Afficher tous les journaux de débogage. |
| Message d’aide | `--help -h` | Afficher la boîte de dialogue d’aide. |
| Requête | `--query` | Définir une chaîne de requête JMESPath pour la sortie JSON. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure/).

Vous trouverez des exemples supplémentaires de scripts CLI Logic Apps dans le [navigateur d’exemples de code de Microsoft](/samples/browse/?products=azure-logic-apps).

Ensuite, vous pouvez créer un exemple d’application logique à l’aide d’Azure CLI en utilisant un exemple de script et de définition de workflow.

> [!div class="nextstepaction"]
> [Créer une application logique à l’aide d’un exemple de script](sample-logic-apps-cli-script.md).
