---
title: Comment créer des graphiques de performances avec Azure Monitor pour les machines virtuelles
description: Performances est une fonctionnalité d’Azure Monitor pour les machines virtuelles qui détecte automatiquement les composants d’application sur les systèmes Windows et Linux et mappe la communication entre les services. Cet article fournit des informations sur son utilisation dans divers scénarios.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/31/2020
ms.openlocfilehash: f9578fadfbe057b723af63e338bf8bda63cf6f21
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91330908"
---
# <a name="how-to-chart-performance-with-azure-monitor-for-vms"></a>Comment créer des graphiques de performances avec Azure Monitor pour les machines virtuelles

Azure Monitor pour les machines virtuelles comprend un ensemble de graphiques de performances qui ciblent divers indicateurs de performance clés (KPI) pour vous aider à déterminer l’intégrité du fonctionnement d’une machine virtuelle. Les graphiques illustrent l’utilisation des ressources sur une période de temps pour que vous puissiez identifier les goulots d’étranglement ou les anomalies. Vous pouvez également basculer sur une perspective répertoriant toutes les machines virtuelles pour afficher l’utilisation des ressources en fonction de la métrique sélectionnée. Il y a beaucoup d’autres éléments à prendre en compte dans la gestion des performances, mais la solution Azure Monitor pour machines virtuelles supervise des indicateurs de performances clés du système d’exploitation liés à l’utilisation du processeur, de la mémoire, de la carte réseau et du disque. La fonctionnalité Performances vient compléter celle de supervision de l’intégrité et permet d’exposer les problèmes indiquant une panne éventuelle d’un composant système, de prendre en charge les réglages et l’optimisation pour améliorer l’efficacité ou de prendre en charge la planification de la capacité.  

## <a name="limitations"></a>Limites
Voici les limitations de la collecte de performances avec Azure Monitor pour machines virtuelles.

- La **Mémoire disponible** n’est pas disponible pour les machines virtuelles exécutant Red Hat Linux (RHEL) 6. Cette mesure est calculée à partir de **MemAvailable** qui a été introduite dans la [version 3.14 du noyau](http://www.man7.org/linux/man-pages/man1/free.1.html).
- Les métriques sont uniquement disponibles pour les disques de données sur des machines virtuelles Linux utilisant la famille de systèmes de fichiers XFS ou EXT (EXT2, EXT3, EXT4).

## <a name="multi-vm-perspective-from-azure-monitor"></a>Perspective avec plusieurs machines virtuelles d’Azure Monitor

Dans Azure Monitor, la fonctionnalité Performances peut afficher l’ensemble des machines virtuelles supervisées déployées sur l’ensemble des groupes de travail de vos abonnements ou de votre environnement. Pour y accéder à partir d’Azure Monitor, procédez comme suit. 

1. Dans le Portail Azure, sélectionnez **Surveiller**. 
2. Choisissez **Machines virtuelles** dans la section **Solutions**.
3. Sélectionnez l’onglet **Performances**.

![Affichage Top N List (Liste N premiers) - Performances - Insights de machine virtuelle](media/vminsights-performance/vminsights-performance-aggview-01.png)

Dans l’onglet **Graphiques N premiers**, si vous avez plusieurs espaces de travail Log Analytics, choisissez celui qui est activé avec la solution à partir du sélecteur **Espace de travail** en haut de la page. Le sélecteur **Groupe** retourne les abonnements, les groupes de ressources, les [groupes d’ordinateurs](../platform/computer-groups.md) et les groupes de machines virtuelles identiques associés à l’espace de travail sélectionné que vous pouvez utiliser pour filtrer davantage les résultats présentés dans les graphiques de cette page et dans les autres pages. Votre sélection s’applique seulement à la fonctionnalité Performances, et n’est pas reportée sur Intégrité ou Carte.  

Par défaut, les graphiques montrent les dernières 24 heures. À l’aide du sélecteur **TimeRange**, vous pouvez interroger les intervalles de temps précédents (jusqu’à 30 jours) pour afficher l’état des performances passées.

Les cinq graphiques d’utilisation de capacité affichés sur la page sont :

* Pourcentage d’utilisation de l’UC : affiche les cinq premières machines avec la moyenne d’utilisation du processeur la plus élevée 
* Mémoire disponible : affiche les cinq premières machines avec la quantité moyenne de mémoire disponible la plus basse 
* Logical Disk Space Used % (% d’espace disque logique utilisé) : affiche les cinq premières machines avec le pourcentage moyen d’espace disque utilisé le plus élevé pour tous les volumes de disque 
* Bytes Sent Rate (Taux d’octets envoyés) : affiche les cinq premières machines avec la moyenne d’octets envoyés la plus élevée 
* Bytes Received Rate (Taux d’octets reçus) : affiche les cinq premières machines avec la moyenne d’octets reçus la plus élevée 

En cliquant sur l’icône d’épingle dans le coin supérieur droit de l’un des cinq graphiques, vous épinglez le graphique sélectionné au dernier tableau de bord Azure que vous avez consulté.  À partir du tableau de bord, vous pouvez redimensionner et repositionner le graphique. En sélectionnant le graphique depuis le tableau de bord, vous allez être redirigé vers Azure Monitor pour machines virtuelles, et charger la portée et la vue appropriées.  

Cliquez sur l’icône située à gauche de l’icône d’épingle, sur l’un des cinq graphiques, pour ouvrir la vue **Top N List** (Liste N premiers).  Ici, vous pouvez voir l’utilisation des ressources pour cette métrique de performances par machine virtuelle dans un affichage de liste, ainsi que la machine virtuelle avec les tendances d’utilisation les plus élevées.  

![Affichage Top N List (Liste N premiers) pour une métrique de performances sélectionnée](media/vminsights-performance/vminsights-performance-topnlist-01.png)

Lorsque vous cliquez sur la machine virtuelle, le volet **Propriétés** est développé à droite pour afficher les propriétés de chacun des éléments sélectionnés, telles que les informations système générées par le système d’exploitation, les propriétés de la machine virtuelle Azure, etc. Cliquer sur l’une des options sous la section **Liens rapides** permet de vous rediriger vers cette fonctionnalité, directement depuis la machine virtuelle sélectionnée.  

![Volet Propriétés de la machine virtuelle](./media/vminsights-performance/vminsights-properties-pane-01.png)

Basculez vers l’onglet **Aggregated Charts** (Graphiques agrégés) pour afficher les métriques de performances filtrées par moyenne ou centiles.  

![Affichage agrégé - Performances - Insights de machine virtuelle](./media/vminsights-performance/vminsights-performance-aggview-02.png)

Les graphiques d’utilisation de la capacité suivants sont fournis :

* CPU Utilization % (% d’utilisation de l’UC) : affiche par défaut la moyenne et le top 95e centile 
* Mémoire disponible : affiche par défaut la moyenne, le top 5e centile et le top 10e centile 
* Logical Disk Space Used % (% d’espace disque logique utilisé) : affiche par défaut la moyenne et le 95e centile 
* Bytes Sent Rate (Taux d’octets envoyés) : affiche par défaut la moyenne d’octets envoyés 
* Bytes Received Rate (Taux d’octets reçus) : affiche par défaut la moyenne d’octets reçus

Vous pouvez également modifier la granularité des graphiques dans l’intervalle de temps en sélectionnant les options du sélecteur de centiles **Moy**, **Min**, **Max**, **50th** (50e), **90e** et **95th** (95e).

Pour voir l’utilisation des ressources par machine virtuelle dans un affichage de liste et déterminer la machine avec les tendances d’utilisation les plus élevées, sélectionnez l’onglet **Top N List** (Liste N premiers).  La page **Top N List** (Liste N premiers) affiche les 20 premières machines triées par utilisation la plus élevée par 95e centile pour la métrique *CPU Utilization %* (% d’utilisation de l’UC).  Vous pouvez voir davantage de machines en sélectionnant **Charger plus** ; les résultats affichent maintenant les 500 premières machines. 

>[!NOTE]
>La liste ne peut pas afficher plus de 500 machines à la fois.  
>

![Exemple de page Top N List (Liste N premiers)](./media/vminsights-performance/vminsights-performance-topnlist-01.png)

Pour filtrer les résultats sur une machine virtuelle spécifique dans la liste, entrez le nom de l’ordinateur dans la zone de texte **Search by name** (Rechercher par nom).  

Si vous préférez afficher l’utilisation à partir d’une métrique de performances différente, dans la liste déroulante **Métrique**, sélectionnez **Mémoire disponible**, **Logical Disk Space Used %** (% d’espace disque logique utilisé), **Network Received Byte/s** (Octets réseau reçus/s) ou **Network Sent Byte/s** (Octets réseau envoyés/s). La liste se met à jour pour afficher l’utilisation en fonction de la métrique sélectionnée.  

Sélectionner une machine virtuelle dans la liste permet d’ouvrir le panneau **Propriétés** à droite de la page, à partir duquel vous pouvez sélectionner **Performance detail** (Détails des performances).  La page **Virtual Machine Detail** (Détails de la machine virtuelle) s’ouvre et affiche uniquement les informations relatives à cette machine virtuelle. Cela revient à accéder à la section de performances des insights de machine virtuelle directement à partir de la machine virtuelle Azure.  

## <a name="view-performance-directly-from-an-azure-vm"></a>Afficher les performances directement à partir d’une machine virtuelle Azure

Pour un accès direct à partir d’une machine virtuelle, procédez comme suit.

1. Dans le portail Azure, sélectionnez **Machines virtuelles**. 
2. Choisissez une machine virtuelle dans la liste, puis, dans la section **Surveillance**, sélectionnez **Insights**.  
3. Sélectionnez l’onglet **Performances**. 

En plus d’inclure des graphiques d’utilisation de performances, cette page fournit un tableau affichant pour chaque disque logique détecté, sa capacité, son utilisation et la moyenne totale pour chaque mesure.  

Les graphiques d’utilisation de la capacité suivants sont fournis :

* CPU Utilization % (% d’utilisation de l’UC) : affiche par défaut la moyenne et le top 95e centile 
* Mémoire disponible : affiche par défaut la moyenne, le top 5e centile et le top 10e centile 
* Logical Disk Space Used % (% d’espace disque logique utilisé) : affiche par défaut la moyenne et le 95e centile 
* Logical Disk IOPS (IOPS de disque logique) : affiche par défaut la moyenne et le 95e centile
* Logical Disk MB/s (Mo/s de disque logique) : affiche par défaut la moyenne et le 95e centile
* Max Logical Disk Used % (% de disque logique max. utilisé) : affiche par défaut la moyenne et le 95e centile
* Bytes Sent Rate (Taux d’octets envoyés) : affiche par défaut la moyenne d’octets envoyés 
* Bytes Received Rate (Taux d’octets reçus) : affiche par défaut la moyenne d’octets reçus

En cliquant sur l’icône d’épingle dans le coin supérieur droit de l’un des graphiques, vous épinglez le graphique sélectionné au dernier tableau de bord Azure que vous avez consulté. À partir du tableau de bord, vous pouvez redimensionner et repositionner le graphique. En sélectionnant le graphique du tableau de bord vous redirige vers Azure Monitor pour machines virtuelles et charge l’affichage des détails sur les performances pour la machine virtuelle.  

![Performances - Insights de machine virtuelle directement depuis l’affichage de la machine virtuelle](./media/vminsights-performance/vminsights-performance-directvm-01.png)

## <a name="view-performance-directly-from-an-azure-virtual-machine-scale-set"></a>Afficher les performances directement à partir d’un groupe de machines virtuelles identiques Azure

Pour un accès direct depuis un groupe de machines virtuelles identiques, procédez comme suit.

1. Dans le Portail Microsoft Azure, sélectionnez **Groupes de machines virtuelles identiques**.
2. Choisissez une machine virtuelle dans la liste, puis, dans la section **Surveillance**, sélectionnez **Insights** afin d’afficher l’onglet **Performance**.

Cette page charge l’affichage des performances Azure Monitor, limité au groupe identique sélectionné. Cela vous permet de voir les n principales instances dans le groupe identique sur l’ensemble des métriques surveillées, d’afficher les performances agrégées sur le groupe identique et de voir les tendances relatives aux métriques sélectionnées sur les instances individuelles et le groupe identique. La sélection d’une instance de la vue de liste vous permet de charger son mappage ou de naviguer dans une vue détaillée des performances pour cette instance.

En cliquant sur l’icône d’épingle dans le coin supérieur droit de l’un des graphiques, vous épinglez le graphique sélectionné au dernier tableau de bord Azure que vous avez consulté. À partir du tableau de bord, vous pouvez redimensionner et repositionner le graphique. En sélectionnant le graphique du tableau de bord vous redirige vers Azure Monitor pour machines virtuelles et charge l’affichage des détails sur les performances pour la machine virtuelle.  

![Afficher des performances associées aux insights de machine virtuelle directement à partir de la vue d’un groupe de machines virtuelles identiques Azure](./media/vminsights-performance/vminsights-performance-directvmss-01.png)

>[!NOTE]
>Vous pouvez également accéder à une vue détaillée des performances pour une instance spécifique à partir de la vue Instances de votre groupe identique. Accédez à **Instances** sous la section **Paramètres**, puis choisissez **Insights**.



## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment utiliser les [classeurs](vminsights-workbooks.md) fournis dans Azure Monitor pour machines virtuelles afin d’analyser plus en détail les métriques de performances et de réseau.  

- Pour en savoir plus sur les dépendances des applications détectées, consultez [View Azure Monitor for VMs Map](vminsights-maps.md) (Afficher la carte d’Azure Monitor pour machines virtuelles).
