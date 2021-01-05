---
title: Sécuriser l’accès aux machines virtuelles distantes dans Azure AD Domain Services | Microsoft Docs
description: Apprenez à sécuriser l'accès à distance aux machines virtuelles à l'aide du serveur NPS (Network Policy Server) et d'Azure AD Multi-Factor Authentication avec un déploiement des services Bureau à distance dans un domaine managé Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: f0605cbd81d8131014a1f6a6bf30e3db0ce9aa90
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618924"
---
# <a name="secure-remote-access-to-virtual-machines-in-azure-active-directory-domain-services"></a>Sécuriser l’accès à distance aux machines virtuelles dans Azure Active Directory Domain Services

Pour sécuriser l’accès à distance aux machines virtuelles qui s’exécutent dans un domaine managé Azure Active Directory Domain Services (Azure AD DS), vous pouvez utiliser les services Bureau à distance (RDS) et le serveur NPS (Network Policy Server). Azure AD DS authentifie les utilisateurs quand ils demandent l’accès par le biais de l’environnement des services Bureau à distance. Pour renforcer la sécurité, vous pouvez intégrer Azure AD Multi-Factor Authentication afin de fournir une invite d'authentification supplémentaire pendant les événements de connexion. Pour fournir cette fonctionnalité, Azure AD Multi-Factor Authentication utilise une extension NPS.

> [!IMPORTANT]
> La méthode recommandée pour se connecter en toute sécurité à vos machines virtuelles dans un domaine managé Azure AD DS consiste à utiliser Azure Bastion, un service PaaS complètement managé par la plateforme, que vous approvisionnez à l’intérieur de votre réseau virtuel. Un hôte bastion offre une connectivité sécurisée et transparente suivant le protocole RDP (Remote Desktop Protocol) à vos machines virtuelles directement dans le portail Azure via le protocole SSL. Lorsque vous vous connectez via un hôte bastion, vos machines virtuelles n’ont pas besoin d’adresse IP publique, et vous n’avez pas besoin d’utiliser des groupes de sécurité réseau pour exposer l’accès au protocole RDP sur le port TCP 3389.
>
> Nous vous recommandons vivement d’utiliser le service Azure Bastion dans toutes les régions où il est pris en charge. Dans les régions où Azure Bastion n’est pas disponible, suivez les étapes décrites dans cet article en attendant que le service soit disponible. Veillez à attribuer des adresses IP publiques aux machines virtuelles jointes à Azure AD DS où tout le trafic RDP entrant est autorisé.
>
> Pour plus d’informations, consultez [Présentation d’Azure Bastion][bastion-overview].

Cet article explique comment configurer les services Bureau à distance dans Azure AD DS et comment utiliser l'extension NPS d'Azure AD Multi-Factor Authentication.

![Vue d’ensemble des services Bureau à distance](./media/enable-network-policy-server/remote-desktop-services-overview.png)

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des ressources suivantes :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, [créez et configurez un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Un sous-réseau de *charges de travail* créé dans votre réseau virtuel Azure Active Directory Domain Services.
    * Si nécessaire, [configurez un réseau virtuel pour une instance Azure Active Directory Domain Services][configure-azureadds-vnet].
* Un compte d’utilisateur membre du groupe *Administrateurs Azure AD DC* dans votre locataire Azure AD.

## <a name="deploy-and-configure-the-remote-desktop-environment"></a>Déployer et configurer l’environnement de Bureau à distance

Pour commencer, créez au minimum deux machines virtuelles Azure exécutant Windows Server 2016 ou Windows Server 2019. Pour assurer la redondance et la haute disponibilité de votre environnement de Bureau à distance, vous pouvez ajouter des hôtes supplémentaires et équilibrer la charge ultérieurement.

Un déploiement des services Bureau à distance suggéré comprend les deux machines virtuelles suivantes :

* *RDGVM01* : exécute les serveurs Broker pour les connexions Bureau à distance, Accès Web des services Bureau à distance et Passerelle des services Bureau à distance.
* *RDSHVM01* : exécute le serveur Hôte de session Bureau à distance.

Assurez-vous que les machines virtuelles sont déployées dans un sous-réseau de *charges de travail* de votre réseau virtuel Azure AD DS, puis joignez les machines virtuelles à un domaine managé. Pour plus d’informations, découvrez comment [créer une machine virtuelle Windows Server et la joindre à un domaine managé][tutorial-create-join-vm].

Le déploiement de l’environnement de Bureau à distance contient un certain nombre d’étapes. Le guide de déploiement de Bureau à distance existant peut être utilisé sans modification spécifique dans un domaine managé :

1. Connectez-vous aux machines virtuelles créées pour l’environnement de Bureau à distance à l’aide d’un compte faisant partie du groupe *Administrateurs Azure AD DC*, par exemple, *contosoadmin*.
1. Pour créer et configurer des services Bureau à distance, utilisez le [Guide de déploiement de l’environnement de Bureau à distance][deploy-remote-desktop] existant. Distribuez les composants du serveur Bureau à distance sur vos machines virtuelles Azure à votre guise.
    * Spécificité d’Azure AD DS : lorsque vous configurez le gestionnaire de licences des services Bureau à distance, définissez-le sur mode **Par appareil**, non **Par utilisateur** comme indiqué dans le Guide de déploiement.
1. Si vous souhaitez fournir l’accès à l’aide d’un navigateur web, [Configurez le client web Bureau à distance pour vos utilisateurs][rd-web-client].

Avec le Bureau à distance déployé dans le domaine managé, vous pouvez gérer et utiliser le service comme vous le feriez avec un domaine AD DS local.

## <a name="deploy-and-configure-nps-and-the-azure-ad-mfa-nps-extension"></a>Déployer et configurer le serveur NPS (Network Policy Server) et l'extension NPS d'Azure AD MFA

Si vous le souhaitez, vous pouvez renforcer la sécurité de l'expérience de connexion utilisateur en intégrant l'environnement de Bureau à distance à Azure AD Multi-Factor Authentication. Avec cette configuration, les utilisateurs reçoivent une invite supplémentaire pendant la connexion pour confirmer leur identité.

Pour fournir cette fonctionnalité, un serveur NPS (Network Policy Server) supplémentaire est installé dans votre environnement avec l'extension NPS d'Azure AD Multi-Factor Authentication. Cette extension s’intègre avec Azure AD pour demander et retourner l’état des invites d’authentification multifacteur.

Les utilisateurs doivent être [inscrits pour utiliser Azure AD Multi-Factor Authentication][user-mfa-registration], ce qui peut nécessiter des licences Azure AD supplémentaires.

Pour intégrer Azure AD Multi-Factor Authentication dans votre environnement de Bureau à distance Azure AD DS, créez un serveur NPS et installez l'extension :

1. Créez une machine virtuelle Windows Server 2016 ou 2019 supplémentaire, par exemple, *NPSVM01*, qui est connectée à un sous-réseau de *charges de travail* dans votre réseau virtuel Azure AD DS. Joignez la machine virtuelle au domaine managé.
1. Connectez-vous à la machine virtuelle NPS en tant que compte faisant partie du groupe *Administrateurs Azure AD DC*, par exemple, *contosoadmin*.
1. Dans le **Gestionnaire de serveur**, sélectionnez **Ajouter des rôles et des fonctionnalités**, puis installez le rôle *Services de stratégie et d’accès réseau*.
1. Consultez l'article de procédure existant pour [installer et configurer l'extension NPS d'Azure AD MFA][nps-extension].

Une fois le serveur NPS et l'extension NPS d'Azure AD Multi-Factor Authentication installés, consultez la section suivante pour les configurer en vue de leur utilisation avec l'environnement de Bureau à distance.

## <a name="integrate-remote-desktop-gateway-and-azure-ad-multi-factor-authentication"></a>Intégrer la Passerelle des services Bureau à distance et Azure AD Multi-Factor Authentication

Pour intégrer l'extension NPS d'Azure AD Multi-Factor Authentication, consultez l'article de procédure expliquant comment [intégrer votre infrastructure de Passerelle des services Bureau à distance à l'aide de l'extension NPS (Network Policy Server) et d'Azure AD][azure-mfa-nps-integration].

Les options de configuration supplémentaires suivantes sont nécessaires pour l’intégration avec un domaine managé :

1. N’[Enregistrez pas le serveur NPS dans Active Directory][register-nps-ad]. Cette étape échoue dans un domaine managé.
1. À l’[étape 4 pour configurer la stratégie réseau][create-nps-policy], activez également la case à cocher **Ignorer les propriétés de numérotation des comptes d’utilisateurs**.
1. Si vous utilisez Windows Server 2019 pour le serveur NPS et l'extension NPS d'Azure AD Multi-Factor Authentication, exécutez la commande suivante pour mettre à jour le canal sécurisé afin de permettre au serveur NPS de communiquer correctement :

    ```powershell
    sc sidtype IAS unrestricted
    ```

Les utilisateurs sont désormais invités à entrer un facteur d’authentification supplémentaire lorsqu’ils se connectent, par exemple un message texte ou une invite dans l’application Microsoft Authenticator.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’amélioration de la résilience de votre déploiement, consultez [Services Bureau à distance – Haute disponibilité][rds-high-availability].

Pour plus d’informations sur la sécurisation de la connexion utilisateurs, consultez [Fonctionnement : Azure AD Multi-Factor Authentication][concepts-mfa].

<!-- INTERNAL LINKS -->
[bastion-overview]: ../bastion/bastion-overview.md
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[configure-azureadds-vnet]: tutorial-configure-networking.md
[tutorial-create-join-vm]: join-windows-vm.md
[user-mfa-registration]: ../active-directory/authentication/howto-mfa-nps-extension.md#register-users-for-mfa
[nps-extension]: ../active-directory/authentication/howto-mfa-nps-extension.md
[azure-mfa-nps-integration]: ../active-directory/authentication/howto-mfa-nps-extension-rdg.md
[register-nps-ad]:../active-directory/authentication/howto-mfa-nps-extension-rdg.md#register-server-in-active-directory
[create-nps-policy]: ../active-directory/authentication/howto-mfa-nps-extension-rdg.md#configure-network-policy
[concepts-mfa]: ../active-directory/authentication/concept-mfa-howitworks.md

<!-- EXTERNAL LINKS -->
[deploy-remote-desktop]: /windows-server/remote/remote-desktop-services/rds-deploy-infrastructure
[rd-web-client]: /windows-server/remote/remote-desktop-services/clients/remote-desktop-web-client-admin
[rds-high-availability]: /windows-server/remote/remote-desktop-services/rds-plan-high-availability