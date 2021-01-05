---
title: Exemple de script Azure CLI – Créer une application logique
description: Exemple de script permettant de créer une application logique via l’extension Logic Apps dans l’interface Azure CLI.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.custom: mvc, devx-track-azurecli
ms.date: 07/30/2020
ms.openlocfilehash: e66edb1325d1c603e89f877f1d34f60c136eb1db
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92740730"
---
# <a name="azure-cli-script-sample---create-a-logic-app"></a>Exemple de script Azure CLI – Créer une application logique

Ce script crée un exemple d’application logique via l’[extension Logic Apps de l’interface Azure CLI](/cli/azure/ext/logic/logic?view=azure-cli-latest), (`az logic`). Pour obtenir un guide détaillé de la création et de la gestion d’applications logiques via l’interface Azure CLI, consultez le [Guide de démarrage rapide Logic Apps pour l’interface Azure CLI](quickstart-logic-apps-azure-cli.md).

> [!WARNING]
> L’extension Logic Apps d’Azure CLI, actuellement *expérimentale* , *ne bénéficie pas d’un support technique* . Utilisez-la CLI avec précaution, en particulier si vous choisissez de l’utiliser dans des environnements de production.

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. Si vous n’avez pas d’abonnement Azure, [créez un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest) installé sur votre ordinateur local.
* [Extension Logic Apps d’Azure CLI](/cli/azure/azure-cli-extensions-list?view=azure-cli-latest) installée sur votre ordinateur. Pour installer cette extension, utilisez la commande suivante : `az extension add --name logic`
* Une [définition de workflow](quickstart-logic-apps-azure-cli.md#workflow-definition) pour votre application logique. Ce fichier JSON doit suivre le [schéma de langage de définition de workflow](logic-apps-workflow-definition-language.md).
* Une connexion d’API à un compte de messagerie via un [connecteur Logic Apps](../connectors/apis-list.md) pris en charge dans le même groupe de ressources que votre application logique. Cet exemple utilise le connecteur [Office 365 Outlook](../connectors/connectors-create-api-office365-outlook.md), mais vous pouvez également utiliser d’autres connecteurs, comme [Outlook.com](../connectors/connectors-create-api-outlook.md).

### <a name="prerequisite-check"></a>Vérification du prérequis

Validez votre environnement avant de commencer :

* Connectez-vous au portail Azure et vérifiez que votre abonnement est actif en exécutant la commande `az login`.

* Vérifiez votre version d’Azure CLI dans un terminal ou une fenêtre de commande en exécutant `az --version`. Pour obtenir la version la plus récente, consultez les [notes de publication les plus récentes](/cli/azure/release-notes-azure-cli?tabs=azure-cli&view=azure-cli-latest).

  * Si vous ne disposez pas de la dernière version, mettez à jour votre installation en suivant le [guide d’installation pour votre système d’exploitation ou votre plateforme](/cli/azure/install-azure-cli?view=azure-cli-latest).

### <a name="sample-workflow-explanation"></a>Exemple d’explication de workflow

Cet exemple de fichier de définition de workflow crée la même application logique de base que le [guide de démarrage rapide Logic Apps pour le portail Azure](quickstart-create-first-logic-app-workflow.md). 

Cet exemple de workflow : 

1. Spécifie un schéma, `$schema`, pour l’application logique.

1. Définit un déclencheur pour l’application logique dans la liste des déclencheurs, `triggers`. Le déclencheur se répète (`recurrence`) toutes les 3 heures. Les actions sont déclenchées lorsqu’un nouvel élément de flux est publié (`When_a_feed_item_is_published`) pour le flux RSS spécifié (`feedUrl`).

1. Définit une action pour l’application logique dans la liste des actions, `actions`. L’action envoie un e-mail (`Send_an_email_(V2)`) via Microsoft 365 avec les détails des éléments de flux RSS, tels qu’ils sont spécifiés dans la section du corps (`body`) des entrées de l’action (`inputs`).

## <a name="sample-workflow-definition"></a>Exemple de définition de workflow

Avant d’exécuter l’exemple de script, vous devez d’abord créer un exemple de [définition de workflow](#prerequisites).

1. Créez un fichier JSON, `testDefinition.json`, sur votre ordinateur. 

1. Copiez le contenu suivant dans le fichier JSON : 
    ```json
    
    {
        "definition": {
            "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
            "actions": {
                "Send_an_email_(V2)": {
                    "inputs": {
                        "body": {
                            "Body": "<p>@{triggerBody()?['publishDate']}<br>\n@{triggerBody()?['title']}<br>\n@{triggerBody()?['primaryLink']}</p>",
                            "Subject": "@triggerBody()?['title']",
                            "To": "test@example.com"
                        },
                        "host": {
                            "connection": {
                                "name": "@parameters('$connections')['office365']['connectionId']"
                            }
                        },
                        "method": "post",
                        "path": "/v2/Mail"
                    },
                    "runAfter": {},
                    "type": "ApiConnection"
                }
            },
            "contentVersion": "1.0.0.0",
            "outputs": {},
            "parameters": {
                "$connections": {
                    "defaultValue": {},
                    "type": "Object"
                }
            },
            "triggers": {
                "When_a_feed_item_is_published": {
                    "inputs": {
                        "host": {
                            "connection": {
                                "name": "@parameters('$connections')['rss']['connectionId']"
                            }
                        },
                        "method": "get",
                        "path": "/OnNewFeed",
                        "queries": {
                            "feedUrl": "https://www.pbs.org/now/rss.xml"
                        }
                    },
                    "recurrence": {
                        "frequency": "Hour",
                        "interval": 3
                    },
                    "splitOn": "@triggerBody()?['value']",
                    "type": "ApiConnection"
                }
            }
        },
        "parameters": {
            "$connections": {
                "value": {
                    "office365": {
                        "connectionId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testResourceGroup/providers/Microsoft.Web/connections/office365",
                        "connectionName": "office365",
                        "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Web/locations/westus/managedApis/office365"
                    },
                    "rss": {
                        "connectionId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testResourceGroup/providers/Microsoft.Web/connections/rss",
                        "connectionName": "rss",
                        "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Web/locations/westus/managedApis/rss"
                    }
                }
            }
        }
    }
    
    ```

1. Mettez à jour les valeurs d’espace réservé avec vos propres informations :

    1. Remplacez l’adresse e-mail de l’espace réservé (`"To": "test@example.com"`). Vous devez utiliser une adresse e-mail compatible avec les connecteurs Logic Apps. Pour plus d’informations, voir les [Conditions préalables](#prerequisites).

    1. Remplacez des détails supplémentaires du connecteur si vous utilisez un connecteur de messagerie autre que le connecteur Office 365 Outlook.

    1. Remplacez les valeurs d’abonnement de l’espace réservé (`00000000-0000-0000-0000-000000000000`) pour vos identificateurs de connexion (`connectionId` et `id`) sous le paramètre de connexion (`$connections`) par vos propres valeurs d’abonnement.

1. Enregistrez vos modifications.

## <a name="sample-script"></a>Exemple de script

> [!NOTE]
> Cet exemple a été écrit pour l’interpréteur de commandes `bash`. Si vous souhaitez exécuter cet exemple dans un autre interpréteur de commandes, tel que Windows PowerShell ou l’invite de commandes, vous devrez peut-être apporter des modifications à votre script.

Avant d’exécuter cet exemple de script, exécutez la commande suivante pour vous connecter à Azure :

```azurecli-interactive

az login

```

Ensuite, accédez au répertoire dans lequel vous avez créé votre définition de workflow. Par exemple, si vous avez créé le fichier JSON de définition de workflow sur votre Bureau :

```azurecli

cd ~/Desktop

```

Exécutez alors ce script pour créer une application logique. 

```azurecli-interactive

#!/bin/bash

# Create a resource group

az group create --name testResourceGroup --location westus

# Create your logic app

az logic workflow create --resource-group "testResourceGroup" --location "westus" --name "testLogicApp" --definition "testDefinition.json"

```

### <a name="clean-up-deployment"></a>Nettoyer le déploiement

Une fois que vous avez fini d’utiliser l’exemple de script, exécutez la commande suivante pour supprimer votre groupe de ressources et toutes ses ressources imbriquées, y compris l’application logique.

```azurecli-interactive

az group delete --name testResourceGroup --yes

```

## <a name="script-explanation"></a>Explication du script

Cet exemple de script utilise les commandes suivantes pour créer un groupe de ressources et une application logique.

| Commande | Notes |
| ------- | ----- |
| [`az group create`](/cli/azure/group?view=azure-cli-latest#az-group-create) | Crée un groupe de ressources dans lequel les ressources de votre application logique sont stockées. |
| [`az logic workflow create`](/cli/azure/ext/logic/logic/workflow?view=azure-cli-latest#ext-logic-az-logic-workflow-create) | Crée une application logique basée sur le workflow défini dans le paramètre `--definition`. |
| [`az group delete`](/cli/azure/vm/extension?view=azure-cli-latest) | Supprime un groupe de ressources et toutes ses ressources imbriquées. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure/?view=azure-cli-latest).

Vous trouverez des exemples supplémentaires de scripts CLI Logic Apps dans le [navigateur d’exemples de code de Microsoft](/samples/browse/?products=azure-logic-apps).
