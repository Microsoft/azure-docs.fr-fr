---
title: Mettre à l’échelle une application dans Azure Spring Cloud | Microsoft Docs
description: Apprenez à mettre à l’échelle une application avec Azure Spring Cloud dans le portail Azure.
ms.service: spring-cloud
ms.topic: how-to
ms.author: brendm
author: bmitchell287
ms.date: 10/06/2019
ms.custom: devx-track-java
ms.openlocfilehash: 5632f9a6126615255306cc89425bd08a9ffa9753
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96531800"
---
# <a name="scale-an-application-in-azure-spring-cloud"></a>Mettre à l’échelle une application dans Azure Spring Cloud

**Cet article s’applique à :** ✔️ Java ✔️ C#

Cette documentation montre comment mettre à l’échelle une application de microservices à partir du tableau de bord Azure Spring Cloud sur le portail Azure.

Effectuez un scale-up ou un scale-down de votre application en modifiant le nombre de processeurs virtuels, ainsi que la quantité de mémoire. Effectuez un scale-in ou un scale-out de votre application en modifiant le nombre d’instances de l’application.

À l’issue de ce tutoriel, vous saurez comment effectuer des modifications manuelles rapides pour chaque application de votre service. La mise à l’échelle prend effet en quelques secondes et ne nécessite aucune modification de code ni de redéploiement.

## <a name="prerequisites"></a>Prérequis

Pour suivre ces procédures, vous avez besoin des éléments suivants :

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer. 
* Une instance du service Azure Spring Cloud déployée.  Pour bien démarrer, suivez notre [guide de démarrage rapide sur le déploiement d’une application par le biais d’Azure CLI](spring-cloud-quickstart.md).
* Au moins une application déjà créée dans votre instance de ce service.

## <a name="navigate-to-the-scale-page-in-the-azure-portal"></a>Accéder à la page Mettre à l’échelle sur le portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Accédez à la page **Vue d’ensemble** de votre instance Azure Spring Cloud.

1. Sélectionnez le groupe de ressources qui contient votre service.

1. Sélectionnez l’onglet **Applications** sous l’en-tête **Paramètres** dans le menu de gauche de la page.

1. Sélectionnez l’application à mettre à l’échelle. Dans cet exemple, sélectionnez l’application nommée **account-service**. La page **Vue d’ensemble** de l’application doit alors s’afficher.

1. Accédez à l’onglet **Mettre à l’échelle** situé sous **Paramètres** dans le menu de gauche de la page. Vous devez voir s’afficher les options des attributs de mise à l’échelle dans la section suivante.

## <a name="scale-your-application"></a>Mettre à l’échelle votre application

Si vous modifiez les attributs de mise à l’échelle, tenez compte des remarques suivantes :

* **Processeurs** : le nombre maximal de processeurs est de quatre par instance d’application. Le nombre total de processeurs pour une application correspond à la valeur définie ici, multipliée par le nombre d’instances d’application.

* **Mémoire/Go** : la quantité maximale de mémoire est de 8 Go par instance d’application. La quantité totale de mémoire pour une application correspond à la valeur définie ici, multipliée par le nombre d’instances d’application.

* **Nombre d’instances d’application** : Avec le niveau Standard, vous pouvez effectuer un scale-out allant jusqu’à 20 instances. Cette valeur change le nombre d’instances distinctes de l’application de microservices en cours d’exécution.

Veillez à sélectionner **Enregistrer** pour appliquer vos paramètres de mise à l’échelle.

![Service de mise à l’échelle dans le portail Azure](media/spring-cloud-tutorial-scale-manual/scale-up-out.png)

Au bout de quelques secondes, les modifications que vous avez apportées sont visibles dans la page **Vue d’ensemble**. Des détails supplémentaires sont disponibles sous l’onglet **Instances de l’application**. La mise à l’échelle ne nécessite aucune modification de code ni de redéploiement.

## <a name="upgrade-to-the-standard-tier"></a>Passer au niveau Standard
Si vous êtes au niveau De base et limité par une ou plusieurs de ces [restrictions](spring-cloud-quotas.md), vous pouvez passer au niveau Standard. Pour ce faire, accédez au menu Niveau tarifaire en sélectionnant d’abord la colonne Niveau Standard et en cliquant sur le bouton **Mettre à niveau**.

## <a name="next-steps"></a>Étapes suivantes

Cet exemple vous a montré comment mettre à l’échelle manuellement une application Azure Spring Cloud. Pour savoir comment superviser une application en configurant des alertes, consultez [Configurer la mise à l’échelle automatique](spring-cloud-tutorial-setup-autoscale.md).

> [!div class="nextstepaction"]
> [Découvrir comment configurer des alertes](spring-cloud-tutorial-alerts-action-groups.md)
