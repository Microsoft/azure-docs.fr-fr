---
title: Questions fréquentes (FAQ) sur Windows 10 Entreprise multisession - Azure
description: Questions fréquentes (FAQ) et bonnes pratiques concernant l’utilisation de Windows 10 Entreprise multisession pour Windows Virtual Desktop.
author: Heidilohr
ms.topic: conceptual
ms.date: 02/19/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 24d27859aad7642392a5702ab7ddbb727b538d51
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025008"
---
# <a name="windows-10-enterprise-multi-session-faq"></a>FAQ sur Windows 10 Entreprise multisession

Cet article répond à des questions fréquemment posées et décrit les meilleures pratiques concernant l’utilisation de Windows 10 Entreprise en mode multisession.

## <a name="what-is-windows-10-enterprise-multi-session"></a>Qu’est-ce que Windows 10 Entreprise multisession ?

Windows 10 Entreprise en mode multisession, (anciennement Windows 10 Entreprise pour bureaux virtuels, ou EVD) est un nouvel hôte de session Bureau à distance qui permet plusieurs sessions interactives simultanées. Auparavant, seul Windows Server pouvait faire cela. Cette fonctionnalité offre aux utilisateurs une expérience Windows 10 familière, et permet au service informatique de tirer parti des avantages économiques que présente l’utilisation du multisession, ainsi que d’utiliser des licences Windows par utilisateur existantes plutôt que des licences d’accès client pour les services Bureau à distance. Pour plus d’informations sur les licences et les tarifs, consultez [Tarifs de Windows Virtual Desktop](https://azure.microsoft.com/pricing/details/virtual-desktop/).

## <a name="how-many-users-can-simultaneously-have-an-interactive-session-on-windows-10-enterprise-multi-session"></a>Combien d’utilisateurs simultanés peuvent ouvrir une session interactive sur Windows 10 Entreprise multisession ?

Le nombre de sessions interactives qui peuvent être actives en même temps dépend des ressources matérielles de votre système (processeurs virtuels, mémoire, disque et processeur graphique virtuel), ainsi que de la façon dont vos utilisateurs se servent de leurs applications lorsqu’ils sont connectés à une session et de la charge de travail de votre système. Nous vous suggérons de vérifier les performances de votre système pour déterminer combien d’utilisateurs simultanés peuvent utiliser Windows 10 Entreprise multisession. Pour plus d’informations, consultez [Tarifs Windows Virtual Desktop](https://azure.microsoft.com/pricing/details/virtual-desktop/).

## <a name="why-does-my-application-report-windows-10-enterprise-multi-session-as-a-server-operating-system"></a>Pourquoi mon application indique-t-elle que Windows 10 Entreprise multisession est un système d’exploitation serveur ?

Windows 10 Entreprise multisession est une édition virtuelle de Windows 10 Entreprise. La différence est que ce système d’exploitation attribue la valeur 3 à [ProductType](/windows/win32/cimwin32prov/win32-operatingsystem), qui est la même que pour Windows Server. Cette propriété permet de maintenir la compatibilité du système d’exploitation avec les outils de gestion existants de l’hôte de connexion à distance, les applications compatibles avec le multisession de l’hôte de connexion à distance et principalement, l’optimisation des performances du système de bas niveau pour les environnements de type hôte de connexion à distance. Certains programmes d’installation de l’application peuvent bloquer l’installation sur Windows 10 multisession s’ils détectent que le ProductType est défini sur Client. Si votre application ne peut pas être installée, contactez son fournisseur pour obtenir une version mise à jour.

## <a name="can-i-run-windows-10-enterprise-multi-session-on-premises"></a>Est-il possible d’exécuter Windows 10 Entreprise multisession localement ?

Windows 10 Entreprise multisession ne peut pas s’exécuter dans des environnements de production locaux, car il est optimisé pour le service Windows Virtual Desktop pour Azure. Il est contraire au contrat de licence d’exécuter Windows 10 Entreprise multisession en dehors d’Azure à des fins de production. Windows 10 Entreprise multisession ne peut pas être activé à l’aide des services locaux de gestion des clés (KMS).

## <a name="how-do-i-customize-the-windows-10-enterprise-multi-session-image-for-my-organization"></a>Comment personnaliser l’image de Windows 10 Entreprise multisession pour mon organisation ?

Vous pouvez démarrer une machine virtuelle dans Azure avec Windows 10 Entreprise multisession, la personnaliser en installant des applications métier, effectuer une généralisation Sysprep, puis créer une image à l’aide du portail Azure.

Pour commencer, créez une machine virtuelle dans Azure avec Windows 10 Entreprise multisession. Au lieu de démarrer la machine virtuelle dans Azure, vous pouvez télécharger directement le disque dur virtuel. Après cela, vous pourrez utiliser le disque dur virtuel que vous avez téléchargé pour créer une machine virtuelle Génération 1 sur un PC Windows 10 où est activé Hyper-V.

Personnalisez l’image selon vos besoins à l’aide d’applications métier et de Sysprep. Lorsque vous avez terminé la personnalisation, chargez l’image dans Azure en y incluant le disque dur virtuel. Ensuite, téléchargez le service Windows Virtual Desktop à partir de la Place de marché Azure, puis utilisez-le pour déployer un nouveau pool d’hôte avec l’image personnalisée.

## <a name="how-do-i-manage-windows-10-enterprise-multi-session-after-deployment"></a>Comment gérer Windows 10 Entreprise multisession après le déploiement ?

Vous pouvez utiliser n’importe quel outil de configuration pris en charge. Toutefois, nous vous recommandons Configuration Manager version 1906, car celui-ci prend en charge Windows 10 entreprise multi-session. Nous travaillons actuellement à la prise en charge de Microsoft Intune.

## <a name="can-windows-10-enterprise-multi-session-be-azure-active-directory-ad-joined"></a>Peut-on joindre Windows 10 Entreprise multisession à un domaine Azure Active Directory (AD) ?

Windows 10 Entreprise multisession peut être joint à un domaine Azure AD Hybride. Une fois que Windows 10 Entreprise multisession est joint à un domaine, utilisez l’objet Stratégie de groupe existant pour activer l’inscription Azure AD. Pour plus d’informations, consultez [Planifier l’implémentation de la jonction Azure Active Directory Hybride](../active-directory/devices/hybrid-azuread-join-plan.md).

## <a name="where-can-i-find-the-windows-10-enterprise-multi-session-image"></a>Où trouver l’image Windows 10 Entreprise multisession ?

Windows 10 Entreprise multisession est disponible dans la galerie Azure. Pour le trouver, accédez au portail Azure, puis recherchez la version de Windows 10 Entreprise pour les bureaux virtuels. Pour obtenir une image intégrée à Microsoft 365 Apps for enterprise, accédez au portail Azure et recherchez **Microsoft Windows 10 + Microsoft 365 Apps for enterprise**.

## <a name="which-windows-10-enterprise-multi-session-image-should-i-use"></a>Quelle image de Windows 10 Entreprise multisession dois-je utiliser ?

La galerie Azure propose plusieurs versions, notamment Windows 10 Entreprise multisession version 1809 et Windows 10 Entreprise multisession version 1903. Nous vous recommandons d’utiliser la dernière version pour de meilleures performances et une plus grande fiabilité.

## <a name="which-windows-10-enterprise-multi-session-versions-are-supported"></a>Quelles sont les versions de Windows 10 Entreprise multisession qui sont prises en charge ?

Les versions de Windows 10 Entreprise multisession qui sont prises en charge et disponibles dans la galerie Azure sont la version 1809 et les versions ultérieures. Ces versions suivent la même stratégie de cycle de vie de support que Windows 10 Entreprise. Autrement dit, la version du mois de mars est prise en charge pendant 18 mois et la version du mois de septembre pendant 30 mois.

## <a name="which-profile-management-solution-should-i-use-for-windows-10-enterprise-multi-session"></a>Quelle solution de gestion des profils dois-je utiliser pour Windows 10 Entreprise multisession ?

Nous vous recommandons d’utiliser des conteneurs de profils FSLogix lorsque vous configurez Windows 10 Entreprise dans des environnements non persistants ou dans d’autres scénarios qui nécessitent un profil stocké de manière centralisée. FSLogix garantit que le profil utilisateur est disponible et à jour pour chaque session utilisateur. Nous vous recommandons également d’utiliser votre conteneur de profils FSLogix pour stocker un profil utilisateur dans un partage SMB avec les autorisations appropriées. Toutefois, vous pouvez stocker les profils utilisateur dans le stockage d’objets blob de pages Azure, si nécessaire. Les utilisateurs de Windows Virtual Desktop peuvent utiliser FSLogix sans coûts supplémentaires.

Pour plus d’informations sur la configuration d’un conteneur de profils FSLogix, consultez [Configurer le conteneur de profils FSLogix](create-host-pools-user-profile.md#configure-the-fslogix-profile-container).

## <a name="which-license-do-i-need-to-access-windows-10-enterprise-multi-session"></a>De quelle licence ai-je besoin pour accéder à Windows 10 Entreprise multisession ?

Pour obtenir la liste complète des licences applicables, consultez [Tarifs Windows Virtual Desktop](https://azure.microsoft.com/pricing/details/virtual-desktop/).

## <a name="why-do-my-apps-disappear-after-i-sign-out"></a>Pourquoi mes applications disparaissent-elles une fois que je me déconnecte ?

Cela se produit parce que vous utilisez Windows 10 entreprise en mode multisession avec une solution de gestion de profil telle que FSLogix. Votre solution d’administrateur ou de profil a configuré votre système pour supprimer les profils utilisateurs lorsque des utilisateurs se déconnectent. Cette configuration signifie que, lorsque votre système supprime votre profil utilisateur après que vous vous êtes déconnecté, il supprime également toutes les applications que vous avez installées au cours de votre session. Si vous souhaitez conserver les applications que vous avez installées, vous devez demander à votre administrateur d’approvisionner ces applications pour tous les utilisateurs figurant dans votre environnement Windows Virtual Desktop.

## <a name="how-do-i-make-sure-apps-dont-disappear-when-users-sign-out"></a>Comment m’assurer que les applications ne disparaissent pas quand les utilisateurs se déconnectent ?

La plupart des environnements virtualisés sont configurés par défaut pour empêcher les utilisateurs d’installer des applications supplémentaires dans leurs profils. Si vous souhaitez vous assurer qu’une application ne disparaisse pas quand l’utilisateur se déconnecte de Windows Virtual Desktop, vous devez approvisionner cette application pour tous les profils utilisateurs au sein de votre environnement. Pour plus d’informations sur l’approvisionnement d’applications, consultez les ressources suivantes :

- [Publier des applications intégrées dans Windows Virtual Desktop](publish-apps.md)
- [Options de ligne de commande pour la maintenance des packages d’applications DISM](/windows-hardware/manufacture/desktop/dism-app-package--appx-or-appxbundle--servicing-command-line-options)
- [Add-AppxProvisionedPackage](/powershell/module/dism/add-appxprovisionedpackage?view=win10-ps)

## <a name="how-do-i-make-sure-users-dont-download-and-install-apps-from-the-microsoft-store"></a>Comment m’assurer que des utilisateurs ne téléchargent et n’installent pas des applications à partir du Microsoft Store ?

Vous pouvez désactiver l’application Microsoft Store pour vous assurer que des utilisateurs ne téléchargent pas d’applications en plus de celles que vous avez déjà approvisionnées.

Pour désactiver l’application du Windows Store :

1. Créez une stratégie de groupe.
2. Sélectionnez **Configuration ordinateur** > **Modèles d’administration** > **Composants Windows**.
3. Sélectionnez **Store**.
4. Sélectionnez **Application du Store**.
5. Sélectionnez **Désactivée**, puis **OK**.
6. Sélectionnez **Appliquer**.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Windows Virtual Desktop et Windows 10 Entreprise multisession :

- Lisez notre [documentation Windows Virtual Desktop](overview.md)
- Consultez le site de la [Tech Community Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).
- Configurez votre déploiement Windows Virtual Desktop à l’aide des [tutoriels Windows Virtual Desktop](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md).