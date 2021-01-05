---
title: Créer un cluster Pivotal Cloud Foundry sur Azure
description: Découvrir comment configurer les paramètres nécessaires au provisionnement d’un cluster Pivotal Cloud Foundry (PCF) sur Azure
services: Cloud Foundry
documentationcenter: CloudFoundry
author: ruyakubu
manager: brunoborges
editor: ruyakubu
ms.assetid: ''
ms.author: ruyakubu
ms.date: 09/13/2018
ms.devlang: ''
ms.service: azure
ms.tgt_pltfrm: multiple
ms.topic: tutorial
ms.workload: web
ms.openlocfilehash: 65d8ade438228d7af71de1fc66639e5b6de2edda
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93040793"
---
# <a name="create-a-pivotal-cloud-foundry-cluster-on-azure"></a>Créer un cluster Pivotal Cloud Foundry sur Azure

Ce didacticiel explique comment créer et générer rapidement les paramètres dont vous avez besoin pour approvisionner un cluster Pivotal Cloud Foundry (PCF) sur Azure. Pour trouver la solution Pivotal Cloud Foundry, effectuez une recherche dans la [Place de marché](https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry) Azure.

![Rechercher Pivotal Cloud Foundry dans Azure](media/deploy/pcf-marketplace.png)


## <a name="generate-an-ssh-public-key"></a>Générer une clé publique SSH

Il existe plusieurs façons de générer une clé Secure Shell (SSH) publique sous Windows, Mac ou Linux.

```Bash
ssh-keygen -t rsa -b 2048
```

Pour plus d’informations, consultez [Utiliser des clés SSH avec Windows sur Azure](../virtual-machines/linux/ssh-from-windows.md).

## <a name="create-a-service-principal"></a>Créer un principal du service

> [!NOTE]
>
> Pour créer un principal du service, vous devez disposer de l’autorisation de compte propriétaire. Vous pouvez également créer un script pour automatiser la création du principal du service. Par exemple, vous pouvez utiliser la commande de l’interface de ligne de commande Azure [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest).

1. Connectez-vous à votre compte Azure.

    `az login`

    ![Connexion à Azure CLI](media/deploy/az-login-output.png )
 
    Copiez la valeur « id » comme **ID d’abonnement** et la valeur « tenantId » à utiliser ultérieurement.

2. Définissez votre abonnement par défaut pour cette configuration.

    `az account set -s {id}`

3. Créez une application Azure Active Directory pour votre PCF. Spécifiez un mot de passe alphanumérique unique. Enregistrez le mot de passe comme étant votre **clientSecret** à utiliser ultérieurement.

    `az ad app create --display-name "Svc Principal for OpsManager" --password {enter-your-password} --homepage "{enter-your-homepage}" --identifier-uris {enter-your-homepage}`

    Copiez la valeur « appId » dans la sortie en tant que **clientID** à utiliser ultérieurement.

    > [!NOTE]
    >
    > Choisissez votre propre page d’accueil d’application et votre URI identificateur, par exemple http\://www\.contoso.com.

4. Créez un principal du service à l’aide de votre nouvel ID d’application.

    `az ad sp create --id {appId}`

5. Définissez le rôle d’autorisation de votre principal du service en tant que Contributeur.

    `az role assignment create --assignee "{enter-your-homepage}" --role "Contributor"`

    Vous pouvez également utiliser

    `az role assignment create --assignee {service-principal-name} --role "Contributor"`

    ![Attribution du rôle de principal du service](media/deploy/svc-princ.png )

6. Vérifiez que vous pouvez vous connecter à votre principal du service à l’aide de l’ID d’application, du mot de passe et de l’ID de locataire.

    `az login --service-principal -u {appId} -p {your-password}  --tenant {tenantId}`

7. Créez un fichier .json au format suivant. Utilisez les valeurs **ID d’abonnement** , **tenantID** , **clientID** et **clientSecret** que vous avez copiées précédemment. Enregistrez le fichier .

    ```json
    {
        "subscriptionID": "{enter-your-subscription-Id-here}",
        "tenantID": "{enter-your-tenant-Id-here}",
        "clientID": "{enter-your-app-Id-here}",
        "clientSecret": "{enter-your-key-here}"
    }
    ```

## <a name="get-the-pivotal-network-token"></a>Obtenir le jeton Pivotal Network

1. Inscrivez-vous ou connectez-vous à votre compte [Pivotal Network](https://network.pivotal.io).
2. Sélectionnez le nom de votre profil dans le coin supérieur droit de la page. Sélectionnez **Modifier le profil**.
3. Faites défiler jusqu’au bas de la page et copiez la valeur **LEGACY API TOKEN**. C’est la valeur de votre **jeton Pivotal Network** qui vous utiliserez ultérieurement.

## <a name="provision-your-cloud-foundry-cluster-on-azure"></a>Approvisionner votre cluster Cloud Foundry sur Azure

Vous disposez maintenant de tous les paramètres dont vous avez besoin pour approvisionner votre [cluster Pivotal Cloud Foundry sur Azure](https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry).
Entrez les paramètres et créez votre cluster PCF.

## <a name="verify-the-deployment-and-sign-in-to-the-pivotal-ops-manager"></a>Vérifier le déploiement et se connecter au Pivotal Ops Manager

1. Votre cluster PCF affiche un état de déploiement.

    ![État de déploiement Azure](media/deploy/deployment.png )

2. Sélectionnez le lien **Déploiements** dans le volet de navigation à gauche pour obtenir des informations d’identification pour votre PCF Ops Manager. Sélectionnez le **nom du déploiement** sur la page suivante.
3. Dans la navigation à gauche, sélectionnez le lien **Sorties** pour afficher l’URL, le nom d’utilisateur et le mot de passe du PCF Ops Manager. La valeur « OPSMAN-FQDN » correspond à l’URL.
 
    ![Sortie du déploiement de Cloud Foundry](media/deploy/deploy-outputs.png )
 
4. Accédez à l’URL dans un navigateur web. Entrez les informations d’identification de l’étape précédente pour vous connecter.

    ![Page de connexion Pivotal](media/deploy/pivotal-login.png )
         
    > [!NOTE]
    >
    > Si le navigateur Internet Explorer échoue à cause d’un message d’avertissement de site non sécurisé, cliquez sur **Plus d’informations** et allez à la page web. Si vous utilisez Firefox, sélectionnez **Advance** et ajoutez la certification pour continuer.

5. Votre PCF Ops Manager affiche les instances Azure déployées. Vous pouvez maintenant déployer et gérer vos applications ici.
               
    ![Instance Azure déployée dans Pivotal](media/deploy/ops-mgr.png )
