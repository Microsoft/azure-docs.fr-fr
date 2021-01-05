---
title: Surveillance de modèles d’évolution sur des machines virtuelles
description: Découvrez comment surveiller des modèles d’évolution sur des machines virtuelles protégées à l’aide d’Azure Site Recovery
author: Sharmistha-Rai
manager: gaggupta
ms.topic: how-to
ms.date: 09/09/2020
ms.author: sharrai
ms.openlocfilehash: aeb89a9d18e4550fa1d6162920d60507fd50c208
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92359864"
---
# <a name="monitoring-churn-patterns-on-virtual-machines"></a>Surveillance de modèles d’évolution sur des machines virtuelles

Cet article fournit une vue d’ensemble des différents outils permettant de surveiller des modèles d’évolution sur une machine virtuelle. Avec des outils appropriés, il est facile de déterminer exactement quelle application est à l’origine d’une évolution importante. Ensuite, vous pouvez appliquer prendre des mesures en lien avec cette application.

## <a name="for-azure-virtual-machines-windows-or-linux"></a>Pour des machines virtuelles Azure (Windows ou Linux)

Si votre machine est hébergée dans Azure et utilise un disque managé ou non pour le stockage, vous pouvez facilement suivre les performances en suivant les métriques du disque. Cela vous permet d’exercer une surveillance étroite et de sélectionner le disque adapté à votre mode d’utilisation de l’application. Ces outils vous permettent également de créer des alertes, de diagnostiquer et d’introduire de l’automatisation. [Plus d’informations](https://azure.microsoft.com/blog/per-disk-metrics-managed-disks/)

Une fois que vous avez protégé vos machines avec Azure Site Recovery, vous pouvez les surveiller à l’aide des journaux Azure Monitor et de Log Analytics. [Plus d’informations](./monitor-log-analytics.md)

Vous pouvez également utiliser certains outils spécifiques du système d’exploitation.

## <a name="for-windows-machines"></a>Pour les machines Windows

Si vous disposez d’un ordinateur, qu’il soit local ou non, qui exécute un système d’exploitation Windows, quelques outils supplémentaires sont disponibles.

Outre la vérification de l’utilisation du disque sur le gestionnaire des tâches, vous pouvez toujours vous référer au **Moniteur de ressources** et à l’ **Analyseur de performances** . Ces outils sont déjà présents sur les ordinateurs Windows.

### <a name="resource-monitor"></a>Moniteur de ressources

Le **Moniteur de ressources** affiche des informations sur l’utilisation des ressources matérielles et logicielles en temps réel. Pour exécuter le Moniteur de ressources sur un ordinateur Windows, procédez comme suit :

1. Appuyez sur Win+R, puis tapez _resmon_ .
1. Une fois la fenêtre de resmon, autrement dit du Moniteur de ressources, ouverte, basculez vers l’onglet Disque. Il affiche la vue suivante :

    ![Onglet Disque du Moniteur de ressources](./media/monitoring-high-churn/resmon-disk-tab.png)

1. Cet onglet doit être surveillé en continu pendant un certain temps pour obtenir une image claire. Dans l’exemple ci-dessus, nous constatons que le fichier _wmiprv.exe_ est évolue beaucoup.

Une fois que vous avez identifié les applications à l’origine de cette évolution importante sur votre ordinateur, vous pouvez prendre les mesures nécessaires pour atténuer l’évolution résultant de ces applications.

### <a name="performance-monitor"></a>Analyseur de performances

L’ **Analyseur de performances** surveille diverses activités sur un ordinateur, telles que l’utilisation du processeur ou de la mémoire. Pour exécuter l’Analyseur de performances sur un ordinateur Windows, procédez comme suit :

1. Appuyez sur Win+R, puis tapez _Perfmon_ .
1. Une fois la fenêtre de Perfmon, autrement dit, de l’Analyseur de performances, ouverte, elle présente l’affichage suivant :

    ![Analyseur de performances Étape1](./media/monitoring-high-churn/perfmon-step1.png)

1. Développez le dossier **Outils d’analyse** à droite, puis cliquez sur Analyseur de performances. Cela a également pour effet d’ouvrir l’affichage ci-dessous, qui fournit des informations en temps réel sur les performances actuelles.

    ![Analyseur de performances Étape2](./media/monitoring-high-churn/perfmon-step1.png)

1. Ce graphique ne surveille actuellement qu’un seul moniteur, à savoir « % de temps processeur », comme l’indique le tableau juste en dessous du graphique. Vous pouvez ajouter des éléments à surveiller en cliquant sur le signe **« + »** en haut de l’outil.
1. Vous trouverez ci-dessous une représentation visuelle de l’apparence de l’Analyseur de performances après l’ajout de compteurs.

    ![Analyseur de performances Étape3](./media/monitoring-high-churn/perfmon-step3.png)

Découvrez des informations supplémentaires sur l’Analyseur de performances [ici](/dynamics365/business-central/dev-itpro/administration/monitor-use-performance-monitor-collect-event-trace-data).

## <a name="for-linux-machines"></a>Pour les machines Linux

Si vous disposez d’un ordinateur, qu’il soit local ou non, qui exécute un système d’exploitation Linux, quelques outils supplémentaires sont disponibles pour surveiller les modèles d’évolution.

### <a name="iotop"></a>Iotop

L’un des outils les plus utilisés est _iotop_ . Il s’agit d’un utilitaire permettant d’afficher l’activité du disque en temps réel. Il est capable de répertorier les processus à l’origine des E/S, ainsi que la bande passante du disque qu’ils utilisent.

Ouvrez l’invite de commandes et exécutez la commande suivante : `iotop`.

### <a name="iostat"></a>IoStat

IoStat est un outil simple qui collecte et affiche les statistiques du périphérique de stockage d’entrées et sorties du système. Cet outil est souvent utilisé pour suivre les problèmes de performances des périphériques de stockage : appareils, disques locaux et disques distants.

Ouvrez l’invite de commandes, puis exécutez la commande suivante : `iostat`.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment surveiller à l’aide de la plateforme [Azure Monitor](monitor-log-analytics.md).