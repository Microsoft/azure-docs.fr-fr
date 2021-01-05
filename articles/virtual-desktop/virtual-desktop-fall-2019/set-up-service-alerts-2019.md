---
title: Configurer des alertes de service pour Windows Virtual Desktop (classique) – Azure
description: Comment configurer Azure Service Health pour recevoir des notifications de service pour Windows Virtual Desktop (classique).
author: Heidilohr
ms.topic: tutorial
ms.date: 05/27/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 01f4f52acaa5b461e2b058f112f26dafda86dcb7
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025025"
---
# <a name="tutorial-set-up-service-alerts-for-windows-virtual-desktop-classic"></a>Tutoriel : Configurer des alertes de service pour Windows Virtual Desktop (classique)

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager. Si vous essayez de gérer des objets Windows Virtual Desktop Azure Resource Manager, consultez [cet article](../set-up-service-alerts.md).

Vous pouvez utiliser Azure Service Health pour surveiller les problèmes de service et les avis d’intégrité pour Windows Virtual Desktop. Azure Service Health peut vous envoyer des notifications avec différents types d’alertes (par exemple, e-mail ou SMS), vous aider à comprendre l’effet d’un problème et vous tenir informé de son évolution. Azure Service Health peut également vous aider à réduire les temps d’arrêt, vous préparer à la maintenance planifiée et aux changements susceptibles d’affecter la disponibilité de vos ressources.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créez et configurez des alertes de service.

Pour en savoir plus sur Azure Service Health, consultez la [Documentation Azure Health](../../service-health/index.yml).

## <a name="prerequisites"></a>Prérequis

- [Tutoriel : Créer un locataire dans Windows Virtual Desktop](tenant-setup-azure-active-directory.md)
- [Tutoriel : Créer des principaux de service et des attributions de rôles avec PowerShell](create-service-principal-role-powershell.md)
- [Tutoriel : Créer un pool d’hôtes avec la Place de marché Azure](create-host-pools-azure-marketplace-2019.md)

## <a name="create-service-alerts"></a>Créer des alertes de service

Cette section vous montre comment configurer Azure Service Health et définir des notifications, auxquelles vous pouvez accéder sur le portail Azure. Vous pouvez définir différents types d’alertes et les planifier pour vous avertir en temps voulu.

### <a name="recommended-service-alerts"></a>Alertes de service recommandées

Nous vous recommandons de créer des alertes de service pour les types d’événements d’intégrité suivants :

- **Problème de service :** Recevez des notifications sur les problèmes majeurs qui affectent la connectivité de vos utilisateurs avec le service ou la possibilité de gérer votre client Windows Virtual Desktop.
- **Avis d’intégrité :** Recevez des notifications qui nécessitent votre attention. Voici quelques exemples de ce type de notification :
    - Des machines virtuelles (VM) n’étant pas configurées de façon sûre avec le port 3389 ouvert
    - Dépréciation de fonctionnalité

### <a name="configure-service-alerts"></a>Configurer les alertes de service

Pour configurer des alertes de service :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Sélectionnez **Service Health.**
3. Suivez les instructions de [Créer des alertes de journal d’activité sur les notifications de service](../../service-health/alerts-activity-log-service-notifications-portal.md?toc=%2fazure%2fservice-health%2ftoc.json) pour configurer vos alertes et notifications.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à configurer et utiliser Azure Service Health pour surveiller les problèmes de service et les avis d’intégrité pour Windows Virtual Desktop. Pour en savoir plus sur la façon de se connecter à Windows Virtual Desktop, passez aux guides pratiques concernant la connexion à Windows Virtual Desktop.

> [!div class="nextstepaction"]
> [Se connecter au client Bureau à distance sur Windows 7 et Windows 10](connect-windows-7-10-2019.md)