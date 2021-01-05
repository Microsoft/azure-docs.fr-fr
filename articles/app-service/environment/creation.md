---
title: Créer un environnement App Service Environment
description: Découvrez comment créer un environnement App Service Environment.
author: ccompy
ms.assetid: 7690d846-8da3-4692-8647-0bf5adfd862a
ms.topic: article
ms.date: 11/16/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 12c9fd43933f9f54e013752c5fe482ce31650536
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95238392"
---
# <a name="create-an-app-service-environment"></a>Créer un environnement App Service Environment

> [!NOTE]
> Cet article traite d’App Service Environment v3 (préversion)
> 

L’environnement [ASE (App Service Environment)][Intro] est un déploiement monolocataire d’App Service, qui s’injecte dans votre réseau virtuel Azure.  ASEv3 prend uniquement en charge l’exposition des applications sur une adresse privée dans votre réseau virtuel. Lorsqu’un environnement ASEv3 est créé dans la préversion, trois ressources sont ajoutées à votre abonnement.

- Environnement App Service
- Zone privée Azure DNS
- Point de terminaison privé

Le déploiement d’un environnement ASE requiert l’utilisation de deux sous-réseaux.  Un sous-réseau contient le point de terminaison privé.  Ce sous-réseau peut être utilisé pour d’autres éléments, comme des machines virtuelles.  L’autre sous-réseau est utilisé pour les appels sortants effectués à partir de l’environnement ASE.  Ce sous-réseau ne peut pas être utilisé à d’autres fins que pour l’environnement ASE. 

## <a name="before-you-create-your-ase"></a>Avant de créer votre ASE

Une fois votre environnement ASE créé, vous ne pouvez plus modifier les éléments suivants :

- Emplacement
- Abonnement
- Resource group
- Réseau virtuel Azure utilisé
- Sous-réseaux utilisés
- Taille du sous-réseau
- Nom de votre environnement ASE

Le sous-réseau sortant doit être suffisamment grand pour contenir la taille maximale à laquelle votre environnement ASE sera mis à l’échelle. Choisissez un sous-réseau suffisamment grand pour prendre en charge vos besoins maximum de mise à l’échelle, car il ne peut pas être modifié après la création. La taille recommandée est /24 avec 256 adresses.

Une fois l’environnement ASE créé, vous pouvez y ajouter des applications. Si votre environnement ASEv3 n’a pas de plan App Service, des frais sont facturés, comme si vous disposiez d’une instance de plan App Service I1v2 dans votre environnement ASE.  

L’environnement ASEv3 est disponible uniquement dans certaines régions sélectionnées. D’autres régions seront ajoutées au fur et à mesure que la préversion passera en disponibilité générale. 

## <a name="creating-an-ase-in-the-portal"></a>Création d’un environnement ASE dans le portail

1. Pour créer un environnement ASE v3, recherchez **App Service Environment (préversion)** sur la marketplace.  
2. Paramètres de base :  Sélectionnez l’abonnement, sélectionnez ou créez le groupe de ressources, puis entrez le nom de votre environnement ASE.  Le nom de l’environnement ASE sera également utilisé pour le suffixe de domaine de votre environnement ASE.  Si le nom de votre environnement ASE est *contoso* le suffixe de domaine sera *contoso.appserviceenvironment.net*.  Ce nom est automatiquement défini dans votre zone privée Azure DNS utilisée par le réseau virtuel dans lequel l’environnement ASE est déployé. 

    ![App Service Environment - Créer un onglet de base](./media/creation/creation-basics.png)

3. Hébergement : Sélectionnez Préférences du système d’exploitation, Déploiement du groupe hôte. Les préférences du système d’exploitation indiquent le système d’exploitation que vous utiliserez initialement pour vos applications dans cet environnement ASE. Vous pouvez ajouter des applications d’un autre système d’exploitation après la création de l’environnement ASE. Le déploiement du groupe hôte est utilisé pour sélectionner un matériel dédié. Avec ASEv3, vous pouvez sélectionner Activé pour accéder à du matériel dédié. Vous êtes facturé pour l’intégralité de l’hôte dédié avec la création de l’environnement ASE, puis un prix réduit vous est facturé pour vos instances de plan App Service. 

    ![App Service Environment - Sélections d’hébergement](./media/creation/creation-hosting.png)

4. Réseaux :  Sélectionnez ou créez votre réseau virtuel, sélectionnez ou créez votre sous-réseau entrant, puis sélectionnez ou créez votre sous-réseau sortant. Tout sous-réseau utilisé pour le trafic sortant doit être vide et délégué à Microsoft.Web/hostingEnvironments. Si vous créez votre sous-réseau via le portail, le sous-réseau est automatiquement délégué pour vous.

    ![App Service Environment - Sélections réseau](./media/creation/creation-networking.png)

5. Vérifier et créer : Vérifiez que votre configuration est correcte, puis sélectionnez Créer. La création de votre environnement ASE prend environ une heure. 

    ![App Service Environment - Vérifier et créer](./media/creation/creation-review.png)

Une fois la création de l’environnement ASE terminée, vous pouvez le sélectionner comme emplacement lors de la création de vos applications. Pour en savoir plus sur la création d’applications dans votre nouvel environnement ASE, consultez [Utilisation d’un environnement App Service Environment][UsingASE]

## <a name="os-preference"></a>Préférence du système d’exploitation
Dans un environnement ASE, vous pouvez avoir des applications Windows, des applications Linux, ou les deux. Dans ASEv2, la préférence initiale sélectionnée lors de la création ajoute l’infrastructure à haute disponibilité pour ce système d’exploitation lors de la création de l’environnement ASE. Pour ajouter des applications de l’autre système d’exploitation, créez simplement les applications comme d’habitude et sélectionnez le système d’exploitation de votre choix. Dans ASEv3, cela n’affecte pas notablement le comportement du back-end.  

## <a name="dedicated-hosts"></a>Hôtes dédiés
L’environnement ASE est normalement déployé sur des machines virtuelles provisionnées sur un hyperviseur multilocataire. Si vous devez déployer sur des systèmes dédiés, notamment le matériel, vous pouvez provisionner votre environnement ASE sur des hôtes dédiés. Dans la préversion ASEv3 initiale, les hôtes dédiés vont par paire. Chaque hôte dédié se trouve dans une zone de disponibilité distincte, si la région le permet. Les déploiements ASE basés sur un hôte dédié ne sont pas facturés au tarif normal. Des frais sont facturés pour l’hôte dédié, puis pour chaque instance de plan App Service.  

<!--Links-->
[Intro]: ./overview.md
[MakeASE]: ./creation.md
[ASENetwork]: ./networking.md
[UsingASE]: ./using.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/network-security-groups-overview.md
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
[ConfigureSSL]: ../configure-ssl-certificate.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[AppDeploy]: ../deploy-local-git.md
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../web-application-firewall/ag/ag-overview.md
[logalerts]: ../../azure-monitor/platform/alerts-log.md
