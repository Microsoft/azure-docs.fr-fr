---
title: Configuration d’Azure Active Directory pour l’authentification des clients
description: Découvrez comment configurer Azure Active Directory (Azure AD) pour authentifier les clients des clusters Service Fabric.
ms.topic: conceptual
ms.date: 6/28/2019
ms.openlocfilehash: 537a81a090828d3fcc9dde6032f1d4eb2df9b4e4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86258773"
---
# <a name="set-up-azure-active-directory-for-client-authentication"></a>Configuration d’Azure Active Directory pour l’authentification des clients

Pour les clusters exécutés sur Azure, il est recommandé d’utiliser Azure Active Directory (Azure AD) pour sécuriser l’accès aux points de terminaison de gestion. Cet article explique comment configurer Azure AD afin d’authentifier les clients pour un cluster Service Fabric.

Dans cet article, le terme « application » est utilisé pour faire référence aux [applications Azure Active Directory](../active-directory/develop/developer-glossary.md#client-application), et non aux applications Service Fabric ; la distinction sera faite si nécessaire. Azure AD permet aux organisation (appelées locataires) de gérer l’accès utilisateur aux applications.

Un cluster Service Fabric offre plusieurs points d’entrée pour ses fonctionnalités de gestion, notamment les outils web [Service Fabric Explorer][service-fabric-visualizing-your-cluster]et [Visual Studio][service-fabric-manage-application-in-visual-studio]. Par conséquent, vous allez créer deux applications Azure AD pour contrôler l’accès au cluster : une application web et une application native. Après avoir créé les applications, vous allez affecter les utilisateurs aux rôles en lecture seule et administrateur.

> [!NOTE]
> Sur Linux, vous devez suivre les étapes ci-dessous pour pouvoir créer le cluster. Sur Windows, vous avez également la possibilité de [configurer l’authentification Azure AD pour un cluster existant](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/Security/Configure%20Azure%20Active%20Directory%20Authentication%20for%20Existing%20Cluster.md).

> [!NOTE]
> Le fait que les applications et les nœuds des clusters compatibles avec Azure AAD ne sont pas visibles sur le portail Azure est un [problème connu](https://github.com/microsoft/service-fabric/issues/399).



## <a name="prerequisites"></a>Prérequis
Dans cet article, nous partons du principe que vous avez déjà créé un locataire. Si ce n’est pas le cas, commencez par lire [Guide pratique pour obtenir un locataire Azure Active Directory][active-directory-howto-tenant].

Pour simplifier certaines des étapes impliquées dans la configuration d’Azure AD avec un cluster Service Fabric, nous avons créé un ensemble de scripts Windows PowerShell.

1. [Clonez le référentiel](https://github.com/Azure-Samples/service-fabric-aad-helpers) sur votre ordinateur.
2. [Assurez-vous d’avoir réuni toutes les conditions préalables](https://github.com/Azure-Samples/service-fabric-aad-helpers#getting-started) pour les scripts installés.

## <a name="create-azure-ad-applications-and-assign-users-to-roles"></a>Créer des applications Azure AD et attribuer des rôles aux utilisateurs

Nous allons utiliser les scripts pour créer deux applications Azure AD afin de contrôler l’accès au cluster : une application web et une application native. Une fois que vous avez créé des applications pour représenter votre cluster, vous allez créer des utilisateurs pour les [rôles pris en charge par Service Fabric](service-fabric-cluster-security-roles.md) : lecture seule et administrateur.

Exécutez `SetupApplications.ps1`, puis entrez l'ID du locataire, le nom du cluster et l'URL de réponse de l'application web en tant que paramètres.  Spécifiez également les noms d'utilisateur et les mots de passe des utilisateurs. Par exemple :

```powershell
$Configobj = .\SetupApplications.ps1 -TenantId '0e3d2646-78b3-4711-b8be-74a381d9890c' -ClusterName 'mysftestcluster' -WebApplicationReplyUrl 'https://mysftestcluster.eastus.cloudapp.azure.com:19080/Explorer/index.html' -AddResourceAccess
.\SetupUser.ps1 -ConfigObj $Configobj -UserName 'TestUser' -Password 'P@ssword!123'
.\SetupUser.ps1 -ConfigObj $Configobj -UserName 'TestAdmin' -Password 'P@ssword!123' -IsAdmin
```

> [!NOTE]
> Pour les clouds nationaux (par exemple, Azure Government, Azure Chine, Azure Allemagne), vous devez également spécifier le paramètre `-Location`.

Pour accéder à votre *TenantId*, exécutez la commande PowerShell `Get-AzureSubscription`. L’exécution de cette commande permet d’afficher le TenantId de chaque abonnement.

Le paramètre *ClusterName* est utilisé pour préfixer les applications Azure AD créées par le script. Il ne doit pas forcément correspondre précisément au nom du cluster. Il sert uniquement à simplifier le mappage des artefacts Azure AD au cluster Service Fabric avec lequel ils sont utilisés.

Le paramètre *WebApplicationReplyUrl* est le point de terminaison par défaut qu'Azure AD renvoie à vos utilisateurs après leur connexion. Définissez ce point de terminaison en tant que point de terminaison Service Fabric Explorer de votre cluster. Si vous créez des applications Azure AD pour représenter un cluster existant, veillez à ce que cette URL corresponde au point de terminaison de votre cluster. Si vous créez des applications pour un nouveau cluster, planifiez le point de terminaison que votre cluster aura et faites attention à ne pas utiliser celui d’un cluster existant. Par défaut, le point de terminaison Service Fabric Explorer est le suivant :

https://&lt;cluster_domain&gt;:19080/Explorer

Vous êtes invité à vous connecter à un compte qui dispose de privilèges d’administration pour le locataire Azure AD. Une fois que vous vous êtes connecté, le script crée les applications web et native pour représenter votre cluster Service Fabric. Si vous regardez les applications du locataire dans le [portail Azure][azure-portal], vous devez voir deux nouvelles entrées :

   * *ClusterName*\_Cluster
   * *ClusterName*\_Client

Comme le script imprime le code JSON exigé par le modèle Azure Resource Manager lors de la [création du cluster avec AAD activé](service-fabric-cluster-creation-create-template.md#add-azure-ad-configuration-to-use-azure-ad-for-client-access), il est préférable de garder la fenêtre PowerShell ouverte.

```json
"azureActiveDirectory": {
  "tenantId":"<guid>",
  "clusterApplication":"<guid>",
  "clientApplication":"<guid>"
},
```

## <a name="troubleshooting-help-in-setting-up-azure-active-directory"></a>Aide pour la résolution des problèmes de configuration d’Azure Active Directory
La configuration et l’utilisation d’Azure AD peuvent être difficiles, voici donc quelques conseils sur ce que vous pouvez faire pour résoudre le problème.

### <a name="service-fabric-explorer-prompts-you-to-select-a-certificate"></a>Service Fabric Explorer vous demande de sélectionner un certificat
#### <a name="problem"></a>Problème
Une fois que vous vous êtes connecté à Azure AD dans Service Fabric Explorer, le navigateur revient à la page d’accueil, mais un message vous invite à sélectionner un certificat.

![Boîte de dialogue Certificat SFX][sfx-select-certificate-dialog]

#### <a name="reason"></a>Motif
Aucun rôle n’a été affecté à l’utilisateur dans l’application Azure AD en cluster. Par conséquent, l’authentification Azure AD échoue sur le cluster Service Fabric. Service Fabric Explorer revient à l’authentification par certificat.

#### <a name="solution"></a>Solution
Suivez les instructions de configuration d’Azure AD et affectez des rôles utilisateur. Nous vous recommandons également d’activer l’option Affectation de l’utilisateur requise pour accéder à l’application, comme le fait `SetupApplications.ps1`.

### <a name="connection-with-powershell-fails-with-an-error-the-specified-credentials-are-invalid"></a>La connexion à PowerShell échoue avec l’erreur suivante : « Les informations d’identification spécifiées sont non valides ».
#### <a name="problem"></a>Problème
Lorsque vous utilisez PowerShell pour vous connecter au cluster à l’aide du mode de sécurité « AzureActiveDirectory », une fois que vous vous êtes connecté à Azure AD, la connexion échoue avec le message d’erreur suivant : « Les informations d’identification spécifiées sont non valides ».

#### <a name="solution"></a>Solution
La solution est la même que pour le problème précédent.

### <a name="service-fabric-explorer-returns-a-failure-when-you-sign-in-aadsts50011"></a>Service Fabric Explorer retourne un code d’échec lors de la connexion : « AADSTS50011 »
#### <a name="problem"></a>Problème
Lorsque vous essayez de vous connecter à Azure AD dans Service Fabric Explorer, la page retourne un message d’échec : « AADSTS50011 : L’adresse de réponse &lt;url&gt; ne correspond pas aux adresses de réponse configurées pour l’application : &lt;guid&gt; ».

![L’adresse de réponse SFX ne correspond pas][sfx-reply-address-not-match]

#### <a name="reason"></a>Motif
L’application en cluster (web) représentant Service Fabric Explorer tente de s’authentifier auprès d’Azure AD. Dans le cadre de cette demande, elle fournit l’URL de redirection. Cette dernière ne figure cependant pas dans la liste **URL DE RÉPONSE** de l’application Azure AD.

#### <a name="solution"></a>Solution
Dans la page d’inscription d’application Azure AD pour votre cluster, sélectionnez **Authentification**, puis dans la section **URI de redirection**, ajoutez l’URL Service Fabric Explorer à la liste. Enregistrez vos modifications.

![URL de réponse d’application web][web-application-reply-url]

### <a name="connecting-to-the-cluster-using-azure-ad-authentication-via-powershell-gives-an-error-when-you-sign-in-aadsts50011"></a>La connexion au cluster à l’aide de l’authentification Azure AD via PowerShell génère une erreur lorsque vous vous connectez : « AADSTS50011 »
#### <a name="problem"></a>Problème
Quand vous essayez de vous connecter à un cluster Service Fabric à l’aide d’Azure AD via PowerShell, la page de connexion renvoie un message d’erreur : « AADSTS50011 : L’URL de réponse spécifiée dans la requête ne correspond pas aux URL de réponse configurées pour l’application : &lt;guid&gt;. »

#### <a name="reason"></a>Motif
À l’instar du problème précédent, PowerShell tente de s’authentifier auprès d’Azure AD, qui fournit une URL de redirection qui n’est pas répertoriée dans la liste **URL de réponse** de l’application Azure AD.  

#### <a name="solution"></a>Solution
Utilisez le même processus que dans le problème précédent, mais l’URL doit être définie sur `urn:ietf:wg:oauth:2.0:oob`, une redirection spéciale pour l’authentification de la ligne de commande.

### <a name="connect-the-cluster-by-using-azure-ad-authentication-via-powershell"></a>Connecter le cluster à l’aide de l’authentification Azure AD via PowerShell
Pour connecter le cluster Service Fabric, utilisez l’exemple de commande PowerShell suivant :

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <endpoint> -KeepAliveIntervalInSec 10 -AzureActiveDirectory -ServerCertThumbprint <thumbprint>
```

Pour plus d’informations, consultez l’applet de commande [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster).

### <a name="can-i-reuse-the-same-azure-ad-tenant-in-multiple-clusters"></a>Puis-je utiliser le même locataire Azure AD pour plusieurs clusters ?
Oui. Cependant, n’oubliez pas d’ajouter l’URL de Service Fabric Explorer à votre application en cluster (web). Si vous ne le faites pas, Service Fabric Explorer ne fonctionnera pas.

### <a name="why-do-i-still-need-a-server-certificate-while-azure-ad-is-enabled"></a>Pourquoi dois-je disposer d’un certificat de serveur lorsqu’Azure AD est activé ?
FabricClient et FabricGateway effectuent une authentification mutuelle. Pendant l’authentification Azure AD, l’intégration Azure AD fournit une identité de client au serveur ; le certificat de serveur est utilisé par le client pour vérifier l’identité du serveur. Pour plus d’informations sur les certificats Service Fabric, consultez [Certificats X.509 et Service Fabric][x509-certificates-and-service-fabric].

## <a name="next-steps"></a>Étapes suivantes
Après avoir configuré les applications Azure Active Directory et défini les rôles des utilisateurs, [configurez et déployez un cluster](service-fabric-cluster-creation-via-arm.md).


<!-- Links -->
[azure-CLI]:https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest
[azure-portal]: https://portal.azure.com/
[service-fabric-cluster-security]: service-fabric-cluster-security.md
[active-directory-howto-tenant]:../active-directory/develop/quickstart-create-new-tenant.md
[service-fabric-visualizing-your-cluster]: service-fabric-visualizing-your-cluster.md
[service-fabric-manage-application-in-visual-studio]: service-fabric-manage-application-in-visual-studio.md
[sf-aad-ps-script-download]:http://servicefabricsdkstorage.blob.core.windows.net/publicrelease/MicrosoftAzureServiceFabric-AADHelpers.zip
[x509-certificates-and-service-fabric]: service-fabric-cluster-security.md#x509-certificates-and-service-fabric

<!-- Images -->
[sfx-select-certificate-dialog]: ./media/service-fabric-cluster-creation-setup-aad/sfx-select-certificate-dialog.png
[sfx-reply-address-not-match]: ./media/service-fabric-cluster-creation-setup-aad/sfx-reply-address-not-match.png
[web-application-reply-url]: ./media/service-fabric-cluster-creation-setup-aad/web-application-reply-url.png
