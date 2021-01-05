---
title: Mesures utilisateur réelles dans Azure Traffic Manager
description: Dans cette introduction, découvrez comment fonctionnent les Mesures utilisateur réelles Azure Traffic Manager.
services: traffic-manager
documentationcenter: traffic-manager
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/16/2018
ms.author: duau
ms.custom: ''
ms.openlocfilehash: 45c889ef8d67570fb075ec710616f20d110b7f50
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89401603"
---
# <a name="traffic-manager-real-user-measurements-overview"></a>Vue d’ensemble de la fonctionnalité Mesures des utilisateurs réels dans Traffic Manager

Quand vous configurez un profil Traffic Manager pour utiliser la méthode de routage des performances, le service recherche la provenance des demandes de requête DNS et prend des décisions de routage pour diriger ces demandeurs vers la région Azure qui leur offre la latence la plus faible. Pour ce faire, il utilise l’intelligence de latence réseau que Traffic Manager gère pour différents réseaux d’utilisateurs finaux.

La fonctionnalité Mesures des utilisateurs réels vous permet d’évaluer les mesures de la latence réseau vers les régions Azure, à partir des applications clientes auxquelles recourent vos utilisateurs finaux, et d’intégrer ces informations aux prises de décision de routage par Traffic Manager. En choisissant d’utiliser la fonctionnalité Mesures des utilisateurs réels, vous pouvez augmenter la précision du routage pour les demandes provenant des réseaux où résident vos utilisateurs finaux. 

## <a name="how-real-user-measurements-work"></a>Fonctionnement de la fonctionnalité Mesures des utilisateurs réels

Grâce à la fonctionnalité Mesures utilisateur réelles, vos applications clientes mesurent la latence vers les régions Azure du point de vue des réseaux d’utilisateurs finaux où elles sont utilisées. Par exemple, si vous avez une page web accessible aux utilisateurs en différents endroits (par exemple, dans les régions d’Amérique du Nord), vous pouvez utiliser la fonctionnalité Mesures utilisateur réelles avec la méthode de routage des performances pour orienter ces utilisateurs vers la meilleure région Azure hébergeant votre application serveur.

Dans un premier temps, un code JavaScript fourni par Azure (et contenant une clé unique) est incorporé dans vos pages web. Ensuite, chaque fois qu’un utilisateur visite cette page web, le code JavaScript interroge Traffic Manager pour obtenir des informations sur les régions Azure qu’il doit mesurer. Le service retourne un ensemble de points de terminaison au script, qui mesure ensuite ces régions consécutivement en téléchargeant une image d’un pixel hébergée dans ces régions Azure et en notant la latence qui s’est écoulée entre l’envoi de la demande et la réception du premier octet. Ces mesures sont ensuite renvoyées au service Traffic Manager.

Au fil du temps, l’exécution répétée de cette opération sur plusieurs réseaux permet à Traffic Manager d’obtenir des informations plus précises sur les caractéristiques de la latence des réseaux dans lesquels se trouvent vos utilisateurs finaux. Ces informations sont progressivement incluses dans les décisions de routage prises par Traffic Manager. Il en résulte une précision accrue des décisions basées sur les mesures des utilisateurs réels envoyées.

Quand vous utilisez la fonctionnalité Mesures des utilisateurs réels, vous êtes facturé en fonction du nombre de mesures envoyées à Traffic Manager. Pour plus d’informations sur les prix, visitez la page [Tarifs Traffic Manager](https://azure.microsoft.com/pricing/details/traffic-manager/).

## <a name="faqs"></a>FAQ

* [Quels sont les avantages de l’utilisation des mesures des utilisateurs réelles ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-are-the-benefits-of-using-real-user-measurements)

* [Puis-je utiliser les mesures des utilisateurs réelles avec des non-régions Azure ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-use-real-user-measurements-with-non-azure-regions)

* [Quelle méthode de routage bénéficie de la fonctionnalité Mesures utilisateur réelles ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#which-routing-method-benefits-from-real-user-measurements)

* [Dois-je activer la fonctionnalité Mesures utilisateur réelles séparément pour chaque profil ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#do-i-need-to-enable-real-user-measurements-each-profile-separately)

* [Comment faire pour désactiver la fonctionnalité Mesures utilisateur réelles pour mon abonnement ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-do-i-turn-off-real-user-measurements-for-my-subscription)

* [Puis-je utiliser la fonctionnalité Mesures utilisateur réelles avec des applications clientes autres que des pages web ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-use-real-user-measurements-with-client-applications-other-than-web-pages)

* [Combien de mesures sont effectuées à chaque affichage d’une page web sur laquelle la fonctionnalité Mesures utilisateur réelles est activée ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-many-measurements-are-made-each-time-my-real-user-measurements-enabled-web-page-is-rendered)

* [Y a-t-il un délai avant l’exécution du script Mesures utilisateur réelles dans ma page web ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#is-there-a-delay-before-real-user-measurements-script-runs-in-my-webpage)

* [Puis-je utiliser la fonctionnalité Mesures utilisateur réelles de manière à mesurer uniquement certaines régions Azure ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-use-real-user-measurements-with-only-the-azure-regions-i-want-to-measure)

* [Est-il possible de limiter le nombre de mesures effectuées à un nombre spécifique ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-limit-the-number-of-measurements-made-to-a-specific-number)

* [Puis-je afficher les mesures prises par mon application cliente dans le cadre de la fonctionnalité Mesures utilisateur réelles ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-see-the-measurements-taken-by-my-client-application-as-part-of-real-user-measurements)

* [Puis-je modifier le script de mesure fourni par Traffic Manager ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-modify-the-measurement-script-provided-by-traffic-manager)

* [Est-ce que d’autres utilisateurs peuvent voir la clé que j’utilise avec la fonctionnalité Mesures utilisateur réelles ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#will-it-be-possible-for-others-to-see-the-key-i-use-with-real-user-measurements)

* [Ma clé Mesures utilisateur réelles peut-elle faire l’objet d’une utilisation malveillante ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-others-abuse-my-rum-key)

* [Dois-je placer le script JavaScript de mesure dans toutes mes pages web ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#do-i-need-to-put-the-measurement-javascript-in-all-my-web-pages)

* [Traffic Manager peut-il identifier des informations sur mes utilisateurs finaux si j’utilise la fonctionnalité Mesures utilisateur réelles ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-information-about-my-end-users-be-identified-by-traffic-manager-if-i-use-real-user-measurements)

* [La page web sur laquelle la fonctionnalité Mesures utilisateur réelles est activée doit-elle utiliser Traffic Manager pour le routage ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#does-the-webpage-measuring-real-user-measurements-need-to-be-using-traffic-manager-for-routing)

* [Dois-je héberger un service sur les régions Azure à utiliser avec la fonctionnalité Mesures utilisateur réelles ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#do-i-need-to-host-any-service-on-azure-regions-to-use-with-real-user-measurements)

* [L’utilisation de la bande passante Azure augmente-t-elle quand j’utilise la fonctionnalité Mesures utilisateur réelles ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#will-my-azure-bandwidth-usage-increase-when-i-use-real-user-measurements)

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment utiliser la [fonctionnalité Mesures des utilisateurs réels avec des pages web](traffic-manager-create-rum-web-pages.md).
- En savoir plus sur le [fonctionnement de Traffic Manager](traffic-manager-overview.md)
- En savoir plus sur [Mobile Center](https://docs.microsoft.com/mobile-center/)
- En savoir plus sur les [méthodes de routage du trafic](traffic-manager-routing-methods.md) prises en charge par Traffic Manager
- En savoir plus sur la [création d’un profil Traffic Manager](traffic-manager-create-profile.md)

