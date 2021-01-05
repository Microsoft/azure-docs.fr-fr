---
title: Activer Update Management à l’aide d’un modèle Azure Resource Manager | Microsoft Docs
description: Cet article explique comment utiliser un modèle Resource Manager pour activer Update Management.
ms.service: automation
ms.subservice: update-management
ms.topic: conceptual
author: mgoedtel
ms.author: magoedte
ms.date: 09/18/2020
ms.openlocfilehash: 88dd82ee7e004a3613d6d9892b0e853406f4caf4
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97696150"
---
# <a name="enable-update-management-using-azure-resource-manager-template"></a>Activer Update Management à l’aide d’un modèle Azure Resource Manager

Vous pouvez utiliser un [modèle Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md) pour activer la fonctionnalité Update Management d’Azure Automation dans votre groupe de ressources. Cet article fournit un exemple de modèle qui automatise les éléments suivants :

* Automatise la création d’un espace de travail Azure Monitor Log Analytics.
* Automatise la création d’un compte Azure Automation.
* Lie le compte Automation à l’espace de travail Log Analytics.
* Ajoute des exemples de runbooks Automation au compte.
* Active la fonctionnalité Gestion des mises à jour.

Le modèle n’automatise pas l’activation d’Update Management sur une ou plusieurs machines virtuelles Azure ou non-Azure.

Si vous disposez déjà d’un espace de travail Log Analytics et d’un compte Automation déployés dans une région prise en charge dans votre abonnement, ils ne sont pas liés. Ce modèle permet de créer le lien et de déployer Update Management.

>[!NOTE]
>La création du compte d’identification Automation n’est pas prise en charge quand vous utilisez un modèle ARM. Pour créer manuellement un compte d’identification à partir du portail ou avec PowerShell, consultez [Gérer les comptes d’identification](../manage-runas-account.md).

Une fois ces étapes accomplies, vous devez [configurer des paramètres de diagnostic](../automation-manage-send-joblogs-log-analytics.md) pour votre compte Automation, afin d’envoyer l’état de la tâche du Runbook et les flux de tâches à l’espace de travail Log Analytics lié.

## <a name="api-versions"></a>Versions d’API

La table suivante répertorie la version d’API pour les ressources utilisées dans cet exemple.

| Ressource | Type de ressource | Version de l'API |
|:---|:---|:---|
| [Espace de travail](/azure/templates/microsoft.operationalinsights/workspaces) | workspaces | 2020-03-01-preview |
| [Compte Automation](/azure/templates/microsoft.automation/automationaccounts) | automation | 2020-01-13-preview |
| [Services liés à l’espace de travail](/azure/templates/microsoft.operationalinsights/workspaces/linkedservices) | workspaces | 2020-03-01-preview |
| [Solutions](/azure/templates/microsoft.operationsmanagement/solutions) | solutions | 2015-11-01-preview |

## <a name="before-using-the-template"></a>Avant d’utiliser le modèle

Le modèle JSON est configuré de manière à vous demander les éléments suivants :

* Nom de l’espace de travail.
* Région dans laquelle créer l’espace de travail.
* Nom du compte Automation.
* Région dans laquelle créer le compte Automation.

Les paramètres suivants dans le modèle sont définis avec une valeur par défaut pour l’espace de travail Log Analytics :

* La valeur par défaut de *sku* est le niveau de tarification par Go publié dans le modèle de tarification d’avril 2018.
* La valeur par défaut de *dataRetention* est 30 jours.

>[!WARNING]
>Si vous voulez créer ou configurer un espace de travail Log Analytics dans un abonnement pour lequel le modèle de tarification d’avril 2018 a été choisi, le seul niveau tarifaire Log Analytics valide est *PerGB2018*.
>

Le modèle JSON spécifie une valeur par défaut pour les autres paramètres susceptibles d’être utilisés comme configuration standard dans votre environnement. Vous pouvez stocker le modèle dans un compte de stockage Azure pour mettre en place un accès partagé dans votre organisation. Pour plus d’informations sur l’utilisation des modèles, consultez [Déployer des ressources à l’aide de modèles ARM et d’Azure CLI](../../azure-resource-manager/templates/deploy-cli.md).

Si vous ne connaissez pas encore Azure Automation et Azure vous devez bien comprendre les détails de configuration suivants. Ils peuvent vous aider à éviter des erreurs quand vous essayez de créer, de configurer et d’utiliser un espace de travail Log Analytics lié à votre nouveau compte Automation.

* Examinez les [détails supplémentaires](../../azure-monitor/samples/resource-manager-workspace.md#create-a-log-analytics-workspace) pour comprendre pleinement les options de configuration de l’espace de travail, telles que le mode de contrôle d’accès, le niveau tarifaire, la conservation des données et le niveau de réservation de capacité.

* Passez en revue les [mappages d’espace de travail](../how-to/region-mappings.md) pour spécifier les régions prises en charge incluses ou figurant dans un fichier de paramètres. Seules certaines régions sont prises en charge pour la liaison d’un espace de travail Log Analytics et d’un compte Automation dans votre abonnement.

* Si vous ne connaissez pas les journaux Azure Monitor et n’avez pas encore déployé un espace de travail, vous devez consulter les [conseils pour la conception de l’espace de travail](../../azure-monitor/platform/design-logs-deployment.md). Ils vous aideront à vous familiariser avec le contrôle d’accès et à comprendre les stratégies d’implémentation de conception que nous recommandons pour votre organisation.

## <a name="deploy-template"></a>Déployer un modèle

1. Copiez et collez la syntaxe JSON suivante dans votre fichier :

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "workspaceName": {
                "type": "string",
                "metadata": {
                    "description": "Workspace name"
                }
            },
            "sku": {
                "type": "string",
                "allowedValues": [
                    "pergb2018",
                    "Free",
                    "Standalone",
                    "PerNode",
                    "Standard",
                    "Premium"
                ],
                "defaultValue": "pergb2018",
                "metadata": {
                    "description": "Pricing tier: perGB2018 or legacy tiers (Free, Standalone, PerNode, Standard or Premium), which are not available to all customers."
                }
            },
            "dataRetention": {
                "type": "int",
                "defaultValue": 30,
                "minValue": 7,
                "maxValue": 730,
                "metadata": {
                    "description": "Number of days to retain data."
                }
            },
            "location": {
                "type": "string",
                "defaultValue": "[resourceGroup().location]",
                "metadata": {
                    "description": "Specifies the location in which to create the workspace."
                }
            },
            "automationAccountName": {
                "type": "string",
                "metadata": {
                    "description": "Automation account name"
                }
            },
            "automationAccountLocation": {
                "type": "string",
                "metadata": {
                    "description": "Specifies the location in which to create the Automation account."
                }
            },
            "sampleGraphicalRunbookName": {
                "type": "String",
                "defaultValue": "AzureAutomationTutorial"
            },
            "sampleGraphicalRunbookDescription": {
                "type": "String",
                "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
            },
            "samplePowerShellRunbookName": {
                "type": "String",
                "defaultValue": "AzureAutomationTutorialScript"
            },
            "samplePowerShellRunbookDescription": {
                "type": "String",
                "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
            },
            "samplePython2RunbookName": {
                "type": "String",
                "defaultValue": "AzureAutomationTutorialPython2"
            },
            "samplePython2RunbookDescription": {
                "type": "String",
                "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
            },
            "_artifactsLocation": {
                "type": "string",
                "defaultValue": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-automation/",
                "metadata": {
                    "description": "URI to artifacts location"
                }
            },
            "_artifactsLocationSasToken": {
                "type": "securestring",
                "defaultValue": "",
                "metadata": {
                    "description": "The sasToken required to access _artifactsLocation.  When the template is deployed using the accompanying scripts, a sasToken will be automatically generated"
                }
            }
        },
        "variables": {
        "Updates": {
            "name": "[concat('Updates', '(', parameters('workspaceName'), ')')]",
            "galleryName": "Updates"
          }
        },
        "resources": [
            {
                "type": "Microsoft.OperationalInsights/workspaces",
                "apiVersion": "2020-03-01-preview",
                "name": "[parameters('workspaceName')]",
                "location": "[parameters('location')]",
                "properties": {
                    "sku": {
                        "name": "[parameters('sku')]"
                    },
                    "retentionInDays": "[parameters('dataRetention')]",
                    "features": {
                        "searchVersion": 1,
                        "legacy": 0
                    }
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "location": "[parameters('location')]",
                "name": "[variables('Updates').name]",
                "type": "Microsoft.OperationsManagement/solutions",
                "id": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.OperationsManagement/solutions/', variables('Updates').name)]",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
                ],
                "properties": {
                    "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
                },
                "plan": {
                    "name": "[variables('Updates').name]",
                    "publisher": "Microsoft",
                    "promotionCode": "",
                    "product": "[concat('OMSGallery/', variables('Updates').galleryName)]"
                }
            },
            {
                "type": "Microsoft.Automation/automationAccounts",
                "apiVersion": "2020-01-13-preview",
                "name": "[parameters('automationAccountName')]",
                "location": "[parameters('automationAccountLocation')]",
                "dependsOn": [
                    "[parameters('workspaceName')]"
                ],
                "properties": {
                    "sku": {
                        "name": "Basic"
                    }
                },
                "resources": [
                    {
                        "type": "runbooks",
                        "apiVersion": "2018-06-30",
                        "name": "[parameters('sampleGraphicalRunbookName')]",
                        "location": "[parameters('automationAccountLocation')]",
                        "dependsOn": [
                            "[parameters('automationAccountName')]"
                        ],
                        "properties": {
                            "runbookType": "GraphPowerShell",
                            "logProgress": "false",
                            "logVerbose": "false",
                            "description": "[parameters('sampleGraphicalRunbookDescription')]",
                            "publishContentLink": {
                                "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorial.graphrunbook', parameters('_artifactsLocationSasToken')))]",
                                "version": "1.0.0.0"
                            }
                        }
                    },
                    {
                        "type": "runbooks",
                        "apiVersion": "2018-06-30",
                        "name": "[parameters('samplePowerShellRunbookName')]",
                        "location": "[parameters('automationAccountLocation')]",
                        "dependsOn": [
                            "[parameters('automationAccountName')]"
                        ],
                        "properties": {
                            "runbookType": "PowerShell",
                            "logProgress": "false",
                            "logVerbose": "false",
                            "description": "[parameters('samplePowerShellRunbookDescription')]",
                            "publishContentLink": {
                                "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorial.ps1', parameters('_artifactsLocationSasToken')))]",
                                "version": "1.0.0.0"
                            }
                        }
                    },
                    {
                        "type": "runbooks",
                        "apiVersion": "2018-06-30",
                        "name": "[parameters('samplePython2RunbookName')]",
                        "location": "[parameters('automationAccountLocation')]",
                        "dependsOn": [
                            "[parameters('automationAccountName')]"
                        ],
                        "properties": {
                            "runbookType": "Python2",
                            "logProgress": "false",
                            "logVerbose": "false",
                            "description": "[parameters('samplePython2RunbookDescription')]",
                            "publishContentLink": {
                                "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorialPython2.py', parameters('_artifactsLocationSasToken')))]",
                                "version": "1.0.0.0"
                            }
                        }
                    }
                ]
            },
            {
                "type": "Microsoft.OperationalInsights/workspaces/linkedServices",
                "apiVersion": "2020-03-01-preview",
                "name": "[concat(parameters('workspaceName'), '/' , 'Automation')]",
                "location": "[parameters('location')]",
                "dependsOn": [
                    "[parameters('workspaceName')]",
                    "[parameters('automationAccountName')]"
                ],
                "properties": {
                    "resourceId": "[resourceId('Microsoft.Automation/automationAccounts', parameters('automationAccountName'))]"
                }
            }
        ]
    }
    ```

2. Modifiez le modèle en fonction de vos besoins. Envisagez de créer un [fichier de paramètres Resource Manager](../../azure-resource-manager/templates/parameter-files.md) au lieu de passer des paramètres en tant que valeurs inline.

3. Enregistrez ce fichier dans un dossier local sous **deployUMSolutiontemplate.json**.

4. Vous êtes prêt à déployer ce modèle. Vous pouvez utiliser PowerShell ou l’interface de ligne de commande Azure. Lorsque vous êtes invité à entrer un nom d’espace de travail et de compte Automation, fournissez un nom globalement unique dans tous les abonnements Azure.

    **PowerShell**

    ```powershell
    New-AzResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile deployUMSolutiontemplate.json
    ```

    **Azure CLI**

    ```cli
    az deployment group create --resource-group <my-resource-group> --name <my-deployment-name> --template-file deployUMSolutiontemplate.json
    ```

    Le déploiement peut prendre plusieurs minutes. Lorsqu’il est terminé, vous voyez un message similaire au message suivant qui inclut le résultat :

    ![Exemple de résultat lorsque le déploiement est achevé](media/enable-from-template/template-output.png)

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Dans le portail Azure, ouvrez le compte Automation que vous avez créé.

3. Dans le volet gauche, sélectionnez **Runbooks**. Dans la page **Runbooks**, trois runbooks de tutoriel créés avec le compte Automation sont répertoriés.

    ![Runbooks de tutoriel créés avec le compte Automation](../media/quickstart-create-automation-account-template/automation-sample-runbooks.png)

4. Dans le volet gauche, sélectionnez **Espace de travail lié**. La page **Espace de travail lié** affiche l’espace de travail Log Analytics que vous avez spécifié précédemment, lié à votre compte Automation.

    ![Compte Automation lié à l’espace de travail Log Analytics](../media/quickstart-create-automation-account-template/automation-account-linked-workspace.png)

5. Dans le volet gauche, sélectionnez **Gestion des mises à jour**. Dans la page **Gestion des mises à jour**, cela affiche la page d’évaluation sans aucune information à la suite de son activation, et les machines ne sont pas configurées pour la gestion.

    ![Vue d’évaluation des fonctionnalités de Gestion des mises à jour](./media/enable-from-template/update-management-assessment-view.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez la solution **Mises à jour** dans l’espace de travail Log Analytics, supprimez la liaison du compte Automation de l’espace de travail, puis supprimez le compte Automation et l’espace de travail.

## <a name="next-steps"></a>Étapes suivantes

* Pour utiliser Update Management pour des machines virtuelles, consultez [Gérer les mises à jour et les correctifs pour vos machines virtuelles](manage-updates-for-vm.md).

* Si vous ne souhaitez plus utiliser Update Management et que vous voulez le supprimer, consultez les instructions dans [Supprimer Update Management](remove-feature.md).

* Pour supprimer des machines virtuelles d’Update Management, consultez [Supprimer des machines virtuelles d’Update Management](remove-vms.md).