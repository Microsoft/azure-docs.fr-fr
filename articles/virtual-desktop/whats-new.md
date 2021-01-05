---
title: Nouveautés de Windows Virtual Desktop - Azure
description: Nouvelles fonctionnalités et mises à jour de produit pour Windows Virtual Desktop.
author: Heidilohr
ms.topic: overview
ms.date: 12/02/2020
ms.author: helohr
ms.reviewer: thhickli; darank
manager: lizross
ms.custom: references_regions
ms.openlocfilehash: e267de136271991446444a47a470dd52789b3613
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96533959"
---
# <a name="whats-new-in-windows-virtual-desktop"></a>Nouveautés de Windows Virtual Desktop

Windows Virtual Desktop est mis à jour régulièrement. Vous trouverez dans cet article des informations sur :

- Les dernières mises à jour.
- Nouvelles fonctionnalités
- Les améliorations apportées aux fonctionnalités existantes.
- Résolution des bogues

Cet article est mis à jour tous les mois. Pensez à consulter cette page régulièrement pour être tenu informé des nouvelles mises à jour.

## <a name="november-2020"></a>Novembre 2020

### <a name="azure-portal-experience"></a>Utilisation du portail Azure

Nous avons résolu deux bogues dans l’expérience utilisateur du portail Azure :

- Le nom convivial de l’application de bureau n’est plus remplacé dans le workflow « Ajouter une machine virtuelle ».
- L’onglet de l’hôte de session se chargera désormais si les hôtes de session font partie de groupes identiques.

### <a name="fslogix-client-version-2009"></a>Client FSLogix, version 2009 

Nous avons publié une nouvelle version du client FSLogix avec de nombreux correctifs et améliorations. Pour en savoir plus, consultez [notre billet de blog](https://social.msdn.microsoft.com/Forums/en-US/defe5828-fba4-4715-a68c-0e4d83eefa6b/release-notes-for-fslogix-apps-release-2009-29762130127?forum=FSLogix).

### <a name="rdp-shortpath-public-preview"></a>Préversion publique de RDP Shortpath

RDP Shortpath introduit une connectivité directe à votre hôte de session Windows Virtual Desktop en utilisant ExpressRoute et des VPN de point à site et de site à site. Il présente également le protocole de transport URCP. RDP Shortpath est conçu pour réduire la latence et les sauts de réseau, afin d’améliorer l’expérience utilisateur. Pour en savoir plus, consultez [RDP Shortpath de Windows Virtual Desktop](shortpath.md).

### <a name="azdesktopvirtualization-version-201"></a>Az.DesktopVirtualization, version 2.0.1

Nous avons publié la version 2.0.1 des applets de commande Windows Virtual Desktop. Cette mise à jour comprend des applets de commande qui vous permettent de gérer l’attachement d’application MSIX. Vous pouvez télécharger la nouvelle version depuis [ PowerShell Gallery](https://www.powershellgallery.com/packages/Az.DesktopVirtualization/2.0.1).

### <a name="azure-advisor-updates"></a>Mises à jour d’Azure Advisor

Azure Advisor comporte désormais une nouvelle recommandation en matière de conseils de proximité dans Windows Virtual Desktop. Pour plus d’informations, consultez le [site web Azure](https://azure.microsoft.com/updates/new-recommendations-from-azure-advisor/).

## <a name="october-2020"></a>Octobre 2020

Voici ce qui a changé en octobre 2020 :

### <a name="improved-performance"></a>performances améliorées

- Nous avons optimisé les performances en réduisant la latence des connexions dans les zones géographiques Azure suivantes :
    - Suisse
    - Canada

Vous pouvez maintenant utiliser l’[Outil d’estimation de l’expérience](https://azure.microsoft.com/services/virtual-desktop/assessment/) pour estimer la qualité de l’expérience utilisateur dans ces zones.

### <a name="azure-government-cloud-availability"></a>Disponibilité du cloud Azure Government

Le cloud Azure Government est désormais en disponibilité générale. Pour en savoir plus, consultez [notre billet de blog](https://azure.microsoft.com/updates/windows-virtual-desktop-is-now-generally-available-in-the-azure-government-cloud/).

### <a name="windows-virtual-desktop-azure-portal-updates"></a>Mises à jour du portail Azure Windows Virtual Desktop

Nous avons apporté des mises à jour au portail Azure Windows Virtual Desktop :

- Correction d’une erreur resourceID qui empêchait les utilisateurs d’ouvrir l’onglet « Sessions ».
- Rationalisation de l’interface utilisateur sous l’onglet « Hôtes de session ».
- Correction des paramètres « Valeurs par défaut », « Facilité d’utilisation » et « Restaurer les valeurs par défaut » sous les propriétés RDP.
- Harmonisation des fonctions de suppression sous tous les onglets.
- Le portail valide désormais les noms d’application dans le workflow « Ajouter une application ».
- Correction d’un problème en raison duquel les données d’exportation de l’hôte de session n’étaient pas alignées dans les colonnes.
- Correction d’un problème en raison duquel le portail ne pouvait pas récupérer les sessions utilisateur.
- Correction d’un problème lors de la récupération de l’hôte de session, qui se produisait quand la machine virtuelle était créée dans un groupe de ressources différent.
- Mise à jour de l’onglet « Hôte de session » pour lister les sessions actives et déconnectées.
- L’onglet « Applications » contient désormais des pages.
- Correction d’un problème en raison duquel le texte « requiert une ligne de commande » ne s’affichait pas correctement sous l’onglet « Liste des applications ».
- Correction d’un problème en raison duquel le portail ne pouvait pas déployer des pools d’hôtes ou des machines virtuelles lors de l’utilisation de la version en allemand de la Galerie d’images partagées.

### <a name="client-updates-for-october-2020"></a>Mises à jour des clients en octobre 2020

Nous avons publié de nouvelles versions des clients. Pour en savoir plus, consultez les articles suivants :

- [Windows](/windows-server/remote/remote-desktop-services/clients/windowsdesktop-whatsnew)
- [iOS](/windows-server/remote/remote-desktop-services/clients/ios-whatsnew)

Pour plus d’informations sur les autres clients, consultez [Mises à jour des clients](#client-updates).

## <a name="september-2020"></a>Septembre 2020

Voici ce qui a changé en septembre 2020 :

- Nous avons optimisé les performances en réduisant la latence des connexions dans les zones géographiques Azure suivantes :
    - Allemagne
    - Afrique du Sud (pour les environnements de validation uniquement)

Vous pouvez maintenant utiliser l’[Outil d’estimation de l’expérience](https://azure.microsoft.com/services/virtual-desktop/assessment/) pour estimer la qualité de l’expérience utilisateur dans ces zones.

- Nous avons publié la version 1.2.1364 du client Windows Desktop pour Windows Virtual Desktop. Dans cette mise à jour, nous avons apporté les modifications suivantes :
    - Correction d’un problème empêchant le bon fonctionnement de l’authentification unique (SSO) sur Windows 7.
    - Correction d’un problème qui provoquait la déconnexion du client quand un utilisateur ayant activé l’optimisation des médias pour Teams tentait d’appeler ou de rejoindre une réunion Teams alors qu’une autre application avait un flux audio ouvert en mode exclusif.
    - Résolution d’un problème où Teams ne listait pas les périphériques audio ou vidéo quand l’optimisation des médias pour Teams était activée.
    - Ajout d’un lien « Besoin d’aide avec les paramètres ? » à la page des paramètres de bureau.
    - Correction d’un problème lié au bouton « S’abonner » lors de l’utilisation de thèmes sombres à contraste élevé.
    
- Grâce à l’aide exceptionnelle de nos utilisateurs, nous avons résolu deux problèmes critiques pour le client Microsoft Store Bureau à distance. Nous continuerons à consulter les commentaires et à résoudre les problèmes au fur et à mesure que nous ouvrons notre publication progressive du client à d’autres utilisateurs dans le monde entier.
    
- Nous avons ajouté une nouvelle fonctionnalité qui vous permet de changer l’emplacement, l’image, le groupe de ressources, le nom du préfixe et la configuration réseau d’une machine virtuelle dans le cadre du workflow pour ajouter une machine virtuelle à votre déploiement dans le portail Azure.

- Les professionnels de l’informatique peuvent désormais gérer des machines virtuelles Windows 10 Entreprise hybrides jointes à Azure Active Directory en utilisant Microsoft Endpoint Manager. Pour plus d’informations, consultez [notre billet de blog](https://techcommunity.microsoft.com/t5/microsoft-endpoint-manager-blog/microsoft-endpoint-manager-announces-support-for-windows-virtual/ba-p/1681048).

## <a name="august-2020"></a>Août 2020

Voici ce qui a changé en août 2020 :

- Nous avons amélioré les performances pour réduire la latence des connexions dans les régions Azure suivantes : 

    - Royaume-Uni
    - France
    - Norvège
    - Corée du Sud

   Vous pouvez utiliser l’[Estimateur d’expérience](https://azure.microsoft.com/services/virtual-desktop/assessment/) pour avoir une idée générale de la façon dont ces modifications affecteront vos utilisateurs.

- Le client Bureau à distance Microsoft Store (v10.2.1522+) est maintenant en disponibilité générale ! Cette version du client Bureau à distance Microsoft Store est compatible avec Windows Virtual Desktop. Nous avons également introduit des flux d’interface utilisateur actualisés pour améliorer l’expérience utilisateur. Cette mise à jour comprend Fluent Design, les modes Light et Dark ainsi que de nombreuses autres modifications intéressantes. Nous avons également réécrit le client pour qu’il utilise le même moteur RDP (Remote Desktop Protocol) sous-jacent que les clients iOS, macOS et Android. Cela nous permet de fournir de nouvelles fonctionnalités plus rapidement sur toutes les plateformes. [Téléchargez le client](https://www.microsoft.com/p/microsoft-remote-desktop/9wzdncrfj3ps?rtc=1&activetab=pivot:overviewtab) et essayez-le !

- Nous avons corrigé un problème dans le client Teams Desktop (version 1.3.00.21759) où le client montrait seulement le fuseau horaire UTC dans le chat, les canaux et le calendrier. Le client mis à jour montre maintenant le fuseau horaire de la session à distance.

- Azure Advisor fait désormais partie de Windows Virtual Desktop. Quand vous accédez à Windows Virtual Desktop via le portail Azure, vous pouvez voir des recommandations pour optimiser votre environnement Windows Virtual Desktop. Découvrez plus d’informations sur [Azure Advisor](azure-advisor.md).

- Azure CLI prend désormais en charge Windows Virtual Desktop (`az desktopvirtualization`) pour vous aider à automatiser vos déploiements Windows Virtual Desktop. Pour obtenir la liste des commandes de l’extension, consultez [desktopvirtualization](/cli/azure/ext/desktopvirtualization/?view=azure-cli-latest&preserve-view=true).

- Nous avons mis à jour nos modèles de déploiement pour les rendre entièrement compatibles avec les interfaces Azure Resource Manager de Windows Virtual Desktop. Vous trouverez les modèles sur [GitHub](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates).

- Le portail US Gov de Windows Virtual Desktop est maintenant en préversion publique. Pour plus d’informations, consultez [notre annonce](https://azure.microsoft.com/updates/windows-virtual-desktop-is-now-available-in-the-azure-government-cloud-in-preview/).

## <a name="july-2020"></a>Juillet 2020  

En juillet, l’intégration de Windows Virtual Desktop avec Gestion des ressources Azure est devenue généralement disponible.

Voici ce qui a changé avec cette nouvelle mise en production : 

- La « mise en production de l’automne 2019 » est désormais appelée « Windows Virtual Desktop (classique) », tandis que la « mise en production du printemps 2020 » s’appelle désormais simplement « Windows Virtual Desktop ». Pour plus d’informations, consultez [ce billet de blog](https://azure.microsoft.com/blog/new-windows-virtual-desktop-capabilities-now-generally-available/). 

Pour en savoir plus sur les nouvelles fonctionnalités, consultez [ce billet de blog](https://techcommunity.microsoft.com/t5/itops-talk-blog/windows-virtual-desktop-spring-update-enters-public-preview/ba-p/1340245). 

### <a name="autoscaling-tool-update"></a>Mise à jour de l’outil de mise à l’échelle automatique

La dernière version de l’outil de mise à l’échelle automatique qui était en préversion est désormais généralement disponible. Cet outil se sert d’un compte Azure Automation et de l’application logique Azure pour arrêter et redémarrer automatiquement les machines virtuelles de l’hôte de la session dans un pool d’hôtes, ce qui réduit les coûts d’infrastructure. Pour en savoir plus, consultez [Procéder à la mise à l’échelle des hôtes de session à l’aide d’Azure Automation](set-up-scaling-script.md).

### <a name="azure-portal"></a>Portail Azure

Vous pouvez désormais effectuer les opérations suivantes avec le portail Azure dans Windows Virtual Desktop : 

- affecter directement des utilisateurs à des hôtes de session de bureau personnel ;  
- modifier le paramètre d’environnement de validation pour les pools d’hôtes. 

### <a name="diagnostics"></a>Diagnostics

Nous avons mis en production de nouvelles requêtes prédéfinies pour l’espace de travail Log Analytics. Pour accéder aux requêtes, accédez à **Journaux**, puis, sous **Catégorie**, sélectionnez **Windows Virtual Desktop**. Pour en savoir plus, consultez [Utiliser Log Analytics pour la fonctionnalité de diagnostic](diagnostics-log-analytics.md).

### <a name="update-for-remote-desktop-client-for-android"></a>Mise à jour du client Bureau à distance pour Android

Le [client Bureau à distance pour Android](https://play.google.com/store/apps/details?id=com.microsoft.rdc.androidx) prend à présent en charge les connexions Windows Virtual Desktop. Depuis la version 10.0.7, le client Android offre une nouvelle interface utilisateur pour une expérience utilisateur améliorée. Le client s’intègre également avec Microsoft Authenticator sur les appareils Android pour activer l’accès conditionnel lors de l’abonnement à des espaces de travail Windows Virtual Desktop.  

La version précédente du client Bureau à distance s’appelle désormais « Bureau à distance 8 ». Toutes les connexions existantes dans la version antérieure du client seront transférées sans problème vers le nouveau client. Le nouveau client a été réécrit dans le même moteur central de protocole RDP (Remote Desktop Protocol) sous-jacent que les clients iOS et macOS, ce qui accélère la mise en production de nouvelles fonctionnalités sur toutes les plateformes. 

### <a name="teams-update"></a>Mise à jour de Microsoft Teams

Nous avons apporté des améliorations à Microsoft Teams pour Windows Virtual Desktop. Plus important encore, Windows Virtual Desktop prend désormais en charge l’optimisation audio et vidéo pour le client Windows Desktop. La redirection améliore la latence en créant des chemins directs entre utilisateurs quand ceux-ci utilise l’audio ou la vidéo dans le cadre d’appels et de réunions. Une distance moins élevée signifie moins de tronçons, ce qui améliore la fluidité sonore et visuelle des appels. Pour plus d’informations, consultez [Utiliser Microsoft Teams sur Windows Virtual Desktop](teams-on-wvd.md).

## <a name="june-2020"></a>Juin 2020

Le mois dernier, nous avons introduit l’intégration de Windows Virtual Desktop avec Azure Resource Manager en préversion. Cette mise à jour comprend de nombreuses nouvelles fonctionnalités passionnantes que nous souhaiterions vous présenter. Voici les nouveautés de cette version de Windows Virtual Desktop.

### <a name="windows-virtual-desktop-is-now-integrated-with-azure-resource-manager"></a>Windows Virtual Desktop est désormais intégré avec Azure Resource Manager

Windows Virtual Desktop est maintenant intégré à Azure Resource Manager. Dans la dernière mise à jour, tous les objets Windows Virtual Desktop sont désormais des ressources Azure Resource Manager. Cette mise à jour est également intégrée avec le contrôle d’accès en fonction du rôle (RBAC) Azure. Pour en savoir plus, consultez [Qu’est-ce qu’Azure Resource Manager ?](../azure-resource-manager/management/overview.md).

Les conséquences de ce changement sont les suivantes :

- Windows Virtual Desktop est désormais intégré au portail Azure. Cela signifie que vous pouvez gérer tout directement dans le portail, sans avoir besoin de PowerShell, d’applications web ou d’outils tiers. Pour bien démarrer, consultez notre tutoriel intitulé [Créer un pool d’hôtes avec le portail Azure](create-host-pools-azure-marketplace.md).

- Avant cette mise à jour, vous pouviez uniquement publier des RemoteApps et des Desktops sur des utilisateurs individuels. Avec Azure Resource Manager, vous pouvez désormais publier des ressources sur des groupes Azure Active Directory.

- La version précédente de Windows Virtual Desktop avait quatre rôles d’administrateur intégrés que vous pouviez attribuer à un locataire ou un pool d’hôtes. Ces rôles sont désormais dans le [contrôle d’accès en fonction du rôle (RBAC) Azure](../role-based-access-control/overview.md). Vous pouvez appliquer ces rôles à chaque objet Azure Resource Manager Windows Virtual Desktop, ce qui vous permet d’avoir un modèle de délégation complet et riche.

- Dans cette mise à jour, vous n’avez plus besoin d’exécuter la Place de marché Azure ou le modèle GitHub de manière répétée pour étendre un pool d’hôtes. Il vous suffit d’accéder à votre pool d’hôtes dans le portail Azure et de sélectionner **+ Ajouter** pour déployer des hôtes de session supplémentaires.

- Le déploiement de pool d’hôtes est maintenant entièrement intégré à [Azure Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md). Shared Image Gallery est un service Azure distinct qui stocke des définitions d’images de machine virtuelle, notamment la gestion de versions d’images. Vous pouvez également utiliser la réplication globale pour copier et envoyer vos images vers d’autres régions Azure en vue d’un déploiement local.

- Les fonctions de supervision qui étaient auparavant effectuées à l’aide de PowerShell ou de l’application web Service de diagnostics ont été déplacées vers Log Analytics dans le portail Azure. Vous disposez désormais aussi de deux options pour visualiser vos rapports. Vous pouvez exécuter des requêtes Kusto et utiliser des classeurs pour créer des rapports visuels.

- Vous n’avez plus besoin de finaliser le consentement Azure Active Directory (Azure AD) pour utiliser Windows Virtual Desktop. Dans cette mise à jour, le locataire Azure AD sur votre abonnement Azure authentifie vos utilisateurs et fournit des contrôles Azure AD pour vos administrateurs.

### <a name="powershell-support"></a>Prise en charge de PowerShell

Nous avons ajouté de nouvelles applets de commande AzWvd au module Az Azure PowerShell avec cette mise à jour. Ce nouveau module est pris en charge dans PowerShell Core, qui s’exécute sur .NET Core.

Pour installer le module, suivez les instructions fournies dans [Configurer le module PowerShell pour Windows Virtual Desktop](powershell-module.md).

Vous pouvez également consulter la liste des commandes disponibles dans la [référence PowerShell AzWvd](/powershell/module/az.desktopvirtualization/?view=azps-4.2.0#desktopvirtualization&preserve-view=true).

Pour plus d’informations sur les nouvelles fonctionnalités, consultez [notre billet de blog](https://techcommunity.microsoft.com/t5/itops-talk-blog/windows-virtual-desktop-spring-update-enters-public-preview/ba-p/1340245).

### <a name="additional-gateways"></a>Passerelles supplémentaires

Nous avons ajouté un nouveau cluster de passerelle en Afrique du Sud pour réduire la latence de connexion.

### <a name="microsoft-teams-on-windows-virtual-desktop-preview"></a>Microsoft Teams sur Windows Virtual Desktop (Préversion)

Nous avons apporté des améliorations à Microsoft Teams pour Windows Virtual Desktop. Plus important encore, Windows Virtual Desktop prend désormais en charge la redirection audio et vidéo pour les appels. La redirection améliore la latence en créant des chemins directs entre les utilisateurs quand ils effectuent des appels audio ou vidéo. Une distance moins élevée signifie moins de tronçons, ce qui améliore la fluidité sonore et visuelle des appels.

Pour plus d’informations, consultez [notre billet de blog](https://azure.microsoft.com/updates/windows-virtual-desktop-media-optimization-for-microsoft-teams-is-now-available-in-public-preview/).

## <a name="client-updates"></a>Mises à jour de client

Consultez ces articles pour en savoir plus sur les mises à jour de nos clients pour Windows Virtual Desktop et Windows Virtual Desktop :

- [Windows](/windows-server/remote/remote-desktop-services/clients/windowsdesktop-whatsnew)
- [macOS](/windows-server/remote/remote-desktop-services/clients/mac-whatsnew)
- [iOS](/windows-server/remote/remote-desktop-services/clients/ios-whatsnew)
- [Android](/windows-server/remote/remote-desktop-services/clients/android-whatsnew)
- [Web](/windows-server/remote/remote-desktop-services/clients/web-client-whatsnew)

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur ce qui est prévu à l’avenir en consultant la [feuille de route Microsoft 365 Windows Virtual Desktop](https://www.microsoft.com/microsoft-365/roadmap?filters=Windows%20Virtual%20Desktop).
