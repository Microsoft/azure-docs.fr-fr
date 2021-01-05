---
title: Exécuter le planificateur de déploiement pour la récupération d’urgence de VMware avec Azure Site Recovery
description: Cet article décrit comment exécuter le planificateur de déploiement Azure Site Recovery pour la reprise d’activité de VMware sur Azure.
author: rajeswari-mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 4/15/2019
ms.author: ramamill
ms.openlocfilehash: 05d260de726c62c130a58938c2a2c9fa2440a96d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004719"
---
# <a name="run-the-deployment-planner-for-vmware-disaster-recovery"></a>Exécuter le planificateur de déploiement pour la récupération d’urgence de VMware
Cet article est le guide de l’utilisateur d’Azure Site Recovery Deployment Planner portant sur les déploiements de production de VMware vers Azure.


## <a name="modes-of-running-deployment-planner"></a>Modes d’exécution du planificateur de déploiement
Vous pouvez exécuter l’outil de ligne de commande (ASRDeploymentPlanner.exe) dans l’un des trois modes suivants :

1.  [Profilage](#profile-vmware-vms)
2.  [Génération de rapport](#generate-report)
3.  [Obtenir le débit](#get-throughput)

Tout d’abord, exécutez l’outil en mode profilage pour rassembler l’activité des données de machine virtuelle et les E/S par seconde. Ensuite, exécutez l’outil pour générer le rapport afin de déterminer les besoins de bande passante réseau et de stockage et les coûts de la récupération d’urgence.

## <a name="profile-vmware-vms"></a>Profilage des machines virtuelles VMware
En mode profilage, l’outil Deployment Planner se connecte au serveur vCenter ou à l’hôte vSphere ESXi pour collecter les données de performance sur la machine virtuelle.

* Le profilage n’affecte pas les performances des machines virtuelles de production, car aucune connexion directe n’est établie. Toutes les données de performance sont collectées à partir du serveur vCenter/de l’hôte vSphere ESXi.
* Pour s’assurer que les effets du profilage sur le serveur sont négligeables, l’outil interroge l’hôte du serveur vCenter/de l’hôte vSphere ESXi toutes les 15 minutes. Cet intervalle de requête ne compromet pas la précision du profilage, car l’outil stocke les données du compteur de performances toutes les minutes.

### <a name="create-a-list-of-vms-to-profile"></a>Créer une liste de machines virtuelles à profiler
Tout d’abord, vous avez besoin d’une liste des machines virtuelles à profiler. Vous pouvez obtenir tous les noms des machines virtuelles sur un serveur vCenter/hôte vSphere ESXi à l’aide des commandes VMware vSphere PowerCLI indiquées dans la procédure suivante. Vous pouvez également répertorier dans un fichier les noms conviviaux ou adresses IP des machines virtuelles que vous souhaitez profiler manuellement.

1. Connectez-vous à la machine virtuelle sur laquelle VMware vSphere PowerCLI est installé.
2. Ouvrez la console VMware vSphere PowerCLI.
3. Assurez-vous que la stratégie d’exécution est activée pour le script. Si elle est désactivée, lancez la console VMware vSphere PowerCLI en mode administrateur, puis activez-la en exécutant la commande suivante :

    ```powershell
    Set-ExecutionPolicy –ExecutionPolicy AllSigned
    ```

4. Vous pourriez avoir besoin d’exécuter la commande suivante, dans le cas où Connect-VIServer ne serait pas reconnu comme nom de cmdlet.

    ```powershell
    Add-PSSnapin VMware.VimAutomation.Core
    ```

5. Pour obtenir tous les noms des machines virtuelles sur un serveur vCenter/hôte vSphere ESXi et enregistrer la liste dans un fichier .txt, exécutez les deux commandes répertoriées ici.
Remplacez &lsaquo;server name&rsaquo;, &lsaquo;user name&rsaquo;, &lsaquo;password&rsaquo; et &lsaquo;outputfile.txt&rsaquo; par vos entrées.

    ```powershell
    Connect-VIServer -Server <server name> -User <user name> -Password <password>

    Get-VM |  Select Name | Sort-Object -Property Name >  <outputfile.txt>
    ```

6. Ouvrez le fichier de sortie dans le bloc-notes et copiez les noms de toutes les machines virtuelles que vous souhaitez profiler dans un autre fichier (par exemple, ProfileVMList.txt), en indiquant un nom de machine virtuelle par ligne. Ce fichier est utilisé comme entrée pour le paramètre *-VMListFile* de l’outil de ligne de commande.

    ![Liste des noms de machines virtuelles dans Deployment planner
](media/site-recovery-vmware-deployment-planner-run/profile-vm-list-v2a.png)

### <a name="start-profiling"></a>Démarrer le profilage
Après avoir établi la liste des machines virtuelles à profiler, vous pouvez exécuter l’outil en mode profilage. Voici la liste des paramètres obligatoires et facultatifs de l’outil à exécuter en mode profilage.

```
ASRDeploymentPlanner.exe -Operation StartProfiling /?
```

| Nom du paramètre | Description |
|---|---|
| -Operation | StartProfiling |
| -Server | Le nom de domaine complet ou l’adresse IP du serveur vCenter/de l’hôte vSphere ESXi dont les machines virtuelles doivent être profilées.|
| -User | Le nom d’utilisateur pour se connecter au serveur vCenter/à l’hôte vSphere ESXi. L’utilisateur doit disposer au moins d’un accès en lecture seule.|
| -VMListFile | Le fichier qui contient la liste des machines virtuelles à profiler. Le chemin d’accès du fichier peut être absolu ou relatif. Le fichier doit contenir un nom/une adresse IP de machine virtuelle par ligne. Le nom de la machine virtuelle spécifié dans le fichier doit être identique au nom de la machine virtuelle sur le serveur vCenter/l’hôte vSphere ESXi.<br>Par exemple, le fichier VMList.txt contient les machines virtuelles suivantes :<ul><li>virtual_machine_A</li><li>10.150.29.110</li><li>virtual_machine_B</li><ul> |
|-NoOfMinutesToProfile|Le nombre de minutes pendant lesquelles le profilage doit être exécuté. La valeur minimale est de 30 minutes.|
|-NoOfHoursToProfile|Le nombre d’heures pendant lesquelles le profilage doit être exécuté.|
| -NoOfDaysToProfile | Le nombre de jours pendant lesquels le profilage doit être exécuté. Nous vous recommandons d’exécuter le profilage pendant plus de 7 jours pour vous assurer que le modèle de charge de travail de votre environnement sur la période spécifiée est observé et utilisé pour fournir une recommandation précise. |
|-Virtualization|Spécifiez le type de virtualisation (VMware ou Hyper-V).|
| -Répertoire | (Facultatif) La convention d’appellation universelle (UNC) ou le chemin d’accès du répertoire local pour stocker les données de profilage générées pendant le profilage. Si aucun nom de répertoire n’est spécifié, le répertoire ProfiledData figurant dans le chemin d’accès actuel est utilisé comme répertoire par défaut. |
| -Mot de passe | (Facultatif) Le mot de passe utilisé pour se connecter au serveur vCenter/à l’hôte vSphere ESXi. Si vous spécifiez aucun mot de passe maintenant, vous êtes invité à l’indiquer à l’exécution de la commande.|
|-Port|(Facultatif) Numéro de port pour se connecter à l’hôte vCenter/ESXi. Le port par défaut est 443.|
|-Protocol| (Facultatif) Spécifie le protocole « http » ou « https » pour vous connecter à vCenter. Le protocole par défaut est https.|
| -StorageAccountName | (Facultatif) Le nom du compte de stockage utilisé pour rechercher le débit réalisable pour la réplication des données locales vers Azure. L’outil charge les données de test sur ce compte de stockage pour calculer le débit. Le compte de stockage doit être de type v1 à usage général (GPv1). |
| -StorageAccountKey | (Facultatif) La clé du compte de stockage utilisée pour accéder au compte de stockage. Accédez au portail Azure > Comptes de stockage > <*Nom du compte de stockage*> > Paramètres > Clés d’accès > Key1. |
| -Environment | (Facultatif) Votre environnement de compte Stockage Azure cible. Ce paramètre peut être défini sur l’une des trois valeurs suivantes : AzureCloud, AzureUSGovernment, AzureChinaCloud. La valeur par défaut est AzureCloud. Utilisez ce paramètre lorsque votre région Azure cible correspond à Azure - Gouvernement des États-Unis ou Azure - Chine 21Vianet. |


Nous vous recommandons de profiler vos machines virtuelles pendant plus de 7 jours. Si le modèle d’activité varie durant un mois, nous vous recommandons d’effectuer le profilage au cours de la semaine lors de l’activité maximale. La meilleure solution est d’effectuer le profilage pendant 31 jours pour obtenir de meilleures recommandations. Pendant la période de profilage, ASRDeploymentPlanner.exe continue de s’exécuter. Les entrées du temps de profilage de l’outil sont indiquées en jours. Pour un test rapide de l’outil ou pour des preuves de concept, vous pouvez faire un profilage pendant quelques heures ou minutes. Le temps de profilage minimum autorisé est de 30 minutes.

Pendant le profilage, vous pouvez éventuellement transmettre un nom et une clé du compte de stockage pour déterminer le débit que Site Recovery peut atteindre au moment de la réplication du serveur de configuration ou du serveur de processus vers Azure. Si la clé et le nom du compte de stockage ne sont pas transmis au cours du profilage, l’outil ne calcule pas le débit réalisable.

Vous pouvez exécuter plusieurs instances de l’outil pour différents ensembles de machines virtuelles. Veillez à ce que les noms des machines virtuelles ne soient pas répétés dans les ensembles de profilage. Exemple : si vous avez profilé dix machines virtuelles (VM1 à VM10) et que, après quelques jours, vous voulez profiler cinq autres machines virtuelles (VM11 à VM15) ; vous pouvez exécuter l’outil à partir d’une autre console de ligne de commande pour le second ensemble de machines virtuelles (VM11 à VM15). Assurez-vous que le second ensemble de machines virtuelles ne comporte pas de noms de machine virtuelle de la première instance de profilage ou utilisez un autre répertoire de sortie pour la seconde exécution. Si deux instances de l’outil sont utilisées pour profiler les mêmes machines virtuelles et que vous utilisez le même répertoire de sortie, le rapport généré sera incorrect.

Par défaut, l'outil est configuré pour profiler et générer un rapport comprenant jusqu'à 1 000 machines virtuelles. Vous pouvez modifier la limite en modifiant la valeur de la clé MaxVMsSupported dans le fichier *ASRDeploymentPlanner.exe.config*.
```
<!-- Maximum number of vms supported-->
<add key="MaxVmsSupported" value="1000"/>
```
Avec les paramètres par défaut, pour profiler 1500 machines virtuelles, créez deux fichiers VMList.txt. Un fichier avec 1 000 machines virtuelles et un autre avec une liste de 500 machines virtuelles. Exécutez les deux instances du planificateur de déploiement Azure Site Recovery, une avec le fichier VMList1.txt et l’autre avec le fichier VMList2.txt. Vous pouvez utiliser le même chemin d’accès de répertoire pour stocker les données profilées des machines virtuelles correspondant aux deux fichiers VMList.

Nous avons vu que, selon la configuration matérielle et particulière en fonction de la mémoire RAM du serveur à partir duquel l’outil pour générer le rapport est exécuté, l’opération peut échouer à cause d’une quantité de mémoire insuffisante. Si vous avez un bon matériel, vous pouvez modifier la clé MaxVMsSupported avec n’importe quelle valeur supérieure.  

Si vous avez plusieurs serveurs vCenter, vous devez exécuter une instance de ASRDeploymentPlanner pour le profilage de chaque serveur vCenter.

Les configurations de machines virtuelles sont capturées une fois au début de l’opération de profilage et stockées dans un fichier appelé VMDetailList.xml. Ces informations sont utilisées lorsque le rapport est généré. Toute modification de configuration de machine virtuelle (par exemple, un nombre accru de cœurs, de disques ou de cartes réseau) du début à la fin du profilage n’est pas capturée. Si une configuration de machines virtuelles profilées est modifiée en cours de profilage, la solution de contournement dans la préversion publique consiste à obtenir les toutes dernières informations des machines virtuelles lorsque vous générez le rapport :

* Sauvegardez VMdetailList.xml et supprimez le fichier de son emplacement actuel.
* Transmettez les arguments -User et -Password au moment de la génération du rapport.

La commande de profilage génère plusieurs fichiers dans le répertoire de profilage. Ne supprimez pas les fichiers, car cela affecterait la génération de rapports.

#### <a name="example-1-profile-vms-for-30-days-and-find-the-throughput-from-on-premises-to-azure"></a>Exemple 1 : Profiler des machines virtuelles pendant 30 jours et déterminer le débit pour le scénario « local vers Azure »
```
ASRDeploymentPlanner.exe -Operation StartProfiling -Virtualization VMware -Directory “E:\vCenter1_ProfiledData” -Server vCenter1.contoso.com -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt”  -NoOfDaysToProfile  30  -User vCenterUser1 -StorageAccountName  asrspfarm1 -StorageAccountKey Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==
```

#### <a name="example-2-profile-vms-for-15-days"></a>Exemple 2 : Profiler des machines virtuelles pendant 15 jours

```
ASRDeploymentPlanner.exe -Operation StartProfiling -Virtualization VMware -Directory “E:\vCenter1_ProfiledData” -Server vCenter1.contoso.com -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt” -NoOfDaysToProfile  15  -User vCenterUser1
```

#### <a name="example-3-profile-vms-for-60-minutes-for-a-quick-test-of-the-tool"></a>Exemple 3 : Profiler des machines virtuelles pendant 60 minutes pour tester rapidement l’outil
```
ASRDeploymentPlanner.exe -Operation StartProfiling -Virtualization VMware -Directory “E:\vCenter1_ProfiledData” -Server vCenter1.contoso.com -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt”  -NoOfMinutesToProfile 60  -User vCenterUser1
```

#### <a name="example-4-profile-vms-for-2-hours-for-a-proof-of-concept"></a>Exemple 4 : Profiler des machines virtuelles pendant 2 heures pour une preuve de concept
```
ASRDeploymentPlanner.exe -Operation StartProfiling -Virtualization VMware -Directory “E:\vCenter1_ProfiledData” -Server vCenter1.contoso.com -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt” -NoOfHoursToProfile 2 -User vCenterUser1
```

>[!NOTE]
>
>* Si le serveur sur lequel s’exécute l’outil est redémarré ou est tombé en panne, ou si vous fermez l’outil à l’aide de Ctrl + C, les données profilées sont conservées. Cependant, il existe un risque de manquer les 15 dernières minutes de données profilées. Dans ce cas, réexécutez l’outil en mode profilage après le redémarrage du serveur.
>* Lorsque le nom et la clé du compte de stockage sont transmis, l’outil mesure le débit au cours de la dernière étape du profilage. Si l’outil est fermé avant la fin du profilage, le débit n’est pas calculé. Pour trouver le débit avant de générer le rapport, vous pouvez exécuter l’opération GetThroughput à partir de la console de ligne de commande. Autrement, le rapport généré ne contient pas les informations de débit.


## <a name="generate-report"></a>Générer le rapport
L’outil génère un fichier Microsoft Excel avec les macros activées (fichier XLSM) en tant que sortie du rapport, qui résume toutes les recommandations de déploiement. Le rapport porte le nom `DeploymentPlannerReport_<unique numeric identifier>.xlsm` et se trouve dans le répertoire spécifié.

>[!NOTE]
>La génération de rapports nécessite un PC Windows ou Windows Server avec Excel 2013 ou version ultérieure. Le symbole décimal configuré sur cet ordinateur doit être «. » pour produire les devis. Si vous avez configuré « , » comme symbole décimal, accédez à « Changer les formats de date, d’heure ou de nombre » dans le Panneau de configuration, puis accédez à « Paramètres supplémentaires » pour changer le symbole décimal en « . ».

À l’issue du profilage, vous pouvez exécuter l’outil en mode génération de rapport. Le tableau suivant contient une liste des paramètres obligatoires et facultatifs de l’outil à exécuter en mode génération de rapport.

`ASRDeploymentPlanner.exe -Operation GenerateReport /?`

|Nom du paramètre | Description |
|-|-|
| -Operation | GenerateReport |
| -Server |  Le nom de domaine complet ou l’adresse IP du serveur vCenter/vSphere (utilisez le même nom ou la même adresse IP que ceux utilisés lors du profilage) sur lequel se trouvent les machines virtuelles profilées dont le rapport doit être généré. Notez que si vous avez utilisé un serveur vCenter au moment du profilage, vous ne pouvez pas utiliser un serveur vSphere pour la génération de rapport, et inversement.|
| -VMListFile | Le fichier qui contient la liste des machines virtuelles profilées pour lesquels le rapport va être généré. Le chemin d’accès du fichier peut être absolu ou relatif. Le fichier doit contenir un nom ou une adresse IP de machine virtuelle par ligne. Les noms de machines virtuelles spécifiés dans le fichier doivent être identiques à ceux des machines virtuelles sur le serveur vCenter/l’hôte vSphere ESXi, et ils correspondent à ce qui a été utilisé au moment du profilage.|
|-Virtualization|Spécifiez le type de virtualisation (VMware ou Hyper-V).|
| -Répertoire | (Facultatif) UNC ou chemin d’accès du répertoire local où les données profilées (fichiers générés lors du profilage) sont stockées. Ces données sont requises pour générer le rapport. Si aucun nom n’est spécifié, le répertoire ProfiledData est utilisé. |
| -GoalToCompleteIR | (Facultatif) Le nombre d’heures pendant lesquelles la réplication initiale des machines virtuelles profilées doit être effectuée. Le rapport généré indique le nombre de machines virtuelles pour lesquelles la réplication initiale peut être effectuée dans le délai spécifié. La valeur par défaut est 72 heures. |
| -User | (Facultatif) Le nom d’utilisateur permettant de se connecter au serveur vCenter/vSphere. Le nom est utilisé pour extraire les dernières informations de configuration des machines virtuelles, comme le nombre de disques, le nombre de cœurs, le nombre de cartes réseau, etc. à utiliser dans le rapport. Si aucun nom n’est fourni, les informations de configuration collectées au début du profilage sont utilisées. |
| -Mot de passe | (Facultatif) Le mot de passe utilisé pour se connecter au serveur vCenter/à l’hôte vSphere ESXi. Si le mot de passe n’est pas spécifié en tant que paramètre, vous êtes invité à l’indiquer ultérieurement à l’exécution de la commande. |
|-Port|(Facultatif) Numéro de port pour se connecter à l’hôte vCenter/ESXi. Le port par défaut est 443.|
|-Protocol|(Facultatif) Spécifie le protocole « http » ou « https » pour vous connecter à vCenter. Le protocole par défaut est https.|
| -DesiredRPO | (Facultatif) L’objectif de point de récupération souhaité, en minutes. La valeur par défaut est 15 minutes.|
| -Bandwidth | Bande passante en Mbits/s. Le paramètre permet de calculer le RPO qui peut être atteint pour la bande passante spécifiée. |
| -StartDate | (Facultatif) La date et l’heure de début au format MM-JJ-AAAA:HH:MM (24 heures). Le paramètre *StartDate* doit être spécifié avec le paramètre *EndDate*. Si le paramètre StartDate est spécifié, le rapport est généré pour les données profilées collectées entre les dates StartDate et EndDate. |
| -EndDate | (Facultatif) La date et l’heure de fin au format MM-JJ-AAAA:HH:MM (24 heures). Le paramètre *EndDate* doit être spécifié avec le paramètre *StartDate*. Si le paramètre EndDate est spécifié, le rapport est généré pour les données profilées collectées entre les dates StartDate et EndDate. |
| -GrowthFactor | (Facultatif) Le facteur de croissance, exprimé en pourcentage. La valeur par défaut est 30 pour cent. |
| -UseManagedDisks | (Facultatif) UseManagedDisks - Oui/Non. La valeur par défaut est Oui. Le nombre de machines qu’il est possible de placer dans un compte de stockage unique est calculé en fonction de si le basculement/test de basculement des machines virtuelles est effectué sur un disque managé au lieu d’un disque non managé. |
|-SubscriptionId |(Facultatif) GUID de l’abonnement. Notez que ce paramètre est nécessaire quand vous devez générer le rapport d’estimation des coûts avec le prix le plus récent en fonction de votre abonnement, de l’offre associée à votre abonnement et de votre région Azure cible dans la **devise spécifiée**.|
|-TargetRegion|(Facultatif) La région Azure cible de la réplication. Étant donné qu’Azure possède différents coûts par région, utilisez ce paramètre pour générer des rapports avec une région cible Azure spécifique.<br>La valeur par défaut est WestUS2 (Ouest des États-Unis) ou la dernière région cible.<br>Reportez-vous à la liste des [régions cibles prises en charge](site-recovery-vmware-deployment-planner-cost-estimation.md#supported-target-regions).|
|-OfferId|(Facultatif) L’offre associée à l’abonnement donné. La valeur par défaut est MS-AZR-0003P (paiement à l’utilisation).|
|-Currency|(Facultatif) La devise dans laquelle le coût est indiqué dans le rapport généré. La valeur par défaut est le Dollar américain ($) ou la dernière devise utilisée.<br>Reportez-vous à la liste des [devises prises en charge](site-recovery-vmware-deployment-planner-cost-estimation.md#supported-currencies).|

Par défaut, l'outil est configuré pour profiler et générer un rapport comprenant jusqu'à 1 000 machines virtuelles. Vous pouvez modifier la limite en modifiant la valeur de la clé MaxVMsSupported dans le fichier *ASRDeploymentPlanner.exe.config*.
```xml
<!-- Maximum number of vms supported-->
<add key="MaxVmsSupported" value="1000"/>
```

#### <a name="example-1-generate-a-report-with-default-values-when-the-profiled-data-is-on-the-local-drive"></a>Exemple 1 : Générer un rapport contenant des valeurs par défaut lorsque les données profilées sont situées sur le lecteur local
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware -Server vCenter1.contoso.com -Directory “E:\vCenter1_ProfiledData” -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt”
```

#### <a name="example-2-generate-a-report-when-the-profiled-data-is-on-a-remote-server"></a>Exemple 2 : Générer un rapport lorsque les données profilées sont situées sur un serveur distant
Vous devez disposer d’un accès en lecture/écriture sur le répertoire distant.
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware -Server vCenter1.contoso.com -Directory “\\PS1-W2K12R2\vCenter1_ProfiledData” -VMListFile “\\PS1-W2K12R2\vCenter1_ProfiledData\ProfileVMList1.txt”
```

#### <a name="example-3-generate-a-report-with-a-specific-bandwidth-and-goal-to-complete-ir-within-specified-time"></a>Exemple 3 : Générer un rapport contenant la bande passante spécifique et les objectifs pour terminer IR dans le temps indiqué
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware -Server vCenter1.contoso.com -Directory “E:\vCenter1_ProfiledData” -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt” -Bandwidth 100 -GoalToCompleteIR 24
```

#### <a name="example-4-generate-a-report-with-a-5-percent-growth-factor-instead-of-the-default-30-percent"></a>Exemple 4 : Générer un rapport avec un facteur de croissance de 5 pour cent au lieu des 30 pour cent par défaut
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware -Server vCenter1.contoso.com -Directory “E:\vCenter1_ProfiledData” -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt” -GrowthFactor 5
```

#### <a name="example-5-generate-a-report-with-a-subset-of-profiled-data"></a>Exemple 5 : Générer un rapport contenant un sous-ensemble de données profilées
Par exemple, vous disposez de 30 jours de données profilées et vous souhaitez générer un rapport pour une période de 20 jours seulement.
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware -Server vCenter1.contoso.com -Directory “E:\vCenter1_ProfiledData” -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt” -StartDate  01-10-2017:12:30 -EndDate 01-19-2017:12:30
```

#### <a name="example-6-generate-a-report-for-5-minute-rpo"></a>Exemple 6 : Générer un rapport pour un RPO de 5 minutes
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware -Server vCenter1.contoso.com -Directory “E:\vCenter1_ProfiledData” -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt”  -DesiredRPO 5
```

#### <a name="example-7-generate-a-report-for-south-india-azure-region-with-indian-rupee-and-specific-offer-id"></a>Exemple 7 : Générer un rapport pour la région Azure Inde Sud avec la roupie indienne et l’ID d’offre spécifique

Notez que l’ID d’abonnement est nécessaire pour générer le rapport des coûts dans une devise spécifique.
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware  -Directory “E:\vCenter1_ProfiledData” -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt”  -SubscriptionID 4d19f16b-3e00-4b89-a2ba-8645edf42fe5 -OfferID MS-AZR-0148P -TargetRegion southindia -Currency INR
```

## <a name="percentile-value-used-for-the-calculation"></a>Valeur de centile utilisée pour le calcul
**Quelle valeur de centile par défaut des indicateurs de performance collectés pendant le profilage l’outil va-t-il utiliser au moment de la génération de rapport ?**

L’outil est configuré par défaut sur les valeurs du 95e centile des E/S par seconde de lecture/écriture, des E/S par seconde d’écriture et de l’activité des données collectées pendant le profilage de toutes les machines virtuelles. Cette mesure garantit que le pic du 100e centile que vos machines virtuelles peuvent voir en raison d’événements temporaires n’est pas utilisé pour déterminer vos besoins de compte de stockage cible et de bande passante source. Par exemple, un événement temporaire peut être une tâche de sauvegarde exécutée une fois par jour, une indexation de base de données périodique ou une activité de génération de rapport d’analyse ou d’autres événements similaires ponctuels et de courte durée.

L’utilisation des valeurs du 95e centile permet de donner une image exacte des caractéristiques des charges de travail réelles et vous offre les meilleures performances lorsque les charges de travail s’exécutent sur Azure. Nous ne prévoyons pas que vous modifiiez ce numéro. Si vous modifiez la valeur (au 90e centile par exemple), vous pouvez mettre à jour le fichier de configuration *ASRDeploymentPlanner.exe.config* dans le dossier par défaut et l’enregistrer pour générer un nouveau rapport sur les données profilées existantes.
```xml
<add key="WriteIOPSPercentile" value="95" />      
<add key="ReadWriteIOPSPercentile" value="95" />      
<add key="DataChurnPercentile" value="95" />
```

## <a name="growth-factor-considerations"></a>Considérations relatives au facteur de croissance
**Pourquoi devrais-je tenir compte du facteur de croissance lors de la planification de déploiement ?**

Il est essentiel de tenir compte de la croissance dans vos caractéristiques de charge de travail en supposant une augmentation potentielle de l’utilisation au fil du temps. Une fois la protection en place, si les caractéristiques de votre charge de travail changent, vous ne pouvez pas basculer vers un compte de stockage différent pour la protection sans désactiver et réactiver la protection.

Par exemple, supposons que, aujourd’hui, votre machine virtuelle s’intègre dans un compte de réplication de stockage standard. Au cours des trois derniers mois, plusieurs modifications sont susceptibles de se produire :

* Le nombre d’utilisateurs de l’application qui s’exécute sur la machine virtuelle va augmenter.
* L’activité accrue en résultant sur la machine virtuelle va nécessiter que la machine virtuelle devienne un stockage premium de sorte que la réplication Site Recovery puisse suivre le rythme.
* Par conséquent, vous devez désactiver et réactiver la protection de compte de stockage premium.

Nous vous recommandons de planifier la croissance pendant la planification du déploiement et pendant que la valeur par défaut est 30 pour cent. Vous êtes expert du modèle d’utilisation de vos applications et des projections de croissance et vous pouvez modifier ce nombre en conséquence lors de la génération d’un rapport. En outre, vous pouvez générer plusieurs rapports avec différents facteurs de croissance avec les mêmes données profilées et déterminer les recommandations de stockage cible et de bande passante source fonctionnent le mieux pour vous.

Le rapport Microsoft Excel généré contient les informations suivantes :

* [Résumé local](site-recovery-vmware-deployment-planner-analyze-report.md#on-premises-summary)
* [Recommandations](site-recovery-vmware-deployment-planner-analyze-report.md#recommendations)
* [VM&lt;-&gt;Storage Placement](site-recovery-vmware-deployment-planner-analyze-report.md#vm-storage-placement)
* [Compatible VMs](site-recovery-vmware-deployment-planner-analyze-report.md#compatible-vms)
* [Incompatible VMs](site-recovery-vmware-deployment-planner-analyze-report.md#incompatible-vms)
* [Estimation des coûts](site-recovery-vmware-deployment-planner-cost-estimation.md)

![Deployment planner](media/site-recovery-vmware-deployment-planner-analyze-report/Recommendations-v2a.png)

## <a name="get-throughput"></a>GetThroughput

Pour estimer le débit que Site Recovery peut atteindre pendant la réplication de données locales vers Azure, exécutez l’outil en mode GetThroughput. L’outil calcule le débit à partir du serveur sur lequel l’outil est exécuté. Idéalement, ce serveur est basé sur le guide de dimensionnement des serveurs de configuration. Si vous avez déjà déployé les composants d’infrastructure Site Recovery en local, exécutez l’outil sur le serveur de configuration.

Ouvrez une console de ligne de commande et accédez au dossier de l’outil de planification du déploiement Site Recovery. Exécutez ASRDeploymentPlanner.exe avec les paramètres ci-dessous.

`ASRDeploymentPlanner.exe -Operation GetThroughput /?`

|Nom du paramètre | Description |
|-|-|
| -Operation | GetThroughput |
|-Virtualization|Spécifiez le type de virtualisation (VMware ou Hyper-V).|
| -Répertoire | (Facultatif) UNC ou chemin d’accès du répertoire local où les données profilées (fichiers générés lors du profilage) sont stockées. Ces données sont requises pour générer le rapport. Si aucun nom de répertoire n’est spécifié, le répertoire ProfiledData est utilisé. |
| -StorageAccountName | Le nom du compte de stockage Azure permettant de déterminer la bande passante utilisée pour la réplication des données locales vers Azure. L’outil charge les données de test sur ce compte de stockage pour trouver la bande passante consommée. Le compte de stockage doit être de type v1 à usage général (GPv1).|
| -StorageAccountKey | La clé du compte de stockage utilisée pour accéder au compte de stockage. Accédez au portail Azure > Comptes de stockage > <*Nom du compte de stockage*> > Paramètres > Clés d’accès > Key1 (ou clé d’accès principale pour un compte de stockage classique). |
| -VMListFile | Le fichier qui contient la liste des machines virtuelles à profiler pour calculer la bande passante consommée. Le chemin d’accès du fichier peut être absolu ou relatif. Le fichier doit contenir un nom/une adresse IP de machine virtuelle par ligne. Les noms de machine virtuelle spécifiés dans le fichier doivent être identiques au nom des machines virtuelles sur le serveur vCenter/l’hôte vSphere ESXi.<br>Par exemple, le fichier VMList.txt contient les machines virtuelles suivantes :<ul><li>VM_A</li><li>10.150.29.110</li><li>VM_B</li></ul>|
| -Environment | (Facultatif) Votre environnement de compte Stockage Azure cible. Ce paramètre peut être défini sur l’une des trois valeurs suivantes : AzureCloud, AzureUSGovernment, AzureChinaCloud. La valeur par défaut est AzureCloud. Utilisez ce paramètre lorsque votre région Azure cible correspond à Azure - Gouvernement des États-Unis ou Azure - Chine 21Vianet. |

L’outil crée plusieurs fichiers asrvhdfile<#>.vhd de 64 Mo (où # représente le nombre de fichiers) dans le répertoire spécifié. L’outil charge ces fichiers sur le compte de stockage pour déterminer le débit. Une fois le débit mesuré, l’outil supprime tous les fichiers du compte de stockage et du serveur local. Si l’outil est interrompu pour une raison quelconque alors qu’il calcule le débit, cela ne supprime pas les fichiers du stockage ou du serveur local. Vous devrez les supprimer manuellement.

Le débit est mesuré à un moment donné et il s’agit du débit maximal que Site Recovery peut atteindre lors d’une réplication, sous réserve que tous les autres facteurs restent identiques. Par exemple, si une application commence à consommer davantage de bande passante sur le même réseau, le débit réel varie pendant la réplication. Si vous exécutez la commande GetThroughput à partir d’un serveur de configuration, l’outil ne reconnaît pas les machines virtuelles protégées ni la réplication en cours. Le résultat du débit mesuré est différent si l’opération GetThroughput est exécutée lorsque les machines virtuelles protégées présentent des taux d’activité élevés. Nous vous recommandons d’exécuter l’outil à différents moments dans le temps pendant le profilage pour comprendre les niveaux de débit pouvant être atteints à des moments différents. Dans le rapport, l’outil affiche le dernier débit mesuré.

### <a name="example"></a>Exemple
```
ASRDeploymentPlanner.exe -Operation GetThroughput -Directory  E:\vCenter1_ProfiledData -Virtualization VMware -VMListFile E:\vCenter1_ProfiledData\ProfileVMList1.txt  -StorageAccountName  asrspfarm1 -StorageAccountKey by8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==
```

>[!NOTE]
>
> Exécutez l’outil sur un serveur doté des mêmes caractéristiques de stockage et de processeur que le serveur de configuration.
>
> Pour la réplication, définissez la bande passante recommandée pour atteindre le RPO souhaité en permanence. Après avoir défini la bande passante appropriée, si vous ne constatez pas d’augmentation du débit atteint signalé par l’outil, vérifiez les points suivants :
>
>  1. Vérifiez si un réseau Qualité de Service (QoS) limite le débit Site Recovery.
>
>  2. Vérifiez si votre coffre Site Recovery se trouve dans la région Microsoft Azure physique prise en charge la plus proche pour réduire la latence du réseau.
>
>  3. Vérifiez les caractéristiques de stockage local pour déterminer si vous pouvez améliorer le matériel (par exemple, passer d’un disque dur à un disque SSD).
>
>  4. Modifiez les paramètres Site Recovery sur le serveur de processus pour [augmenter la quantité de bande passante réseau utilisée pour la réplication](./site-recovery-plan-capacity-vmware.md#control-network-bandwidth).

## <a name="next-steps"></a>Étapes suivantes
* [Analysez le rapport généré](site-recovery-vmware-deployment-planner-analyze-report.md).
