---
title: Connexion par clé de sécurité sans mot de passe à des ressources locales (préversion) – Azure Active Directory
description: Découvrez comment activer la connexion par clé de sécurité sans mot de passe à des ressources locales en utilisant Azure Active Directory (préversion)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 03/09/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: librown, aakapo
ms.collection: M365-identity-device-management
ms.openlocfilehash: e1459442f7d99d1de88a685eed34493da530c1a4
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743477"
---
# <a name="enable-passwordless-security-key-sign-in-to-on-premises-resources-with-azure-active-directory-preview"></a>Activer la connexion par clé de sécurité sans mot de passe à des ressources locales avec Azure Active Directory (préversion)

Ce document se concentre sur l’activation de l’authentification sans mot de passe à des ressources locales pour des environnements composés d’appareils Windows 10 de types **Joint à Azure AD** et **Joint à une version hybride d’Azure AD**. Cette fonctionnalité fournit une authentification unique (SSO) transparente auprès de ressources locales en utilisant des clés de sécurité compatibles Microsoft.

> [!NOTE]
> Les clés de sécurité FIDO2 sont une fonctionnalité d’évaluation publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="sso-to-on-premises-resources-using-fido2-keys"></a>Authentification unique auprès de ressources locales à l’aide de clés FIDO2

Azure Active Directory (AD) peut émettre des Kerberos Ticket Granting Tickets (TGTs) pour un ou plusieurs de vos domaines Active Directory. Cette fonctionnalité permet aux utilisateurs de se connecter à Windows avec des informations d’identification modernes telles que des clés de sécurité FIDO2 pour accéder à des ressources Active Directory traditionnelles. Les tickets de service Kerberos et l’autorisation continuent d’être contrôlés par vos contrôleurs de domaine Active Directory locaux.

Un objet serveur Kerberos Azure AD est créé dans votre Active Directory local, puis publié en toute sécurité sur Azure Active Directory. L’objet n’est associé à aucun serveur physique. Il s’agit simplement d’une ressource qu’Azure Active Directory peut utiliser afin de générer des TGT Kerberos pour votre domaine Active Directory.

![Obtention d’un ticket TGT (Ticket Granting Ticket) auprès d’Azure AD et d’AD DS](./media/howto-authentication-passwordless-on-premises/fido2-ticket-granting-ticket-exchange-process.png)

1. L’utilisateur se connecte à son appareil Windows 10 avec une clé de sécurité FIDO2 et s’authentifie auprès d’Azure AD.
1. Azure AD recherche dans l’annuaire une clé de serveur Kerberos correspondant au domaine AD local de l’utilisateur.
   1. Azure AD génère un TGT Kerberos pour le domaine AD local de l’utilisateur. Le TGT contient uniquement le SID de l’utilisateur. Aucune donnée d’autorisation n’y est incluse.
1. Le TGT est renvoyé au client avec son jeton d’actualisation principal (PRT, Primary Refresh Token) Azure AD.
1. L’ordinateur du client contacte un contrôleur de domaine Active Directory local et échange le TGT partiel contre un TGT entièrement formé.
1. L’ordinateur du client disposant à présent d’un PRT Azure AD et d’un TGT Active Directory complet, il peut accéder aux ressources cloud et locales.

## <a name="requirements"></a>Spécifications

Les organisations doivent effectuer les étapes permettant d’[activer la connexion par clé de sécurité sans mot de passe à des appareils Windows 10 (préversion)](howto-authentication-passwordless-security-key.md) avant d’effectuer les étapes décrites dans cet article.

Les organisations doivent également présenter la configuration logicielle suivante.

- Les appareils doivent exécuter Windows 10 version 2004 ou ultérieure.
- Vous devez disposer de la version 1.4.32.0 ou ultérieure d’[Azure AD Connect](../hybrid/how-to-connect-install-roadmap.md#install-azure-ad-connect).
  - Pour plus d’informations sur les options d’authentification hybride Azure AD disponibles, consultez [Choisir la méthode d’authentification adaptée à votre solution d’identité hybride Azure Active Directory](../hybrid/choose-ad-authn.md) et [Sélectionner le type d’installation à utiliser pour Azure AD Connect](../hybrid/how-to-connect-install-select-installation.md).
- Les correctifs logiciels suivants doivent être installés sur vos contrôleurs de domaine Windows Server :
    - Pour Windows Server 2016 : https://support.microsoft.com/help/4534307/windows-10-update-kb4534307
    - Pour Windows Server 2019 : https://support.microsoft.com/help/4534321/windows-10-update-kb4534321

### <a name="supported-scenarios"></a>Scénarios pris en charge

Le scénario prend en charge l’authentification unique (SSO) dans les deux scénarios suivants :

- Pour les ressources cloud telles que Microsoft 365 et d’autres applications compatibles SAML.
- Pour les ressources locales et l’Authentification Windows intégrée sur les sites web. Les ressources peuvent inclure des sites web et des sites SharePoint qui requièrent une authentification IIS et/ou des ressources qui utilisent l’authentification NTLM.

### <a name="unsupported-scenarios"></a>Scénarios non pris en charge

Les scénarios suivants ne sont pas pris en charge :

- Déploiement de Windows Server joint à un domaine Active Directory Domain Services (AD DS) [appareils locaux uniquement].
- Scénarios RDP, VDI et Citrix utilisant une clé de sécurité.
- S/MIME utilisant une clé de sécurité.
- Identification utilisant une clé de sécurité.
- Connexion à un serveur à l’aide d’une clé de sécurité.

## <a name="create-kerberos-server-object"></a>Créer un objet serveur Kerberos

Les administrateurs utilisent les outils PowerShell de leur serveur Azure AD Connect pour créer un objet serveur Kerberos Azure AD dans leur répertoire local. Exécutez les étapes suivantes dans chaque domaine et forêt de votre organisation contenant des utilisateurs Azure AD :

1. Mettez à niveau vers la dernière version d’Azure AD Connect. Les instructions partent du principe que vous avez déjà configuré Azure AD Connect pour prendre en charge votre environnement hybride.
1. Sur le serveur Azure AD Connect, ouvrez une invite PowerShell avec élévation de privilèges, puis accédez à `C:\Program Files\Microsoft Azure Active Directory Connect\AzureADKerberos\`.
1. Exécutez les commandes PowerShell suivantes pour créer un objet serveur Kerberos Azure AD dans votre domaine Active Directory local et votre locataire Azure Active Directory.

> [!NOTE]
> Remplacez `contoso.corp.com` dans l’exemple suivant par le nom de votre domaine Active Directory local.

```powerShell
Import-Module ".\AzureAdKerberos.psd1"

# Specify the on-premises Active Directory domain. A new Azure AD
# Kerberos Server object will be created in this Active Directory domain.
$domain = "contoso.corp.com"

# Enter an Azure Active Directory global administrator username and password.
$cloudCred = Get-Credential

# Enter a domain administrator username and password.
$domainCred = Get-Credential

# Create the new Azure AD Kerberos Server object in Active Directory
# and then publish it to Azure Active Directory.
Set-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred
```

### <a name="viewing-and-verifying-the-azure-ad-kerberos-server"></a>Affichage et vérification du serveur Kerberos Azure AD

Vous pouvez afficher et vérifier le serveur Kerberos Azure AD que vous venez de créer à l’aide de la commande suivante :

```powerShell
Get-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred
```

Cette commande génère les propriétés du serveur Kerberos Azure AD. Vous pouvez examiner les propriétés pour vérifier que tout est en ordre.

| Propriété | Description |
| --- | --- |
| id | ID unique de l’objet contrôleur de domaine AD DS. Cet ID est parfois appelé « emplacement » ou « ID de branche ». |
| DomainDnsName | Nom de domaine DNS du domaine Active Directory. |
| ComputerAccount | Objet compte d’ordinateur de l’objet serveur Kerberos Azure AD (le contrôleur de domaine). |
| UserAccount | Objet compte d’utilisateur désactivé contenant la clé de chiffrement du TGT du serveur Kerberos Azure AD. Le nom de domaine de ce compte est `CN=krbtgt_AzureAD,CN=Users,<Domain-DN>`. |
| KeyVersion | Version de clé de la clé de chiffrement du TGT du serveur Kerberos Azure AD. La version est attribuée lors de la création de la clé. La version est ensuite incrémentée à chaque rotation de la clé. Les incréments sont basés sur des métadonnées de réplication et sont probablement supérieurs à un. Par exemple, la *KeyVersion* initiale pourrait être *192272*. À la première rotation de la clé, la version peut passer à *212621*. L’élément important à vérifier est que la *KeyVersion* de l’objet local et la *CloudKeyVersion* de l’objet cloud sont identiques. |
| KeyUpdatedOn | Date et heure auxquelles la clé de chiffrement du TGT du serveur Kerberos Azure AD a été mise à jour ou créée. |
| KeyUpdatedFrom | Contrôleur de domaine où la clé de chiffrement du TGT du serveur Kerberos Azure AD a été mise à jour pour la dernière fois. |
| CloudId | ID de l’objet Azure AD. Doit correspondre à l’ID au-dessus. |
| CloudDomainDnsName | *DomainDnsName* de l’objet Azure AD. Doit correspondre à la valeur *DomainDnsName* ci-dessus. |
| CloudKeyVersion | *KeyVersion* de l’objet Azure AD. Doit correspondre à la valeur *KeyVersion* ci-dessus. |
| CloudKeyUpdatedOn | *KeyUpdatedOn* de l’objet Azure AD. Doit correspondre à la valeur *KeyUpdatedOn* ci-dessus. |

### <a name="rotating-the-azure-ad-kerberos-server-key"></a>Rotation de la clé de serveur Kerberos Azure AD

Les clés krbtgt de chiffrement du serveur Kerberos Azure AD doivent faire l’objet d’une rotation régulière. Il est recommandé de suivre la même planification que celle utilisée pour la rotation de toutes les autres clés krbtgt du contrôleur de domaine Active Directory.

> [!WARNING]
> D’autres outils peuvent effectuer une rotation des clés krbtgt. Toutefois, vous devez utiliser les outils mentionnés dans ce document pour opérer la rotation des clés krbtgt de votre serveur Kerberos Azure AD. Cela garantit que les clés sont mises à jour tant dans l’AD local que dans Azure AD.

```powerShell
Set-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred -RotateServerKey
```

### <a name="removing-the-azure-ad-kerberos-server"></a>Suppression du serveur Kerberos Azure AD

Si vous souhaitez annuler le scénario et supprimer le serveur Kerberos Azure AD tant de l’Active Directory local que d’Azure Active Directory, exécutez la commande suivante :

```powerShell
Remove-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred
```

### <a name="multi-forest-and-multi-domain-scenarios"></a>Scénarios à plusieurs forêts et plusieurs domaines

L’objet serveur Kerberos Azure AD est représenté dans Azure AD en tant qu’objet *KerberosDomain*. Chaque domaine Active Directory local est représenté en tant qu’objet *KerberosDomain* unique dans Azure AD.

Imaginons, par exemple, que votre organisation possède une forêt Active Directory avec deux domaines, `contoso.com` et `fabrikam.com`. Si vous choisissez d’autoriser Azure AD à émettre des TGT Kerberos pour l’ensemble de la forêt, il y aura deux objets *KerberosDomain* dans Azure AD. Un objet *KerberosDomain* pour `contoso.com`, et un autre pour `fabrikam.com`. Si vous avez plusieurs forêts Active Directory, il existe un objet *KerberosDomain* pour chaque domaine dans chaque forêt.

Vous devez exécuter les étapes permettant de [créer un objet serveur Kerberos](#create-kerberos-server-object) dans chaque domaine et forêt de votre organisation contenant des utilisateurs Azure AD.

## <a name="known-behavior"></a>Comportement connu

La connexion avec FIDO est bloquée si votre mot de passe a expiré. Le comportement attendu de l’utilisateur et qu’il réinitialise son mot de passe avant de se connecter à l’aide de FIDO.

## <a name="troubleshooting-and-feedback"></a>Résolution des problèmes de commentaires

Si vous souhaitez partager des commentaires ou si vous rencontrez des problèmes lors de l’évaluation de cette fonctionnalité, partagez vos remarques via l’application Hub de commentaires Windows en procédant comme suit :

1. Lancez le **concentrateur de commentaires** et assurez-vous que vous êtes connecté.
1. Classez vos commentaires dans les catégories suivantes avant de les envoyer :
   - Catégorie : Sécurité et confidentialité
   - Sous-catégorie : FIDO
1. Pour capturer des journaux, utilisez l’option **Recréer mon problème**

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="does-this-work-in-my-on-premises-environment"></a>Cela fonctionne-t-il dans mon environnement local ?

Cette fonctionnalité ne fonctionne pas pour un environnement Active Directory Domain Services (AD DS) entièrement local.

### <a name="my-organization-requires-two-factor-authentication-to-access-resources-what-can-i-do-to-support-this-requirement"></a>Mon organisation requiert une authentification à deux facteurs pour accéder aux ressources. Que puis-je faire pour prendre en charge cette exigence ?

Les clés de sécurité sont disponibles dans différents facteurs de forme. Contactez le fabricant de l’appareil pour savoir comment le paramétrer afin d’utiliser un code PIN ou un accès biométrique comme deuxième facteur.

### <a name="can-admins-set-up-security-keys"></a>Les administrateurs peuvent-ils configurer des clés de sécurité ?

Nous nous efforçons actuellement d’assurer la disponibilité générale de cette fonctionnalité.

### <a name="where-can-i-go-to-find-compliant-security-keys"></a>Où puis-je trouver des clés de sécurité conformes ?

[Clés de sécurité FIDO2](concept-authentication-passwordless.md#fido2-security-keys)

### <a name="what-do-i-do-if-i-lose-my-security-key"></a>Que dois-je faire si je perds ma clé de sécurité ?

Vous pouvez supprimer des clés de sécurité à partir du Portail Microsoft Azure, en accédant à la page **Informations relatives à la sécurité** et en supprimant les clés.

### <a name="im-not-able-to-use-fido-immediately-after-i-create-a-hybrid-azure-ad-joined-machine"></a>Je ne parviens pas à utiliser FIDO immédiatement après avoir créé une machine jointe à Azure AD Hybride.

Si vous effectuez une nouvelle installation de machine jointe à Azure AD Hybride, après le processus de jonction de domaine et de redémarrage, vous devez vous connecter avec un mot de passe et attendre la synchronisation de la stratégie avant de pouvoir utiliser FIDO pour vous connecter.

- Vérifiez votre état actuel en saisissant `dsregcmd /status` dans une fenêtre de commande, puis en contrôlant que *AzureAdJoined* et *DomainJoined* indiquent *YES*.
- Ce délai est une limitation connue des appareils joints à un domaine et qui n’est pas spécifique à FIDO.

### <a name="im-unable-to-get-sso-to-my-ntlm-network-resource-after-signing-in-with-fido-and-get-a-credential-prompt"></a>Je ne parviens pas à effectuer l’authentification unique auprès de ma ressource réseau NTLM après m’être connecté avec FIDO, et je reçois une invite à entrer des informations d’identification.

Assurez-vous qu’un nombre suffisant de contrôleurs de domaine sont corrigés pour répondre dans les temps afin de servir votre demande de ressource. Pour vérifier si vous pouvez voir un contrôleur de domaine exécutant la fonctionnalité, consultez la sortie de la commande `nltest /dsgetdc:contoso /keylist /kdc`.

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus l’approche sans mot de passe](concept-authentication-passwordless.md)
