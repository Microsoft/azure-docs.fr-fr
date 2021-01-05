---
title: Créer une application web dans ASE v1
description: Découvrez comment créer des applications web dans un environnement App Service v1. Ce document s’adresse uniquement aux clients qui utilisent l’environnement ASE v1 hérité.
author: ccompy
ms.assetid: 983ba055-e9e4-495a-9342-fd3708dcc9ac
ms.topic: article
ms.date: 07/11/2017
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: d0ac8e0cf187d1a6a329f2c7332c9ec37018d4cf
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005229"
---
# <a name="create-a-web-app-in-an-app-service-environment-v1"></a>Créer une application web dans un environnement App Service Environment v1

> [!NOTE]
> Cet article traite de l’environnement App Service Environment v1.  Il existe une version plus récente de l’environnement App Service Environment, plus facile à utiliser et qui s’exécute sur des infrastructures plus puissantes. Pour en savoir plus sur la nouvelle version, commencez par la section [Présentation de l’environnement App Service Environment](intro.md).
> 

## <a name="overview"></a>Vue d’ensemble
Ce didacticiel explique comment créer des applications web et des plans App Service dans un [environnement App Service Environment v1](app-service-app-service-environment-intro.md) (ASE). 

> [!NOTE]
> Si vous voulez apprendre à créer une application web mais pas spécialement dans un environnement App Service, consultez [Créer une application web .NET](../quickstart-dotnetcore.md) ou l’un des didacticiels associés pour d’autres langages et infrastructures.
> 
> 

## <a name="prerequisites"></a>Prérequis
Ce didacticiel part du principe que vous avez créé un environnement App Service. Si ce n’est pas encore le cas, consultez [Créer un environnement App Service](app-service-web-how-to-create-an-app-service-environment.md). 

## <a name="create-a-web-app"></a>Créer une application web
1. Dans le [portail Azure](https://portal.azure.com/), cliquez sur **Créer une ressource > Web + Mobile > Application web**. 
   
    ![Capture d’écran montrant où créer une application web dans le Portail Azure.][1]
2. Sélectionnez votre abonnement.  
   
    Si vous disposez de plusieurs abonnements, n’oubliez pas que pour créer une application dans votre environnement App Service, vous devez utiliser le même abonnement que celui utilisé lors de la création de l’environnement. 
3. Sélectionnez ou créez un groupe de ressources.
   
    Les *groupes de ressources* vous permettent de gérer les ressources Azure connexes en tant qu’unité et sont utiles lors de l’établissement de règles de *contrôle d’accès en fonction du rôle Azure (Azure RBAC)* pour vos applications. Pour plus d’informations, consultez [Présentation d’Azure Resource Manager][ResourceGroups]. 
4. Sélectionnez ou créez un plan App Service.
   
    *plans App Service* sont des ensembles gérés d’applications web.  Normalement, lorsque vous sélectionnez la tarification, le prix facturé s’applique au plan App Service et non aux applications individuelles. Dans un ASE, vous payez les instances de calcul allouées à l’ASE plutôt que ce que vous avez répertorié avec votre ASP.  Pour augmenter le nombre d’instances d’une application web, vous augmentez la taille des instances de votre plan App Service, et toutes les applications web du plan sont affectées.  Certaines fonctionnalités, telles que les emplacements de site ou l'intégration de réseau virtuel, présentent aussi des restrictions de quantité à l'intérieur du plan.  Pour plus d’informations, consultez la rubrique [Présentation des plans Azure App Service](../overview-hosting-plans.md)
   
    Vous pouvez identifier les plans App Service dans votre environnement App Service en examinant l’emplacement indiqué sous le nom du plan.  
   
    ![Capture d’écran montrant où voir les plans de App Service dans votre App Service Environment.][5]
   
    Si vous souhaitez utiliser un plan App Service existant dans votre environnement App Service, sélectionnez ce plan. Si vous souhaitez créer un plan App Service, consultez la section suivante de ce didacticiel : [Créer un plan App Service dans un environnement App Service](#createplan).
5. Entrez le nom de votre application web, puis cliquez sur **Créer**. 
   
    Si votre ASE utilise une adresse VIP externe, l’URL d’une application dans un ASE est : [*sitename*]. [*nom de votre environnement App Service*]. p.azurewebsites.net au lieu de [*sitename*]. azurewebsites.net
   
    Si votre ASE utilise une adresse VIP interne, alors l’URL d’une application de l’ASE aura le format : [*nomsite*]. [*sous-domaine spécifié lors de la création de l’ASE*]   
    Après avoir sélectionné votre ASP lors de la création de l’ASE vous verrez le sous-domaine mis à jour sous **Nom**

## <a name="create-an-app-service-plan"></a><a name="createplan"></a> Créer un plan App Service
Lorsque vous créez un plan App Service dans un environnement App Service, vos choix de travaux sont différents, car il n’existe pas de travaux partagés dans un environnement App Service.  Les travaux que vous devez utiliser sont ceux qui ont été alloués à l’ASE par l’administrateur.  Cela signifie que pour créer un plan, le nombre de travaux alloués à votre pool de travail ASE doit être supérieur au nombre total d’instances à travers l’ensemble de vos plans déjà présents dans ce pool de travail.  Si vous n’avez pas suffisamment de travaux dans votre pool de travail ASE pour créer votre plan, demandez à l’administrateur de votre ASE de les ajouter.

L’absence du choix de tarification constitue une autre différence avec les plans App Service hébergés par un environnement App Service.  Lorsque vous avez un environnement App Service, vous payez les ressources de calcul utilisées par le système et n’avez pas de frais supplémentaires pour les plans de cet environnement.  Normalement lorsque vous créez un plan App Service, vous sélectionnez un niveau de tarification qui détermine votre facturation.  Un environnement App Service est essentiellement un emplacement privé où vous pouvez créer un contenu.  Vous payez l'environnement et non l'hébergement de votre contenu.

Les instructions suivantes indiquent comment créer un plan App Service lors de la création d’une application web, comme expliqué dans la section précédente du didacticiel.

1. Cliquez sur **Nouveau** dans l’interface utilisateur de sélection de plan et attribuez un nom à votre plan, exactement comme vous le feriez en dehors d’un environnement ASE.
2. Dans le sélecteur d’emplacements, sélectionnez l’environnement ASE que vous souhaitez utiliser.
   
    Un environnement App Service étant essentiellement un emplacement de déploiement privé, il s’affiche sous Emplacement. 
   
    ![Capture d’écran montrant le sélecteur d’emplacement pour la sélection de l’ASE de votre choix.][2]
   
    Après la sélection d’un environnement ASE dans le sélecteur d’emplacements, l’interface utilisateur de création de plan App Service se met à jour.  L’emplacement affiche désormais le nom du système ASE et la région dans laquelle il se trouve, et le sélecteur de plans de tarification est remplacé par un sélecteur de pool de travail.  
   
    ![Capture d’écran montrant les détails du système ASE après avoir sélectionné l’ASE dans le sélecteur d’emplacement.][3]

### <a name="selecting-a-worker-pool"></a>Sélection d’un pool de travail
Normalement, dans Azure App Service et en dehors d’un environnement App Service, il existe 3 tailles de calcul disponibles avec la sélection d’un plan de tarification dédié.  De la même manière, pour un ASE, vous pouvez définir jusqu’à 3 pools de travaux et spécifier la taille de calcul utilisée pour ce pool de travail.  Pour les clients de l’ASE, cela signifie que, au lieu de sélectionner un plan de tarification avec une taille de calcul pour votre plan App Service, vous sélectionnez ce qu’on appelle un *pool de travail*.  

L’interface utilisateur de sélection du pool de travaux affiche la taille de calcul utilisée pour le pool de travaux sous le nom.  La quantité disponible fait référence au nombre d’instances de calcul disponibles pour une utilisation dans le pool.  Le pool total peut avoir plus d’instances que ce nombre, mais la valeur fait simplement référence à ceux qui ne sont pas en cours d’utilisation.  Si vous avez besoin d’ajuster votre environnement App Service pour ajouter des ressources de calcul, consultez [Configuration de votre environnement App Service](app-service-web-configure-an-app-service-environment.md).

![Capture d’écran montrant le volet Worker Pool dans lequel vous pouvez sélectionner Worker Pools pour l’ASE.][4]

Dans cet exemple, vous constatez que seuls deux pools de travaux sont disponibles. La raison en est que l’administrateur de l’ASE n’a alloué les hôtes qu’à ces deux pools de travaux.  Le troisième apparaît lorsque des machines virtuelles lui ont été attribuées.  

## <a name="after-web-app-creation"></a>Après la création d'une application web
Il existe quelques considérations à prendre en compte pour l’exécution d’applications web et la gestion de plans App Service dans un environnement App Service.  

Comme indiqué précédemment, le propriétaire de l’environnement App Service est responsable de la taille du système et, par conséquent, il est également chargé de s’assurer qu’il existe une capacité suffisante pour héberger les plans App Service souhaités. Si aucun travail n’est disponible, vous ne pourrez pas créer votre plan App Service.  Il est en de même pour l'augmentation de la taille de votre application web.  Si vous avez besoin de davantage d'instances, demandez à l'administrateur de votre environnement App Service d'ajouter plus de travaux.

Après avoir créé votre application web et votre plan App Service, il peut être judicieux de les mettre à l’échelle.  Dans un environnement App Service, vous devez toujours avoir au moins 2 instances de votre plan App Service pour fournir une tolérance de panne à vos applications.  La mise à l’échelle d’un plan App Service dans un ASE s’effectue selon la procédure normale via l’interface utilisateur du plan App Service.  Pour plus d’informations sur la mise à l’échelle, consultez [Mise à l’échelle des applications web dans un environnement App Service](app-service-web-scale-a-web-app-in-an-app-service-environment.md)

<!--Image references-->
[1]: ./media/app-service-web-how-to-create-a-web-app-in-an-ase/createaspnewwebapp.png
[2]: ./media/app-service-web-how-to-create-a-web-app-in-an-ase/createasplocation.png
[3]: ./media/app-service-web-how-to-create-a-web-app-in-an-ase/createaspselected.png
[4]: ./media/app-service-web-how-to-create-a-web-app-in-an-ase/createaspworkerpool.png
[5]: ./media/app-service-web-how-to-create-a-web-app-in-an-ase/selectaspinase.png

<!--Links-->
[WhatisASE]: app-service-app-service-environment-intro.md
[Appserviceplans]: ../overview-hosting-plans.md
[HowtoCreateASE]: app-service-web-how-to-create-an-app-service-environment.md
[HowtoScale]: app-service-web-scale-a-web-app-in-an-app-service-environment.md
[HowtoConfigureASE]: app-service-web-configure-an-app-service-environment.md
[ResourceGroups]: ../../azure-resource-manager/management/overview.md
[AzurePowershell]: /powershell/azure/?view=azps-3.8.0