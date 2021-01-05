---
title: Intégration et livraison continues dans Azure Data Factory
description: Découvrez comment utiliser l’intégration et la livraison continues pour déplacer les pipelines Data Factory d’un environnement (développement, test, production) à un autre.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
manager: jroth
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: 84e156074d6db837556ba4ed9febdb43bcdf3318
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96902300"
---
# <a name="continuous-integration-and-delivery-in-azure-data-factory"></a>Intégration et livraison continues dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

## <a name="overview"></a>Vue d’ensemble

L’intégration continue consiste à tester automatiquement et, dès que possible, chaque modification apportée à votre code base.  La livraison continue fait suite au test effectué pendant l’intégration continue, et envoie (push) les modifications à un système de préproduction ou de production.

Dans Azure Data Factory, l’intégration et la livraison continues (CI/CD) impliquent de déplacer des pipelines Data Factory d’un environnement (développement, test, production) vers un autre. Azure Data Factory utilise des [modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md) pour stocker la configuration de vos diverses entités ADF (pipelines, jeux de données, flux de données, etc.). Deux méthodes sont recommandées pour promouvoir une fabrique de données dans un autre environnement :

-    Déploiement automatisé grâce à l’intégration de Data Factory avec [Azure Pipelines](/azure/devops/pipelines/get-started/what-is-azure-pipelines?view=azure-devops).
-    Chargement manuel d’un modèle Resource Manager en tirant parti de l’intégration de l’expérience utilisateur de Data Factory avec Azure Resource Manager.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="cicd-lifecycle"></a>Cycle de vie d’intégration et de livraison continues

Vous trouverez ci-dessous une vue d’ensemble du cycle de vie d’intégration et de livraison continues dans une fabrique de données Azure configurée avec Azure Repos Git. Pour plus d’informations sur la configuration d’un dépôt Git, consultez [Contrôle de code source dans Azure Data Factory](source-control.md).

1.  Une fabrique de données de développement est créée et configurée avec Azure Repos Git. Tous les développeurs doivent avoir l’autorisation de créer des ressources Data Factory telles que des pipelines et des jeux de données.

1.  Un développeur [crée une branche de fonctionnalité](source-control.md#creating-feature-branches) pour apporter une modification. Il débogue les exécutions de son pipeline avec ses modifications les plus récentes. Pour plus d’informations sur le débogage d’une exécution de pipeline, consultez [Développement et débogage itératifs avec Azure Data Factory](iterative-development-debugging.md).

1.  Une fois que le développeur est satisfait de ses modifications, il crée une demande de tirage (pull) à partir de sa branche de fonctionnalité vers la branche principale ou la branche de collaboration pour que ces modifications soient examinées par des pairs.

1.  Une fois qu’une demande de tirage (pull) a été approuvée et que les modifications ont été fusionnées dans la branche principale, les modifications sont publiées dans la fabrique de développement.

1.  Lorsque l’équipe est prête à déployer les modifications dans une fabrique de test ou de test d’acceptation utilisateur (UAT), elle accède à la mise en production sur Azure Pipelines et déploie la version souhaitée de la fabrique de développement vers l’UAT. Ce déploiement a lieu dans le cadre d’une tâche Azure Pipelines et utilise des paramètres de modèle Resource Manager pour appliquer la configuration appropriée.

1.  Une fois les modifications vérifiées dans la fabrique de test, opérez le déploiement vers la fabrique de production en utilisant la tâche suivante de la mise en production de pipelines.

> [!NOTE]
> Seule la fabrique de développement est associée à un dépôt Git. Les fabriques de test et de production ne doivent pas avoir de dépôt Git associé et ne doivent être mises à jour que via un pipeline Azure DevOps ou un modèle de gestion des ressources.

L’image ci-dessous met en évidence les différentes étapes de ce cycle de vie.

![Diagramme de l’intégration continue à Azure Pipelines](media/continuous-integration-deployment/continuous-integration-image12.png)

## <a name="automate-continuous-integration-by-using-azure-pipelines-releases"></a>Automatiser l’intégration continue à l’aide des versions d’Azure Pipelines

Vous trouverez ci-après un guide de configuration d’une mise en production Azure Pipelines qui automatise le déploiement d’une fabrique de données dans plusieurs environnements.

### <a name="requirements"></a>Spécifications

-   Un abonnement Azure lié à Visual Studio Team Foundation Server ou Azure Repos qui utilise le [point de terminaison de service Azure Resource Manager](/azure/devops/pipelines/library/service-endpoints#sep-azure-resource-manager).

-   Une fabrique de données configurée avec l’intégration d’Azure Repos Git.

-   Un [coffre de clés Azure](https://azure.microsoft.com/services/key-vault/) contenant les secrets pour chaque environnement.

### <a name="set-up-an-azure-pipelines-release"></a>Configurer une version d’Azure Pipelines

1.  Dans [Azure DevOps](https://dev.azure.com/), ouvrez le projet configuré avec votre fabrique de données.

1.  Sur le côté gauche de la page, sélectionnez **Pipelines**, puis sélectionnez **Versions**.

    ![Sélectionner Pipelines, Versions](media/continuous-integration-deployment/continuous-integration-image6.png)

1.  Sélectionnez **Nouveau pipeline** ou, si vous avez des pipelines existants, sélectionnez **Nouveau** puis **Nouveau pipeline de mise en production**.

1.  Sélectionnez le modèle **Tâche vide**.

    ![Sélectionner Projet vide](media/continuous-integration-deployment/continuous-integration-image13.png)

1.  Dans la zone **Nom de la phase**, entrez le nom de votre environnement.

1.  Sélectionnez **Ajouter un artefact**, puis choisissez le dépôt Git configuré avec votre fabrique de données de développement. Sélectionnez la [branche de publication](source-control.md#configure-publishing-settings) du dépôt comme **branche par défaut**. Par défaut, cette branche de publication est `adf_publish`. Pour **Version par défaut**, sélectionnez **La dernière de la branche par défaut**.

    ![Ajouter un artefact](media/continuous-integration-deployment/continuous-integration-image7.png)

1.  Ajoutez une tâche de déploiement Azure Resource Manager :

    a.  Dans la vue des phases, sélectionnez **Afficher les tâches de phase**.

    ![Vue des phases](media/continuous-integration-deployment/continuous-integration-image14.png)

    b.  Créer une tâche. Recherchez **Déploiement de modèles ARM**, puis sélectionnez **Ajouter**.

    c.  Dans la tâche Déploiement, sélectionnez l’abonnement, le groupe de ressources et l’emplacement de la fabrique de données cible. Fournissez les informations d’identification si nécessaire.

    d.  Dans la liste **Actions**, sélectionnez **Créer ou mettre à jour un groupe de ressources**.

    e.  Sélectionnez le bouton de sélection ( **…** ) en regard de la zone **Modèle**. Recherchez le modèle Azure Resource Manager généré dans votre branche de publication du dépôt Git configuré. Recherchez le fichier `ARMTemplateForFactory.json` dans le dossier <FactoryName> de la branche adf_publish.

    f.  Sélectionnez **…** en regard de la zone **Paramètres du modèle** pour choisir le fichier de paramètres. Recherchez le fichier `ARMTemplateParametersForFactory.json` dans le dossier <FactoryName> de la branche adf_publish.

    g.  Sélectionnez **…** en regard de la zone **Remplacer les paramètres du modèle** et entrez les valeurs de paramètre souhaitées pour la fabrique de données cible. Pour les informations d’identification provenant d’Azure Key Vault, entrez le nom du secret entre guillemets doubles. Par exemple, si le nom du secret est cred1, entrez **"$(cred1)"** pour cette valeur.

    h. Sélectionnez **Incrémentiel** comme **Mode de déploiement**.

    > [!WARNING]
    > En mode de déploiement complet, les ressources présentes dans le groupe de ressources mais pas spécifiées dans le modèle Resource Manager sont **supprimées**. Pour plus d’informations, consultez [Modes de déploiement Azure Resource Manager](../azure-resource-manager/templates/deployment-modes.md).

    ![Déploiement en production de Data Factory](media/continuous-integration-deployment/continuous-integration-image9.png)

1.  Enregistrez le pipeline de mise en production.

1. Pour déclencher une mise en production, sélectionnez **Créer une mise en production**. Pour automatiser la création des mises en production, consultez les [déclencheurs de mise en production Azure DevOps](/azure/devops/pipelines/release/triggers?view=azure-devops).

   ![Sélectionner Créer une mise en production](media/continuous-integration-deployment/continuous-integration-image10.png)

> [!IMPORTANT]
> Dans les scénarios d’intégration et de livraison continues, le type de runtime d’intégration (IR) doit être le même dans les différents environnements. Par exemple, si vous avez un runtime d’intégration auto-hébergé dans l’environnement de développement, le même runtime d’intégration doit aussi être de type auto-hébergé dans les autres environnements (test, production). De même, si vous partagez des runtimes d’intégration dans plusieurs phases, vous devez les configurer comme étant liés et auto-hébergés dans tous les environnements (développement, test, production).

### <a name="get-secrets-from-azure-key-vault"></a>Obtenez les secrets à partir du coffre Azure Key Vault

Si vous avez des secrets à transmettre dans un modèle Azure Resource Manager, nous vous recommandons d’utiliser Azure Key Vault avec la version Azure Pipelines.

Il existe deux moyens de gérer les secrets :

1.  Ajoutez les secrets au fichier de paramètres. Pour plus d’informations, consultez [Use Azure Key Vault to pass secure parameter value during deployment](../azure-resource-manager/templates/key-vault-parameter.md) (Utiliser Azure Key Vault pour transmettre une valeur de paramètre sécurisée lors du déploiement).

    Créez une copie du fichier de paramètres qui sera chargée dans la branche de publication. Définissez les valeurs des paramètres que vous souhaitez obtenir à partir du coffre de clés avec le format suivant :

    ```json
    {
        "parameters": {
            "azureSqlReportingDbPassword": {
                "reference": {
                    "keyVault": {
                        "id": "/subscriptions/<subId>/resourceGroups/<resourcegroupId> /providers/Microsoft.KeyVault/vaults/<vault-name> "
                    },
                    "secretName": " < secret - name > "
                }
            }
        }
    }
    ```

    Lorsque vous utilisez cette méthode, le secret est automatiquement extrait du coffre de clés.

    Le fichier de paramètres doit également être dans la branche de publication.

1. Ajoutez une [tâche Azure Key Vault](/azure/devops/pipelines/tasks/deploy/azure-key-vault) avant la tâche de déploiement d’Azure Resource Manager décrite dans la section précédente :

    1.  Dans l’onglet **Tâches**, créez une nouvelle tâche. Recherchez **Azure Key Vault** et ajoutez-le.

    1.  Dans la tâche Key Vault, sélectionnez l’abonnement dans lequel vous avez créé le coffre de clés. Fournissez les informations d’identification si nécessaire, puis sélectionnez le coffre de clés.

    ![Ajouter une tâche Key Vault](media/continuous-integration-deployment/continuous-integration-image8.png)

#### <a name="grant-permissions-to-the-azure-pipelines-agent"></a>Accorder des autorisations à l’agent Azure Pipelines

La tâche Azure Key Vault peut échouer avec une erreur d’accès refusé si les autorisations appropriées ne sont pas définies. Téléchargez les journaux de la version, puis recherchez le fichier .ps1 contenant la commande pour accorder des autorisations à l’agent Azure Pipelines. Vous pouvez exécuter directement la commande. Vous pouvez aussi copier l’ID du principal à partir du fichier et ajouter manuellement la stratégie d’accès dans le Portail Azure. `Get` et `List` sont les autorisations minimales requises.

### <a name="updating-active-triggers"></a>Mise à jour des déclencheurs actifs

Le déploiement peut échouer si vous tentez de mettre à jour les déclencheurs actifs. Pour mettre à jour les déclencheurs actifs, vous devez les arrêter manuellement puis les redémarrer après le déploiement. Vous pouvez le faire à l’aide d’une tâche Azure PowerShell :

1.  Dans l’onglet **Tâches** de la version, ajoutez une tâche **Azure Powershell**. Choisissez une tâche de version 4.*. 

1.  Sélectionnez l’abonnement dans lequel se trouve votre fabrique.

1.  Sélectionnez **Chemin d’accès au fichier de script** comme type de script. Pour cela, vous devez enregistrer votre script PowerShell dans votre référentiel. Le script PowerShell suivant peut être utilisé pour arrêter des déclencheurs :

    ```powershell
    $triggersADF = Get-AzDataFactoryV2Trigger -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName

    $triggersADF | ForEach-Object { Stop-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.name -Force }
    ```

Vous pouvez suivre des étapes similaires (avec la fonction `Start-AzDataFactoryV2Trigger`) pour redémarrer les déclencheurs après le déploiement.

L’équipe Data Factory a fourni un [exemple de script de pré-déploiement et de post-déploiement](#script) qui se trouve au bas de cet article. 

## <a name="manually-promote-a-resource-manager-template-for-each-environment"></a>Promouvoir manuellement un modèle Resource Manager pour chaque environnement

1. Dans la liste **Modèle ARM**, sélectionnez **Exporter un modèle ARM** pour exporter le modèle Resource Manager de votre fabrique de données dans l’environnement de développement.

   ![Exporter un modèle Resource Manager](media/continuous-integration-deployment/continuous-integration-image1.png)

1. Dans vos fabriques de données de test et de production, sélectionnez **Importer un modèle ARM**. Cette action ouvre le portail Azure, dans lequel vous pouvez importer le modèle exporté. Sélectionnez **Créer votre propre modèle dans l’éditeur** pour ouvrir l’éditeur de modèle Resource Manager.

   ![Créer votre propre modèle](media/continuous-integration-deployment/custom-deployment-build-your-own-template.png) 

1. Sélectionnez **Charger le fichier**, puis sélectionnez le modèle Resource Manager généré. Il s’agit du fichier **arm_template.json** situé dans le fichier .zip exporté à l’étape 1.

   ![Modifier un modèle](media/continuous-integration-deployment/custom-deployment-edit-template.png)

1. Dans la section des paramètres, entrez les valeurs de configuration telles que les informations d’identification du service lié. Lorsque vous avez terminé, sélectionnez **Acheter** pour déployer le modèle Resource Manager.

   ![Section Paramètres](media/continuous-integration-deployment/continuous-integration-image5.png)

## <a name="use-custom-parameters-with-the-resource-manager-template"></a>Utiliser des paramètres personnalisés avec le modèle Resource Manager

Si votre fabrique de développement dispose d’un dépôt Git associé, vous pouvez remplacer les paramètres du modèle Resource Manager par défaut du modèle Resource Manager généré en publiant ou exportant le modèle. Vous souhaiterez peut-être remplacer le modèle de paramétrage par défaut dans les scénarios suivants :

* Vous utilisez CI/CD automatisé et souhaitez modifier certaines propriétés pendant le déploiement de Resource Manager, mais les propriétés ne sont pas paramétrables par défaut.
* Votre fabrique est si volumineuse que le modèle Resource Manager par défaut n’est pas valide car il dépasse le nombre maximum autorisé de paramètres (256).

Pour remplacer le modèle de paramétrage par défaut, accédez au hub de gestion et sélectionnez **Modèle de paramétrage** dans la section de contrôle de code source. Sélectionnez **Modifier le modèle** pour ouvrir l’éditeur de code du modèle de paramétrage. 

![Gérer les paramètres personnalisés](media/author-management-hub/management-hub-custom-parameters.png)

La création d’un modèle de paramétrage personnalisé, crée un fichier nommé **arm-template-parameters-definition.json** dans le dossier racine de votre branche Git. Vous devez utiliser ce nom de fichier exact.

![Fichier des paramètres personnalisé](media/continuous-integration-deployment/custom-parameters.png)

Lors de la publication à partir de la branche de collaboration, Data Factory lit ce fichier et utilise sa configuration pour générer les propriétés qui sont paramétrées. Si aucun fichier n’est trouvé, le modèle par défaut est utilisé.

Lors de l’exportation d’un modèle Resource Manager, Data Factory lit ce fichier à partir de la branche sur laquelle vous travaillez actuellement, et pas de la branche de collaboration. Vous pouvez créer ou modifier le fichier à partir d’une branche privée, dans laquelle vous pouvez tester vos modifications en sélectionnant **Exporter le modèle ARM** dans l’interface utilisateur. Vous pouvez ensuite fusionner le fichier dans la branche de collaboration.

> [!NOTE]
> Un modèle de paramétrage personnalisé ne change pas la limite de 256 paramètres du modèle ARM. Il vous permet de choisir et de diminuer le nombre de propriétés paramétrées.

### <a name="custom-parameter-syntax"></a>Syntaxe de paramètre personnalisé

Vous trouverez ci-dessous quelques recommandations à suivre lorsque vous créez le fichier de paramètres personnalisés **arm-template-parameters-definition.json**. Le fichier comprend une section pour chaque type d’entité : déclencheur, pipeline, service lié, jeu de données, runtime d’intégration et flux de données.

* Entrez le chemin d’accès de propriété sous le type d’entité correspondant.
* Définir un nom de propriété sur `*` indique que vous souhaitez paramétrer toutes les propriétés dans celle-ci (uniquement jusqu’au premier niveau, pas de manière récursive). Vous pouvez également fournir des exceptions à cette configuration.
* Définir la valeur d’une propriété sous forme de chaîne indique que vous souhaitez paramétrer la propriété. Utilisez le format `<action>:<name>:<stype>`.
   *  `<action>` peut être l’un des caractères suivants :
      * `=` permet de conserver la valeur actuelle en tant que valeur par défaut pour le paramètre.
      * `-` permet de ne pas conserver la valeur par défaut pour le paramètre.
      * `|` est un cas particulier pour les secrets Azure Key Vault pour les chaînes de connexion ou les clés.
   * `<name>` correspond au nom du paramètre. S’il est vide, il prend le nom du Si la valeur commence par un caractère `-`, le nom est abrégé. Par exemple, `AzureStorage1_properties_typeProperties_connectionString` serait abrégé en `AzureStorage1_connectionString`.
   * `<stype>` correspond au type de paramètre. Si `<stype>` est vide, le type par défaut est `string`. Valeurs prises en charge : `string`, `securestring`, `int`, `bool`, `object`, `secureobject` et `array`.
* La spécification d’un tableau dans le fichier de définition indique que la propriété correspondante dans le modèle est un tableau. Data Factory effectue une itération sur tous les objets du tableau en utilisant la définition spécifiée dans l’objet de runtime d’intégration du tableau. Le second objet, une chaîne, correspond alors au nom de la propriété et sert de nom au paramètre pour chaque itération.
* Une définition ne peut pas être spécifique à une instance de ressource. Toute définition s’applique à toutes les ressources de ce type.
* Par défaut, toutes les chaînes sécurisées, telles que les secrets Key Vault, et les chaînes sécurisées, telles que les chaînes de connexion, les clés et les jetons, sont paramétrables.
 
### <a name="sample-parameterization-template"></a>Exemple de modèle de paramétrage

Voici un exemple de ce que à quoi peut ressembler un modèle de paramétrage :

```json
{
    "Microsoft.DataFactory/factories/pipelines": {
        "properties": {
            "activities": [{
                "typeProperties": {
                    "waitTimeInSeconds": "-::int",
                    "headers": "=::object"
                }
            }]
        }
    },
    "Microsoft.DataFactory/factories/integrationRuntimes": {
        "properties": {
            "typeProperties": {
                "*": "="
            }
        }
    },
    "Microsoft.DataFactory/factories/triggers": {
        "properties": {
            "typeProperties": {
                "recurrence": {
                    "*": "=",
                    "interval": "=:triggerSuffix:int",
                    "frequency": "=:-freq"
                },
                "maxConcurrency": "="
            }
        }
    },
    "Microsoft.DataFactory/factories/linkedServices": {
        "*": {
            "properties": {
                "typeProperties": {
                    "accountName": "=",
                    "username": "=",
                    "connectionString": "|:-connectionString:secureString",
                    "secretAccessKey": "|"
                }
            }
        },
        "AzureDataLakeStore": {
            "properties": {
                "typeProperties": {
                    "dataLakeStoreUri": "="
                }
            }
        }
    },
    "Microsoft.DataFactory/factories/datasets": {
        "properties": {
            "typeProperties": {
                "*": "="
            }
        }
    }
}
```
Voici une explication de la façon dont le modèle précédent est construit, décomposé par type de ressource.

#### <a name="pipelines"></a>Pipelines
    
* Toute propriété du chemin `activities/typeProperties/waitTimeInSeconds` est paramétrable. Toute activité dans un pipeline qui a une propriété au niveau du code nommée `waitTimeInSeconds` (par exemple, l’activité `Wait`) est paramétrable en tant que nombre, avec un nom par défaut. Mais elle n’a pas de valeur par défaut dans le modèle Resource Manager. Il s’agit d’une entrée obligatoire lors du déploiement de Resource Manager.
* De même, une propriété appelée `headers` (par exemple, dans une activité `Web`) est paramétrable avec le type `object` (JObject). Elle a une valeur par défaut, qui est la même que celle de la fabrique source.

#### <a name="integrationruntimes"></a>IntegrationRuntimes

* Toutes les propriétés, sous le chemin `typeProperties` sont paramétrables avec des valeurs par défaut respectives. Par exemple, deux propriétés existent sous les propriétés de type `IntegrationRuntimes` : `computeProperties` et `ssisProperties`. Les deux types de propriété sont créés avec leurs valeurs et types (objet) par défaut respectifs.

#### <a name="triggers"></a>Déclencheurs

* Sous `typeProperties`, deux propriétés sont paramétrables. La première est `maxConcurrency`, qui est spécifiée pour avoir une valeur par défaut et est de type `string`. Elle porte le nom de paramètre par défaut `<entityName>_properties_typeProperties_maxConcurrency`.
* La propriété `recurrence` est également paramétrable. Sous celle-ci, toutes les propriétés à ce niveau sont spécifiées pour être paramétrables sous forme de chaînes, avec des valeurs et noms de paramètres par défaut. La propriété `interval` est une exception, qui est paramétrée en tant que type `int`. Le nom du paramètre a pour suffixe `<entityName>_properties_typeProperties_recurrence_triggerSuffix`. De même, la propriété `freq` est une chaîne et peut être paramétrée en tant que chaîne. La propriété `freq` est toutefois paramétrable sans valeur par défaut. Le nom est abrégé et suivi d’un suffixe. Par exemple : `<entityName>_freq`.

#### <a name="linkedservices"></a>LinkedServices

* Les services liés sont uniques. Étant donné que les services liés et les jeux de données sont de types différents, vous pouvez fournir une personnalisation spécifique au type. Dans cet exemple, pour tous les services liés de type `AzureDataLakeStore`, un modèle spécifique est appliqué. Pour tous les autres (via `*`), un autre modèle est appliqué.
* La propriété `connectionString` est paramétrée en tant que valeur `securestring`. Elle n’a pas de valeur par défaut. Elle a un nom de paramètre raccourci doté du suffixe `connectionString`.
* La propriété `secretAccessKey` se trouve être un `AzureKeyVaultSecret` (par exemple, dans un service lié Amazon S3). Elle est paramétrable automatiquement en tant que secret Azure Key Vault et extraite du coffre de clés configuré. Vous pouvez également paramétrer le coffre de clés proprement dit.

#### <a name="datasets"></a>Groupes de données

* La personnalisation spécifique au type est disponible pour les jeux de données, mais vous pouvez fournir une configuration sans avoir explicitement de configuration au niveau \*. Dans l’exemple précédent, toutes les propriétés du jeu de données sous `typeProperties` sont paramétrables.

### <a name="default-parameterization-template"></a>Modèle de paramétrage par défaut

Vous trouverez ci-dessous le modèle actuel. Si vous n’avez besoin d’ajouter que quelques paramètres, il peut être judicieux de modifier directement ce modèle, car vous ne perdrez pas la structure de paramétrage existante.

```json
{
    "Microsoft.DataFactory/factories": {
        "properties": {
            "globalParameters": {
                "*": {
                    "value": "="
                }
            }
        },
        "location": "="
    },
    "Microsoft.DataFactory/factories/pipelines": {
    },
    "Microsoft.DataFactory/factories/dataflows": {
    },
    "Microsoft.DataFactory/factories/integrationRuntimes":{
        "properties": {
            "typeProperties": {
                "ssisProperties": {
                    "catalogInfo": {
                        "catalogServerEndpoint": "=",
                        "catalogAdminUserName": "=",
                        "catalogAdminPassword": {
                            "value": "-::secureString"
                        }
                    },
                    "customSetupScriptProperties": {
                        "sasToken": {
                            "value": "-::secureString"
                        }
                    }
                },
                "linkedInfo": {
                    "key": {
                        "value": "-::secureString"
                    },
                    "resourceId": "="
                },
                "computeProperties": {
                    "dataFlowProperties": {
                        "externalComputeInfo": [{
                                "accessToken": "-::secureString"
                            }
                        ]
                    }
                }
            }
        }
    },
    "Microsoft.DataFactory/factories/triggers": {
        "properties": {
            "pipelines": [{
                    "parameters": {
                        "*": "="
                    }
                },  
                "pipelineReference.referenceName"
            ],
            "pipeline": {
                "parameters": {
                    "*": "="
                }
            },
            "typeProperties": {
                "scope": "="
            }
        }
    },
    "Microsoft.DataFactory/factories/linkedServices": {
        "*": {
            "properties": {
                "typeProperties": {
                    "accountName": "=",
                    "username": "=",
                    "userName": "=",
                    "accessKeyId": "=",
                    "servicePrincipalId": "=",
                    "userId": "=",
                    "host": "=",
                    "clientId": "=",
                    "clusterUserName": "=",
                    "clusterSshUserName": "=",
                    "hostSubscriptionId": "=",
                    "clusterResourceGroup": "=",
                    "subscriptionId": "=",
                    "resourceGroupName": "=",
                    "tenant": "=",
                    "dataLakeStoreUri": "=",
                    "baseUrl": "=",
                    "database": "=",
                    "serviceEndpoint": "=",
                    "batchUri": "=",
                    "poolName": "=",
                    "databaseName": "=",
                    "systemNumber": "=",
                    "server": "=",
                    "url":"=",
                    "functionAppUrl":"=",
                    "environmentUrl": "=",
                    "aadResourceId": "=",
                    "sasUri": "|:-sasUri:secureString",
                    "sasToken": "|",
                    "connectionString": "|:-connectionString:secureString",
                    "hostKeyFingerprint": "="
                }
            }
        },
        "Odbc": {
            "properties": {
                "typeProperties": {
                    "userName": "=",
                    "connectionString": {
                        "secretName": "="
                    }
                }
            }
        }
    },
    "Microsoft.DataFactory/factories/datasets": {
        "*": {
            "properties": {
                "typeProperties": {
                    "folderPath": "=",
                    "fileName": "="
                }
            }
        }
    },
    "Microsoft.DataFactory/factories/managedVirtualNetworks/managedPrivateEndpoints": {
        "properties": {
            "*": "="
        }
    }
}
```

### <a name="example-parameterizing-an-existing-azure-databricks-interactive-cluster-id"></a>Exemple : paramétrage d’un ID de cluster interactif Azure Databricks existant

L’exemple suivant montre comment ajouter une valeur unique au modèle de paramétrage par défaut. Nous voulons seulement ajouter un ID de cluster Azure Databricks interactif pour un service Databricks lié au fichier de paramètres. Notez que ce fichier est le même que le fichier précédent, à l’exception de l’ajout de `existingClusterId` sous le champ de propriétés de `Microsoft.DataFactory/factories/linkedServices`.

```json
{
    "Microsoft.DataFactory/factories": {
        "properties": {
            "globalParameters": {
                "*": {
                    "value": "="
                }
            }
        },
        "location": "="
    },
    "Microsoft.DataFactory/factories/pipelines": {
    },
    "Microsoft.DataFactory/factories/dataflows": {
    },
    "Microsoft.DataFactory/factories/integrationRuntimes":{
        "properties": {
            "typeProperties": {
                "ssisProperties": {
                    "catalogInfo": {
                        "catalogServerEndpoint": "=",
                        "catalogAdminUserName": "=",
                        "catalogAdminPassword": {
                            "value": "-::secureString"
                        }
                    },
                    "customSetupScriptProperties": {
                        "sasToken": {
                            "value": "-::secureString"
                        }
                    }
                },
                "linkedInfo": {
                    "key": {
                        "value": "-::secureString"
                    },
                    "resourceId": "="
                }
            }
        }
    },
    "Microsoft.DataFactory/factories/triggers": {
        "properties": {
            "pipelines": [{
                    "parameters": {
                        "*": "="
                    }
                },  
                "pipelineReference.referenceName"
            ],
            "pipeline": {
                "parameters": {
                    "*": "="
                }
            },
            "typeProperties": {
                "scope": "="
            }
 
        }
    },
    "Microsoft.DataFactory/factories/linkedServices": {
        "*": {
            "properties": {
                "typeProperties": {
                    "accountName": "=",
                    "username": "=",
                    "userName": "=",
                    "accessKeyId": "=",
                    "servicePrincipalId": "=",
                    "userId": "=",
                    "clientId": "=",
                    "clusterUserName": "=",
                    "clusterSshUserName": "=",
                    "hostSubscriptionId": "=",
                    "clusterResourceGroup": "=",
                    "subscriptionId": "=",
                    "resourceGroupName": "=",
                    "tenant": "=",
                    "dataLakeStoreUri": "=",
                    "baseUrl": "=",
                    "database": "=",
                    "serviceEndpoint": "=",
                    "batchUri": "=",
                    "poolName": "=",
                    "databaseName": "=",
                    "systemNumber": "=",
                    "server": "=",
                    "url":"=",
                    "aadResourceId": "=",
                    "connectionString": "|:-connectionString:secureString",
                    "existingClusterId": "-"
                }
            }
        },
        "Odbc": {
            "properties": {
                "typeProperties": {
                    "userName": "=",
                    "connectionString": {
                        "secretName": "="
                    }
                }
            }
        }
    },
    "Microsoft.DataFactory/factories/datasets": {
        "*": {
            "properties": {
                "typeProperties": {
                    "folderPath": "=",
                    "fileName": "="
                }
            }
        }}
}
```

## <a name="linked-resource-manager-templates"></a>Modèles Resource Manager liés

Si vous avez configuré l’intégration et la livraison continues (CI/CD) pour vos fabriques de données, vous pouvez dépasser les limites du modèle Azure Resource Manager lorsque votre fabrique croît. Par exemple, le nombre maximal de ressources dans un modèle Resource Manager est une limite. Pour prendre en compte des fabriques de grande taille tout en générant le modèle Resource Manager complet pour une fabrique, Data Factory génère désormais des modèles Resource Manager liés. Avec cette fonctionnalité, la charge utile de fabrique entière est divisée en plusieurs fichiers, pour que vous ne soyez pas contraint par les limites.

Si vous avez configuré Git, les modèles liés sont générés et enregistrés en même temps que les modèles Resource Manager complets, dans la branche adf_publish, dans un nouveau dossier nommé linkedTemplates :

![Dossier de modèles Resource Manager liés](media/continuous-integration-deployment/linked-resource-manager-templates.png)

Les modèles Resource Manager liés sont généralement composés d’un modèle maître et d’un ensemble de modèles enfants liés au maître. Le modèle parent est appelé ArmTemplate_master.json et les modèles enfants sont nommés selon le modèle ArmTemplate_0.json, ArmTemplate_1.json, etc. 

Pour utiliser des modèles liés à la place du modèle Resource Manager complet, mettez à jour votre tâche CI/CD de manière à pointer vers ArmTemplate_master.json plutôt que vers ArmTemplateForFactory.json (modèle Resource Manager complet). Resource Manager exige également que vous chargiez les modèles liés dans un compte de stockage pour qu’Azure puisse y accéder pendant le déploiement. Pour plus d’informations, consultez [Deploying linked Resource Manager templates with VSTS](/archive/blogs/najib/deploying-linked-arm-templates-with-vsts) (Déploiement de modèles Resource Manager liés avec VSTS).

N’oubliez pas d’ajouter les scripts Data Factory dans votre pipeline CI/CD avant et après la tâche de déploiement.

Si vous n’avez pas configuré Git, vous pouvez accéder aux modèles liés via **Exporter le modèle ARM** dans la liste **Modèle ARM**.

## <a name="hotfix-production-environment"></a>Environnement de production de correctif logiciel

Si vous déployez une fabrique en production et détectez un bogue qui doit être corrigé immédiatement, mais que vous ne pouvez pas déployer la branche de collaboration actuelle, vous devrez peut-être déployer un correctif logiciel. Cette approche est également connue sous le nom de QFE (Quick-Fix Engineering).

1.    Dans Azure DevOps, accédez à la version qui a été déployée en production. Recherchez la dernière validation qui a été déployée.

2.    À partir du message de validation, obtenez l’ID de validation de la branche de collaboration.

3.    Créez une nouvelle branche de correctifs logiciels à partir de cette validation.

4.    Accédez à l’expérience utilisateur Azure Data Factory et basculez vers cette branche de correctifs logiciels.

5.    Corrigez le bogue en utilisant l’expérience utilisateur Azure Data Factory. Tester vos modifications.

6.    Une fois le correctif vérifié, sélectionnez **Exporter le modèle ARM** pour obtenir le modèle Resource Manager du correctif logiciel.

7.    Archivez manuellement cette build dans la branche adf_publish.

8.    Si vous avez configuré votre pipeline de mise en production pour qu’il se déclenche automatiquement en fonction des archivages adf_publish, une nouvelle version démarre automatiquement. Sinon, créez manuellement une file d'attente de versions.

9.    Déployez la version avec correctif logiciel dans les fabriques de test et de production. Cette version contient la charge utile de production précédente ainsi que la correction apportée à l’étape 5.

10.   Ajoutez les modifications issues du correctif logiciel dans la branche de développement pour que les versions ultérieures n’incluent pas le même bogue.

Regardez la vidéo ci-dessous, un didacticiel vidéo détaillé sur la façon de corriger vos environnements à chaud. 

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4I7fi]

## <a name="exposure-control-and-feature-flags"></a>Indicateurs de contrôle d’exposition et de fonctionnalité

Lorsque vous travaillez en équipe, vous pouvez souhaiter fusionner des modifications, sans les exécuter dans des environnements élevés tels que PROD et QA. Pour gérer un tel scénario, l’équipe ADF recommande [le concept DevOps utilisant les indicateurs de fonctionnalité](/azure/devops/migrate/phase-features-with-feature-flags?view=azure-devops). Dans ADF, vous pouvez combiner les [paramètres globaux](author-global-parameters.md) et l’[activité IfCondition](control-flow-if-condition-activity.md) pour masquer des ensembles de logique en fonction de ces indicateurs d’environnement.

Pour savoir comment configurer un indicateur de fonctionnalité, consultez le tutoriel vidéo ci-dessous :

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4IxdW]

## <a name="best-practices-for-cicd"></a>Meilleures pratiques pour CI/CD

Si vous utilisez une intégration Git avec votre fabrique de données, et disposez d’un pipeline CI/CD qui déplace vos modifications du développement aux tests, puis en production, nous vous recommandons les bonnes pratiques suivantes :

-   **Intégration Git**. Configurez uniquement votre fabrique de données de développement avec l’intégration Git. Les modifications au niveau des tests et de la production sont déployées via CI/CD et ne nécessitent pas d’intégration Git.

-   **Script de pré-déploiement et de post-déploiement**. Avant l’étape de déploiement Resource Manager dans CI/CD, vous devez effectuer certaines tâches, telles que l’arrêt et le redémarrage des déclencheurs, et le nettoyage. Nous vous recommandons d’utiliser des scripts PowerShell avant et après la tâche de déploiement. Pour plus d’informations, consultez [Mettre à jour des déclencheurs actifs](#updating-active-triggers). L’équipe Data Factory a [fourni un script](#script) à utiliser, qui se trouve en bas de cette page.

-   **Runtimes d’intégration et partage**. Les runtimes d’intégration ne changent pas souvent et sont similaires dans toutes les phases de CI/CD. Ainsi, Data Factory s’attend à ce que vous ayez le même nom et le même type de runtime d’intégration dans toutes les phases de CI/CD. Si vous voulez partager les runtimes d’intégration dans toutes les phases, envisagez d’utiliser une fabrique ternaire qui contiendra uniquement les runtimes d’intégration partagés. Vous pouvez utiliser cette fabrique partagée dans tous vos environnements en tant que type de runtime d’intégration lié.

-   **Déploiement du point de terminaison privé managé**. Si un point de terminaison privé existe déjà dans une fabrique et que vous essayez de déployer un modèle ARM qui contient un point de terminaison privé portant le même nom mais dont les propriétés sont modifiées, le déploiement échoue. En d’autres termes, vous pouvez déployer avec succès un point de terminaison privé, à condition qu’il ait les mêmes propriétés que celui qui existe déjà dans la fabrique. Si une propriété est différente d’un environnement à un autre, vous pouvez la remplacer en paramétrant cette propriété et en fournissant la valeur correspondante pendant le déploiement.

-   **Key Vault**. Lorsque vous utilisez des services liés dont les informations de connexion sont stockées dans Azure Key Vault, il est recommandé de conserver des coffres de clés distincts pour les différents environnements. Vous pouvez également configurer des niveaux d’autorisation distincts pour chaque coffre de clés. Par exemple, vous ne souhaitez peut-être pas que les membres de votre équipe disposent d’autorisations sur les secrets de production. Si vous suivez cette approche, nous vous recommandons de conserver les mêmes noms de secrets dans toutes les phases. Si vous conservez les mêmes noms secrets, vous n’avez pas besoin de paramétrer chaque chaîne de connexion dans les environnements d’intégration et de livraison continues, car la seule chose qui change est le nom du coffre de clés, qui est un paramètre distinct.

-  **Nommage des ressources** En raison de contraintes liées au modèle ARM, des problèmes de déploiement peuvent survenir si le nom de vos ressources contient des espaces. L’équipe Azure Data Factory recommande d’utiliser des caractères « _ » ou « - » au lieu d’espaces dans les noms de ressources. Par exemple, le nom « Pipeline_1 » est préférable à « Pipeline 1 ».

## <a name="unsupported-features"></a>Fonctionnalités non prises en charge

- Par conception, Data Factory n’autorise pas le cherry-picking des validations ni la publication sélective des ressources. Les publications incluent toutes les modifications apportées dans la fabrique de données.

    - Les entités Data Factory dépendent les unes des autres. Par exemple, les déclencheurs dépendent des pipelines et les pipelines dépendent des jeux de données et d’autres pipelines. La publication sélective d’un sous-ensemble de ressources peut engendrer des comportements inattendus et des erreurs.
    - Dans les rares cas où vous avez besoin d’une publication sélective, envisagez d’utiliser un correctif logiciel. Pour plus d’informations, consultez [Environnement de production de correctif logiciel](#hotfix-production-environment).

- L’équipe Azure Data Factory ne recommande pas d’attribuer des contrôles Azure RBAC à des entités individuelles (pipelines, jeux de données, etc.) dans une fabrique de données. Par exemple, si un développeur a accès à un pipeline ou à un jeu de données, il doit être en mesure d’accéder à tous les pipelines ou jeux de données de la fabrique de données. Si vous estimez que vous devez implémenter de nombreux rôles Azure dans une fabrique de données, envisagez de déployer une deuxième fabrique de données.

-   Vous ne pouvez pas publier à partir de branches privées.

-   Vous ne pouvez actuellement pas héberger de projets sur Bitbucket.

## <a name="sample-pre--and-post-deployment-script"></a><a name="script"></a> Exemple de script de pré-déploiement et de post-déploiement

L’exemple de script suivant peut être utilisé pour arrêter les déclencheurs avant le déploiement, et les redémarrer après. Le script inclut également le code pour supprimer les ressources qui ont été retirées. Enregistrez le script dans un dépôt git Azure DevOps et référencez-le par le biais d’une tâche Azure PowerShell à l’aide de la version 4.*.

Lors de l’exécution d’un script de prédéploiement, vous devez spécifier une variation des paramètres suivants dans le champ **Arguments de script**.

`-armTemplate "$(System.DefaultWorkingDirectory)/<your-arm-template-location>" -ResourceGroupName <your-resource-group-name> -DataFactoryName <your-data-factory-name>  -predeployment $true -deleteDeployment $false`


Lors de l’exécution d’un script de post-déploiement, vous devez spécifier une variation des paramètres suivants dans le champ **Arguments de script**.

`-armTemplate "$(System.DefaultWorkingDirectory)/<your-arm-template-location>" -ResourceGroupName <your-resource-group-name> -DataFactoryName <your-data-factory-name>  -predeployment $false -deleteDeployment $true`

![Tâche Azure PowerShell](media/continuous-integration-deployment/continuous-integration-image11.png)

Voici le script qui peut être utilisé avant et après le déploiement. Il prend en compte les ressources supprimées et les références de ressources.

  
```powershell
param
(
    [parameter(Mandatory = $false)] [String] $armTemplate,
    [parameter(Mandatory = $false)] [String] $ResourceGroupName,
    [parameter(Mandatory = $false)] [String] $DataFactoryName,
    [parameter(Mandatory = $false)] [Bool] $predeployment=$true,
    [parameter(Mandatory = $false)] [Bool] $deleteDeployment=$false
)

function getPipelineDependencies {
    param([System.Object] $activity)
    if ($activity.Pipeline) {
        return @($activity.Pipeline.ReferenceName)
    } elseif ($activity.Activities) {
        $result = @()
        $activity.Activities | ForEach-Object{ $result += getPipelineDependencies -activity $_ }
        return $result
    } elseif ($activity.ifFalseActivities -or $activity.ifTrueActivities) {
        $result = @()
        $activity.ifFalseActivities | Where-Object {$_ -ne $null} | ForEach-Object{ $result += getPipelineDependencies -activity $_ }
        $activity.ifTrueActivities | Where-Object {$_ -ne $null} | ForEach-Object{ $result += getPipelineDependencies -activity $_ }
        return $result
    } elseif ($activity.defaultActivities) {
        $result = @()
        $activity.defaultActivities | ForEach-Object{ $result += getPipelineDependencies -activity $_ }
        if ($activity.cases) {
            $activity.cases | ForEach-Object{ $_.activities } | ForEach-Object{$result += getPipelineDependencies -activity $_ }
        }
        return $result
    } else {
        return @()
    }
}

function pipelineSortUtil {
    param([Microsoft.Azure.Commands.DataFactoryV2.Models.PSPipeline]$pipeline,
    [Hashtable] $pipelineNameResourceDict,
    [Hashtable] $visited,
    [System.Collections.Stack] $sortedList)
    if ($visited[$pipeline.Name] -eq $true) {
        return;
    }
    $visited[$pipeline.Name] = $true;
    $pipeline.Activities | ForEach-Object{ getPipelineDependencies -activity $_ -pipelineNameResourceDict $pipelineNameResourceDict}  | ForEach-Object{
        pipelineSortUtil -pipeline $pipelineNameResourceDict[$_] -pipelineNameResourceDict $pipelineNameResourceDict -visited $visited -sortedList $sortedList
    }
    $sortedList.Push($pipeline)

}

function Get-SortedPipelines {
    param(
        [string] $DataFactoryName,
        [string] $ResourceGroupName
    )
    $pipelines = Get-AzDataFactoryV2Pipeline -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
    $ppDict = @{}
    $visited = @{}
    $stack = new-object System.Collections.Stack
    $pipelines | ForEach-Object{ $ppDict[$_.Name] = $_ }
    $pipelines | ForEach-Object{ pipelineSortUtil -pipeline $_ -pipelineNameResourceDict $ppDict -visited $visited -sortedList $stack }
    $sortedList = new-object Collections.Generic.List[Microsoft.Azure.Commands.DataFactoryV2.Models.PSPipeline]
    
    while ($stack.Count -gt 0) {
        $sortedList.Add($stack.Pop())
    }
    $sortedList
}

function triggerSortUtil {
    param([Microsoft.Azure.Commands.DataFactoryV2.Models.PSTrigger]$trigger,
    [Hashtable] $triggerNameResourceDict,
    [Hashtable] $visited,
    [System.Collections.Stack] $sortedList)
    if ($visited[$trigger.Name] -eq $true) {
        return;
    }
    $visited[$trigger.Name] = $true;
    if ($trigger.Properties.DependsOn) {
        $trigger.Properties.DependsOn | Where-Object {$_ -and $_.ReferenceTrigger} | ForEach-Object{
            triggerSortUtil -trigger $triggerNameResourceDict[$_.ReferenceTrigger.ReferenceName] -triggerNameResourceDict $triggerNameResourceDict -visited $visited -sortedList $sortedList
        }
    }
    $sortedList.Push($trigger)
}

function Get-SortedTriggers {
    param(
        [string] $DataFactoryName,
        [string] $ResourceGroupName
    )
    $triggers = Get-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName
    $triggerDict = @{}
    $visited = @{}
    $stack = new-object System.Collections.Stack
    $triggers | ForEach-Object{ $triggerDict[$_.Name] = $_ }
    $triggers | ForEach-Object{ triggerSortUtil -trigger $_ -triggerNameResourceDict $triggerDict -visited $visited -sortedList $stack }
    $sortedList = new-object Collections.Generic.List[Microsoft.Azure.Commands.DataFactoryV2.Models.PSTrigger]
    
    while ($stack.Count -gt 0) {
        $sortedList.Add($stack.Pop())
    }
    $sortedList
}

function Get-SortedLinkedServices {
    param(
        [string] $DataFactoryName,
        [string] $ResourceGroupName
    )
    $linkedServices = Get-AzDataFactoryV2LinkedService -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName
    $LinkedServiceHasDependencies = @('HDInsightLinkedService', 'HDInsightOnDemandLinkedService', 'AzureBatchLinkedService')
    $Akv = 'AzureKeyVaultLinkedService'
    $HighOrderList = New-Object Collections.Generic.List[Microsoft.Azure.Commands.DataFactoryV2.Models.PSLinkedService]
    $RegularList = New-Object Collections.Generic.List[Microsoft.Azure.Commands.DataFactoryV2.Models.PSLinkedService]
    $AkvList = New-Object Collections.Generic.List[Microsoft.Azure.Commands.DataFactoryV2.Models.PSLinkedService]

    $linkedServices | ForEach-Object {
        if ($_.Properties.GetType().Name -in $LinkedServiceHasDependencies) {
            $HighOrderList.Add($_)
        }
        elseif ($_.Properties.GetType().Name -eq $Akv) {
            $AkvList.Add($_)
        }
        else {
            $RegularList.Add($_)
        }
    }

    $SortedList = New-Object Collections.Generic.List[Microsoft.Azure.Commands.DataFactoryV2.Models.PSLinkedService]($HighOrderList.Count + $RegularList.Count + $AkvList.Count)
    $SortedList.AddRange($HighOrderList)
    $SortedList.AddRange($RegularList)
    $SortedList.AddRange($AkvList)
    $SortedList
}

$templateJson = Get-Content $armTemplate | ConvertFrom-Json
$resources = $templateJson.resources

#Triggers 
Write-Host "Getting triggers"
$triggersInTemplate = $resources | Where-Object { $_.type -eq "Microsoft.DataFactory/factories/triggers" }
$triggerNamesInTemplate = $triggersInTemplate | ForEach-Object {$_.name.Substring(37, $_.name.Length-40)}

$triggersDeployed = Get-SortedTriggers -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName

$triggersToStop = $triggersDeployed | Where-Object { $triggerNamesInTemplate -contains $_.Name } | ForEach-Object { 
    New-Object PSObject -Property @{
        Name = $_.Name
        TriggerType = $_.Properties.GetType().Name 
    }
}
$triggersToDelete = $triggersDeployed | Where-Object { $triggerNamesInTemplate -notcontains $_.Name } | ForEach-Object { 
    New-Object PSObject -Property @{
        Name = $_.Name
        TriggerType = $_.Properties.GetType().Name 
    }
}
$triggersToStart = $triggersInTemplate | Where-Object { $_.properties.runtimeState -eq "Started" -and ($_.properties.pipelines.Count -gt 0 -or $_.properties.pipeline.pipelineReference -ne $null)} | ForEach-Object { 
    New-Object PSObject -Property @{
        Name = $_.name.Substring(37, $_.name.Length-40)
        TriggerType = $_.Properties.type
    }
}

if ($predeployment -eq $true) {
    #Stop all triggers
    Write-Host "Stopping deployed triggers`n"
    $triggersToStop | ForEach-Object {
        if ($_.TriggerType -eq "BlobEventsTrigger") {
            Write-Host "Unsubscribing" $_.Name "from events"
            $status = Remove-AzDataFactoryV2TriggerSubscription -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name
            while ($status.Status -ne "Disabled"){
                Start-Sleep -s 15
                $status = Get-AzDataFactoryV2TriggerSubscriptionStatus -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name
            }
        }
        Write-Host "Stopping trigger" $_.Name
        Stop-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name -Force
    }
}
else {
    #Deleted resources
    #pipelines
    Write-Host "Getting pipelines"
    $pipelinesADF = Get-SortedPipelines -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
    $pipelinesTemplate = $resources | Where-Object { $_.type -eq "Microsoft.DataFactory/factories/pipelines" }
    $pipelinesNames = $pipelinesTemplate | ForEach-Object {$_.name.Substring(37, $_.name.Length-40)}
    $deletedpipelines = $pipelinesADF | Where-Object { $pipelinesNames -notcontains $_.Name }
    #dataflows
    $dataflowsADF = Get-AzDataFactoryV2DataFlow -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
    $dataflowsTemplate = $resources | Where-Object { $_.type -eq "Microsoft.DataFactory/factories/dataflows" }
    $dataflowsNames = $dataflowsTemplate | ForEach-Object {$_.name.Substring(37, $_.name.Length-40) }
    $deleteddataflow = $dataflowsADF | Where-Object { $dataflowsNames -notcontains $_.Name }
    #datasets
    Write-Host "Getting datasets"
    $datasetsADF = Get-AzDataFactoryV2Dataset -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
    $datasetsTemplate = $resources | Where-Object { $_.type -eq "Microsoft.DataFactory/factories/datasets" }
    $datasetsNames = $datasetsTemplate | ForEach-Object {$_.name.Substring(37, $_.name.Length-40) }
    $deleteddataset = $datasetsADF | Where-Object { $datasetsNames -notcontains $_.Name }
    #linkedservices
    Write-Host "Getting linked services"
    $linkedservicesADF = Get-SortedLinkedServices -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
    $linkedservicesTemplate = $resources | Where-Object { $_.type -eq "Microsoft.DataFactory/factories/linkedservices" }
    $linkedservicesNames = $linkedservicesTemplate | ForEach-Object {$_.name.Substring(37, $_.name.Length-40)}
    $deletedlinkedservices = $linkedservicesADF | Where-Object { $linkedservicesNames -notcontains $_.Name }
    #Integrationruntimes
    Write-Host "Getting integration runtimes"
    $integrationruntimesADF = Get-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName
    $integrationruntimesTemplate = $resources | Where-Object { $_.type -eq "Microsoft.DataFactory/factories/integrationruntimes" }
    $integrationruntimesNames = $integrationruntimesTemplate | ForEach-Object {$_.name.Substring(37, $_.name.Length-40)}
    $deletedintegrationruntimes = $integrationruntimesADF | Where-Object { $integrationruntimesNames -notcontains $_.Name }

    #Delete resources
    Write-Host "Deleting triggers"
    $triggersToDelete | ForEach-Object { 
        Write-Host "Deleting trigger "  $_.Name
        $trig = Get-AzDataFactoryV2Trigger -name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName
        if ($trig.RuntimeState -eq "Started") {
            if ($_.TriggerType -eq "BlobEventsTrigger") {
                Write-Host "Unsubscribing trigger" $_.Name "from events"
                $status = Remove-AzDataFactoryV2TriggerSubscription -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name
                while ($status.Status -ne "Disabled"){
                    Start-Sleep -s 15
                    $status = Get-AzDataFactoryV2TriggerSubscriptionStatus -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name
                }
            }
            Stop-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name -Force 
        }
        Remove-AzDataFactoryV2Trigger -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
    }
    Write-Host "Deleting pipelines"
    $deletedpipelines | ForEach-Object { 
        Write-Host "Deleting pipeline " $_.Name
        Remove-AzDataFactoryV2Pipeline -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
    }
    Write-Host "Deleting dataflows"
    $deleteddataflow | ForEach-Object { 
        Write-Host "Deleting dataflow " $_.Name
        Remove-AzDataFactoryV2DataFlow -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
    }
    Write-Host "Deleting datasets"
    $deleteddataset | ForEach-Object { 
        Write-Host "Deleting dataset " $_.Name
        Remove-AzDataFactoryV2Dataset -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
    }
    Write-Host "Deleting linked services"
    $deletedlinkedservices | ForEach-Object { 
        Write-Host "Deleting Linked Service " $_.Name
        Remove-AzDataFactoryV2LinkedService -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
    }
    Write-Host "Deleting integration runtimes"
    $deletedintegrationruntimes | ForEach-Object { 
        Write-Host "Deleting integration runtime " $_.Name
        Remove-AzDataFactoryV2IntegrationRuntime -Name $_.Name -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Force 
    }

    if ($deleteDeployment -eq $true) {
        Write-Host "Deleting ARM deployment ... under resource group: " $ResourceGroupName
        $deployments = Get-AzResourceGroupDeployment -ResourceGroupName $ResourceGroupName
        $deploymentsToConsider = $deployments | Where { $_.DeploymentName -like "ArmTemplate_master*" -or $_.DeploymentName -like "ArmTemplateForFactory*" } | Sort-Object -Property Timestamp -Descending
        $deploymentName = $deploymentsToConsider[0].DeploymentName

       Write-Host "Deployment to be deleted: " $deploymentName
        $deploymentOperations = Get-AzResourceGroupDeploymentOperation -DeploymentName $deploymentName -ResourceGroupName $ResourceGroupName
        $deploymentsToDelete = $deploymentOperations | Where { $_.properties.targetResource.id -like "*Microsoft.Resources/deployments*" }

        $deploymentsToDelete | ForEach-Object { 
            Write-host "Deleting inner deployment: " $_.properties.targetResource.id
            Remove-AzResourceGroupDeployment -Id $_.properties.targetResource.id
        }
        Write-Host "Deleting deployment: " $deploymentName
        Remove-AzResourceGroupDeployment -ResourceGroupName $ResourceGroupName -Name $deploymentName
    }

    #Start active triggers - after cleanup efforts
    Write-Host "Starting active triggers"
    $triggersToStart | ForEach-Object { 
        if ($_.TriggerType -eq "BlobEventsTrigger") {
            Write-Host "Subscribing" $_.Name "to events"
            $status = Add-AzDataFactoryV2TriggerSubscription -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name
            while ($status.Status -ne "Enabled"){
                Start-Sleep -s 15
                $status = Get-AzDataFactoryV2TriggerSubscriptionStatus -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name
            }
        }
        Write-Host "Starting trigger" $_.Name
        Start-AzDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $_.Name -Force
    }
}
```
