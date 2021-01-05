---
title: Solution VMware Monitoring dans Azure Monitor | Microsoft Docs
description: Découvrez comment la solution de supervision VMware peut vous aider à gérer les journaux d’activité et à surveiller les hôtes ESXi.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/04/2018
ms.openlocfilehash: be50deb836082354db899e84ef24d75c4d403432
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91450391"
---
# <a name="vmware-monitoring-deprecated-solution-in-azure-monitor"></a>Solution VMware Monitoring (dépréciée) dans Azure Monitor

![Symbole VMware](./media/vmware/vmware-symbol.png)

> [!NOTE]
> La solution de supervision VMware est désormais dépréciée.  Les clients qui ont déjà installé la solution VMware Monitoring peuvent continuer de l’utiliser, mais ils ne peuvent pas l’ajouter à de nouveaux espaces de travail.

VMware Monitoring dans Azure Monitor est une solution qui vous permet de créer une approche de journalisation et de supervision centralisée pour les journaux VMware volumineux. Cet article décrit comment dépanner, capturer et gérer les hôtes ESXi dans un emplacement unique à l’aide de la solution. La solution vous permet de consulter des données détaillées pour tous vos hôtes ESXi dans un emplacement unique. Vous pouvez voir le nombre, l’état et les tendances des principaux événements des hôtes de machine virtuelle et ESXi, fournis via les journaux d’activité d’hôte ESXi. Vous pouvez résoudre des problèmes en consultant des journaux d’activité d’hôte ESXi centralisés et en y effectuant des recherches. Et vous pouvez créer des alertes basées sur des requêtes de recherche de journal.

La solution utilise la fonctionnalité syslog native de l’hôte ESXi pour transmettre des données à une machine virtuelle cible, qui dispose de l’agent Log Analytics. Toutefois, la solution n’écrit pas de fichiers dans syslog sur la machine virtuelle cible. L’agent Log Analytics ouvre le port 1514 et l’écoute. Une fois les données reçues, l’agent Log Analytics les envoie (par push) dans Azure Monitor.

## <a name="install-and-configure-the-solution"></a>Installer et configurer la solution
Utilisez les informations suivantes pour installer et configurer la solution.

* Ajoutez la solution VMware Monitoring à votre abonnement en suivant la procédure décrite dans [Installer une solution de supervision](./solutions.md#install-a-monitoring-solution).

#### <a name="supported-vmware-esxi-hosts"></a>Hôtes VMware ESXi pris en charge
vSphere ESXi Host 5.5, 6.0 et 6.5

#### <a name="prepare-a-linux-server"></a>Préparer un serveur Linux
Créez une machine virtuelle de système d’exploitation Linux pour recevoir toutes les données Syslog des hôtes ESXi. [L’agent Log Analytics Linux](../learn/quick-collect-linux-computer.md) est le point de regroupement de toutes les données syslog de l’hôte ESXi. Vous pouvez utiliser plusieurs hôtes ESXi pour transférer des journaux d’activité à un seul serveur Linux, comme dans l’exemple suivant.

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]  

   ![Flux Syslog](./media/vmware/diagram.png)

### <a name="configure-syslog-collection"></a>Configurer la collecte Syslog
1. Configurez le transfert de Syslog à VSphere. Pour plus d’informations sur la configuration du transfert de Syslog, consultez [Configuring syslog on ESXi 5.0 and higher (2003322)](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2003322) (Configuration de Syslog sur ESXi 5.0 et versions ultérieures (2003322)). Accédez à **Configuration de l’hôte ESXi** > **Logiciel** > **Paramètres avancés** > **Syslog**.
   ![vsphereconfig](./media/vmware/vsphere1.png)  
1. Dans le champ *Syslog.global.logHost*, ajoutez votre serveur Linux et le numéro de port *1514*. Par exemple, `tcp://hostname:1514` ou `tcp://123.456.789.101:1514`.
1. Ouvrez le pare-feu d’hôte ESXi pour Syslog. **Configuration de l’hôte ESXi** > **Logiciels** > **Profil de sécurité** > **Pare-feu**, puis ouvrez **Propriétés**.  

    ![vspherefw](./media/vmware/vsphere2.png)  

    ![vspherefwproperties](./media/vmware/vsphere3.png)  
1. Vérifiez sur la Console vSphere que ce Syslog est correctement configuré. Vérifiez sur l’hôte ESXI que le port **1514** est configuré.
1. Téléchargez et installez l’agent Log Analytics pour Linux sur le serveur Linux. Pour plus d’informations, consultez la [documentation de l’agent Log Analytics pour Linux](https://github.com/Microsoft/OMS-Agent-for-Linux).
1. Une fois l’agent Log Analytics pour Linux installé, accédez au répertoire /etc/opt/microsoft/omsagent/sysconf/omsagent.d, copiez le fichier vmware_esxi.conf dans le répertoire /etc/opt/microsoft/omsagent/conf/omsagent.d, puis modifiez le propriétaire/groupe et les autorisations du fichier. Par exemple :

    ```
    sudo cp /etc/opt/microsoft/omsagent/sysconf/omsagent.d/vmware_esxi.conf /etc/opt/microsoft/omsagent/conf/omsagent.d
   sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/conf/omsagent.d/vmware_esxi.conf
    ```
1. Redémarrez l’agent Log Analytics pour Linux en exécutant `sudo /opt/microsoft/omsagent/bin/service_control restart`.
1. Testez la connectivité entre le serveur Linux et de l’hôte ESXi en utilisant la commande `nc`sur l’hôte ESXi. Par exemple :

    ```
    [root@ESXiHost:~] nc -z 123.456.789.101 1514
    Connection to 123.456.789.101 1514 port [tcp/*] succeeded!
    ```

1. Dans le portail Azure, effectuez une requête de journal pour `VMware_CL`. Quand Azure Monitor collecte les données Syslog, il conserve le format Syslog. Dans le portail, certains des champs sont capturés, tel que *Hostname* et *ProcessName*.  

    ![La capture d’écran affiche une requête de journal pour Type = VMware_CL avec un résultat horodaté.](./media/vmware/type.png)  

    Si vos résultats de recherche de vue de journal d’activité sont similaires à l’image ci-dessus, vous êtes prêt à utiliser le tableau de bord de la solution de supervision VMware.  

## <a name="vmware-data-collection-details"></a>Détails sur la collecte de données Linux
La solution de supervision VMware collecte diverses mesures de performances et données de journaux à partir des hôtes ESXi à l’aide des agents Log Analytics pour Linux que vous avez activés.

Le tableau suivant présente les méthodes de collecte des données et d’autres informations sur le mode de collecte.

| plateforme | Agent Log Analytics pour Linux | Agent System Center Operations Manager | Stockage Azure | Operations Manager requis ? | Données de l’agent Operations Manager envoyées via un groupe d’administration | Fréquence de collecte |
| --- | --- | --- | --- | --- | --- | --- |
| Linux |&#8226; |  |  |  |  |Toutes les 3 minutes. |

Le tableau suivant affiche des exemples de champs de données collectés par la solution de supervision VMware :

| Nom du champ | description |
| --- | --- |
| Device_s |appareils de stockage VMware |
| ESXIFailure_s |types d’échec |
| EventTime_t |heure à laquelle l’événement s’est produit |
| HostName_s |nom d’hôte ESXi |
| Operation_s |créer ou supprimer une machine virtuelle |
| ProcessName_s |nom de l’événement |
| ResourceId_s |nom de l’hôte VMware |
| ResourceLocation_s |VMware |
| ResourceName_s |VMware |
| ResourceType_s |Hyper-V |
| SCSIStatus_s |Statut SCSI de VMware |
| SyslogMessage_s |données Syslog |
| UserName_s |utilisateur qui a créé ou supprimé la machine virtuelle |
| VMName_s |nom de la machine virtuelle |
| Computer |ordinateur hôte |
| TimeGenerated |heure à laquelle les données ont été générées |
| DataCenter_s |centre de données VMware |
| StorageLatency_s |latence de stockage (ms) |

## <a name="vmware-monitoring-solution-overview"></a>Présentation de la solution de supervision VMware
La vignette VMware s’affiche dans votre espace de travail Log Analytics. Elle fournit une vue d’ensemble des erreurs. Lorsque vous cliquez sur la vignette, vous accédez à l’affichage du tableau de bord.

![La capture d’écran montre la vignette VMware, affichant neuf échecs.](./media/vmware/tile.png)

#### <a name="navigate-the-dashboard-view"></a>Accédez à l’affichage du tableau de bord
Dans l’affichage du tableau de bord **VMware**, les panneaux sont organisés par :

* Nombre d’états d’échec
* Hôte principal par nombre d’événements
* Nombres d’événements principaux
* Activités de machine virtuelle
* Événements de disque de l’hôte ESXi

![solution1](./media/vmware/solutionview1-1.png)

![solution2](./media/vmware/solutionview1-2.png)

Cliquez sur n’importe quel panneau pour ouvrir le volet de recherche de Log Analytics qui affiche des informations détaillées spécifiques pour le panneau.

À ce stade, vous pouvez modifier la requête de journal pour l’adapter à un aspect spécifique. Pour plus d’informations sur la création de requêtes de journal, consultez [Rechercher des données à l’aide de requêtes de journal dans Azure Monitor](../log-query/log-query-overview.md).

#### <a name="find-esxi-host-events"></a>Rechercher des événements de l’hôte ESXi
Un seul hôte ESXi génère plusieurs journaux d’activité basés sur leurs processus. La solution de supervision VMware les centralise et résume les nombres d’événements. Cette vue centralisée vous permet de comprendre quel hôte ESXi a un volume élevé d’événements ainsi que les événements qui se produisent le plus fréquemment dans votre environnement.

![événement](./media/vmware/events.png)

Vous pouvez approfondir davantage en cliquant sur un hôte ESXi ou un type d’événement.

Lorsque vous cliquez sur un nom d’hôte ESXi, vous voyez les informations de cet hôte ESXi. Si vous souhaitez affiner des résultats avec le type d’événement, ajoutez `“ProcessName_s=EVENT TYPE”` à votre requête de recherche. Vous pouvez sélectionner **ProcessName** dans le filtre de recherche. Cela restreint les informations pour vous.

![Capture d’écran des panneaux ESXi Host Per Event Count et Breakdown Per Event Type dans l’affichage du tableau de bord VMware Monitoring.](./media/vmware/eventhostdrilldown.png)

#### <a name="find-high-vm-activities"></a>Rechercher les activités de machine virtuelle élevées
Une machine virtuelle peut être créée et supprimée sur tout hôte ESXi. Il est utile pour un administrateur d’identifier le nombre de machines virtuelles que crée un hôte ESXi. Cela aide ensuite à comprendre la planification des performances et de la capacité. Il est essentiel de suivre les événements d’activité de machine virtuelle lors de la gestion de votre environnement.

![Capture d’écran du panneau Activités de machine virtuelle dans le tableau de bord VMware Monitoring, montrant un graphique de création et suppression de machine virtuelle par l’hôte ESXi.](./media/vmware/vmactivities1.png)

Si vous souhaitez voir d’autres données de création de machine virtuelle hôte ESXi, cliquez sur un nom d’hôte ESXi.

![Capture d’écran d’un volet du tableau de bord VMware Monitoring montrant un tableau avec une ligne de données pour chaque création de machine virtuelle par un hôte ESXi.](./media/vmware/createvm.png)

#### <a name="common-log-queries"></a>Requêtes de journal courantes
La solution inclut d’autres requêtes utiles qui peuvent vous aider à gérer vos hôtes ESXi, telles que l’espace de stockage élevé, la latence du stockage et la défaillance de chemin.

![La capture d’écran montre RECHERCHES RECOMMANDÉES, qui sont des requêtes stockées utiles.](./media/vmware/queries.png)


#### <a name="save-queries"></a>Enregistrer des requêtes
L’enregistrement de requêtes de journal est une fonctionnalité standard dans Azure Monitor ; elle peut vous aider à conserver toutes les requêtes que vous avez trouvées utiles. Après avoir créé une requête que vous trouvez utile, enregistrez-la en cliquant sur **Favorites**. Vous pouvez réutiliser facilement une requête enregistrée à partir de la page [Mon tableau de bord](../learn/tutorial-logs-dashboards.md) dans laquelle vous pouvez créer vos propres tableaux de bord personnalisés.

![La capture d’écran montre une partie d’un tableau de bord personnalisé nommé Recherche dans les journaux avec des icônes pour les options Annuler, Exporter, Alertes, Enregistrer, Favoris et Historique.](./media/vmware/dockerdashboardview.png)

#### <a name="create-alerts-from-queries"></a>Créer des alertes à partir de requêtes
Après avoir créé vos requêtes, vous pouvez les utiliser pour vous avertir quand des événements spécifiques se produisent. Pour plus d’informations sur la création d’alertes, voir [Alertes dans Log Analytics](../platform/alerts-overview.md). Pour obtenir des exemples de requêtes d’alerte et d’autres requêtes, voir le billet de blog [Monitor VMware using Log Analytics](/archive/blogs/msoms/monitor-vmware-using-oms-log-analytics) (Analyser VMware à l’aide Log Analytics).

## <a name="frequently-asked-questions"></a>Forum aux questions
### <a name="what-do-i-need-to-do-on-the-esxi-host-setting-what-impact-will-it-have-on-my-current-environment"></a>Que dois-je faire avec les paramètres d’hôte ESXi ? Quel sera l’impact sur mon environnement actuel ?
La solution utilise le mécanisme de transfert syslog natif de l’hôte ESXi. Vous n’avez pas besoin d’autres logiciels Microsoft sur l’hôte ESXi pour capturer les journaux d’activité. L’impact sur votre environnement existant devrait être faible. Toutefois, vous devez absolument définir le transfert syslog, une fonctionnalité d’ESXI.

### <a name="do-i-need-to-restart-my-esxi-host"></a>Dois-je redémarrer mon hôte ESXi ?
Non. Ce processus ne nécessite pas de redémarrage. Parfois, vSphere n’actualise pas correctement syslog. Dans ce cas, ouvrez une session sur l’hôte ESXi et rechargez le journal système. Encore une fois, il n’est pas obligatoire de redémarrer l’hôte. Ainsi, ce processus ne perturbe pas votre environnement.

### <a name="can-i-increase-or-decrease-the-volume-of-log-data-sent-to-log-analytics"></a>Puis-je augmenter ou diminuer le volume de données de journal envoyé à Log Analytics ?
Oui, vous pouvez. Vous pouvez utiliser les paramètres de niveau de journal hôte ESXi dans vSphere. La collecte des journaux est basée sur le niveau *info*. Ainsi, si vous souhaitez vérifier la création ou la suppression de machines virtuelles, vous devez conserver le niveau *info* sur Hostd. Pour plus d’informations, consultez la [Base de connaissances VMware](https://kb.vmware.com/selfservice/microsites/search.do?&cmd=displayKC&externalId=1017658).

### <a name="why-is-hostd-not-providing-data-to-log-analytics-my-log-setting-is-set-to-info"></a>Pourquoi Hostd ne fournit-il pas de données à Log Analytics ? Mon journal est défini sur info.
Il y a un problème avec l’hôte ESXi pour l’horodatage de syslog. Pour plus d’informations, consultez la [Base de connaissances VMware](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2111202). Après avoir appliqué la solution de contournement, Hostd devrait fonctionner normalement.

### <a name="can-i-have-multiple-esxi-hosts-forwarding-syslog-data-to-a-single-vm-with-omsagent"></a>Est-il possible que plusieurs hôtes ESXi transfèrent les données de syslog vers une seule machine virtuelle avec omsagent ?
Oui. Plusieurs hôtes ESXi peuvent transférer des données vers une seule machine virtuelle avec omsagent.

### <a name="why-dont-i-see-data-flowing-into-log-analytics"></a>Pourquoi ne puis-je pas voir les données échangées avec Log Analytics ?
Il peut y avoir plusieurs raisons :

* L’hôte ESXi ne transmet pas correctement les données vers la machine virtuelle qui exécute omsagent. Pour tester cela, procédez comme suit :

  1. Pour vérifier cela, ouvrez une session sur l’hôte ESXi à l’aide de ssh et exécutez la commande suivante : `nc -z ipaddressofVM 1514`

      Si cela n’a pas réussi, les paramètres de vSphere dans la Configuration avancée ne sont probablement pas corrects. Consultez [Configurer la collecte Syslog](#configure-syslog-collection) pour plus d’informations sur la configuration de l’hôte ESXi pour le transfert de syslog.
  1. Si la connectivité du port syslog est correcte, mais que vous ne voyez toujours pas de données, rechargez syslog sur l’hôte ESXi à l’aide de ssh pour exécuter la commande suivante : `esxcli system syslog reload`
* La machine virtuelle avec l’agent Log Analytics n’est pas configurée correctement. Pour tester cela, procédez comme suit :

  1. Log Analytics écoute le port 1514. Pour vérifier qu’il est ouvert, utilisez la commande suivante : `netstat -a | grep 1514`
  1. Vous devez voir le port `1514/tcp` ouvert. Si ce n’est pas le cas, vérifiez qu’omsagent est correctement installé. Si vous ne voyez pas les informations de port, le port syslog n’est pas ouvert sur la machine virtuelle.

    a. Vérifiez que l’agent Log Analytics s’exécute à l’aide de `ps -ef | grep oms`. S’il n’est pas en cours d’exécution, démarrez le processus en exécutant la commande `sudo /opt/microsoft/omsagent/bin/service_control start`

     b. Ouvrez le fichier `/etc/opt/microsoft/omsagent/conf/omsagent.d/vmware_esxi.conf` .

     c. Vérifiez que les paramètres de groupe et d’utilisateur sont valides, comme dans : `-rw-r--r-- 1 omsagent omiusers 677 Sep 20 16:46 vmware_esxi.conf`

     d. Si le fichier n’existe pas ou que les paramètres de groupe et d’utilisateur sont incorrects, prenez une mesure corrective en [Préparant un serveur Linux](#prepare-a-linux-server).

## <a name="next-steps"></a>Étapes suivantes
* Utiliser des [requêtes de journal](../log-query/log-query-overview.md) dans Log Analytics pour afficher des données détaillées sur l’hôte VMware.
* [Créer vos propres tableaux de bord](../learn/tutorial-logs-dashboards.md) affichant des données de l’hôte VMware.
* [Créer des alertes](../platform/alerts-overview.md) lorsque des événements d’hôte VMware spécifiques se produisent.

