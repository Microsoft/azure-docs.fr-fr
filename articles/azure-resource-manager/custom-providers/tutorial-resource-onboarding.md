---
title: Tutoriel - Intégration des ressources
description: L’intégration de ressources via des fournisseurs personnalisés vous permet de manipuler et d’étendre des ressources Azure existantes.
ms.topic: tutorial
ms.author: jobreen
author: jjbfour
ms.date: 09/17/2019
ms.openlocfilehash: 22d1dcd997a4ddb94aba184c5dace4c00509054d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75648606"
---
# <a name="tutorial-resource-onboarding-with-azure-custom-providers"></a>Tutoriel : Intégration de ressources avec des fournisseurs personnalisés Azure

Ce didacticiel explique comment déployer dans Azure un fournisseur de ressources personnalisé qui étend l’API Azure Resource Manager avec le type de ressource Microsoft. CustomProviders/associations. Ce didacticiel montre comment étendre des ressources existantes qui se trouvent en dehors du groupe de ressources où se trouve l’instance du fournisseur personnalisé. Dans ce didacticiel, le fournisseur de ressources personnalisé est alimenté par une application logique Azure, mais vous pouvez utiliser n’importe quel point de terminaison d’API public.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous devez savoir :

* Les capacités des [fournisseurs personnalisés Azure](overview.md).
* Les informations de base sur l’[intégration de ressources avec des fournisseurs personnalisés](concepts-resource-onboarding.md).

## <a name="get-started-with-resource-onboarding"></a>Prise en main de l’intégration de ressources

Dans ce didacticiel, deux éléments doivent être déployés : le fournisseur personnalisé et l’association. Pour faciliter le processus, vous pouvez utiliser un modèle unique qui déploie les deux.

Le modèle utilisera les ressources suivantes :

* Microsoft.CustomProviders/resourceProviders
* Microsoft.Logic/workflows
* Microsoft.CustomProviders/associations

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "allowedValues": [
                "australiaeast",
                "eastus",
                "westeurope"
            ],
            "metadata": {
                "description": "Location for the resources."
            }
        },
        "logicAppName": {
            "type": "string",
            "defaultValue": "[uniqueString(resourceGroup().id)]",
            "metadata": {
                "description": "Name of the logic app to be created."
            }
        },
        "customResourceProviderName": {
            "type": "string",
            "defaultValue": "[uniqueString(resourceGroup().id)]",
            "metadata": {
                "description": "Name of the custom provider to be created."
            }
        },
        "customResourceProviderId": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The resource ID of an existing custom provider. Provide this to skip deployment of new logic app and custom provider."
            }
        },
        "associationName": {
            "type": "string",
            "defaultValue": "myAssociationResource",
            "metadata": {
                "description": "Name of the custom resource that is being created."
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2017-05-10",
            "condition": "[empty(parameters('customResourceProviderId'))]",
            "name": "customProviderInfrastructureTemplate",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {
                        "logicAppName": {
                            "type": "string",
                            "defaultValue": "[parameters('logicAppName')]"
                        }
                    },
                    "resources": [
                        {
                            "type": "Microsoft.Logic/workflows",
                            "apiVersion": "2017-07-01",
                            "name": "[parameters('logicAppName')]",
                            "location": "[parameters('location')]",
                            "properties": {
                                "state": "Enabled",
                                "definition": {
                                    "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
                                    "actions": {
                                        "Switch": {
                                            "cases": {
                                                "Case": {
                                                    "actions": {
                                                        "CreateCustomResource": {
                                                            "inputs": {
                                                                "body": {
                                                                    "properties": "@addProperty(triggerBody().Body['properties'], 'myDynamicProperty', 'myDynamicValue')"
                                                                },
                                                                "statusCode": 200
                                                            },
                                                            "kind": "Http",
                                                            "type": "Response"
                                                        }
                                                    },
                                                    "case": "CREATE"
                                                }
                                            },
                                            "default": {
                                                "actions": {
                                                    "DefaultHttpResponse": {
                                                        "inputs": {
                                                            "statusCode": 200
                                                        },
                                                        "kind": "Http",
                                                        "type": "Response"
                                                    }
                                                }
                                            },
                                            "expression": "@triggerBody().operationType",
                                            "type": "Switch"
                                        }
                                    },
                                    "contentVersion": "1.0.0.0",
                                    "outputs": {},
                                    "parameters": {},
                                    "triggers": {
                                        "CustomProviderWebhook": {
                                            "inputs": {
                                                "schema": {}
                                            },
                                            "kind": "Http",
                                            "type": "Request"
                                        }
                                    }
                                }
                            }
                        },
                        {
                            "type": "Microsoft.CustomProviders/resourceProviders",
                            "apiVersion": "2018-09-01-preview",
                            "name": "[parameters('customResourceProviderName')]",
                            "location": "[parameters('location')]",
                            "properties": {
                                "resourceTypes": [
                                    {
                                        "name": "associations",
                                        "mode": "Secure",
                                        "routingType": "Webhook,Cache,Extension",
                                        "endpoint": "[[listCallbackURL(concat(resourceId('Microsoft.Logic/workflows', parameters('logicAppName')), '/triggers/CustomProviderWebhook'), '2017-07-01').value]"
                                    }
                                ]
                            }
                        }
                    ],
                    "outputs": {
                        "customProviderResourceId": {
                            "type": "string",
                            "value": "[resourceId('Microsoft.CustomProviders/resourceProviders', parameters('customResourceProviderName'))]"
                        }
                    }
                }
            }
        },
        {
            "type": "Microsoft.CustomProviders/associations",
            "apiVersion": "2018-09-01-preview",
            "name": "[parameters('associationName')]",
            "location": "global",
            "properties": {
                "targetResourceId": "[if(empty(parameters('customResourceProviderId')), reference('customProviderInfrastructureTemplate').outputs.customProviderResourceId.value, parameters('customResourceProviderId'))]",
                "myCustomInputProperty": "myCustomInputValue",
                "myCustomInputObject": {
                    "Property1": "Value1"
                }
            }
        }
    ],
    "outputs": {
        "associationResource": {
            "type": "object",
            "value": "[reference(parameters('associationName'), '2018-09-01-preview', 'Full')]"
        }
    }
}
```

### <a name="deploy-the-custom-provider-infrastructure"></a>Déployer l’infrastructure du fournisseur personnalisé

La première partie du modèle déploie l’infrastructure du fournisseur personnalisé. Cette infrastructure définit l’effet de la ressource associations. Si vous n’êtes pas familiarisé avec les fournisseurs personnalisés, voir [Présentation des fournisseurs de ressources personnalisés Azure](overview.md).

Déployons l’infrastructure du fournisseur personnalisé. Copiez, enregistrez et déployez le modèle précédent, ou suivez et la procédure de déploiement via le portail Azure.

1. Accédez au [portail Azure](https://portal.azure.com).

2. Recherchez **Modèles** dans **Tous les services** ou en utilisant la zone de recherche principale :

   ![Rechercher des modèles](media/tutorial-resource-onboarding/templates.png)

3. Dans le volet **Modèles**, sélectionnez **Ajouter** :

   ![Sélectionner Ajouter](media/tutorial-resource-onboarding/templatesadd.png)

4. Sous **Général**, entrez un **Nom** et une **Description** pour le nouveau modèle :

   ![Nom et description du modèle](media/tutorial-resource-onboarding/templatesdescription.png)

5. Créez le modèle Resource Manager en copiant dans le modèle JSON à partir de la section « Prise en main de l’intégration de ressources » de cet article :

   ![Créer un modèle Resource Manager](media/tutorial-resource-onboarding/templatesarmtemplate.png)

6. Sélectionnez **Ajouter** pour créer le modèle. Si le nouveau modèle n’apparaît pas, sélectionnez **Actualiser**.

7. Sélectionnez le modèle nouvellement créé, puis choisissez **Déployer** :

   ![Sélectionnez le nouveau modèle, puis choisissez Déployer](media/tutorial-resource-onboarding/templateselectspecific.png)

8. Entrez les paramètres des champs obligatoires, puis sélectionnez l’abonnement et le groupe de ressources. Vous pouvez laisser la zone **ID du fournisseur de ressources personnalisé** vide.

   | Nom du paramètre | Requis ? | Description |
   | ------------ | -------- | ----------- |
   | Emplacement | Oui | Emplacement des ressources dans le modèle. |
   | Nom de l’application logique | Non | Nom de l’application logique. |
   | Nom du fournisseur de ressources personnalisé | Non | Nom du fournisseur de ressources personnalisé. |
   | ID du fournisseur de ressources personnalisé | Non | Fournisseur de ressources personnalisé prenant en charge la ressource d’association. Si vous spécifiez une valeur ici, le déploiement de l’application logique et du fournisseur personnalisé sont ignorés. |
   | Nom de l’association | Non | Nom de la ressource d’association. |

   Exemples de paramètres :

   ![Entrer les paramètres du modèle](media/tutorial-resource-onboarding/templatescustomprovider.png)

9. Accédez au déploiement et attendez qu’il se termine. Un écran similaire à cette capture d’écran doit s’afficher. Vous devez voir la nouvelle ressource d’association comme sortie :

   ![Déploiement réussi](media/tutorial-resource-onboarding/customproviderdeployment.png)

   Voici le groupe de ressources, avec l’option **Afficher les types masqués** sélectionnée :

   ![Déploiement du fournisseur personnalisé](media/tutorial-resource-onboarding/showhidden.png)

10. Explorez l’onglet **Historique des exécutions** de l’application logique afin de voir les appels pour la création d’association :

    ![Historique des exécutions de l’application logique](media/tutorial-resource-onboarding/logicapprun.png)

## <a name="deploy-additional-associations"></a>Déployer des associations supplémentaires

Une fois l’infrastructure du fournisseur personnalisé configurée, vous pouvez facilement déployer davantage d’associations. Le groupe de ressources pour les associations supplémentaires n’a pas besoin d’être le même que le groupe de ressources dans lequel vous avez déployé l’infrastructure du fournisseur personnalisé. Pour créer une association, vous devez disposer d’autorisations Microsoft.CustomProviders/resourceproviders/write sur l’ID du fournisseur de ressources personnalisé spécifié.

1. Accédez à la ressource **Microsoft. CustomProviders/resourceProviders** du fournisseur personnalisé dans le groupe de ressources du déploiement précédent. Vous devez activer la case à cocher **Afficher les types masqués** :

   ![Accéder à la ressource](media/tutorial-resource-onboarding/showhidden.png)

2. Copiez la propriété ID de ressource du fournisseur personnalisé.

3. Recherchez **Modèles** dans **Tous les services** ou en utilisant la zone de recherche principale :

   ![Rechercher des modèles](media/tutorial-resource-onboarding/templates.png)

4. Sélectionnez le modèle créé précédemment, puis choisissez **Déployer** :

   ![Sélectionner le modèle créé précédemment, puis choisir Déployer](media/tutorial-resource-onboarding/templateselectspecific.png)

5. Entrez les paramètres des champs obligatoires, puis sélectionnez l’abonnement et un autre groupe de ressources. Pour le paramètre **ID du fournisseur de ressources personnalisé**, entrez l’ID de ressource que vous avez copié à partir du fournisseur personnalisé que vous avez déployé précédemment.

6. Accédez au déploiement et attendez qu’il se termine. Il doit maintenant déployer uniquement la nouvelle ressource associations :

   ![Nouvelle ressource associations](media/tutorial-resource-onboarding/createdassociationresource.png)

Si vous le souhaitez, vous pouvez revenir à l’**historique d’exécution** de l’application logique, et voir qu’un autre appel a été adressé à l’application logique. Vous pouvez mettre à jour l’application logique pour augmenter les fonctionnalités supplémentaires de chaque association créée.

## <a name="getting-help"></a>Obtenir de l’aide

Si vous avez des questions sur les fournisseurs de ressources personnalisés Azure, essayez de les poser sur [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-custom-providers). Avant de publier une question, vérifiez si une question similaire a déjà reçu une réponse. Ajoutez le mot clé `azure-custom-providers` pour obtenir une réponse rapide !

