---
title: Historique des versions du Planificateur de déploiement Azure Site Recovery
description: Correctifs et limitations connues des versions du Planificateur de déploiement Azure Site Recovery, ainsi que leurs dates de publication.
services: site-recovery
author: Daya-Patil
manager: carmonm
ms.topic: article
ms.service: site-recovery
ms.date: 6/4/2020
ms.author: dapatil
ms.openlocfilehash: feb4f6a24653aca8da825af90341c8016255e8b6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86133805"
---
# <a name="azure-site-recovery-deployment-planner-version-history"></a>Historique des versions du Planificateur de déploiement Azure Site Recovery

Cet article fournit un historique de toutes les versions du Planificateur de déploiement Azure Site Recovery, et décrit les correctifs et limitations connues de chaque version ainsi que leurs dates de publication.

## <a name="version-252"></a>Version 2.52

**Date de publication : 4 juin 2020**

**Correctifs :**

- Ajout de la prise en charge de vCenter 7.0
- Ajout de la prise en charge des systèmes d’exploitation suivants :

    - SUSE Linux Enterprise 15 (avec toutes les versions mineures)
    - Red Hat Enterprise Linux 8 (avec toutes les versions mineures)


## <a name="version-251"></a>Version 2.51

**Date de publication : 22 août 2019**

**Correctifs :**

- Correction du problème de recommandation de coût avec le Planificateur de déploiement version 2.5

## <a name="version-25"></a>Version 2.5

**Date de publication : 29 juillet 2019**

**Correctifs :**

- Pour les machines virtuelles VMware et les machines physiques, les suggestions sont mises à jour pour être basée sur la réplication vers les disques managés.
- Ajout de la prise en charge pour Windows 10 (x64), Windows 8.1 (x64), Windows 8 (x64), Windows 7 (x64) SP1 ou une version ultérieure

## <a name="version-24"></a>Version 2.4

**Date de publication : 17 avril 2019**

**Correctifs :**

- Amélioration de la compatibilité avec le système d’exploitation, en particulier lors du traitement d’erreurs de localisation.
- Ajout de machines virtuelles avec jusqu’à 20 Mbits/s de taux de modifications des données (évolution) à la liste de contrôle de compatibilité.
- Amélioration des messages d’erreur
- Ajout de la prise en charge de vCenter 6.7.
- Ajout de la prise en charge de Windows Server 2019 et de station de travail Red Hat Enterprise Linux (RHEL).



## <a name="version-23"></a>Version 2.3

**Date de publication : 3 décembre 2018**

**Correctifs :**

- Correction d’un problème qui empêchait le Planificateur de déploiement de générer un rapport avec l’emplacement cible et l’abonnement fournis.

## <a name="version-22"></a>Version 2.2 

**Date de publication : 25 avril 2018**

**Correctifs :**

- Opérations GetVMList :
  - Correction d’un problème qui provoquait un échec de GetVMList si le dossier spécifié n’existait pas. GetVMList crée désormais le répertoire par défaut ou le répertoire spécifié dans le paramètre outputfile.
  - Ajout de motifs d’échec plus détaillés pour GetVMList.
- Ajout d’informations sur le type de machine virtuelle en tant que colonne dans la feuille des machines virtuelles compatibles du rapport du planificateur de déploiement.
- Récupération d’urgence de Hyper-V sur Azure :
  - Exclusion du profilage des machines virtuelles dotées de disques durs virtuels partagés et de disques PassThrough. L’opération Startprofiling affiche la liste des machines virtuelles exclues dans la console.
  - Ajout des machines virtuelles dotées de plus de 64 disques à la liste des machines virtuelles incompatibles.
  - Mise à jour du facteur de compression de la réplication initiale et de la réplication delta.
  - Ajout d’un support limité pour le stockage SMB.

## <a name="version-21"></a>Version 2.1

**Date de publication : 3 janvier 2018**

**Correctifs :**

- Mise à jour du rapport Excel.
- Correction des bogues dans l’opération GetThroughput.
- Ajout d’option pour limiter le nombre de machines virtuelles pour profiler ou générer le rapport. La limite par défaut est fixée à 1 000 machines virtuelles.
- Récupération d’urgence de VMware sur Azure :
  - Correction d’un problème de machine virtuelle Windows Server 2016 placée dans la table incompatible. 
  - Mise à jour des messages de compatibilité pour les machines virtuelles Windows Extensible Firmware Interface (EFI).
- Mise à jour de la limite d’évolution des données de VMware sur Azure et de Hyper-V sur Azure, par machine virtuelle. 
- Amélioration de la fiabilité de l’analyse de fichier de liste de machines virtuelles.

## <a name="version-201"></a>Version 2.0.1

**Date de publication : 7 décembre 2017**

**Correctifs :**

- Ajout d’une suggestion pour optimiser la bande passante réseau.

## <a name="version-20"></a>Version 2.0

**Date de publication : 28 novembre 2017**

**Correctifs :**

- Ajout de la prise en charge de la récupération d’urgence de Hyper-V sur Azure.
- Ajout du calcul du coût.
- Ajout du contrôle de la version du système d’exploitation pour la récupération d’urgence de VMware sur Azure afin de déterminer si la machine virtuelle est compatible ou non pour la protection. L’outil se sert de la chaîne de version du système d’exploitation qui est retournée par le serveur vCenter pour cette machine virtuelle. Il s’agit de la version du système d’exploitation invité que l’utilisateur a sélectionnée lors de la création de la machine virtuelle dans VMware.

**Limitations connues :**

- Pour la récupération d’urgence de Hyper-V sur Azure, les machines virtuelles dont le nom contient des caractères `,`, `"`, `[`, `]` et ``` ` ``` ne sont pas prises en charge. En cas de profilage d’une telle machine, la génération de rapport échoue ou produit un résultat incorrect.
- Pour la récupération d’urgence de VMware sur Azure, les machines virtuelles dont le nom contient un virgule ne sont pas prises en charge. En cas de profilage d’une telle machine, la génération de rapport échoue ou produit un résultat incorrect.

## <a name="version-131"></a>Version 1.3.1

**Date de publication : 19 juillet 2017** 

**Correctifs :**

- Ajout de la prise en charge des disques volumineux (> 1 To) dans la génération de rapport. Vous pouvez désormais utiliser le Planificateur de déploiement pour planifier la réplication de machines virtuelles dont les tailles de disque sont supérieures à 1 To (jusqu’à 4 095 Go).
En savoir plus sur la [Prise en charge des disques volumineux sur Azure Site Recovery](https://azure.microsoft.com/blog/azure-site-recovery-large-disks/)

## <a name="version-13"></a>Version 1.3

**Date de publication : 9 mai 2017**

**Correctifs :**

- Ajout de la prise en charge de disque managé dans la génération de rapport. Le nombre de machines virtuelles pouvant être placées sur un même compte de stockage est calculé selon que le disque managé est sélectionné pour le basculement/test de basculement.

## <a name="version-12"></a>Version 1.2

**Date de publication : 7 avril 2017**

**Correctifs :**

- Ajout de contrôles du type de démarrage (BIOS ou EFI) pour chaque machine virtuelle afin de déterminer si celle-ci est compatible ou non pour la protection.
- Ajout d’informations sur le type de système d’exploitation pour chaque machine virtuelle dans les feuilles de calcul des machines virtuelles compatibles et incompatibles.
- Ajout de la prise en charge de l’opération GetThroughput dans les régions Microsoft Azure US Government et Chine.
- Ajout de quelques autres vérifications préalables pour les serveurs vCenter et ESXi.
- Correction d’un problème de génération de rapport incorrect lorsque les paramètres régionaux sont définis sur une langue autre que l’anglais.

## <a name="version-11"></a>Version 1.1

**Date de publication : 9 mars 2017**

**Correctifs :**

- Correction d’un problème qui empêchait le profilage des machines virtuelles quand il y avait au moins deux machines virtuelles ayant le même nom ou la même adresse IP sur plusieurs hôtes ESXi de vCenter.
- Correction d’un problème qui avait pour effet de désactiver la copie et la recherche pour les feuilles de calcul des machines virtuelles compatibles et incompatibles.

## <a name="version-10"></a>Version 1.0

**Date de publication : 23 février 2017**

**Limitations connues :**

- Prend en charge uniquement les scénarios de récupération d’urgence de VMware sur Azure. Pour les scénarios de récupération d’urgence de Hyper-V sur Azure, servez-vous de l’[outil de planification de la capacité Hyper-V](./hyper-v-deployment-planner-overview.md).
- Ne prend pas en charge l’opération GetThroughput dans les régions Microsoft Azure US Government et Chine.
- L’outil ne peut pas profiler de machines virtuelles si le serveur vCenter possède au moins deux machines virtuelles ayant le même nom ou la même adresse IP sur plusieurs hôtes ESXi.
Dans cette version, l’outil ignore le profilage des noms ou adresses IP des machines virtuelles en double dans le fichier VMListFile. La solution de contournement consiste à profiler des machines virtuelles à l’aide d’un hôte ESXi au lieu du serveur vCenter. Veillez à n’exécuter qu’une seule instance par hôte ESXi.
