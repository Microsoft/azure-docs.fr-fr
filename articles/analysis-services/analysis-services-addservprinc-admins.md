---
title: Ajouter un principal de service au rôle d’administrateur Azure Analysis Services | Microsoft Docs
description: Découvrez comment ajouter un principal de service d’automatisation au rôle d’administrateur du serveur Azure Analysis Services
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: fasttrack-edit
ms.openlocfilehash: b1d5f8ec628245756c53e4dfeeecdeb3a4bebc2e
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93100726"
---
# <a name="add-a-service-principal-to-the-server-administrator-role"></a>Ajouter un principal de service au rôle d’administrateur du serveur 

 Pour automatiser les tâches PowerShell sans assistance, un principal de service doit disposer des privilèges **Administrateur du serveur** sur le serveur Analysis Services géré. Cet article décrit comment ajouter un principal de service au rôle d’administrateurs du serveur sur un serveur Azure AS. Pour ce faire, vous pouvez utiliser SQL Server Management Studio ou un modèle Resource Manager. 

> [!NOTE]
> Les principaux de service doivent être ajoutés directement au rôle d’administrateur de serveur. L’ajout d’un principal de service à un groupe de sécurité, puis l’ajout de ce groupe de sécurité au rôle d’administrateur de serveur n’est pas pris en charge. 

## <a name="before-you-begin"></a>Avant de commencer
Avant d’exécuter cette tâche, vous devez disposer d’un principal de service inscrit dans Azure Active Directory.

[Créer un principal de service - Portail Azure](../active-directory/develop/howto-create-service-principal-portal.md)   
[Créer un principal de service - PowerShell](../active-directory/develop/howto-authenticate-service-principal-powershell.md)

## <a name="using-sql-server-management-studio"></a>Utilisation de SQL Server Management Studio

Vous pouvez configurer des administrateurs du serveur à l’aide de SQL Server Management Studio (SSMS). Pour effectuer cette tâche, vous devez disposer d’autorisations [Administrateur du serveur](analysis-services-server-admins.md) sur le serveur Azure AS. 

1. Dans SSMS, connectez-vous à votre serveur Azure AS.
2. Dans **Propriétés du serveur** > **Sécurité** , cliquez sur **Ajouter**.
3. Dans **Sélectionnez un utilisateur ou un groupe** , recherchez votre application inscrite par nom, sélectionnez, puis cliquez sur **Ajouter**.

    ![Rechercher le compte de principal de service](./media/analysis-services-addservprinc-admins/aas-add-sp-ssms-picker.png)

4. Vérifiez l’ID de compte de principal de service, puis cliquez sur **OK**.
    
    ![Capture d’écran montrant l’ID de compte de principal de service et mettant en évidence le bouton OK.](./media/analysis-services-addservprinc-admins/aas-add-sp-ssms-add.png)

## <a name="using-a-resource-manager-template"></a>Utilisation d’un modèle Resource Manager

Vous pouvez également configurer des administrateurs du serveur en déployant le serveur Analysis Services à l’aide d’un modèle Azure Resource Manager. L’identité qui exécute le déploiement doit appartenir au rôle **Contributeur** pour la ressource dans le [contrôle d’accès en fonction du rôle (Azure RBAC)](../role-based-access-control/overview.md).

> [!IMPORTANT]
> Le principal du service doit être ajouté au format `app:{service-principal-client-id}@{azure-ad-tenant-id}`.

Le modèle Resource Manager suivant déploie un serveur Analysis Services avec un principal de service spécifié ajouté au rôle Administrateur d’Analysis Services :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "analysisServicesServerName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "analysisServicesSkuName": {
            "type": "string"
        },
        "analysisServicesCapacity": {
            "type": "int"
        },
        "servicePrincipalClientId": {
            "type": "string"
        },
        "servicePrincipalTenantId": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('analysisServicesServerName')]",
            "type": "Microsoft.AnalysisServices/servers",
            "apiVersion": "2017-08-01",
            "location": "[parameters('location')]",
            "sku": {
                "name": "[parameters('analysisServicesSkuName')]",
                "capacity": "[parameters('analysisServicesCapacity')]"
            },
            "properties": {
                "asAdministrators": {
                    "members": [
                        "[concat('app:', parameters('servicePrincipalClientId'), '@', parameters('servicePrincipalTenantId'))]"
                    ]
                }
            }
        }
    ]
}
```

## <a name="using-managed-identities"></a>Utilisation des identités managées

Une identité managée peut également être ajoutée à la liste des administrateurs Analysis Services. Ainsi, vous pouvez avoir [une application logique avec une identité managée affectée par le système](../logic-apps/create-managed-service-identity.md) et souhaiter lui accorder la possibilité d’administrer votre serveur Analysis Services.

Dans pratiquement tout le portail Azure et la majeure partie des API, les identités managées sont identifiées au moyen de leur ID d’objet de principal de service. Pour autant, Analysis Services exige qu’elles soient identifiées à l’aide de leur ID client. Pour obtenir l’ID client d’un principal de service, vous pouvez utiliser l’interface Azure CLI :

```bash
az ad sp show --id <ManagedIdentityServicePrincipalObjectId> --query appId -o tsv
```

Vous pouvez également utiliser PowerShell :

```powershell
(Get-AzureADServicePrincipal -ObjectId <ManagedIdentityServicePrincipalObjectId>).AppId
```

Ensuite, vous utilisez cet ID client conjointement avec l’ID de locataire pour ajouter l’identité managée à la liste des administrateurs Analysis Services, comme décrit ci-dessus.

## <a name="related-information"></a>Informations connexes

* [Télécharger le module SQL Server PowerShell](/sql/ssms/download-sql-server-ps-module)   
* [Télécharger SSMS](/sql/ssms/download-sql-server-management-studio-ssms)