---
title: Considérations sur les performances d’Azure Traffic Manager | Microsoft Docs
description: Comprendre les performances sur Traffic Manager et comment tester les performances de votre site Web lors de l’utilisation de Traffic Manager
services: traffic-manager
documentationcenter: ''
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/16/2017
ms.author: duau
ms.openlocfilehash: f080f5011311101ed8f9faa2ee807c030a048409
ms.sourcegitcommit: b4f303f59bb04e3bae0739761a0eb7e974745bb7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91653041"
---
# <a name="performance-considerations-for-traffic-manager"></a>Considérations sur les performances de Traffic Manager

Cette page explique les considérations relatives aux performances de Traffic Manager. Examinez le cas suivant :

Vous avez des instances de votre site web dans les régions WestUS et EastAsia. Une des instances ne réussit pas la vérification d’intégrité du sondage Traffic Manager. Le trafic de l’application est dirigé vers la région saine. Ce basculement est attendu, mais les performances peuvent constituer un problème en fonction de la latence du trafic qui transite maintenant vers une région distante.

## <a name="performance-considerations-for-traffic-manager"></a>Considérations sur les performances de Traffic Manager

Le seul impact sur les performances que Traffic Manager peut avoir sur votre site web est la recherche DNS initiale. Une requête DNS pour le nom de votre profil Traffic Manager est gérée par le serveur racine Microsoft DNS qui héberge la zone trafficmanager.net. Traffic Manager renseigne et met régulièrement à jour les serveurs racine Microsoft DNS en fonction de la stratégie Traffic Manager et des résultats du sondage. Donc, même pendant la recherche DNS initiale, aucune requête DNS n’est envoyée à Traffic Manager.

Traffic Manager est constitué de plusieurs composants : des serveurs de noms DNS, un service d’API, la couche de stockage et le service de surveillance des points de terminaison. Si un composant du service Traffic Manager échoue, cela n’a aucune incidence sur le nom DNS associé à votre profil Traffic Manager. Les enregistrements dans les serveurs DNS Microsoft restent inchangés. Toutefois, la surveillance des points de terminaison et la mise à jour DNS n’ont pas lieu. Par conséquent, Traffic Manager n’est pas en mesure de mettre à jour le DNS pour qu’il pointe vers votre site de basculement lorsque votre site principal tombe en panne.

La résolution de noms DNS est rapide et les résultats sont mis en cache. La vitesse de la recherche DNS initiale dépend des serveurs DNS que le client utilise pour la résolution de noms. En règle générale, un client peut effectuer une recherche DNS en environ 50 ms. Les résultats de la recherche sont mis en cache pour la durée de vie (TTL) du DNS. La durée de vie par défaut de Traffic Manager est de 300 secondes.

Le trafic N’est PAS transmis via Traffic Manager. Au terme de la recherche DNS, le client a une adresse IP pour une instance de votre site web. Le client se connecte directement à cette adresse et ne passe pas par le biais de Traffic Manager. La stratégie Traffic Manager que vous choisissez n’a aucune influence sur les performances du DNS. Toutefois, une méthode de routage des performances peut affecter négativement l’expérience de l’application. Par exemple, si votre stratégie redirige le trafic d’Amérique du Nord vers une instance hébergée en Asie, la latence du réseau pour ces sessions peut constituer un problème de performances.

## <a name="measuring-traffic-manager-performance"></a>Mesure des performances de Traffic Manager

Il existe plusieurs sites web que vous pouvez utiliser pour comprendre les performances et le comportement d’un profil Traffic Manager. La plupart de ces sites sont gratuits mais peuvent comporter des limites. Certains sites proposent des fonctionnalités payantes de surveillance et de génération de rapports améliorées.

Les outils sur ces sites mesurent les latences DNS et affichent les adresses IP résolues pour les emplacements clients du monde entier. La plupart de ces outils ne mettent pas en cache les résultats DNS. Par conséquent, les outils affichent la recherche DNS complète chaque fois qu’un test est exécuté. Lorsque vous testez votre propre client, vous obtenez uniquement des performances de recherche DNS complètes une seule fois pendant la durée de vie.

## <a name="sample-tools-to-measure-dns-performance"></a>Exemples d’outils pour mesurer les performances DNS

* [SolveDNS](https://www.solvedns.com/dns-comparison/)

    SolveDNS offre de nombreux outils de performances. L’outil de comparaison de DNS peut vous indiquer la durée nécessaire pour résoudre le nom DNS en effectuant une comparaison avec d’autres fournisseurs de service DNS.

* [WebSitePulse](https://www.websitepulse.com/help/tools.php)

    L’un des outils les plus simples est WebSitePulse. Entrez l’URL pour voir le temps de résolution DNS, le premier octet, le dernier octet et d’autres statistiques de performances. Vous pouvez choisir entre trois emplacements de test différents. Dans cet exemple, vous voyez que la première exécution montre que la recherche DNS dure 0,204 s.

    ![Capture d’écran montrant l’outil « WebSitePulse » avec le résultat de la recherche « DNS » mis en surbrillance.](./media/traffic-manager-performance-considerations/traffic-manager-web-site-pulse.png)

    Comme les résultats sont mis en cache, la deuxième fois que nous exécutons ce test sur le même point de terminaison Traffic Manager, la recherche DNS prend 0,002 s.

    ![pulse2](./media/traffic-manager-performance-considerations/traffic-manager-web-site-pulse2.png)

* [CA App Synthetic Monitor](https://asm.ca.com/en/checkit.php)

    Auparavant appelé « outil Watch-mouse Check Website », ce site vous montre le temps de résolution DNS simultanément à partir de plusieurs régions géographiques. Entrez l’URL pour voir le temps de résolution DNS, l’heure de la connexion et la vitesse à partir de plusieurs emplacements géographiques. Ce test permet de voir quel service hébergé est retourné pour différents emplacements dans le monde entier.

    ![pulse1](./media/traffic-manager-performance-considerations/traffic-manager-web-site-watchmouse.png)

* [Pingdom](https://tools.pingdom.com/)

    Cet outil fournit des statistiques de performances pour chaque élément d’une page web. L’onglet Analyse de la page affiche le pourcentage de temps consacré à la recherche DNS.

* [Qu’est-ce que Mon DNS ?](https://www.whatsmydns.net/)

    Ce site effectue une recherche DNS à partir de 20 emplacements différents et affiche les résultats sur une carte.

* [Dig Web Interface](https://www.digwebinterface.com)

    Ce site affiche plus d’informations DNS détaillées, notamment les enregistrements CNAME et A. Veillez à cocher « Coloriser la sortie » et « Statistiques » sous les options et sélectionnez « Tous » sous Noms de serveurs.

## <a name="next-steps"></a>Étapes suivantes

[À propos des méthodes de routage du trafic de Traffic Manager](traffic-manager-routing-methods.md)

[Tester vos paramètres de Traffic Manager](traffic-manager-testing-settings.md)

[Opérations sur Traffic Manager (Référence sur l’API REST)](https://go.microsoft.com/fwlink/?LinkId=313584)

[Applets de commande Azure Traffic Manager](https://docs.microsoft.com/powershell/module/az.trafficmanager)

