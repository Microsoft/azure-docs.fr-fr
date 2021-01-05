---
title: Guide pratique pour utiliser PerfInsights Linux dans Microsoft Azure | Microsoft Docs
description: Découvrez comment utiliser PerfInsights pour résoudre les problèmes de performances des machines virtuelles Linux.
services: virtual-machines-linux'
documentationcenter: ''
author: anandhms
manager: cshepard
editor: na
tags: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: troubleshooting
ms.date: 7/10/2019
ms.author: genli
ms.openlocfilehash: f785f44b88c7f0d5f0b6f43114070888bb23146d
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96302483"
---
# <a name="how-to-use-perfinsights"></a>Utilisation de PerfInsights

[PerfInsights Linux](https://aka.ms/perfinsightslinuxdownload) est un outil de diagnostic autonome qui collecte et analyse les données de diagnostic, puis fournit un rapport qui aide à résoudre les problèmes de performances des machines virtuelles Linux dans Azure. Vous pouvez exécuter PerfInsights sur les machines virtuelles prises en charge en tant qu’outil autonome, ou directement à partir du portail en utilisant [Performance Diagnostics pour les machines virtuelles Azure](performance-diagnostics.md).

Si vous rencontrez des problèmes de performances avec des machines virtuelles, exécutez cet outil avant de contacter le support.

## <a name="supported-troubleshooting-scenarios"></a>Scénarios de résolution des problèmes pris en charge

PerfInsights peut collecter et analyser plusieurs types d’informations. Les sections suivantes couvrent les scénarios courants.

### <a name="quick-performance-analysis"></a>Analyse rapide des performances

Ce scénario collecte des informations de base, telles que la configuration du stockage et du matériel de votre machine virtuelle, les divers journaux, notamment :

- Informations sur le système d’exploitation

- Informations sur les appareils PCI

- Journaux des systèmes d’exploitation invités généraux

- Fichiers de configuration

- Informations sur le stockage

- Configuration des machines virtuelles Azure (collectée avec [Azure Instance Metadata Service](../windows/instance-metadata-service.md))

- Liste des processus exécutés, utilisation du disque, de la mémoire et du processeur

- Informations réseau

Il s’agit d’une collecte passive d’informations qui ne sont pas censées affecter le système.

>[!Note]
>Le scénario d’analyse rapide des performances est automatiquement inclus dans chacun des scénarios suivants :

### <a name="performance-analysis"></a>Analyse des performances

Ce scénario est similaire à l’analyse rapide des performances, mais permet de capturer des informations de diagnostic pour une durée plus longue.

## <a name="what-kind-of-information-is-collected-by-perfinsights"></a>Quel type d’informations collecte PerfInsights

Les informations relatives à la machine virtuelle Linux, au système d’exploitation, aux appareils de traitement par blocs, aux grands consommateurs de ressources, à la configuration et aux différents journaux sont collectées. Voici plus de détails :

- Système d’exploitation
  - Distribution et version Linux
  - Informations sur le noyau
  - Informations sur les pilotes

- Matériel
  - Appareils PCI [`*`]

- Processus et mémoire
  - Liste des processus (nom de la tâche, mémoire utilisée, fichiers ouverts)
  - Mémoire physique totale, disponible et libre
  - Mémoire d’échange totale, disponible et libre
  - Capture de profilage du processeur et de l’utilisation du processeur par les processus à intervalle de 5 secondes
  - Capture de profilage de l’utilisation de la mémoire par les processus à intervalle de 5 secondes

- Mise en réseau  
  - Liste des cartes réseau avec statistiques des cartes
  - Tableau de routage réseau
  - Ports ouverts et état

- Stockage
  - Liste des appareils de traitement par blocs
  - Liste des partitions
  - Liste des points de montage
  - Informations sur le volume MDADM
  - Informations sur le volume LVM
  - Capture de profilage sur tous les disques à intervalle de 5 secondes

- Journaux d’activité
  - /var/log/messages
  - /var/log/syslog
  - /var/log/kern.log
  - /var/log/cron.log
  - /var/log/boot.log
  - /var/log/yum.log
  - /var/log/dpkg.log
  - /var/log/sysstat or /var/log/sa [`**`]
  - /var/log/cloud-init.log
  - /var/log/cloud-init-output.log
  - /var/log/gpu-manager.log
  - /var/log/waagent.log
  - /var/log/azure/[extension folder]/\*log\*
  - /var/opt/microsoft/omsconfig/omsconfig.log
  - /var/opt/microsoft/omsagent/log/omsagent.log
  - /etc/waagent.config
  - Sortie de journalctl des cinq derniers jours

- [Métadonnées d’instance de machine virtuelle Azure](../windows/instance-metadata-service.md)

>[!Note]
>[`*`] Les informations PCI ne sont pas encore collectées sur les distributions Debian et SLES.
> 
>[`**`] /var/log/sysstat ou/var/log/sa contient les fichiers de rapport d’activité système (SAR) qui sont collectés par le package sysstat. Si le package sysstat n’est pas installé sur la machine virtuelle, l’outil PerfInsights recommande de l’installer.

## <a name="run-the-perfinsights-linux-on-your-vm"></a>Exécuter PerfInsights Linux sur votre machine virtuelle

### <a name="what-do-i-have-to-know-before-i-run-the-tool"></a>Que dois-je savoir avant d’exécuter l’outil

#### <a name="tool-requirements"></a>Exigences de l’outil

- Cet outil doit être exécuté sur la machine virtuelle qui présente le problème de performances.
- Python 3.x ou Python 2.7 doit être installé sur la machine virtuelle.

- Les distributions prises en charge sont les suivantes :

    | Distribution               | Version                                         |
    |----------------------------|-------------------------------------------------|
    | Oracle Linux Server        | 6.10 [`*`], 7.3, 7.6, 7.5 |
    | CentOS                     | 6.5 [`*`], 7.6                                    |
    | RHEL                       | 7.6, 7.7, 7.8, 7.9                                |
    | Ubuntu                     | 14.04, 16.04, 18.04, 20.04                               |
    | Debian                     | 8, 9, 10 [`*`]                                    |
    | SLES                       | 12 SP4 [`*`]                                      |
    |                            |                                                   |

>[!Note]
>[`*`] Reportez-vous à la section [Problèmes connus](#known-issues)

### <a name="known-issues"></a>Problèmes connus

- RHEL 8 n’a pas Python installé par défaut. Pour exécuter PerfInsights Linux, vous devez d’abord installer Python 2.7

- La collecte des informations de l’agent invité peut échouer sur CentOS 6.x

- Les informations sur les appareils PCI ne sont pas collectées sur les distributions Debian

- Les informations LVM sont partiellement collectées sur certaines distributions

### <a name="how-do-i-run-perfinsights"></a>Comment exécuter PerfInsights

Vous pouvez exécuter PerfInsights sur une machine virtuelle en installant Azure Performance Diagnostics à partir du portail Azure. Vous pouvez également l’exécuter en tant qu’outil autonome.

>[!Note]
>PerfInsights collecte et analyse simplement les données. Il n’apporte aucune modification au système.

#### <a name="install-and-run-perfinsights-from-the-azure-portal"></a>Installer et exécuter PerfInsights à partir du portail Azure

Pour plus d’informations sur cette option, consultez [Azure Performance Diagnostics](performance-diagnostics.md).  

#### <a name="run-perfinsights-in-standalone-mode"></a>Exécuter PerfInsights en mode autonome

Pour exécuter l’outil PerfInsights, suivez ces étapes :

1. Téléchargez [PerfInsights.tar.gz](https://aka.ms/perfinsightslinuxdownload) dans un dossier sur votre machine virtuelle et extrayez le contenu à l’aide des commandes ci-dessous à partir du terminal.

   ```bash
   wget https://download.microsoft.com/download/9/F/8/9F80419C-D60D-45F1-8A98-718855F25722/PerfInsights.tar.gz
   ```

   ```bash
   tar xzvf PerfInsights.tar.gz
   ```

2. Accédez au dossier qui contient le fichier `perfinsights.py`, puis exécutez `perfinsights.py` pour voir les paramètres de ligne de commande disponibles.

    ```bash
    cd <the path of PerfInsights folder>
    sudo python perfinsights.py
    ```

    ![Capture d’écran de la sortie de ligne de commande de PerfInsights Linux](media/how-to-use-perfinsights-linux/perfinsights-linux-commandline.png)

    La syntaxe de base pour l’exécution de scénarios PerfInsights est la suivante :

    ```bash
    sudo python perfinsights.py -r <ScenarioName> -d [duration]<H | M | S> [AdditionalOptions]
    ```

    Vous pouvez utiliser l’exemple ci-dessous pour exécuter le scénario d’analyse rapide des performances pendant 1 minute et créer les résultats sous le dossier /tmp/output :

    ```bash
    sudo python perfinsights.py -r quick -d 1M -a -o /tmp/output
    ```

    Vous pouvez utiliser l’exemple suivant pour exécuter le scénario d’analyse de performances pendant 5 minutes et charger le fichier tar ball de résultat dans le compte de stockage :

    ```bash
    sudo python perfinsights.py -r vmslow -d 300S -a -t <StorageAccountName> -k <StorageAccountKey> -i <full resource Uri of the current VM>
    ```

    >[!Note]
    >Avant d’exécuter un scénario, PerfInsights invite l’utilisateur à accepter de partager les informations de diagnostic et à accepter le CLUF. Utilisez l’option **-a ou --accept-disclaimer-and-share-diagnostics** pour ignorer ces invites.
    >
    >Si vous avez un ticket de support actif avec Microsoft et que vous exécutez PerfInsights sur demande de l’ingénieur de support avec qui vous travaillez, veillez à fournir le numéro du ticket de support à l’aide de l’option **-s ou --support-request**.

Une fois l’exécution terminée, un nouveau fichier tar s’affiche dans le même dossier que PerfInsights, sauf si aucun dossier de sortie n’est spécifié. Le nom du fichier est **PerformanceDiagnostics\_aaaa-MM-jj\_hh-mm-ss-fff.tar.gz.** Vous pouvez envoyer ce fichier à l’agent du support pour analyse ou ouvrir le rapport dans le fichier pour examiner les résultats et recommandations.

## <a name="review-the-diagnostics-report"></a>Examinez le rapport de diagnostics

Le fichier **PerformanceDiagnostics\_aaaa-MM-jj\_hh-mm-ss-fff.tar.gz** peut inclure un rapport HTML détaillant les conclusions de PerfInsights. Pour consulter le rapport, développez le fichier **PerformanceDiagnostics\_aaaa-MM-jj\_hh-mm-ss-fff.tar.gz**, puis ouvrez le fichier **PerfInsights Report.html**.

### <a name="overview-tab"></a>Onglet Overview

L’onglet **Overview** fournit des détails de base sur l’exécution et des informations sur les machines virtuelles. L’onglet **Findings** affiche un résumé des recommandations de toutes les différentes sections du rapport PerfInsights.

![Capture d’écran de l’onglet Vue d’ensemble du rapport PerfInsights.](media/how-to-use-perfinsights-linux/perfinsights-linux-overview.png)  
![Capture d’écran de l’onglet Linux du rapport PerfInsights.](media/how-to-use-perfinsights-linux/perfinsights-linux-findings-tab.png)

> [!NOTE]
> Les conclusions identifiées comme étant d’importance élevée sont des problèmes connus qui peuvent conduire à des problèmes de performances. Les conclusions identifiées comme étant d’importance moyenne représentent des configurations non optimales ne provoquant pas forcément de problèmes de performances. Les conclusions identifiées comme étant d’importance faible sont des instructions données à titre informatif uniquement.

Examinez les suggestions et les liens pour toutes les conclusions d’importance élevée et moyenne. En savoir plus sur la façon dont elles peuvent affecter les performances, et sur les meilleures pratiques pour les configurations de performances optimisées.

### <a name="cpu-tab"></a>Onglet CPU

L’onglet **CPU** fournit des informations sur la consommation du processeur au niveau du système pendant l’exécution de PerfInsights. Les informations sur les périodes de grande utilisation du processeur et sur les principaux consommateurs de processeur durables sont utiles pour résoudre les gros problèmes de processeur.

![Capture d’écran de l’onglet CPU du rapport PerfInsights](media/how-to-use-perfinsights-linux/perfinsights-linux-cpu-tab.png)

### <a name="storage-tab"></a>Onglet Stockage

La section **Conclusions** affiche les différentes conclusions et recommandations relatives au stockage.

L’onglet **Block Devices** et autres sections associées comme les onglets **Partitions**, **LVM** et **MDADM** décrivent comment les appareils de traitement par blocs sont configurés et liés entre eux.

![Capture d’écran de l’onglet Storage](media/how-to-use-perfinsights-linux/perfinsights-linux-storage-tab.png)  
![Capture d’écran de l’onglet MDADM](media/how-to-use-perfinsights-linux/perfinsights-linux-mdadm-config.png)

### <a name="linux-tab"></a>Onglet Linux

L’onglet **Linux** contient des informations sur le matériel et le système d’exploitation exécuté sur votre machine virtuelle. Les détails incluent une liste des processus en cours d’exécution et des informations sur l’agent invité, les appareils PCI, le processeur, les pilotes et les pilotes LIS.

![Capture d’écran de l’onglet Linux](media/how-to-use-perfinsights-linux/perfinsights-linux-tab.png)

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez charger les journaux de diagnostic et les rapports vers le Support Microsoft pour un examen approfondi. Lorsque vous travaillez avec le support Microsoft, il vous sera probablement demandé de transmettre la sortie générée par PerfInsights pour aider le processus de dépannage.

La capture d’écran suivante affiche un message semblable à ce que vous pouvez recevoir :

![Capture d’écran de l’exemple de message envoyé par le Support Microsoft](media/how-to-use-perfinsights-linux/support-email.png)

Suivez les instructions dans le message pour accéder à l’espace de travail de transfert de fichier. Pour plus de sécurité, vous devez modifier votre mot de passe à la première utilisation.

Après vous être connecté, vous voyez s’afficher une boîte de dialogue vous invitant à charger le fichier **PerformanceDiagnostics\_aaaa-MM-jj\_hh-mm-ss-fff.tar.gz** collecté par PerfInsights.
