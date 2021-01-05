---
title: Exécuter le planificateur de déploiement Hyper-V dans Azure Site Recovery
description: Cet article décrit comment exécuter le planificateur de déploiement Azure Site Recovery pour la reprise d’activité de Hyper-V sur Azure.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/09/2019
ms.author: mayg
ms.openlocfilehash: ba1979c940d4a92b3d1a7a52a4f356b2896ece55
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74082618"
---
# <a name="run-the-azure-site-recovery-deployment-planner-for-hyper-v-disaster-recovery-to-azure"></a>Exécuter le planificateur de déploiement Azure Site Recovery pour la reprise d’activité de Hyper-V sur Azure

Vous pouvez exécuter l’outil en ligne de commande (ASRDeploymentPlanner.exe) du planificateur de déploiement Site Recovery dans l’un des quatre modes suivants : 
-   Accéder à la liste de machines virtuelles
-   [Profil](#profile-hyper-v-vms)
-   Générer un rapport
-   [Obtenir le débit](#get-throughput)

Exécutez tout d’abord l’outil pour obtenir la liste des machines virtuelles à partir d’un ou plusieurs hôtes Hyper-V. Exécutez ensuite l’outil en mode profilage pour rassembler l’activité des données de machine virtuelle et les E/S par seconde. Ensuite, exécutez l’outil pour générer le rapport afin de déterminer les besoins en bande passante réseau et en stockage.

## <a name="get-the-vm-list-for-profiling-hyper-v-vms"></a>Obtenir la liste des machines virtuelles pour le profilage de machines virtuelles Hyper-V
Tout d’abord, vous avez besoin d’une liste des machines virtuelles à profiler. Utilisez le mode GetVMList du planificateur de déploiement pour générer la liste des machines virtuelles présentes sur plusieurs hôtes Hyper-V dans une seule et même commande. Une fois la liste complète générée, vous pouvez supprimer les machines virtuelles que vous ne voulez pas profiler du fichier de sortie. Utilisez ensuite le fichier de sortie pour toutes les autres opérations : profilage, génération de rapport et obtention du débit.

Vous pouvez générer la liste des machines virtuelles en pointant l’outil sur un cluster Hyper-V ou un hôte Hyper-V autonome, ou une combinaison des deux.

### <a name="command-line-parameters"></a>Paramètres de ligne de commande
Le tableau suivant contient une liste des paramètres obligatoires et facultatifs de l’outil pour l’exécution en mode GetVMList. 
```
ASRDeploymentPlanner.exe -Operation GetVMList /?
```

| Nom du paramètre | Description |
|---|---|
| -Operation | GetVMList |
| -User | Le nom d’utilisateur pour se connecter à l’hôte Hyper-V ou au cluster Hyper-V. L’utilisateur doit avoir un accès administratif.|
| -ServerListFile | Fichier avec la liste des serveurs contenant les machines virtuelles à profiler. Le chemin d’accès du fichier peut être absolu ou relatif. Ce fichier doit contenir une des valeurs suivantes dans chaque ligne :<ul><li>Nom ou adresse IP d’hôte Hyper-V</li><li>Nom ou adresse IP de cluster Hyper-V</li></ul><br>**Exemple :** le fichier ServerList.txt contient les serveurs suivants :<ul><li>Host_1</li><li>10.8.59.27</li><li>Cluster_1</li><li>Host_2</li>|
| -Répertoire|(Facultatif) La convention d’affectation de noms universelle (UNC) ou le chemin d’accès du répertoire local pour stocker les données générées pendant cette opération. Si aucun nom n’est spécifié, le répertoire ProfiledData figurant dans le chemin d’accès actuel est utilisé comme répertoire par défaut.|
|-OutputFile| (Facultatif) Le fichier avec la liste des machines virtuelles extraites à partir des serveurs Hyper-V donnés est enregistré. Si un nom n’est pas indiqué, les détails sont alors stockés dans VMList.txt.  Utilisez le fichier pour démarrer le profilage après avoir supprimé les machines virtuelles que vous n’avez pas besoin de profiler.|
|-Mot de passe|(Facultatif) Le mot de passe pour se connecter à l’hôte Hyper-V. Si vous ne le spécifiez pas comme paramètre, il vous sera demandé lors de l’exécution de la commande.|

### <a name="getvmlist-discovery"></a>Découverte de GetVMList

- **Cluster Hyper-V** : quand le nom de cluster Hyper-V est indiqué dans le fichier de la liste des serveurs, l’outil recherche tous les nœuds Hyper-V du cluster et obtient les machines virtuelles présentes sur chacun des hôtes Hyper-V.
**Hôte Hyper-V** : quand le nom d’hôte Hyper-V est indiqué, l’outil vérifie d’abord s’il fait partie d’un cluster. Si oui, l’outil extrait les nœuds appartenant au cluster. Il obtient ensuite les machines virtuelles de chaque hôte Hyper-V. 

Vous pouvez également choisir de répertorier dans un fichier les noms conviviaux ou adresses IP des machines virtuelles que vous souhaitez profiler manuellement.

Ouvrez le fichier de sortie dans le bloc-notes et copiez les noms de toutes les machines virtuelles que vous souhaitez profiler dans un autre fichier (par exemple, ProfileVMList.txt). Utilisez un nom de machine virtuelle par ligne. Ce fichier est utilisé en entrée du paramètre -VMListFile de l’outil pour toutes les autres opérations : profilage, génération de rapport et obtention du débit.

### <a name="examples"></a>Exemples

#### <a name="store-the-list-of-vms-in-a-file"></a>Stocker la liste des machines virtuelles dans un fichier
```
ASRDeploymentPlanner.exe -Operation GetVMlist -ServerListFile "E:\Hyper-V_ProfiledData\ServerList.txt" -User Hyper-VUser1 -OutputFile "E:\Hyper-V_ProfiledData\VMListFile.txt"
```

#### <a name="store-the-list-of-vms-at-the-default-location--directory-path"></a>Stocker la liste des machines virtuelles à l’emplacement par défaut (chemin d’accès au répertoire)
```
ASRDeploymentPlanner.exe -Operation GetVMList -Directory "E:\Hyper-V_ProfiledData" -ServerListFile "E:\Hyper-V_ProfiledData\ServerList.txt" -User Hyper-VUser1
```

## <a name="profile-hyper-v-vms"></a>Profiler des machines virtuelles Hyper-V
En mode profilage, le planificateur de déploiement se connecte à chacun des hôtes Hyper-V pour collecter les données de performances sur les machines virtuelles. 

Le profilage n’affecte pas les performances des machines virtuelles de production, car aucune connexion directe n’est établie. Toutes les données de performances sont collectées à partir de l’hôte Hyper-V.

L’outil interroge l’hôte Hyper-V toutes les 15 secondes pour garantir l’exactitude du profilage. Il stocke les données de compteur de performance en moyenne chaque minute.

L’outil gère parfaitement la migration des machines virtuelles d’un nœud vers un autre nœud du cluster et la migration du stockage au sein d’un hôte.

### <a name="getting-the-vm-list-to-profile"></a>Obtenir la liste des machines virtuelles à profiler
Pour créer une liste de machines virtuelles à profiler, reportez-vous à l'opération GetVMList.

Après avoir établi la liste des machines virtuelles à profiler, vous pouvez exécuter l’outil en mode profilage. 

### <a name="command-line-parameters"></a>Paramètres de ligne de commande
Le tableau suivant dresse la liste des paramètres obligatoires et facultatifs de l’outil pour l’exécution en mode profilage. L’outil est courant pour les scénarios de passage de VMware vers Azure et le passage de Hyper-V vers Azure. Ces paramètres sont applicables à Hyper-V. 
```
ASRDeploymentPlanner.exe -Operation StartProfiling /?
```

| Nom du paramètre | Description |
|---|---|
| -Operation | StartProfiling |
| -User | Le nom d’utilisateur pour se connecter à l’hôte Hyper-V ou au cluster Hyper-V. L’utilisateur doit avoir un accès administratif.|
| -VMListFile | Fichier contenant la liste des machines virtuelles à profiler. Le chemin d’accès du fichier peut être absolu ou relatif. Pour Hyper-V, ce fichier est le fichier de sortie de l’opération GetVMList. Si vous préparez manuellement, le fichier doit contenir un nom ou une adresse IP de serveur suivi du nom de machine virtuelle (séparé par un \ par ligne). Le nom de la machine virtuelle spécifié dans le fichier doit être identique au nom de la machine virtuelle sur l’hôte Hyper-V.<br><br>**Exemple :** le fichier VMList.txt contient les machines virtuelles suivantes :<ul><li>Host_1\VM_A</li><li>10.8.59.27\VM_B</li><li>Host_2\VM_C</li><ul>|
|-NoOfMinutesToProfile|Le nombre de minutes pendant lesquelles le profilage doit être exécuté. La valeur minimale est de 30 minutes.|
|-NoOfHoursToProfile|Le nombre d’heures pendant lesquelles le profilage doit être exécuté.|
|-NoOfDaysToProfile |Le nombre de jours pendant lesquels le profilage doit être exécuté. Nous vous recommandons d’exécuter le profilage pendant plus de 7 jours. Cette durée assure que le modèle de charge de travail de votre environnement sur la période spécifiée est observé et utilisé pour fournir une recommandation précise.|
|-Virtualization|Le type de virtualisation (VMware ou Hyper-V).|
|-Répertoire|(Facultatif) L’UNC ou le chemin d’accès du répertoire local pour stocker les données de profilage générées pendant cette opération. Si aucun nom n’est spécifié, le répertoire « ProfiledData » figurant dans le chemin d’accès actuel est utilisé comme répertoire par défaut.|
|-Mot de passe|(Facultatif) Le mot de passe pour se connecter à l’hôte Hyper-V. Si vous ne le spécifiez pas comme paramètre, il vous sera demandé lors de l’exécution de la commande.|
|-StorageAccountName|(Facultatif) Le nom du compte de stockage utilisé pour rechercher le débit réalisable pour la réplication des données locales vers Azure. L’outil charge les données de test sur ce compte de stockage pour calculer le débit. Le compte de stockage doit être de type v1 à usage général (GPv1).|
|-StorageAccountKey|(Facultatif) La clé utilisée pour accéder au compte de stockage. Accédez au portail Azure > **Comptes de stockage** > *Nom du compte de stockage* > **Paramètres** > **Clés d’accès** > **Key1** (ou clé d’accès principale pour un compte de stockage classique).|
|-Environment|(Facultatif) Votre environnement cible pour le compte de stockage Azure. Ce paramètre peut être défini sur l’une des trois valeurs suivantes : AzureCloud, AzureUSGovernment ou AzureChinaCloud. La valeur par défaut est AzureCloud. Utilisez ce paramètre lorsque votre région cible correspond à Azure - Gouvernement des États-Unis ou Azure - Chine 21Vianet.|

Nous vous recommandons de profiler vos machines virtuelles pendant plus de 7 jours. Si le modèle d’activité varie durant un mois, nous vous recommandons d’effectuer le profilage au cours de la semaine lors de l’activité maximale. La meilleure solution est d’effectuer le profilage pendant 31 jours pour obtenir de meilleures recommandations. 

Pendant la période de profilage, ASRDeploymentPlanner.exe continue de s’exécuter. Les entrées du temps de profilage de l’outil sont indiquées en jours. Pour un test rapide de l’outil ou pour des preuves de concept, vous pouvez faire un profilage pendant quelques heures ou minutes. Le temps de profilage minimum autorisé est de 30 minutes. 

Pendant le profilage, vous pouvez éventuellement transmettre un nom et une clé du compte de stockage pour déterminer le débit qu’Azure Site Recovery peut atteindre au moment de la réplication du serveur Hyper-V vers Azure. Si la clé et le nom du compte de stockage ne sont pas transmis au cours du profilage, l’outil ne calcule pas le débit réalisable.

Vous pouvez exécuter plusieurs instances de l’outil pour différents ensembles de machines virtuelles. Veillez à ce que les noms des machines virtuelles ne soient pas répétés dans les ensembles de profilage. Par exemple, supposons que vous avez profilé 10 machines virtuelles (VM1 à VM10). Après quelques jours, vous voulez profiler 5 autres machines virtuelles (VM11 à VM15). Vous pouvez exécuter l’outil à partir d’une autre console de lignes de commande pour le deuxième ensemble de machines virtuelles (VM11 à VM15). 

Assurez-vous que le second ensemble de machines virtuelles ne contient pas de noms de machine virtuelle de la première instance de profilage ou utilisez un autre répertoire de sortie pour la seconde exécution. Si deux instances de l’outil sont utilisées pour profiler les mêmes machines virtuelles et que vous utilisez le même répertoire de sortie, le rapport généré sera incorrect. 

Par défaut, l’outil est configuré pour profiler et générer un rapport comprenant jusqu’à 1 000 machines virtuelles. Vous pouvez modifier la limite en modifiant la valeur de la clé MaxVMsSupported dans le fichier ASRDeploymentPlanner.exe.config.
```
<!-- Maximum number of VMs supported-->
<add key="MaxVmsSupported" value="1000"/>
```
Avec les paramètres par défaut, pour profiler (par exemple) 1 500 machines virtuelles, créez deux fichiers VMList.txt. Un fichier contient 1 000 machines virtuelles, et l’autre en contient 500. Exécutez les deux instances du planificateur de déploiement Azure Site Recovery, une avec le fichier VMList1.txt et l’autre avec le fichier VMList2.txt. Vous pouvez utiliser le même chemin d’accès de répertoire pour stocker les données profilées des machines virtuelles correspondant aux deux fichiers VMList. 

Selon la configuration matérielle (particulièrement en fonction de la mémoire RAM) du serveur à partir duquel l’outil pour générer le rapport est exécuté, l’opération peut échouer à cause d’une quantité de mémoire insuffisante. Si vous avez un bon matériel, vous pouvez modifier la clé MaxVMsSupported avec n’importe quelle valeur supérieure.  

Les configurations de machines virtuelles sont capturées une fois au début de l’opération de profilage et stockées dans un fichier appelé VMDetailList.xml. Ces informations sont utilisées lorsque le rapport est généré. Toute modification de configuration de machine virtuelle (par exemple, un nombre accru de cœurs, de disques ou de cartes réseau) du début à la fin du profilage n’est pas capturée. Si une configuration de machines virtuelles profilées est modifiée pendant le profilage, la solution de contournement consiste à obtenir les toutes dernières informations des machines virtuelles lorsque vous générez le rapport :

* Sauvegardez VMdetailList.xml et supprimez le fichier de son emplacement actuel.
* Transmettez les arguments -User et -Password au moment de la génération du rapport.

La commande de profilage génère plusieurs fichiers dans le répertoire de profilage. Ne supprimez pas les fichiers, car cela affecterait la génération de rapports.

### <a name="examples"></a>Exemples

#### <a name="profile-vms-for-30-days-and-find-the-throughput-from-on-premises-to-azure"></a>Profiler des machines virtuelles pendant 30 jours et déterminer le débit pour le scénario « local vers Azure »
```
ASRDeploymentPlanner.exe -Operation StartProfiling -virtualization Hyper-V -Directory "E:\Hyper-V_ProfiledData" -VMListFile "E:\Hyper-V_ProfiledData\ProfileVMList1.txt"  -NoOfDaysToProfile 30 -User Contoso\HyperVUser1 -StorageAccountName  asrspfarm1 -StorageAccountKey Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==
```

#### <a name="profile-vms-for-15-days"></a>Profiler des machines virtuelles pendant 15 jours
```
ASRDeploymentPlanner.exe -Operation StartProfiling -Virtualization Hyper-V -Directory "E:\Hyper-V_ProfiledData" -VMListFile "E:\vCenter1_ProfiledData\ProfileVMList1.txt"  -NoOfDaysToProfile  15  -User contoso\HypreVUser1
```

#### <a name="profile-vms-for-60-minutes-for-a-quick-test-of-the-tool"></a>Profiler des machines virtuelles pendant 60 minutes pour tester rapidement l’outil
```
ASRDeploymentPlanner.exe -Operation StartProfiling -Virtualization Hyper-V -Directory "E:\Hyper-V_ProfiledData" -VMListFile "E:\Hyper-V_ProfiledData\ProfileVMList1.txt"  -NoOfMinutesToProfile 60 -User Contoso\HyperVUser1
```

#### <a name="profile-vms-for-2-hours-for-a-proof-of-concept"></a>Profiler des machines virtuelles pendant 2 heures pour une preuve de concept
```
ASRDeploymentPlanner.exe -Operation StartProfiling -Virtualization Hyper-V -Directory "E:\Hyper-V_ProfiledData" -VMListFile "E:\Hyper-V_ProfiledData\ProfileVMList1.txt"  -NoOfHoursToProfile 2 -User Contoso\HyperVUser1
```

### <a name="considerations-for-profiling"></a>Considérations relatives au profilage

Si le serveur sur lequel s’exécute l’outil est redémarré ou est tombé en panne, ou si vous fermez l’outil à l’aide de Ctrl + C, les données profilées sont conservées. Cependant, il existe un risque de manquer les 15 dernières minutes de données profilées. Dans ces cas, réexécutez l’outil en mode profilage après le redémarrage du serveur.

Lorsque le nom et la clé du compte de stockage sont transmis, l’outil mesure le débit au cours de la dernière étape du profilage. Si l’outil est fermé avant la fin du profilage, le débit n’est pas calculé. Pour trouver le débit avant de générer le rapport, vous pouvez exécuter l’opération GetThroughput à partir de la console de ligne de commande. Autrement, le rapport généré ne contient pas d’informations de débit.

Azure Site Recovery ne prend pas en charge les machines virtuelles contenant des disques iSCSI et pass-through. L’outil ne peut pas détecter et profiler des disques iSCSI et pass-through joints aux machines virtuelles.

## <a name="generate-a-report"></a>Générer un rapport
L’outil génère un fichier Microsoft Excel avec les macros activées (fichier XLSM) en tant que sortie du rapport. Il récapitule toutes les recommandations concernant le déploiement. Le rapport est intitulé DeploymentPlannerReport_*identificateur numérique unique*.xlsm et placé dans le répertoire spécifié.

À l’issue du profilage, vous pouvez exécuter l’outil en mode génération de rapport. 

### <a name="command-line-parameters"></a>Paramètres de ligne de commande
Le tableau suivant contient une liste des paramètres obligatoires et facultatifs de l’outil à exécuter en mode génération de rapport. L’outil est courant pour le passage de VMware vers Azure et le passage de Hyper-V vers Azure. Les paramètres suivants sont applicables à Hyper-V.
```
ASRDeploymentPlanner.exe -Operation GenerateReport /?
```

| Nom du paramètre | Description |
|---|---|
| -Operation | GenerateReport |
|-VMListFile | Le fichier qui contient la liste des machines virtuelles profilées pour lesquelles le rapport va être généré. Le chemin d’accès du fichier peut être absolu ou relatif. Pour Hyper-V, ce fichier est le fichier de sortie de l’opération GetVMList. Si vous préparez manuellement, le fichier doit contenir un nom ou une adresse IP de serveur suivi du nom de machine virtuelle (séparé par un \ par ligne). Le nom de la machine virtuelle spécifié dans le fichier doit être identique au nom de la machine virtuelle sur l’hôte Hyper-V.<br><br>**Exemple :** le fichier VMList.txt contient les machines virtuelles suivantes :<ul><li>Host_1\VM_A</li><li>10.8.59.27\VM_B</li><li>Host_2\VM_C</li><ul>|
|-Virtualization|Le type de virtualisation (VMware ou Hyper-V).|
|-Répertoire|(Facultatif) UNC ou chemin d’accès du répertoire local où les données profilées (fichiers générés lors du profilage) sont stockées. Ces données sont requises pour générer le rapport. Si aucun nom n’est spécifié, le répertoire « ProfiledData » figurant dans le chemin d’accès actuel est utilisé comme répertoire par défaut.|
| -User | (Facultatif) Le nom d’utilisateur pour se connecter à l’hôte Hyper-V ou au cluster Hyper-V. L’utilisateur doit avoir un accès administratif. Le nom d’utilisateur et le mot de passe sont utilisés pour extraire les dernières informations de configuration des machines virtuelles (comme le nombre de disques, le nombre de cœurs et le nombre de cartes réseau) à utiliser dans le rapport. Si cette valeur n’est pas fournie, les informations de configuration collectées pendant le profilage sont utilisées.|
|-Mot de passe|(Facultatif) Le mot de passe pour se connecter à l’hôte Hyper-V. Si vous ne le spécifiez pas comme paramètre, il vous sera demandé lors de l’exécution de la commande.|
| -DesiredRPO | (Facultatif) L’objectif de point de récupération (RPO) souhaité, en minutes. La valeur par défaut est 15 minutes.|
| -Bandwidth | (Facultatif) La bande passante en mégabits par seconde. Utilisez ce paramètre pour calculer le RPO qui peut être atteint pour la bande passante spécifiée. |
| -StartDate | (Facultatif) La date et l’heure de début au format MM-JJ-AAAA:HH:MM (24 heures). Le paramètre StartDate doit être spécifié avec le paramètre EndDate. Si le paramètre StartDate est spécifié, le rapport est généré pour les données profilées collectées entre les dates StartDate et EndDate. |
| -EndDate | (Facultatif) La date et l’heure de fin au format MM-JJ-AAAA:HH:MM (24 heures). Le paramètre EndDate doit être spécifié avec le paramètre StartDate. Si le paramètre EndDate est spécifié, le rapport est généré pour les données profilées collectées entre les dates StartDate et EndDate. |
| -GrowthFactor | (Facultatif) Le facteur de croissance, exprimé en pourcentage. La valeur par défaut est 30 pour cent. |
| -UseManagedDisks | (Facultatif) UseManagedDisks : Oui/Non. La valeur par défaut de ce paramètre est Oui. Le nombre de machines virtuelles qu’il est possible de placer dans un compte de stockage unique est calculé à partir de l’exécution du basculement/test de basculement des machines virtuelles sur un disque managé au lieu d’un disque non managé. |
|-SubscriptionId |(Facultatif) GUID de l’abonnement. Utilisez ce paramètre pour générer le rapport d’estimation des coûts avec le prix le plus récent selon votre abonnement, l’offre associée à votre abonnement et votre région cible Azure dans la devise indiquée.|
|-TargetRegion|(Facultatif) La région Azure cible de la réplication. Étant donné qu’Azure possède différents coûts par région, utilisez ce paramètre pour générer des rapports avec une région cible Azure spécifique. La valeur par défaut est WestUS2 (Ouest des États-Unis) ou la dernière région cible. Reportez-vous à la liste des [régions cibles prises en charge](hyper-v-deployment-planner-cost-estimation.md#supported-target-regions).|
|-OfferId|(Facultatif) L’offre associée à l’abonnement. La valeur par défaut est MS-AZR-0003P (paiement à l’utilisation).|
|-Currency|(Facultatif) La devise dans laquelle le coût est indiqué dans le rapport généré. La valeur par défaut est le Dollar américain ($) ou la dernière devise utilisée. Reportez-vous à la liste des [devises prises en charge](hyper-v-deployment-planner-cost-estimation.md#supported-currencies).|

Par défaut, l’outil est configuré pour profiler et générer un rapport comprenant jusqu’à 1 000 machines virtuelles. Vous pouvez modifier la limite en modifiant la valeur de la clé MaxVMsSupported dans le fichier ASRDeploymentPlanner.exe.config.
```
<!-- Maximum number of VMs supported-->
<add key="MaxVmsSupported" value="1000"/>
```

### <a name="examples"></a>Exemples
#### <a name="generate-a-report-with-default-values-when-the-profiled-data-is-on-the-local-drive"></a>Générer un rapport contenant des valeurs par défaut lorsque les données profilées sont situées sur le lecteur local
```
ASRDeploymentPlanner.exe -Operation GenerateReport -virtualization Hyper-V -Directory "E:\Hyper-V_ProfiledData" -VMListFile "E:\Hyper-V_ProfiledData\ProfileVMList1.txt"
```

#### <a name="generate-a-report-when-the-profiled-data-is-on-a-remote-server"></a>Générer un rapport lorsque les données profilées sont situées sur un serveur distant
Vous devez disposer d’un accès en lecture/écriture sur le répertoire distant.
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization Hyper-V -Directory "\\PS1-W2K12R2\Hyper-V_ProfiledData" -VMListFile "\\PS1-W2K12R2\vCenter1_ProfiledData\ProfileVMList1.txt"
```

#### <a name="generate-a-report-with-a-specific-bandwidth-that-you-will-provision-for-the-replication"></a>Générer un rapport avec une bande de passante spécifique que vous configurez pour la réplication
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization Hyper-V -Directory "E:\Hyper-V_ProfiledData" -VMListFile "E:\Hyper-V_ProfiledData\ProfileVMList1.txt" -Bandwidth 100
```

#### <a name="generate-a-report-with-a-5-percent-growth-factor-instead-of-the-default-30-percent"></a>Générer un rapport avec un facteur de croissance de 5 pour cent au lieu des 30 pour cent par défaut 
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization Hyper-V -Directory "E:\Hyper-V_ProfiledData" -VMListFile "E:\Hyper-V_ProfiledData\ProfileVMList1.txt" -GrowthFactor 5
```

#### <a name="generate-a-report-with-a-subset-of-profiled-data"></a>Générer un rapport contenant un sous-ensemble de données profilées
Par exemple, vous disposez de 30 jours de données profilées et vous souhaitez générer un rapport pour une période de 20 jours seulement.
```
ASRDeploymentPlanner.exe -Operation GenerateReport -virtualization Hyper-V -Directory "E:\Hyper-V_ProfiledData" -VMListFile "E:\Hyper-V_ProfiledData\ProfileVMList1.txt" -StartDate  01-10-2017:12:30 -EndDate 01-19-2017:12:30
```

#### <a name="generate-a-report-for-a-5-minute-rpo"></a>Générer un rapport pour un RPO de 5 minutes
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization Hyper-V -Directory "E:\Hyper-V_ProfiledData" -VMListFile "E:\Hyper-V_ProfiledData\ProfileVMList1.txt"  -DesiredRPO 5
```

#### <a name="generate-a-report-for-the-south-india-azure-region-with-indian-rupee-and-a-specific-offer-id"></a>Générer un rapport pour la région Azure Inde Sud avec la roupie indienne et l’ID d’offre spécifique
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization Hyper-V -Directory "E:\Hyper-V_ProfiledData" -VMListFile "E:\Hyper-V_ProfiledData\ProfileVMList1.txt"  -SubscriptionID 4d19f16b-3e00-4b89-a2ba-8645edf42fe5 -OfferID MS-AZR-0148P -TargetRegion southindia -Currency INR
```


### <a name="percentile-value-used-for-the-calculation"></a>Valeur de centile utilisée pour le calcul
Lorsque l’outil génère un rapport, il remet la valeur par défaut du centile de 95 en E/S de lecture/écriture par seconde, E/S d’écriture par seconde et l’évolution des données. Ces valeurs sont collectées pendant le profilage de toutes les machines virtuelles. Cette mesure garantit que le pic du centile de 100 que vos machines virtuelles peuvent voir en raison d’événements temporaires n’est pas utilisé pour déterminer vos besoins de compte de stockage cible et de bande passante source. Par exemple, un événement temporaire peut être une tâche de sauvegarde exécutée une fois par jour, une indexation de base de données périodique ou une activité de génération de rapport d’analyse ou un autre événement ponctuel et de courte durée.

L’utilisation de la valeur du centile de 95 permet de donner une image exacte des caractéristiques des charges de travail réelles et vous offre les meilleures performances lorsque les charges de travail s’exécutent sur Azure. Nous ne prévoyons pas de modification de ce numéro. Si vous modifiez la valeur (au centile de 90 par exemple), vous pouvez mettre à jour le fichier de configuration ASRDeploymentPlanner.exe.config dans le dossier par défaut et l’enregistrer pour générer un nouveau rapport sur les données profilées existantes.
```
<add key="WriteIOPSPercentile" value="95" />      
<add key="ReadWriteIOPSPercentile" value="95" />      
<add key="DataChurnPercentile" value="95" />
```

### <a name="considerations-for-growth-factor"></a>Considérations sur le facteur de croissance
Il est essentiel de tenir compte de la croissance dans vos caractéristiques de charge de travail en supposant une augmentation potentielle de l’utilisation au fil du temps. Une fois la protection en place, si les caractéristiques de votre charge de travail changent, vous ne pouvez pas basculer vers un compte de stockage différent pour la protection sans désactiver et réactiver la protection.

Par exemple, supposons que, aujourd’hui, votre machine virtuelle s’intègre dans un compte de réplication de stockage standard. Au cours des trois derniers mois, ces modifications sont susceptibles de se produire :

1. Le nombre d’utilisateurs de l’application qui s’exécute sur la machine virtuelle va augmenter.
2. L’activité accrue sur la machine virtuelle va nécessiter que la machine virtuelle devienne un stockage premium de sorte que la réplication Azure Site Recovery puisse suivre le rythme.
3. Vous devez désactiver et réactiver la protection de compte de stockage premium.

Nous vous recommandons de planifier la croissance pendant la planification du déploiement. Bien que la valeur par défaut soit de 30 pour cent, vous êtes l’expert du modèle d’utilisation et des projections de croissance de votre application. Vous pouvez modifier ce nombre en conséquence pendant la génération d’un rapport. En outre, vous pouvez générer plusieurs rapports avec différents facteurs de croissance et les mêmes données profilées. Vous pouvez ensuite déterminer les meilleures recommandations de stockage cible et de bande passante source pour vous. 

Le rapport Microsoft Excel généré contient les informations suivantes :

* [Résumé local](hyper-v-deployment-planner-analyze-report.md#on-premises-summary)
* [Recommandations](hyper-v-deployment-planner-analyze-report.md#recommendations)
* [VM-Storage Placement](hyper-v-deployment-planner-analyze-report.md#vm-storage-placement-recommendation)
* [Compatible VMs](hyper-v-deployment-planner-analyze-report.md#compatible-vms)
* [Incompatible VMs](hyper-v-deployment-planner-analyze-report.md#incompatible-vms)
* [Exigence de stockage local](hyper-v-deployment-planner-analyze-report.md#on-premises-storage-requirement)
* [Traitement par lot IR](hyper-v-deployment-planner-analyze-report.md#initial-replication-batching)
* [Estimation des coûts](hyper-v-deployment-planner-cost-estimation.md)

![Rapport du planificateur de déploiement](media/hyper-v-deployment-planner-run/deployment-planner-report-h2a.png)


## <a name="get-throughput"></a>GetThroughput
Pour estimer le débit qu’Azure Site Recovery peut atteindre pendant la réplication de données locales vers Azure, exécutez l’outil en mode GetThroughput. L’outil calcule le débit à partir du serveur sur lequel l’outil est exécuté. Dans l’idéal, ce serveur est le serveur Hyper-V dont les machines virtuelles devront être protégées. 

### <a name="command-line-parameters"></a>Paramètres de ligne de commande 
Ouvrez une console de ligne de commande et accédez au dossier pour le planificateur de déploiement Azure Site Recovery. Exécutez ASRDeploymentPlanner.exe avec les paramètres suivants.
```
ASRDeploymentPlanner.exe -Operation GetThroughput /?
```

 Nom du paramètre | Description |
|---|---|
| -Operation | GetThroughput |
|-Virtualization|Le type de virtualisation (VMware ou Hyper-V).|
|-Répertoire|(Facultatif) UNC ou chemin d’accès du répertoire local où les données profilées (fichiers générés lors du profilage) sont stockées. Ces données sont requises pour générer le rapport. Si aucun nom n’est spécifié, le répertoire « ProfiledData » figurant dans le chemin d’accès actuel est utilisé comme répertoire par défaut.|
| -StorageAccountName | Le nom du compte de stockage Azure permettant de déterminer la bande passante utilisée pour la réplication des données locales vers Azure. L’outil charge les données de test sur ce compte de stockage pour trouver la bande passante consommée. Le compte de stockage doit être de type v1 à usage général (GPv1).|
| -StorageAccountKey | La clé du compte de stockage utilisée pour accéder au compte de stockage. Accédez au portail Azure > **Comptes de stockage** > *Nom du compte de stockage* > **Paramètres** > **Clés d’accès** > **Key1**.|
| -VMListFile | Le fichier qui contient la liste des machines virtuelles à profiler pour calculer la bande passante consommée. Le chemin d’accès du fichier peut être absolu ou relatif. Pour Hyper-V, ce fichier est le fichier de sortie de l’opération GetVMList. Si vous préparez manuellement, le fichier doit contenir un nom ou une adresse IP de serveur suivi du nom de machine virtuelle (séparé par un \ par ligne). Le nom de la machine virtuelle spécifié dans le fichier doit être identique au nom de la machine virtuelle sur l’hôte Hyper-V.<br><br>**Exemple :** le fichier VMList.txt contient les machines virtuelles suivantes :<ul><li>Host_1\VM_A</li><li>10.8.59.27\VM_B</li><li>Host_2\VM_C</li><ul>|
|-Environment|(Facultatif) Votre environnement cible pour le compte de stockage Azure. Ce paramètre peut être défini sur l’une des trois valeurs suivantes : AzureCloud, AzureUSGovernment ou AzureChinaCloud. La valeur par défaut est AzureCloud. Utilisez ce paramètre lorsque votre région Azure cible correspond à Azure - Gouvernement des États-Unis ou Azure - Chine 21Vianet.|

### <a name="example"></a>Exemple
```
ASRDeploymentPlanner.exe -Operation GetThroughput -Virtualization Hyper-V -Directory "E:\Hyper-V_ProfiledData" -VMListFile "E:\Hyper-V_ProfiledData\ProfileVMList1.txt"  -StorageAccountName  asrspfarm1 -StorageAccountKey by8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==
```

### <a name="throughput-considerations"></a>Considérations de débit

L’outil crée plusieurs fichiers asrvhdfile*nombre*.vhd de 64 Mo (où *nombre* représente le nombre de fichiers) dans le répertoire spécifié. L’outil charge ces fichiers sur le compte de stockage pour déterminer le débit. Une fois le débit mesuré, l’outil supprime tous les fichiers du compte de stockage et du serveur local. Si l’outil est interrompu pour une raison quelconque alors qu’il calcule le débit, cela ne supprime pas les fichiers du compte de stockage ou du serveur local. Vous devez les supprimer manuellement.

Le débit est mesuré à un point spécifié dans le temps. Il s’agit du débit maximal que Azure Site Recovery peut atteindre lors de la réplication, si tous les autres facteurs restent les mêmes. Par exemple, si une application commence à consommer davantage de bande passante sur le même réseau, le débit réel varie pendant la réplication. Le résultat du débit mesuré est différent si l’opération GetThroughput est exécutée lorsque les machines virtuelles protégées présentent des taux d’activité élevés. 

Pour comprendre les niveaux de débit pouvant être atteints à des moments différents, nous vous recommandons d’exécuter l’outil à différents moments dans le temps pendant le profilage. Dans le rapport, l’outil affiche le dernier débit mesuré.

> [!NOTE]
> Exécutez l’outil sur un serveur doté des mêmes caractéristiques de stockage et de processeur qu’un serveur Hyper-V.

Pour la réplication, définissez la bande passante recommandée pour atteindre le RPO souhaité en permanence. Après avoir défini la bande passante appropriée, si vous ne constatez pas d’augmentation du débit atteint signalé par l’outil, vérifiez les points suivants :

1. Vérifiez si un problème de Qualité de Service (QoS) du réseau limite le débit Azure Site Recovery.
2. Vérifiez si votre coffre Azure Site Recovery se trouve dans la région Microsoft Azure physique prise en charge la plus proche pour réduire la latence du réseau.
3. Vérifiez les caractéristiques de stockage local pour déterminer si vous pouvez améliorer le matériel (par exemple, passer d’un disque dur à un disque SSD).

    
## <a name="next-steps"></a>Étapes suivantes
* [Analyser le rapport généré](hyper-v-deployment-planner-analyze-report.md)
