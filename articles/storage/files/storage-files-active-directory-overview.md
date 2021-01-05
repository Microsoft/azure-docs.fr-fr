---
title: Vue d’ensemble – Autorisation basée sur l’identité Azure Files
description: Azure Files prend en charge l’authentification basée sur l’identité sur SMB (Server Message Block) au travers d’Azure Active Directory Domain Services (AD DS) et d’Active Directory. Vos machines virtuelles Windows jointes à un domaine peuvent alors accéder aux partages de fichiers Azure en utilisant les informations d’identification Azure AD.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: conceptual
ms.date: 05/29/2020
ms.author: rogarana
ms.openlocfilehash: 0493b66928d944e251e6fe4abda4f1d3d8fb7e23
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94695892"
---
# <a name="overview-of-azure-files-identity-based-authentication-options-for-smb-access"></a>Vue d’ensemble des options d’authentification basée sur l’identité Azure Files pour l’accès SMB
[!INCLUDE [storage-files-aad-auth-include](../../../includes/storage-files-aad-auth-include.md)]

Pour savoir comment activer l’authentification Active Directory Domain Services locale pour les partages de fichiers Azure, voir [Activer l’authentification Active Directory Domain Services en local sur SMB pour les partages de fichiers Azure](storage-files-identity-auth-active-directory-enable.md).

Pour savoir comment activer l’authentification Azure AD DS pour les partages de fichiers Azure, voir [Activer l’authentification Azure Active Directory Domain Services sur Azure Files](storage-files-identity-auth-active-directory-domain-service-enable.md).

## <a name="glossary"></a>Glossaire 
Vous devez comprendre certains termes clés relatifs à l’authentification Azure AD Domain Service sur SMB pour les partages de fichiers Azure :

-   **Authentification Kerberos**

    Kerberos est un protocole d’authentification utilisé pour vérifier l’identité d’un utilisateur ou d’un hôte. Pour plus d’informations sur Kerberos, consultez [Vue d’ensemble de l’authentification Kerberos](/windows-server/security/kerberos/kerberos-authentication-overview).

-  **Protocole SMB (Server Message Block)**

    SMB est un protocole de partage de fichier réseau standard. SMB est aussi appelé Common Internet File System ou CIFS. Pour plus d’informations sur SMB, consultez [Vue d’ensemble du protocole SMB et du protocole CIFS de Microsoft](/windows/desktop/FileIO/microsoft-smb-protocol-and-cifs-protocol-overview).

-   **Azure Active Directory (Azure AD)**

    Azure Active Directory (Azure AD) est le service cloud de gestion des identités et des annuaires multilocataire de Microsoft. Azure AD associe des services d’annuaires principaux, la gestion de l’accès aux applications et la protection des identités dans une même solution. Les machines virtuelles Windows jointes à Azure AD ne peuvent pas accéder aux partages de fichiers Azure à l’aide de vos informations d’identification Azure AD. Pour plus d’informations, consultez [Qu’est-ce qu’Azure Active Directory ?](../../active-directory/fundamentals/active-directory-whatis.md)

-   **Azure Active Directory Domain Services (Azure AD DS)**

    Azure AD DS fournit des services de domaine gérés, comme la jonction de domaine, les stratégies de groupe, le protocole LDAP et l’authentification Kerberos/NTLM. Ces services sont entièrement compatibles avec Active Directory Domain Services. Pour plus d’informations, consultez [Azure Active Directory Domain Services](../../active-directory-domain-services/overview.md).

- **Active Directory Domain Services (AD DS) en local**

    L’intégration d’Active Directory Domain Services (AD DS) local à Azure Files fournit les méthodes permettant de stocker des données d’annuaire tout en les mettant à la disposition des utilisateurs et administrateurs du réseau. La sécurité est intégrée avec AD DS par le biais de l’authentification d’ouverture de session et du contrôle d’accès aux objets de l’annuaire. Avec une simple ouverture de session réseau, les administrateurs peuvent gérer les données et l’organisation de l’annuaire au sein de leur réseau, et les utilisateurs du réseau autorisés peuvent accéder aux ressources n’importe où sur le réseau. AD DS est couramment adopté par des entreprises dans des environnements locaux, et les informations d’identification AD DS sont utilisées comme identité pour le contrôle d’accès. Pour plus d’informations, voir [Présentation des services de domaine Active Directory](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview).

-   **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**

    Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) permet une gestion des accès affinée pour Azure. Grâce à Azure RBAC, vous pouvez gérer l’accès aux ressources en accordant aux utilisateurs les autorisations minimales dont ils ont besoin pour effectuer leur travail. Pour plus d’informations sur Azure RBAC, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (Azure RBAC) ?](../../role-based-access-control/overview.md)

## <a name="common-use-cases"></a>Cas d’utilisation courants

L’authentification basée sur l’identité et la prise en charge des listes de contrôle d’accès Windows sur Azure Files sont optimisées pour les cas d’utilisation suivants :

### <a name="replace-on-premises-file-servers"></a>Remplacer des serveurs de fichiers locaux

L’abandon et le remplacement de serveurs de fichiers locaux éparpillés est un problème courant que chaque entreprise rencontre dans son parcours de modernisation informatique. Les partages de fichiers Azure avec une authentification AD DS en local sont les plus adaptés pour la migration de données vers Azure Files. Une migration complète vous permet de tirer parti des avantages de la haute disponibilité et de l’extensibilité, tout en minimisant les modifications côté client. Elle offre une expérience de migration transparente aux utilisateurs finaux qui peuvent ainsi continuer à accéder à leurs données avec les mêmes informations d’identification à l’aide de leurs machines jointes au domaine existantes.

### <a name="lift-and-shift-applications-to-azure"></a>Migration lift-and-shift des applications vers Azure

Lorsque vous effectuez une migration lift-and-shift d’applications vers le cloud, vous souhaitez conserver le même modèle d’authentification pour vos données. Du fait que nous étendons l’expérience de contrôle d’accès basée sur l’identité aux partages de fichiers Azure, il n’est plus nécessaire de modifier vos applications en méthodes d’authentification modernes et d’accélérer l’adoption du cloud. Les partages de fichiers Azure offrent la possibilité d’effectuer une intégration à Azure AD DS ou AD DS local pour l’authentification. Si votre plan doit être 100% de natif du cloud et réduire les efforts de gestion des infrastructures cloud, Azure AD DS serait plus approprié en tant que service de domaine entièrement managé. Si vous avez besoin d’une compatibilité complète avec les fonctionnalités d’AD DS, vous pouvez envisager d’étendre votre environnement AD DS au cloud en auto-hébergeant des contrôleurs de domaine sur des machines virtuelles. Dans les deux cas, nous offrons la possibilité de choisir les services de domaine adaptés à vos besoins métier.

### <a name="backup-and-disaster-recovery-dr"></a>Sauvegarde et récupération d’urgence (DR)

Si vous conservez votre stockage de fichiers principal localement, les partages de fichiers Azure peuvent servir de stockage idéal pour la sauvegarde ou la récupération d’urgence, afin d’améliorer la continuité des activités. Vous pouvez utiliser des partages de fichiers Azure pour sauvegarder vos données à partir de serveurs de fichiers existants tout en conservant des listes de contrôle d’accès discrétionnaire (DACL) Windows. Pour les scénarios de récupération d’urgence, vous pouvez configurer une option d’authentification pour prendre en charge l’application d’un contrôle d’accès approprié au moment du basculement.

## <a name="supported-scenarios"></a>Scénarios pris en charge

Le tableau suivant récapitule les scénarios d’authentification de partages de fichiers Azure pris en charge pour Azure AD DS et AD DS local. Nous vous recommandons de sélectionner le service de domaine que vous avez adopté pour l’intégration de votre environnement client avec Azure Files. Si AD DS est déjà configuré localement ou sur Azure, et que vos appareils sont joints à votre domaine AD, vous devez choisir d’utiliser AD DS pour l’authentification des partages de fichiers Azure. De même, si vous avez déjà adopté Azure AD DS, vous devez l’utiliser pour l’authentification auprès des partages de fichiers Azure.


|Authentification Azure AD DS  | Authentification AD DS locale  |
|---------|---------|
|Des machines Windows jointes à AD DS peuvent accéder à des partages de fichiers Azure avec des informations d’identification Azure AD sur SMB.     |Des machines Windows jointes à un domaine Azure AD DS ou AD DS local peuvent accéder à des partages de fichiers Azure avec des informations d’identification Active Directory locales synchronisées avec Azure AD via SMB. Votre client doit pouvoir visualiser votre instance AD DS.        |

### <a name="restrictions"></a>Restrictions

- L’authentification Azure AD DS et AD DS en local ne prend pas en charge l’authentification par rapport à des comptes d’ordinateur. Vous pouvez envisager d’utiliser un compte d’ouverture de session de service à la place.
- Ni l’authentification Azure AD DS, ni l’authentification AD DS locale n’est prise en charge par rapport aux appareils joints à Azure AD ou aux appareils inscrits auprès d’Azure AD.
- Les partages de fichiers Azure prennent uniquement en charge l’authentification basée sur l’identité par rapport à l’un des services de domaine suivants, [Azure Active Directory Domain Services (Azure AD DS)](#azure-ad-ds) ou [Active Directory Domain Services (AD DS) local](#ad-ds).
- Aucune méthode d’authentification basée sur l’identité n’est prise en charge avec le système NFS (Network File System), qui est en préversion.

## <a name="advantages-of-identity-based-authentication"></a>Avantages de l’authentification basée sur l’identité
L’authentification basée sur l’identité pour Azure Files offre plusieurs avantages par rapport à l’utilisation de l’authentification par clé partagée :

-   **Étendre l’expérience d’accès au partage de fichiers traditionnel basé sur l’identité au cloud avec AD DS en local et Azure AD DS**  
    Si vous prévoyez d’effectuer une migration « lift-and-shift » de votre application vers le cloud en remplaçant les serveurs de fichiers classiques par des partages de fichiers Azure, votre application peut s’authentifier avec des informations d’identification AD DS en local ou Azure AD DS pour accéder aux données de fichier. Le service Azure Files prend en charge l’utilisation d’informations d’identification AD DS en local ou Azure AD DS pour accéder aux partages de fichiers Azure sur SMB à partir de machines virtuelles AD DS en local ou Azure AD DS jointes à un domaine.

-   **Appliquer un contrôle d’accès granulaire sur les partages de fichiers Azure**  
    Vous pouvez accorder des autorisations à une identité spécifique au niveau du partage, du répertoire ou du fichier. Par exemple, supposons que vous avez plusieurs équipes qui utilisent un partage de fichiers Azure pour la collaboration de projet. Vous pouvez accorder à toutes les équipes un accès aux répertoires non sensibles, et limiter l’accès aux répertoires contenant les données financières sensibles uniquement à l’équipe Finance. 

-   **Sauvegarder les listes de contrôle d’accès (ACL) Windows (également appelée NTFS) avec vos données**  
    Vous pouvez utiliser des partages de fichiers Azure pour sauvegarder vos partages de fichiers locaux existants. Le service Azure Files conserve vos ACL avec vos données quand vous sauvegardez un partage de fichiers dans des partages de fichiers Azure sur SMB.

## <a name="how-it-works"></a>Fonctionnement

Les partages de fichiers Azure tirent parti du protocole Kerberos pour s’authentifier auprès d’AD DS local ou Azure AD DS. Quand une identité associée à un utilisateur ou à une application s’exécutant sur un client tente d’accéder à des données se trouvant dans des partages de fichiers Azure, la requête est envoyée à un service de domaine, AD DS ou Azure AD DS, pour authentifier l’identité. Si l’authentification réussit, un jeton Kerberos est renvoyé. L’application envoie une requête incluant le jeton Kerberos, que les partages de fichiers Azure utilisent pour autoriser la requête. Les partages de fichiers Azure ne reçoivent que le jeton Kerberos, et non les informations d’identification d’accès.

Avant de pouvoir activer l’authentification basée sur l’identité sur des partages de fichiers Azure, vous devez configurer votre environnement de domaine.

### <a name="ad-ds"></a>AD DS

Pour l’authentification AD DS locale, vous devez configurer vos contrôleurs de domaine AD et joindre à un domaine vos machines ou machines virtuelles. Vous pouvez héberger vos contrôleurs de domaine sur des machines virtuelles Azure ou localement. Dans les deux cas, vos clients joints à un domaine doivent pouvoir accéder au service de domaine. Ils doivent donc se trouver dans le réseau virtuel (VNET) ou le réseau d’entreprise de votre service de domaine.

Le diagramme suivant illustre l’authentification AD DS locale pour l’accès aux partages de fichiers Azure sur SMB. L’environnement AD DS local doit être synchronisé avec Azure AD à l’aide de la solution Azure AD Connect Sync. Seuls les utilisateurs hybrides, qui existent à la fois dans AD DS local et Azure AD, peuvent être authentifiés et autorisés à accéder aux partages de fichiers Azure. Cela est dû au fait que l’autorisation au niveau du partage est configurée par rapport à l’identité représentée dans Azure AD, alors que l’autorisation au niveau du répertoire/fichier est appliquée avec elle dans AD DS. Veillez à configurer correctement les autorisations pour le même utilisateur hybride.

:::image type="content" source="media/storage-files-active-directory-overview/Files-on-premises-AD-DS-Diagram.png" alt-text="Diagramme illustrant l’authentification AD DS locale sur des partages de fichiers Azure via SMB.":::

### <a name="azure-ad-ds"></a>Azure AD DS

Pour l’authentification Azure AD DS, vous devez activer Azure AD Domain Services et joindre à un domaine les machines virtuelles à partir desquelles vous prévoyez d’accéder aux données de fichier. Votre machine virtuelle jointe à un domaine doit résider dans le même réseau virtuel qu’Azure AD DS. 

Le diagramme suivant illustre le flux de travail de l’authentification Azure AD DS pour l’accès aux partages de fichiers Azure via SMB. Il suit un modèle similaire à l’authentification AD DS locale pour l’accès aux partages de fichiers Azure. Deux différences importantes sont à noter :

- Tout d’abord, vous n’avez pas besoin de créer l’identité dans Azure AD DS pour représenter le compte de stockage. Cette opération est effectuée par le processus d’activation en arrière-plan.

- Ensuite, tous les utilisateurs présents dans Azure AD peuvent être authentifiés et autorisés. L’utilisateur peut être de type cloud uniquement ou hybride. La synchronisation depuis Azure AD vers Azure AD DS est gérée par la plateforme sans nécessiter de configuration utilisateur. Toutefois, le client doit être joint à un domaine Azure AD DS ; il ne peut pas être inscrit ou joint à Azure AD. 

:::image type="content" source="media/storage-files-active-directory-overview/Files-Azure-AD-DS-Diagram.png" alt-text="Diagramme":::

### <a name="enable-identity-based-authentication"></a>Activer une authentification basée sur l’identité

Vous pouvez activer l’authentification basée sur l’identité avec Azure AD DS ou AD DS local pour l’accès aux partages de fichiers Azure sur vos comptes de stockage nouveaux et existants. Vous ne pouvez utiliser qu’une seul service de domaine pour l’authentification d’accès aux fichiers sur le compte de stockage, qui s’applique à tous les partages de fichiers du compte. Vous trouverez un guide détaillé sur la configuration des partages de fichiers pour l’authentification auprès d’Azure AD DS dans notre article [Activer l’authentification Azure Active Directory Domain Services sur Azure Files](storage-files-identity-auth-active-directory-domain-service-enable.md), et des conseils pour AD DS local dans notre article [Activer l’authentification Active Directory Domain Services locale sur SMB pour les partages de fichiers Azure](storage-files-identity-auth-active-directory-enable.md).

### <a name="configure-share-level-permissions-for-azure-files"></a>Configurer les autorisations au niveau du partage pour Azure Files

Une fois l’authentification Azure AD DS ou AD DS locale activée, vous pouvez utiliser des rôles Azure intégrés ou configurer des rôles personnalisés pour les identités Azure AD, et attribuer des droits d’accès à tous les partages de fichiers de vos comptes de stockage. L’autorisation attribuée permet à l’identité accordée d’obtenir l’accès uniquement au partage, mais pas au répertoire racine. Vous devez toujours configurer séparément les autorisations au niveau du répertoire ou du fichier pour les partages de fichiers Azure.

### <a name="configure-directory-or-file-level-permissions-for-azure-files"></a>Configurer des autorisations au niveau du répertoire ou du fichier pour Azure Files

Le service Azure Files applique des autorisations de fichiers Windows standard au niveau du répertoire et du fichier, y compris au niveau du répertoire racine. La configuration des autorisations au niveau du répertoire ou du fichier est prise en charge sur SMB et REST. Montez le partage de fichiers cible à partir de votre machine virtuelle, et configurez des autorisations à l’aide de l’Explorateur de fichiers Windows, de la commande Windows [icacls](/windows-server/administration/windows-commands/icacls) ou de la commande [Set-ACL](/powershell/module/microsoft.powershell.security/get-acl?view=powershell-6).

### <a name="use-the-storage-account-key-for-superuser-permissions"></a>Utiliser la clé de compte de stockage pour les autorisations de superutilisateur

Un utilisateur disposant de la clé de compte de stockage peut accéder aux partages de fichiers Azure avec des autorisations de superutilisateur. Des autorisations de superutilisateur contournent toutes les restrictions de contrôle d’accès.

> [!IMPORTANT]
> Notre meilleure pratique de sécurité recommandée consiste à éviter de partager vos clés de compte de stockage et à tirer parti autant que possible de l’authentification basée sur l’identité.

### <a name="preserve-directory-and-file-acls-when-importing-data-to-azure-file-shares"></a>Conserver les ACL de répertoire et de fichier lors de l’importation de données dans des partages de fichiers Azure

Le service Azure Files prend en charge la conservation des ACL de répertoire ou de fichier lors de la copie de données vers des partages de fichiers Azure. Vous pouvez copier des listes de contrôle d’accès (ACL) d’un répertoire ou d’un fichier vers des partages de fichiers Azure à l’aide d’Azure File Sync ou d’outils de déplacement de fichiers communs. Par exemple, vous pouvez utiliser [robocopy](/windows-server/administration/windows-commands/robocopy) avec l’étiquette `/copy:s` pour copier des données et des ACL vers un partage de fichiers Azure. Les listes de contrôle d’accès étant conservées par défaut, vous n’êtes pas obligé d’activer l’authentification basée sur l’identité sur votre compte de stockage pour les conserver.

## <a name="pricing"></a>Tarifs
L’activation d’une authentification basée sur l’identité via SMB sur votre compte de stockage n’engendre pas de frais de service supplémentaires. Pour plus d’informations sur les prix, consultez [Tarifs Azure Files](https://azure.microsoft.com/pricing/details/storage/files/) et [Tarifs Azure AD Domain Services](https://azure.microsoft.com/pricing/details/active-directory-ds/).

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur Azure Files et l’authentification basée sur l’identité sur SMB, voir les ressources suivantes :

- [Planification d’un déploiement Azure Files](storage-files-planning.md)
- [Activer l’authentification Active Directory Domain Services locale sur SMB pour les partages de fichiers Azure](storage-files-identity-auth-active-directory-enable.md)
- [Activer l’authentification Azure Active Directory Domain Services sur Azure Files](storage-files-identity-auth-active-directory-domain-service-enable.md)
- [FORUM AUX QUESTIONS](storage-files-faq.md)