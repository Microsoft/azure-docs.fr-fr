---
title: Outil de diagnostic pour la résolution des problèmes de l’appareil StorSimple 8000 | Microsoft Docs
description: Cet article offre une vue d’ensemble de l’outil de diagnostic et explique comment utiliser cet outil avec un appareil StorSimple.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/09/2018
ms.author: alkohli
ms.openlocfilehash: bf0570cd62a2c329407cfb6dd087020f53cdc991
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94954067"
---
# <a name="use-the-storsimple-diagnostics-tool-to-troubleshoot-8000-series-device-issues"></a>Utiliser l’outil de diagnostic StorSimple pour résoudre les problèmes des appareils de la gamme 8000

## <a name="overview"></a>Vue d’ensemble

L’outil de diagnostic StorSimple permet de détecter les problèmes liés au système, aux performances, au réseau et à l’intégrité des composants matériels sur un appareil StorSimple. L’outil de diagnostic peut être utilisé dans différents scénarios. Ces scénarios incluent la planification de la charge de travail, le déploiement d’un appareil StorSimple, l’évaluation de l’environnement réseau et la détermination des performances d’un appareil opérationnel. Cet article offre une vue d’ensemble de l’outil de diagnostic et explique comment utiliser cet outil avec un appareil StorSimple.

L’outil de diagnostic est destiné principalement aux appareils locaux de la gamme StorSimple 8000 (8100 et 8600).

## <a name="run-diagnostics-tool"></a>Exécuter l’outil de diagnostic

Cet outil peut être exécuté à partir de l’interface Windows PowerShell de votre appareil StorSimple. Pour accéder à l’interface locale de votre appareil, deux options s’offrent à vous :

* [Utiliser PuTTY pour vous connecter à la console série de l’appareil](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console).
* [Accéder à l’outil à distance via Windows PowerShell pour StorSimple](storsimple-8000-remote-connect.md).

Dans cet article, nous partons du principe que vous vous êtes connecté à la console série de l’appareil via PuTTY.

#### <a name="to-run-the-diagnostics-tool"></a>Pour exécuter l’outil de diagnostic

Une fois que vous êtes connecté à l’interface Windows PowerShell de l’appareil, procédez comme suit pour exécuter l’applet de commande.
1. Ouvrez une session sur la console série de l'appareil en suivant les étapes dans [Utilisation de PuTTY pour se connecter à la console série de l'appareil](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console).

2. Tapez la commande suivante :

    `Invoke-HcsDiagnostics`

    Si le paramètre d’étendue (Scope) n’est pas spécifié, l’applet de commande exécute tous les tests de diagnostic. Ceux-ci incluent des tests du système, de l’intégrité des composants matériels, du réseau et des performances. 
    
    Pour exécuter un test spécifique, spécifiez le paramètre Scope. Par exemple, pour exécuter uniquement le test du réseau, tapez :

    `Invoke-HcsDiagnostics -Scope Network`

3. Sélectionnez et copiez la sortie de la fenêtre PuTTY dans un fichier texte pour approfondir l’analyse.

## <a name="scenarios-to-use-the-diagnostics-tool"></a>Scénarios d’utilisation de l’outil de diagnostic

L’outil de diagnostic permet de résoudre les problèmes de réseau, de performances, de système et d’intégrité matérielle de l’appareil. Voici quelques scénarios possibles :

* **Appareil hors connexion** : votre appareil de la gamme StorSimple 8000 est hors connexion. Toutefois, dans l’interface Windows PowerShell, les deux contrôleurs semblent être opérationnels.
    * Vous pouvez utiliser cet outil pour déterminer l’état du réseau.
         
         > [!NOTE]
         > N’utilisez pas cet outil pour évaluer les performances et les paramètres réseau sur un appareil avant son inscription (ou sa configuration via l’Assistant d’installation). Une adresse IP valide est affectée à l’appareil lors de sa configuration via l’Assistant installation et de son inscription. Vous pouvez exécuter cette applet de commande sur un appareil qui n’est pas inscrit pour évaluer l’intégrité matérielle et le système. Utilisez le paramètre Scope, par exemple :
         >
         > `Invoke-HcsDiagnostics -Scope Hardware`
         >
         > `Invoke-HcsDiagnostics -Scope System`

* **Problèmes d’appareil persistants** : vous rencontrez des problèmes d’appareil qui semblent persister. Par exemple, son inscription échoue. Vous pouvez également rencontrer des problèmes alors que l’appareil a été inscrit avec succès et que vous l’utilisez depuis un certain temps.
    * Dans ce cas, utilisez cet outil pour procéder à un dépannage préliminaire avant d’enregistrer une demande de service auprès du Support Microsoft. Nous vous recommandons d’exécuter cet outil et de consigner sa sortie. Vous pouvez ensuite fournir cette sortie à la prise en charge afin d’accélérer la résolution des problèmes.
    * En cas de défaillances de composants matériels ou de clusters, vous devez enregistrer une demande de Support.

* **Faibles performances de l’appareil** : votre appareil StorSimple est lent.
    * Dans ce cas, exécutez cette applet de commande avec le paramètre Scope défini sur Performance. Analysez la sortie. Vous obtenez les latences de lecture-écriture dans le cloud. Utilisez les latences signalées en tant que cible maximale possible, prenez en compte une certaine surcharge pour le traitement interne des données, puis déployez les charges de travail sur le système. Pour obtenir des informations sur l’utilisation du test du réseau pour résoudre les problèmes de performances de l’appareil, consultez la section [Test du réseau](#network-test).


## <a name="diagnostics-test-and-sample-outputs"></a>Tests de diagnostic et exemples de sorties

### <a name="hardware-test"></a>Test du matériel

Ce test permet de déterminer l’état des composants matériels, du microprogramme USM et du microprogramme de disque de votre système.

* Les composants matériels signalés sont ceux pour lesquels le test a échoué ou qui ne sont pas présents dans le système.
* Les versions du microprogramme USM et du microprogramme de disque sont indiquées pour le contrôleur 0, le contrôleur 1 et les composants partagés de votre système. Pour obtenir une liste complète des composants matériels, consultez :

    * [Composants du boîtier principal](storsimple-8000-monitor-hardware-status.md#component-list-for-primary-enclosure-of-storsimple-device)
    * [Composants du boîtier EBOD](storsimple-8000-monitor-hardware-status.md#component-list-for-ebod-enclosure-of-storsimple-device)

> [!NOTE]
> Si le test du matériel signale des composants défectueux, [enregistrez une demande de service auprès du Support Microsoft](storsimple-8000-contact-microsoft-support.md).

#### <a name="sample-output-of-hardware-test-run-on-an-8100-device"></a>Exemple de sortie du test du matériel sur un appareil 8100

Voici un exemple de sortie pour un appareil StorSimple 8100. Le modèle 8100 ne présente pas de boîtier EBOD. Par conséquent, les composants du contrôleur du boîtier EBOD ne sont pas signalés.

```
Controller0>Invoke-HcsDiagnostics -Scope Hardware
Running hardware diagnostics ...
--------------------------------------------------
Hardware components failed or not present
----------------------

           Type           State      Controller           Index     EnclosureId
           ----           -----      ----------           -----     -----------
...rVipResource      NotPresent            None               1            None
...rVipResource      NotPresent            None               2            None
...rVipResource      NotPresent            None               3            None
...rVipResource      NotPresent            None               4            None
...rVipResource      NotPresent            None               5            None
...rVipResource      NotPresent            None               6            None
...rVipResource      NotPresent            None               7            None
...rVipResource      NotPresent            None               8            None
...rVipResource      NotPresent            None               9            None
...rVipResource      NotPresent            None              10            None
...rVipResource      NotPresent            None              11            None

Firmware information
----------------------
TalladegaController : ActiveBIOS:0.45.0010
                      BackupBIOS:0.45.0006
                      MainCPLD:17.0.000b
                      ActiveBMCRoot:2.0.001F
                      BackupBMCRoot:2.0.001F
                      BMCBoot:2.0.0002
                      LsiFirmware:20.00.04.00
                      LsiBios:07.37.00.00
                      Battery1Firmware:06.2C
                      Battery2Firmware:06.2C
                      DomFirmware:X231600
                      CanisterFirmware:3.5.0.56
                      CanisterBootloader:5.03
                      CanisterConfigCRC:0x9134777A
                      CanisterVPDStructure:0x06
                      CanisterGEMCPLD:0x19
                      CanisterVPDCRC:0x142F7DC2
                      MidplaneVPDStructure:0x0C
                      MidplaneVPDCRC:0xA6BD4F64
                      MidplaneCPLD:0x10
                      PCM1Firmware:1.00|1.05
                      PCM1VPDStructure:0x05
                      PCM1VPDCRC:0x41BEF99C
                      PCM2Firmware:1.00|1.05
                      PCM2VPDStructure:0x05
                      PCM2VPDCRC:0x41BEF99C

EbodController      :
DisksFirmware       : SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08

TalladegaController : ActiveBIOS:0.45.0010
                      BackupBIOS:0.45.0006
                      MainCPLD:17.0.000b
                      ActiveBMCRoot:2.0.001F
                      BackupBMCRoot:2.0.001F
                      BMCBoot:2.0.0002
                      LsiFirmware:20.00.04.00
                      LsiBios:07.37.00.00
                      Battery1Firmware:06.2C
                      Battery2Firmware:06.2C
                      DomFirmware:X231600
                      CanisterFirmware:3.5.0.56
                      CanisterBootloader:5.03
                      CanisterConfigCRC:0x9134777A
                      CanisterVPDStructure:0x06
                      CanisterGEMCPLD:0x19
                      CanisterVPDCRC:0x142F7DC2
                      MidplaneVPDStructure:0x0C
                      MidplaneVPDCRC:0xA6BD4F64
                      MidplaneCPLD:0x10
                      PCM1Firmware:1.00|1.05
                      PCM1VPDStructure:0x05
                      PCM1VPDCRC:0x41BEF99C
                      PCM2Firmware:1.00|1.05
                      PCM2VPDStructure:0x05
                      PCM2VPDCRC:0x41BEF99C

EbodController      :
DisksFirmware       : SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08

--------------------------------------------------
```

### <a name="system-test"></a>Test du système

Ce test fait état des informations système, des mises à jour disponibles, des informations sur les clusters et des informations sur les services pour votre appareil.

* Les informations système incluent le modèle, le numéro de série de l’appareil, le fuseau horaire, l’état du contrôleur et le détail de la version du logiciel exécuté sur le système. Pour comprendre les différents paramètres du système signalés dans la sortie, consultez [Interprétation des informations système](#appendix-interpreting-system-information).

* Les informations sur la disponibilité de mises à jour indiquent si des mises à jour sont disponibles pour les modes normal et maintenance et, le cas échéant, le nom des packages associés. Si `RegularUpdates` et `MaintenanceModeUpdates` présentent la valeur `false`, cela indique qu’aucune mise à jour n’est disponible. Votre appareil est à jour.
* Les informations sur les clusters incluent des informations sur les différents composants logiques de l’ensemble des groupes de clusters HCS et sur leur état respectif. Si un groupe de clusters hors connexion apparaît dans cette section du rapport, [contactez le Support Microsoft](storsimple-8000-contact-microsoft-support.md).
* Les informations sur les services incluent le nom et l’état de tous les services HCS et CiS exécutés sur votre appareil. Ces informations aident le Support Microsoft à résoudre les problèmes d’appareil associés.

#### <a name="sample-output-of-system-test-run-on-an-8100-device"></a>Exemple de sortie du test du système sur un appareil 8100

Voici un exemple de sortie du test du système sur un appareil 8100.

```
Controller0>Invoke-HcsDiagnostics -Scope System
Running system diagnostics ...
--------------------------------------------------

System information
----------------------
Controller0:

InstanceId              : 7382407f-a56b-4622-8f3f-756fe04cfd38
Name                    : 8100-SHX0991003G467K
Model                   : 8100
SerialNumber            : SHX0991003G467K
TimeZone                : (UTC-08:00) Pacific Time (US & Canada)
CurrentController       : Controller0
ActiveController        : Controller0
Controller0Status       : Normal
Controller1Status       : Normal
SystemMode              : Normal
FriendlySoftwareVersion : StorSimple 8000 Series Update 4.0
HcsSoftwareVersion      : 6.3.9600.17820
ApiVersion              : 9.0.0.0
VhdVersion              : 6.3.9600.17759
OSVersion               : 6.3.9600.0
CisAgentVersion         : 1.0.9441.0
MdsAgentVersion         : 35.2.2.0
Lsisas2Version          : 2.0.78.0
Capacity                : 219902325555200
RemoteManagementMode    : Disabled
FipsMode                : Enabled

Controller1:
InstanceId              : 7382407f-a56b-4622-8f3f-756fe04cfd38
Name                    : 8100-SHX0991003G467K
Model                   : 8100
SerialNumber            : SHX0991003G467K
TimeZone                :
CurrentController       : Controller0
ActiveController        : Controller0
Controller0Status       : Normal
Controller1Status       : Normal
SystemMode              : Normal
FriendlySoftwareVersion : StorSimple 8000 Series Update 4.0
HcsSoftwareVersion      : 6.3.9600.17820
ApiVersion              : 9.0.0.0
VhdVersion              : 6.3.9600.17759
OSVersion               : 6.3.9600.0
CisAgentVersion         : 1.0.9441.0
MdsAgentVersion         : 35.2.2.0
Lsisas2Version          : 2.0.78.0
Capacity                : 219902325555200
RemoteManagementMode    : HttpsAndHttpEnabled
FipsMode                : Enabled

Update availability
----------------------
RegularUpdates              : False
MaintenanceModeUpdates      : False
RegularUpdatesTitle         : {}
MaintenanceModeUpdatesTitle : {}

Cluster information
----------------------
Name                          State OwnerGroup
----                          ----- ----------
ApplicationHostRLUA           Online HCS Cluster Group
Data0v4                       Online HCS Cluster Group
HCS Vnic Resource             Online HCS Cluster Group
hcs_cloud_connectivity_...    Online HCS Cluster Group
hcs_controller_replacement    Online HCS Cluster Group
hcs_datapath_service          Online HCS Cluster Group
hcs_management_service        Online HCS Cluster Group
hcs_nvram_service             Online HCS Cluster Group
hcs_passive_datapath          Online HCS Passive Cluster Group
hcs_platform_service          Online HCS Cluster Group
hcs_saas_agent_service        Online HCS Cluster Group
HddDataClusterDisk            Online HCS Cluster Group
HddMgmtClusterDisk            Online HCS Cluster Group
HddReplClusterDisk            Online HCS Cluster Group
iSCSI Target Server           Online HCS Cluster Group
NvramClusterDisk              Online HCS Cluster Group
SSAdminRLUA                   Online HCS Cluster Group
SsdDataClusterDisk            Online HCS Cluster Group
SsdNvramClusterDisk           Online HCS Cluster Group

Service information
----------------------
Name                                          Status DisplayName
----                                          ------ -----------
CiSAgentSvc                                   Stopped CiS Service Agent
hcs_cloud_connectivity_...                    Running hcs_cloud_connectivity...
hcs_controller_replacement                    Running HCS Controller Replace...
hcs_datapath_service                          Running HCS Datapath Service
hcs_management_service                        Running HCS Management Service
hcs_minishell                                 Running hcs_minishell
HCS_NVRAM_Service                             Running HCS NVRAM Service
hcs_passive_datapath                          Stopped HCS Passive Datapath S...
hcs_platform_service                          Running HCS Platform Monitor S...
hcs_saas_agent_service                        Running hcs_saas_agent_service
hcs_startup                                   Stopped hcs_startup
--------------------------------------------------
```

### <a name="network-test"></a>Test du réseau

Ce test valide l’état des interfaces réseau, des ports, de la connectivité des serveurs DNS et NTP, du certificat TLS/SSL, des informations d’identification de compte de stockage, de la connectivité aux serveurs de mise à jour et de la connectivité du proxy web sur votre appareil StorSimple.

#### <a name="sample-output-of-network-test-when-only-data0-is-enabled"></a>Exemple de sortie du test du réseau lorsque seule l’interface DATA0 est activée

Voici un exemple de sortie pour l’appareil 8100. Vous pouvez observer dans la sortie que :
* Seules les interfaces réseau DATA0 et DATA1 sont activées et configurées.
* Les interfaces DATA2 à DATA5 ne sont pas activées dans le portail.
* La configuration du serveur DNS est valide et l’appareil peut se connecter via le serveur DNS.
* La connectivité du serveur NTP est également correcte.
* Les ports 80 et 443 sont ouverts. Toutefois, le port 9354 est bloqué. Conformément à la [configuration réseau requise pour le système](./storsimple-8000-system-requirements.md), vous devez ouvrir ce port pour la communication avec Service Bus.
* La certification TLS/SSL est valide.
* L’appareil peut se connecter au compte de stockage _myss8000storageacct_.
* La connectivité aux serveurs de mise à jour est valide.
* Le proxy web n’est pas configuré sur cet appareil.

#### <a name="sample-output-of-network-test-when-data0-and-data1-are-enabled"></a>Exemple de sortie du test du réseau lorsque les interfaces DATA0 et DATA1 sont activées

```
Controller0>Invoke-HcsDiagnostics -Scope Network
Running network diagnostics ....
--------------------------------------------------
Validating networks .....
Name                Entity              Result              Details
----                ------              ------              -------
Network interface   Data0               Valid
Network interface   Data1               Valid
Network interface   Data2               Not enabled
Network interface   Data3               Not enabled
Network interface   Data4               Not enabled
Network interface   Data5               Not enabled
DNS                 10.222.118.154      Valid
NTP                 time.windows.com    Valid
Port                80                  Open
Port                443                 Open
Port                9354                Blocked
SSL certificate     https://myss8000... Valid
Storage account ... myss8000storageacct Valid
URL                 http://download.... Valid
URL                 http://download.... Valid
Web proxy                               Not enabled         Web proxy is not...
--------------------------------------------------
```

### <a name="performance-test"></a>Test de performance

Ce test fait état des performances cloud via les latences de lecture-écriture dans le cloud pour votre appareil. Cet outil peut être utilisé afin de déterminer les performances cloud potentielles de l’appareil StorSimple. L’outil signale les performances maximales (scénario optimal pour les latences de lecture-écriture) que vous pouvez obtenir pour votre connexion.

Comme l’outil signale les performances maximales possibles, nous pouvons utiliser les latences de lecture-écriture signalées en tant que cibles lors du déploiement des charges de travail.

Le test simule les tailles d’objet blob associées aux types différents de volumes sur l’appareil. Les volumes hiérarchisés standard et les sauvegardes des volumes épinglés localement utilisent une taille d’objet blob de 64 Ko. Les volumes hiérarchisés avec option d’archivage activée utilisent une taille d’objet blob de 512 Ko. Si des volumes hiérarchisés et épinglés localement sont configurés pour votre appareil, seul le test correspondant à la taille d’objet blob de 64 Ko est exécuté.

Pour utiliser cet outil, procédez comme suit :

1.  Tout d’abord, créez un ensemble de volumes hiérarchisés et de volumes hiérarchisés avec option d’archivage activée. L’outil exécutera ainsi les tests pour les tailles d’objet blob de 64 Ko et 512 Ko.

2. Une fois que vous avez créé et configuré les volumes, exécutez l’applet de commande. Tapez :

    `Invoke-HcsDiagnostics -Scope Performance`

3. Notez les latences de lecture-écriture signalées par l’outil. L’affichage des résultats du test peut prendre plusieurs minutes.

4. Si les latences de connexion sont toutes inférieures à la plage attendue, les latences signalées par l’outil peuvent être utilisées en tant que cible maximale possible lors du déploiement des charges de travail. Prenez en compte une certaine surcharge pour le traitement interne des données.

    Si les latences de lecture-écriture signalées par l’outil de diagnostic sont élevées :

    1. Configurez Storage Analytics pour les services BLOB et analysez la sortie pour déterminer les latences du compte de stockage Azure. Pour obtenir des instructions détaillées sur l’activation et la configuration de Storage Analytics, consultez [cet article](../storage/blobs/monitor-blob-storage.md). Si ces latences sont également élevées et comparables aux valeurs fournies par l’outil de diagnostic StorSimple, vous devez enregistrer une demande de service auprès de Stockage Azure.

    2. Si les latences du compte de stockage sont faibles, contactez votre administrateur réseau pour qu’il recherche les éventuels problèmes de latence du réseau.

#### <a name="sample-output-of-performance-test-run-on-an-8100-device"></a>Exemple de sortie du test des performances sur un appareil 8100

```
Controller0>Invoke-HcsDiagnostics -Scope Performance
Running performance diagnostics...
--------------------------------------------------
Cloud performance: writing blobs
Cloud write latency: 194 ms using credential 'myss8000storageacct', blob size '64KB'
Cloud performance: reading blobs..
Cloud read latency: 544 ms using credential 'myss8000storageacct', blob size '64KB'
Cloud performance: writing blobs.
Cloud write latency: 369 ms using credential 'myss8000storageacct', blob size '512KB'
Cloud performance: reading blobs...
Cloud read latency: 4924 ms using credential 'myss8000storageacct', blob size '512KB'
--------------------------------------------------
Controller0>
```

## <a name="appendix-interpreting-system-information"></a>Annexe : Interprétation des informations système

Le tableau ci-dessous décrit les différents paramètres Windows PowerShell dans les informations système. 

| Paramètre PowerShell    | Description  |
|-------------------------|------------------|
| ID de l’instance             | Un identificateur unique ou un GUID est associé à chaque contrôleur.|
| Nom                    | Nom convivial configuré pour l’appareil via le portail Azure lors du déploiement de l’appareil. Le nom convivial par défaut est le numéro de série de l’appareil. |
| Modèle                   | Modèle de votre appareil de la gamme StorSimple 8000. Il peut s’agir du modèle 8100 ou 8600.|
| SerialNumber            | Le numéro de série de l’appareil est attribué en usine et comprend 15 caractères. Par exemple, 8600-SHX0991003G44HT indique :<br> 8600 : modèle de l’appareil.<br>SHX : site de fabrication.<br> 0991003 : produit spécifique. <br> G44HT : les 5 derniers chiffres sont incrémentés pour créer des numéros de série uniques. Il ne s’agit pas nécessairement d’une suite.|
| TimeZone                | Fuseau horaire configuré pour l’appareil dans le portail Azure lors du déploiement de l’appareil.|
| CurrentController       | Contrôleur auquel vous êtes connecté via l’interface Windows PowerShell de votre appareil StorSimple.|
| ActiveController        | Contrôleur qui est actif sur votre appareil et contrôle toutes les opérations de réseau et de disque. Il peut s’agir du contrôleur 0 ou du contrôleur 1.  |
| Controller0Status       | État du contrôleur 0 sur votre appareil. L’état du contrôleur peut être Normal, Recovery (Récupération) ou Unreachable (Inaccessible).|
| Controller1Status       | État du contrôleur 1 sur votre appareil.  L’état du contrôleur peut être Normal, Recovery (Récupération) ou Unreachable (Inaccessible).|
| SystemMode              | État général de votre appareil StorSimple. L’état de l’appareil peut être Normal, Maintenance ou Decommissioned (Retiré) (correspond à Désactivé dans le portail Azure).|
| FriendlySoftwareVersion | Chaîne conviviale qui correspond à la version du logiciel de l’appareil. Pour un système exécutant Update 4, la version conviviale du logiciel serait StorSimple 8000 Series Update 4.0.|
| HcsSoftwareVersion      | Version du logiciel HCS exécutée sur votre appareil. Par exemple, la version du logiciel HCS correspondant à StorSimple 8000 Series Update 4.0 est 6.3.9600.17820. |
| ApiVersion              | Version du logiciel de l’API Windows PowerShell de l’appareil HCS.|
| VhdVersion              | Version du logiciel de l’image d’usine fournie avec l’appareil. Si vous avez rétabli les paramètres d’usine de votre appareil, ce dernier exécute cette version du logiciel.|
| OSVersion:               | Version du logiciel du système d’exploitation Windows Server exécuté sur l’appareil. L’appareil StorSimple est basé sur Windows Server 2012 R2, qui correspond à la version 6.3.9600.|
| CisAgentVersion         | Version de l’agent Cis exécuté sur votre appareil StorSimple. Cet agent permet de communiquer avec le service StorSimple Manager exécuté dans Azure.|
| MdsAgentVersion         | Version correspondant à l’agent Mds exécuté sur votre appareil StorSimple. Cet agent transfère des données au service de surveillance et de diagnostics (Monitoring and Diagnostics Service, ou MDS).|
| Lsisas2Version          | Version correspondant aux pilotes LSI exécutés sur votre appareil StorSimple.|
| Capacité                | Capacité totale de l’appareil en octets.|
| RemoteManagementMode    | Indique si l’appareil peut être géré à distance via son interface Windows PowerShell. |
| FipsMode                | Indique si le mode FIPS (Federal Information Processing Standard) est activé sur votre appareil. La norme FIPS 140 définit les algorithmes de chiffrement qui sont approuvés pour une utilisation sur les systèmes informatiques du gouvernement fédéral américain dans le but de protéger les données sensibles. Pour les appareils exécutant Update 4 ou version ultérieure, le mode FIPS est activé par défaut. |

## <a name="next-steps"></a>Étapes suivantes

* Découvrez la [syntaxe de l’applet de commande Invoke-HcsDiagnostics](/previous-versions/windows/powershell-scripting/mt795371(v=wps.630)).

* Découvrez plus en détail comment [résoudre les problèmes de déploiement](./storsimple-8000-troubleshoot-deployment.md) sur votre appareil StorSimple.