---
title: Résoudre les problèmes de processeur élevé de machines virtuelles Windows Azure
description: .
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: mnanda
manager: dcscontentpm
editor: v-jesits
tags: azure-resource-manager
ms.assetid: 9971cc26-916f-4e49-83cd-71eca74804f0
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 9/24/2020
ms.author: mnanda
ms.openlocfilehash: ffac5ac4d1a8143590e1d72aaafc8a02d6ab04ca
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91977253"
---
# <a name="troubleshoot-high-cpu-issues-for-azure-windows-virtual-machines"></a>Résoudre les problèmes de processeur élevé de machines virtuelles Windows Azure

## <a name="summary"></a>Résumé

Des problèmes de performances se produisent dans différents systèmes d’exploitation ou applications, et chaque problème requiert une approche unique pour le dépannage. La plupart de ces problèmes tournent autour du processeur, de la mémoire, de la mise en réseau et des entrées/sorties (E/S) en tant qu’emplacements clés où le problème se produit. Chacune de ces zones génère différents symptômes (parfois simultanément) et requiert un diagnostic et une solution différents.

Cet article décrit les problèmes de processeur élevé sur des machines virtuelles Windows Azure qui exécutent le système d’exploitation Windows.

### <a name="high-cpu-issues-on-azure-windows-vms"></a>Problèmes de processeur élevé sur des machines virtuelles Windows Azure

Outre les problèmes d’E/S et de latence du réseau, la résolution des problèmes liés au processeur et à la mémoire nécessite les mêmes outils et étapes que les serveurs locaux. L’un des outils pris en charge par Microsoft est généralement PerfInsights (disponible pour Windows et Linux). PerfInsights peut fournir un diagnostic des meilleures pratiques pour les machines virtuelles Azure dans un rapport convivial. PerfInsights est également un outil wrapper qui peut vous aider à collecter des données Perfmon, Xperf et Netmon en fonction des indicateurs sélectionnés dans l’outil.

La plupart de vos outils de résolution des problèmes de performances existants, tels que Perfmon ou Procmon, qui sont utilisés pour les serveurs locaux fonctionnent sur les machines virtuelles Windows Azure. Toutefois, PerfInsights est explicitement conçu pour les machines virtuelles Azure afin de fournir plus d’informations, notamment les meilleures pratiques Azure, les meilleures pratiques SQL, les graphiques de latence d’E/S haute résolution, les onglets de processeur et de mémoire, etc.

Qu’il s’exécute en mode utilisateur ou en mode noyau, les threads d’un processus actif requièrent des cycles de processeur pour exécuter le code à partir duquel il est créé. De nombreux problèmes sont directement liés à la charge de travail. Le type de charge de travail existant sur le serveur détermine la consommation des ressources, y compris le processeur.

#### <a name="common-factors"></a>Facteurs courants

Les facteurs suivants sont courants dans une situation de processeur élevé :

- Une modification ou un déploiement de code récent qui s’applique principalement aux applications telles que Internet Information Services (IIS), Microsoft SharePoint, Microsoft SQL Server ou des applications tierces.

- Une mise à jour récente qui peut être liée à une mise à jour au niveau du système d’exploitation ou à des mises à jour et des correctifs cumulatifs au niveau de l’application.

- Une modification de la requête ou des index obsolètes. Les applications du niveau de données SQL Server et Oracle ont également une optimisation du plan de requête comme autre facteur. Des modifications de données ou un manque d’index appropriés peuvent entraîner un plus grand nombre de requêtes.

- Spécifique à la machine virtuelle Azure. Certains processus, tels que RDAgent, et les processus spécifiques à l’extension, tels que l’agent d’analyse, l’agent MMA ou le client de sécurité, peuvent entraîner une consommation du processeur élevé. Ces processus doivent être consultés à partir d’un point de vue de la configuration ou des problèmes connus.

## <a name="troubleshoot-the-issue"></a>Résoudre le problème

Cet article se concentre sur l’isolation du processus problématique. Une analyse plus poussée sera spécifique au processus qui entraîne une consommation du processeur élevé.

Par exemple, si le processus est SQL Server (sqlservr.exe), les étapes suivantes vont consister à analyser la requête qui utilisait le plus de cycles processeur sur une période de temps spécifique.

### <a name="scope-the-issue"></a>Étendre le problème

Voici quelques questions à vous poser lors de la résolution du problème :

- Existe-t-il un modèle pour le problème ? Par exemple, le problème de processeur élevé se produit-il à un moment donné tous les jours, toutes les semaines ou tous les mois ? Si c’est le cas, pouvez-vous mettre en corrélation ce problème avec un travail, un rapport ou une connexion de l’utilisateur ?

- Le problème de processeur élevé est-il apparu après une modification récente de code ? Avez-vous appliqué une mise à jour dans Windows ou dans une application ?

- Le problème de processeur élevé est-il apparu après une modification de la charge de travail, comme une augmentation du nombre d’utilisateurs, un flux de données plus élevé ou un plus grand nombre de rapports ?

- Pour Azure, le problème de processeur élevé est-il apparu dans l’une des conditions suivantes ?

  - Après un redéploiement ou un redémarrage récent
  - Quand une référence SKU ou un type de machine virtuelle a changé
  - Quand une nouvelle extension a été ajoutée
  - Une fois les modifications de l’équilibreur de charge effectuées

### <a name="azure-caveats"></a>Mises en garde Azure

Comprenez votre charge de travail. Lorsque vous sélectionnez une machine virtuelle, vous pouvez estimer le nombre de processeurs virtuels lorsque vous examinez le coût global de l’hébergement mensuel. Si votre charge de travail est gourmande en ressources de calcul, la sélection d’une référence SKU de machine virtuelle plus petite avec un ou deux processeurs virtuels peut entraîner des problèmes de charge de travail. Testez différentes configurations pour votre charge de travail afin de déterminer la meilleure capacité de calcul requise.

Certaines séries de machines virtuelles, telles que les séries B (mode rafale), sont recommandées pour l’assurance qualité (AQ) et les tests. L’utilisation de ces séries dans l’environnement de production limite la capacité de calcul après épuisement des crédits du processeur.

Pour les applications connues comme SQL Server, Oracle, RDS (Remote Desktop Services - Services de bureau à distance), Windows Virtual Desktop, IIS ou SharePoint, il existe des articles sur les Meilleures pratiques intégrant des recommandations relatives à la configuration minimale pour ces charges de travail.

### <a name="ongoing-high-cpu-issues"></a>Problèmes de processeur élevé en cours

Si le problème se produit actuellement, il s’agit de la meilleure opportunité de capturer la trace du processus pour déterminer ce qui est à l’origine du problème. Vous pouvez utiliser les outils existants que vous utilisez pour les serveurs Windows locaux pour localiser le processus. Les outils suivants sont recommandés par Support Azure pour les machines virtuelles Azure.

### <a name="perfinsights"></a>PerfInsights

PerfInsights est l’outil recommandé de support Azure pour les problèmes de performances des machines virtuelles. Il est conçu pour couvrir les meilleures pratiques et les onglets d’analyse dédiés pour le processeur, la mémoire et les graphiques d’E/S haute résolution. Vous pouvez l’exécuter OnDemand à l’aide du Portail Azure ou à partir de la machine virtuelle. Vous pouvez partager les données avec l’équipe support Azure.

#### <a name="run-perfinsights"></a>Exécuter PerfInsights

PerfInsights est disponible sur les systèmes d’exploitation [Windows](./how-to-use-perfinsights.md) et [Linux](./how-to-use-perfinsights-linux.md). Pour Windows, les options proposées sont les suivantes.

#### <a name="run-and-analyze-reports-through-azure-portal"></a>Exécuter et analyser des rapports avec le Portail Azure

Lorsqu’il est [installé via le Portail Azure](./performance-diagnostics.md), il installe une extension sur la machine virtuelle. Les utilisateurs peuvent également installer PerfInsights en tant qu’extension en accédant directement aux [Extensions dans le panneau de la machine virtuelle](./performance-diagnostics-vm-extension.md), puis en choisissant une option Diagnostics des performances.

#### <a name="azure-portal-option-1"></a>Portail Azure Option 1

Parcourez le panneau de la machine virtuelle, puis sélectionnez l’option **Diagnostics de performances**. Vous devrez ensuite installer l’option (utilise des extensions) sur la machine virtuelle pour laquelle vous l’avez sélectionnée.

  ![Installer des diagnostics de performances](./media/troubleshoot-high-cpu-issues-azure-windows-vm/1-install-performance-diagnostics.png)

#### <a name="azure-portal-option-2"></a>Portail Azure Option 2

Accédez à **Diagnostiquer et résoudre les problèmes** dans le panneau de la machine virtuelle et recherchez **Problèmes de performances de la machine virtuelle**.

  ![Résoudre les problèmes de performance des machines virtuelles](./media/troubleshoot-high-cpu-issues-azure-windows-vm/2-troubleshoot-vm-performance-issues.png)

En sélectionnant **Résoudre**, l’écran d’installation de PerfInsights se charge.

En sélectionnant **Installer**, l’installation fournit différentes options de regroupement.

  ![Analyse des performances](./media/troubleshoot-high-cpu-issues-azure-windows-vm/3-performance-analysis.png)

Les options numérotées dans la capture d’écran sont associées aux commentaires suivants :

1. Pour l’option **Processeur élevé**, sélectionnez **Analyse des performances** ou **Avancé**.

2. Lorsque vous ajoutez des symptômes ici, ils sont ajoutés au rapport, ce qui vous permet de partager des informations avec Support Azure.

3. Sélectionnez la durée de la collecte des données. Pour l’option processeur élevé, sélectionnez au moins 15 minutes ou plus. En mode Portail Azure, vous pouvez collecter jusqu’à 15 minutes de données. Pour les périodes de collecte plus longues, vous devez exécuter le programme en tant qu’exécutable au sein de la machine virtuelle.

4. Si Support Azure vous demande de collecter ces données, vous pouvez ajouter le numéro de ticket ici. Ce champ est facultatif.

5. Sélectionnez ce champ pour accepter le Contrat de licence utilisateur final (CLUF).

6. Sélectionnez ce champ si vous envisagez de mettre ce rapport à la disposition de l’équipe Support Azure pour l’aider dans ce cas.

Le rapport est stocké sur l’un des comptes de stockage de votre abonnement. Vous pouvez les consulter et les télécharger ultérieurement.

#### <a name="run-perfinsights-from-within-the-vm"></a>Exécuter PerfInsights à partir de la machine virtuelle

Cette méthode peut être utilisée si vous avez l’intention d’exécuter PerfInsights pour des durées plus longues. L’[article PerfInsights](./how-to-use-perfinsights.md#how-do-i-run-perfinsights) fournit une procédure pas à pas détaillée des différentes commandes et indicateurs requis pour exécuter PerfInsights en tant qu’exécutable. Dans le cadre de l’utilisation du processeur élevé, vous aurez besoin de l’un des modes suivants :

- Scénario avancé

  - `PerfInsights /run advanced xp /d 300 /AcceptDisclaimerAndShareDiagnostics`

- Scénario de ralentissement des machines virtuelles (performances)

  - `PerfInsights /run vmslow /d 300 /AcceptDisclaimerAndShareDiagnostics /sa <StorageAccountName> /sk <StorageAccountKey>`

La sortie de la commande se trouve dans le même dossier que celui dans lequel vous avez enregistré l’exécutable PerfInsights.

#### <a name="what-to-look-for-in-the-report"></a>Éléments à rechercher dans le rapport

Après avoir exécuté le rapport, l’emplacement du contenu varie selon qu’il a été exécuté via le Portail Azure ou comme un fichier exécutable. Pour les deux options, accédez au dossier du journal généré ou téléchargez (si Portail Azure) localement pour l’analyse.

### <a name="run-through-the-azure-portal"></a>Exécuter par le biais du Portail Azure

  ![Diagnostics de performances à fort impact](./media/troubleshoot-high-cpu-issues-azure-windows-vm/4-high-impact-performance-diagnostics.png)

  ![Télécharger le rapport](./media/troubleshoot-high-cpu-issues-azure-windows-vm/5-download-report.png)

#### <a name="run-from-within-vm"></a>Exécuter à partir de la machine virtuelle

Votre structure de dossier doit ressembler aux images suivantes :

  ![Sélectionner une sortie](./media/troubleshoot-high-cpu-issues-azure-windows-vm/6-select-output.png)

  ![Structure de dossiers](./media/troubleshoot-high-cpu-issues-azure-windows-vm/7-folder-structure.png)

1. Toutes les collections supplémentaires, telles que Perfmon, Xperf, Netmon, les journaux SMB, les journaux des événements, etc., se trouvent dans le dossier Sortie.

1. Le rapport réel, ainsi que l’analyse et les recommandations.

1. Pour Performance (VMslow) et Advanced, le rapport collecte des informations **perfmon** pour la durée de l’exécution de PerfInsights.

1. Les journaux des événements affichent un aperçu rapide des détails de l’incident au niveau du système ou du processus.

#### <a name="where-to-start"></a>Où commencer

Ouvrez le rapport PerfInsights. L’onglet **Findings** enregistre les valeurs hors norme en termes de consommation des ressources. S’il existe des instances d’utilisation du processeur élevé, l’onglet **Findings** les catégorisera en tant qu’impact élevé ou impact moyen.

  ![Ressources de niveau d’impact](./media/troubleshoot-high-cpu-issues-azure-windows-vm/8-impact-level-resources.png)

Comme dans l’exemple précédent, PerfInsights a été exécuté pendant 30 minutes. Pendant une demi-heure, le processus en surbrillance a épuisé le processeur du côté supérieur. Si le même processus s’exécutait pendant toute la durée de la collection, le niveau d’impact aurait été remplacé par **HIGH**.

Si vous développez l’événement **Findings** , vous verrez plusieurs détails clés. L’onglet répertorie les processus dans l’ordre décroissant par consommation **moyenne du processeur** et indique si le processus a été lié au système, à une application Microsoft (SQL, IIS) ou à un processus tiers.

#### <a name="more-details"></a>Détails supplémentaires

Nous disposons d’un sous-onglet dédié sous **CPU** qui peut être utilisé pour l’analyse détaillée des modèles, par cœur ou par processus.

L’onglet **Top CPU Consumers** (Les plus grands consommateurs de processeurs) contient deux sections d’intérêt distinctes et vous pouvez afficher les statistiques par processeur ici. La conception de l’application est souvent monothread ou épinglée à un seul processeur. Dans ce scénario, un ou plusieurs cœurs s’exécutent à 100 %, tandis que d’autres cœurs s’exécutent aux niveaux attendus. Ces scénarios sont plus complexes, car le processeur moyen sur le serveur semble s’exécuter comme prévu, mais les processus qui sont épinglés sur les cœurs qui ont une utilisation intensive seront plus lents que prévu.

  ![Utilisation du processeur élevé](./media/troubleshoot-high-cpu-issues-azure-windows-vm/9-high-cpu-usage.png)

La deuxième section (tout aussi importante) est **Top Long Running CPU Consumers** (les plus grands consommateurs longue durée de processeurs). Cette section présente les détails du processus et leur modèle d’utilisation de processeur. La liste est triée en utilisant des consommateurs de processeurs moyens élevés en haut.

  ![Les plus grands consommateurs longue durée de processeurs](./media/troubleshoot-high-cpu-issues-azure-windows-vm/10-top-long-running-cpu-consumers.png)

Ces deux onglets sont suffisants pour définir le chemin d’accès vers les étapes de dépannage suivantes. Selon le processus qui pilote la condition de processeur élevé, vous devez répondre aux questions qui ont été posées précédemment. Les processus tels que les SQL Server (sqlservr.exe) ou IIS (w3wp.exe) requièrent une exploration spécifique des modifications de requête ou de code qui sont à l’origine de cette condition. Pour les processus système tels que WMI ou Lsass.exe, vous devez suivre un chemin d’accès différent.

Pour les processus liés aux machines virtuelles Azure, tels que RDAgent, OMS et les exécutables d’extension de surveillance, vous devrez peut-être corriger une nouvelle build ou version en obtenant de l’aide auprès de l’équipe Support Azure.

### <a name="perfmon"></a>Perfmon

**Perfmon** est l’un des outils les plus récents pour résoudre un problème de ressource sur Windows Server. Il ne fournit pas de rapport clair contenant des recommandations ou des conclusions. Au lieu de cela, il demande à l’utilisateur d’explorer les données collectées et d’utiliser un filtre spécifique sous les différentes catégories de compteurs.

PerfInsights collecte Perfmon sous la forme d’un journal supplémentaire pour les scénarios VMSlow et avancés. Toutefois, les Perfmon peuvent être collectés indépendamment et présentent les avantages supplémentaires suivants :

- Il peut être collecté à distance.

- Il peut être planifié via des **Tâches**.

- Il peut être collecté pour des durées plus longues ou en mode continu en utilisant la fonctionnalité de basculement.

Prenons l’exemple illustré dans PerfInsights pour voir comment Perfmon affiche ces données. Les catégories de compteurs requises sont les suivantes :

- Processor Information > %Processor Time > _Total

- Process > %ProcessorTime > All Instances

#### <a name="where-to-start"></a>Où commencer

Les noms de fichiers de sortie de Perfmon ont une extension `.blg`. Vous pouvez collecter ces fichiers indépendamment ou en utilisant PerfInsights. Pour cette discussion, vous allez utiliser le `.blg` Perfmon qui est inclus dans les données de PerfInsights qui ont été collectées pour l’exemple précédent.

Aucun rapport d’utilisateur par défaut n’est disponible dans Perfmon. Il existe différents affichages qui modifient le type de graphique, mais la filtration de processus (ou le travail nécessaire pour identifier les processus de coupable) est manuelle.

> [!NOTE]
> L’outil de [liste d’accès à la publication](https://github.com/clinthuffman/PAL) peut consommer des fichiers `.blg` et générer des rapports détaillés.

Pour démarrer, sélectionnez la catégorie **Add Counters** (Ajouter des compteurs).

1. Sous **Available counters**  (Compteurs disponibles), sélectionnez le compteur **%ProcessorTime** dans la catégorie **Processor Information** (Informations sur le processeur).

1. Sélectionnez **_Total**, qui vous donne les statistiques de tous les cœurs combinés.

1. Sélectionnez **Ajouter**. La fenêtre affiche **%ProcessorTime** sous  **Added counters** (Compteurs ajoutés).

  ![Ajouter un temps de processeur](./media/troubleshoot-high-cpu-issues-azure-windows-vm/11-add-processor-time.png)

Une fois les compteurs chargés, vous verrez des graphiques en courbes dans le cadre de la durée de collection. Vous pouvez sélectionner ou effacer les compteurs. Jusqu’à présent, vous n’avez ajouté qu’un seul compteur.

  ![Paramètres Analyseur de performances](./media/troubleshoot-high-cpu-issues-azure-windows-vm/12-performance-monitor-1.png)

Chaque compteur aura une valeur **Moyenne**, **Minimale** et **Maximale**. Concentrez-vous sur les valeurs **Moyennes** et **Maximales**, car la valeur moyenne peut varier en fonction de la durée de la collecte de données. Si l’activité du processeur élevé a été observée pendant 10 minutes alors que l’ensemble de la collection était de 40 minutes, les valeurs moyennes seront beaucoup plus basses.

Le graphique de tendance précédent montre que le **Processeur total** était de près de 80 % pendant environ 15 minutes.

#### <a name="identify-the-process"></a>Identifier le processus

Nous avons identifié que le serveur avait une consommation du processeur élevé pendant un laps de temps spécifié, mais nous n’avons pas encore identifié le pilote. Contrairement à l’utilisation de PerfInsights, vous devez rechercher manuellement le processus coupable dans ce cas.

Pour cette tâche, vous devez effacer ou supprimer les compteurs **%ProcessorTime** précédemment ajoutés, puis ajouter une nouvelle catégorie :

- Process > %ProcessorTime > All Instances

Cette catégorie charge les compteurs pour tous les processus en cours d’exécution à ce moment-là.

  ![Ajouter des compteurs](./media/troubleshoot-high-cpu-issues-azure-windows-vm/13-add-counters.png)

Sur un ordinateur de production classique, des centaines de processus peuvent être en cours d’exécution. Par conséquent, il peut être nécessaire de supprimer tous les compteurs qui semblent avoir un graphique de tendance faible ou plat.

Pour accélérer ce processus, utilisez l'affichage **Histogramme** et remplacez le type d’affichage **Ligne** par **Histogramme**, ce qui vous donnera un graphique à barres. Vous trouverez qu’il est plus facile de choisir les processus qui subissent une utilisation élevée du processeur pendant le temps de collecte.

Étant donné qu’il y a toujours une barre pour **Total**, concentrez-vous sur les barres qui affichent un taux d’épuisement élevé. Vous pouvez supprimer les autres barres pour nettoyer l’affichage. À présent, revenez à l’affichage **Ligne**.

  ![Indicateurs de l’analyseur de performances](./media/troubleshoot-high-cpu-issues-azure-windows-vm/14-performance-monitor-2.png)

Il est désormais plus facile d’intercepter le processus coupable. Par défaut, les valeurs **Max** et **Min** sont des multiples du nombre de cœurs sur le serveur ou de threads du processus.

  ![Résultats de l’analyseur de performances](./media/troubleshoot-high-cpu-issues-azure-windows-vm/15-performance-monitor-3.png)

La liste des outils disponibles ne se termine pas par PerfInsights pour Perfmon. Vous avez accès à d’autres outils, tels que **ProcessMonitor** (ProcMon) ou **Xperf**. De nombreux outils tiers peuvent être utilisés en fonction des besoins.

### <a name="azure-monitoring-tools"></a>Outils de supervision Azure

Les machines virtuelles Azure ont des indicateurs de performance fiables qui incluent des informations de base telles que le processeurs, l’E/S réseau et les octets d’E/S. Pour les indicateurs de performance avancés, telles que Azure Monitor, vous devez effectuer uniquement quelques sélections pour configurer et utiliser un compte de stockage que vous spécifiez.

#### <a name="basic-default-counters"></a>Compteurs de base (par défaut)

  ![Indicateurs de performance du graphique](./media/troubleshoot-high-cpu-issues-azure-windows-vm/16-chart-metrics.png)

#### <a name="enabling-azure-monitor"></a>Activer Azure Monitor

Une fois que vous avez activé les indicateurs de performance Azure Monitor, le logiciel installe une extension sur la machine virtuelle, puis commence à collecter des mesures granulaires, y compris des compteurs Perfmon.

  ![Compte de stockage de diagnostics](./media/troubleshoot-high-cpu-issues-azure-windows-vm/17-diagnostics-storage-account.png)

Les catégories de compteurs **de base** sont définies **par défaut**. Toutefois, vous pouvez également définir une collection **Personnalisée**.

  ![Compteurs de performance](./media/troubleshoot-high-cpu-issues-azure-windows-vm/18-performance-counters.png)

Une fois les paramètres activés, vous pouvez afficher ces compteurs **Guest** (Invité) dans la section **Metrics** (Indicateurs de performance). Vous pouvez également définir des **Alerts** (Alertes) (y compris des messages électroniques) si les indicateurs de performance atteignent un certain seuil.

  ![Espace de noms des indicateurs de performance](./media/troubleshoot-high-cpu-issues-azure-windows-vm/19-metrics-namespace.png)

Pour plus d’informations sur l’utilisation d’Azure Monitor pour gérer des machines virtuelles Azure, consultez [Surveillance des machines virtuelles Azure avec Azure Monitor](../../azure-monitor/insights/monitor-vm-azure.md).

### <a name="reactive-troubleshooting"></a>Résolution des problèmes de réactivité

Si le problème s’est déjà produit, vous devez découvrir ce qui a provoqué le problème de processeur élevé en premier lieu. La position réactive peut être délicate. Le mode de collecte des données n’est pas aussi utile car le problème s’est déjà produit.

Si ce problème ne s’est produit qu’une seule fois, il peut être difficile de déterminer quelle application l’a provoquée. Si la machine virtuelle Azure a été configurée pour utiliser OMS ou un autre suivi des diagnostics, vous pouvez toujours obtenir des insights sur ce qui a provoqué le problème.

Si vous avez affaire à un modèle répétitif, collectez les données pendant que le problème est susceptible de se produire ensuite.

PerfInsights n’a pas encore de capacité **d’exécution planifiée**. Toutefois, Perfmon peut être exécuté et planifié par le biais de la ligne de commande.

### <a name="logman-command"></a>Commande Logman

La commande **Logman create Counter** est utilisée pour exécuter la collecte Perfmon via la ligne de commande, pour la planifier par le biais de **Task Manager**ou pour l’exécuter à distance.

**Échantillon** (comprend le mode de collecte à distance)

`Logman create counter LOGNAME -u DOMAIN\USERNAME * -f bincirc -v mmddhhmm -max 300 -c "\\SERVERNAME\LogicalDisk(*)\*" "\\SERVERNAME\Memory\*" "\\SERVERNAME\Network Interface(*)\*" "\\SERVERNAME\Paging File(*)\*" "\\SERVERNAME\PhysicalDisk(*)\*" "\\SERVERNAME\Process(*)\*" "\\SERVERNAME\Redirector\*" "\\SERVERNAME\Server\*" "\\SERVERNAME\System\*" "\\SERVERNAME\Terminal Services\*" "\\SERVERNAME\Processor(*)\*" "\\SERVERNAME\Cache\*" -si 00:01:00`

Logman.exe peut également être démarré à partir d’un ordinateur de machine virtuelle Azure homologue dans le même réseau virtuel.

Pour en savoir plus sur ces paramètres, consultez [Logman create Counter](/windows-server/administration/windows-commands/logman-create-counter).

Une fois les données Perfmon collectées pendant que le problème se produit, les étapes restantes pour analyser les données sont les mêmes que celles décrites précédemment.

## <a name="conclusion"></a>Conclusion

Pour tout problème de performances, la compréhension de votre charge de travail est essentielle pour résoudre le problème. Vous devez évaluer les options sur les différentes références SKU de machine virtuelle et les différentes options de stockage sur disque en conservant le focus sur la charge de travail de production. Le processus de test de solutions sur des machines virtuelles différentes peut vous aider à prendre la meilleure décision.

Étant donné que les opérations utilisateur et la quantité de données varient, conservez toujours une mémoire tampon dans les capacités de calcul, la mise en réseau et les E/S de la machine virtuelle. À présent, toute modification soudaine de la charge de travail n’a pas autant d’effet.

Si vous prévoyez une augmentation rapide de la charge de travail, passez à une référence SKU plus élevée qui dispose de davantage de puissance de calcul. Si la charge de travail est gourmande en ressources de calcul, choisissez les références SKU de machine virtuelle avec soin.