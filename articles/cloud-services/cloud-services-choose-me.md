---
title: Présentation d’Azure Cloud Services | Microsoft Docs
description: Découvrez Azure Cloud Services, offre conçue pour prendre en charge des applications évolutives, fiables et dont l’exploitation est peu coûteuse.
services: cloud-services
author: tgore03
ms.service: multiple
ms.topic: article
ms.date: 04/19/2017
ms.author: tagore
ms.openlocfilehash: 0013a3a29bae9d2dde7896b3ae23d0d358946f2b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88224286"
---
# <a name="overview-of-azure-cloud-services"></a>Vue d’ensemble d’Azure Cloud Services
Azure Cloud Service est un exemple d’implémentation de [platform as a service](https://azure.microsoft.com/overview/what-is-paas/) (PaaS). Tout comme [Azure App Service](../app-service/overview.md), cette technologie est conçue pour prendre en charge des applications évolutives, fiables et dont l’exploitation est peu coûteuse. Comme App Service, Azure Cloud Services est hébergé sur des machines virtuelles. Toutefois, vous avez davantage de contrôle sur les machines virtuelles. Vous pouvez installer votre propre logiciel sur des machines virtuelles utilisant Azure Cloud Services, et y accéder à distance.

![Diagramme Azure Cloud Services](./media/cloud-services-choose-me/diagram.png)

Un contrôle supérieur signifie également une convivialité réduite. Si vous n’avez pas besoin d’options de contrôle supplémentaires, il est généralement plus rapide et plus facile de faire fonctionner une application web dans la fonctionnalité Web Apps d’App Service que dans Azure Cloud Services.

Il existe deux types de rôles Azure Cloud Services. La seule différence entre eux est le mode d’hébergement de votre rôle sur les machines virtuelles :

* **Rôle web** : Déploie et héberge automatiquement votre application, via IIS.

* **Rôle de travail** : Exécute votre application de façon autonome, sans utiliser IIS.

Par exemple, une application simple peut utiliser un rôle web unique, servant un site web. Une application plus complexe pourrait utiliser un rôle web pour traiter les requêtes entrantes des utilisateurs, puis transmettre ces requêtes à un rôle de travail à des fins de traitement (cette communication pourrait utiliser [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) ou [Stockage File d’attente Azure](../storage/common/storage-introduction.md)).

Ainsi que le suggère la figure ci-dessus, toutes les machines virtuelles d’une même application s’exécutent dans le même service cloud. Les utilisateurs accèdent à l’application par le biais d’une seule adresse IP publique, avec un équilibrage automatique de la charge des requêtes entre les machines virtuelles de cette application. La plateforme [met à l’échelle et déploie](cloud-services-how-to-scale-portal.md) les machines virtuelles dans une application Cloud Services de façon à éviter un point unique de défaillance matérielle.

Bien que les applications s’exécutent sur des machines virtuelles, il est important de comprendre qu’Azure Cloud Services utilise une technologie PaaS et non IaaS. Voici ce que cela signifie. Avec la technologie IaaS, utilisée par exemple sur les machines virtuelles Azure, vous commencez par créer et configurer l’environnement dans lequel votre application s’exécute. Vous déployez ensuite votre application dans cet environnement. Il vous appartient de gérer une grande partie de ce monde, par exemple, en déployant de nouvelles versions corrigées du système d’exploitation sur chaque machine virtuelle. Avec PaaS, en revanche, c'est comme si l'environnement existait déjà. Vous n'avez plus qu'à déployer votre application. La gestion de la plateforme sur laquelle elle s'exécute, y compris le déploiement de nouvelles versions du système d'exploitation, est assurée à votre place.

## <a name="scaling-and-management"></a>Mise à l'échelle et gestion
Avec Azure Cloud Services, vous ne créez pas de machines virtuelles. Au contraire, vous fournissez un fichier de configuration qui indique à Azure le nombre d’instances de chaque type voulu, par exemple, trois instances de rôle web et deux instances de rôle de travail. La plateforme les crée ensuite pour vous. Vous continuez de choisir [la taille](cloud-services-sizes-specs.md) que doivent avoir ces machines virtuelles de stockage, mais vous ne les créez pas vous-même de manière explicite. Si votre application a besoin de traiter une charge plus importante, vous pouvez demander davantage de machines virtuelles ; Azure créera ces instances. Si la charge diminue, vous pouvez arrêter ces instances et cesser de payer pour elles.

Une application Azure Cloud Services est normalement mise à la disposition des utilisateurs au moyen d’un processus en deux étapes. Un développeur [télécharge d'abord l'application](cloud-services-how-to-create-deploy-portal.md) dans la zone de transit de la plateforme. Lorsqu’il est prêt à activer l’application, il utilise le portail de gestion Azure pour la faire passer de l’étape intermédiaire à l’environnement de production. Le [basculement entre la zone de transit et l'environnement de production](cloud-services-how-to-manage-portal.md#swap-deployments-to-promote-a-staged-deployment-to-production) peut se faire sans temps d'arrêt, ce qui permet de mettre à niveau une application en cours d'exécution vers une nouvelle version sans perturbation pour les utilisateurs.

## <a name="monitoring"></a>Surveillance
Azure Cloud Services assure également la surveillance. À l’instar de Machines virtuelles Azure, il détecte l’échec d’un serveur physique et redémarre sur un nouvel ordinateur les machines virtuelles qui étaient exécutées sur ce serveur. Toutefois, au-delà des défaillances matérielles, Azure Cloud Services détecte aussi les échecs des machines virtuelles et des applications. À la différence des machines virtuelles, ils ont un agent à l'intérieur de chaque rôle Web et de travail, et sont donc en mesure de démarrer de nouvelles instances des machines virtuelles et des applications lorsqu'une défaillance se produit.

La nature PaaS d’Azure Cloud Services a également d’autres implications. L'une des principales est le fait que les applications basées sur cette technologie doivent être écrites pour s'exécuter correctement en cas d'échec d'un rôle Web ou de travail. Cela nécessite qu’une application Azure Cloud Services ne conserve pas l’état de ses propres machines virtuelles dans le système de fichiers. Contrairement aux machines virtuelles créées avec Machines virtuelles Azure, les écritures effectuées sur les machines virtuelles Azure Cloud Services ne sont pas persistantes. Il n’est de tel qu’un disque de données Machines virtuelles Azure. En revanche, une application Azure Cloud Services doit écrire explicitement tous les états dans Azure SQL Database, les objets blob, les tables ou un autre stockage externe. Les applications créées de la sorte sont plus faciles à mettre à l’échelle et résistent davantage aux défaillances, deux objectifs importants d’Azure Cloud Services.

## <a name="next-steps"></a>Étapes suivantes
* [Créer une application de service cloud dans .NET](cloud-services-dotnet-get-started.md) 
* [Créer une application de service cloud dans Node.js](cloud-services-nodejs-develop-deploy-app.md) 
* [Créer une application de service cloud dans PHP](../cloud-services-php-create-web-role.md) 
* [Création d’une application de service cloud dans Python](cloud-services-python-ptvs.md)






