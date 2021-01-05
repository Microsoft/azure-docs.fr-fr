---
title: Architecture réseau de SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Architecture réseau du déploiement de SAP HANA sur Azure (grandes instances).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/15/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: b18e1cd20a4b0a886258fd56003cd273d92381fa
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97093976"
---
# <a name="sap-hana-large-instances-network-architecture"></a>Architecture réseau de SAP HANA (grandes instances)

L’architecture des services réseau Azure est un composant clé de la réussite du déploiement des applications SAP sur la grande instance HANA. En règle générale, les déploiements SAP HANA sur Azure (grandes instances) ont un plus grand paysage SAP avec plusieurs solutions SAP distinctes et tailles de bases de données variées, une consommation des ressources d’UC et une utilisation de la mémoire différentes. Il est probable que certains systèmes informatiques ne trouvent pas encore dans Azure. Votre paysage SAP est souvent également hybride sur le plan des SGBD et des applications SAP, utilisant une combinaison de NetWeaver, de S/4HANA et SAP HANA, ainsi que d’autres SGBD. Azure offre différents services qui vous permettent d’exécuter différents systèmes SGBD, NetWeaver et S/4HANA dans Azure. Azure offre également une technologie réseau qui lui permet de se présenter comme un centre de données virtuel pour vos déploiements de logiciels locaux.

À moins que l’ensemble de vos systèmes informatiques soient hébergés dans Azure. La fonctionnalité de mise en réseau Azure permet de connecter les ressources locales à vos actifs Azure de sorte qu’Azure ressemble à un centre de données virtuel. La fonctionnalité réseau Azure utilisée est la suivante : 

- Les réseaux virtuels Azure sont connectés au circuit [ExpressRoute](https://azure.microsoft.com/services/expressroute/) qui se connecte à vos ressources réseau locales.
- Un circuit ExpressRoute se connectant en local à Azure doit avoir une bande passante minimale de [1 Gbit/s](https://azure.microsoft.com/pricing/details/expressroute/). La bande passante minimale permet une bande passante adéquate pour le transfert de données entre les systèmes locaux et les systèmes qui s’exécutent sur des machines virtuelles. Il permet également à la bande passante adéquate de se connecter aux systèmes Azure à partir d’utilisateurs locaux.
- Tous les systèmes SAP dans Azure sont configurés dans des réseaux virtuels pour communiquer entre eux.
- Active Directory et le DNS hébergés localement sont étendus à Azure via ExpressRoute, ou s’exécutent entièrement dans Azure.

Dans le cas spécifique de l’intégration de Grandes instances HANA dans la structure réseau du centre de données Azure, la technologie Azure ExpressRoute est également utilisée.


> [!NOTE] 
> Un seul abonnement Azure ne peut être lié qu’à un seul locataire dans un tampon de Grande instance HANA situé dans une région Azure spécifique. À l’inverse, un seul locataire de tampon de Grande instance HANA ne peut être lié qu’à un seul abonnement Azure. Cette exigence est cohérente avec d’autres objets facturables dans Azure.

Si SAP HANA sur Azure (Grandes instances) est déployé dans plusieurs régions Azure différentes, un locataire séparé est déployé dans le tampon de Grande instance HANA. Vous pouvez exécuter ces deux abonnés dans le même abonnement Azure tant que ces instances font partie du même paysage SAP. 

> [!IMPORTANT] 
> Seule la méthode de déploiement Azure Resource Manager est prise en charge avec SAP HANA sur Azure (Grandes instances).

 

## <a name="additional-virtual-network-information"></a>Informations supplémentaires relatives au réseau virtuel

Pour connecter un réseau virtuel à ExpressRoute, une passerelle Azure ExpressRoute doit être créée. Pour plus d’informations, voir [À propos des passerelles ExpressRoute pour ExpressRoute](../../../expressroute/expressroute-about-virtual-network-gateways.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). 

Une passerelle Azure ExpressRoute est utilisée avec ExpressRoute sur une infrastructure située hors d’Azure ou sur un tampon de Grande instance Azure. Vous pouvez connecter la passerelle Azure ExpressRoute à un jusque quatre circuits ExpressRoute différents, tant que ceux-ci proviennent de routeurs de périphérie d’entreprise Microsoft distincts. Pour plus d’informations, consultez [Infrastructure et connectivité SAP HANA (grandes instances) sur Azure](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). 

> [!NOTE] 
> Le débit maximal pouvant être atteint avec une passerelle ExpressRoute est de 10 Gbits/s, à l’aide d’une connexion ExpressRoute. La copie de fichiers entre une machine virtuelle résidant dans un réseau virtuel et un système local (en tant que flux de copie unique) n’atteint pas le débit total des différentes références SKU des passerelles. Pour tirer parti de la bande passante complète de la passerelle ExpressRoute, utilisez plusieurs flux de données. Ou bien, vous devez copier des fichiers différents dans des flux parallèles d’un seul fichier.


## <a name="networking-architecture-for-hana-large-instance"></a>Architecture de mise en réseau pour la grande instance HANA
L’architecture de mise en réseau pour la grande instance HANA peut être divisée en quatre parties différentes :

- Mise en réseau locale et connexion ExpressRoute à Azure. Cette partie est le domaine des clients et est connectée à Azure via ExpressRoute. Ce circuit Expressroute est entièrement payé par vous en tant que client. La bande passante doit être suffisante pour gérer le trafic réseau entre vos ressources locales et la région Azure à laquelle vous vous connectez. Consultez la partie inférieure droite de la figure suivante.
- Services réseau Azure, comme indiqué précédemment, avec des réseaux virtuels qui ont besoin de l’ajout de passerelles ExpressRoute. Il s’agit d’une zone dans laquelle vous devez rechercher les conceptions appropriées pour les exigences, la sécurité et les exigences de conformité de votre application. L’utilisation de la grande instance HANA ou pas est à prendre en considération en ce qui concerne le nombre de réseaux virtuels et les références SKU de la passerelle Azure à partir desquels choisir. Consultez la partie supérieure droite de la figure.
- Connectivité de la grande instance HANA via la technologie ExpressRoute dans Azure. Cette partie est déployée et gérée par Microsoft. Il vous suffit de fournir certaines plages d’adresses IP après la connexion du déploiement de vos ressources dans la grande instance HANA au circuit ExpressRoute pour les réseaux virtuels. Pour plus d’informations, consultez [Infrastructure et connectivité SAP HANA (grandes instances) sur Azure](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). En tant que client, vous ne payez aucun frais supplémentaire pour la connectivité entre la structure réseau du centre de données Azure et les unités de Grande instance HANA.
- Mise en réseau dans le tampon de Grande instance HANA, essentiellement transparente pour vous.

![Réseau virtuel connecté à SAP HANA sur Azure (grandes instances) et en local](./media/hana-overview-architecture/image1-architecture.png)

L’obligation de connecter vos ressources locales via ExpressRoute à Azure perdure malgré l’utilisation de la grande instance HANA. La nécessité d’utiliser un ou plusieurs réseaux virtuels qui exécutent les machines virtuelles, hébergeant la couche Application qui se connecte aux instances HANA hébergées dans les unités de grande instance HANA ne change pas non plus. 

Les différences dans les déploiements SAP dans Azure sont les suivantes :

- Les unités de grande instance HANA de votre abonné client sont connectées via un autre circuit ExpressRoute à vos réseaux virtuels. Afin de séparer les conditions de charge, les circuits ExpressRoute entre les réseaux locaux et le réseau virtuel Azure, ainsi qu’entre les réseaux virtuels Azure et les Grandes instances HANA ne partagent pas les mêmes routeurs.
- Le profil de la charge de travail entre la couche Application SAP et la grande instance HANA est de nature différente, avec de nombreuses petites requêtes, et agit comme des transferts de données (jeux de résultats) à partir de SAP HANA dans la couche Application.
- L’architecture d’application SAP est plus sensible à la latence du réseau que les scénarios classiques dans lesquels les données sont échangées entre les machines locales et Azure.
- La passerelle Azure ExpressRoute possède au moins deux connexions ExpressRoute. Un circuit connecté à partir de l’emplacement local et un autre connecté à partir de Grandes instances HANA. Cela ne laisse de place que pour deux circuits supplémentaires à partir de routeurs de périphérie d’entreprise Microsoft (MSEE) distincts pour se connecter via une passerelle ExpressRoute. Cette restriction est indépendante de l’utilisation d’ExpressRoute Fast Path. Tous les circuits connectés partagent la bande passante maximale pour les données entrantes en provenance de la passerelle ExpressRoute.

Avec la Révision 3 des tampons Grande instance HANA, la latence du réseau rencontrée entre les machines virtuelles et les unités de grande instance HANA peut être supérieure à la latence standard aller-retour sur un réseau de machine virtuelle à machine virtuelle. En fonction de la région Azure, les valeurs mesurées peuvent dépasser une latence aller-retour de 0,7 ms, classée en dessous de la moyenne dans [Remarque SAP n° 1100926 - FAQ : Performances du réseau](https://launchpad.support.sap.com/#/notes/1100926/E). Selon la région Azure et l’outil utilisé pour mesurer la latence aller-retour entre une machine virtuelle Azure et une unité de grande instance SAP HANA, la latence mesurée peut avoisiner les 2 millisecondes maximum. Néanmoins, les clients déploient des applications SAP de production basées sur SAP HANA avec succès sur la grande instance SAP HANA. Vérifiez avoir testé minutieusement vos processus métier dans la Grande instance HANA Azure. Une nouvelle fonctionnalité nommée ExpressRoute Fast Path permet de réduire considérablement la latence réseau entre les Grandes instances HANA et les machines virtuelles de la couche Application en grande partie dans Azure (voir ci-dessous). 

Avec la Révision 4 des tampons Grande instance HANA, la latence du réseau entre les machines virtuelles Azure déployées à proximité du tampon Grande instance Hana respecte la classification Moyenne ou Supérieure à la moyenne, tel qu’indiqué dans [Remarque SAP n° 1100926 - FAQ : Performances du réseau](https://launchpad.support.sap.com/#/notes/1100926/E) si le chemin rapide Azure ExpressRoute est configuré (voir ci-dessous). Pour déployer des machines virtuelles Azure à proximité des unités Grande instance Hana de la Révision 4, vous devez tirer parti des [groupes de placements de proximité Azure](../../linux/co-location.md). La méthode d’utilisation des groupes de placement de proximité pour localiser la couche d’application SAP dans le même centre de données Azure que les unités de grande instance HANA hébergées en version 4 est décrite dans la section [Groupes de placement de proximité Azure pour une latence réseau optimale avec les applications SAP](sap-proximity-placement-scenarios.md).

Afin d’offrir une latence réseau déterministe entre les machines virtuelles Azure et une Grande instance HANA, le choix de la référence SKU de la passerelle ExpressRoute est essentiel. Contrairement aux modèles de trafic entre machines locales et machines virtuelles, le modèle de trafic entre les machines virtuelles et la grande instance HANA peut développer des pics courts mais élevés de requêtes et de volumes de données à transmettre. Pour bien gérer ces pics, nous vous recommandons fortement d’utiliser la référence SKU de passerelle UltraPerformance. Pour les références SKU de Grande instance HANA de classe Type II, l’utilisation de la référence SKU de la passerelle UltraPerformance en tant que passerelle ExpressRoute est obligatoire.

> [!IMPORTANT] 
> Étant donné le trafic réseau global entre les couches Base de données et Application SAP, seules les références SKU de passerelle HighPerformance et UltraPerformance pour les réseaux virtuels sont prises en charge pour la connexion à SAP HANA sur Azure (grandes instances). Pour les références SKU de Grande instance HANA de Type II, seule la référence SKU de passerelle UltraPerformance est prise en charge en tant que passerelle ExpressRoute. Des exceptions s’appliquent lorsque vous utilisez ExpressRoute Fast Path (voir ci-dessous)

### <a name="expressroute-fast-path"></a>ExpressRoute Fast Path
Pour réduire la latence, ExpressRoute Fast Path a été introduit et publié en mai 2019 pour la connectivité spécifique des Grandes instances HANA aux réseaux virtuels Azure qui hébergent les machines virtuelles exécutant l’application SAP. La principale différence par rapport à la solution déployée jusqu’à présent est que les flux de données entre les machines virtuelles et les Grandes instances HANA ne sont plus routés via la passerelle ExpressRoute. Au lieu de cela, les machines virtuelles affectés dans les sous-réseaux du réseau virtuel Azure communiquent directement avec le routeur de périphérie d’entreprise dédié. 

> [!IMPORTANT] 
> La fonctionnalité ExpressRoute Fast Path nécessite que les sous-réseaux exécutant les machines virtuelles d’application SAP se trouvent dans le réseau virtuel Azure connecté aux Grandes instances HANA. Les machines virtuelles situées dans des réseaux virtuels Azure appairés au réseau virtuel Azure connecté directement aux unités de Grande instance HANA ne bénéficient pas d’ExpressRoute Fast Path. En conséquence, pour les conceptions de réseau virtuel hub-and-spoke (en étoile) typiques, où les circuits ExpressRoute se connectent à un réseau virtuel faisant office de hub, et où les réseaux virtuels contenant la couche Application SAP (spokes) sont appairés, l’optimisation par ExpressRoute Fast Path ne fonctionne pas. De plus, ExpressRoute Fast Path ne prend pas en charge actuellement les règles de routage définies par l’utilisateur. Pour plus d’informations, voir [Passerelle de réseau virtuel ExpressRoute et FastPath](../../../expressroute/expressroute-about-virtual-network-gateways.md). 


Pour plus d’informations sur la configuration ExpressRoute Fast Path, voir [Connecter un réseau virtuel à de Grandes instances HANA](./hana-connect-vnet-express-route.md).    

> [!NOTE]
> Une passerelle ExpressRoute UltraPerformance est nécessaire pour qu’ExpressRoute Fast Path fonctionne


## <a name="single-sap-system"></a>Système SAP unique

L’infrastructure locale présentée précédemment est connectée via ExpressRoute dans Azure. Le circuit ExpressRoute se connecte à un routeur de périphérie d’entreprise Microsoft (MSEE). Pour plus d’informations, consultez [Vue d’ensemble technique d’ExpressRoute](../../../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Une fois l’itinéraire établi, il se connecte au routeur principal Azure.

> [!NOTE] 
> Pour exécuter des paysages SAP dans Azure, connectez le routeur de périphérie d’entreprise le plus proche de la région Azure dans le paysage SAP. Les tampons de Grande instance HANA sont connectés via les périphériques dédiés du routeur de périphérie d’entreprise afin de réduire la latence réseau entre les machines virtuelles dans Azure IaaS et les tampons de Grande instance HANA.

La passerelle ExpressRoute pour les machines virtuelles qui hébergent des instances d’application SAP est connectée à un circuit ExpressRoute qui se connecte au réseau local. Le même réseau virtuel est connecté à un routeur de périphérie d’entreprise distinct dédié à la connexion aux tampons de grande Instance. Avec ExpressRoute Fast Path, les flux de données des Grandes instances HANA vers les machines virtuelles de la couche Application SAP ne sont plus routés via la passerelle ExpressRoute, ce qui réduit la latence d’aller-retour du réseau.

Ce système est un exemple simple d’un système SAP unique. La couche Application SAP est hébergée dans Azure. La base de données SAP HANA s’exécute sur SAP HANA sur Azure (grandes instances). Nous supposons que le débit de 2 ou 10 Gbit/s de la bande passante de la passerelle ExpressRoute ne constitue pas un goulot d’étranglement.

## <a name="multiple-sap-systems-or-large-sap-systems"></a>Plusieurs systèmes SAP ou grands systèmes SAP

Si plusieurs systèmes SAP ou si de grands systèmes SAP sont déployés pour se connecter à SAP HANA sur Azure (Grandes instances), le débit de la passerelle ExpressRoute peut constituer un goulot d’étranglement. Vous pouvez isoler les systèmes dédiés à la production et ceux qui ne le sont pas dans des réseaux virtuels Azure distincts. Dans ce cas, divisez les couches Application en plusieurs réseaux virtuels. Vous pouvez également créer un réseau virtuel spécial qui se connecte à la grande instance HANA dans les cas suivants :

- Exécution de sauvegardes directement à partir des instances HANA dans la grande instance HANA vers une machine virtuelle dans Azure hébergeant des partages NFS.
- Copie de sauvegardes volumineuses ou d’autres fichiers des unités de grandes instances HANA vers l’espace disque géré dans Azure.

Utilisez un réseau virtuel distinct pour héberger les machines virtuelles qui gèrent le stockage pour le transfert en masse de données entre les Grandes instances HANA et Azure. Ce procédé permet d’éviter les effets d’un fichier ou d’un transfert de données volumineux à partir d’une Grande instance HANA vers Azure sur la passerelle ExpressRoute qui sert les machines virtuelles exécutant la couche Application SAP. 

Pour une architecture réseau plus évolutive :

- Utilisez plusieurs réseaux virtuels pour une couche Application SAP unique, plus grande.
- Déployez un réseau virtuel distinct pour chaque système SAP déployé, plutôt que de combiner ces systèmes SAP en sous-réseaux séparés dans le même réseau virtuel.

  Une architecture réseau plus évolutive pour SAP HANA sur Azure (grandes instances) :

![Déployer la couche Application SAP sur plusieurs réseaux virtuels](./media/hana-overview-architecture/image4-networking-architecture.png)

En fonction des règles et restrictions que vous souhaitez appliquer entre les différents réseaux virtuels hébergeant des machines virtuelles de différents systèmes SAP, vous devez appairer ces réseaux virtuels. Pour plus d’informations sur le peering du réseau virtuel, consultez [Peering de réseaux virtuels](../../../virtual-network/virtual-network-peering-overview.md).


## <a name="routing-in-azure"></a>Routage dans Azure

Pour un déploiement par défaut, trois points importants sont à prendre en compte en matière de routage pour SAP HANA sur Azure (Grandes instances) :

* L’accès à SAP HANA sur Azure (grandes instances) se fait uniquement via des machines virtuelles Azure dans la connexion ExpressRoute dédiée, non directement via le système local. L’accès direct du réseau local aux unités de grande instance HANA, tel que fourni par Microsoft, n’est pas possible immédiatement. Les restrictions de routage transitives sont dues à l’architecture réseau Azure actuelle utilisée pour la grande instance SAP HANA. Certains clients d’administration et toutes les applications nécessitant un accès direct, tel que le gestionnaire de solutions SAP s’exécutant en local, ne peuvent pas se connecter à la base de données SAP HANA. Pour les exceptions, voir la section « Routage direct vers de Grandes instances HANA ».

* Si vous avez des unités de Grande instance HANA déployées dans deux régions Azure différentes à des fins de récupération d’urgence, les mêmes restrictions de routage temporaire s’appliquaient dans le passé. En d’autres termes, les adresses IP d’une unité de Grande instance HANA dans une région (par exemple, USA Ouest) n’étaient pas routées vers une unité de Grande instance HANA déployée dans une autre région (par exemple, USA Est). Cette restriction était indépendante de l’utilisation du peering de réseau Azure entre les différentes régions ou des interconnexions des circuits ExpressRoute qui connectent les unités de Grande instance HANA aux réseaux virtuels. Pour obtenir une représentation graphique, consultez la figure de la section « Utiliser des unités de grande instance HANA dans plusieurs régions. » Cette restriction, qui résultait de l’architecture déployée, empêchait l’utilisation immédiate de la réplication du système HANA en tant que fonctionnalité de récupération d’urgence. Pour les modifications récentes, voir la section « Utiliser des unités de Grande instance HANA dans plusieurs régions ». 

* Les unités SAP HANA sur Azure (Grandes instances) ont une adresse IP assignée à partir de la plage d’adresses du pool d’adresses IP du serveur que vous avez envoyée lors de la demande du déploiement de Grande instance HANA. Pour plus d’informations, consultez [Infrastructure et connectivité SAP HANA (grandes instances) sur Azure](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Cette adresse IP est accessible via les abonnements Azure et le circuit qui connecte les réseaux virtuels Azure aux Grandes instances HANA. L’adresse IP assignée hors que plage d’adresses de pools IP du serveur est directement assignée à l’unité matérielle. Elle n’est *plus* assignée via la traduction d'adresses réseau, comme c’était le cas dans les premiers déploiements de cette solution. 

### <a name="direct-routing-to-hana-large-instances"></a>Routage direct vers de Grandes Instances HANA

Par défaut, le routage transitif ne fonctionne pas dans les scénarios suivants :

* Entre les unités de grande instance HANA et un déploiement local.

* Entre des routages de grande instance HANA déployés dans deux régions différentes.

Il existe trois façons d’activer le routage transitif dans ces scénarios :

- Un proxy inverse pour router les données, dans un sens et dans l’autre. Par exemple, F5 BIG-IP, NGINX avec Traffic Manager déployé dans le réseau virtuel Azure qui se connecte aux Grandes instances HANA et au réseau local en tant que solution de routage de pare-feu/trafic virtuel.
- Utilisation des règles IPTables sur une machine virtuelle Linux pour activer le routage entre des emplacements locaux et des unités de grande instance HANA ou entre des unités de grande instance HANA dans différentes régions. La machine virtuelle exécutant IPTables doit être déployée sur le réseau virtuel Azure qui se connecte aux Grandes instances HANA et au réseau local. La machine virtuelle doit être dimensionnée en conséquence, de sorte que son débit réseau soit suffisant pour le trafic réseau attendu. Pour plus d’informations sur la bande passante réseau machines virtuelles, voir l’article [Tailles des machines virtuelles Linux dans Azure](../../sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
- Un [Pare-feu Azure](https://azure.microsoft.com/services/azure-firewall/) serait une autre solution pour permettre un trafic direct entre le réseau local et les unités de Grande instance HANA. 

Tout le trafic de ces solutions serait acheminé via un réseau virtuel Azure. De ce fait, le trafic pourrait également être limité par les appliances logicielles utilisées ou par des groupes de sécurité réseau Azure. Ainsi, certaines adresses ou plages d’adresses IP du réseau local pourraient être bloquées ou explicitement autorisées à accéder aux Grandes instances HANA. 

> [!NOTE]  
> N’oubliez pas que l’implémentation et la prise en charge des solutions personnalisées qui impliquent des appliances réseau ou IPTables tierces n’est pas fournie par Microsoft. La prise en charge doit être réalisée par le fournisseur du composant utilisé ou de l’intégrateur. 

#### <a name="express-route-global-reach"></a>ExpressRoute Global Reach
Microsoft a introduit une nouvelle fonctionnalité appelée [ExpressRoute Global Reach](../../../expressroute/expressroute-global-reach.md). Global Reach est utilisable pour les Grandes instances HANA dans deux scénarios :

- Permettre l’accès direct à partir du réseau local à vos unités de Grande instance HANA déployées dans différentes régions
- Permettre une communication directe entre vos unités de Grande instance HANA déployées dans différentes régions


##### <a name="direct-access-from-on-premises"></a>Accès direct à partir du réseau local
Dans les régions Azure où Global Reach est proposé, vous pouvez demander l’activation de la fonctionnalité Global Reach pour votre circuit ExpressRoute qui connecte votre réseau local au réseau virtuel Azure qui se connecte également à vos unités de Grande instance HANA. Il existe des implications de coûts pour le côté local de votre circuit ExpressRoute. Pour connaître les prix, voir [Composant additionnel Global Reach](https://azure.microsoft.com/pricing/details/expressroute/). Le circuit qui connecte les unités de Grande instance HANA à Azure n’occasionne aucun coût supplémentaire pour vous. 

> [!IMPORTANT]  
> Si vous utilisez Global Reach pour permettre un accès direct entre vos unités de Grande instance HANA et vos ressources locales, les flux de données et de contrôle du réseau ne sont **pas routés via des réseaux virtuels Azure**, mais directement entre les routeurs d’échange d’entreprise Microsoft. Par conséquent, aucune règle de groupe de sécurité réseau ou de groupe de sécurité d’application, ni aucun type de pare-feu, de groupe de sécurité réseau ou de proxy que vous avez déployés dans un réseau virtuel Azure, ne sont concernés. **Si vous utilisez ExpressRoute Global Reach pour permettre un accès direct du réseau local aux unités de Grande instance HANA, les restrictions et autorisations d’accès aux unités de Grande instance HANA doivent être définies dans des pare-feu côté local** 

##### <a name="connecting-hana-large-instances-in-different-azure-regions"></a>Connexion de Grandes Instances HANA dans différentes régions Azure
De la même manière, ExpressRoute Global Reach pouvant être utilisé pour la connexion du réseau local aux unités de Grande instance HANA, il peut également l’être pour se connecter à des locataires de Grande instance HANA déployés pour vous dans deux régions différentes. L’isolation est assurée par les circuits ExpressRoute que vos locataires de Grande instance HANA utilisent pour se connecter à Azure dans les deux régions. La connexion de deux locataires de Grande instance HANA déployés dans deux régions différentes n’occasionne pas de frais supplémentaires. 

> [!IMPORTANT]  
> Les flux de données et de contrôle du trafic réseau entre les différents locataires de Grande instance HANA ne sont pas routés via des réseaux Azure. Par conséquent, vous ne pouvez pas utiliser de fonctionnalité Azure ou d’appliances de réseau virtuel pour appliquer des restrictions de communication entre vos deux locataires de Grande instance HANA. 

Pour plus d’informations sur l’activation d’ExpressRoute Global Reach, voir [Connecter un réseau virtuel à de Grandes instances HANA](./hana-connect-vnet-express-route.md).


## <a name="internet-connectivity-of-hana-large-instance"></a>Connectivité internet de la grande instance HANA
La grande instance HANA ne bénéficie *pas* d’une connectivité directe à Internet. Par exemple, cette limitation peut restreindre votre capacité à enregistrer l’image du système d’exploitation directement avec le fournisseur du système d’exploitation. Vous devrez peut-être travailler avec votre serveur d’outil de gestion des abonnements SUSE Linux Enterprise Server local ou d’un gestionnaire des abonnements Red Hat Enterprise Linux.

## <a name="data-encryption-between-vms-and-hana-large-instance"></a>Chiffrement des données entre des machines virtuelles et la grande instance HANA
Les données transférées entre la grande instance HANA et des machines virtuelles ne sont pas chiffrées. En revanche, vous pouvez activer le chiffrement du trafic pour l’échange simple entre le SGBD HANA et les applications basées sur JDBC/ODBC. Pour plus d’informations, consultez [cette documentation par SAP](https://help.sap.com/viewer/102d9916bf77407ea3942fef93a47da8/1.0.11/en-US/dbd3d887bb571014bf05ca887f897b99.html).

## <a name="use-hana-large-instance-units-in-multiple-regions"></a>Utiliser des unités de grande instance HANA dans plusieurs régions

Pour réaliser des configurations de récupération d’urgence, vous devez disposer d’unités de Grande instance HANA dans plusieurs régions Azure. Même avec Azure [Global Vnet Peering], le routage transitif par défaut ne fonctionne pas entre des locataires de Grande instance HANA situés dans deux régions différentes. Cependant, Global Reach ouvre le chemin de communication entre les unités de Grande instance HANA que vous avez approvisionnées dans deux régions différentes. Ce scénario d’utilisation de Microsoft Azure ExpressRoute permet d’effectuer les opérations suivantes :

 - Réplication de système HANA sans proxy ou pare-feu supplémentaires
 - Copie de sauvegardes entre des unités de Grande instance HANA situées dans deux régions différentes pour effectuer des copies ou actualisations du système


![Réseau virtuel connecté aux tampons de grande instance Azure dans différentes régions Azure](./media/hana-overview-architecture/image8-multiple-regions.png)

La figure illustre comment les différents réseaux virtuels dans les deux régions sont connectés à deux circuits ExpressRoute distincts utilisés pour se connecter à SAP HANA sur Azure (Grandes instances) dans les deux régions Azure (lignes grises). Ces deux interconnexions visent à offrir une protection contre les pannes de routeur de périphérie d’entreprise Microsoft (MSEE) de chaque côté. Le flux de communication entre les deux réseaux virtuels dans les deux régions Azure est censé être géré via le [peering mondial](/archive/blogs/azureedu/how-to-setup-global-vnet-peering-in-azure) des deux réseaux virtuels dans les deux régions (ligne pointillée bleue). La ligne rouge épaisse décrit la connexion ExpressRoute Global Reach qui permet aux unités de Grande instance HANA de vos locataires dans deux régions différentes de communiquer entre elles. 

> [!IMPORTANT] 
> Si plusieurs circuits ExpressRoute sont utilisés, les paramètres de préfixation AS PATH et Local Preference BGP doivent être utilisés pour garantir le routage correct du trafic.

**Étapes suivantes**
- Consultez [Architecture de stockage de SAP HANA (grandes instances)](hana-storage-architecture.md)