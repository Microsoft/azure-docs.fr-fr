---
title: Mettre à l’échelle une application dans ASE v1
description: Mise à l'échelle d'une application dans un environnement App Service. Ce document s’adresse uniquement aux clients qui utilisent l’environnement ASE v1 hérité.
author: ccompy
ms.assetid: 78eb1e49-4fcd-49e7-b3c7-f1906f0f22e3
ms.topic: article
ms.date: 10/17/2016
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 0e665ec27da0a898e754817f946b965ac7360fda
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86220556"
---
# <a name="scaling-apps-in-an-app-service-environment-v1"></a>Mise à l’échelle d’applications dans un environnement App Service v1
Dans Azure App Service, il existe normalement trois éléments que vous pouvez mettre à l'échelle :

* Plan de tarification
* Taille du travail 
* Nombre d'instances

Dans un environnement App Service, il est inutile de sélectionner ou de modifier le plan de tarification.  En termes de fonctionnalités, il est déjà au niveau de tarification Premium.  

En ce qui concerne les tailles de travaux, l'administrateur de l'environnement App Service peut affecter la taille de la ressource de calcul à utiliser pour chaque pool de travaux.  Cela signifie que vous pouvez avoir le pool de travaux 1 avec les ressources de calcul P4 et le pool de travaux 2 avec les ressources de calcul P1, si vous le souhaitez.  Ils n'ont pas à être par ordre de taille.  Pour plus d’informations sur les tailles et leurs tarifications, consultez le document [Tarification de Azure App Service][AppServicePricing].  Les options de mise à l'échelle des applications web et des plans App Service d'un environnement App Service sont alors les suivantes :

* Sélection du pool de travaux
* Nombre d'instances

La modification de l'un ou l'autre élément s'effectue via l'interface utilisateur appropriée pour vos plans App Service hébergés par ASE.  

![Capture d’écran montrant où afficher les détails du plan de service Scale et du plan de service Worker Pool.][1]

Vous ne pouvez pas mettre à l'échelle votre ASP au-delà du nombre de ressources de calcul disponibles du pool de travaux où se trouve votre ASP.  Si vous avez besoin de ressources de calcul dans ce pool de travaux, vous devez demander à l'administrateur de votre environnement App Service de les ajouter.  Pour plus d’informations sur la reconfiguration de votre environnement App Service Environment, consultez [Comment configurer un environnement App Service][HowtoConfigureASE].  Vous pouvez également tirer parti des fonctionnalités de mise à l'échelle automatique d'ASE pour ajouter de la capacité en fonction de la planification ou des mesures.  Pour obtenir des informations sur la configuration de mise à l’échelle automatique de l’environnement ASE lui-même, consultez [Configuration de la mise à l’échelle automatique pour un environnement App Service][ASEAutoscale].

Vous pouvez créer plusieurs plans App Service à l’aide de ressources de calcul provenant de pools de travaux différents ou vous pouvez utiliser le même pool de travail.  Par exemple, si vous disposez de (10) ressources de calcul disponibles dans le pool de travaux 1, vous pouvez choisir de créer un plan App Service à l’aide de (6) ressources de calcul et planifier un deuxième service d’application qui utilise (4) ressources de calcul.

### <a name="scaling-the-number-of-instances"></a>Mise à l'échelle du nombre d'instances
Lorsque vous créez votre application web dans un environnement App Service, elle ne contient qu’1 instance.  Vous pouvez ensuite effectuer un scale-out pour fournir des ressources de calcul supplémentaires à votre application.   

Si votre environnement Application Service a une capacité suffisante, l'opération est assez simple.  Vous accédez à votre plan App Service qui contient les sites que vous souhaitez mettre à l'échelle et vous sélectionnez Mettre à l'échelle.  Ceci affiche l'interface utilisateur où vous pouvez définir manuellement l'échelle de votre ASP ou configurer des règles de mise à l'échelle automatique pour votre ASP.  Pour mettre à l'échelle manuellement votre application, il vous suffit de définir ***Mise à l'échelle selon*** sur ***un nombre d'instances saisi manuellement***.  À partir de là, faites glisser le curseur vers la quantité souhaitée ou entrez la quantité souhaitée dans la zone à côté du curseur.  

![Capture d’écran montrant où vous pouvez définir l'échelle de votre ASP ou configurer des règles de mise à l'échelle automatique pour votre ASP.][2] 

Les règles de mise à l'échelle automatique d'un ASP dans un travail ASE fonctionnent de manière normale.  Vous pouvez sélectionner ***Pourcentage UC*** sous ***Mise à l'échelle selon*** et créer des règles de mise à l'échelle automatique de votre ASP basées sur le pourcentage UC ou vous pouvez créer des règles plus complexes à l'aide des ***règles de performances et de planification***.  Pour plus d’informations sur la configuration de la mise à l’échelle automatique, utilisez le guide [Mise à l’échelle d’une application dans Azure App Service][AppScale]. 

### <a name="worker-pool-selection"></a>Sélection du pool de travaux
Comme indiqué précédemment, la sélection du pool de travaux s'effectue à partir de l'interface utilisateur du plan App Service (ASP).  Ouvrez le panneau de l'ASP que vous souhaitez mettre à l'échelle et sélectionnez le pool de travaux.  Tous les pools de travaux que vous avez configurés dans votre environnement App Service s'affichent.  Si vous n'avez qu'un pool de travaux, lui seul apparaît.  Pour modifier le pool de travaux dans lequel se trouve votre ASP, il suffit de sélectionner le pool de travaux vers lequel vous souhaitez déplacer votre plan App Service.  

![Capture d’écran montrant où vous pouvez modifier le worker pool dans lequel se trouve votre ASP.][3]

Avant de déplacer votre ASP à partir d'un pool de travaux vers un autre, il est important de vous assurer que la capacité est adaptée pour votre ASP.  Dans la liste des pools de travaux, non seulement figure le nom du pool de travaux, mais également le nombre de travaux disponibles dans ce pool.  Veillez à ce qu'il y ait suffisamment d'instances disponibles pour contenir votre plan App Service.  Si vous avez besoin de plus de ressources de calcul dans le pool de travaux vers lequel vous souhaitez déplacer votre plan, demandez à l'administrateur de votre environnement App Service de les ajouter.  

> [!NOTE]
> Le déplacement d’un ASP d’un pool de travaux entraîne le redémarrage à froid de vos applications dans cet ASP.  Cela peut provoquer le ralentissement de l’exécution des demandes, car votre application démarre à froid sur les nouvelles ressources de calcul.  Le démarrage à froid peut être évité en utilisant la [fonctionnalité de préchauffage des applications][AppWarmup] dans Azure App Service.  Le module Initialisation d’application décrit dans l’article fonctionne également pour les démarrages à froid, car le processus d'initialisation est également appelé lorsque les applications sont démarrées à froid sur les nouvelles ressources de calcul. 
> 
> 

## <a name="getting-started"></a>Prise en main
Pour prendre en main les environnements App Service, consultez [Comment créer un environnement App Service][HowtoCreateASE]

<!--Image references-->
[1]: ./media/app-service-web-scale-a-web-app-in-an-app-service-environment/aseappscale-aspblade.png
[2]: ./media/app-service-web-scale-a-web-app-in-an-app-service-environment/aseappscale-manualscale.png
[3]: ./media/app-service-web-scale-a-web-app-in-an-app-service-environment/aseappscale-sizescale.png

<!--Links-->
[WhatisASE]: app-service-app-service-environment-intro.md
[ScaleWebapp]: ../manage-scale-up.md
[HowtoCreateASE]: app-service-web-how-to-create-an-app-service-environment.md
[HowtoConfigureASE]: app-service-web-configure-an-app-service-environment.md
[CreateWebappinASE]: app-service-web-how-to-create-a-web-app-in-an-ase.md
[Appserviceplans]: ../overview-hosting-plans.md
[AppServicePricing]: https://azure.microsoft.com/pricing/details/app-service/ 
[ASEAutoscale]: app-service-environment-auto-scale.md
[AppScale]: ../manage-scale-up.md
[AppWarmup]: https://ruslany.net/2015/09/how-to-warm-up-azure-web-app-during-deployment-slots-swap/
