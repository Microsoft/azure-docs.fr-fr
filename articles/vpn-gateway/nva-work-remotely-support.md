---
title: 'Travail à distance : Considérations relatives aux appliances virtuelles réseau (NVA) pour le travail à distance | Passerelle VPN Azure'
description: Cet article vous aide à comprendre les points à prendre en compte lorsque vous utilisez des appliances virtuelles réseau dans Azure pendant la pandémie de COVID-19.
services: vpn-gateway
author: scottnap
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: scottnap
ms.openlocfilehash: 70b5732e1293e35127a19fbe736d8562056a870b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499677"
---
# <a name="working-remotely-network-virtual-appliance-nva-considerations-for-remote-work"></a>Travail à distance : Considérations relatives aux appliances virtuelles réseau (NVA) pour le travail à distance

>[!NOTE]
>Cet article décrit comment vous pouvez tirer parti des appliances virtuelles réseau, d’Azure, des réseaux Microsoft et de l’écosystème de partenaires Azure pour travailler à distance et atténuer les problèmes de réseau auxquels vous faites face dans le contexte de l’épidémie de COVID-19.
>

Certains clients Azure utilisent des appliances virtuelles réseau tierces de la place de marché Azure pour fournir des services critiques comme le VPN de point à site pour leurs employés travaillant de chez eux pendant l’épidémie de COVID-19. Cet article décrit quelques conseils de haut niveau à prendre en considération lors de l’utilisation d’appliances virtuelles réseau dans Azure pour fournir des solutions d’accès à distance.

## <a name="nva-performance-considerations"></a>Considérations relatives aux performances des appliances virtuelles réseau

Tous les principaux fournisseurs d’appliances virtuelles réseausur la place de marché Azure doivent fournir des suggestions relatives à la taille des machines virtuelles et au nombre d’instances à utiliser lors du déploiement de la solution.  Bien que presque tous les fournisseurs d’appliances virtuelles réseau vous permettent de choisir la taille à votre disposition dans une région donnée, il est très important de suivre les suggestions données par les fournisseurs pour les tailles d’instance des machines virtuelles Azure, car ces recommandations sont les tailles de machine virtuelle dont le fournisseur a testé les performances dans Azure.  

### <a name="consider-the-following"></a>Tenez compte des éléments suivants

- **Capacité et nombre d’utilisateurs simultanés** : ce nombre est particulièrement important pour les utilisateurs VPN de point à site, car chaque utilisateur connecté crée un tunnel chiffré (VPN IPSec ou SSL).  
- **Débit agrégé** : quelle est la bande passante agrégée dont vous avez besoin pour prendre en charge tous les utilisateurs auxquels vous devez fournir l’accès à distance.
- **Taille de machine virtuelle nécessaire** : vous devez toujours utiliser les tailles de machines virtuelles recommandées par le fournisseur d’appliances virtuelles réseau.  Dans le cas d’un VPN de point à site, si vous avez des connexions utilisateur simultanées, vous devez utiliser des machines virtuelles de plus grande taille, par exemple des machines virtuelles [dv2 et DSv2](../virtual-machines/dv2-dsv2-series.md "Série dv2 et Dsv2"). Ces machines virtuelles ont en général plus de processeurs virtuels et peuvent gérer plus de sessions VPN simultanées.  En plus d’avoir davantage de cœurs virtuels, les machines virtuelles de grande taille dans Azure disposent de plus de capacité de bande passante que les machines virtuelles de petite taille.
    > **Important :** Chaque fournisseur utilise les ressources de façon différente.  S’il n’est pas évident de connaître les tailles d’instance que vous devez utiliser pour accueillir la charge utilisateur estimée, contactez le fournisseur du logiciel directement et demandez-lui conseil.
- **Nombre d’instances** : si vous prévoyez un grand nombre d’utilisateurs et de connexions, certaines limites s’appliquent à la possibilité d’augmenter la taille des instances d’appliances virtuelles réseau.  Envisagez de déployer plusieurs instances de machine virtuelle.
- **VPN IPSec et VPN SSL** : en général, les implémentations VPN IPSec fonctionnent mieux que les implémentations VPN SSL.  
- **Licences** : assurez-vous que les licences logicielles que vous avez achetées pour la solution d’appliance virtuelle réseau couvrent la croissance à laquelle vous pouvez vous trouver confronté pendant l’épidémie de COVID-19.  De nombreux programmes de licence d’appliance virtuelle réseau limitent le nombre de connexions ou la bande passante de la solution.
- **Mise en réseau accélérée** : envisagez une solution d’appliance virtuelle réseau qui prend en charge la mise en réseau accélérée.  Une mise en réseau accélérée permet d’opérer une virtualisation d’E/S d’une racine unique (SR-IOV) sur une machine virtuelle, ce qui améliore considérablement les performances de mise en réseau. Cette voie hautement performante court-circuite l’hôte à partir du chemin d’accès aux données, réduisant ainsi la latence, l’instabilité et l’utilisation du processeur pour servir les charges de travail réseau les plus exigeantes sur les types de machines virtuelles pris en charge. La mise en réseau accélérée est prise en charge dans la plupart des instances d’usage général et optimisées pour le calcul (deux processeurs virtuels ou plus).

## <a name="monitoring-resources"></a>Surveillance des ressources

Chaque solution d’appliance virtuelle réseau possède ses propres outils et ressources pour surveiller les performances de ses appliances virtuelles réseau.  Consultez la documentation de votre fournisseur pour vous assurer que vous comprenez les limitations des performances et que vous pouvez détecter si votre appliance virtuelle réseau est proche de sa capacité maximale.  En outre, vous pouvez consulter Azure Monitor Network Insights et voir des informations de performances de base sur vos appliances virtuelles réseau, par exemple :

- Utilisation du processeur
- Network In
- Network Out
- Flux entrants
- Flux sortants

## <a name="next-steps"></a>Étapes suivantes

La plupart de grands services d’appliances virtuelles réseau ont publié des conseils sur la préparation en vue d’une croissance soudaine et inattendue pendant l’épidémie de COVID-19. Voici quelques liens utiles vers les ressources des partenaires.

[Barracuda facilite le travail à domicile tout en sécurisant vos données pendant l’épidémie de COVID-19](https://www.barracuda.com/covid-19/work-from-home "Faciliter le travail à domicile tout en sécurisant vos données pendant l’épidémie de COVID-19")

[Check Point - Personnel en distanciel sécurisé pendant le coronavirus](https://www.checkpoint.com/solutions/secure-remote-workforce-during-coronavirus/ "Personnel en distanciel sécurisé pendant le coronavirus")

[Implémentation de Cisco AnyConnect et informations de référence sur les performances et la mise à l’échelle pour la préparation pendant le COVID-19](https://www.cisco.com/c/en/us/support/docs/security/anyconnect-secure-mobility-client/215331-anyconnect-implementation-and-performanc.html "Implémentation de Cisco AnyConnect et informations de référence sur les performances et la mise à l’échelle pour la préparation pendant le COVID-19")

[Centre de support Citrix COVID-19](https://www.citrix.com/support/covid-19-coronavirus.html "Centre de support Citrix COVID-19")

[Conseils F5 pour faire face à l’augmentation spectaculaire du nombre travailleurs à distance](https://www.f5.com/business-continuity "Conseils F5 pour faire face à l’augmentation spectaculaire du nombre travailleurs à distance")

[Informations COVID-19 de Fortinet pour les clients et partenaires](https://www.fortinet.com/covid-19.html "Informations COVID-19 de Fortinet pour les clients et partenaires")

[Centre de réponse COVID-19 de Palo Alto Networks](https://live.paloaltonetworks.com/t5/COVID-19-Response-Center/ct-p/COVID-19_Response_Center "Centre de réponse COVID-19 de Palo Alto Networks")

[Kemp - Activer le travail à distance et expérience d’application toujours accessible pour la continuité de l’activité](https://kemptechnologies.com/remote-work-always-on-application-experience-business-continuity/ "Kemp - Activer le travail à distance et expérience d’application toujours accessible pour la continuité de l’activité")

