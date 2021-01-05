---
title: Renouvellement des certificats pour les utilisateurs de Microsoft 365 et d’Azure AD | Microsoft Docs
description: Cet article explique aux utilisateurs de Microsoft 365 comment procéder s’ils reçoivent un e-mail les invitant à renouveler un certificat.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: 543b7dc1-ccc9-407f-85a1-a9944c0ba1be
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/20/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 78dcd9d020923251439a05316569b559c19057d1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89661450"
---
# <a name="renew-federation-certificates-for-microsoft-365-and-azure-active-directory"></a>Renouveler des certificats de fédération pour Microsoft 365 et Azure Active Directory
## <a name="overview"></a>Vue d’ensemble
Afin d’assurer la fédération réussie entre Azure Active Directory (Azure AD) et Active Directory Federation Services (AD FS), les certificats utilisés par AD FS pour signer les jetons de sécurité destinés à Azure AD doivent correspondre à la configuration d’Azure AD. Toute incompatibilité peut entraîner une rupture de l’approbation. Azure AD garantit la synchronisation de ces informations lorsque vous déployez AD FS et le proxy d’application web (pour l’accès extranet).

Cet article fournit des informations supplémentaires pour gérer vos certificats de signature de jetons et les maintenir synchronisés avec Azure AD dans les scénarios suivants :

* Vous ne déployez pas le proxy d’application web ; les métadonnées de fédération ne sont donc pas disponibles dans l’extranet.
* Vous n’utilisez pas la configuration par défaut d’AD FS pour les certificats de signature de jetons.
* Vous utilisez un fournisseur d’identité tiers.

## <a name="default-configuration-of-ad-fs-for-token-signing-certificates"></a>Configuration par défaut d’AD FS pour les certificats de signature de jetons
Les certificats de signature et de déchiffrement de jetons sont généralement des certificats auto-signés, valables pendant un an. Par défaut, AD FS inclut un processus de renouvellement automatique appelé **AutoCertificateRollover**. Si vous utilisez AD FS 2.0 ou version ultérieure, Microsoft 365 et Azure AD mettent automatiquement à jour votre certificat avant qu’il n’arrive à expiration.

### <a name="renewal-notification-from-the-microsoft-365-admin-center-or-an-email"></a>Notification de renouvellement à partir du centre d’administration Microsoft 365 ou d’un courrier électronique
> [!NOTE]
> Si vous avez reçu un e-mail ou une notification du portail vous invitant à renouveler votre certificat pour Office, consultez [Gestion des modifications apportées aux certificats de signature de jeton](#managecerts) pour savoir si une action est requise de votre part. Microsoft est conscient d’un problème pouvant générer l’envoi de notifications invitant l’utilisateur à renouveler le certificat, même si aucune action n’est requise.
>
>

Azure AD tente d’analyser les métadonnées de fédération et de mettre à jour les certificats de signature de jeton, comme indiqué par ces métadonnées. Pour vérifier la disponibilité de nouveaux certificats, Azure AD interroge les métadonnées de fédération 30 jours avant l’expiration des certificats de signature de jeton.

* Aucune notification n’est envoyée par e-mail ou ne s’affiche dans le Centre d’administration Microsoft 365 si Azure AD réussit à récupérer les nouveaux certificats après interrogation des métadonnées de fédération.
* Si, en revanche, les nouveaux certificats de signature de jetons ne peuvent être récupérés (les métadonnées de fédération étant inaccessibles ou la substitution automatique des certificats étant inactivée), Azure AD enverra une notification par e-mail et affichera également une annonce dans le Centre d’administration Microsoft 365.

![Notification du portail Office 365](./media/how-to-connect-fed-o365-certs/notification.png)

> [!IMPORTANT]
> Si vous utilisez AD FS, vérifiez que vos serveurs possèdent les mises à jour suivantes afin d’éviter les échecs d’authentification pour les problèmes connus ; la continuité des activités est ainsi assurée. Cela limite les problèmes connus du serveur proxy AD FS pour ce renouvellement ainsi que pour les périodes de renouvellement suivantes :
>
> Server 2012 R2 - [Windows Server : correctif cumulatif de mai 2014](https://support.microsoft.com/kb/2955164)
>
> Server 2008 R2 et 2012 [Échec de l’authentification par proxy dans Windows Server 2008 R2 SP1 ou dans Windows Server 2012](https://support.microsoft.com/kb/3094446)
>
>

## <a name="check-if-the-certificates-need-to-be-updated"></a>Vérifiez si les certificats doivent être mis à jour <a name="managecerts"></a>
### <a name="step-1-check-the-autocertificaterollover-state"></a>Étape 1 : Vérifier l’état de AutoCertificateRollover
Sur votre serveur AD FS, ouvrez Powershell. Vérifiez que la valeur de AutoCertificateRollover est définie sur True.

```azurepowershell-interactive
Get-Adfsproperties
```

![AutoCertificateRollover](./media/how-to-connect-fed-o365-certs/autocertrollover.png)

>[!NOTE] 
>Si vous utilisez AD FS 2.0, commencez par exécuter Add-Pssnapin Microsoft.Adfs.Powershell.

### <a name="step-2-confirm-that-ad-fs-and-azure-ad-are-in-sync"></a>Étape 2 : Vérifier que les services AD FS et Azure AD sont synchronisés
Sur votre serveur AD FS, ouvrez l’invite de commandes MSOnline PowerShell et connectez-vous à Azure AD.

> [!NOTE]
> Les cmdlets MSOL font partie du module MSOnline PowerShell.
> Vous pouvez télécharger le module MSOnline PowerShell directement à partir de PowerShell Gallery.
> 
>

```azurepowershell-interactive
Install-Module MSOnline
```

Connectez-vous à Azure AD à l’aide du module MSOnline PowerShell.

```azurepowershell-interactive
Import-Module MSOnline
Connect-MsolService
```

Vérifiez les certificats configurés dans les propriétés d’approbation d’AD FS et d’Azure AD pour le domaine spécifié.

```azurepowershell-interactive
Get-MsolFederationProperty -DomainName <domain.name> | FL Source, TokenSigningCertificate
```

![Get-MsolFederationProperty](./media/how-to-connect-fed-o365-certs/certsync.png)

Si les Thumbprints des deux sorties correspondent, vos certificats sont synchronisés avec Azure AD.

### <a name="step-3-check-if-your-certificate-is-about-to-expire"></a>Étape 3 : Vérifier si votre certificat est sur le point d’expirer
Dans la sortie de Get-MsolFederationProperty ou Get-AdfsCertificate, vérifiez la date sous « Not after ». Si la date indiquée est fixée à moins de 30 jours restants, vous devez agir en conséquence.

| AutoCertificateRollover | Certificats synchronisés avec Azure AD | Les métadonnées de fédération sont accessibles publiquement | Validité | Action |
|:---:|:---:|:---:|:---:|:---:|
| Oui |Oui |Oui |- |Aucune action n'est nécessaire. Voir [Renouveler le certificat de signature de jetons automatiquement](#autorenew). |
| Oui |Non |- |Moins de 15 jours |Renouvelez immédiatement. Voir [Renouveler le certificat de signature de jetons manuellement](#manualrenew). |
| Non |- |- |Moins de 30 jours |Renouvelez immédiatement. Voir [Renouveler le certificat de signature de jetons manuellement](#manualrenew). |

\[-]  N’a pas d’importance

## <a name="renew-the-token-signing-certificate-automatically-recommended"></a>Renouveler le certificat de signature de jetons automatiquement (recommandé) <a name="autorenew"></a>
Vous n’avez pas besoin de suivre de procédure manuelle si les deux affirmations suivantes sont vraies :

* Vous avez déployé le proxy d’application Web, qui permet d’accéder aux métadonnées de fédération à partir de l’extranet.
* Vous utilisez la configuration AD FS par défaut (AutoCertificateRollover est activé).

Vérifiez les points suivants pour confirmer que le certificat peut être mis à jour automatiquement.

**1. La propriété AutoCertificateRollover d’AD FS doit être définie sur True.** Cela indique qu’AD FS génère automatiquement de nouveaux certificats de signature de jetons et de déchiffrement de jeton avant que les anciens certificats n’arrivent à expiration.

**2. Les métadonnées de fédération AD FS sont accessibles publiquement.** Vérifiez que vos métadonnées de fédération sont publiquement accessibles en accédant à l’URL ci-dessous à partir d’un ordinateur relié au réseau Internet public (en dehors du réseau de l’entreprise) :

https://(votre_nom_FS)/federationmetadata/2007-06/federationmetadata.xml

où `(your_FS_name)` est remplacé par le nom d’hôte du service de fédération utilisé par votre organisation, tel que fs.contoso.com.  Si vous êtes parvenu à vérifier la présence de ces deux paramètres, vous n’avez rien d’autre à faire.  

Exemple : `https://fs.contoso.com/federationmetadata/2007-06/federationmetadata.xml`
## <a name="renew-the-token-signing-certificate-manually"></a>Renouveler le certificat de signature de jetons manuellement <a name="manualrenew"></a>
Vous pouvez choisir le renouvellement manuel des certificats de signature de jeton. Par exemple, il se peut que les scénarios fonctionnent mieux pour le renouvellement manuel :

* Les certificats de signature de jeton ne sont pas auto-signés. Le plus souvent, ceci s’explique par le fait que votre organisation gère les certificats AD FS inscrits auprès d’une autorité de certification d’organisation.
* La sécurité réseau ne permet pas de rendre publiques les métadonnées de fédération.

Dans ces scénarios, vous devez mettre à jour votre domaine Microsoft 365 à l’aide de la commande PowerShell Update-MsolFederatedDomain à chaque fois que vous mettez à jour les certificats de signature de jeton.

### <a name="step-1-ensure-that-ad-fs-has-new-token-signing-certificates"></a>Étape 1 : Vérifier qu’AD FS dispose de nouveaux certificats de signature de jetons
**Configuration différente de la configuration par défaut**

Si vous êtes dans une configuration différente de la configuration par défaut d’AD FS, dans laquelle **AutoCertificateRollover** est défini sur **False**, vous utilisez probablement des certificats personnalisés (pas auto-signés). Pour plus d’informations sur le renouvellement des certificats de signature de jetons AD FS, consultez [Exigences en matière de certificats pour les serveurs fédérés](/windows-server/identity/ad-fs/design/certificate-requirements-for-federation-servers).

**Les métadonnées de fédération ne sont pas disponibles publiquement**

En revanche, si **AutoCertificateRollover** est défini sur **True** sans que vos métadonnées de fédération ne soient disponibles publiquement, assurez-vous que les nouveaux certificats de signature de jetons ont été générés par AD FS. Confirmez que vous disposez des nouveaux certificats de signature de jeton en suivant les étapes suivantes :

1. Vérifiez que vous êtes connecté au serveur AD FS principal.
2. Vérifiez les certificats de signature actuels dans AD FS en ouvrant une fenêtre de commande PowerShell et en exécutant la commande suivante :

    PS C:\>Get-ADFSCertificate –CertificateType token-signing

   > [!NOTE]
   > Si vous utilisez AD FS 2.0, vous devez commencer par exécuter Add-Pssnapin Microsoft.Adfs.Powershell.
   >
   >
3. Examinez les certificats répertoriés dans la sortie de la commande. Si AD FS a généré un nouveau certificat, vous devez voir deux certificats dans la sortie : l’un dont la valeur **IsPrimary** est définie sur **True** et dont la date **NotAfter** correspond à 5 jours, et l’autre avec une valeur **IsPrimary** définie sur **False** et une valeur **NotAfter** correspondant à environ un an après la date du jour.
4. Si vous ne voyez qu’un seul certificat et que la date **NotAfter** est définie sur 5 jours, vous devez générer un nouveau certificat.
5. Pour générer un nouveau certificat, exécutez la commande ci-dessous au niveau d’une invite de commande PowerShell : `PS C:\>Update-ADFSCertificate –CertificateType token-signing`.
6. Vérifiez la mise à jour en réexécutant la commande suivante : PS C:\>Get-ADFSCertificate –CertificateType token-signing

Vous devez alors voir apparaître deux certificats, dont l’un présente une date **NotAfter** correspondant à environ un an après la date du jour et dont la valeur **IsPrimary** est définie sur **False**.

### <a name="step-2-update-the-new-token-signing-certificates-for-the-microsoft-365-trust"></a>Étape 2 : Mettre à jour les nouveaux certificats de signature de jetons pour l’approbation de Microsoft 365
Mettez Microsoft 365 à jour avec les nouveaux certificats de signature de jeton devant être utilisés pour l’approbation en suivant la procédure ci-dessous.

1. Ouvrez le Module Microsoft Azure Active Directory pour Windows PowerShell.
2. Exécutez la commande $cred=Get-Credential. Lorsque cette applet de commande vous demande des informations d’identification, tapez vos informations d’identification de compte administrateur de services cloud.
3. Exécutez la commande Connect-MsolService –Credential $cred. Cette applet de commande vous connecte au service cloud. Avant d’exécuter l’une des applets de commande supplémentaires installées par l’outil, vous devez créer un contexte qui vous connecte au service cloud.
4. Si vous exécutez ces commandes sur un ordinateur autre que le serveur de fédération principal AD FS, exécutez Set-MSOLAdfscontext -Computer &lt;serveur principal AD FS&gt;, où &lt;serveur principal AD FS&gt; est le nom de domaine complet interne du serveur AD FS principal. Cette applet de commande crée un contexte qui vous connecte à AD FS.
5. Exécutez Update-MSOLFederatedDomain –DomainName &lt;domaine&gt;. Cette applet de commande met à jour les paramètres d’AD FS dans le service cloud et configure la relation d’approbation entre les deux.

> [!NOTE]
> Si vous avez besoin de prendre en charge plusieurs domaines de premier niveau, par exemple contoso.com et fabrikam.com, vous devez utiliser le commutateur **SupportMultipleDomain** avec les applets de commande. Pour plus d’informations, consultez [Prise en charge de plusieurs domaines de premier niveau](how-to-connect-install-multiple-domains.md).
>


## <a name="repair-azure-ad-trust-by-using-azure-ad-connect"></a>Réparer l’approbation Azure AD à l’aide d’Azure AD Connect <a name="connectrenew"></a>
Si vous avez configuré votre batterie de serveurs AD FS et l’approbation Azure AD à l’aide d’Azure AD Connect, vous pouvez utiliser Azure AD Connect afin de vérifier si vous devez agir ou non pour vos certificats de signature de jeton. Si vous devez renouveler les certificats, vous pouvez le faire à l’aide d’Azure AD Connect.

Pour plus d’informations, consultez [Réparation de l’approbation](how-to-connect-fed-management.md).

## <a name="ad-fs-and-azure-ad-certificate-update-steps"></a>Étapes de mise à jour des certificats AD FS et Azure AD
Les certificats de signature de jetons sont des certificats X509 standard qui sont utilisés pour signer de manière sécurisée tous les jetons émis par le serveur de fédération. Les certificats de déchiffrement de jetons sont des certificats X509 standard qui sont utilisés pour déchiffrer les jetons entrants. 

Par défaut, AD FS est configuré pour générer automatiquement des certificats de signature et de déchiffrement de jetons, lors de la configuration initiale et lorsque les certificats approchent de leur date d’expiration.

Azure AD tente de récupérer un nouveau certificat à partir de vos métadonnées de service de fédération 30 jours avant l’expiration du certificat actuel. Si aucun nouveau certificat n’est disponible à ce moment-là, Azure AD va continuer à surveiller les métadonnées de manière quotidienne. Dès que le nouveau certificat est disponible dans les métadonnées, les paramètres de fédération du domaine sont mis à jour avec les informations du nouveau certificat. Vous pouvez utiliser `Get-MsolDomainFederationSettings` pour voir si le nouveau certificat est présent dans NextSigningCertificate ou SigningCertificate.

Pour plus d’informations sur les certificats de signature de jetons dans AD FS, consultez [Obtenir et configurer des certificats de signature et de déchiffrement de jetons pour AD FS](/windows-server/identity/ad-fs/operations/configure-ts-td-certs-ad-fs).
