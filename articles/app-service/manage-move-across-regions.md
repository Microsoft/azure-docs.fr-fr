---
title: Déplacer une application vers une autre région
description: Découvrez comment déplacer des ressources App Service d’une région à une autre.
ms.topic: how-to
ms.date: 02/27/2020
ms.custom: subject-moving-resources
ms.openlocfilehash: 87a2da31802a8b6858e875c23ef1dbd2d6d006bc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86524852"
---
# <a name="move-an-app-service-resource-to-another-region"></a>Déplacer une ressource App Service vers une autre région

Cet article décrit comment déplacer vos ressources App Service vers une autre région Azure. Vous pouvez être amené à déplacer vos ressources vers une autre région pour diverses raisons. Par exemple, pour tirer parti d’une nouvelle région Azure, pour déployer des fonctionnalités ou des services disponibles dans des régions spécifiques uniquement, pour répondre à des exigences de stratégie et de gouvernance internes ou pour respecter des exigences de planification de la capacité.

Les ressources App Service sont spécifiques à une région et ne peuvent pas être déplacées d’une région à l’autre. Vous devez créer une copie de vos ressources App Service existantes dans la région cible, puis déplacer votre contenu vers la nouvelle application. Si votre application source utilise un domaine personnalisé, vous pouvez [le migrer vers la nouvelle application dans la région cible](manage-custom-dns-migrate-domain.md) quand vous avez terminé.

Pour faciliter la copie de votre application, vous pouvez [cloner une application App Service individuelle](app-service-web-app-cloning.md) dans le plan App Service d’une autre région, mais cette méthode a ses [limites](app-service-web-app-cloning.md#current-restrictions), notamment parce qu’elle ne prend pas en charge les applications Linux.

## <a name="prerequisites"></a>Prérequis

- Vérifiez que l’application App Service se trouve dans la région Azure à partir de laquelle vous souhaitez effectuer le déplacement.
- Assurez-vous que la région cible prend en charge App Service et tout service associé dont vous souhaitez déplacer les ressources.
<!-- - Domain bindings, certificates, and managed identities can't replicated using the **Export template** method. You must create them manually. -->

## <a name="prepare"></a>Préparation

Identifiez toutes les ressources App Service que vous utilisez actuellement. Par exemple :

- Applications App Service
- [Plans App Service](overview-hosting-plans.md)
- [Emplacements de déploiement](deploy-staging-slots.md)
- [Domaines personnalisés achetés dans Azure](manage-custom-dns-buy-domain.md)
- [Certificats SSL](configure-ssl-certificate.md)
- [Intégration de Réseau virtuel Microsoft Azure](web-sites-integrate-with-vnet.md)
- [Connexions hybrides](app-service-hybrid-connections.md)
- [Identités managées](overview-managed-identity.md)
- [Paramètres de sauvegarde](manage-backup.md)

Certaines ressources, telles que les certificats importés ou les connexions hybrides, contiennent une intégration avec d’autres services Azure. Pour plus d’informations sur la façon de déplacer ces ressources d’une région à l’autre, consultez la documentation des services respectifs.

## <a name="move"></a>Déplacer

1. [Créez une sauvegarde de l’application source](manage-backup.md).
1. [Créez une application dans un nouveau plan App Service, dans la région cible](app-service-plan-manage.md#create-an-app-service-plan).
2. [Restaurez la sauvegarde dans l’application cible](web-sites-restore.md).
2. Si vous utilisez un domaine personnalisé, [liez-le de manière préventive à l’application cible](manage-custom-dns-migrate-domain.md#bind-the-domain-name-preemptively) avec `awverify.` et [activez le domaine dans l’application cible](manage-custom-dns-migrate-domain.md#enable-the-domain-for-your-app).
3. Configurez tout le reste dans votre application cible pour qu’elle soit identique à l’application source et vérifiez votre configuration.
4. Lorsque vous êtes prêt pour que le domaine personnalisé soit dirigé vers l’application cible, [remappez le nom de domaine](manage-custom-dns-migrate-domain.md#remap-the-active-dns-name).

<!-- 1. Login to the [Azure portal](https://portal.azure.com) > **Resource Groups**.
2. Locate the Resource Group that contains the source App Service resources and click on it.
3. Select > **Settings** > **Export template**.
4. Choose **Deploy** in the **Export template** blade.
5. Click **TEMPLATE** > **Edit template** to open the template in the online editor.
6. Click inside the online editor and type Ctrl+F (or ⌘+F on a Mac) and type `"identity": {` to find any managed identity definition. The following is an example if you have a user-assigned managed identity.
    ```json
    "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
            "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/<group-name>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<identity-name>": {
                "principalId": "00000000-0000-0000-0000-000000000000",
                "clientId": "00000000-0000-0000-0000-000000000000"
            }
        }
    },
    ```
6. Click inside the online editor and type Ctrl+F (or ⌘+F on a Mac) and type `"Microsoft.Web/sites/hostNameBindings` to find all hostname bindings. The following is an example if you have a user-assigned managed identity.
    ```json
    {
        "type": "Microsoft.Web/sites/hostNameBindings",
        "apiVersion": "2018-11-01",
        "name": "[concat(parameters('sites_webapp_name'), '/', parameters('sites_webapp_name'), '.azurewebsites.net')]",
        "location": "West Europe",
        "dependsOn": [
            "[resourceId('Microsoft.Web/sites', parameters('sites_webapp_name'))]"
        ],
        "properties": {
            "siteName": "<app-name>",
            "hostNameType": "Verified"
        }
    },
    ```
6. Click inside the online editor and type Ctrl+F (or ⌘+F on a Mac) and type `"Microsoft.Web/certificates` to find all hostname bindings. The following is an example if you have a user-assigned managed identity.
    ```json
    {
        "type": "Microsoft.Web/certificates",
        "apiVersion": "2018-11-01",
        "name": "[parameters('certificates_test2_cephaslin_com_name')]",
        "location": "West Europe",
        "properties": {
            "hostNames": [
                "[parameters('certificates_test2_cephaslin_com_name')]"
            ],
            "password": "[parameters('certificates_test2_cephaslin_com_password')]"
        }
    },
    ```
7. Delete the entire JSON block. Click **Save** in the online editor.
8. Click **BASICS** > **Create new** to create a new resource group. Type the group name and click **OK**.
9. In **BASICS** > **Location**, select the region you want.   -->

## <a name="clean-up-source-resources"></a>Nettoyer les ressources sources

Supprimez l’application source et le plan App Service. [Un plan App Service appartenant à un niveau tarifaire non gratuit comporte des frais, même si aucune application ne s’exécute dans celui-ci.](app-service-plan-manage.md#delete-an-app-service-plan)

## <a name="next-steps"></a>Étapes suivantes

[Clonage de l’application Azure App Service à l’aide de PowerShell](app-service-web-app-cloning.md)