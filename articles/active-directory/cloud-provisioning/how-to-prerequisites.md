---
title: Prérequis pour le provisionnement cloud Azure AD Connect dans Azure AD
description: Cet article décrit les conditions préalables et la configuration matérielle requise pour l’approvisionnement du cloud.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/11/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1f0c94ba6fb9ee5ab019458043095271123e325e
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97671010"
---
# <a name="prerequisites-for-azure-ad-connect-cloud-provisioning"></a>Prérequis pour le provisionnement cloud Azure AD Connect
Cet article fournit des conseils sur la façon de choisir et d’utiliser l’approvisionnement cloud Azure Active Directory (Azure AD) Connect en tant que solution d’identité.

## <a name="cloud-provisioning-agent-requirements"></a>Conditions requises de l’agent de provisionnement cloud
Vous avez besoin des éléments suivants pour utiliser le provisionnement cloud Azure AD Connect :

- Les informations d’identification Administrateur de domaine ou Administrateur d’entreprise pour créer le compte de service managé de groupe (gMSA) Azure AD Connect Cloud Sync pour exécuter le service agent. 
- Un compte d’administrateur d’identité hybride pour votre locataire Azure AD qui n’est pas un utilisateur invité.
- Un serveur local pour l’agent de provisionnement avec Windows 2012 R2 ou ultérieur.  Ce serveur doit être un serveur de niveau 0 basé sur le [modèle de niveau d’administration Active Directory](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material).
- Des configurations de pare-feu locales.

## <a name="group-managed-service-accounts"></a>Group Managed Service Accounts
Un compte de service managé de groupe est un compte de domaine managé qui fournit la gestion automatique des mots de passe, la gestion simplifiée du nom de principal du service (SPN), la possibilité de déléguer la gestion à d’autres administrateurs et cette fonctionnalité s’étend sur plusieurs serveurs.  Azure AD Connect Cloud Sync prend en charge et utilise un gMSA pour l’exécution de l’agent.  Vous serez invité à fournir des informations d’identification d’administration lors de l’installation, afin de créer ce compte.  Le compte s’affiche sous la forme (domain\provAgentgMSA$).  Pour plus d’informations sur un gMSA, consultez [Comptes de service managés de groupe](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) 

### <a name="prerequisites-for-gmsa"></a>Prérequis pour gMSA :
1.  Le schéma Active Directory dans la forêt du domaine gMSA doit être mis à jour vers Windows Server 2012
2.  Les [modules RSAT PowerShell](/windows-server/remote/remote-server-administration-tools) sur un contrôleur de domaine
3.  Au moins un contrôleur de domaine dans le domaine doit exécuter Windows Server 2012.
4.  Un serveur joint à un domaine sur lequel l’agent est en cours d’installation doit être configuré avec Windows Server 2012 ou une version ultérieure.

Pour connaître les étapes de la mise à niveau d’un agent existant afin d’utiliser un compte gMSA, consultez [Comptes de service managés de groupe](how-to-install.md#group-managed-service-accounts).

### <a name="in-the-azure-active-directory-admin-center"></a>Dans le Centre d’administration Azure Active Directory

1. Créez un compte d’administrateur d’identité hybride « cloud uniquement » dans votre locataire Azure AD. De cette façon, vous pouvez gérer la configuration de votre locataire si vos services locaux venaient à échouer ou ne plus être disponibles. Découvrez comment [ajouter un compte d’administrateur d’identité hybride de type cloud uniquement](../fundamentals/add-users-azure-active-directory.md). Cette étape est essentielle si vous voulez éviter de vous retrouver en dehors de votre locataire.
1. Ajoutez un ou plusieurs [noms de domaine personnalisés](../fundamentals/add-custom-domain.md) à votre locataire Azure AD. Vos utilisateurs peuvent se connecter à l’aide de l’un de ces noms de domaine.

### <a name="in-your-directory-in-active-directory"></a>Dans votre annuaire dans Azure Active Directory

Exécutez l’[outil IdFix](/office365/enterprise/prepare-directory-attributes-for-synch-with-idfix) afin de préparer les attributs d’annuaire pour la synchronisation.

### <a name="in-your-on-premises-environment"></a>Dans votre environnement local

1. Identifiez un serveur hôte joint à un domaine exécutant Windows Server 2012 R2 ou ultérieur, avec au minimum 4 Go de RAM et .NET 4.7.1 + Runtime.

    >[!NOTE]
    > N’oubliez pas que la définition d’un filtre d’étendue occasionne un coût de mémoire sur le serveur hôte.  Si aucun filtre d’étendue n’est utilisé, il n’y a aucun coût de mémoire supplémentaire. Le minimum de 4 Go prendra en charge la synchronisation de jusqu’à 12 unités d’organisation définies dans le filtre d’étendue. Si vous devez synchroniser des unités d’organisation supplémentaires, vous devez augmenter la quantité minimale de mémoire. Utilisez le tableau suivant comme guide :
    >
    >
    > | Nombre d’unités d’organisation dans le filtre d’étendue| Mémoire minimale nécessaire|
    > | --- | --- |
    > | 12 | 4 Go |
    > | 18 | 5,5 GO|
    > | 28 | > 10 GO|

2. La stratégie d’exécution de PowerShell sur le serveur local doit être définie sur Undefined ou sur RemoteSigned.

3. S’il existe un pare-feu entre vos serveurs et Azure AD, configurez les éléments suivants :
    - Assurez-vous que les agents peuvent effectuer des requêtes *sortantes* sur Azure AD sur les ports suivants :

      | Numéro de port | Utilisation |
      | --- | --- |
      | **80** | Télécharge les listes de révocation de certificats lors de la validation du certificat TLS/SSL.  |
      | **443** | Gère toutes les communications sortantes avec le service. |
      |**8082**|Requis pour l’installation et si vous souhaitez configurer l’API d’administration HIS.  Ce port peut être supprimé une fois que l’agent est installé et que vous n’envisagez pas d’utiliser l’API.   |
      | **8080** (facultatif) | Les agents signalent leur état toutes les 10 minutes sur le port 8080, si le port 443 n’est pas disponible. Cet état est affiché sur le portail Azure AD. |

    - Si votre pare-feu applique les règles en fonction des utilisateurs d’origine, ouvrez ces ports au trafic provenant des services Windows exécutés en tant que service réseau.
    - Si votre pare-feu ou proxy vous permet de spécifier des suffixes approuvés, ajoutez des connexions à \*.msappproxy.net et \*.servicebus.windows.net. Dans le cas contraire, autorisez l’accès aux [plages d’adresses IP du centre de données Azure](https://www.microsoft.com/download/details.aspx?id=41653), qui sont mises à jour chaque semaine.
    - Vos agents doivent accéder à login.windows.net et à login.microsoftonline.com pour l’inscription initiale. Par conséquent, ouvrez également votre pare-feu pour ces URL.
    - Pour valider le certificat, débloquez les URL suivantes : mscrl.microsoft.com:80, crl.microsoft.com:80, ocsp.msocsp.com:80 et www\.microsoft.com:80. Ces URL sont utilisées pour la validation de certificat avec d’autres produits Microsoft : elles seront peut-être déjà débloquées.

    >[!NOTE]
    > L’installation de l’agent de provisionnement cloud sur Windows Server Core n’est pas prise en charge.

### <a name="additional-requirements"></a>Autres conditions requises

- [Microsoft .NET Framework 4.7.1](https://www.microsoft.com/download/details.aspx?id=56116) 

#### <a name="tls-requirements"></a>Exigences relatives à TLS

> [!NOTE]
> Le protocole TLS (Transport Layer Security) est un protocole qui fournit des communications sécurisées. La modification des paramètres TLS affecte l’ensemble de la forêt. Pour plus d’informations, consultez [Mise à jour pour activer TLS 1.1 et TLS 1.2 en tant que protocoles sécurisés par défaut dans WinHTTP sur Windows](https://support.microsoft.com/help/3140245/update-to-enable-tls-1-1-and-tls-1-2-as-default-secure-protocols-in-wi).

Le protocole TLS 1.2 doit être activé sur le serveur Windows qui hébergera l’agent de provisionnement cloud Azure AD Connect avant son installation.

Pour activer TLS 1.2, procédez comme suit.

1. Définissez les clés de Registre suivantes :

    ```
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2]
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319] "SchUseStrongCrypto"=dword:00000001
    ```

1. Redémarrez le serveur.

## <a name="known-limitations"></a>Limitations connues

Les limitations connues sont les suivantes :

### <a name="delta-synchronization"></a>Synchronisation d’écart

- Le filtre d’étendue du groupe pour la synchronisation delta ne prend pas en charge plus de 1 500 membres.
- Lorsque vous supprimez un groupe utilisé dans le cadre d’un filtre d’étendue de groupe, les utilisateurs qui sont membres du groupe ne sont pas supprimés. 
- Lorsque vous renommez l’unité d’organisation ou le groupe qui se trouve dans l’étendue, la synchronisation delta ne supprime pas les utilisateurs.

### <a name="provisioning-logs"></a>Journaux de provisionnement
- Les journaux d’approvisionnement ne font pas clairement la différence entre les opérations de création et de mise à jour.  Vous pouvez voir une opération de création pour une mise à jour et une opération de mise à jour pour une création.

### <a name="group-re-naming-or-ou-re-naming"></a>Renommer le groupe ou renommer l’unité d’organisation
- Si vous renommez un groupe ou une unité d’organisation dans Active Directory dans le cadre d’une configuration donnée, le travail d’approvisionnement cloud ne pourra pas reconnaître le changement de nom dans Active Directory. Le travail ne sera pas mis en quarantaine et restera sain.


## <a name="next-steps"></a>Étapes suivantes 

- [Présentation du provisionnement](what-is-provisioning.md)
- [Présentation du provisionnement cloud Azure AD Connect](what-is-cloud-provisioning.md)