---
title: Résoudre les problèmes de déploiement de service cloud | Microsoft Docs
description: Vous pouvez rencontrer quelques problèmes courants lors du déploiement d’un service cloud sur Azure. Cet article fournit des solutions pour certains d’entre eux.
services: cloud-services
documentationcenter: ''
author: simonxjx
manager: dcscontentpm
editor: ''
tags: top-support-issue
ms.assetid: a18ae415-0d1c-4bc4-ab6c-c1ddea02c870
ms.service: cloud-services
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 06/15/2018
ms.author: v-six
ms.openlocfilehash: 0e7cd496f031f76320df5127d7e1aa3f2f7b06c7
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92075074"
---
# <a name="troubleshoot-cloud-service-deployment-problems"></a>Résoudre les problèmes de déploiement de service cloud
Lorsque vous déployez un package d’application de service cloud sur Azure, vous pouvez obtenir des informations concernant le déploiement dans le volet **Propriétés** du portail Azure. Vous pouvez utiliser les détails figurant dans ce volet pour faciliter la résolution des problèmes du service cloud, et vous pouvez fournir ces informations au support Azure lors de l’ouverture d’une nouvelle demande de support.

Vous trouverez le volet **Propriétés** comme suit :

* Dans le portail Azure, cliquez successivement sur le déploiement de votre service cloud, sur **Tous les paramètres** et sur **Propriétés**.

> [!NOTE]
> Vous pouvez copier le contenu du volet **Propriétés** dans le Presse-papiers en cliquant sur l’icône dans le coin supérieur droit du volet.
>
>

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="problem-i-cannot-access-my-website-but-my-deployment-is-started-and-all-role-instances-are-ready"></a>Problème : Je ne peux pas accéder à mon site web bien que mon déploiement soit démarré et que toutes les instances de rôle soient prêtes
Le lien URL du site web affiché dans le portail n’inclut pas le port. Le port par défaut pour les sites web est 80. Si votre application est configurée pour s’exécuter dans un autre port, vous devez ajouter le numéro de port approprié à l’URL lorsque vous accédez au site web.

1. Dans le portail Azure, cliquez sur le déploiement de votre service cloud.
2. Dans le volet **Propriétés** du portail Azure, vérifiez les ports des instances de rôle (sous **Points de terminaison d’entrée**).
3. Si le port n’est pas le port 80, ajoutez la valeur de port correcte à l’URL lorsque vous accédez à l’application. Pour spécifier un port non défini par défaut, tapez l’URL, suivie de deux-points (:) et du numéro de port sans espaces.

## <a name="problem-my-role-instances-recycled-without-me-doing-anything"></a>Problème : Mes instances de rôle sont recyclées sans action de ma part
Une réparation de service se produit automatiquement lorsque Azure détecte des nœuds problématiques et déplace donc les instances de rôle vers de nouveaux nœuds. Le cas échéant, il est possible que vos instances de rôle se recyclent automatiquement. Pour savoir si une réparation de service a eu lieu :

1. Dans le portail Azure, cliquez sur le déploiement de votre service cloud.
2. Dans le volet **Propriétés** du portail Azure, passez en revue les informations et déterminez si une réparation de service a eu lieu au moment où vous observiez le recyclage des rôles.

Les rôles sont recyclés également environ une fois par mois pendant les mises à jour du système d’exploitation hôte et du système d’exploitation invité.  
Pour plus d’informations, consultez le billet de blog [Redémarrages d’instances de rôles pour cause de mises à niveau du système d’exploitation](/archive/blogs/kwill/role-instance-restarts-due-to-os-upgrades)

## <a name="problem-i-cannot-do-a-vip-swap-and-receive-an-error"></a>Problème : Impossible d’effectuer un échange d’adresses IP virtuelles, une erreur s’affiche
Un échange d’adresses IP virtuelles n’est pas autorisé si une mise à jour de déploiement est en cours. Les mises à jour de déploiement peuvent se produire automatiquement dans les situations suivantes :

* Un nouveau système d’exploitation invité est disponible et votre installation est configurée pour les mises à jour automatiques.
* Une réparation de service se produit.

Pour savoir si une mise à jour automatique vous empêche d’effectuer un échange d’adresses IP virtuelles :

1. Dans le portail Azure, cliquez sur le déploiement de votre service cloud.
2. Dans le volet **Propriétés** du portail Azure, examinez la valeur de l’**État**. Si elle affiche **Prêt**, vérifiez la **Dernière opération** pour voir si une opération récente empêche l’échange d’adresses IP virtuelles.
3. Répétez les étapes 1 et 2 pour le déploiement en production.
4. Si une mise à jour automatique est en cours, attendez qu’elle se termine avant d’essayer d’effectuer l’échange d’adresses IP virtuelles.

## <a name="problem-a-role-instance-is-looping-between-started-initializing-busy-and-stopped"></a>Problème : Une instance de rôle est exécutée en boucle entre Démarrée, Initialisation, Occupée et Arrêtée
Cette condition peut indiquer un problème lié à votre code d’application, package ou fichier de configuration. Dans ce cas, vous devez être en mesure de voir l’état changer à des intervalles de quelques minutes, et le portail Azure peut indiquer des états tels que **Recyclage**, **Occupé** ou **Initialisation**. Cela indique qu’il existe un problème au niveau de l’application qui empêche l’exécution de l’instance de rôle.

Pour plus d’informations sur la résolution de ce problème, consultez les billets de blog [Données de diagnostic de calcul PaaS Azure](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data) et [Problèmes courants provoquant le recyclage des rôles](cloud-services-troubleshoot-common-issues-which-cause-roles-recycle.md).

## <a name="problem-my-application-stopped-working"></a>Problème : Mon application a cessé de fonctionner
1. Dans le portail Azure, cliquez sur l’instance de rôle.
2. Dans le volet **Propriétés** du portail Azure, tenez compte des conditions suivantes pour résoudre votre problème :
   * Si l’instance de rôle a été récemment arrêtée (vous pouvez vérifier la valeur **Nombre d’abandons**), il est possible que le déploiement soit en cours de mise à jour. Attendez de voir si l’instance de rôle recommence à fonctionner par elle-même.
   * Si l’instance de rôle est **Occupée**, vérifiez votre code d’application pour voir si l’événement [StatusCheck](/previous-versions/azure/reference/ee758135(v=azure.100)) est géré. Vous devrez peut-être ajouter ou corriger le code qui gère cet événement.
   * Passez en revue les données de diagnostic et les scénarios de résolution des problèmes dans le billet de blog [Données de diagnostic de calcul PaaS Azure](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data).

> [!WARNING]
> Si vous recyclez votre service cloud, vous réinitialisez les propriétés de déploiement, et effacez ainsi efficacement les informations du problème d’origine.
>
>

## <a name="next-steps"></a>Étapes suivantes
Affichez plus d’ [articles de résolution des problèmes](./cloud-services-allocation-failures.md) liés aux services cloud.

Pour découvrir comment résoudre les problèmes liés aux rôles de service cloud à l’aide des données de diagnostic de calcul PaaS Azure, consultez la [série de blogs de Kevin Williamson](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data).