---
title: Azure Red Hat OpenShift exécutant OpenShift 4 - Configurer l’authentification Azure Active Directory à l’aide du portail Azure et de la console web OpenShift
description: Découvrez comment configurer l’authentification Azure Active Directory pour un cluster Azure Red Hat OpenShift exécutant OpenShift 4 à l’aide du portail Azure et de la console web OpenShift
ms.service: container-service
ms.topic: article
ms.date: 03/12/2020
author: sabbour
ms.author: asabbour
keywords: aro, openshift, az aro, red hat, cli
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 1b9e4d1f1b989caa317384292d013af255530f11
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92748077"
---
# <a name="configure-azure-active-directory-authentication-for-an-azure-red-hat-openshift-4-cluster-portal"></a>Configurer l’authentification Azure Active Directory pour un cluster Azure Red Hat OpenShift 4 (portail)

Si vous choisissez d’installer et d’utiliser l’interface CLI localement, ce tutoriel demande au minimum la version 2.6.0 d’Azure CLI. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="before-you-begin"></a>Avant de commencer

Construisez l’ **URL de rappel OAuth** du cluster et notez-la. Veillez à remplacer **aro-rg** par le nom de votre groupe de ressources et **aro-cluster** par le nom de votre cluster.

> [!NOTE]
> La section `AAD` de l’URL de rappel OAuth doit correspondre au nom du fournisseur d’identité OAuth que vous allez configurer ultérieurement.

```azurecli-interactive
domain=$(az aro show -g aro-rg -n aro-cluster --query clusterProfile.domain -o tsv)
location=$(az aro show -g aro-rg -n aro-cluster --query location -o tsv)
echo "OAuth callback URL: https://oauth-openshift.apps.$domain.$location.aroapp.io/oauth2callback/AAD"
```

## <a name="create-an-azure-active-directory-application-for-authentication"></a>Créer une application Azure Active Directory pour l’authentification

Connectez-vous au portail Azure et accédez au [panneau Inscriptions d’applications](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade), puis cliquez sur **Nouvelle inscription** pour créer une application.

Fournissez un nom pour l’application, par exemple **aro-azuread-auth** et renseignez l’ **URI de redirection** à l’aide de la valeur de l’URL de rappel OAuth que vous avez récupérée.

![Nouvelle inscription d’application](media/aro4-ad-registerapp.png)

Accédez à **Certificats et secrets** , puis cliquez sur **Nouveau secret client** et renseignez les détails. Notez la valeur de la clé, car vous l’utiliserez dans une étape ultérieure. Vous ne pourrez plus la récupérer une nouvelle fois.

![Créer un secret](media/aro4-ad-clientsecret.png)

Accédez à la **vue d’ensemble** et notez l’ **ID d’application (client)** et l’ **ID de l’annuaire (locataire)** . Vous en aurez besoin à une phase ultérieure.

![Récupérer les ID d’application (client) et d’annuaire (locataire)](media/aro4-ad-ids.png)

## <a name="configure-optional-claims"></a>Configurer des revendications facultatives

Les développeurs d’applications peuvent utiliser des [revendications facultatives](../active-directory/develop/active-directory-optional-claims.md) dans leurs applications Azure AD pour spécifier les revendications souhaitées dans les jetons envoyés à leur application.

Vous pouvez utiliser des revendications facultatives pour :

* Sélectionner des revendications supplémentaires à inclure dans les jetons pour votre application.
* Modifier le comportement de certaines revendications retournées par Azure AD dans les jetons.
* Ajouter et accéder à des revendications personnalisées pour votre application.

Nous allons configurer OpenShift pour qu’il utilise la revendication `email` et revenir à `upn` pour définir le nom d’utilisateur par défaut en ajoutant l’`upn` dans le cadre du jeton d’ID retourné par Azure Active Directory.

Accédez à **Configuration du jeton (préversion)** puis cliquez sur **Ajouter une revendication facultative** . Sélectionnez **ID** , puis vérifiez les revendications **email** et **upn** .

![Capture d’écran montrant les revendications email et upn qui ont été ajoutées.](media/aro4-ad-tokens.png)

## <a name="assign-users-and-groups-to-the-cluster-optional"></a>Affecter des utilisateurs et des groupes au cluster (facultatif)

Les applications inscrites dans un locataire Azure Active Directory (Azure AD) sont, par défaut, disponibles pour tous les utilisateurs du locataire qui parviennent à s’authentifier. Azure AD permet aux développeurs et administrateurs de locataires de limiter une application à un ensemble spécifique d’utilisateurs ou de groupes de sécurité dans le locataire.

Suivez les instructions de la documentation d’Azure Active Directory pour [affecter des utilisateurs et des groupes à l’application](../active-directory/develop/howto-restrict-your-app-to-a-set-of-users.md#app-registration).

## <a name="configure-openshift-openid-authentication"></a>Configurer l’authentification OpenShift OpenID

Récupérez les informations d’identification de `kubeadmin`. Exécutez la commande suivante pour rechercher le mot de passe de l’utilisateur `kubeadmin`.

```azurecli-interactive
az aro list-credentials \
  --name aro-cluster \
  --resource-group aro-rg
```

L’exemple de sortie suivant montre que le mot de passe se trouve dans `kubeadminPassword`.

```json
{
  "kubeadminPassword": "<generated password>",
  "kubeadminUsername": "kubeadmin"
}
```

Vous trouverez l’URL de la console de cluster en exécutant la commande suivante, qui se présente comme suit : `https://console-openshift-console.apps.<random>.<region>.aroapp.io/`

```azurecli-interactive
 az aro show \
    --name aro-cluster \
    --resource-group aro-rg \
    --query "consoleProfile.url" -o tsv
```

Lancez l’URL de la console dans un navigateur et connectez-vous à l’aide des informations d’identification `kubeadmin`.

Accédez à **Administration** , cliquez sur **Paramètres de cluster** , puis sélectionnez l’onglet **Configuration globale** . Faites défiler pour sélectionner **OAuth** .

Faites défiler vers le bas jusqu’à la section **Fournisseurs d’identité** , sélectionnez **Ajouter** , puis sélectionnez **OpenID Connect** .
![Sélectionner OpenID Connect dans la liste déroulante Fournisseurs d’identité](media/aro4-oauth-idpdrop.png)

Renseignez le nom en indiquant **AAD** , l’ **ID client** en indiquant l’ **ID d’application** et le **Secret client** . L’ **URL de l’émetteur** se présente sous la forme suivante : `https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. Remplacez l’espace réservé par l’ID de locataire que vous avez récupéré.

![Renseigner les détails OAuth](media/aro4-oauth-idp-1.png)

Faites défiler vers le bas jusqu’à la section **Revendications** et mettez à jour le **Nom d’utilisateur par défaut** pour utiliser la valeur de la revendication **upn** .

![Renseigner les détails des revendications](media/aro4-oauth-idp-2.png)

## <a name="verify-login-through-azure-active-directory"></a>Vérifier la connexion via Azure Active Directory

Si vous vous déconnectez maintenant de la console web OpenShift et que vous tentez de vous reconnecter, une nouvelle option s’affiche pour vous permettre de vous connecter avec **AAD** . Vous devrez peut-être patienter quelques minutes.

![Écran de connexion avec l’option Azure Active Directory](media/aro4-login-2.png)
