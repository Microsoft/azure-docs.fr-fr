---
title: Suggestions de protection de point de terminaison dans Azure Security Centers
description: Découvrez comment les solutions de protection de point de terminaison sont détectées et identifiées comme saines.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 2730a2f5-20bc-4027-a1c2-db9ed0539532
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/29/2019
ms.author: memildin
ms.openlocfilehash: 778feda8d72101d4ae2a0f0c27549701dfb53340
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92341411"
---
# <a name="endpoint-protection-assessment-and-recommendations-in-azure-security-center"></a>Évaluation de la protection de point de terminaison et recommandations dans Azure Security Center

Azure Security Center fournit des évaluations d’intégrité des versions [prises en charge](security-center-services.md#endpoint-supported) des solutions de protection de point de terminaison. Cet article explique les scénarios qui conduisent Security Center à générer les deux recommandations suivantes :

* **Installer les solutions de protection de point de terminaison sur votre machine virtuelle**
* **Résoudre les problèmes d'intégrité de la protection du point de terminaison sur vos machines**

## <a name="windows-defender"></a>Windows Defender

* Security Center vous recommande d’ **« Installer des solutions de protection de point de terminaison sur une machine virtuelle »** lorsque la commande [Get-MpComputerStatus](/powershell/module/defender/get-mpcomputerstatus?preserve-view=true&view=win10-ps) s’exécute et renvoie **AMServiceEnabled : False**

* Security Center vous recommande de **« Résoudre les problèmes d’intégrité de la protection de point de terminaison sur vos machines »** lorsque commande [Get-MpComputerStatus](/powershell/module/defender/get-mpcomputerstatus?preserve-view=true&view=win10-ps) s’exécute et qu’une des situations suivantes se présente :

  * L’une des propriétés suivantes est false :

    - **AMServiceEnabled**
    - **AntispywareEnabled**
    - **RealTimeProtectionEnabled**
    - **BehaviorMonitorEnabled**
    - **IoavProtectionEnabled**
    - **OnAccessProtectionEnabled**

  * Si au moins une des deux propriétés suivantes a une valeur égale ou supérieure à 7 :

    - **AntispywareSignatureAge**
    - **AntivirusSignatureAge**

## <a name="microsoft-system-center-endpoint-protection"></a>Protection du point de terminaison Microsoft System Center

* Security Center vous recommande d’ **« Installer des solutions de protection de point de terminaison sur la machine virtuelle »** lors de l’importation de **SCEPMpModule ("$env:ProgramFiles\Microsoft Security Client\MpProvider\MpProvider.psd1")** quand l’exécution de la commande **Get-MProtComputerStatus** retourne **AMServiceEnabled = false**.

* Security Center vous recommande de **« Résoudre les problèmes d’intégrité de la protection de point de terminaison sur vos machines »** lorsque commande **Get-MprotComputerStatus** s’exécute et qu’une des situations suivantes se présente :

  * Au moins une des propriétés suivantes est False :

    - **AMServiceEnabled**
    - **AntispywareEnabled**
    - **RealTimeProtectionEnabled**
    - **BehaviorMonitorEnabled**
    - **IoavProtectionEnabled**
    - **OnAccessProtectionEnabled**

  * Si une voire les deux mises à jour de signature sont supérieures ou égales à 7 :

    * **AntispywareSignatureAge**
    * **AntivirusSignatureAge**

## <a name="trend-micro"></a>Trend Micro

* Security Center vous recommande d’ **« Installer les solutions de protection de point de terminaison sur la machine virtuelle »** si une des vérifications suivantes n’est pas satisfaite :
    - **HKLM:\SOFTWARE\TrendMicro\Deep Security Agent** existe
    - **HKLM:\SOFTWARE\TrendMicro\Deep Security Agent\InstallationFolder** existe
    - Le fichier **dsa_query.cmd** se trouve dans le dossier d’installation
    - L'exécution de **dsa_query.cmd** indique **Component.AM.mode : on - Trend Micro Deep Security Agent détecté**

## <a name="symantec-endpoint-protection"></a>Protection de point de terminaison Symantec
Security Center vous recommande d’ **« Installer les solutions de protection de point de terminaison sur la machine virtuelle »** si une des vérifications suivantes n’est pas satisfaite :

- **HKLM:\Software\Symantec\Symantec Endpoint Protection\CurrentVersion\PRODUCTNAME = "Symantec Endpoint Protection"**
- **HKLM:\Software\Symantec\Symantec Endpoint Protection\CurrentVersion\public-opstate\ASRunningStatus = 1**

ou

- **HKLM:\Software\Wow6432Node\Symantec\Symantec Endpoint Protection\CurrentVersion\PRODUCTNAME = "Symantec Endpoint Protection"**
- **HKLM:\Software\Wow6432Node\Symantec\Symantec Endpoint Protection\CurrentVersion\public-opstate\ASRunningStatus = 1**

Security Center vous recommande de **« Résoudre les problèmes d’intégrité de la protection de point de terminaison sur vos machines »** si une des vérifications suivantes n’est pas satisfaite :

- Vérifier la version de Symantec > = 12 : Emplacement du registre : **HKLM:\Software\Symantec\Symantec Endpoint Protection\CurrentVersion" -Value "PRODUCTVERSION"**
- Vérifier l'état de la protection en temps réel : **HKLM:\Software\Wow6432Node\Symantec\Symantec Endpoint Protection\AV\Storages\Filesystem\RealTimeScan\OnOff == 1**
- Vérifier l’état de la mise à jour de la signature : **HKLM\Software\Symantec\Symantec Endpoint Protection\CurrentVersion\public-opstate\LatestVirusDefsDate <= 7 days**
- Vérifier l’état de l’analyse complète : **HKLM:\Software\Symantec\Symantec Endpoint Protection\CurrentVersion\public-opstate\LastSuccessfulScanDateTime <= 7 days**
- Rechercher le numéro de version de signature pour Symantec 12 : **Registry Paths+ "CurrentVersion\SharedDefs" -Value "SRTSP"** 
- Chemin d’accès à la version de signature pour Symantec 14 : **Registry Paths+ "CurrentVersion\SharedDefs\SDSDefs" -Value "SRTSP"**

Chemins d’accès au Registre :
- **"HKLM:\Software\Symantec\Symantec Endpoint Protection" + $Path;**
- **"HKLM:\Software\Wow6432Node\Symantec\Symantec Endpoint Protection" + $Path**

## <a name="mcafee-endpoint-protection-for-windows"></a>Protection de point de terminaison McAfee pour Windows

Security Center vous recommande d’ **« Installer les solutions de protection de point de terminaison sur la machine virtuelle »** si une des vérifications suivantes n’est pas satisfaite :

- **HKLM:\SOFTWARE\McAfee\Endpoint\AV\ProductVersion** exists
- **HKLM:\SOFTWARE\McAfee\AVSolution\MCSHIELDGLOBAL\GLOBAL\enableoas = 1**

Security Center vous recommande de **« Résoudre les problèmes d’intégrité de la protection de point de terminaison sur vos machines »** si une des vérifications suivantes n’est pas satisfaite :

- Version McAfee : **HKLM:\SOFTWARE\McAfee\Endpoint\AV\ProductVersion >= 10**
- Rechercher la version de signature : **HKLM:\Software\McAfee\AVSolution\DS\DS -Value "dwContentMajorVersion"**
- Rechercher la date signature : **HKLM:\Software\McAfee\AVSolution\DS\DS -Value "szContentCreationDate" >= 7 days**
- Rechercher la date d'analyser : **HKLM:\Software\McAfee\Endpoint\AV\ODS -Value "LastFullScanOdsRunTime" >= 7 days**

## <a name="mcafee-endpoint-security-for-linux-threat-prevention"></a>McAfee Endpoint Security for Linux Threat Prevention 

Security Center vous recommande d’ **« Installer les solutions de protection de point de terminaison sur la machine virtuelle »** si une des vérifications suivantes n’est pas satisfaite :

- Le fichier **/opt/isec/ens/threatprevention/bin/isecav** existe
- La sortie **"/opt/isec/ens/threatprevention/bin/isecav --version"** est : **McAfee name = McAfee Endpoint Security for Linux Threat Prevention and McAfee version >= 10**

Security Center vous recommande de **« Résoudre les problèmes d’intégrité de la protection de point de terminaison sur vos machines »** si une des vérifications suivantes n’est pas satisfaite :

- **"/opt/isec/ens/threatprevention/bin/isecav --listtask"** retourne **Quick scan, Full scan** et les deux analyses <= 7 jours
- **"/opt/isec/ens/threatprevention/bin/isecav --listtask"** retourne **DAT and engine Update time** et les deux <= 7 jours
- **"/opt/isec/ens/threatprevention/bin/isecav --getoasconfig --summary"** retourne l’état **On Access Scan**

## <a name="sophos-antivirus-for-linux"></a>Sophos Anti-Virus pour Linux 

Security Center vous recommande d’ **« Installer les solutions de protection de point de terminaison sur la machine virtuelle »** si une des vérifications suivantes n’est pas satisfaite :
- Le fichier **/opt/sophos-av/bin/savdstatus** se ferme ou recherche l'emplacement personnalisé **"readlink $(which savscan)"**
- **"/opt/sophos-av/bin/savdstatus --version"** retourne **Sophos name = Sophos Anti-Virus and Sophos version >= 9**

Security Center vous recommande de **« Résoudre les problèmes d’intégrité de la protection de point de terminaison sur vos machines »** si une des vérifications suivantes n’est pas satisfaite :
- **"/opt/sophos-av/bin/savlog --maxage=7 | grep -i "Scheduled scan .\* completed" | tail -1"** , retourne une valeur
- **"/opt/sophos-av/bin/savlog --maxage=7 | grep "scan finished"** | tail -1", retourne une valeur
- **"/opt/sophos-av/bin/savdstatus --lastupdate"** retourne lastUpdate qui doit être <= 7 jours 
- **"/opt/sophos-av/bin/savdstatus -v"** est égal à **"On-access scanning is running"** 
- **"/opt/sophos-av/bin/savconfig get LiveProtection"** retourne activé

## <a name="troubleshoot-and-support"></a>Dépannage et support technique

### <a name="troubleshoot"></a>Dépanner

Les journaux d’activité de l’extension Microsoft Antimalware sont disponibles à l’emplacement suivant : **%Systemdrive%\WindowsAzure\Logs\Plugins\Microsoft.Azure.Security.IaaSAntimalware(Ou PaaSAntimalware)\1.5.5.x(version#)\CommandExecution.log**

### <a name="support"></a>Support

Pour obtenir une aide supplémentaire, contactez les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur Obtenir un support. Pour plus d’informations sur l’utilisation du support Azure, lisez le [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).