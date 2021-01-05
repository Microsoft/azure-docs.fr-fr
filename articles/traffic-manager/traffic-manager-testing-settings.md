---
title: Vérifier les paramètres d’Azure Traffic Manager
description: Dans cet article, découvrez comment vérifier vos paramètres Traffic Manager et tester la méthode de routage du trafic.
services: traffic-manager
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/16/2017
ms.author: duau
ms.openlocfilehash: 4bb7458d4258dffb074d663e5cf712c7a0245cc7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022324"
---
# <a name="verify-traffic-manager-settings"></a>Vérifier les paramètres Traffic Manager

Pour tester vos paramètres Traffic Manager, vous devez disposer de plusieurs clients, à différents emplacements, d’où vous pouvez exécuter vos tests. Ensuite, mettez les points de terminaison de votre profil Traffic Manager hors service, l’un après l’autre.

* Définissez une durée de vie (TTL) DNS assez courte pour que les modifications se propagent rapidement (par exemple, 30 secondes).
* Veillez à connaître les adresses IP de vos services cloud et sites web Azure du profil que vous testez.
* Utilisez des outils permettant de résoudre un nom DNS en adresse IP et d’afficher cette adresse.

Vous devez vérifier que les noms DNS sont résolus en adresses IP des points de terminaison de votre profil. Les noms doivent être résolus de manière cohérente par rapport à la méthode de routage du trafic définie dans le profil Traffic Manager. Vous pouvez utiliser les outils **nslookup** ou **dig** pour résoudre les noms DNS.

Les exemples suivants vous aideront à tester votre profil Traffic Manager.

### <a name="check-traffic-manager-profile-using-nslookup-and-ipconfig-in-windows"></a>Vérifier le profil Traffic Manager à l’aide de nslookup et ipconfig dans Windows

1. Ouvrez une commande ou une invite de commandes Windows PowerShell en tant qu’administrateur.
2. Entrez `ipconfig /flushdns` pour vider le cache de résolution DNS.
3. Tapez `nslookup <your Traffic Manager domain name>`. Par exemple, la commande suivante vérifie le nom de domaine doté du préfixe *myapp.contoso*

    ```powershell
    nslookup myapp.contoso.trafficmanager.net
    ```

    Les résultats contiennent généralement les informations suivantes :

    + le nom DNS et l’adresse IP du serveur DNS utilisé pour résoudre le nom de domaine Traffic Manager ;
    + le nom de domaine Traffic Manager saisi dans la ligne de commande après « nslookup » et l’adresse IP correspondant au domaine Traffic Manager. La deuxième adresse IP est celle à vérifier. Elle doit correspondre à l’adresse IP virtuelle (VIP) publique de l’un des services cloud ou sites web du profil Traffic Manager testé.

## <a name="how-to-test-the-failover-traffic-routing-method"></a>Comment tester la méthode de routage du trafic par basculement

1. Laissez tous les points de terminaison en fonction.
2. À partir d’un même client, demandez la résolution DNS du nom de domaine de votre entreprise à l’aide de l’outil nslookup.exe ou d’un utilitaire similaire.
3. Vérifiez que l’adresse IP résolue correspond au point de terminaison principal.
4. Mettez votre point de terminaison principal hors service ou supprimez le fichier de surveillance pour que Traffic Manager considère que l’application est hors service.
5. Patientez pendant une période correspondant à la durée de vie (TTL, Time-to-Live) DNS du profil Traffic Manager en rajoutant deux minutes. Par exemple, si la TTL de votre DNS est de 300 secondes (5 minutes), vous devez attendre 7 minutes.
6. Videz le cache de votre client DNS et demandez une résolution DNS à l’aide de nslookup. Dans Windows, vous pouvez vider votre cache DNS à l’aide de la commande ipconfig /flushdns.
7. Vérifiez que l’adresse IP résolue correspond à votre point de terminaison secondaire.
8. Répétez le processus en mettant tour à tour chaque point de terminaison hors service. Vérifiez que le système DNS retourne l’adresse IP du point de terminaison suivant dans la liste. Une fois tous les points de terminaison hors fonction, vous devez obtenir à nouveau l’adresse IP du point de terminaison principal.

## <a name="how-to-test-the-weighted-traffic-routing-method"></a>Comment tester la méthode de routage du trafic par pondération

1. Laissez tous les points de terminaison en fonction.
2. À partir d’un même client, demandez la résolution DNS du nom de domaine de votre entreprise à l’aide de l’outil nslookup.exe ou d’un utilitaire similaire.
3. Vérifiez que l’adresse IP résolue correspond à l’un de vos points de terminaison.
4. Videz le cache de votre client DNS et répétez les étapes 2 et 3 pour chaque point de terminaison. Vous devriez alors obtenir différentes adresses IP pour chacun de vos points de terminaison.

## <a name="how-to-test-the-performance-traffic-routing-method"></a>Comment tester la méthode de routage du trafic basé sur les performances

Pour tester efficacement une méthode de routage du trafic basé sur les performances, vous devez disposer de plusieurs clients répartis en différents emplacements dans le monde. Vous pouvez créer des clients dans différentes régions Azure, qui peuvent servir à tester vos services. Si votre réseau est d’envergure mondiale, vous pouvez vous connecter à des clients situés dans d’autres parties du monde et exécuter vos tests à partir de ces emplacements.

Sinon, différents services de recherche et d’obtention de DNS sont disponibles gratuitement sur Internet. Certains de ces outils permettent de contrôler la résolution de noms DNS à partir de divers emplacements à travers le monde. Faites une recherche sur « Recherche DNS » pour obtenir des exemples. Certains services tiers comme Gomez ou Keynote vous permettent de vérifier que vos profils distribuent le trafic comme prévu.

## <a name="next-steps"></a>Étapes suivantes

* [À propos des méthodes de routage du trafic de Traffic Manager](traffic-manager-routing-methods.md)
* [Considérations sur les performances de Traffic Manager](traffic-manager-performance-considerations.md)
* [Résolution des problèmes liés à l’état Détérioré de Traffic Manager](traffic-manager-troubleshooting-degraded.md)
