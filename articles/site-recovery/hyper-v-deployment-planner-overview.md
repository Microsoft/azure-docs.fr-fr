---
title: Planificateur de déploiement pour la récupération d’urgence Hyper-V avec Azure Site Recovery
description: En savoir plus sur le planificateur de déploiement Azure Site Recovery pour la récupération d’urgence de Hyper-V sur Azure.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 3/13/2020
ms.author: mayg
ms.openlocfilehash: e4f1931aab056306ac5e9f9e9ef402ca26ec2d19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86528942"
---
# <a name="about-the-azure-site-recovery-deployment-planner-for-hyper-v-disaster-recovery-to-azure"></a>À propos du planificateur de déploiement Azure Site Recovery pour la récupération d’urgence de Hyper-V sur Azure

Cet article est le guide de l’utilisateur du planificateur de déploiement Azure Site Recovery portant sur les déploiements de production de Hyper-V vers Azure.

Avant de commencer à protéger les machines virtuelles Hyper-V à l’aide de Site Recovery, allouez suffisamment de bande passante en fonction de votre taux de modifications des données par jour pour atteindre votre objectif de point de récupération (RPO) souhaité, et allouez suffisant d’espace de stockage disponible sur chaque volume de stockage Hyper-V local.

Vous devez également créer le type approprié et le nombre de comptes de stockage Azure cibles. Vous créez des comptes de stockage standard ou premium, en tenant compte de la croissance de vos serveurs de production sources en raison d’une utilisation accrue au fil du temps. Vous choisissez le type de stockage par machine virtuelle, en fonction des caractéristiques de charge de travail, par exemple, les opérations de lecture/écriture d’E/S par seconde, ou l’activité des données, et des limites Azure Site Recovery. 

Le planificateur de déploiement Azure Site Recovery est un outil de ligne de commande pour les scénarios de récupération d’urgence de Hyper-V vers Azure et de VMware vers Azure. Vous pouvez profiler à distance vos machines virtuelles Hyper-V présentes sur plusieurs hôtes Hyper-V à l’aide de cet outil (sans conséquences sur la production) pour comprendre les besoins de bande passante et de stockage Azure afin d’assurer la réussite de la réplication et du test de basculement/basculement. Vous pouvez exécuter l’outil sans installer les composants Azure Site Recovery locaux. Toutefois, pour obtenir des résultats de débit précis, nous vous recommandons d’exécuter le planificateur sur un serveur Windows qui possède la même configuration matérielle que celle d’un des serveurs Hyper-V que vous utiliserez pour activer la protection de récupération d’urgence vers Azure. 

L’outil fournit les informations suivantes :

**Évaluation de la compatibilité**

* Évaluation de l’éligibilité de la machine virtuelle en fonction du nombre de disques, de la taille du disque, des E/S par seconde, de l’activité et de quelques caractéristiques de machine virtuelle.

**Besoin de bande passante réseau et évaluation de RPO**

* La bande passante réseau estimée requise pour la réplication différentielle
* Débit pouvant être obtenu par Azure Site Recovery dans le scénario « local vers Azure »
* RPO qui peut être obtenu pour une bande passante donnée
* Impact sur le RPO souhaité si une bande passante inférieure est configurée.

    
**Exigences de l’infrastructure Azure**

* Le type de stockage (compte de stockage standard ou premium) pour chaque machine virtuelle
* Le nombre total de comptes de stockage standard et premium à configurer pour la réplication
* Suggestions d’affectation de noms pour les comptes de stockage en fonction des conseils liés au Stockage Azure
* L’emplacement du compte de stockage pour toutes les machines virtuelles
* Le nombre de cœurs Azure à configurer avant le test de basculement ou le basculement sur l’abonnement
* La taille recommandée de machine virtuelle Azure pour chaque machine virtuelle sur site

**Exigences de l’infrastructure locale**
* Espace de stockage disponible requis sur chaque volume de stockage Hyper-V pour la réussite de la réplication initiale et de la réplication delta afin de s’assurer que la réplication de machine virtuelle n’entraînera pas de temps d’arrêt indésirables pour vos applications de production
* Fréquence de copie maximale à définir pour la réplication Hyper-V

**Conseils de traitement par lot de réplication initiale** 
* Nombre de lots de machines virtuelles à utiliser pour la protection
* Liste des machines virtuelles de chaque lot
* Ordre dans lequel chaque lot doit être protégé
* Durée estimée d’exécution de la réplication initiale de chaque lot

**Estimation du coût de récupération d’urgence vers Azure**
* Estimation du coût total de récupération d’urgence vers Azure : coût de calcul, stockage, réseau et de licence Azure Site Recovery
* Analyse détaillée du coût par machine virtuelle



>[!IMPORTANT]
>
>Étant donné que l’utilisation est susceptible d’augmenter au fil du temps, tous les calculs de l’outil précédent sont effectués en supposant un facteur de croissance de 30 % de caractéristiques de charge de travail et à l’aide d’une valeur de 95e centile de toutes les mesures profilage (E/S par seconde en lecture/écriture, activité, etc.). Ces deux éléments (facteur de croissance et calcul de centile), sont configurables. Pour en savoir plus sur le facteur de croissance, consultez la section « Considérations relatives au facteur de croissance ». Pour en savoir plus sur la valeur de centile, consultez la section « Valeur de centile utilisée pour le calcul ».
>

## <a name="support-matrix"></a>Matrice de prise en charge

|**Catégories** | **VMware vers Azure** |**Hyper-V vers Azure**|**Azure vers Azure**|**Hyper-V vers un site secondaire**|**VMware vers un site secondaire**
--|--|--|--|--|--
Scénarios pris en charge |Oui|Oui|Non|Oui*|Non
Version prise en charge | vCenter 6.7, 6.5, 6.0 ou 5.5| Windows Server 2016, Windows Server 2012 R2 | N/D |Windows Server 2016, Windows Server 2012 R2|N/D
Configuration prise en charge|vCenter, ESXi| Cluster Hyper-V, hôte Hyper-V|N/D|Cluster Hyper-V, hôte Hyper-V|N/D|
Nombre de serveurs pouvant être profilés par instance en cours d’exécution du planificateur de déploiement Azure Site Recovery |Unique (des machines virtuelles appartenant à un vCenter Server ou un serveur ESXi peuvent être profilées à la fois)|Plusieurs (des machines virtuelles sur plusieurs hôtes ou clusters hôtes peuvent être profilées à la fois)| N/D |Plusieurs (des machines virtuelles sur plusieurs hôtes ou clusters hôtes peuvent être profilées à la fois)| N/D

\* L’outil s’applique principalement au scénario de récupération d’urgence Hyper-V vers Azure. Pour la récupération d’urgence Hyper-V vers un site secondaire, il permet uniquement de disposer de recommandations côté source telles que la bande passante réseau requise, l’espace de stockage disponible requis sur chacun des serveurs Hyper-V source, et les numéros et définitions de lots de réplication initiale.  Ignorez les recommandations et coûts Azure dans le rapport. De plus, l’opération d’obtention du débit ne s’applique pas au scénario de récupération d’urgence Hyper-V vers un site secondaire.

## <a name="prerequisites"></a>Prérequis
L’outil comprend trois phases principales pour Hyper-V : obtention de la liste de machines virtuelles, profilage et génération de rapports. En outre, une quatrième option permet de calculer le débit uniquement. La configuration requise pour le serveur sur lequel les différentes phases doivent être exécutées est présentée dans le tableau suivant :

| Configuration requise du serveur | Description |
|---|---|
|Obtention de la liste de machines virtuelles, profilage et mesure du débit |<ul><li>Système d’exploitation : Microsoft Windows Server 2016 ou Microsoft Windows Server 2012 R2 </li><li>Configuration de l’ordinateur : 8 processeurs virtuels, 16 Go de RAM, disque dur de 300 Go</li><li>[Microsoft .NET Framework 4.5](https://aka.ms/dotnet-framework-45)</li><li>[Microsoft Visual C++ Redistributable pour Visual Studio 2012](https://aka.ms/vcplusplus-redistributable)</li><li>Accès Internet à Azure (*.blob.core.windows.net) à partir de ce serveur, port 443<br>[Cette étape est facultative. Vous pouvez choisir de fournir manuellement la bande passante disponible pendant la génération de rapport.]</li><li>Compte Azure Storage</li><li>Accès administrateur sur le serveur</li><li>Au minimum 100 Go d’espace disque disponible (en supposant que 1 000 machines virtuelles avec une moyenne de trois disques chacune, profilage pour 30 jours)</li><li>La machine virtuelle sur laquelle vous exécutez l’outil de planification du déploiement Azure Site Recovery doit être ajoutée à la liste des hôtes approuvés de tous les serveurs Hyper-V.</li><li>Tous les serveurs Hyper-V à profiler doivent être ajoutées à la liste des hôtes approuvés de la machine virtuelle client sur laquelle l’outil est exécuté. [En savoir plus sur l’ajout de serveurs à la liste des hôtes approuvés](#steps-to-add-servers-into-trustedhosts-list). </li><li> L’outil doit être exécuté avec des privilèges d’administrateur à partir de PowerShell ou de la console de ligne de commande sur le client</ul></ul>|
| Génération de rapport | Un PC Windows ou serveur Windows Server doté de Microsoft Excel 2013 ou version ultérieure |
| Autorisations utilisateur | Compte Administrateur pour accéder au cluster Hyper-V/ à l’hôte Hyper-V au moment de l’obtention de la liste de machines virtuelles virtuels et du profilage.<br>Tous les hôtes à profiler doivent disposer d’un compte Administrateur de domaine avec les mêmes informations d’identification, à savoir le nom d’utilisateur et le mot de passe
 |

## <a name="steps-to-add-servers-into-trustedhosts-list"></a>Étapes d’ajout de serveurs à la liste des hôtes approuvés
1. Toutes les hôtes à profiler doivent être répertoriés dans la liste des hôtes approuvés de la machine virtuelle sur laquelle l’outil doit être déployé. Pour ajouter le client à la liste des hôtes approuvés, exécutez la commande suivante à partir d’une session PowerShell avec élévation de privilèges sur la machine virtuelle. La machine virtuelle peut être un Windows Server 2012 R2 ou Windows Server 2016. 

   ```powershell
   set-item wsman:\localhost\Client\TrustedHosts -value '<ComputerName>[,<ComputerName>]' -Concatenate
   ```
1. Chaque hôte Hyper-V à profiler doit comporter :

    a. La machine virtuelle sur laquelle l’outil sera exécuté dans sa liste des hôtes approuvés. Exécutez la commande suivante à partir d’une session PowerShell avec élévation de privilèges sur l’hôte Hyper-V.

      ```powershell
      set-item wsman:\localhost\Client\TrustedHosts -value '<ComputerName>[,<ComputerName>]' -Concatenate
      ```

    b. Accès distant PowerShell activé.

      ```powershell
      Enable-PSRemoting -Force
      ```

## <a name="download-and-extract-the-deployment-planner-tool"></a>Télécharger et extraire l’outil de planification du déploiement

1.  Téléchargez la dernière version [d’Azure Site Recovery deployment planner](https://aka.ms/asr-deployment-planner).
L’outil se présente dans un dossier .zip. Il prend en charge les scénarios de récupération d’urgence de VMware vers Azure et de Hyper-V vers Azure. Vous pouvez également utiliser cet outil pour le scénario de récupération d’urgence de Hyper-V vers un site secondaire, mais ignorez la recommandation de l’infrastructure Azure dans le rapport.

1.  Copiez le dossier .zip sur le Windows Serveur sur lequel vous voulez exécuter l’outil. Vous pouvez exécuter l’outil sur un Windows Server 2012 R2 ou Windows Server 2016. Le serveur doit disposer d’un accès réseau pour se connecter au cluster Hyper-V ou à l’hôte Hyper-V contenant les machines virtuelles à profiler. Nous vous recommandons d’avoir la même configuration matérielle de machine virtuelle, sur laquelle l’outil sera exécuté, que celle du serveur Hyper-V que vous voulez protéger. Cette configuration garantit que le débit atteint rapporté par l’outil corresponde au débit qu’Azure Site Recovery peut atteindre lors de la réplication. Le calcul du débit dépend de la bande passante réseau disponible sur le serveur et de la configuration matérielle (processeur, stockage, etc.) du serveur. Le débit est calculé à partir du serveur sur lequel l’outil est exécuté vers Azure. Si la configuration matérielle du serveur est différente de celle du serveur Hyper-V, le débit atteint rapporté par l’outil sera inexact.
Configuration recommandée pour la machine virtuelle : 8 processeurs virtuels, 16 Go de RAM, disque dur de 300 Go.

1.  Extrayez le dossier .zip.
Le dossier contient plusieurs fichiers et sous-dossiers. Le fichier exécutable s’appelle ASRDeploymentPlanner.exe dans le dossier parent.

Exemple : Copiez le fichier .zip sur le lecteur E:\ et extrayez-le. E:\ASR Deployment Planner_v2.3.zip

E:\ASR Deployment Planner_v2.3\ASRDeploymentPlanner.exe

### <a name="updating-to-the-latest-version-of-deployment-planner"></a>Mise à jour vers la dernière version du planificateur de déploiement

Les dernières mises à jour sont résumées dans l’[historique des versions](site-recovery-deployment-planner-history.md) du Planificateur de déploiement.

Si vous disposez d’une version précédente du planificateur de déploiement, effectuez l’une des actions suivantes :
 * Si la dernière version ne contient pas de correctif de profilage, et que le profilage est déjà en cours sur votre version actuelle de la planification, passez au profilage.
 * Si la dernière version contient un correctif de profilage, nous vous recommandons d’arrêter le profilage sur votre version actuelle et de redémarrer le profilage avec la nouvelle version.


  >[!NOTE]
  >
  >Lorsque vous démarrez le profilage avec la nouvelle version, transmettez le même chemin de répertoire de sortie de sorte que l’outil ajoute les données de profil sur les fichiers existants. Un ensemble complet de données profilées est utilisé pour générer le rapport. Si vous transmettez un répertoire de sortie différent, des fichiers sont créés et les anciennes données profilées ne sont pas utilisées pour générer les rapports.
  >
  >Chaque nouveau deployment planner est une mise à jour cumulative du fichier .zip. Vous n’avez pas besoin de copier les fichiers les plus récents dans le dossier précédent. Vous pouvez créer un dossier et l’utiliser.

## <a name="version-history"></a>Historique des versions
La dernière version de l’outil Planificateur de déploiement Azure Site Recovery est la version 2.5.
Reportez-vous à la page [Historique des versions du Planificateur de déploiement Azure Site Recovery](https://social.technet.microsoft.com/wiki/contents/articles/51049.asr-deployment-planner-version-history.aspx) pour voir les correctifs ajoutés à chaque mise à jour.


## <a name="next-steps"></a>Étapes suivantes
* [Exécutez le planificateur de déploiement](./hyper-v-deployment-planner-run.md).
