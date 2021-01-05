---
title: Insights pour Azure Load Balancer
description: Utiliser les insights de l’équilibreur de charge pour obtenir une localisation rapide des erreurs et faire des choix de conception informés
services: load-balancer
documentationcenter: na
author: erichrt
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/27/2020
ms.author: errobin
ms.openlocfilehash: 63b91194c9ffb10fd8f4c5f1341eaf74bc81f5e1
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94694879"
---
# <a name="using-insights-to-monitor-and-configure-your-azure-load-balancer"></a>Utilisation des Insights pour surveiller et configurer votre Azure Load Balancer

Par le biais d’Azure Monitor pour réseaux, vous obtenez des visualisations des dépendances fonctionnelles et un tableau de bord de mesures préconfiguré pour vos équilibreurs de charge. Ces représentations visuelles vous aident à faire des choix de conception informés et à localiser, diagnostiquer et résoudre rapidement toutes les erreurs.

>[!NOTE] 
>Notez que cette fonctionnalité est en préversion, par conséquent il se peut que la vue des dépendances fonctionnelles et le tableau de bord préconfiguré changent pour améliorer cette expérience.

>[!IMPORTANT]
>Standard Load Balancer est requis pour afficher les mesures à partir l’espace de noms de l’équilibreur de charge dans le tableau de bord de mesures préconfiguré. Vous pourrez toujours voir les mesures à partir de la machine virtuelle, du groupe de machines virtuelles identiques et des espaces de noms du Moniteur de connexion. Nous vous recommandons toutefois de [mettre à niveau vers la version Standard](./upgrade-basic-standard.md) pour les charges de travail de production afin de tirer parti de l’important jeu de mesures d’Azure Load Balancer.

## <a name="functional-dependency-view"></a>Vue des dépendances fonctionnelles

Le vue des dépendances fonctionnelles vous permet de vous représenter les configurations d’équilibreur de charge les plus complexes. Grâce à une représentation visuelle sur votre dernière configuration de Load Balancer, vous pouvez effectuer des mises à jour tout en gardant à l’esprit votre configuration.

Vous pouvez accéder à cette vue en visitant le panneau Insights de votre ressource Load Balancer dans Azure.

:::image type="content" source="./media/load-balancer-insights/load-balancer-functional-dependency-visual.png" alt-text="Description de la vue des dépendances fonctionnelles. On voit que le serveur frontal de l’équilibreur de charge se connecte aux membres du pool principal via les règles configurées. Pour Standard Load Balancer, les lignes qui relient les règles d’équilibrage de charge et les instances du pool principal suivent un code de couleur en fonction de l’état de la sonde d’intégrité." border="true":::

Pour Standard Load Balancer, les ressources de votre pool principal suivent un code de couleur en fonction de l’état de la sonde d’intégrité, indiquant la disponibilité actuelle de votre pool principal pour traiter le trafic. Parallèlement à la topologie ci-dessus, un graphique temporel de l’état d’intégrité vous est présenté, offrant une vue instantanée de l’intégrité de votre application.

## <a name="metrics-dashboard"></a>Tableau de bord des métriques

Dans le panneau Insights de votre équilibreur de charge, vous pouvez sélectionner l’option Mesures plus détaillées pour afficher un [classeur Azure Monitor](../azure-monitor/platform/workbooks-overview.md) préconfiguré contenant des représentations visuelles des mesures en rapport avec des aspects spécifiques de votre équilibreur de charge. Ce tableau de bord présente l’état de l’équilibreur de charge et des liens vers la documentation relative en haut de la page.

Dans un premier temps, l’onglet Vue d’ensemble s’affiche. Vous pouvez parcourir les différents onglets disponibles, dont chacun contient des représentations visuelles relatives à un aspect spécifique de votre équilibreur de charge. Des conseils explicites sont disponibles pour chacun d’eux dans le tableau de bord, en bas de chaque onglet.

Les onglets du tableau de bord actuellement disponibles sont les suivants :
* Vue d'ensemble
* Disponibilité des front-ends et des back-ends
* Débit de données
* Distribution des flux
* Moniteurs de connexion
* Définitions de métriques 

### <a name="overview-tab"></a>Onglet Overview
L’onglet Vue d’ensemble contient une grille pouvant faire l’objet d’une recherche avec les mesures Disponibilité du chemin de données et État de la sonde d’intégrité pour chacune des adresses IP frontales attachées à votre équilibreur de charge. Ces mesures indiquent si l’adresse IP frontale est réactive et si les instances de calcul de votre pool principal sont réactives individuellement aux connexions entrantes.

Vous pouvez également voir le débit de données global pour chaque adresse IP frontale sur cette page pour savoir si vous êtes en train de produire et si vous recevez les niveaux de trafic attendus. Les conseils situés en bas de la page vous dirigent vers l’onglet approprié en cas d’apparition de valeurs irrégulières.

### <a name="frontend-and-backend-availability-tab"></a>Onglet Disponibilité des front-ends et des back-ends
L’onglet Disponibilité des front-ends et des back-ends présente les mesures Débit du chemin de données et État de la sonde d’intégrité, en plusieurs vues utiles. Le premier graphique montre la valeur agrégée pour vous permettre de déterminer s’il y a un problème. Les autres graphiques montrent ces mesures divisées en diverses dimensions pour vous permettre de détecter et d’identifier les sources des éventuels problèmes de disponibilité entrante.

Un flux de travail pour l’affichage de ces graphiques est fourni en bas de la page, avec les causes courantes de différents symptômes. 

### <a name="data-throughput-tab"></a>Onglet Débit de données
L’onglet Débit de données vous permet de passer en revue le débit entrant et sortant pour déterminer si vos modèles de trafic sont conformes aux attentes. Il présente le débit de données entrant et sortant réparti en IP frontale et Port frontal pour vous permettre de déterminer comment les services que vous exécutez fonctionnent de manière individuelle.

### <a name="flow-distribution"></a>Distribution des flux
L’onglet Distribution des flux vous permet de visualiser et de gérer le nombre de flux que vos instances de serveur principal reçoivent et produisent. Il présente le Taux de création de flux et le Nombre de flux pour le trafic entrant et sortant ainsi que le Trafic réseau que reçoit chaque machine virtuelle et chaque instance de groupe de machines virtuelles identiques. 

Ces vues peuvent vous indiquer que la configuration de votre équilibreur de charge ou vos modèles de trafic entraînent un déséquilibre du trafic. Par exemple, si une affinité de session est configurée et qu’un seul client effectue un nombre disproportionné de demandes. Il vous informe également si vous approchez de la [limite de flux par machine virtuelle](../virtual-network/virtual-machine-network-throughput.md#flow-limits-and-recommendations) pour votre taille de machine.

### <a name="connection-monitors"></a>Moniteurs de connexion
L’onglet Moniteurs de connexion vous montre la latence aller-retour sur une carte pour tous les [moniteurs de connexion](../network-watcher/connection-monitor.md) configurés. Ces représentations visuelles fournissent des informations utiles aux services avec des exigences strictes en matière de latence. Pour répondre à vos besoins, vous devrez peut-être ajouter des déploiements régionaux supplémentaires ou utiliser un modèle d’[équilibrage de charge inter-région](./cross-region-overview.md)

### <a name="metric-definitions"></a>Définitions de métriques
L’onglet Définitions de métriques contient toutes les informations présentées dans l’[article Métriques multidimensionnelles](./load-balancer-standard-diagnostics.md#multi-dimensional-metrics).

## <a name="next-steps"></a>Étapes suivantes
* Passez en revue le tableau de bord et n’hésitez pas à fournir des suggestions d’améliorations à l’aide du lien ci-dessous
* [Passez en revue la documentation sur les métriques pour vérifier que vous comprenez bien comment est calculée chacune d’elles](./load-balancer-standard-diagnostics.md#multi-dimensional-metrics)
* [Créer des moniteurs de connexion pour votre équilibreur de charge](../network-watcher/connection-monitor.md)
* [Créez vos propres classeurs](../azure-monitor/platform/workbooks-overview.md). Inspirez-vous des éléments disponibles en cliquant sur le bouton Modifier de votre tableau de bord de métriques détaillé