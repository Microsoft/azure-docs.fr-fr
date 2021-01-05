---
title: Utiliser une identité managée pour authentifier votre travail Azure Stream Analytics pour la sortie vers Power BI
description: Cet article explique comment utiliser des identités managées pour authentifier votre tâche Azure Stream Analytics vers la sortie Power BI.
author: cedarbaum
ms.author: sacedarb
ms.service: stream-analytics
ms.topic: how-to
ms.date: 3/10/2020
ms.openlocfilehash: 6aea427848afd39b9b9ca166856470a2f9b88694
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97705096"
---
# <a name="use-managed-identity-to-authenticate-your-azure-stream-analytics-job-to-power-bi-preview"></a>Utiliser une identité managée pour authentifier votre travail Azure Stream Analytics vers Power BI (préversion)

L’[Authentification d’une identité managée](../active-directory/managed-identities-azure-resources/overview.md) pour la sortie vers Power BI donne aux travaux Stream Analytics un accès direct à un espace de travail au sein de votre compte Power BI. Cette fonctionnalité permet de déployer des tâches Stream Analytics entièrement automatisées, car il n’est plus nécessaire pour un utilisateur de se connecter de manière interactive à Power BI via le Portail Azure. En outre, les travaux durables qui écrivent dans Power BI sont désormais mieux pris en charge, car vous n’aurez pas besoin de réautoriser régulièrement le travail.

Cet article vous montre comment activer une identité managée pour une ou des sorties Power BI d’un travail Azure Stream Analytics via le Portail Azure et un déploiement d’Azure Resource Manager.

## <a name="prerequisites"></a>Prérequis

Les éléments suivants sont requis pour l’utilisation de cette fonctionnalité :

- Un compte Power BI avec une [licence Pro](/power-bi/service-admin-purchasing-power-bi-pro).

- Un espace de travail mis à niveau dans votre compte Power BI. Voir l’[annonce de cette fonction par Power BI](https://powerbi.microsoft.com/blog/announcing-new-workspace-experience-general-availability-ga/) pour plus d’informations.

## <a name="create-a-stream-analytics-job-using-the-azure-portal"></a>Créer un travail Stream Analytics à l’aide du Portail Azure

1. Créez un travail Stream Analytics ou ouvrez un travail existant dans le portail Azure. Dans la barre de menus située sur le côté gauche de l’écran, sélectionnez **Identité managée** sous **Configurer**. Vérifiez que l’option « Utiliser l’identité managée attribuée par le système » est sélectionnée, puis sélectionnez le bouton **Enregistrer** en bas de l’écran.

   ![Configurer une identité managée Stream Analytics](./media/common/stream-analytics-enable-managed-identity.png)

2. Avant de configurer la sortie, accordez au travail Stream Analytics l’accès à votre espace de travail Power BI en suivant les instructions fournies dans la section [Autoriser le travail Stream Analytics à accéder à votre espace de travail Power BI](#give-the-stream-analytics-job-access-to-your-power-bi-workspace) de cet article.

3. Accédez à la section **Sorties** de votre tâche Stream Analytics, sélectionnez **+ Ajouter**, puis **Power BI**. Ensuite, sélectionnez le bouton **Autoriser** et connectez-vous avec votre compte Power BI.

   ![Autoriser à l’aide d’un compte Power BI](./media/stream-analytics-powerbi-output-managed-identity/stream-analytics-authorize-powerbi.png)

4. Une fois l’autorisation accordée, une liste déroulante est remplie avec tous les espaces de travail auxquels vous avez accès. Sélectionnez l’espace de travail que vous avez autorisé à l’étape précédente. Puis sélectionnez **Identité managée** comme « Mode d’authentification ». Enfin, sélectionnez le bouton **Enregistrer**.

   ![Configurer la sortie Power BI avec une identité managée](./media/stream-analytics-powerbi-output-managed-identity/stream-analytics-configure-powerbi-with-managed-id.png)

## <a name="azure-resource-manager-deployment"></a>Déploiement Azure Resource Manager

Azure Resource Manager vous permet d’automatiser entièrement le déploiement de votre travail Stream Analytics. Vous pouvez déployer des modèles Resource Manager à l’aide d’Azure PowerShell ou de l’[interface de ligne de commande Azure](/cli/azure/). Les exemples suivants utilisent l’interface de ligne de commande Azure.


1. Vous pouvez créer une ressource **Microsoft.StreamAnalytics/streamingjobs** avec une identité managée en incluant la propriété suivante dans la section ressources de votre modèle Resource Manager :

    ```json
    "identity": {
        "type": "SystemAssigned",
    }
    ```

   Cette propriété indique à Azure Resource Manager de créer et manager l’identité de votre travail Stream Analytics. Voici un exemple de modèle Resource Manager qui déploie un travail Stream Analytics avec l’identité managée activée et un récepteur de sortie Power BI qui utilise l’identité managée :

    ```json
    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
            {
                "apiVersion": "2017-04-01-preview",
                "name": "pbi_managed_id",
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
                                "datasource":{
                                    "type":"PowerBI",
                                    "properties":{
                                        "dataset": "dataset_name",
                                        "table": "table_name",
                                        "groupId": "01234567-89ab-cdef-0123-456789abcdef",
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

    Déployez le travail ci-dessus dans le groupe de ressources **ExampleGroup** à l’aide de l’interface de ligne de commande Azure ci-dessous :

    ```azurecli
    az deployment group create --resource-group ExampleGroup -template-file StreamingJob.json
    ```

2. Une fois le travail créé, utilisez Azure Resource Manager pour récupérer la définition complète du travail.

    ```azurecli
    az resource show --ids /subscriptions/<subsription-id>/resourceGroups/<resource-group>/providers/Microsoft.StreamAnalytics/StreamingJobs/<resource-name>
    ```

    La commande ci-dessus retourne une réponse similaire à ce qui suit :

    ```json
    {
        "id": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.StreamAnalytics/streamingjobs/<resource-name>",
        "identity": {
            "principalId": "<principal-id>",
            "tenantId": "<tenant-id>",
            "type": "SystemAssigned",
            "userAssignedIdentities": null
        },
        "kind": null,
        "location": "West US",
        "managedBy": null,
        "name": "<resource-name>",
        "plan": null,
        "properties": {
            "compatibilityLevel": "1.0",
            "createdDate": "2019-07-12T03:11:30.39Z",
            "dataLocale": "en-US",
            "eventsLateArrivalMaxDelayInSeconds": 5,
            "jobId": "<job-id>",
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
        "resourceGroup": "<resource-group>",
        "sku": null,
        "tags": null,
        "type": "Microsoft.StreamAnalytics/streamingjobs"
    }
    ```

    Si vous envisagez d’utiliser l’API REST de Power BI pour ajouter le travail Stream Analytics à votre espace de travail Power BI, notez la valeur « principalId » renvoyée.

3. Maintenant que le travail est créé, passez à la section [Autoriser le travail Stream Analytics à accéder à votre espace de travail Power BI](#give-the-stream-analytics-job-access-to-your-power-bi-workspace) de cet article.


## <a name="give-the-stream-analytics-job-access-to-your-power-bi-workspace"></a>Autoriser le travail Stream Analytics à accéder à votre espace de travail Power BI

À présent que le travail Stream Analytics a été créé, il peut être autorisé à accéder à un espace de travail Power BI. Une fois que vous avez donné l’accès à votre travail, quelques minutes sont nécessaires pour que l’identité se propage.

### <a name="use-the-power-bi-ui"></a>Utiliser l’interface utilisateur Power BI

   > [!Note]
   > Pour ajouter le travail Stream Analytics à votre espace de travail Power BI à l’aide de l’interface utilisateur, vous devez également activer l’accès au principal du service dans les **Paramètres du développeur** dans le portail d’administration Power BI. Pour plus d’informations, consultez [Bien démarrer avec un principal de service](/power-bi/developer/embed-service-principal).

1. Accédez aux paramètres d’accès de l’espace de travail. Consultez cet article pour plus de détails : [Autoriser l’accès à votre espace de travail](/power-bi/service-create-the-new-workspaces#give-access-to-your-workspace).

2. Saisissez le nom de votre travail Stream Analytics dans la zone de texte et sélectionnez **Contributeur** comme niveau d’accès.

3. Sélectionnez **Ajouter** et fermez le volet.

   ![Ajouter un travail Stream Analytics à un espace de travail Power BI](./media/stream-analytics-powerbi-output-managed-identity/stream-analytics-add-job-to-powerbi-workspace.png)

### <a name="use-the-power-bi-powershell-cmdlets"></a>Utiliser les cmdlets PowerShell Power BI

1. Installez les cmdlets PowerShell Power BI `MicrosoftPowerBIMgmt`.

   > [!Important]
   > Vérifiez que vous utilisez la version 1.0.821 ou ultérieure des cmdlets.

```powershell
Install-Module -Name MicrosoftPowerBIMgmt
```

2. Connectez-vous à Power BI.

```powershell
Login-PowerBI
```

3. Ajoutez votre travail Stream Analytics en tant que contributeur à l’espace de travail.

```powershell
Add-PowerBIWorkspaceUser -WorkspaceId <group-id> -PrincipalId <principal-id> -PrincipalType App -AccessRight Contributor
```

### <a name="use-the-power-bi-rest-api"></a>Utiliser l’API REST Power BI

Le travail Stream Analytics peut également être ajouté en tant que Contributeur à l’espace de travail à l’aide de l’API REST « Ajouter un utilisateur de groupe » directement. Vous trouverez la documentation complète de cette API ici : [Groupes – Ajouter un utilisateur de groupe](/rest/api/power-bi/groups/addgroupuser).

**Exemple de demande**
```http
POST https://api.powerbi.com/v1.0/myorg/groups/{groupId}/users
```
Corps de la demande
```json
{
    "groupUserAccessRight": "Contributor",
    "identifier": "<principal-id>",
    "principalType": "App"
}
```

## <a name="remove-managed-identity"></a>Supprimer l’identité managée

L’identité gérée créée pour une tâche Stream Analytics est supprimée uniquement lorsque la tâche est supprimée. Il n’existe aucun moyen de supprimer l’identité gérée sans supprimer la tâche. Si vous ne souhaitez plus utiliser l’identité gérée, vous pouvez modifier la méthode d’authentification pour la sortie. L’identité gérée continuera d’exister jusqu’à ce que la tâche soit supprimée, et sera réemployée si vous décidez à nouveau d’utiliser l’authentification d’identité gérée.

## <a name="limitations"></a>Limites
Vous trouverez plus bas les limitations de cette fonctionnalité :

- Les espaces de travail Power BI classiques ne sont pas pris en charge.

- Comptes Azure sans Azure Active Directory.

- L’accès multi-locataire n’est pas pris en charge. Le principal de service créé pour un travail Stream Analytics donné doit résider dans le même locataire Azure Active Directory que celui dans lequel le travail a été créé. Il ne peut pas être utilisé avec une ressource qui réside sur un autre locataire Azure Active Directory.

- Une [Identité attribuée par l’utilisateur](../active-directory/managed-identities-azure-resources/overview.md) n’est pas prise en charge. Cela signifie que vous n’êtes pas en mesure d’entrer dans votre propre principal de service à utiliser par votre travail Stream Analytics. Le principal de service doit être généré par Azure Stream Analytics.

## <a name="next-steps"></a>Étapes suivantes

* [Intégration de tableau de bord Power BI à Azure Stream Analytics](./stream-analytics-power-bi-dashboard.md)
* [Comprendre les sorties d’Azure Stream Analytics](./stream-analytics-define-outputs.md)