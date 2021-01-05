---
title: Synchronisation de l’heure pour les machines virtuelles Linux dans Azure
description: Synchronisation de l’heure pour les machines virtuelles Linux.
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.topic: how-to
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/20/2020
ms.author: cynthn
ms.openlocfilehash: 399022c1ef740865e4b2f7b82e2175e748a2a925
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91306954"
---
# <a name="time-sync-for-linux-vms-in-azure"></a>Synchronisation de l’heure pour les machines virtuelles Linux dans Azure

La synchronisation de l’heure est importante pour la sécurité et la corrélation des événements. Elle est parfois utilisée pour l’implémentation de transactions distribuées. La précision de l’heure entre plusieurs systèmes informatiques est obtenue via la synchronisation. La synchronisation peut être influencée par plusieurs facteurs, tels que les redémarrages et le trafic réseau entre la source de l’heure et l’ordinateur qui récupère l’heure. 

Azure est soutenu par une infrastructure exécutant Windows Server 2016. Windows Server 2016 utilise des algorithmes améliorés qui permettent de corriger l’heure et de configurer l’horloge locale pour la synchroniser avec l’heure UTC.  La fonctionnalité d’heure précise de Windows Server 2016 a considérablement amélioré la façon dont le service VMICTimeSync régit les machines virtuelles avec l’hôte pour une heure précise. Les améliorations incluent une heure initiale plus précise au démarrage ou lors de la restauration d’une machine virtuelle, ainsi que l’interruption de la correction de la latence. 

> [!NOTE]
> Pour une présentation rapide du service d’heure de Windows, regardez cette [vidéo de présentation globale](https://aka.ms/WS2016TimeVideo).
>
> Pour plus d’informations, consultez la rubrique [Précision de l’heure sur Windows Server 2016](/windows-server/networking/windows-time-service/accurate-time). 

## <a name="overview"></a>Vue d’ensemble

La précision de l’horloge de l’ordinateur est évaluée en fonction de la proximité de l’horloge de l’ordinateur par rapport à la norme de temps universel coordonné (UTC). L’heure UTC est définie par l’exemple multinational des horloges atomiques précises qui ne peuvent être décalées que d’une seconde en 300 ans. Toutefois, la lecture directe de l’heure UTC requiert du matériel spécialisé. Au lieu de cela, les serveurs de temps sont synchronisés à l’heure UTC et sont accessibles depuis d’autres ordinateurs dans un souci d’évolutivité et de fiabilité. Chaque ordinateur possède un service de synchronisation d’heure en cours d’exécution qui sait quels serveurs de temps utiliser et vérifie régulièrement si l’horloge de l’ordinateur doit être corrigée, et ajuste l’heure si nécessaire. 

Les hôtes Azure sont synchronisés avec les serveurs de temps internes Microsoft qui extraient l’heure depuis des appareils de couche 1 appartenant à Microsoft, grâce à des antennes GPS. Les machines virtuelles dans Azure peuvent dépendre de leur hôte pour transférer l’heure précise (*heure de l’hôte*) sur la machine virtuelle. La machine virtuelle peut également obtenir directement l’heure à partir d’un serveur de temps ou par une combinaison des deux. 

Sur un matériel autonome, le système d’exploitation Linux lit uniquement l’horloge du matériel hôte au démarrage. Ensuite, l’horloge est gérée à l’aide de la minuterie d’interruption dans le noyau Linux. Dans cette configuration, l’horloge présente un décalage au fil du temps. Dans les nouvelles distributions Linux sur Azure, les machines virtuelles peuvent utiliser le fournisseur VMICTimeSync, inclus dans les services d’intégration Linux (LIS), pour demander des mises à jour de l’horloge à l’hôte plus fréquemment.

Les interactions de la machine virtuelle avec l’hôte peuvent également influer sur l’horloge. Au cours de la [maintenance avec préservation de la mémoire](../maintenance-and-updates.md#maintenance-that-doesnt-require-a-reboot), les machines virtuelles sont interrompues jusqu’à 30 secondes. Par exemple, avant le début de la maintenance, l’horloge de la machine virtuelle affiche 10:00:00 pendant 28 secondes. Une fois que la machine virtuelle reprend, l’horloge affiche toujours 10:00:00 et est donc décalée de 28 secondes. Pour corriger cela, le service VMICTimeSync surveille ce qu’il se passe sur l’hôte et demande des modifications sur les machines virtuelles pour compenser.

Si la synchronisation de l’heure ne fonctionne pas, l’horloge de la machine virtuelle accumule les erreurs. S’il n’y a qu’une seule machine virtuelle, l’effet n’est pas forcément significatif, sauf si la charge de travail nécessite une synchronisation extrêmement précise de l’horloge. Mais dans la plupart des cas, nous possédons plusieurs machines virtuelles interconnectées qui utilisent l’heure pour suivre les transactions. L’heure doit donc être cohérente tout au long du déploiement. Lorsque l’heure est différente d’une machine virtuelle à l’autre, vous pouvez rencontrer les effets suivants :

- L’authentification échoue. Les protocoles de sécurité, comme Kerberos, ou les technologies dépendantes d’un certificat reposent sur la précision de l’heure entre les systèmes.
- Il est très difficile de déterminer ce qu’il peut se passer dans un système si l’heure des journaux d’activité (ou d’autres données) diffère. Un même événement peut avoir l’air de s’être produit à différents moments, rendant la corrélation difficile.
- Si l’horloge est désactivée, la facturation peut être calculée de manière incorrecte.



## <a name="configuration-options"></a>Options de configuration

Il existe généralement trois façons de configurer la synchronisation de l’heure pour vos machines virtuelles Linux hébergées dans Azure :

- La configuration par défaut pour les images de la place de marché Microsoft Azure utilise à la fois l’heure NTP et l’heure de l’hôte VMICTimeSync. 
- Hôte uniquement à l’aide de VMICTimeSync.
- Utilisez un autre serveur de temps externe, avec ou sans l’heure de l’hôte VMICTimeSync.


### <a name="use-the-default"></a>Utiliser la valeur par défaut

Par défaut, la plupart des images de la place de marché Azure pour Linux sont configurés pour la synchronisation à partir de deux sources : 

- NTP comme source principale, qui obtient l’heure à partir d’un serveur NTP. Par exemple, les images de la place de marché Ubuntu 16.04 LTS utilisent **ntp.ubuntu.com**.
- Le service VMICTimeSync, utilisé comme source secondaire pour communiquer l’heure de l’hôte aux machines virtuelles et apporter des corrections une fois que la machine virtuelle est interrompue à des fins de maintenance. Les hôtes Azure utilisent des appareils de couche 1 appartenant à Microsoft pour maintenir une heure précise.

Dans les nouvelles distributions Linux, le service VMICTimeSync fournit une source d’horloge matérielle Precision Time Protocol (PTP), mais les distributions antérieures ne fournissent pas forcément cette source d’horloge matérielle et ont recours à NTP pour obtenir l’heure à partir de l’hôte.

Pour vérifier la synchronisation correcte de NTP, exécutez la commande `ntpq -p`.

### <a name="host-only"></a>Hôte uniquement 

Étant donné que les serveurs NTP comme time.windows.com et ntp.ubuntu.com sont publics, la synchronisation de l’heure avec eux requiert d’envoyer le trafic sur Internet. Des retards de paquets variables peuvent nuire à la qualité de la synchronisation de l’heure. La suppression de NTP en basculant vers la synchronisation avec l’hôte uniquement peut parfois améliorer les résultats de synchronisation de l’heure.

Le passage à la synchronisation de l’heure avec l’hôte uniquement est judicieux si vous rencontrez des problèmes de synchronisation de l’heure avec la configuration par défaut. Essayez la synchronisation avec l’hôte uniquement pour voir si cela améliore la synchronisation de l’heure sur la machine virtuelle. 

### <a name="external-time-server"></a>Serveur d’heure externe

Si vous avez des exigences spécifiques concernant la synchronisation de l’heure, vous pouvez également utiliser des serveurs de temps externes. Les serveurs de temps externes peuvent fournir une heure spécifique, ce qui s’avère parfois utile pour les scénarios de test. Ils garantissent l’uniformité de l’heure avec les machines hébergées dans des centres de données non Microsoft ou gèrent les secondes intercalaires d’une façon particulière.

Vous pouvez combiner un serveur de temps externe avec le service VMICTimeSync pour fournir des résultats semblables à la configuration par défaut. La combinaison d’un serveur de temps externe avec VMICTimeSync est la meilleure option pour résoudre des problèmes éventuels lorsque les machines virtuelles sont suspendues pour maintenance. 

## <a name="tools-and-resources"></a>Outils et ressources

Il existe des commandes de base pour la vérification de la configuration de synchronisation de l’heure. La documentation sur la distribution de Linux offre plus de détails sur la meilleure façon de configurer la synchronisation de l’heure pour cette distribution.

### <a name="integration-services"></a>Service d’intégration

Vérifiez si le service d’intégration (hv_utils) est chargé.

```bash
lsmod | grep hv_utils
```
Vous devez voir quelque chose de semblable à ceci :

```
hv_utils               24418  0
hv_vmbus              397185  7 hv_balloon,hyperv_keyboard,hv_netvsc,hid_hyperv,hv_utils,hyperv_fb,hv_storvsc
```

Vérifiez que si le démon de services d’intégration Hyper-V est en cours d’exécution.

```bash
ps -ef | grep hv
```

Vous devez voir quelque chose de semblable à ceci :

```
root        229      2  0 17:52 ?        00:00:00 [hv_vmbus_con]
root        391      2  0 17:52 ?        00:00:00 [hv_balloon]
```


### <a name="check-for-ptp-clock-source"></a>Vérifier la source de l’horloge PTP

Avec les versions plus récentes de Linux, une source d’horloge de protocole de temps de précision (PTP) est disponible avec le fournisseur VMICTimeSync. Dans les versions antérieures de Red Hat Enterprise Linux ou CentOS 7.x, les [service d’intégration Linux](https://github.com/LIS/lis-next) peuvent être téléchargé et utilisés pour installer le pilote mis à jour. Lorsque la source de l’horloge PTP est disponible, le périphérique Linux se présente sous la forme /dev/ptp*x*. 

Consultez les sources d’horloge PTP disponibles.

```bash
ls /sys/class/ptp
```

Dans cet exemple, la valeur renvoyée est *ptp0*, ce qui permet de vérifier le nom de l’horloge. Pour vérifier l’appareil, consultez le nom de l’horloge.

```bash
cat /sys/class/ptp/ptp0/clock_name
```

Le résultat renvoyé doit être `hyperv`.

### <a name="chrony"></a>chrony

Sur Ubuntu 19.10 et versions ultérieures, Red Hat Enterprise Linux et CentOS 8.x, [chrony](https://chrony.tuxfamily.org/) est configuré pour utiliser une horloge de source PTP. Au lieu de Chrony, les versions antérieures de Linux utilisent le démon ntpd (Network Time Protocol Daemon), qui ne prend pas en charge les sources PTP. Pour activer PTP dans ces versions, chrony doit être installé et configuré manuellement (dans chrony.conf) à l’aide du code suivant :

```bash
refclock PHC /dev/ptp0 poll 3 dpoll -2 offset 0
```

Pour plus d’informations sur Ubuntu et NTP, consultez [Synchronisation temporelle](https://ubuntu.com/server/docs/network-ntp).

Pour plus d’informations sur Red Hat et NTP, consultez [Configurer NTP](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_ntpd#s1-Configure_NTP). 

Pour plus d’informations sur chrony, consultez [Utilisation de chrony](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_the_chrony_suite#sect-Using_chrony).

Si les sources chrony et VMICTimeSync sont activées simultanément, vous pouvez en marquer une comme **favorite**, l’autre devenant alors la source de secours. Étant donné que les services NTP ne mettent pas à jour l’horloge pour les grands décalages, sauf après une longue période, la source VMICTimeSync permet de récupérer l’horloge à partir d’événements de machine virtuelle en pause beaucoup plus rapidement que les outils NTP seuls.

Par défaut, chronyd accélère ou ralentit l’horloge système pour corriger toute dérive temporelle. En cas de dérive trop importante, chrony n’est pas capable de la résoudre. Pour surmonter cela, le paramètre `makestep` dans **/etc/chrony.conf** peut être modifié pour forcer une synchronisation de l’heure si la dérive dépasse le seuil spécifié.

 ```bash
makestep 1.0 -1
```

Ici, chrony force une mise à jour temporelle si la dérive est supérieure à 1 seconde. Pour appliquer les modifications, redémarrez le service chronyd :

```bash
systemctl restart chronyd
```

### <a name="systemd"></a>systemd 

Sur les versions SUSE et Ubuntu antérieures à 19.10, la synchronisation de l’heure est configurée à l’aide de [systemd](https://www.freedesktop.org/wiki/Software/systemd/). Pour plus d’informations sur Ubuntu, consultez [Synchronisation de l’heure](https://help.ubuntu.com/lts/serverguide/NTP.html). Pour plus d’informations sur SUSE, consultez la section 4.5.8 des [notes de publication de SUSE Linux Enterprise Server 12 SP3](https://www.suse.com/releasenotes/x86_64/SUSE-SLES/12-SP3/#InfraPackArch.ArchIndependent.SystemsManagement).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez la rubrique [Précision de l’heure sur Windows Server 2016](/windows-server/networking/windows-time-service/accurate-time).


