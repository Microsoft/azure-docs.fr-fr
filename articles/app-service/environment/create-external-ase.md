---
title: Créer un environnement ASE externe
description: Découvrez comment créer un environnement App Service contenant une application ou un environnement ASE (vide) autonome.
author: ccompy
ms.assetid: 94dd0222-b960-469c-85da-7fcb98654241
ms.topic: article
ms.date: 06/13/2017
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: c953c31792b8d01199d409cbd91124138a6ebb15
ms.sourcegitcommit: dd45ae4fc54f8267cda2ddf4a92ccd123464d411
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92927445"
---
# <a name="create-an-external-app-service-environment"></a>Créer un environnement App Service externe

Un environnement Azure App Service est un déploiement d’Azure App Service dans un sous-réseau de réseau virtuel Azure.

> [!NOTE]
> Chaque instance d’App Service Environment a une adresse IP virtuelle qui peut être utilisée pour contacter App Service Environment.

Il existe deux façons de déployer un environnement App Service (ASE, App Service Environment) :

- avec une adresse IP virtuelle sur une adresse IP externe, solution souvent appelée ASE externe ;
- avec l'adresse IP virtuelle sur une adresse IP interne, solution souvent appelée environnement App Service ILB car le point de terminaison interne est un équilibreur de charge interne (ILB, Internal Load Balancer).

Cet article vous explique comment créer un ASE externe. Pour obtenir une présentation de l’environnement App Service, consultez [Présentation de l’environnement App Service Environment][Intro]. Pour plus d’informations sur la façon de créer un environnement App Service ILB, consultez [Créer et utiliser un équilibreur de charge interne avec un environnement Azure App Service Environment][MakeILBASE].

## <a name="before-you-create-your-ase"></a>Avant de créer votre ASE

Une fois l’environnement App Service créé, les éléments suivants ne peuvent plus être modifiés :

- Emplacement
- Abonnement
- Resource group
- Réseau virtuel utilisé
- Sous-réseau utilisé
- Taille du sous-réseau

> [!NOTE]
> Quand vous choisissez un réseau virtuel et spécifiez un sous-réseau, vérifiez que leur taille leur permet de prendre en compte les évolutions futures et les besoins de mise à l'échelle. Nous vous recommandons une taille de `/24` avec 256 adresses.
>

## <a name="three-ways-to-create-an-ase"></a>Trois façons de créer un ASE

Il existe trois façons de créer un environnement App Service :

- **Dans le cadre d’un plan App Service**. Avec cette méthode, l’environnement App Service et le plan App Service sont créés lors de la même étape.
- **De façon autonome**. Avec cette méthode, seul l’environnement App Service est créé. Le processus de création est plus avancé et permet de créer un environnement App Service avec un équilibreur de charge interne (ILB).
- **À l’aide d’un modèle Azure Resource Manager**. Cette méthode est destinée aux utilisateurs expérimentés. Pour plus d’informations, consultez [Créer un environnement ASE à l’aide d’un modèle][MakeASEfromTemplate].

Un environnement App Service externe a une adresse IP virtuelle publique, ce qui signifie que le trafic HTTP/HTTPS entrant dans les applications de l’environnement App Service atteint une adresse IP accessible via Internet. Un environnement App Service avec un équilibreur de charge interne (ILB) a une adresse IP issue du sous-réseau utilisé par l’environnement App Service. Les applications hébergées dans un environnement App Service ILB ne sont pas exposées directement à Internet.

## <a name="create-an-ase-and-an-app-service-plan-together"></a>Créer un ASE et un plan App Service ensemble

Le plan App Service est un conteneur d’applications. Lorsque vous créez une application dans App Service, vous devez sélectionner ou créer un plan App Service. Les environnements App Service contiennent les plans App Service, et les plans App Service contiennent les applications.

Pour créer un environnement App Service en même temps que le plan App Service :

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez **Créer une ressource** > **Web + Mobile** > **Application web**.

    ![Capture d’écran du portail Azure montrant l’option Web + Mobile sélectionnée dans Place de marché Azure et l’écran de création d’une nouvelle application web ouvert à droite.][1]

2. Sélectionnez votre abonnement. L’application et l’environnement App Service sont créés dans les mêmes abonnements.

3. Sélectionnez ou créez un groupe de ressources. Vous pouvez utiliser des groupes de ressources pour gérer des ressources Azure connexes en tant qu’unité. Les groupes de ressources sont également utiles lorsque vous souhaitez établir des règles de contrôle d’accès en fonction du rôle (RBAC) pour vos applications. Pour plus d’informations, consultez l’article [Présentation d’Azure Resource Manager][ARMOverview].

4. Sélectionnez votre système d’exploitation (Windows, Linux ou Docker). 

5. Cliquez sur le plan App Service, puis sélectionnez **Créer un nouveau**. Les applications web Linux et les applications web Windows ne peuvent pas se trouver dans le même plan App Service, mais elles peuvent être dans le même environnement ASE. 

    ![Capture d’écran du portail Azure montrant le volet Application web, le volet Plan App Service et le volet Nouveau plan App Service ouverts.][2]

6. Dans la liste déroulante **Emplacement** , sélectionnez la région où vous souhaitez créer l’environnement App Service. Si vous sélectionnez un environnement App Service existant, aucun environnement App Service n’est créé. Le plan App Service est créé dans l’environnement App Service que vous avez sélectionné. 

7. Sélectionnez **Niveau tarifaire** , puis choisissez l’une des références SKU de tarification **Isolé**. Si vous choisissez une carte de référence SKU **Isolé** et un emplacement autre qu’un environnement App Service, un nouvel environnement App Service est créé à cet emplacement. Pour démarrer le processus de création d’un environnement App Service, cliquez sur **Sélectionner**. La référence SKU **Isolé** n’est disponible qu’avec un environnement App Service. Vous ne pouvez pas utiliser une autre référence SKU de tarification dans un environnement App Service qui n’est pas **Isolé**. 

    ![Sélection du niveau tarifaire][3]

8. Indiquez le nom de votre ASE. Ce nom est utilisé dans le nom adressable de vos applications. Si le nom de l’environnement App Service est _appsvcenvdemo_ , le nom du domaine est *.appsvcenvdemo.p.azurewebsites.net*. Si vous créez une application nommée *mytestapp* , elle est adressable à l’adresse mytestapp.appsvcenvdemo.p.azurewebsites.net. Vous ne pouvez pas utiliser d’espace blanc dans le nom. Si vous utilisez des majuscules dans le nom, le nom de domaine correspond à la version complète de ce nom en minuscules.

    ![Nom du nouveau plan App Service][4]

9. Spécifiez les informations concernant votre réseau virtuel Azure. Choisissez **Créer un nouveau** ou **Sélectionner**. L’option permettant de sélectionner un réseau virtuel existant est disponible uniquement si un réseau virtuel se trouve dans la région sélectionnée. Si vous sélectionnez **Créer un nouveau** , entrez un nom pour le réseau virtuel. Un nouveau réseau virtuel Resource Manager portant ce nom est alors créé. Il utilise l’espace d’adressage `192.168.250.0/23` dans la région sélectionnée. Si vous choisissez **Sélectionner** , vous devez :

    a. Sélectionner le bloc d’adresses du réseau virtuel, si vous en avez plusieurs

    b. Entrer un nom pour le nouveau sous-réseau

    c. Sélectionner la taille du sous-réseau. *Veillez à définir une taille de sous-réseau suffisamment grande pour s’adapter à toute future croissance de votre environnement App Service.* Nous vous recommandons d’utiliser `/24`, qui comprend 256 adresses et peut gérer un environnement ASE de taille maximale. Par exemple, `/28` est déconseillé, car 16 adresses seulement sont disponibles. L’infrastructure utilise au moins 7 adresses et Azure Networking 5 autres. Dans un sous-réseau `/28`, vous vous retrouvez avec une mise à l’échelle maximale de 4 instances de plan App Service pour un ASE externe et uniquement 3 instances de plan App Service pour un ASE ILB.

    d. Sélectionner la plage IP du sous-réseau

10. Sélectionnez **Créer** pour créer l’environnement App Service. Ce processus crée également le plan App Service et l’application. L’environnement App Service, le plan App Service et l’application sont regroupés au sein du même abonnement et du même groupe de ressources. Si votre environnement App Service a besoin d’un groupe de ressources distinct, ou si vous avez besoin d’un environnement App Service ILB, procédez aux étapes permettant de créer un environnement App Service de manière autonome.

## <a name="create-an-ase-and-a-linux-web-app-using-a-custom-docker-image-together"></a>Créer un environnement ASE et une application web Linux à l’aide d’une image Docker personnalisée

1. Dans le [portail Azure](https://portal.azure.com/), cliquez sur **Créer une ressource** > **Web + Mobile** > **Web App pour conteneurs.** 

    ![Capture d’écran du portail Azure montrant l’option Web + Mobile sélectionnée dans Place de marché Azure et le volet Web App pour conteneurs ouvert à droite.][7]

1. Sélectionnez votre abonnement. L’application et l’environnement App Service sont créés dans les mêmes abonnements.

1. Sélectionnez ou créez un groupe de ressources. Vous pouvez utiliser des groupes de ressources pour gérer des ressources Azure connexes en tant qu’unité. Les groupes de ressources sont également utiles lorsque vous souhaitez établir des règles de contrôle d’accès en fonction du rôle (RBAC) pour vos applications. Pour plus d’informations, consultez l’article [Présentation d’Azure Resource Manager][ARMOverview].

1. Cliquez sur le plan App Service, puis sélectionnez **Créer un nouveau**. Les applications web Linux et les applications web Windows ne peuvent pas se trouver dans le même plan App Service, mais elles peuvent être dans le même environnement ASE. 

    ![Capture d’écran du portail Azure montrant le volet Web App pour conteneurs, le volet Plan App Service et le volet Nouveau plan App Service ouverts.][8]

1. Dans la liste déroulante **Emplacement** , sélectionnez la région où vous souhaitez créer l’environnement App Service. Si vous sélectionnez un environnement App Service existant, aucun environnement App Service n’est créé. Le plan App Service est créé dans l’environnement App Service que vous avez sélectionné. 

1. Sélectionnez **Niveau tarifaire** , puis choisissez l’une des références SKU de tarification **Isolé**. Si vous choisissez une carte de référence SKU **Isolé** et un emplacement autre qu’un environnement App Service, un nouvel environnement App Service est créé à cet emplacement. Pour démarrer le processus de création d’un environnement App Service, cliquez sur **Sélectionner**. La référence SKU **Isolé** n’est disponible qu’avec un environnement App Service. Vous ne pouvez pas utiliser une autre référence SKU de tarification dans un environnement App Service qui n’est pas **Isolé**. 

    ![Sélection du niveau tarifaire][3]

1. Indiquez le nom de votre ASE. Ce nom est utilisé dans le nom adressable de vos applications. Si le nom de l’environnement App Service est _appsvcenvdemo_ , le nom du domaine est *.appsvcenvdemo.p.azurewebsites.net*. Si vous créez une application nommée *mytestapp* , elle est adressable à l’adresse mytestapp.appsvcenvdemo.p.azurewebsites.net. Vous ne pouvez pas utiliser d’espace blanc dans le nom. Si vous utilisez des majuscules dans le nom, le nom de domaine correspond à la version complète de ce nom en minuscules.

    ![Nom du nouveau plan App Service][4]

1. Spécifiez les informations concernant votre réseau virtuel Azure. Choisissez **Créer un nouveau** ou **Sélectionner**. L’option permettant de sélectionner un réseau virtuel existant est disponible uniquement si un réseau virtuel se trouve dans la région sélectionnée. Si vous sélectionnez **Créer un nouveau** , entrez un nom pour le réseau virtuel. Un nouveau réseau virtuel Resource Manager portant ce nom est alors créé. Il utilise l’espace d’adressage `192.168.250.0/23` dans la région sélectionnée. Si vous choisissez **Sélectionner** , vous devez :

    a. Sélectionner le bloc d’adresses du réseau virtuel, si vous en avez plusieurs

    b. Entrer un nom pour le nouveau sous-réseau

    c. Sélectionner la taille du sous-réseau. *Veillez à définir une taille de sous-réseau suffisamment grande pour s’adapter à toute future croissance de votre environnement App Service.* Nous recommandons `/24`, qui comprend 128 adresses et peut gérer un environnement App Service de taille maximale. Par exemple, `/28` est déconseillé, car 16 adresses seulement sont disponibles. L’infrastructure utilise au moins 7 adresses et Azure Networking 5 autres. Dans un sous-réseau `/28`, vous vous retrouvez avec une mise à l’échelle maximale de 4 instances de plan App Service pour un ASE externe et uniquement 3 instances de plan App Service pour un ASE ILB.

    d. Sélectionner la plage IP du sous-réseau

1.  Sélectionnez Configurer le conteneur.
    * Entrez le nom de votre image personnalisée (vous pouvez utiliser Azure Container Registry, Docker Hub et votre propre registre privé). Si vous ne souhaitez pas utiliser votre propre conteneur personnalisé, vous pouvez simplement ajouter votre code et utiliser une image intégrée avec App Service sur Linux, en suivant les instructions ci-dessus. 

    ![Configurer le conteneur][9]

1. Sélectionnez **Créer** pour créer l’environnement App Service. Ce processus crée également le plan App Service et l’application. L’environnement App Service, le plan App Service et l’application sont regroupés au sein du même abonnement et du même groupe de ressources. Si votre environnement App Service a besoin d’un groupe de ressources distinct, ou si vous avez besoin d’un environnement App Service ILB, procédez aux étapes permettant de créer un environnement App Service de manière autonome.


## <a name="create-an-ase-by-itself"></a>Créer un ASE autonome

Lorsque vous créez un environnement App Service autonome, celui-ci est vide. Même vide, un environnement App Service occasionne des frais mensuels pour l’infrastructure. Effectuez les étapes permettant de créer un environnement App Service ILB ou un environnement App Service dans son propre groupe de ressources. Une fois l’environnement App Service créé, vous pouvez y créer des applications à l’aide de la procédure normale. Sélectionnez votre nouvel environnement App Service comme emplacement.

1. Recherchez **App Service Environment** dans la Place de marché Azure, ou sélectionnez **Créer une ressource** > **Web + Mobile** > **App Service Environment**. 

1. Entrez le nom de votre environnement App Service. Ce nom est utilisé pour les applications créées dans l’environnement App Service. Si le nom est *mynewdemoase* , le nom du sous-domaine est *.mynewdemoase.p.azurewebsites.net*. Si vous créez une application nommée *mytestapp* ,celle-ci est adressable à l’adresse mytestapp.mynewdemoase.p.azurewebsites.net. Vous ne pouvez pas utiliser d’espace blanc dans le nom. Si vous utilisez des majuscules dans le nom, le nom de domaine correspond à la version complète de ce nom en minuscules. Si vous utilisez un équilibreur de charge interne (ILB), le nom de votre environnement App Service n’est pas utilisé dans votre sous-domaine, mais il est explicitement indiqué lors de la création de l’environnement App Service.

    ![Attribution d’un nom à l’environnement App Service][5]

1. Sélectionnez votre abonnement. Cet abonnement est également celui que toutes les applications de l’environnement App Service utilisent. Vous ne pouvez pas placer votre environnement App Service dans un réseau virtuel se trouvant dans un autre abonnement.

1. Sélectionnez ou spécifiez un nouveau groupe de ressources. Le groupe de ressources utilisé pour votre environnement App Service doit être le même que celui utilisé pour votre réseau virtuel. Si vous sélectionnez un réseau virtuel existant, la sélection du groupe de ressources pour votre environnement App Service est mise à jour pour refléter celle de votre réseau virtuel. *Si vous utilisez un modèle Resource Manager, vous pouvez créer un environnement App Service avec un groupe de ressources différent de celui du réseau virtuel.* Pour créer un environnement App Service à partir d’un modèle, consultez [Créer un environnement ASE à l’aide d’un modèle][MakeASEfromTemplate].

    ![Sélection du groupe de ressources][6]

1. Sélectionnez le réseau virtuel et l’emplacement. Vous pouvez créer un réseau virtuel ou sélectionner un réseau virtuel existant : 

    * Si vous sélectionnez un nouveau réseau virtuel, vous pouvez spécifier un nom et un emplacement. 
    
    * Le nouveau réseau virtuel se voit affecter la plage d’adresses 192.168.250.0/23 et un sous-réseau nommé default. Le sous-réseau reçoit la plage d’adresses 192.168.250.0/24. Vous pouvez uniquement sélectionner un réseau virtuel Resource Manager. La sélection du **type d’adresse IP virtuelle** détermine si votre environnement App Service est accessible directement à partir d’Internet (externe) ou s’il utilise un équilibreur de charge interne (ILB). Pour plus d’informations, consultez [Créer et utiliser un équilibreur de charge interne avec un environnement Azure App Service Environment][MakeILBASE]. 

      * Si vous sélectionnez un **Type d’adresse IP virtuelle** **Externe** , vous pouvez choisir le nombre d’adresses IP externes avec lesquelles est créé le système, en vue d’une utilisation du protocole SSL basé sur IP. 
    
      * Si vous sélectionnez un **Type d’adresse IP virtuelle** **Interne** , vous devez spécifier le domaine que votre environnement App Service utilise. Vous pouvez déployer un environnement App Service dans un réseau virtuel qui utilise des plages d’adresses publiques ou privées. Pour utiliser un réseau virtuel avec une plage d’adresses publiques, vous devez créer le réseau virtuel à l’avance. 
    
    * Si vous sélectionnez un réseau virtuel existant, un nouveau sous-réseau est créé en même temps que l’environnement App Service. *Vous ne pouvez pas utiliser un sous-réseau créé au préalable dans le portail. Si vous utilisez un modèle Resource Manager, vous pouvez créer un environnement App Service avec un sous-réseau existant.* Pour créer un environnement App Service à partir d’un modèle, consultez [Créer un environnement ASE à partir d’un modèle][MakeASEfromTemplate].

## <a name="app-service-environment-v1"></a>Environnement App Service v1

Vous pouvez toujours créer des instances de la première version d’App Service Environment (ASEv1). Pour commencer, recherchez **App Service Environment v1** dans la Place de marché. Créez l’environnement App Service de la même façon que pour un environnement App Service autonome. Une fois créée, votre instance d’ASEv1 comprend deux front-ends et deux Workers. Avec ASEv1, vous devez gérer les front-ends et les Workers. Ils ne sont pas ajoutés automatiquement lors de la création de vos plans App Service. Les front-ends servent de points de terminaison HTTP/HTTPS et envoient le trafic aux Workers. Les Workers correspondent aux rôles qui hébergent vos applications. Vous pouvez ajuster la quantité de front-ends et de Workers après la création de l’environnement App Service. 

Pour plus d’informations sur ASEv1, consultez [Présentation d’App Service Environment v1][ASEv1Intro]. Pour plus d’informations sur la mise à l’échelle, la gestion et la supervision d’ASEv1, consultez [Configuration d’un environnement App Service][ConfigureASEv1].

<!--Image references-->
[1]: ./media/how_to_create_an_external_app_service_environment/createexternalase-create.png
[2]: ./media/how_to_create_an_external_app_service_environment/createexternalase-aspcreate.png
[3]: ./media/how_to_create_an_external_app_service_environment/createexternalase-pricing.png
[4]: ./media/how_to_create_an_external_app_service_environment/createexternalase-embeddedcreate.png
[5]: ./media/how_to_create_an_external_app_service_environment/createexternalase-standalonecreate.png
[6]: ./media/how_to_create_an_external_app_service_environment/createexternalase-network.png
[7]: ./media/how_to_create_an_external_app_service_environment/createexternalase-createwafc.png
[8]: ./media/how_to_create_an_external_app_service_environment/createexternalase-aspcreatewafc.png
[9]: ./media/how_to_create_an_external_app_service_environment/createexternalase-configurecontainer.png



<!--Links-->
[Intro]: ./intro.md
[MakeExternalASE]: ./create-external-ase.md
[MakeASEfromTemplate]: ./create-from-template.md
[MakeILBASE]: ./create-ilb-ase.md
[ASENetwork]: ./network-info.md
[UsingASE]: ./using-an-ase.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/network-security-groups-overview.md
[ConfigureASEv1]: app-service-web-configure-an-app-service-environment.md
[ASEv1Intro]: app-service-app-service-environment-intro.md
[webapps]: ../overview.md
[mobileapps]: /previous-versions/azure/app-service-mobile/app-service-mobile-value-prop
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
