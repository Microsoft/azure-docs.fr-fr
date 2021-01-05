---
title: Authentifier la sortie blob avec Managed Identity Azure Stream Analytics
description: Cet article explique comment utiliser des identités managées afin d’authentifier votre tâche Azure Stream Analytics pour la sortie de stockage Blob Azure.
author: cedarbaum
ms.author: sacedarb
ms.service: stream-analytics
ms.topic: how-to
ms.date: 03/11/2020
ms.openlocfilehash: f1aad2464e4979ecbf35deceaad4de70e05b3004
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96573661"
---
# <a name="use-managed-identity-preview-to-authenticate-your-azure-stream-analytics-job-to-azure-blob-storage-output"></a>Utiliser Identité managée (préversion) pour authentifier votre travail Azure Stream Analytics avec la sortie Stockage Blob Azure

L’[authentification par identité managée](../active-directory/managed-identities-azure-resources/overview.md) (préversion) pour la sortie Stockage Blob Azure donne aux travaux Stream Analytics un accès direct à un compte de stockage au lieu d’utiliser une chaîne de connexion. Outre la sécurité améliorée, cette fonctionnalité vous permet également d’écrire des données dans un compte de stockage dans un réseau virtuel (VNET) au sein d’Azure.

Cet article vous montre comment activer une identité managée pour une ou des sorties Blob d’un travail Azure Stream Analytics via le portail Azure et un déploiement d’Azure Resource Manager.

## <a name="create-the-stream-analytics-job-using-the-azure-portal"></a>Créer le travail Stream Analytics à l’aide du portail Azure

1. Créez un travail Stream Analytics ou ouvrez un travail existant dans le portail Azure. Dans la barre de menus située sur le côté gauche de l’écran, sélectionnez **Identité managée** sous **Configurer**. Vérifiez que l’option « Utiliser l'identité managée affectée par le système » est sélectionnée, puis cliquez sur le bouton **Enregistrer** en bas de l’écran.

   ![Configurer une identité managée Stream Analytics](./media/common/stream-analytics-enable-managed-identity.png)

2. Dans la fenêtre de propriétés de sortie du récepteur de sortie de stockage Blob Azure, sélectionnez la liste déroulante Mode d’authentification, puis **Identité managée**. Pour plus d’informations sur les autres propriétés de sortie, consultez [Comprendre les sorties d’Azure Stream Analytics](./stream-analytics-define-outputs.md). Quand vous avez terminé, cliquez sur **Enregistrer**.

   ![Configurer la sortie de stockage Blob Azure](./media/stream-analytics-managed-identities-blob-output-preview/stream-analytics-blob-output-blade.png)

3. Maintenant que le travail est créé, consultez la section [Permettre au travail Stream Analytics d’accéder à votre compte de stockage](#give-the-stream-analytics-job-access-to-your-storage-account) de cet article.

## <a name="azure-resource-manager-deployment"></a>Déploiement Azure Resource Manager

L’utilisation d’Azure Resource Manager vous permet d’automatiser entièrement le déploiement de votre travail Stream Analytics. Vous pouvez déployer des modèles Resource Manager à l’aide d’Azure PowerShell ou de l’[interface de ligne de commande Azure](/cli/azure/). Les exemples suivants utilisent l’interface de ligne de commande Azure.


1. Vous pouvez créer une ressource **Microsoft.StreamAnalytics/streamingjobs** avec une identité managée en incluant la propriété suivante dans la section ressources de votre modèle Resource Manager :

    ```json
    "Identity": {
      "Type": "SystemAssigned",
    },
    ```

   Cette propriété indique à Azure Resource Manager de créer et manager l’identité de votre travail Stream Analytics. Voici un exemple de modèle Resource Manager qui déploie un travail Stream Analytics avec l’identité managée activée et un récepteur de sortie Blob qui utilise l’identité managée :

    ```json
    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
            {
                "apiVersion": "2017-04-01-preview",
                "name": "MyStreamingJob",
                "location": "[resourceGroup().location]",
                "type": "Microsoft.StreamAnalytics/StreamingJobs",
                "identity": {
                    "type": "systemAssigned"
                },
                "properties": {
                    "sku": {
                        "name": "standard"
                    },
                    "outputs":[
                        {
                            "name":"output",
                            "properties":{
                                "serialization": {
                                    "type": "JSON",
                                    "properties": {
                                        "encoding": "UTF8"
                                    }
                                },
                                "datasource":{
                                    "type":"Microsoft.Storage/Blob",
                                    "properties":{
                                        "storageAccounts": [
                                            { "accountName": "MyStorageAccount" }
                                        ],
                                        "container": "test",
                                        "pathPattern": "segment1/{date}/segment2/{time}",
                                        "dateFormat": "yyyy/MM/dd",
                                        "timeFormat": "HH",
                                        "authenticationMode": "Msi"
                                    }
                                }
                            }
                        }
                    ]
                }
            }
        ]
    }
    ```

    Le travail ci-dessus doit être déployé sur le groupe de ressources **ExampleGroup** à l’aide de l’interface de ligne de commande Azure ci-dessous :

    ```azurecli
    az group deployment create --resource-group ExampleGroup -template-file StreamingJob.json
    ```

2. Une fois le travail créé, vous pouvez utiliser Azure Resource Manager pour récupérer la définition complète du travail.

    ```azurecli
    az resource show --ids /subscriptions/{SUBSCRIPTION_ID}/resourceGroups/{RESOURCE_GROUP}/providers/Microsoft.StreamAnalytics/StreamingJobs/{RESOURCE_NAME}
    ```

    La commande ci-dessus retourne une réponse similaire à ce qui suit :

    ```json
    {
        "id": "/subscriptions/{SUBSCRIPTION_ID}/resourceGroups/{RESOURCE_GROUP}/providers/Microsoft.StreamAnalytics/streamingjobs/{RESOURCE_NAME}",
        "identity": {
            "principalId": "{PRINCIPAL_ID}",
            "tenantId": "{TENANT_ID}",
            "type": "SystemAssigned",
            "userAssignedIdentities": null
        },
        "kind": null,
        "location": "West US",
        "managedBy": null,
        "name": "{RESOURCE_NAME}",
        "plan": null,
        "properties": {
            "compatibilityLevel": "1.0",
            "createdDate": "2019-07-12T03:11:30.39Z",
            "dataLocale": "en-US",
            "eventsLateArrivalMaxDelayInSeconds": 5,
            "jobId": "{JOB_ID}",
            "jobState": "Created",
            "jobStorageAccount": null,
            "jobType": "Cloud",
            "outputErrorPolicy": "Stop",
            "package": null,
            "provisioningState": "Succeeded",
            "sku": {
                "name": "Standard"
            }
        },
        "resourceGroup": "{RESOURCE_GROUP}",
        "sku": null,
        "tags": null,
        "type": "Microsoft.StreamAnalytics/streamingjobs"
    }
    ```

   Notez le **principalId** à partir de la définition du travail, qui identifie l’identité gérée de votre travail dans Azure Active Directory et sera utilisé à l’étape suivante pour autoriser le travail Stream Analytics à accéder au compte de stockage.

3. Maintenant que le travail est créé, consultez la section [Permettre au travail Stream Analytics d’accéder à votre compte de stockage](#give-the-stream-analytics-job-access-to-your-storage-account) de cet article.


## <a name="give-the-stream-analytics-job-access-to-your-storage-account"></a>Autorisez le travail Stream Analytics à accéder à votre compte de stockage

Il existe deux niveaux d’accès que vous pouvez choisir pour votre travail Stream Analytics :

1. **Accès au niveau du conteneur :** cette option autorise le travail à accéder à un conteneur existant spécifique.
2. **Accès au niveau du compte :** cette option donne au travail un accès général au compte de stockage, notamment la possibilité de créer des conteneurs.

À moins d’avoir besoin du travail pour créer des conteneurs en votre nom, vous devriez choisir l’**accès au niveau du conteneur**, car cette option donne au travail le niveau d’accès minimal requis. Les deux options sont expliquées plus bas pour le Portail Azure et la ligne de commande.

### <a name="grant-access-via-the-azure-portal"></a>Autoriser l’accès via le portail azure

#### <a name="container-level-access"></a>Accès au niveau du conteneur

1. Accédez au volet de configuration du conteneur au sein de votre compte de stockage.

2. Sélectionnez **Contrôle d’accès (IAM)** sur le côté gauche.

3. Dans la section « Ajouter une attribution de rôle » , cliquez sur **Ajouter**.

4. Dans le volet d’attribution de rôle :

    1. Définissez le **Rôle** sur « Contributeur aux données Blob du stockage »
    2. Vérifiez que la liste déroulante **Attribuer l’accès à** est définie sur « Utilisateur, groupe ou principal de service Azure AD ».
    3. Tapez le nom de votre travail Stream Analytics dans le champ de recherche.
    4. Sélectionnez votre travail Stream Analytics, puis cliquez sur **Enregistrer**.

   ![Accorder l’accès au conteneur](./media/stream-analytics-managed-identities-blob-output-preview/stream-analytics-container-access-portal.png)

#### <a name="account-level-access"></a>Accès au niveau du compte

1. Accédez à votre compte de stockage.

2. Sélectionnez **Contrôle d’accès (IAM)** sur le côté gauche.

3. Dans la section « Ajouter une attribution de rôle » , cliquez sur **Ajouter**.

4. Dans le volet d’attribution de rôle :

    1. Définissez le **Rôle** sur « Contributeur aux données Blob du stockage »
    2. Vérifiez que la liste déroulante **Attribuer l’accès à** est définie sur « Utilisateur, groupe ou principal de service Azure AD ».
    3. Tapez le nom de votre travail Stream Analytics dans le champ de recherche.
    4. Sélectionnez votre travail Stream Analytics, puis cliquez sur **Enregistrer**.

   ![Autoriser l’accès au compte](./media/stream-analytics-managed-identities-blob-output-preview/stream-analytics-account-access-portal.png)

### <a name="grant-access-via-the-command-line"></a>Autoriser l’accès via la ligne de commande

#### <a name="container-level-access"></a>Accès au niveau du conteneur

Pour autoriser l’accès à un conteneur spécifique, exécutez la commande suivante à l’aide de l’interface de ligne de commande Azure :

   ```azurecli
   az role assignment create --role "Storage Blob Data Contributor" --assignee <principal-id> --scope /subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/blobServices/default/containers/<container-name>
   ```

#### <a name="account-level-access"></a>Accès au niveau du compte

Pour autoriser l’accès à la totalité du compte, exécutez la commande suivante à l’aide de l’interface de ligne de commande Azure :

   ```azurecli
   az role assignment create --role "Storage Blob Data Contributor" --assignee <principal-id> --scope /subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>
   ```

## <a name="enable-vnet-access"></a>Activer l’accès au réseau virtuel

Lorsque vous configurez les **pare-feu et les réseaux virtuels** de votre compte de stockage, vous pouvez éventuellement autoriser le trafic réseau à partir d’autres services Microsoft approuvés. Lorsque Stream Analytics s’authentifie à l’aide de l’identité managée, il fournit la preuve que la demande provient d’un service approuvé. Vous trouverez ci-dessous des instructions pour activer cette exception d’accès au réseau virtuel.

1.    Accédez au volet « Pare-feu et réseaux virtuels » dans le volet de configuration du compte de stockage.
2.    Vérifiez que l’option « Autoriser les services Microsoft approuvés à accéder à ce compte de stockage » est activée.
3.    Si vous l’avez activée, cliquez sur **Enregistrer**.

   ![Activer l’accès au réseau virtuel](./media/stream-analytics-managed-identities-blob-output-preview/stream-analytics-vnet-exception.png)

## <a name="remove-managed-identity"></a>Supprimer l’identité managée

L’identité gérée créée pour une tâche Stream Analytics est supprimée uniquement lorsque la tâche est supprimée. Il n’existe aucun moyen de supprimer l’identité gérée sans supprimer la tâche. Si vous ne souhaitez plus utiliser l’identité gérée, vous pouvez modifier la méthode d’authentification pour la sortie. L’identité gérée continuera d’exister jusqu’à ce que la tâche soit supprimée, et sera réemployée si vous décidez à nouveau d’utiliser l’authentification d’identité gérée.

## <a name="limitations"></a>Limites
Vous trouverez plus bas les limitations actuelles de cette fonctionnalité :

1. Comptes Stockage Azure classiques.

2. Comptes Azure sans Azure Active Directory.

3. L’accès multi-locataire n’est pas pris en charge. Le principal de service créé pour un travail Stream Analytics donné doit résider dans le même locataire Azure Active Directory que celui dans lequel le travail a été créé. Il ne peut pas être utilisé avec une ressource qui réside sur un autre locataire Azure Active Directory.

4. Une [Identité attribuée par l’utilisateur](../active-directory/managed-identities-azure-resources/overview.md) n’est pas prise en charge. Cela signifie que l’utilisateur n’est pas en mesure d’entrer dans son propre principal de service à utiliser par son travail Stream Analytics. Le principal de service doit être généré par Azure Stream Analytics.

## <a name="next-steps"></a>Étapes suivantes

* [Comprendre les sorties d’Azure Stream Analytics](./stream-analytics-define-outputs.md)
* [Partitionnement personnalisé de sortie Blob dans Azure Stream Analytics](./stream-analytics-custom-path-patterns-blob-storage-output.md)