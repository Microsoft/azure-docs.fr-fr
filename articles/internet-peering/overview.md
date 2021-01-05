---
title: Configurer le peering avec Microsoft
titleSuffix: Azure
description: Présentation du peering
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: overview
ms.date: 12/15/2020
ms.author: prmitiki
ms.openlocfilehash: 8f960e58d47963cbb239d498af52e7adfc77caa9
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97586864"
---
# <a name="internet-peering-overview"></a>Présentation du peering Internet

Le peering est l’interconnexion qui existe entre votre réseau et le réseau mondial de Microsoft (AS8075), à des fins d’échange de trafic Internet entre les services en ligne Microsoft et les services Microsoft Azure. Les opérateurs ou les fournisseurs de services peuvent demander à se connecter à Microsoft au niveau de n’importe quel réseau de périmètre. Chaque demande est examinée par Microsoft dans le but de vérifier qu’elle respecte notre stratégie de peering. Il existe deux façons de configurer un peering avec le réseau Microsoft :

* **Peering direct :**

    le peering est établi via des connexions physiques directes entre votre réseau et le réseau de périmètre Microsoft. Des sessions BGP sont configurées sur ces connexions conformément à notre stratégie de routage et à l’accord prénégocié. C’est ce qui est aussi appelé le « PNI ».

* **Peering Exchange :**

    cela correspond aux connexions de peering public standard au niveau des échanges Internet. Les connexions physiques entre le réseau Microsoft et votre réseau s’effectuent par le biais d’un « switched fabric » qui est géré par des échanges Internet. Les sessions BGP sont configurées à l’aide de l’espace IP fourni par les échanges Internet.

## <a name="benefits-of-peering-with-microsoft"></a>Avantages du peering avec Microsoft
* Réduisez vos coûts de transit en fournissant du trafic Microsoft à l’aide du peering avec Microsoft.
* Améliorez les performances de vos clients en réduisant les tronçons et la latence réseau vers le réseau de périmètre Microsoft.
* Protégez le trafic client contre les défaillances de votre réseau ou de celui de votre fournisseur de transit, en effectuant un peering Microsoft avec des sites redondants.
* Découvrez les métriques de performance relatives à vos connexions de peering et utilisez des insights pour dépanner votre réseau.

## <a name="benefits-of-using-azure-to-set-up-peering"></a>Avantages de l’utilisation d’Azure pour configurer le peering

Vous pouvez demander le peering avec Microsoft dans Azure PowerShell ou le portail. Une telle configuration du peering est gérée comme une ressource Azure et offre les avantages suivants :
* Simplification et automatisation de la configuration et de la gestion du peering avec Microsoft
* Moyen simple et rapide d’afficher et de gérer tous vos peerings au même endroit
* Suivi de l’état et des données de bande passante pour toutes vos connexions
* Possibilité d’utiliser le même abonnement pour accéder à Azure Cloud Services

Si vous avez déjà établi des peerings avec Microsoft, on parlera alors de **peerings hérités**. Vous pouvez choisir de gérer ces peerings comme une ressource Azure afin de tirer parti des avantages ci-dessus. Pour envoyer une nouvelle demande de peering ou convertir un peering hérité en ressource Azure, suivez les liens fournis dans la section **Étapes suivantes** ci-dessous.

## <a name="peering-policy"></a>Stratégie de peering
Microsoft dispose d’une stratégie de peering sélective, mais plutôt ouverte. Les pairs sont sélectionnés en fonction de leurs performances, de leurs fonctionnalités et d’un éventuel avantage mutuel. En outre, ils sont soumis à certaines exigences techniques, commerciales et juridiques. Pour plus d’informations, consultez [Stratégie de peering](policy.md).

## <a name="faq"></a>Questions fréquentes (FAQ)
Pour obtenir des réponses aux questions fréquemment posées sur le peering, consultez [Peering Internet - Questions fréquentes (FAQ)](faqs.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les étapes de configuration d’un peering direct avec Microsoft, suivez la [procédure pas à pas de configuration d’un peering direct](walkthrough-direct-all.md).
* Pour plus d’informations sur les étapes de configuration d’un peering Exchange avec Microsoft, suivez la [procédure pas à pas de configuration d’un peering Exchange](walkthrough-exchange-all.md).
* En savoir plus sur les autres [fonctionnalités de mise en réseau](../networking/networking-overview.md) clés d’Azure.