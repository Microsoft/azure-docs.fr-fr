---
title: Configurer votre environnement de développement Azure Red Hat OpenShift
description: Voici les prérequis pour travailler avec Microsoft Azure Red Hat OpenShift.
keywords: configurer red hat openshift configuration
author: jimzim
ms.author: jzim
ms.date: 11/04/2019
ms.topic: conceptual
ms.service: container-service
ms.custom: devx-track-azurecli
ms.openlocfilehash: c189c0902e694dc49d81a48433e3269e9f2a438c
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92216876"
---
# <a name="set-up-your-azure-red-hat-openshift-dev-environment"></a>Configurer votre environnement de développement Azure Red Hat OpenShift

> [!IMPORTANT]
> Azure Red Hat OpenShift 3.11 sera mis hors service le 30 juin 2022. La prise en charge de la création de nouveaux clusters Azure Red Hat OpenShift 3.11 se poursuit jusqu'au 30 novembre 2020. Après la mise hors service, les clusters Azure Red Hat OpenShift 3.11 restants seront arrêtés pour éviter des failles de sécurité.
> 
> Suivez ce guide pour [créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md).
> Si vous avez des questions, [n'hésitez pas à nous contacter](mailto:arofeedback@microsoft.com).

Pour générer et exécuter des applications Microsoft Azure Red Hat OpenShift, vous devrez :

* Installer la version 2.0.65 (ou ultérieure) d’Azure CLI (ou utiliser Azure Cloud Shell).
* Vous inscrire pour obtenir la fonctionnalité `AROGA` et les fournisseurs de ressources associés.
* Créer un locataire Azure Active Directory (Azure AD).
* Créer un objet d’application Azure AD.
* Créer un utilisateur Azure AD.

Les instructions suivantes vous guideront tout au long de ces prérequis.

## <a name="install-the-azure-cli"></a>Installer l’interface de ligne de commande Microsoft Azure

Azure Red Hat OpenShift requiert la version 2.0.65 ou une version ultérieure d’Azure CLI. Si vous avez déjà installé Azure CLI, vous pouvez vérifier la version que vous avez en exécutant :

```azurecli
az --version
```

La première ligne de la sortie contient la version CLI, par exemple `azure-cli (2.0.65)`.

Voici les instructions pour [installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest) si une nouvelle installation ou une mise à niveau est nécessaire.

Vous pouvez également utiliser [Azure Cloud Shell](../cloud-shell/overview.md). Lorsque vous utilisez Azure Cloud Shell, veillez à sélectionner l’environnement **Bash** si vous envisagez de suivre la série de tutoriels [Créer et gérer un cluster Azure Red Hat OpenShift](tutorial-create-cluster.md).

## <a name="register-providers-and-features"></a>Inscrire des fournisseurs et des fonctionnalités

La fonctionnalité `Microsoft.ContainerService AROGA`, les fournisseurs `Microsoft.Solutions`, `Microsoft.Compute`, `Microsoft.Storage`, `Microsoft.KeyVault` et `Microsoft.Network` doivent être inscrits à votre abonnement manuellement avant le déploiement de votre premier cluster Azure Red Hat OpenShift.

Pour enregistrer manuellement ces fonctionnalités et fournisseurs, utilisez les instructions suivantes à partir d’un shell Bash si vous avez installé l’interface CLI, ou à partir de la session Azure Cloud Shell (Bash) dans votre portail Azure :

1. Si vous possédez plusieurs abonnements Azure, indiquez l’ID d’abonnement pertinent :

    ```azurecli
    az account set --subscription <SUBSCRIPTION ID>
    ```

1. Inscrire la fonctionnalité AROGA Microsoft.ContainerService :

    ```azurecli
    az feature register --namespace Microsoft.ContainerService -n AROGA
    ```

1. Inscrire le fournisseur Microsoft.Storage :

    ```azurecli
    az provider register -n Microsoft.Storage --wait
    ```
    
1. Inscrire le fournisseur Microsoft.Compute :

    ```azurecli
    az provider register -n Microsoft.Compute --wait
    ```

1. Inscrire le fournisseur Microsoft.Solutions :

    ```azurecli
    az provider register -n Microsoft.Solutions --wait
    ```

1. Inscrire le fournisseur Microsoft.Network :

    ```azurecli
    az provider register -n Microsoft.Network --wait
    ```

1. Inscrire le fournisseur Microsoft.KeyVault :

    ```azurecli
    az provider register -n Microsoft.KeyVault --wait
    ```

1. Actualisez l’inscription du fournisseur de ressources Microsoft.ContainerService :

    ```azurecli
    az provider register -n Microsoft.ContainerService --wait
    ```

## <a name="create-an-azure-active-directory-azure-ad-tenant"></a>Créer un locataire Azure Active Directory (Azure AD)

Le service Azure Red Hat OpenShift nécessite un locataire Azure Active Directory (Azure AD) associé qui représente votre organisation et sa relation à Microsoft. Votre locataire Azure AD vous permet d’enregistrer, de générer et de gérer les applications, et d’utiliser d’autres services Azure.

Si vous n’avez pas d’Azure AD à utiliser en tant que le locataire pour votre cluster Azure Red Hat OpenShift, ou si vous souhaitez créer un locataire de test, suivez les instructions dans [Créer un locataire Azure AD pour votre cluster Azure Red Hat OpenShift](howto-create-tenant.md) avant continuer avec ce guide.

## <a name="create-an-azure-ad-user-security-group-and-application-object"></a>Créer un utilisateur, un groupe de sécurité et un objet application Azure AD

Azure Red Hat OpenShift nécessite des autorisations pour effectuer des tâches sur votre cluster, comme la configuration du stockage. Ces autorisations sont représentées par un [principal du service](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object). Vous devez également créer un nouvel utilisateur Active Directory pour tester les applications s’exécutant sur votre cluster Azure Red Hat OpenShift.

Suivez les instructions dans [Créer un objet application et un utilisateur Azure AD](howto-aad-app-configuration.md) pour créer un principal du service, générer un secret client et une URL de rappel d’authentification pour votre application, et créez un nouveau groupe de sécurité et un nouvel utilisateur Azure AD pour accéder au cluster.

## <a name="next-steps"></a>Étapes suivantes

Vous êtes maintenant prêt à utiliser Azure Red Hat OpenShift !

Essayez le tutoriel :
> [!div class="nextstepaction"]
> [Créer un cluster Azure Red Hat OpenShift](tutorial-create-cluster.md)

[azure-cli-install]: /cli/azure/install-azure-cli