---
title: Vue d’ensemble - Authentification AD DS locale pour des partages de fichiers Azure
description: Découvrez l’authentification AD DS (Azure Active Directory Domain Services) pour les partages de fichiers Azure. Cet article passe en revue les scénarios de support et la disponibilité. Il explique également comment les autorisations fonctionnent entre votre AD DS et Azure Active Directory.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 09/13/2020
ms.author: rogarana
ms.openlocfilehash: 2214dbc9dcbd4ba7728065ee45471e9f94b9e513
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95739993"
---
# <a name="overview---on-premises-active-directory-domain-services-authentication-over-smb-for-azure-file-shares"></a>Vue d’ensemble - Authentification Active Directory Domain Services locale sur SMB pour les partages de fichiers Azure

[Azure Files](storage-files-introduction.md) prend en charge l’authentification basée sur l’identité sur SMB (Server Message Block) via deux types de services de domaine : Active Directory Domain Services (AD DS) en local et Azure Active Directory Domain Services (Azure AD DS). Nous vous recommandons vivement de consulter la section [Fonctionnement](./storage-files-active-directory-overview.md#how-it-works) pour sélectionner le service de domaine approprié pour l’authentification. L’installation est différente selon le service de domaine que vous choisissez. Cette série d’articles se concentre sur l’activation et la configuration d’AD DS en local pour l’authentification auprès de partages de fichiers Azure.

Si vous débutez avec les partages de fichiers Azure, nous vous recommandons de lire notre [guide de planification](storage-files-planning.md) avant de vous lancer dans les articles suivants.

## <a name="supported-scenarios-and-restrictions"></a>Scénarios pris en charge et restrictions

- Les identités AD DS utilisées pour l’authentification AD DS en local Azure Files doivent être synchronisées avec Azure AD. La synchronisation de hachage de mot de passe est facultative. 
- Prend en charge les partages de fichiers Azure gérés par Azure File Sync.
- Prend en charge l’authentification Kerberos avec AD, ainsi qu’avec le [chiffrement AES 256](./storage-troubleshoot-windows-file-connection-problems.md#azure-files-on-premises-ad-ds-authentication-support-for-aes-256-kerberos-encryption) et le chiffrement RC4-HMAC. La prise en charge du chiffrement AES 256 est actuellement limitée aux comptes de stockage dont les noms n’excèdent pas 15 caractères. Le chiffrement Kerberos AES 128 n’est pas encore pris en charge.
- Prend en charge l’authentification unique.
- Pris en charge uniquement sur les clients exécutant des systèmes d’exploitation ultérieurs à Windows 7 ou Windows Server 2008 R2.
- Pris en charge uniquement sur la forêt AD auprès de laquelle le compte de stockage est inscrit. Vous pouvez uniquement accéder aux partages de fichiers Azure avec les informations d’identification AD DS à partir d’une forêt unique par défaut. Si vous avez besoin d’accéder à votre partage de fichiers Azure à partir d’une autre forêt, vérifiez que l’approbation de forêt appropriée est configurée. Pour plus d’informations, consultez le [FAQ](storage-files-faq.md#ad-ds--azure-ad-ds-authentication).
- Ne prend pas en charge l’authentification sur des comptes d’ordinateur créés dans AD DS.
- Ne prend pas en charge l’authentification auprès des partages de fichiers NFS (Network File System).

Quand vous activez AD DS pour les partages de fichiers Azure sur SMB, vos machines jointes à AD DS peuvent monter des partages de fichiers Azure à l’aide de vos informations d’identification AD DS existantes. Cette capacité peut être activée avec un environnement AD DS hébergé sur des machines locales ou hébergé dans Azure.

> [!NOTE]
> Afin de vous aider à configurer l’authentification AD Azure Files pour certains cas d’usage courants, nous avons publié deux vidéos avec des instructions pas à pas pour les scénarios suivants :
> - [Remplacement des serveurs de fichiers locaux par Azure Files (y compris l’installation sur un lien privé pour les fichiers et l’authentification AD)](https://sec.ch9.ms/ch9/3358/0addac01-3606-4e30-ad7b-f195f3ab3358/ITOpsTalkAzureFiles_high.mp4)
> - [Utilisation d’Azure Files comme conteneur de profil pour Windows Virtual Desktop (y compris l’installation sur l’authentification AD et la configuration FsLogix)](https://www.youtube.com/embed/9S5A1IJqfOQ)

## <a name="prerequisites"></a>Prérequis 

Avant d’activer l’authentification AD DS pour des partages de fichiers Azure, assurez-vous que les conditions préalables suivantes sont remplies : 

- Sélectionnez ou créez votre [environnement AD DS](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview), puis [synchronisez-le à Azure AD](../../active-directory/hybrid/how-to-connect-install-roadmap.md). 

    Vous pouvez activer la fonctionnalité sur un environnement AD DS local nouveau ou existant. Les identités utilisées pour l’accès doivent être synchronisées à Azure AD. Le locataire Azure AD et le partage de fichiers auquel vous accédez doivent être associés au même abonnement.

- Joignez le domaine d’un ordinateur local ou d’une machine virtuelle Azure à AD DS en local. Pour plus d’informations sur la façon de joindre un domaine, reportez-vous à [Joindre un ordinateur à un domaine](/windows-server/identity/ad-fs/deployment/join-a-computer-to-a-domain).

    Si votre ordinateur n’est pas joint à un domaine AD DS, vous pouvez toujours utiliser les informations d’identification AD pour l’authentification si votre ordinateur a une ligne de vue sur le contrôleur de domaine Active Directory.

- Sélectionnez ou créez un compte de stockage Azure.  Pour des performances optimales, nous vous recommandons de déployer le compte de stockage dans la même région que celle du client à partir duquel vous prévoyez d’accéder au partage. Ensuite, [montez le partage de fichiers Azure](storage-how-to-use-files-windows.md) avec la clé de votre compte de stockage. Le montage avec la clé de compte de stockage vérifie la connectivité.

    Assurez-vous que le compte de stockage contenant vos partages de fichiers n’est pas déjà configuré pour l’authentification Azure AD DS. Si l’authentification Azure AD DS pour Azure Files est activée sur le compte de stockage, elle doit être désactivée avant de changer pour AD DS en local. Cela implique que les ACL existantes configurées dans l’environnement Azure AD DS devront être reconfigurées pour l’application adéquate des autorisations.


    Si vous rencontrez des problèmes lors de la connexion à Azure Files, consultez [l’outil de dépannage que nous avons publié pour les erreurs de montage Azure Files sur Windows](https://azure.microsoft.com/blog/new-troubleshooting-diagnostics-for-azure-files-mounting-errors-on-windows/). Nous fournissons également des [conseils](./storage-files-faq.md#on-premises-access) pour contourner des situations où le port 445 est bloqué. 


- Effectuez toute configuration du réseau appropriée avant d’activer et de configurer l’authentification AD DS pour vos partages de fichiers Azure. Pour plus d’informations, consultez [Considérations relatives aux réseaux Azure Files](storage-files-networking-overview.md).

## <a name="regional-availability"></a>Disponibilité régionale

L’authentification Azure Files avec AD DS est disponible dans [toutes les régions publiques et Gov Azure](https://azure.microsoft.com/global-infrastructure/locations/).

## <a name="overview"></a>Vue d’ensemble

Si vous envisagez d’activer des configurations de mise en réseau sur votre partage de fichiers, nous vous recommandons de lire l’article sur les [considérations en matière de réseau](./storage-files-networking-overview.md) et de terminer la configuration associée avant d’activer l’authentification AD DS.

L’activation de l’authentification AD DS pour vos partages de fichiers Azure vous permet de vous authentifier auprès de vos partages de fichiers Azure avec vos informations d’identification AD DS locales. Elle vous permet en outre de mieux gérer vos autorisations pour contrôler les accès avec précision. Pour ce faire, vous devez synchroniser les identités de l’AD DS local sur Azure AD avec Azure AD Connect. Vous contrôlez l’accès au niveau partage avec des identités synchronisées sur Azure AD tout en gérant l’accès au niveau fichier/partage avec les informations d’identification AD DS locales.

Suivez ensuite les étapes ci-dessous pour configurer Azure Files pour l’authentification AD DS : 

1. [Première partie : activer l’authentification AD DS sur votre compte de stockage](storage-files-identity-ad-ds-enable.md)

1. [Deuxième partie : attribuer des autorisations d’accès pour un partage à une identité Azure AD (utilisateur, groupe ou principal de service) synchronisée avec l’identité AD cible](storage-files-identity-ad-ds-assign-permissions.md)

1. [Troisième partie : configurer des listes de contrôle d’accès sur SMB pour les répertoires et les fichiers](storage-files-identity-ad-ds-configure-permissions.md)
 
1. [Quatrième partie : monter un partage de fichiers Azure sur une machine virtuelle jointe à votre AD DS](storage-files-identity-ad-ds-mount-file-share.md)

1. [Mettre à jour le mot de passe de l’identité de votre compte de stockage dans AD DS](storage-files-identity-ad-ds-update-password.md)

Le diagramme ci-dessous illustre le workflow de bout en bout pour l’activation de l’authentification Azure AD sur SMB pour les partages de fichiers Azure. 

![Diagramme de workflow AD pour Files](media/storage-files-active-directory-domain-services-enable/diagram-files-ad.png)

Les identités utilisées pour accéder aux partages de fichiers Azure doivent être synchronisées avec Azure AD pour appliquer les autorisations de fichiers au niveau du partage via le modèle de [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md). Les [DACL de style Windows](/previous-versions/technet-magazine/cc161041(v=msdn.10)) sur des fichiers/répertoires reportés provenant de serveurs de fichiers existants seront conservées et appliquées. Il en résulte une intégration fluide à votre environnement AD DS d’entreprise. Lorsque vous remplacez des serveurs de fichiers locaux par des partages de fichiers Azure, les utilisateurs existants peuvent accéder aux partages de fichiers Azure à partir de leurs clients actuels à l’aide d’une expérience d’authentification unique, sans modification des informations d’identification utilisés.  

## <a name="next-steps"></a>Étapes suivantes

Pour activer l’authentification AD DS locale pour votre partage de fichiers Azure, passez à l’article suivant :

[Première partie : activer l’authentification AD DS sur votre compte](storage-files-identity-ad-ds-enable.md)
