---
title: Compréhension de cloud-init
description: Compréhension approfondie de la configuration d’une machine virtuelle Azure à l’aide de cloud-init.
author: danielsollondon
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: conceptual
ms.date: 07/06/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: f5028abadbe5600058c83a144d0095aee1278fe6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86042071"
---
# <a name="diving-deeper-into-cloud-init"></a>Approfondissement de cloud-init
Pour en savoir plus sur [cloud-init](https://cloudinit.readthedocs.io/en/latest/index.html) ou pour le dépanner à un niveau plus approfondi, vous devez comprendre son fonctionnement. Ce document met en évidence les parties importantes et explique les spécificités d’Azure.

Quand cloud-init est inclus dans une image généralisée et qu’une machine virtuelle est créée à partir de cette image, les configurations sont traitées sur 5 index lors du démarrage initial. Ces index sont importants, car ils vous indiquent à quel point cloud-init appliquera les configurations. 


## <a name="understand-cloud-init-configuration"></a>Présentation de la configuration cloud-init
Pour configurer une machine virtuelle afin qu’elle s’exécute sur une plateforme, cloud-init doit appliquer plusieurs configurations, en tant que contrôle serveur consommateur d’images, les principales configurations avec lesquelles vous interagissez sont `User data` (customData), prenant en charge plusieurs formats. Pour en savoir plus, cliquez [ici](https://cloudinit.readthedocs.io/en/latest/topics/format.html#user-data-formats). Vous avez également la possibilité d’ajouter et d’exécuter des scripts (/var/lib/cloud/scripts) pour une configuration supplémentaire. Pour plus d’informations, voir ci-dessous.

Certaines configurations sont déjà intégrées aux images de la place de marché Azure qui accompagnent cloud-init, par exemple :

1. **Source de données Cloud** : cloud-init contient du code qui peut interagir avec les plateformes de cloud, celles-ci sont appelées « datasources ». Quand une machine virtuelle est créée à partir d’une image cloud-init dans [Azure](https://cloudinit.readthedocs.io/en/latest/topics/datasources/azure.html#azure), cloud-init charge la source de donnée Azure, qui interagit avec les points de terminaison de métadonnées Azure pour obtenir la configuration spécifique à la machine virtuelle.
2. **Configuration du runtime** (/run/cloud-init)
3. **Configuration des images** (/etc/cloud), comme `/etc/cloud/cloud.cfg`, `/etc/cloud/cloud.cfg.d/*.cfg`. Par exemple, dans Azure, il est courant que les images du système d’exploitation Linux avec cloud-init disposent d’une directive de source de données Azure, qui indique à cloud-init quelle source de données utiliser, ce qui permet de gagner du temps sur cloud-init :

   ```bash
   /etc/cloud/cloud.cfg.d# cat 90_dpkg.cfg
   # to update this file, run dpkg-reconfigure cloud-init
   datasource_list: [ Azure ]
   ```


## <a name="cloud-init-boot-stages-processing-configuration"></a>Index de démarrage de cloud-init (configuration du traitement)

La configuration avec cloud-init implique 5 index de démarrage, qui traitent la configuration et sont affichés dans les journaux.

1. [Index de générateur](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#generator): Le générateur du système cloud-init démarre et détermine que cloud-init doit être inclus dans les objectifs de démarrage. dans ce cas, il active cloud-init. 

2. [Index local cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#local) : Ici, cloud-init recherche la source de données « Azure » locale, ce qui permet à cloud-init d’interagir avec Azure et d’appliquer une configuration de mise en réseau, y compris la mise en réseau de secours.

3. [Index d’initialisation cloud-init (Réseau)](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#network) : La mise en réseau doit être en ligne et les informations sur la carte réseau et la table de routage doivent être générées. À ce niveau, les modules figurant dans `cloud_init_modules` dans /etc/cloud/cloud.cfg sont exécutés. La machine virtuelle dans Azure est montée, le disque éphémère est formaté, le nom d’hôte est défini, ainsi que d’autres tâches.

   Voici quelques-uns des `cloud_init_modules` :
   
   ```bash
   - migrator
   - seed_random
   - bootcmd
   - write-files
   - growpart
   - resizefs
   - disk_setup
   - mounts
   - set_hostname
   - update_hostname
   - ssh
   ```
   
   Après cet index, cloud-init signalera à la plateforme Azure que la machine virtuelle a été configurée avec succès. Certains modules peuvent avoir échoué, mais les défaillances de module ne provoquent pas l’échec de la configuration.

4. [Index de configuration cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#config): À ce niveau, les modules figurant dans `cloud_config_modules` définis et répertoriés dans /etc/cloud/cloud.cfg sont exécutés.


5. [Index final cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/boot.html#final): Lors de cet index final, les modules figurant dans `cloud_final_modules`, répertoriés dans /etc/cloud/cloud.cfg sont exécutés. Voici les modules qui doivent être exécutés à la fin de l’exécution du processus de démarrage, tels que les installations de packages et les scripts d’exécution, etc. 

   -   Au cours de cet index, vous pouvez exécuter des scripts en les plaçant dans les répertoires sous `/var/lib/cloud/scripts` :
   - `per-boot` : les scripts dans ce répertoire s’exécutent à chaque démarrage
   - `per-instance` : les scripts dans ce répertoire s’exécutent lorsqu’une nouvelle instance est d’abord démarrée
   - `per-once` : les scripts dans ce répertoire s’exécutent une seule fois

## <a name="next-steps"></a>Étapes suivantes

[Résolution des problèmes cloud-init](cloud-init-troubleshooting.md).
