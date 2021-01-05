---
title: Configurer l’accélération automatique de la connexion à l’aide de la découverte du domaine d’accueil
description: Découvrez comment configurer la stratégie de découverte du domaine d’accueil pour l’authentification Azure Active Directory pour les utilisateurs fédérés (et notamment l’accélération automatique et les indications de domaine).
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 04/08/2019
ms.author: kenwith
ms.custom: seoapril2019
ms.collection: M365-identity-device-management
ms.openlocfilehash: 873a87ed2c75d41e0a249bde4b6a29921b7e5ce5
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94648053"
---
# <a name="configure-azure-active-directory-sign-in-behavior-for-an-application-by-using-a-home-realm-discovery-policy"></a>Configurer le comportement de la connexion Azure Active Directory pour une application à l’aide d’une stratégie de découverte du domaine d’accueil

Cet article fournit une introduction à la configuration du comportement d’authentification Azure Active Directory pour les utilisateurs fédérés. Il couvre la configuration des restrictions d’accélération automatique et d’authentification pour les utilisateurs de domaines fédérés.

## <a name="home-realm-discovery"></a>Découverte du domaine d’accueil
La découverte du domaine d’accueil est le processus qui permet à Azure Active Directory (Azure AD) de déterminer, au moment de la connexion, où un utilisateur doit s’authentifier.  Quand un utilisateur se connecte à un locataire Azure AD pour accéder à une ressource ou à la page de connexion courante d’Azure AD, il tape un nom d’utilisateur (UPN). Azure AD l’utilise pour découvrir où l’utilisateur doit se connecter. 

L’utilisateur peut devoir être dirigé vers l’un des emplacements suivants pour être authentifié :

- Le locataire de base de l’utilisateur (peut être le même locataire que la ressource à laquelle l’utilisateur tente d’accéder). 

- Compte Microsoft.  L’utilisateur est un invité dans le locataire de la ressource.

-  Un fournisseur d’identité local tel que les services de fédération Active Directory (AD FS).

- Un autre fournisseur d’identité qui est fédéré avec le locataire Azure AD.

## <a name="auto-acceleration"></a>Accélération automatique 
Certaines organisations configurent des domaines dans leur locataire Azure Active Directory pour le fédérer avec un autre fournisseur d’identité, tel qu’AD FS, pour l’authentification des utilisateurs.  

Quand un utilisateur se connecte à une application, il se voit d’abord présenter une page de connexion Azure AD. Après avoir entré son UPN, s’il se trouve dans un domaine fédéré, il est alors dirigé vers la page de connexion de l’IdP desservant ce domaine. Dans certaines circonstances, les administrateurs peuvent vouloir diriger les utilisateurs vers la page de connexion quand ils se connectent à des applications spécifiques. 

En conséquence, les utilisateurs peuvent ignorer la page Azure Active Directory initiale. Ce processus est dénommée « accélération automatique de la connexion ».

Dans les cas où le locataire est fédéré avec un autre fournisseur d’identité pour la connexion, l’accélération automatique simplifie la connexion de l’utilisateur.  Vous pouvez configurer l’accélération automatique pour des applications individuelles.

>[!NOTE]
>Si vous configurez une application pour l’accélération automatique, les utilisateurs invités ne peuvent pas se connecter. Si vous dirigez un utilisateur directement vers un fournisseur d’identité fédéré pour l’authentification, il lui est impossible de revenir à la page de connexion Azure Active Directory. Les utilisateurs invités, qui doivent éventuellement être dirigés vers d’autres locataires ou un fournisseur d’identité externe tel qu’un compte Microsoft, ne peuvent pas se connecter à cette application, car ils ignorent l’étape de la découverte du domaine d’accueil.  

Il existe deux façons de contrôler l’accélération automatique vers un fournisseur d’identité :   

- Utiliser une indication de domaine sur les demandes d’authentification pour une application. 
- Configurer une stratégie de découverte du domaine d’accueil pour activer l’accélération automatique.

### <a name="domain-hints"></a>Indications de domaine    
Les indications de domaine sont des directives qui sont incluses dans la demande d’authentification émise par une application. Elles peuvent servir à accélérer l’utilisateur vers la page de connexion de son fournisseur d’identité fédéré. Ou elles peuvent être utilisées par une application multilocataire pour accélérer l’utilisateur directement vers la page de connexion Azure AD personnalisée pour son locataire.  

Par exemple, l’application « largeapp.com » peut permettre à ses clients d’accéder à l’application depuis une URL « contoso.largeapp.com » personnalisée. L’application peut également inclure une indication de domaine pour contoso.com dans la demande d’authentification. 

La syntaxe des indications de domaine varie selon le protocole utilisé et est généralement configurée dans l’application.

**WS-Federation** : whr=contoso.com dans la chaîne de requête.

**SAML** :  requête d’authentification SAML qui contient une indication de domaine ou une chaîne de requête whr=contoso.com.

**Open ID Connect** : chaîne de requête domain_hint=contoso.com. 

Si une indication de domaine est incluse dans la demande d’authentification émise par l’application et que le locataire est fédéré avec ce domaine, Azure Active Directory tente de rediriger la connexion vers le fournisseur d’identité qui est configuré pour ce domaine. 

Si l’indication de domaine ne fait pas référence à un domaine fédéré vérifié, elle est ignorée et la découverte du domaine d’accueil normale est appelée.

Pour plus d’informations sur l’accélération automatique à l’aide des indications de domaine qui sont prises en charge par Azure Active Directory, consultez le [blog Enterprise Mobility + Security](https://cloudblogs.microsoft.com/enterprisemobility/2015/02/11/using-azure-ad-to-land-users-on-their-custom-login-page-from-within-your-app/).

>[!NOTE]
>Si une indication de domaine est incluse dans une requête d’authentification, sa présence remplace toute accélération automatique définie pour l’application dans la stratégie de découverte du domaine d’accueil.

### <a name="home-realm-discovery-policy-for-auto-acceleration"></a>Stratégie de découverte du domaine d’accueil pour l’accélération automatique
Certaines applications n’offrent pas la possibilité de configurer la demande d’authentification qu’elles émettent. Dans ces cas, il est impossible d’utiliser des indications de domaine pour contrôler l’accélération automatique. Vous pouvez configurer l’accélération automatique par le biais d’une stratégie pour obtenir le même comportement.  

## <a name="enable-direct-ropc-authentication-of-federated-users-for-legacy-applications"></a>Activer l’authentification ROPC directe des utilisateurs fédérés pour les applications héritées
Pour les applications, il est recommandé d’utiliser des bibliothèques AAD et une connexion interactive pour authentifier les utilisateurs. Les bibliothèques gèrent les flux d’utilisateurs fédérés.  Parfois, les applications héritées, en particulier celles qui utilisent l’octroi ROPC, envoient le nom d’utilisateur et le mot de passe directement à Azure AD ; elles ne sont pas écrites pour comprendre la fédération. Elles n’exécutent pas la découverte de domaine d’accueil et n’interagissent pas avec le point de terminaison fédéré approprié pour authentifier un utilisateur. Si vous le souhaitez, vous pouvez utiliser la stratégie de découverte du domaine d’accueil pour autoriser des applications héritées spécifiques qui envoient des informations d’identification de nom d’utilisateur/mot de passe avec l’octroi ROPC à s’authentifier directement auprès d’Azure Active Directory. La synchronisation de hachage de mot de passe doit être activée. 

> [!IMPORTANT]
> Activez uniquement l’authentification directe si vous avez activé la synchronisation de hachage de mot de passe et que vous pouvez authentifier cette application sans aucune stratégie implémentée par votre fournisseur d’identité locale. Si vous désactivez la synchronisation de hachage de mot de passe ou la synchronisation d’annuaires avec AD Connect pour une raison quelconque, vous devez supprimer cette stratégie pour empêcher la possibilité d’authentification directe à l’aide d’un hachage de mot de passe obsolète.

## <a name="set-hrd-policy"></a>Définir une stratégie de découverte du domaine d’accueil
Il existe trois étapes de définition de la stratégie de découverte du domaine d’accueil sur une application pour l’accélération automatique de connexion fédérée ou les applications cloud directes :

1. Créez une stratégie de découverte du domaine d’accueil.

2. Localisez le principal de service auquel attacher la stratégie.

3. Attachez la stratégie au principal de service. 

Les stratégies prennent effet pour une application spécifique uniquement lorsqu’elles sont attachées à un principal de service. 

Une seule stratégie de découverte du domaine d’accueil peut être active sur un principal de service à tout moment.  

Vous pouvez utiliser les cmdlets PowerShell Azure Active Directory pour créer et gérer la stratégie de découverte du domaine d’accueil.

Voici un exemple de définition de la stratégie de découverte du domaine d’accueil :
    
 ```
   {  
    "HomeRealmDiscoveryPolicy":
    {  
    "AccelerateToFederatedDomain":true,
    "PreferredDomain":"federated.example.edu",
    "AllowCloudPasswordValidation":false
    }
   }
```

Le type de stratégie est « HomeRealmDiscoveryPolicy ».

**AccelerateToFederatedDomain** est facultatif. Si **AccelerateToFederatedDomain** a la valeur false, la stratégie n’a aucun effet sur l’accélération automatique. Si **AccelerateToFederatedDomain** a la valeur true et qu’il existe un seul domaine vérifié et fédéré dans le client, les utilisateurs seront directement dirigés vers le fournisseur d'identité fédéré pour la connexion. S’il a la valeur true et qu’il existe plusieurs domaines vérifiés dans le client, **PreferredDomain** doit être spécifié.

**PreferredDomain** est facultatif. **PreferredDomain** doit indiquer un domaine vers lequel vous souhaitez accélérer. Il peut être omis si le locataire n’a qu’un seul domaine fédéré.  S’il est omis et qu’il existe plusieurs domaines fédérés vérifiés, la stratégie n’a aucun effet.

 Si **PreferredDomain** est spécifié, il doit correspondre à un domaine fédéré vérifié pour le locataire. Tous les utilisateurs de l’application doivent être en mesure de se connecter à ce domaine.

**AllowCloudPasswordValidation** est facultatif. Si **AllowCloudPasswordValidation** est true, l’application est autorisée à authentifier un utilisateur fédéré à l’aide des informations d’identification de nom d’utilisateur/mot de passe directement auprès du point de terminaison de jeton Azure Active Directory. Cela fonctionne uniquement si la synchronisation de hachage de mot de passe est activée.

### <a name="priority-and-evaluation-of-hrd-policies"></a>Priorité et évaluation des stratégies de découverte du domaine d’accueil
Les stratégies de découverte du domaine d’accueil peuvent être créées, puis affectées à des organisations et principaux de service spécifiques. Il est donc possible d’appliquer plusieurs stratégies à une application spécifique. La stratégie de découverte du domaine d’accueil appliquée suit les règles ci-dessous :


- Si une indication de domaine est présente dans la requête d’authentification, toute stratégie de découverte du domaine d’accueil est ignorée pour l’accélération automatique. Le comportement spécifié par l’indication de domaine est utilisé.

- Sinon, si une stratégie est explicitement affectée au principal de service, elle est appliquée. 

- S’il n’existe aucune indication de domaine et qu’aucune stratégie n’est explicitement affectée au principal de service, une stratégie qui est explicitement affectée à l’organisation parente du principal de service est appliquée. 

- S’il n’existe aucune indication de domaine et qu’aucune stratégie n’a été affectée au principal de service ou à l’organisation, le comportement de découverte du domaine d’accueil par défaut est utilisé.

## <a name="tutorial-for-setting-hrd-policy-on-an-application"></a>Tutoriel pour définir une stratégie de découverte du domaine d’accueil sur une application 
Nous allons utiliser des applets de commande PowerShell Azure AD dans le cadre de quelques scénarios, dont les suivants :


- Configurer une stratégie de découverte du domaine d’accueil afin d’exécuter l’accélération automatique pour une application dans un client avec un seul domaine fédéré.

- Configurer une stratégie de découverte du domaine d’accueil afin d’exécuter l’accélération automatique pour une application vers un des domaines parmi ceux qui sont vérifiés pour votre client.

- Configurer une stratégie de découverte du domaine d’accueil afin d’effectuer l’authentification directe du nom d’utilisateur/mot de passe sur Azure Active Directory pour un utilisateur fédéré.

- Répertorier les applications pour lesquelles une stratégie est configurée.


### <a name="prerequisites"></a>Prérequis
Dans les exemples suivants, vous créez, mettez à jour, liez et supprimez des stratégies sur des principaux de service d’application dans Azure AD.

1.  Pour commencer, téléchargez la dernière préversion des applets de commande Azure AD PowerShell. 

2.  Une fois que vous avez téléchargé les applets de commande Azure AD PowerShell, exécutez la commande Connect pour vous connecter à Azure AD avec votre compte d’administrateur :

    ``` powershell
    Connect-AzureAD -Confirm
    ```
3.  Exécutez la commande suivante pour afficher toutes les stratégies dans votre organisation :

    ``` powershell
    Get-AzureADPolicy
    ```

Si aucun résultat n’est retourné, cela signifie qu’aucune stratégie n’est créée dans votre locataire.

### <a name="example-set-an-hrd-policy-for-an-application"></a>Exemple : Définir une stratégie de découverte du domaine d’accueil pour une application 

Dans cet exemple, vous créez une stratégie qui, lorsqu’elle est assignée à une application : 
- accélère automatiquement les utilisateurs vers un écran de connexion AD FS quand ils se connectent à une application lorsque votre client comporte un seul domaine. 
- accélère automatiquement les utilisateurs vers un écran de connexion AD FS quand il existe plusieurs domaines fédérés dans votre client.
- active la connexion du nom d’utilisateur/mot de passe non interactive directement dans Azure Active Directory pour les utilisateurs fédérés dans les applications auxquelles la stratégie est affectée.

#### <a name="step-1-create-an-hrd-policy"></a>Étape 1 : Créer une stratégie de découverte du domaine d’accueil

La stratégie suivante accélère automatiquement les utilisateurs vers un écran de connexion AD FS quand ils se connectent à une application lorsque votre client comporte un seul domaine.

``` powershell
New-AzureADPolicy -Definition @("{`"HomeRealmDiscoveryPolicy`":{`"AccelerateToFederatedDomain`":true}}") -DisplayName BasicAutoAccelerationPolicy -Type HomeRealmDiscoveryPolicy
```
La stratégie suivante accélère automatiquement les utilisateurs vers un écran de connexion AD FS quand il existe plusieurs domaines fédérés dans votre client. Si vous avez plusieurs domaines fédérés qui authentifient les utilisateurs pour les applications, vous devez spécifier le domaine pour l’accélération automatique.

``` powershell
New-AzureADPolicy -Definition @("{`"HomeRealmDiscoveryPolicy`":{`"AccelerateToFederatedDomain`":true, `"PreferredDomain`":`"federated.example.edu`"}}") -DisplayName MultiDomainAutoAccelerationPolicy -Type HomeRealmDiscoveryPolicy
```

Pour créer une stratégie pour activer l’authentification du nom d’utilisateur/mot de passe pour les utilisateurs fédérés directement avec Azure Active Directory pour des applications spécifiques, exécutez la commande suivante :

``` powershell
New-AzureADPolicy -Definition @("{`"HomeRealmDiscoveryPolicy`":{`"AllowCloudPasswordValidation`":true}}") -DisplayName EnableDirectAuthPolicy -Type HomeRealmDiscoveryPolicy
```


Pour afficher votre nouvelle stratégie et obtenir son **ID d’objet**, exécutez la commande ci-après :

``` powershell
Get-AzureADPolicy
```


Pour appliquer une stratégie de découverte du domaine d’accueil après l’avoir créée, vous pouvez l’affecter à plusieurs principaux de service d’application.

#### <a name="step-2-locate-the-service-principal-to-which-to-assign-the-policy"></a>Étape 2 : Rechercher le principal de service auquel affecter la stratégie  
Vous avez besoin de **l’ID d’objet** des principaux de service auxquels vous souhaitez affecter la stratégie. Il existe plusieurs façons de rechercher **l’ID d’objet** des principaux de service.    

Vous pouvez utiliser le portail, ou vous pouvez interroger [Microsoft Graph](/graph/api/resources/serviceprincipal?view=graph-rest-beta). Vous pouvez également accéder à [l’outil Afficheur Graph](https://developer.microsoft.com/graph/graph-explorer) et vous connecter à votre compte Azure AD pour voir tous les principaux de service de votre organisation. 

Étant donné que vous utilisez PowerShell, vous pouvez utiliser l’applet de commande suivante pour lister les principaux de service et leurs identifiants.

``` powershell
Get-AzureADServicePrincipal
```

#### <a name="step-3-assign-the-policy-to-your-service-principal"></a>Étape 3 : Affecter la stratégie à votre principal de service  
Une fois que vous avez **l’ID d’objet** du principal de service de l’application pour laquelle vous souhaitez configurer l’accélération automatique, exécutez la commande suivante. Cette commande associe la stratégie de découverte du domaine d’accueil que vous avez créée à l’étape 1 au principal de service que vous avez localisé à l’étape 2.

``` powershell
Add-AzureADServicePrincipalPolicy -Id <ObjectID of the Service Principal> -RefObjectId <ObjectId of the Policy>
```

Vous pouvez répéter cette commande pour chaque principal de service auquel vous souhaitez ajouter la stratégie.

Dans le cas où une stratégie HomeRealmDiscovery est déjà affectée à une application, vous ne pourrez pas en ajouter une deuxième.  Dans ce cas, modifiez la définition de la stratégie de découverte du domaine d’accueil affectée à l’application pour ajouter des paramètres supplémentaires.

#### <a name="step-4-check-which-application-service-principals-your-hrd-policy-is-assigned-to"></a>Étape 4 : Vérifier les principaux de service d’application auxquels votre stratégie de découverte du domaine d’accueil est affectée
Pour déterminer les applications pour lesquelles la stratégie de découverte du domaine d’accueil est configurée, utilisez l’applet de commande **Get-AzureADPolicyAppliedObject**. Passez-lui **l’ID d’objet** de la stratégie sur laquelle porte la vérification.

``` powershell
Get-AzureADPolicyAppliedObject -id <ObjectId of the Policy>
```
#### <a name="step-5-youre-done"></a>Étape 5 : Vous avez terminé !
Testez l’application pour vérifier que la nouvelle stratégie fonctionne.

### <a name="example-list-the-applications-for-which-hrd-policy-is-configured"></a>Exemple : répertorier les applications pour lesquelles une stratégie de découverte du domaine d’accueil est configurée

#### <a name="step-1-list-all-policies-that-were-created-in-your-organization"></a>Étape 1 : Répertorier toutes les stratégies créées dans votre organisation 

``` powershell
Get-AzureADPolicy
```

Notez **l’ID d’objet** de la stratégie dont vous souhaitez répertorier les affectations.

#### <a name="step-2-list-the-service-principals-to-which-the-policy-is-assigned"></a>Étape 2 : Répertorier les principaux de service auxquels la stratégie est affectée  

``` powershell
Get-AzureADPolicyAppliedObject -id <ObjectId of the Policy>
```

### <a name="example-remove-an-hrd-policy-from-an-application"></a>Exemple : Supprimer une stratégie de découverte du domaine d’accueil dans une application
#### <a name="step-1-get-the-objectid"></a>Étape 1 : Obtenir l’ID d’objet
Utilisez l’exemple précédent pour obtenir **l’ID d’objet** de la stratégie et celui du principal de service d’application dont vous souhaitez la supprimer. 

#### <a name="step-2-remove-the-policy-assignment-from-the-application-service-principal"></a>Étape 2 : Supprimer l’affectation de stratégie du principal de service d’application  

``` powershell
Remove-AzureADServicePrincipalPolicy -id <ObjectId of the Service Principal>  -PolicyId <ObjectId of the policy>
```

#### <a name="step-3-check-removal-by-listing-the-service-principals-to-which-the-policy-is-assigned"></a>Étape 3 : Vérifier la suppression en listant les principaux de service auxquels la stratégie est affectée 

``` powershell
Get-AzureADPolicyAppliedObject -id <ObjectId of the Policy>
```
## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur le fonctionnement de l’authentification dans Azure AD, consultez la section [Scénarios d’authentification pour Azure AD](../develop/authentication-vs-authorization.md).
- Pour plus d’informations sur l’authentification unique de l’utilisateur, consultez [Authentification unique aux applications dans Azure Active Directory](what-is-single-sign-on.md).
- Visitez la [plateforme d’identités Microsoft](../develop/v2-overview.md) pour avoir une vue d’ensemble de la documentation destinée aux développeurs.