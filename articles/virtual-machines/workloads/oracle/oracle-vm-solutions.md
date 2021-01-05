---
title: Solutions Oracle sur les machines virtuelles Azure | Microsoft Docs
description: Découvrez les configurations prises en charge et les limitations des images de machine virtuelle Oracle sur Microsoft Azure.
author: dbakevlar
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 05/12/2020
ms.author: kegorman
ms.reviewer: cynthn
ms.openlocfilehash: 093d3c0da0b40f2a287b1cb451a419f71ce4b8d7
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94957586"
---
# <a name="oracle-vm-images-and-their-deployment-on-microsoft-azure"></a>Images de machine virtuelle Oracle et leur déploiement sur Microsoft Azure

Cet article présente les solutions Oracle basées sur des images de machine virtuelle publiées par Oracle sur la Place de marché Microsoft Azure. Si vous êtes intéressé par des solutions d’application entre cloud avec l’infrastructure cloud Oracle, consultez [Solutions d’applications Oracle intégrant Microsoft Azure et l’infrastructure cloud Oracle](oracle-oci-overview.md).

Pour obtenir la liste des images actuellement disponibles, exécutez la commande suivante :

```azurecli-interactive
az vm image list --publisher oracle -o table --all
```

Depuis juin 2020, les images suivantes sont disponibles :

```bash
Offer                   Publisher    Sku                     Urn                                                          Version
----------------------  -----------  ----------------------  -----------------------------------------------------------  -------------
oracle-database-19-3    Oracle       oracle-db-19300         Oracle:oracle-database-19-3:oracle-db-19300:19.3.0           19.3.0
Oracle-Database-Ee      Oracle       12.1.0.2                Oracle:Oracle-Database-Ee:12.1.0.2:12.1.20170220             12.1.20170220
Oracle-Database-Ee      Oracle       12.2.0.1                Oracle:Oracle-Database-Ee:12.2.0.1:12.2.20180725             12.2.20180725
Oracle-Database-Ee      Oracle       18.3.0.0                Oracle:Oracle-Database-Ee:18.3.0.0:18.3.20181213             18.3.20181213
Oracle-Database-Se      Oracle       12.1.0.2                Oracle:Oracle-Database-Se:12.1.0.2:12.1.20170220             12.1.20170220
Oracle-Database-Se      Oracle       12.2.0.1                Oracle:Oracle-Database-Se:12.2.0.1:12.2.20180725             12.2.20180725
Oracle-Database-Se      Oracle       18.3.0.0                Oracle:Oracle-Database-Se:18.3.0.0:18.3.20181213             18.3.20181213
Oracle-Linux            Oracle       6.10                    Oracle:Oracle-Linux:6.10:6.10.00                             6.10.00
Oracle-Linux            Oracle       6.8                     Oracle:Oracle-Linux:6.8:6.8.0                                6.8.0
Oracle-Linux            Oracle       6.8                     Oracle:Oracle-Linux:6.8:6.8.20190529                         6.8.20190529
Oracle-Linux            Oracle       6.9                     Oracle:Oracle-Linux:6.9:6.9.0                                6.9.0
Oracle-Linux            Oracle       6.9                     Oracle:Oracle-Linux:6.9:6.9.20190529                         6.9.20190529
Oracle-Linux            Oracle       7.3                     Oracle:Oracle-Linux:7.3:7.3.0                                7.3.0
Oracle-Linux            Oracle       7.3                     Oracle:Oracle-Linux:7.3:7.3.20190529                         7.3.20190529
Oracle-Linux            Oracle       7.4                     Oracle:Oracle-Linux:7.4:7.4.1                                7.4.1
Oracle-Linux            Oracle       7.4                     Oracle:Oracle-Linux:7.4:7.4.20190529                         7.4.20190529
Oracle-Linux            Oracle       7.5                     Oracle:Oracle-Linux:7.5:7.5.1                                7.5.1
Oracle-Linux            Oracle       7.5                     Oracle:Oracle-Linux:7.5:7.5.2                                7.5.2
Oracle-Linux            Oracle       7.5                     Oracle:Oracle-Linux:7.5:7.5.20181207                         7.5.20181207
Oracle-Linux            Oracle       7.5                     Oracle:Oracle-Linux:7.5:7.5.20190529                         7.5.20190529
Oracle-Linux            Oracle       7.6                     Oracle:Oracle-Linux:7.6:7.6.2                                7.6.2
Oracle-Linux            Oracle       7.6                     Oracle:Oracle-Linux:7.6:7.6.3                                7.6.3
Oracle-Linux            Oracle       7.6                     Oracle:Oracle-Linux:7.6:7.6.4                                7.6.4
Oracle-Linux            Oracle       77                      Oracle:Oracle-Linux:77:7.7.1                                 7.7.1
Oracle-Linux            Oracle       77                      Oracle:Oracle-Linux:77:7.7.2                                 7.7.2
Oracle-Linux            Oracle       77                      Oracle:Oracle-Linux:77:7.7.3                                 7.7.3
Oracle-Linux            Oracle       77                      Oracle:Oracle-Linux:77:7.7.4                                 7.7.4
Oracle-Linux            Oracle       77                      Oracle:Oracle-Linux:77:7.7.5                                 7.7.5
Oracle-Linux            Oracle       77-ci                   Oracle:Oracle-Linux:77-ci:7.7.01                             7.7.01
Oracle-Linux            Oracle       77-ci                   Oracle:Oracle-Linux:77-ci:7.7.02                             7.7.02
Oracle-Linux            Oracle       77-ci                   Oracle:Oracle-Linux:77-ci:7.7.03                             7.7.03
Oracle-Linux            Oracle       78                      Oracle:Oracle-Linux:78:7.8.01                                7.8.01
Oracle-Linux            Oracle       8                       Oracle:Oracle-Linux:8:8.0.2                                  8.0.2
Oracle-Linux            Oracle       8-ci                    Oracle:Oracle-Linux:8-ci:8.0.11                              8.0.11
Oracle-Linux            Oracle       81                      Oracle:Oracle-Linux:81:8.1.0                                 8.1.0
Oracle-Linux            Oracle       81                      Oracle:Oracle-Linux:81:8.1.2                                 8.1.2
Oracle-Linux            Oracle       81-ci                   Oracle:Oracle-Linux:81-ci:8.1.0                              8.1.0
Oracle-Linux            Oracle       81-gen2                 Oracle:Oracle-Linux:81-gen2:8.1.11                           8.1.11
Oracle-Linux            Oracle       ol77-ci-gen2            Oracle:Oracle-Linux:ol77-ci-gen2:7.7.1                       7.7.1
Oracle-Linux            Oracle       ol77-gen2               Oracle:Oracle-Linux:ol77-gen2:7.7.01                         7.7.01
Oracle-Linux            Oracle       ol77-gen2               Oracle:Oracle-Linux:ol77-gen2:7.7.02                         7.7.02
Oracle-Linux            Oracle       ol78-gen2               Oracle:Oracle-Linux:ol78-gen2:7.8.11                         7.8.11
Oracle-WebLogic-Server  Oracle       Oracle-WebLogic-Server  Oracle:Oracle-WebLogic-Server:Oracle-WebLogic-Server:12.1.2  12.1.2
weblogic-122130-jdk8u3  Oracle       owls-122130-8u131-ol73  Oracle:weblogic-122130-jdk8u131-ol73:owls-122130-8u131-ol7   1.1.6
weblogic-122130-jdk8u4  Oracle       owls-122130-8u131-ol74  Oracle:weblogic-122130-jdk8u131-ol74:owls-122130-8u131-ol7   1.1.1
weblogic-122140-jdk8u6  Oracle       owls-122140-8u251-ol76  Oracle:weblogic-122140-jdk8u251-ol76:owls-122140-8u251-ol7   1.1.1
weblogic-141100-jdk116  Oracle       owls-141100-11_07-ol76  Oracle:weblogic-141100-jdk11_07-ol76:owls-141100-11_07-ol7   1.1.1
weblogic-141100-jdk8u6  Oracle       owls-141100-8u251-ol76  Oracle:weblogic-141100-jdk8u251-ol76:owls-141100-8u251-ol7   1.1.1
```

Ces images sont considérées comme de type BYOL (apportez votre propre licence). Par conséquent, vous serez facturé uniquement pour les frais de calcul, de stockage et de mise en réseau induits par l’exécution d’une machine virtuelle.  Vous êtes supposé disposer des licences appropriées pour utiliser les logiciels Oracle, ainsi que d’un contrat de support avec Oracle. Oracle a garanti la mobilité des licence locale vers Azure. Pour plus d’informations sur la mobilité des licences, voir la note publiée [Oracle et Microsoft](https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html).

Les utilisateurs peuvent également choisir de baser leurs solutions sur une image personnalisée qu’ils créent de toutes pièces dans Azure, ou de charger des images personnalisées à partir de leur environnement local.

## <a name="oracle-database-vm-images"></a>Images de machine virtuelle Oracle Database

Oracle prend en charge l’exécution des éditions Standard et Entreprise d’Oracle Database 12.1 et versions ultérieures dans Azure sur des images de machine virtuelle basées sur Oracle Linux.  Pour optimiser les performances des charges de travail de production d’Oracle Database sur Azure, veillez à dimensionner correctement l’image de machine virtuelle et à utiliser des disques managés SSD Premium ou SSD Ultra. Pour obtenir des instructions sur la façon d’obtenir rapidement une base de données Oracle opérationnelle dans Azure à l’aide de l’image de machine virtuelle publiée d’Oracle, [essayez de suivre la procédure de démarrage rapide de base de données Oracle](oracle-database-quick-create.md).

### <a name="attached-disk-configuration-options"></a>Options de configuration des disques connectés

Les disques attachés s’appuient sur le service de stockage d’objets blob Microsoft Azure. En théorie, chaque disque standard peut proposer un maximum de 500 entrées/sorties par seconde (IOPS). Notre offre de disque Premium est conseillée pour les charges de travail de base de données hautes performances et peut atteindre jusqu’à 5 000 IOPS par disque. Vous pouvez utiliser un seul disque si cela correspond à vos besoins en matière de performances. Toutefois, vous pouvez améliorer les performances d’IOPS effectives si vous utilisez plusieurs disques attachés, répartissez les données de la base de données sur ces disques, puis lancez ASM (Oracle Automatic Storage Management). Pour plus d’informations spécifiques d’Oracle ASM, voir [vue d’ensemble du stockage automatique Oracle](https://www.oracle.com/technetwork/database/index-100339.html). Pour un exemple d’installation et de configuration d’Oracle ASM sur une machine virtuelle Linux Azure, consultez le tutoriel [Installation et configuration d’Oracle Automated Storage Management](configure-oracle-asm.md).

### <a name="shared-storage-configuration-options"></a>Options de configuration de stockage partagé

Azure NetApp Files a été conçu pour répondre aux besoins essentiels de l’exécution de charges de travail hautes performances comme des bases de données dans le cloud, et fournit ;

- un service de stockage NFS partagé natif Azure pour l’exécution de charges de travail Oracle via un client NFS natif de machine virtuelle ou Oracle dNFS ;
- des niveaux de performances évolutifs qui reflètent la portée réelle des demandes IOPS ;
- Latence faible
- Une haute disponibilité, une durabilité élevée et une facilité de gestion à grande échelle, généralement nécessaires pour des charges de travail critiques de l’entreprise (par exemple, SAP et Oracle) ;
- une sauvegarde rapide et efficace pour obtenir des contrats SLA de RTO et RPO agressifs.

Ces fonctionnalités sont possibles car Azure NetApp Files est basé sur des systèmes 100 % Flash NetApp® ONTAP® qui s’exécutent au sein d’un environnement de centre de données Azure, tel un service natif Azure. En résulte une technologie de stockage de base idéale qui peut être approvisionnée et utilisée comme n’importe quelle autre option de stockage Azure. Consultez la [documentation Azure NetApp Files](../../../azure-netapp-files/index.yml) pour savoir comment déployer des volumes NFS Azure NetApp Files et y accéder. Consultez [Guide des meilleures pratiques de déploiement d’Oracle sur Azure à l’aide d’Azure NetApp Files](https://www.netapp.com/us/media/tr-4780.pdf) pour connaître les meilleures pratiques concernant l’emploi d’une base de données Oracle sur Azure NetApp Files.

## <a name="licensing-oracle-database--software-on-azure"></a>Gestion des licences logiciels Oracle Database sur Azure

Microsoft Azure est un environnement cloud agréé pour l’exécution d’Oracle Database. Le tableau Oracle Core Factor ne s’applique pas à la gestion des licences de bases de données Oracle dans le cloud. À la place, lorsque vous utilisez des machines virtuelles avec la technologie Hyper-Threading activée pour les bases de données Édition Entreprise, comptez deux vCPU comme équivalant à une licence Oracle Processor si l’hyperthreading est activé (comme indiqué dans le document de stratégie). Vous trouverez les détails de la stratégie [ici](http://www.oracle.com/us/corporate/pricing/cloud-licensing-070579.pdf).
Les bases de données Oracle nécessitent généralement davantage de mémoire et d’E/S. C’est pourquoi l’usage de [machines virtuelles à mémoire optimisée](../../sizes-memory.md) est recommandé pour ces charges de travail. Pour optimiser davantage vos charges de travail, il est recommandé d’utiliser des [processeurs virtuels au nombre de cœurs limité](../../constrained-vcpu.md) pour les charges de travail Oracle Database nécessitant des capacités importantes de mémoire, de stockage et de bande passante d’E/S, mais pas un nombre important de cœurs.

Lors de la migration de logiciels et charges de travail Oracle d’un environnement local vers Microsoft Azure, Oracle offre la mobilité de licence décrite dans le [Forum aux questions concernant Oracle sur Azure](https://www.oracle.com/cloud/technologies/oracle-azure-faq.html)

## <a name="high-availability-and-disaster-recovery-considerations"></a>Remarques relatives à la haute disponibilité et à la récupération d’urgence

Lors de l’utilisation de bases de données Oracle dans Azure, vous êtes responsable de l’implémentation d’une solution de haute disponibilité et récupération d’urgence pour éviter tout temps d’arrêt.

Pour assurer la haute disponibilité et la récupération d’urgence pour Oracle Database Enterprise Edition (sans reposer sur RAC) sur Microsoft Azure, vous pouvez utiliser [Data Guard, Active Data Guard](https://www.oracle.com/database/technologies/high-availability/dataguard.html) ou [Oracle Golden Gate](https://www.oracle.com/technetwork/middleware/goldengate), en plaçant deux bases de données sur deux machines virtuelles distinctes. Ces deux machines doivent se trouver dans le même [réseau virtuel](https://azure.microsoft.com/documentation/services/virtual-network/), afin de pouvoir être accessibles l’une à l’autre via l’adresse IP privée persistante.  En outre, nous vous recommandons de placer les machines virtuelles au sein du même groupe à haute disponibilité, afin de permettre à Azure de les placer dans des domaines d’erreur et de mise à niveau distincts. Si vous souhaitez disposer de la géo-redondance, configurez ces deux bases de données de sorte qu’elles soient répliquées dans deux régions différentes, et connectez les deux instances avec une passerelle VPN.

Le didacticiel [Implémenter Oracle DataGuard sur Azure](configure-oracle-dataguard.md), qui vous guide dans la procédure d’installation de base sur Azure.  

Grâce à Oracle Data Guard, vous pouvez assurer la haute disponibilité du système en plaçant la base de données primaire sur une machine virtuelle et une base de données secondaire (de veille) sur une autre, et en configurant une réplication monodirectionnelle entre ces bases de données. Vous bénéficiez ainsi d’un accès en lecture à la copie de la base de données. Oracle GoldenGate vous permet de configurer une réplication bidirectionnelle entre les deux bases de données. Pour savoir comment configurer une solution de haute disponibilité pour vos bases de données en utilisant ces outils, voir la documentation relative à [Active Data Guard](https://www.oracle.com/database/technologies/high-availability/dataguard.html) et [GoldenGate](https://docs.oracle.com/goldengate/1212/gg-winux/index.html) sur le site web d’Oracle. Si vous avez besoin d’un accès en lecture et en écriture à la copie de la base de données, vous pouvez utiliser [Oracle Active Data Guard](https://www.oracle.com/uk/products/database/options/active-data-guard/overview/index.html).

Le didacticiel [Implémenter Oracle GoldenGate sur Azure](configure-oracle-golden-gate.md), qui vous guide dans la procédure d’installation de base sur Azure.

Bien qu’une solution de haute disponibilité et récupération d’urgence soit architecturée dans Azure, vous devez veiller à avoir une stratégie de sauvegarde en place pour restaurer votre base de données. Le didacticiel [Sauvegarde et restauration d’une base de données Oracle](./oracle-overview.md) vous guide tout au long de la procédure de base pour l’établissement d’une sauvegarde cohérente.

## <a name="support-for-jd-edwards"></a>Prise en charge de JD Edwards

Selon la note de prise en charge d’Oracle [Doc ID 2178595.1](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=573435677515785&id=2178595.1&_afrWindowMode=0&_adf.ctrl-state=o852dw7d_4), JD Edwards Enterprise One versions 9.2 et ultérieures est pris en charge sur **toute offre de cloud public** conforme aux `Minimum Technical Requirements` (MTR).  Vous devez créer des images personnalisées répondant à leurs spécifications MTR pour la compatibilité des applications logicielles et des systèmes d’exploitation.

## <a name="oracle-weblogic-server-virtual-machine-offers"></a>Offres de machines virtuelles Oracle WebLogic Server

Oracle et Microsoft collaborent pour mettre WebLogic Server sur Place de marché Azure sous la forme d’une collection d’offres Azure Application.  Ces offres sont décrites dans l’article [Applications Azure Oracle WebLogic Server](oracle-weblogic.md).

### <a name="oracle-weblogic-server-virtual-machine-images"></a>Images de machines virtuelles Oracle WebLogic Server

- **Le clustering est pris en charge sur la version Enterprise Edition uniquement.** Vous disposez d’une licence pour utiliser le clustering WebLogic uniquement lorsque vous utilisez l’édition Enterprise d’Oracle WebLogic Server. N’utilisez pas le clustering avec Oracle WebLogic Server Standard Edition.
- **La multidiffusion UDP n’est pas gérée.** Azure prend en charge la monodiffusion UDP, mais ni la diffusion, ni la multidiffusion. Oracle WebLogic Server est capable de s’appuyer sur les fonctionnalités de monodiffusion UDP d’Azure. Pour optimiser les résultats lorsque vous recourez à la monodiffusion UDP, nous vous recommandons de conserver une taille de cluster statique, ou de maintenir ce dernier à une taille maximale de 10 serveurs gérés.
- **Oracle WebLogic Server attend des ports publics et privés identiques pour l’accès T3 (par exemple, lorsque vous utilisez Enterprise JavaBeans).** Imaginez un scénario à plusieurs niveaux, dans lequel une application de couche de service (EJB) est en cours d’exécution sur un cluster Oracle WebLogic Server comprenant au moins deux machines virtuelles, au sein d’un réseau virtuel nommé *SLWLS*. Le niveau client se trouve dans un autre sous-réseau du même réseau virtuel, qui exécute un programme Java simple s’efforçant d’appeler Enterprise JavaBeans dans la couche de service. Comme il est nécessaire d’équilibrer la charge de la couche de service, un point de terminaison public à charge équilibrée doit être créé pour les machines virtuelles du cluster Oracle WebLogic Server. Si le port privé que vous spécifiez est différent du port public (par exemple, 7006:7008), une erreur semblable à la suivante se produit :

```bash
   [java] javax.naming.CommunicationException [Root exception is java.net.ConnectException: t3://example.cloudapp.net:7006:

   Bootstrap to: example.cloudapp.net/138.91.142.178:7006' over: 't3' got an error or timed out]
```

   En effet, Oracle WebLogic Server attend un port d’équilibreur de charge et un port de serveur managé WebLogic identiques pour tout accès T3 distant. Dans le cas précédent, le client accède au port 7006 (port d’équilibreur de charge) et le serveur managé est à l’écoute sur le port 7008 (port privé). Cette restriction porte uniquement sur l’accès T3, et non sur le protocole HTTP.

   Pour éviter ce problème, utilisez l’une des solutions de contournement suivantes :

- Utilisez les mêmes numéros de port public et privé pour les points de terminaison avec équilibrage de charge dédiés à l’accès T3.
- Incluez le paramètre JVM suivant lors du démarrage d’Oracle WebLogic Server :

```bash
   -Dweblogic.rjvm.enableprotocolswitch=true
```

Pour plus d’informations, consultez l’article de la Base de connaissances **860340.1** à l’adresse <https://support.oracle.com>.

- **Limitations relatives à l’équilibrage de charge et au clustering dynamique.** Partons du principe que vous souhaitez utiliser un cluster dynamique dans Oracle WebLogic Server et l’exposer par le biais d’un seul point de terminaison public à charge équilibrée dans Azure. Cette opération est possible tant que vous utilisez un numéro de port fixe pour chaque serveur géré (et non affecté de manière dynamique à partir d’une plage de valeurs), et que le nombre de serveurs gérés démarrés reste inférieur à celui des machines surveillées par l’administrateur. En d’autres termes, il faut un seul serveur géré par machine virtuelle. Si, suite à votre configuration, le nombre de serveurs Oracle WebLogic Server démarrés est supérieur à celui des machines virtuelles (c’est-à-dire si plusieurs instances Oracle WebLogic Server partagent la même machine virtuelle), une seule de ces instances de serveurs Oracle WebLogic Server peut être liée à un numéro de port donné à la fois. Les autres instances de la machine virtuelle échouent.

   Si vous configurez le serveur d’administration de façon à affecter automatiquement des numéros de port uniques à ses serveurs managés, aucun équilibrage de charge n’est possible, car Microsoft Azure ne prend pas en charge le mappage d’un port public unique à plusieurs ports privés (opération alors requise pour cette configuration).
- **Exécution de plusieurs instances Oracle WebLogic Server sur une seule machine virtuelle.** Selon les exigences liées à votre déploiement, vous pouvez envisager d’exécuter plusieurs instances Oracle WebLogic Server sur la même machine virtuelle, si la taille de cette dernière est suffisante. Ainsi, sur une machine virtuelle de taille moyenne contenant deux cœurs, vous pouvez opter pour l’exécution de deux instances Oracle WebLogic Server. Toutefois, nous vous recommandons malgré tout d’éviter toute introduction de points de défaillance uniques dans votre architecture (situation susceptible de survenir si vous utilisez une seule machine virtuelle exécutant plusieurs instances Oracle WebLogic Server). L’utilisation d’un minimum de deux machines virtuelles peut s’avérer plus efficace ; chaque machine virtuelle peut alors exécuter plusieurs instances Oracle WebLogic Server. Chaque instance Oracle WebLogic Server peut continuer de faire partie du même cluster. Toutefois, il n’est actuellement pas possible d’utiliser Azure pour équilibrer la charge des points de terminaison exposés par des déploiements Oracle WebLogic Server de ce type au sein d’une même machine virtuelle. En effet, l’équilibreur de charge Azure requiert la répartition des serveurs à charge équilibrée sur des machines virtuelles uniques.

## <a name="oracle-jdk-virtual-machine-images"></a>Images de machine virtuelle JDK Oracle

- **Dernières mises à jour de JDK 6 et 7.** Nous recommandons d’utiliser la dernière version publique de Java prise en charge (il s’agit actuellement de la version 8). Cependant, Azure propose également des images de JDK 6 et 7. Ces images sont destinées aux applications héritées qui ne peuvent pas encore être mises à niveau vers la version 8 de JDK. Il se peut que les mises à jour des images JDK précédentes ne soient plus mises à la disposition du public. Toutefois, suite au partenariat entre Microsoft et Oracle, les images de JDK 6 et 7 proposées par Microsoft Azure ont pour objectif de contenir une mise à jour plus récente, non destinée au public, qui est généralement proposée par Oracle à certains clients spécifiques bénéficiant d’un support Oracle. De nouvelles versions des images de JDK seront mises à disposition au fur et à mesure, en parallèle avec les versions de JDK 6 et 7 mises à jour.

   Le JDK disponible dans ces images de JDK 6 et 7, ainsi que les machines virtuelles et les images dérivées, peuvent uniquement être utilisées au sein d’Azure.
- **JDK 64 bits.** Les images de machine virtuelle Oracle WebLogic Server et les images de machine virtuelle Oracle JDK fournies par Microsoft Azure contiennent les versions 64 bits de Windows Server et de JDK.

## <a name="next-steps"></a>Étapes suivantes

Vous avez maintenant une vue d’ensemble des Solutions Oracle actuelles en fonction des images de machine virtuelle dans Microsoft Azure. L’étape suivante consiste à déployer votre première base de données Oracle sur Azure.

> [!div class="nextstepaction"]
> [Créer une base de données Oracle sur Azure](oracle-database-quick-create.md)
