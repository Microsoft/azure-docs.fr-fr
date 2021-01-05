---
title: Fédération directe avec un fournisseur d’identité pour B2B - Azure AD
description: Fédérer directement avec un fournisseur d’identité SAML ou WS-Fed de sorte que les invités puissent se connecter à vos applications Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 06/24/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5beae56a5d38c4620481c27c3f42c52602984e6b
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860624"
---
# <a name="direct-federation-with-ad-fs-and-third-party-providers-for-guest-users-preview"></a>Fédération directe avec AD FS et des fournisseurs tiers pour les utilisateurs invités (version préliminaire)

> [!NOTE]
>  La fédération directe est une fonctionnalité d’évaluation publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Cet article décrit comment configurer la fédération directe avec une autre organisation pour la collaboration B2B. Vous pouvez configurer la fédération directe avec n’importe quelle organisation dont le fournisseur d’identité (IdP) prend en charge le protocole SAML 2.0 ou WS-Fed.
Lorsque vous configurez la fédération directe avec un fournisseur d’identité partenaire, les nouveaux utilisateurs invités de ce domaine peuvent utiliser leur propre compte professionnel géré par le fournisseur d’identité pour se connecter à votre locataire Azure AD et commencer à travailler avec vous. Il n’est pas nécessaire pour l’utilisateur invité de créer un compte Azure AD séparé.
> [!NOTE]
> Vos utilisateurs invités de fédération directe doivent se connecter à l’aide d’un lien incluant le contexte du locataire (par exemple, `https://myapps.microsoft.com/?tenantid=<tenant id>` ou `https://portal.azure.com/<tenant id>`, ou dans le cas d’un domaine vérifié, `https://myapps.microsoft.com/\<verified domain>.onmicrosoft.com`). Liens directs aux applications et ressources fonctionnent également tant qu’ils incluent le contexte client. Les utilisateurs de la fédération directe ne peuvent actuellement pas se connecter à l’aide de points de terminaison communs qui n’ont aucun contexte locataire. Par exemple, l’utilisation de `https://myapps.microsoft.com`, `https://portal.azure.com`, ou `https://teams.microsoft.com` entraîne une erreur.
 
## <a name="when-is-a-guest-user-authenticated-with-direct-federation"></a>Quand un utilisateur invité est-il authentifié avec la fédération directe ?
Après avoir configuré la fédération directe avec une organisation, tous les utilisateurs invités que vous invitez seront authentifiés à l’aide de la fédération directe. Il est important de noter que configurer la fédération directe ne change pas la méthode d’authentification pour les utilisateurs invités qui ont déjà utilisé une invitation de votre part. Voici quelques exemples :
 - Si les utilisateurs invités ont déjà utilisé des invitations provenant de vous, et que vous avez par la suite configuré la fédération directe avec leur organisation, ces utilisateurs invités continueront à utiliser la même méthode d’authentification qu’ils ont utilisée avant que vous ne configuriez la fédération directe.
 - Si vous configurez la fédération directe avec une organisation partenaire et invitez des utilisateurs invités et qu’ensuite l’organisation partenaire passe à Azure AD, les utilisateurs invités qui ont déjà utilisé des invitations continueront d’utiliser la fédération directe, tant que la stratégie existe dans votre locataire.
 - Si vous supprimez la fédération directe avec une organisation partenaire, tous les utilisateurs invités qui utilisent actuellement la fédération directe ne pourront plus se connecter.

Dans un de ces scénarios, vous pouvez mettre à jour la méthode d’authentification d’un utilisateur invité en supprimant le compte d’utilisateur invité de votre répertoire et en le réinvitant.

La fédération directe est liée aux espaces de noms domaine, par exemple contoso.com et fabrikam.com. Lorsque vous établissez une configuration de la fédération directe avec AD FS ou un fournisseur d’identité tiers, les organisations associent un ou plusieurs espaces de noms domaine à ces fournisseurs d’identité. 

## <a name="end-user-experience"></a>Expérience de l’utilisateur final 
Avec la fédération directe, les utilisateurs invités se connectent à votre client Azure AD à l’aide de leur propre compte professionnel. Lorsqu’ils accèdent à des ressources partagées et qu’ils sont invités à se connecter, les utilisateurs de la fédération directe sont redirigés vers leur fournisseur d’identité. Après s’être connectés, ils sont renvoyés vers Azure AD pour accéder aux ressources. Les jetons d’actualisation des utilisateurs de la fédération directe sont valides pendant 12 heures, qui est la [durée par défaut de transfert d’un jeton d’actualisation](../develop/active-directory-configurable-token-lifetimes.md#exceptions) dans Azure AD. Si le fournisseur d’identité fédéré dispose de l’authentification unique, l’utilisateur en profitera et ne verra aucune invite de connexion après son authentification initiale.

## <a name="limitations"></a>Limites

### <a name="dns-verified-domains-in-azure-ad"></a>Domaines vérifiés par DNS dans Azure AD
Le domaine avec lequel vous voulez vous fédérer ne doit ***pas** _ être vérifié par le système DNS dans Azure AD. Vous êtes autorisé à configurer la fédération directe avec des locataires Azure AD non managés (vérifiés par e-mail ou « viraux »), car ils ne sont pas vérifiés par le système DNS.

### <a name="authentication-url"></a>URL d’authentification
La fédération directe est uniquement autorisée pour les stratégies où le domaine de l’URL d’authentification correspond au domaine cible, ou lorsque l’URL d’authentification correspond à l’un de ces fournisseurs d’identité autorisés (cette liste est susceptible de changer) :

-   accounts.google.com
-   pingidentity.com
-   login.pingone.com
-   okta.com
-   oktapreview.com
-   okta-emea.com
-   my.salesforce.com
-   federation.exostar.com
-   federation.exostartest.com

Par exemple, lorsque vous configurez la fédération directe pour _*fabrikam.com**, l'URL d'authentification `https://fabrikam.com/adfs` transmet la validation. Un ordinateur hôte dans le même domaine passe également, par exemple `https://sts.fabrikam.com/adfs`. Toutefois, l’URL d’authentification `https://fabrikamconglomerate.com/adfs` ou `https://fabrikam.com.uk/adfs` pour le même domaine ne passera pas.

### <a name="signing-certificate-renewal"></a>Renouvellement du certificat de signature
Si vous spécifiez l’URL de métadonnées dans les paramètres du fournisseur d’identité, Azure AD renouvelle automatiquement le certificat de signature à son expiration. Toutefois, si le certificat pivote avant le délai d’expiration pour une raison quelconque, ou si vous ne fournissez pas une URL de métadonnées, Azure AD ne pourra pas le renouveler. Dans ce cas, vous devez mettre à jour le certificat de signature manuellement.

### <a name="limit-on-federation-relationships"></a>Limite des relations de fédération
Pour le moment, le nombre de relations de fédération maximum pris en charge est limité à 1 000. Cette limite concerne à la fois les [fédérations internes et les fédérations directes](/powershell/module/msonline/set-msoldomainfederationsettings).

### <a name="limit-on-multiple-domains"></a>Limite des domaines multiples
Nous ne prenons pas en charge actuellement la fédération directe avec plusieurs domaines du même locataire.

## <a name="frequently-asked-questions"></a>Forum aux questions
### <a name="can-i-set-up-direct-federation-with-a-domain-for-which-an-unmanaged-email-verified-tenant-exists"></a>Puis-je configurer la fédération directe avec un domaine pour lequel un client non géré (vérifié par e-mail) existe ? 
Oui. Si le domaine n’a pas été vérifié et que le locataire n’a pas subi de [prise en charge administrateur](../enterprise-users/domains-admin-takeover.md), vous pouvez configurer la fédération directe avec de domaine. Les clients non gérés, ou vérifiés par e-mail, sont créés lorsqu’un utilisateur a utilisé une invitation B2B ou effectue une inscription libre-service pour Azure AD à l’aide d’un domaine qui n’existe pas actuellement. Vous pouvez configurer la fédération directe avec ces domaines. Si vous essayez de configurer la fédération directe avec un domaine vérifié par DNS, dans le portail Azure ou via PowerShell, vous rencontrerez une erreur.
### <a name="if-direct-federation-and-email-one-time-passcode-authentication-are-both-enabled-which-method-takes-precedence"></a>Si la fédération directe et l’authentification par code secret à usage unique par e-mail sont activées, quelle méthode est prioritaire ?
Lorsque la fédération directe est établie avec une organisation partenaire, elle est prioritaire sur l’authentification par code secret à usage unique par e-mail pour les nouveaux utilisateurs invités de cette organisation. Si un utilisateur invité a utilisé une invitation à l’aide de l’authentification par code secret à usage unique avant que vous n’ayez configuré la fédération directe, il continue d’utiliser l’authentification par code secret à usage unique. 
### <a name="does-direct-federation-address-sign-in-issues-due-to-a-partially-synced-tenancy"></a>La fédération directe répond-elle aux problèmes de connexion, étant une location partiellement synchronisée ?
Non, la fonctionnalité de connexion par [code secret à usage unique par e-mail](one-time-passcode.md) doit être utilisée dans ce cas. Une « location partiellement synchronisée » fait référence à un locataire Azure AD partenaire où les identités utilisateur locales ne sont pas entièrement synchronisées avec le cloud. Un invité dont l’identité n’existe pas encore dans le cloud mais qui essaie d’utiliser votre invitation B2B ne pourra pas se connecter. La fonctionnalité de code secret à usage unique permettrait à cet invité de se connecter. La fonctionnalité de fédération directe répond aux scénarios où l’invité a son propre compte professionnel géré par le fournisseur d’identité, mais que l’organisation n’a aucune présence sur Azure AD.

## <a name="step-1-configure-the-partner-organizations-identity-provider"></a>Étape 1 : Configurer le fournisseur d’identité de l’organisation partenaire
Votre organisation partenaire doit d’abord configurer son fournisseur d’identité avec les revendications requises et les approbations de partie de confiance. 

> [!NOTE]
> Pour illustrer comment configurer un fournisseur d’identité pour la fédération directe, nous allons utiliser les services de fédération Active Directory (AD FS) comme exemple. Consultez l’article [Configurer la fédération directe avec AD FS](direct-federation-adfs.md), qui donne des exemples montrant comment configurer AD FS en tant que fournisseur d’identité SAML 2.0 ou WS-Fed en vue de la fédération directe.

### <a name="saml-20-configuration"></a>Configuration SAML 2.0

Azure AD B2B peut être configuré pour la fédération avec les fournisseurs d’identité qui utilisent le protocole SAML avec certaines exigences spécifiques indiquées ci-dessous. Pour plus d’informations sur la configuration d’une approbation entre votre fournisseur d’identité SAML et Azure AD, consultez [Utiliser un fournisseur d’identité SAML 2.0 pour l’authentification unique](../hybrid/how-to-connect-fed-saml-idp.md).  

> [!NOTE]
> Le domaine cible pour la fédération directe ne doit pas être vérifié par DNS sur Azure AD. Le domaine de l’URL d’authentification doit correspondre au domaine cible, ou doit être le domaine d’un fournisseur d’identité autorisé. Consultez la section [Limitations](#limitations) pour plus d’informations. 

#### <a name="required-saml-20-attributes-and-claims"></a>Attributs SAML 2.0 et revendications requises
Les tableaux suivants présentent la configuration requise pour les attributs spécifiques et les revendications qui doivent être configurés au niveau du fournisseur d’identité tiers. Pour configurer la fédération directe, les attributs suivants doivent être reçus dans la réponse SAML 2.0 à partir du fournisseur d’identité. Ces attributs peuvent être configurés en liant le fichier XML du service d’émission de jeton de sécurité en ligne ou en les entrant manuellement.

Attributs requis pour la réponse SAML 2.0 du fournisseur d’identité :

|Attribut  |Valeur  |
|---------|---------|
|AssertionConsumerService     |`https://login.microsoftonline.com/login.srf`         |
|Public visé     |`urn:federation:MicrosoftOnline`         |
|Émetteur     |L’URI de l’émetteur du fournisseur d’identité partenaire, par exemple `http://www.example.com/exk10l6w90DHM0yi...`         |


Revendications requises pour le jeton SAML 2.0 émis par le fournisseur d’identité :

|Attribut  |Valeur  |
|---------|---------|
|Format NameID     |`urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`         |
|emailaddress     |`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`         |

### <a name="ws-fed-configuration"></a>Configuration WS-Fed 
Azure AD B2B peut être configuré pour la fédération avec les fournisseurs d’identité qui utilisent le protocole WS-Fed avec certaines exigences spécifiques indiquées ci-dessous. Actuellement, les deux fournisseurs WS-Fed testés pour assurer la compatibilité avec Azure AD incluent AD FS et Shibboleth. Pour plus d’informations sur l’établissement d’une partie de confiance entre un fournisseur compatible WS-Fed avec Azure AD, consultez le document « STS Integration Paper using WS Protocols » (Intégration » STS à l’aide des protocoles WS) disponible dans [Compatibilité des fournisseurs d’identité Azure AD](https://www.microsoft.com/download/details.aspx?id=56843).

> [!NOTE]
> Le domaine cible pour la fédération directe ne doit pas être vérifié par DNS sur Azure AD. Le domaine de l’URL d’authentification doit correspondre au domaine cible ou au domaine d’un fournisseur d’identité autorisé. Consultez la section [Limitations](#limitations) pour plus d’informations. 

#### <a name="required-ws-fed-attributes-and-claims"></a>Attributs WS-Fed et revendications requises

Les tableaux suivants présentent la configuration requise pour les attributs spécifiques et les revendications qui doivent être configurés au niveau du fournisseur d’identité WS-Fed tiers. Pour configurer la fédération directe, les attributs suivants doivent être reçus dans le message WS-Fed à partir du fournisseur d’identité. Ces attributs peuvent être configurés en liant le fichier XML du service d’émission de jeton de sécurité en ligne ou en les entrant manuellement.

Attributs requis dans le message WS-Fed du fournisseur d’identité :
 
|Attribut  |Valeur  |
|---------|---------|
|PassiveRequestorEndpoint     |`https://login.microsoftonline.com/login.srf`         |
|Public visé     |`urn:federation:MicrosoftOnline`         |
|Émetteur     |L’URI de l’émetteur du fournisseur d’identité partenaire, par exemple `http://www.example.com/exk10l6w90DHM0yi...`         |

Revendications requises pour le jeton WS-Fed émis par le fournisseur d’identité :

|Attribut  |Valeur  |
|---------|---------|
|ImmutableID     |`http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID`         |
|emailaddress     |`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`         |

## <a name="step-2-configure-direct-federation-in-azure-ad"></a>Étape 2 : Configurer la fédération directe dans Azure AD 
Ensuite, vous allez configurer la fédération avec le fournisseur d’identité configuré à l’étape 1 dans Azure AD. Vous pouvez utiliser le portail Azure AD ou PowerShell. La stratégie de la fédération directe prend effet au bout de 5 à 10 minutes. Pendant ce temps, n’essayez pas d’utiliser une invitation pour le domaine de la fédération directe. Les attributs suivants sont requis :
- URI de l’émetteur du fournisseur d’identité du partenaire
- Point de terminaison de l’authentification passive du fournisseur d’identité du partenaire (https uniquement est pris en charge)
- Certificat

### <a name="to-configure-direct-federation-in-the-azure-ad-portal"></a>Pour configurer la fédération directe dans le portail Azure AD

1. Accédez au [portail Azure](https://portal.azure.com/). Sélectionnez **Azure Active Directory** dans le volet de gauche. 
2. Sélectionnez **Identités externes** > **Tous les fournisseurs d’identité**.
3. Sélectionnez ensuite **Nouvel IdP SAML/WS-Fed**.

    ![Capture d’écran montrant le bouton pour ajouter un nouveau fournisseur d’identité SAML ou WS-Fed](media/direct-federation/new-saml-wsfed-idp.png)

4. Sur la page **Nouveau fournisseur d’identité SAML/WS-Fed**, sous **Protocole de fournisseur d’identité**, sélectionnez **SAML** ou **WS-FED**.

    ![Capture d’écran montrant le bouton d’analyse sur la page Fournisseur d’identité SAML ou WS-Fed](media/direct-federation/new-saml-wsfed-idp-parse.png)

5. Entrez le nom de domaine de votre organisation partenaire, qui sera le nom de domaine cible pour la fédération directe
6. Vous pouvez télécharger un fichier de métadonnées pour renseigner les informations des métadonnées. Si vous choisissez d’entrer les métadonnées manuellement, entrez les informations suivantes :
   - Nom de domaine du fournisseur d’identité du partenaire
   - ID d’entité du fournisseur d’identité du partenaire
   - Point de terminaison du demandeur passif du fournisseur d’identité du partenaire
   - Certificat
   > [!NOTE]
   > L’URL des métadonnées est facultative, mais fortement recommandée. Si vous fournissez l’URL des métadonnées, Azure AD peut renouveler automatiquement le certificat de signature lorsqu’il arrive à expiration. Si le certificat pivote avant le délai d’expiration pour une raison quelconque, ou si vous ne fournissez pas une URL de métadonnées, Azure AD ne pourra pas le renouveler. Dans ce cas, vous devez mettre à jour le certificat de signature manuellement.

7. Sélectionnez **Enregistrer**. 

### <a name="to-configure-direct-federation-in-azure-ad-using-powershell"></a>Pour configurer la fédération directe dans Azure AD via PowerShell

1. Installez la dernière version d’Azure AD PowerShell pour le module Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)). (Si vous avez besoin d’obtenir des instructions détaillées, le démarrage rapide pour l’ajout d’un utilisateur invité comprend la section [Installer le dernier module AzureADPreview](b2b-quickstart-invite-powershell.md#install-the-latest-azureadpreview-module).) 
2. Exécutez la commande suivante : 
   ```powershell
   Connect-AzureAD
   ```
1. À l’invite de connexion, connectez-vous avec le compte d’administrateur général géré. 
2. Exécutez les commandes suivantes, en remplaçant les valeurs à partir du fichier de métadonnées de fédération. Pour le serveur AD FS et Okta, le fichier de la fédération est federationmetadata.xml, par exemple : `https://sts.totheclouddemo.com/federationmetadata/2007-06/federationmetadata.xml`. 

   ```powershell
   $federationSettings = New-Object Microsoft.Open.AzureAD.Model.DomainFederationSettings
   $federationSettings.PassiveLogOnUri ="https://sts.totheclouddemo.com/adfs/ls/"
   $federationSettings.LogOffUri = $federationSettings.PassiveLogOnUri
   $federationSettings.IssuerUri = "http://sts.totheclouddemo.com/adfs/services/trust"
   $federationSettings.MetadataExchangeUri="https://sts.totheclouddemo.com/adfs/services/trust/mex"
   $federationSettings.SigningCertificate= <Replace with X509 signing cert’s public key>
   $federationSettings.PreferredAuthenticationProtocol="WsFed" OR "Samlp"
   $domainName = <Replace with domain name>
   New-AzureADExternalDomainFederation -ExternalDomainName $domainName  -FederationSettings $federationSettings
   ```

## <a name="step-3-test-direct-federation-in-azure-ad"></a>Étape 3 : Tester la fédération directe dans Azure AD
Testez maintenant la configuration de votre fédération directe en invitant un utilisateur invité B2B. Pour plus d’informations, consultez [Ajouter des utilisateurs Azure Active Directory B2B Collaboration dans le Portail Azure](add-users-administrator.md).
 
## <a name="how-do-i-edit-a-direct-federation-relationship"></a>Comment modifier une relation de fédération directe ?

1. Accédez au [portail Azure](https://portal.azure.com/). Sélectionnez **Azure Active Directory** dans le volet de gauche. 
2. Sélectionnez **Identités externes**.
3. Sélectionnez **Tous les fournisseurs d’identité**
4. Sous **Fournisseurs d’identité SAML/WS-Fed**, sélectionnez le fournisseur.
5. Dans le volet des informations du fournisseur identité, mettez à jour les valeurs.
6. Sélectionnez **Enregistrer**.


## <a name="how-do-i-remove-direct-federation"></a>Comment supprimer la fédération directe ?
Vous pouvez supprimer la configuration de votre fédération directe. Si vous le faites, les utilisateurs invités par fédération directe qui ont déjà utilisé leur invitation ne pourront plus se connecter. Mais vous pouvez leur donner accès à vos ressources à nouveau en les supprimant du répertoire et en les réinvitant. Pour supprimer une fédération directe avec un fournisseur d’identité dans le portail Azure AD :

1. Accédez au [portail Azure](https://portal.azure.com/). Sélectionnez **Azure Active Directory** dans le volet de gauche. 
2. Sélectionnez **Identités externes**.
3. Sélectionnez **Tous les fournisseurs d’identité**.
4. Sélectionnez le fournisseur d’identité, puis **Supprimer**. 
5. Sélectionnez **Oui** pour confirmer la suppression. 

Pour supprimer une fédération directe avec un fournisseur d’identité via PowerShell :
1. Installez la dernière version d’Azure AD PowerShell pour le module Graph ([AzureADPreview](https://www.powershellgallery.com/packages/AzureADPreview)).
2. Exécutez la commande suivante : 
   ```powershell
   Connect-AzureAD
   ```
3. À l’invite de connexion, connectez-vous avec le compte d’administrateur général géré. 
4. Entrez la commande suivante :
   ```powershell
   Remove-AzureADExternalDomainFederation -ExternalDomainName  $domainName
   ```

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur l’[expérience d’acceptation d’invitation](redemption-experience.md) lorsque des utilisateurs externes se connectent avec différents fournisseurs d’identité.
