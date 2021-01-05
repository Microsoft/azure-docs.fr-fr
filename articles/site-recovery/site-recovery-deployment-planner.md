---
title: Planificateur de déploiement Azure Site Recovery pour la reprise d’activité après sinistre de VMware
description: En savoir plus sur le planificateur de déploiement Azure Site Recovery pour la récupération d’urgence de machines virtuelles VMware sur Azure.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 03/13/2020
ms.author: mayg
ms.openlocfilehash: 9a78b0ec53dfce0a1c1478790f404adb78a8c6b9
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92359847"
---
# <a name="about-the-azure-site-recovery-deployment-planner-for-vmware-to-azure"></a>À propos du planificateur de déploiement Azure Site Recovery pour VMware sur Azure
Cet article est le guide de l’utilisateur d’Azure Site Recovery Deployment Planner portant sur les déploiements de production de VMware vers Azure.

## <a name="overview"></a>Vue d’ensemble

Avant de commencer à protéger les machines virtuelles VMware à l’aide de Azure Site Recovery, allouez suffisamment de bande passante en fonction de votre taux de modifications des données par jour pour atteindre votre objectif de point de récupération (RPO) souhaité. Assurez-vous de déployer le nombre approprié de serveurs de configuration et de serveurs de processus en local.

Vous devez également créer le type approprié et le nombre de comptes de stockage Azure cibles. Vous créez des comptes de stockage standard ou premium, en tenant compte de la croissance de vos serveurs de production sources en raison d’une utilisation accrue au fil du temps. Vous choisissez le type de stockage par machine virtuelle, en fonction des caractéristiques de charge de travail (par exemple, les opérations de lecture/écriture d’E/S par seconde, ou l’activité des données) et des limites Site Recovery.

 Le planificateur de déploiement Site Recovery est un outil de ligne de commande pour les scénarios de récupération d’urgence de Hyper-V vers Azure et de VMware vers Azure. Vous pouvez profiler à distance vos machines virtuelles VMware à l’aide de cet outil (sans conséquence sur la production) pour comprendre les besoins de stockage et de bande passante afin d’assurer la réussite de la réplication et du test de basculement. Vous pouvez exécuter l’outil sans installer les composants Site Recovery locaux. Pour obtenir des résultats de débit précis, exécutez l’outil sur un serveur Windows Server répondant à la configuration requise minimale du serveur de configuration Site Recovery que vous devrez éventuellement déployer dans les premières étapes du déploiement de production.

L’outil fournit les informations suivantes :

**Évaluation de la compatibilité**

* Une évaluation de l’éligibilité de la machine virtuelle en fonction du nombre de disques, de la taille du disque, des E/S par seconde, de l’activité, du type de démarrage (EFI/BIOS) et de la version du système d’exploitation

**Besoin de bande passante réseau et évaluation de RPO**

* La bande passante réseau estimée requise pour la réplication différentielle
* Le débit pouvant être obtenu par Site Recovery dans le scénario « local vers Azure »
* Le nombre de machines virtuelles à traiter par lot en fonction de la bande passante estimée pour effectuer une réplication initiale pendant une durée donnée
* RPO qui peut être obtenu pour une bande passante donnée
* Impact sur le RPO souhaité si une bande passante inférieure est configurée

**Exigences de l’infrastructure Azure**

* Exigences relatives au type de stockage (stockage standard ou premium) pour chaque machine virtuelle
* Nombre total de comptes de stockage standard et premium à configurer pour la réplication (y compris les comptes de stockage de cache)
* Suggestions d’affectation de noms pour les comptes de stockage en fonction des conseils liés au stockage
* Le nombre de cœurs Azure à configurer avant le test de basculement ou le basculement sur l’abonnement
* La taille recommandée de machine virtuelle Azure pour chaque machine virtuelle sur site

**Exigences de l’infrastructure locale**
* Nombre requis de serveurs de configuration et de serveurs de processus à déployer en local

**Estimation du coût de récupération d’urgence vers Azure**
* Estimation du coût total de récupération d’urgence vers Azure : coût de calcul, stockage, réseau et de licence Site Recovery
* Analyse détaillée du coût par machine virtuelle


>[!IMPORTANT]
>
>Étant donné que l’utilisation est susceptible d’augmenter au fil du temps, tous les calculs de l’outil précédent sont effectués en supposant un facteur de croissance de 30 pour cent de caractéristiques de charge de travail. Les calculs utilisent également une valeur de 95e centile de toutes les mesures profilage (E/S par seconde en lecture/écriture, activité, etc.). Ces deux éléments, facteur de croissance et calcul de centile, sont configurables. Pour en savoir plus sur le facteur de croissance, consultez la section « Considérations relatives au facteur de croissance ». Pour en savoir plus sur la valeur de centile, consultez la section « Valeur de centile utilisée pour le calcul ».
>

## <a name="support-matrix"></a>Matrice de prise en charge

| **Catégorie** | **VMware vers Azure** |**Hyper-V vers Azure**|**Azure vers Azure**|**Hyper-V vers un site secondaire**|**VMware vers un site secondaire**
--|--|--|--|--|--
Scénarios pris en charge |Oui|Oui|Non|Oui*|Non
Version prise en charge | vCenter 6.7, 6.5, 6.0 ou 5.5| Windows Server 2016, Windows Server 2012 R2 | N/D |Windows Server 2016, Windows Server 2012 R2|N/D
Configuration prise en charge|vCenter, ESXi| Cluster Hyper-V, hôte Hyper-V|N/D|Cluster Hyper-V, hôte Hyper-V|N/D|
Nombre de serveurs pouvant être profilés par instance en cours d’exécution du planificateur de déploiement Site Recovery |Unique (des machines virtuelles appartenant à un vCenter Server ou un serveur ESXi peuvent être profilées à la fois)|Plusieurs (des machines virtuelles sur plusieurs hôtes ou clusters hôtes peuvent être profilées à la fois)| N/D |Plusieurs (des machines virtuelles sur plusieurs hôtes ou clusters hôtes peuvent être profilées à la fois)| N/D

\* L’outil s’applique principalement au scénario de récupération d’urgence Hyper-V vers Azure. Pour la récupération d’urgence Hyper-V vers un site secondaire, il permet uniquement de disposer de recommandations côté source telles que la bande passante réseau requise, l’espace de stockage disponible requis sur chacun des serveurs Hyper-V source, et les numéros et définitions de lots de réplication initiale. Ignorez les recommandations et coûts Azure dans le rapport. De plus, l’opération d’obtention du débit ne s’applique pas au scénario de récupération d’urgence Hyper-V vers un site secondaire.

## <a name="prerequisites"></a>Prérequis
L’outil comporte deux phases principales : le profilage et la génération de rapport. En outre, une troisième option permet de calculer le débit uniquement. La configuration requise pour le serveur à partir de laquelle le profilage et la mesure du débit sont initiés est présentée dans le tableau suivant.

| Configuration requise du serveur | Description|
|---|---|
|Profilage et mesure du débit| <ul><li>Système d’exploitation : Windows Server 2016 ou Windows Server 2012 R2<br>(dans l’idéal, correspondant au moins aux [recommandations de taille pour le serveur de configuration](/en-in/azure/site-recovery/site-recovery-plan-capacity-vmware#size-recommendations-for-the-configuration-server))</li><li>Configuration de l’ordinateur : 8 processeurs virtuels, 16 Go de RAM, disque dur de 300 Go</li><li>[.NET Framework 4.5](https://aka.ms/dotnet-framework-45)</li><li>[VMware vSphere PowerCLI 6.0 R3](https://aka.ms/download_powercli)</li><li>[Visual C++ Redistributable pour Visual Studio 2012](https://aka.ms/vcplusplus-redistributable)</li><li>Accès Internet à Azure (*.blob.core.windows.net) à partir de ce serveur, port 443<br>[Cette étape est facultative. Vous pouvez choisir de fournir manuellement la bande passante disponible pendant la génération de rapport.]</li><li>Compte Azure Storage</li><li>Accès administrateur sur le serveur</li><li>Au minimum 100 Go d’espace disque disponible (en supposant que 1 000 machines virtuelles avec une moyenne de trois disques chacune, profilage pour 30 jours)</li><li>Les paramètres de niveau de statistiques de VMware vCenter peuvent être définis sur 1 ou un niveau supérieur</li><li>Autoriser le port vCenter (port 443 par défaut) : Le planificateur de déploiement Site Recovery se sert de ce port pour se connecter au serveur vCenter/à l’hôte ESXi</ul></ul>|
| Génération de rapport | Un PC Windows ou serveur Windows Server doté de Excel 2013 ou version ultérieure.<li>[.NET Framework 4.5](https://aka.ms/dotnet-framework-45)</li><li>[Visual C++ Redistributable pour Visual Studio 2012](https://aka.ms/vcplusplus-redistributable)</li><li>[VMware vSphere PowerCLI 6.0 R3](https://aka.ms/download_powercli) est requis uniquement lorsque vous passez l’option -User dans la commande de génération de rapport pour extraire les dernières informations de configuration de machine virtuelle des machines virtuelles. Le planificateur de déploiement se connecte au serveur vCenter. Autoriser le port vCenter (port 443 par défaut) à se connecter au serveur vCenter.</li>|
| Autorisations utilisateur | Autorisation de lecture seule pour le compte d’utilisateur utilisé pour accéder au serveur VMware vCenter/à l’hôte VMware vSphere ESXi lors du profilage |

> [!NOTE]
>
>L’outil peut profiler uniquement des machines virtuelles avec des disques de machines virtuelles et des disques RDM. Il ne peut pas profiler les machines virtuelles équipées de disques iSCSI ou NFS. Site Recovery prend en charge les disques iSCSI et NFS pour les serveurs VMware. Comme le planificateur de déploiement ne se trouve pas dans l’invité et n’effectue le profilage qu’à l’aide des compteurs de performance vCenter, il n’a pas de visibilité sur ces types de disques.
>

## <a name="download-and-extract-the-deployment-planner-tool"></a>Télécharger et extraire l’outil de planification du déploiement
1. Téléchargez la dernière version du [Planificateur de déploiement Azure Site Recovery](https://aka.ms/asr-deployment-planner).
L’outil se présente dans un dossier .zip. Sa version actuelle ne prend en charge que le scénario VMware vers Azure.

2. Copiez le dossier .zip sur le serveur Windows à partir duquel vous voulez exécuter l’outil.
Vous pouvez exécuter l’outil à partir de Windows Server 2012 R2 si le serveur a accès au réseau pour se connecter au serveur vCenter/à l’hôte vSphere ESXi qui contient les machines virtuelles à profiler. Toutefois, nous vous recommandons d’exécuter l’outil sur un serveur dont la configuration matérielle répond aux [indications de dimensionnement pour les serveurs de configuration](/en-in/azure/site-recovery/site-recovery-plan-capacity-vmware#size-recommendations-for-the-configuration-server). Si vous avez déjà déployé les composants Site Recovery en local, exécutez l’outil à partir du serveur de configuration.

    Nous vous recommandons d’adopter la même configuration matérielle que le serveur de configuration (qui dispose d’un serveur de processus intégré) sur le serveur sur lequel vous exécutez l’outil. Cette configuration garantit que le débit atteint rapporté par l’outil corresponde au débit que Site Recovery peut atteindre lors de la réplication. Le calcul du débit dépend de la bande passante réseau disponible sur le serveur et de la configuration matérielle (comme le processeur, le stockage, etc.) du serveur. Si vous exécutez l’outil à partir de n’importe quel autre serveur, le débit est calculé à partir de ce serveur vers Azure. En outre, étant donné que la configuration matérielle du serveur peut différer de celle du serveur de configuration, le débit atteint rapporté par l’outil peut être inexact.

3. Extrayez le dossier .zip.
Le dossier contient plusieurs fichiers et sous-dossiers. Le fichier exécutable s’appelle ASRDeploymentPlanner.exe dans le dossier parent.

    Exemple : Copiez le fichier .zip sur le lecteur E:\ et extrayez-le.
    E:\ASR Deployment Planner_v2.3.zip

    E:\ASR Deployment Planner_v2.3\ASRDeploymentPlanner.exe

### <a name="update-to-the-latest-version-of-deployment-planner"></a>Mise à jour vers la dernière version du planificateur de déploiement

Les dernières mises à jour sont résumées dans l’[historique des versions](site-recovery-deployment-planner-history.md) du Planificateur de déploiement.

Si vous disposez d’une version précédente du planificateur de déploiement, effectuez l’une des actions suivantes :
 * Si la dernière version ne contient pas de correctif de profilage, et que le profilage est déjà en cours sur votre version actuelle de la planification, passez au profilage.
 * Si la dernière version contient un correctif de profilage, nous vous recommandons d’arrêter le profilage sur votre version actuelle et de redémarrer le profilage avec la nouvelle version.


 >[!NOTE]
 >
 >Lorsque vous démarrez le profilage avec la nouvelle version, transmettez le même chemin de répertoire de sortie de sorte que l’outil ajoute les données de profil sur les fichiers existants. Un ensemble complet de données profilées est utilisé pour générer le rapport. Si vous transmettez un répertoire de sortie différent, des fichiers sont créés et les anciennes données profilées ne sont pas utilisées pour générer les rapports.
 >
 >Chaque nouvelle version du planificateur de déploiement est une mise à jour cumulative du fichier .zip. Vous n’avez pas besoin de copier les fichiers les plus récents dans le dossier précédent. Vous pouvez créer un dossier et l’utiliser.


## <a name="version-history"></a>Historique des versions
La dernière version de l’outil Planificateur de déploiement Site Recovery est la version 2.5.
Reportez-vous à la page [Historique des versions du Planificateur de déploiement Site Recovery](./site-recovery-deployment-planner-history.md) pour voir les correctifs ajoutés à chaque mise à jour.

## <a name="next-steps"></a>Étapes suivantes
[Exécuter le Planificateur de déploiement Site Recovery](site-recovery-vmware-deployment-planner-run.md)