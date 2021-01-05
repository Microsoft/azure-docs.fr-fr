---
title: Profils Traffic Manager imbriqués dans Azure
titleSuffix: Azure Traffic Manager
description: Cet article explique la fonctionnalité des profils imbriqués d’Azure Traffic Manager
services: traffic-manager
documentationcenter: ''
author: duongau
manager: twooley
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/22/2018
ms.author: duau
ms.openlocfilehash: d96cd9b3c83357c2591edf4bbb09a82b897c6a24
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91826030"
---
# <a name="nested-traffic-manager-profiles"></a>Profils Traffic Manager imbriqués

Traffic Manager comprend une série de méthodes de routage du trafic qui vous permet de contrôler la manière dont Traffic Manager choisit le point de terminaison qui doit recevoir le trafic de chaque utilisateur final. Pour plus d’informations, consultez la rubrique relative aux [méthodes de routage du trafic dans Traffic Manager](traffic-manager-routing-methods.md).

Chaque profil Traffic Manager spécifie une seule méthode de routage du trafic. Cependant, certains scénarios exigent une méthode de routage du trafic plus sophistiquée que celle proposée par un profil Traffic Manager unique. Vous pouvez imbriquer des profils Traffic Manager pour combiner les avantages de plusieurs méthodes de routage du trafic. Les profils imbriqués permettent de modifier le comportement par défaut de Traffic Manager pour prendre en charge des déploiements d’applications plus importants et plus complexes.

Les exemples suivants illustrent l’utilisation de profils Traffic Manager imbriqués dans divers scénarios.

## <a name="example-1-combining-performance-and-weighted-traffic-routing"></a>Exemple 1: combinaison de routage du trafic « performant (Performance) » et « pondéré (Weighted) »

Supposons que vous avez déployé une application dans les régions Azure suivantes : USA Ouest, Europe Ouest et Asie Est. Vous utilisez la méthode de routage du trafic « Performance » de Traffic Manager pour acheminer le trafic vers la région la plus proche de l’utilisateur.

![Profil Traffic Manager unique][4]

Maintenant, supposons que vous souhaitez tester une mise à jour de votre service avant de déployer celui-ci plus largement. Vous souhaitez utiliser la méthode de routage du trafic « Pondéré » pour acheminer un petit pourcentage du trafic vers votre déploiement de tests. Vous configurez le déploiement de tests en même temps que le déploiement de production existant dans la région Europe Ouest.

Vous ne pouvez pas combiner les routages de trafic « Pondéré » et « Performance » dans un seul profil. Pour prendre en charge ce scénario, vous créez un profil Traffic Manager en utilisant les deux points de terminaison d’Europe Ouest et la méthode de routage du trafic « Pondéré ». Ensuite, vous ajoutez ce profil « enfant » en tant que point de terminaison au profil « parent ». Le profil parent utilise encore la méthode de routage du trafic « Performances », et contient les autres déploiements globaux en tant que points de terminaison.

Le schéma suivant illustre cet exemple :

![Profils Traffic Manager imbriqués][2]

Dans cette configuration, le trafic dirigé via le profil parent distribue le trafic entre les régions normalement. Dans la région Europe Ouest, le profil imbriqué distribue le trafic vers les points de terminaison de production et de test en fonction des pondérations assignées.

Lorsque le profil parent utilise la méthode de routage du trafic « Performance », un emplacement doit être assigné à chaque point de terminaison. L’emplacement est assigné lorsque vous configurez le point de terminaison. Choisissez la région Azure la plus proche de votre déploiement. Les régions Azure sont les valeurs d’emplacement prises en charge par la Table de latence Internet. Pour plus d’informations, voir [Méthode de routage du trafic « Performance » d’Azure Traffic Manager](traffic-manager-routing-methods.md#performance).

## <a name="example-2-endpoint-monitoring-in-nested-profiles"></a>Exemple 2 : analyse de points de terminaison dans des profils imbriqués

Traffic Manager surveille activement l'intégrité de chaque point de terminaison de service. Si un point de terminaison n’est pas intègre, Traffic Manager dirige les utilisateurs vers d’autres points de terminaison pour préserver la disponibilité de votre service. Ce comportement de surveillance et de basculement des points de terminaison s’applique à toutes les méthodes de routage du trafic. (pour plus d’informations, voir la rubrique relative à la [surveillance des points de terminaison avec Traffic Manager](traffic-manager-monitoring.md)) ; La surveillance des points de terminaison fonctionne différemment pour les profils imbriqués. Avec des profils imbriqués, le profil parent n’effectue pas de contrôles d’intégrité directement sur le profil enfant. Au lieu de cela, l’intégrité des points de terminaison du profil enfant est utilisée pour calculer l’intégrité globale du profil enfant. Ces informations d’intégrité sont propagées vers le haut de la hiérarchie de profils imbriqués. Le profil parent utilise cette intégrité agrégée pour déterminer s’il faut diriger le trafic vers le profil enfant. Pour plus d’informations sur la surveillance de l’intégrité des profils imbriqués, consultez le [FAQ](traffic-manager-FAQs.md#traffic-manager-nested-profiles).

En revenant à l’exemple précédent, supposons que le déploiement de production dans la région Europe Ouest est défaillant. Par défaut, le profil « enfant » dirige tout le trafic vers le déploiement de tests. Si le déploiement de tests échoue également, le profil parent détermine que le profil enfant ne doit pas recevoir de trafic, car aucun des points de terminaison enfants n’est intègre. Ensuite, le profil parent distribue le trafic aux autres régions.

![Basculement de profil imbriqué (comportement par défaut)][3]

Il se peut que vous soyez satisfait de cette organisation. Il se peut également que vous vous inquiétiez du fait que tout le trafic pour la région Europe Ouest aille désormais vers le déploiement de tests, au lieu du trafic d’un sous-ensemble limité. Quelle que soit l’intégrité du déploiement du test, vous souhaitez basculer vers les autres régions en cas d’échec du déploiement de production dans la région Europe Ouest. Pour activer ce basculement, vous pouvez spécifier le paramètre « MinChildEndpoints » lorsque vous configurez le profil enfant en tant que point de terminaison dans le profil parent. Le paramètre détermine le nombre minimal de points de terminaison disponibles dans le profil de l’enfant. La valeur par défaut est « 1 ». Pour ce scénario, vous définissez la valeur de MinChildEndpoints sur 2. Sous ce seuil, le profil parent considère que le profil enfant entier est indisponible et dirige le trafic vers les autres points de terminaison.

Le schéma suivant illustre cette configuration :

![Basculement de profil imbriqué avec « MinChildEndpoints » = 2][4]

> [!NOTE]
> La méthode de routage du trafic « Priorité » distribue tout le trafic vers un seul point de terminaison. Il est par conséquent peu utile de définir un paramètre MinChildEndpoints autre que « 1 » pour un profil enfant.

## <a name="example-3-prioritized-failover-regions-in-performance-traffic-routing"></a>Exemple 3 : basculement hiérarchisé des régions dans le routage du trafic de type « Performance »

Le comportement par défaut de la méthode de routage « Performance » est que, lorsque vos points de terminaison se trouvent sur des emplacements géographiques différents, les utilisateurs finaux sont renvoyés au point de terminaison « le plus proche » en termes de latence réseau la plus faible.

Toutefois, supposons que vous préfériez basculer le trafic vers Europe Ouest plutôt que vers USA Ouest, et diriger le trafic vers d’autres régions uniquement lorsque les deux points de terminaison sont indisponibles. Vous pouvez créer cette solution en utilisant un profil enfant avec la méthode de routage du trafic « Priorité ».

![Routage du trafic « Performance » avec basculement préférentiel][6]

Étant donné que le point de terminaison Europe Ouest a une priorité plus élevée que le point de terminaison de la région USA Ouest, tout le trafic est envoyé au point de terminaison Europe Ouest lorsque les deux points de terminaison sont en ligne. En cas de défaillance du point de terminaison Europe Ouest, le trafic est dirigé vers USA Ouest. Avec le profil imbriqué, le trafic est dirigé vers Asie Est uniquement en cas de défaillances des points de terminaison Europe Ouest et USA Ouest.

Vous pouvez répéter ce modèle pour toutes les régions. Remplacez les trois points de terminaison dans le profil parent par trois profils enfants, chacun offrant une séquence de basculement hiérarchisée.

## <a name="example-4-controlling-performance-traffic-routing-between-multiple-endpoints-in-the-same-region"></a>Exemple 4 : contrôle du routage du trafic de type « Performance » entre plusieurs points de terminaison dans la même région

Supposons que la méthode de routage du trafic « Performance » est utilisée dans un profil qui a plusieurs points de terminaison dans une région spécifique. Par défaut, le trafic dirigé vers cette région est réparti uniformément entre tous les points de terminaison disponibles dans cette région.

![Distribution du trafic au sein d’une région dans le cadre d’un routage du trafic « Performance » (comportement par défaut)][7]

Au lieu que plusieurs points de terminaison soient ajoutés dans Europe Ouest, ces points de terminaison sont regroupés dans un profil enfant distinct. Le profil enfant est ajouté au parent en tant que seul point de terminaison dans Europe Ouest. Les paramètres du profil enfant peuvent contrôler la distribution du trafic vers la région Europe Ouest en permettant un routage du trafic basé sur la priorité ou pondéré au sein de cette région.

![Routage du trafic « Performance » avec distribution personnalisée du trafic au sein de la région][8]

## <a name="example-5-per-endpoint-monitoring-settings"></a>Exemple 5 : paramètres de surveillance par point de terminaison

Supposons que vous utilisez Traffic Manager pour migrer en douceur le trafic d’un site web local hérité vers une nouvelle version cloud hébergée dans Azure. Pour le site hérité, vous souhaitez utiliser l’URI de la page d’accueil pour surveiller l’intégrité du site. Mais pour la nouvelle version cloud, vous implémentez une page de surveillance personnalisée (chemin d’accès « /monitor.aspx ») qui inclut des vérifications supplémentaires.

![Surveillance des points de terminaison Traffic Manager (comportement par défaut)][9]

Les paramètres d’analyse dans un profil Traffic Manager s’appliquent à tous les points de terminaison au sein d’un même profil. Avec des profils imbriqués, vous utilisez un profil enfant distinct par site pour définir des paramètres de surveillance différents.

![Surveillance des points de terminaison Traffic Manager avec paramétrage par point de terminaison][10]

## <a name="faqs"></a>FAQ

* [Comment configurer des profils imbriqués ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#traffic-manager-nested-profiles)

* [Combien de couches d’imbrication Traffic Manager prend-il en charge ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-many-layers-of-nesting-does-traffic-manger-support)

* [Puis-je combiner d’autres types de point de terminaison avec des profils enfants imbriqués dans le même profil Traffic Manager ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-mix-other-endpoint-types-with-nested-child-profiles-in-the-same-traffic-manager-profile)

* [Comment le modèle de facturation s’applique-t-il aux profils imbriqués ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-does-the-billing-model-apply-for-nested-profiles)

* [Y a-t-il un impact sur les performances pour les profils imbriqués ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#is-there-a-performance-impact-for-nested-profiles)

* [Comment Traffic Manager calcule-t-il l’intégrité d’un point de terminaison imbriqué dans un profil parent ?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-does-traffic-manager-compute-the-health-of-a-nested-endpoint-in-a-parent-profile)

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [profils Traffic Manager](traffic-manager-overview.md)

En savoir plus sur la [création d’un profil Traffic Manager](traffic-manager-create-profile.md)

<!--Image references-->
[1]: ./media/traffic-manager-nested-profiles/figure-1.png
[2]: ./media/traffic-manager-nested-profiles/figure-2.png
[3]: ./media/traffic-manager-nested-profiles/figure-3.png
[4]: ./media/traffic-manager-nested-profiles/figure-4.png
[5]: ./media/traffic-manager-nested-profiles/figure-5.png
[6]: ./media/traffic-manager-nested-profiles/figure-6.png
[7]: ./media/traffic-manager-nested-profiles/figure-7.png
[8]: ./media/traffic-manager-nested-profiles/figure-8.png
[9]: ./media/traffic-manager-nested-profiles/figure-9.png
[10]: ./media/traffic-manager-nested-profiles/figure-10.png
