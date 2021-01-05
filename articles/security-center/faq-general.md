---
title: FAQ sur Azure Security Center - Questions générales
description: Questions fréquentes (FAQ) générales sur Azure Security Center, produit qui vous aide à prévenir, détecter et contrer les menaces
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/25/2020
ms.author: memildin
ms.openlocfilehash: 061b190af6e66cf0c7e8095251a3ef77d9aaf247
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92341683"
---
# <a name="faq---general-questions-about-azure-security-center"></a>FAQ - Questions générales sur Azure Security Center

## <a name="what-is-azure-security-center"></a>Qu’est-ce que le Centre de sécurité Azure ?
Azure Security Center vous aide à prévenir, détecter et contrer les menaces grâce à une visibilité et un contrôle accrus de la sécurité de vos ressources. Il fournit une surveillance de la sécurité et une gestion des stratégies intégrées pour l’ensemble de vos abonnements, vous aidant ainsi à détecter les menaces qui pourraient passer inaperçues. De plus, il est compatible avec un vaste écosystème de solutions de sécurité.

Security Center utilise l’agent Log Analytics pour collecter et stocker les données. Pour plus d’informations, consultez [Collecte de données dans Azure Security Center](security-center-enable-data-collection.md).


## <a name="how-do-i-get-azure-security-center"></a>Comment obtenir le Centre de sécurité Azure ?
Azure Security Center est disponible avec votre abonnement Microsoft Azure et accessible à partir du [Portail Azure](https://azure.microsoft.com/features/azure-portal/). Pour y accéder, [connectez-vous au portail](https://portal.azure.com), sélectionnez **Parcourir**, puis faites défiler la page jusqu’à **Security Center**.


## <a name="which-azure-resources-are-monitored-by-azure-security-center"></a>Quelles ressources Azure sont surveillées par Azure Security Center ?
Azure Security Center surveille les ressources Azure suivantes :

* Machines virtuelles (VM) (y compris [Cloud Services](../cloud-services/cloud-services-choose-me.md))
* Groupes identiques de machines virtuelles
* Solutions de partenaires intégrées à votre abonnement Azure, par exemple pare-feu d’applications web sur les machines virtuelles et sur App Service Environment
* [Nombreux services Azure PaaS listés dans la vue d’ensemble du produit](features-paas.md)


## <a name="how-can-i-see-the-current-security-state-of-my-azure-resources"></a>Comment connaître l’état de sécurité actuel de mes ressources Azure ?
La page **Vue d’ensemble de Security Center** affiche les mesures de sécurité globales de votre environnement pour les ressources suivantes : Compute, Mise en réseau, Stockage et données, et Applications. Chaque type de ressource possède un indicateur montrant les failles de sécurité identifiées. Cliquez sur chaque vignette pour afficher une liste des problèmes de sécurité identifiés par Security Center, ainsi qu’un inventaire des ressources dans votre abonnement.



## <a name="what-is-a-security-policy"></a>Qu’est-ce qu’une stratégie de sécurité ?
Une stratégie de sécurité définit l’ensemble des contrôles recommandés pour les ressources d’un abonnement spécifique. Dans Azure Security Center, vous définissez les stratégies de vos abonnements Azure en fonction des exigences de sécurité de votre entreprise et du type d’applications ou du niveau de confidentialité des données de chaque abonnement.

Les stratégies de sécurité activées dans Azure Security Center déterminent les recommandations et la surveillance liées à la sécurité. Pour plus d’informations sur les stratégies de sécurité, consultez [Surveillance de l’intégrité de la sécurité dans le Centre de sécurité Azure](security-center-monitoring.md).


## <a name="who-can-modify-a-security-policy"></a>Qui peut modifier une stratégie de sécurité ?
Pour modifier une stratégie de sécurité, vous devez avoir le rôle d’**administrateur de la sécurité** ou de **propriétaire** pour l’abonnement concerné.

Pour savoir comment configurer une stratégie de sécurité, consultez [Définition de stratégies de sécurité dans le Centre de sécurité Azure](tutorial-security-policy.md).


## <a name="what-is-a-security-recommendation"></a>Qu’est-ce qu’une recommandation de sécurité ?
Le Centre de sécurité Azure analyse l’état de sécurité de vos ressources Azure. Quand des failles de sécurité potentielles sont identifiées, des recommandations sont créées. Les recommandations vous guident tout au long du processus de configuration du contrôle. Voici quelques exemples :

* Approvisionnement d’un logiciel anti-programme malveillant pour identifier et supprimer les programmes malveillants
* [Groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md) et règles pour contrôler le trafic vers les machines virtuelles
* Approvisionnement d’un pare-feu d’applications web pour se défendre des attaques ciblant vos applications web
* Déploiement de mises à jour système manquantes
* Correction des configurations de système d’exploitation qui ne suivent pas les recommandations

Seules les recommandations qui sont activées dans les stratégies de sécurité sont affichées ici.


## <a name="what-triggers-a-security-alert"></a>Qu’est-ce qui déclenche une alerte de sécurité ?
Azure Security Center collecte, analyse et fusionne automatiquement les données du journal à partir de vos ressources Azure, du réseau et des solutions partenaires telles que les logiciels anti-programme malveillant et les pare-feu. Quand des menaces sont détectées, une alerte de sécurité est créée. Voici quelques exemples de détections :

* Des machines virtuelles compromises qui communiquent avec des adresses IP connues comme étant malveillantes
* Des programmes malveillants avancés qui sont détectés à l’aide du rapport d’erreurs Windows
* Des attaques par force brute contre des machines virtuelles
* Des alertes de sécurité émises par des solutions de sécurité partenaires intégrées, telles que des logiciels anti-programme malveillant ou des pare-feu d’applications web


## <a name="whats-the-difference-between-threats-detected-and-alerted-on-by-microsoft-security-response-center-versus-azure-security-center"></a>Quelle est la différence entre les menaces détectées et faisant l’objet d’une alerte par Microsoft Security Response Center et par Azure Security Center ?
Microsoft Security Response Center (MSRC) effectue certaines analyses de sécurité sur l'infrastructure et le réseau Azure et reçoit des informations sur les menaces et des plaintes pour mauvaise utilisation provenant de tiers. Lorsque MSRC constate que les données client ont été utilisées par un tiers illégal ou non autorisé ou que l'utilisation d’Azure par le client ne respecte pas les conditions de bon usage, un gestionnaire des incidents de sécurité informe le client. La notification correspond généralement à un courrier électronique envoyé aux contacts de sécurité spécifiés dans Azure Security Center ou au propriétaire de l’abonnement Azure si aucun contact de sécurité n’est spécifié.

Security Center est un service Azure qui surveille en continu l'environnement Azure du client et applique des analyses de façon à détecter automatiquement un large éventail d'activités potentiellement malveillantes. Ces détections sont signalées en tant qu'alertes de sécurité dans le tableau de bord du Security Center.