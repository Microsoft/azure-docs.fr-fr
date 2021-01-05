---
title: Fonctionnalités prises en charge disponibles dans Azure Security Center | Microsoft Docs
description: Ce document fournit la liste des services pris en charge par Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 870ebc8d-1fad-435b-9bf9-c477f472ab17
ms.service: security-center
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/12/2020
ms.author: memildin
ms.openlocfilehash: 40d70c7bd740d8c72ddb72047a3bae3586e1aa1f
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94445249"
---
# <a name="feature-coverage-for-machines"></a>Couverture des fonctionnalités pour les machines

Les deux tableaux ci-dessous montrent les fonctionnalités d’Azure Security Center disponibles pour machines virtuelles et les serveurs Windows et Linux.

## <a name="supported-features-for-virtual-machines-and-servers"></a>Fonctionnalités prises en charge pour les machines virtuelles et les serveurs <a name="vm-server-features"></a>

### <a name="windows-machines"></a>[**Machines Windows**](#tab/features-windows)

|**Fonctionnalité**|**Machines virtuelles Azure**|**Groupes de machines virtuelles identiques Azure**|**Machines compatibles avec Azure Arc**|**Azure Defender requis**
|----|:----:|:----:|:----:|:----:|
|[Intégration de Microsoft Defender for Endpoint](security-center-wdatp.md)|✔</br>(sur les versions prises en charge)|✔</br>(sur les versions prises en charge)|✔|Oui|
|[Analytique comportementale des machines virtuelles (et alertes de sécurité)](alerts-reference.md)|✔|✔|✔|Oui|
|[Alertes de sécurité sans fichier](alerts-reference.md#alerts-windows)|✔|✔|✔|Oui|
|[Alertes de sécurité réseau](other-threat-protections.md#network-layer)|✔|✔|-|Oui|
|[Accès juste-à-temps aux machines virtuelles](security-center-just-in-time.md)|✔|-|-|Oui|
|[Évaluation native des vulnérabilités](deploy-vulnerability-assessment-vm.md#overview-of-the-integrated-vulnerability-scanner)|✔|-|✔|Oui|
|[Supervision de l’intégrité des fichiers](security-center-file-integrity-monitoring.md)|✔|✔|✔|Oui|
|[Contrôles d’application adaptative](security-center-adaptive-application.md)|✔|-|✔|Oui|
|[Mappage réseau](security-center-network-recommendations.md#network-map)|✔|✔|-|Oui|
|[Durcissement réseau adaptatif](security-center-adaptive-network-hardening.md)|✔|-|-|Oui|
|[Tableau de bord et rapports de conformité réglementaire](security-center-compliance-dashboard.md)|✔|✔|✔|Oui|
|Recommandations et protection contre les menaces sur les conteneurs IaaS hébergés dans Docker|-|-|-|Oui|
|Évaluation des correctifs de système d’exploitation manquants|✔|✔|✔|Azure : Non<br><br>Compatible avec Arc : Oui|
|Évaluation des erreurs de configuration de la sécurité|✔|✔|✔|Azure : Non<br><br>Compatible avec Arc : Oui|
|[Évaluation de la protection des points de terminaison](security-center-services.md#supported-endpoint-protection-solutions-)|✔|✔|✔|Azure : Non<br><br>Compatible avec Arc : Oui|
|Évaluation du chiffrement des disques|✔</br>(pour les [scénarios pris en charge](../virtual-machines/windows/disk-encryption-windows.md#unsupported-scenarios))|✔|-|Non|
|Évaluation des vulnérabilités tierces|✔|-|✔|Non|
|[Évaluation de la sécurité réseau](security-center-network-recommendations.md)|✔|✔|-|Non|


### <a name="linux-machines"></a>[**Machines Linux**](#tab/features-linux)

|**Fonctionnalité**|**Machines virtuelles Azure**|**Groupes de machines virtuelles identiques Azure**|**Machines compatibles avec Azure Arc**|**Azure Defender requis**
|----|:----:|:----:|:----:|:----:|
|[Intégration de Microsoft Defender for Endpoint](security-center-wdatp.md)|-|-|-|Oui|
|[Analytique comportementale des machines virtuelles (et alertes de sécurité)](./azure-defender.md)|✔</br>(sur les versions prises en charge)|✔</br>(sur les versions prises en charge)|✔|Oui|
|[Alertes de sécurité sans fichier](alerts-reference.md#alerts-windows)|-|-|-|Oui|
|[Alertes de sécurité réseau](other-threat-protections.md#network-layer)|✔|✔|-|Oui|
|[Accès juste-à-temps aux machines virtuelles](security-center-just-in-time.md)|✔|-|-|Oui|
|[Évaluation native des vulnérabilités](deploy-vulnerability-assessment-vm.md#overview-of-the-integrated-vulnerability-scanner)|✔|-|✔|Oui|
|[Supervision de l’intégrité des fichiers](security-center-file-integrity-monitoring.md)|✔|✔|✔|Oui|
|[Contrôles d’application adaptative](security-center-adaptive-application.md)|✔|-|✔|Oui|
|[Mappage réseau](security-center-network-recommendations.md#network-map)|✔|✔|-|Oui|
|[Durcissement réseau adaptatif](security-center-adaptive-network-hardening.md)|✔|-|-|Oui|
|[Tableau de bord et rapports de conformité réglementaire](security-center-compliance-dashboard.md)|✔|✔|✔|Oui|
|Recommandations et protection contre les menaces sur les conteneurs IaaS hébergés dans Docker|✔|✔|✔|Oui|
|Évaluation des correctifs de système d’exploitation manquants|✔|✔|✔|Azure : Non<br><br>Compatible avec Arc : Oui|
|Évaluation des erreurs de configuration de la sécurité|✔|✔|✔|Azure : Non<br><br>Compatible avec Arc : Oui|
|[Évaluation de la protection des points de terminaison](security-center-services.md#supported-endpoint-protection-solutions-)|-|-|-|Non|
|Évaluation du chiffrement des disques|✔</br>(pour les [scénarios pris en charge](../virtual-machines/windows/disk-encryption-windows.md#unsupported-scenarios))|✔|-|Non|
|Évaluation des vulnérabilités tierces|✔|-|✔|Non|
|[Évaluation de la sécurité réseau](security-center-network-recommendations.md)|✔|✔|-|Non|

--- 


> [!TIP]
>Pour expérimenter les fonctionnalités disponibles uniquement avec Azure Defender, vous pouvez vous inscrire à une version d’évaluation de 30 jours. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/security-center/).


## <a name="supported-endpoint-protection-solutions"></a>Solutions de protection du point de terminaison prises en charge <a name="endpoint-supported"></a>

Le tableau suivant fournit une matrice de ce qui suit :

 - Si vous pouvez utiliser Azure Security Center pour installer chaque solution.
 - Les solutions de protection du point de terminaison que Security Center peut détecter. Si l’une des solutions de protection du point de terminaison de cette liste est détectée, Security Center vous déconseillera d'en installer une autre.

Pour plus d’informations sur le moment où les recommandations sont générées pour chacune de ces protections, consultez [Évaluation de la protection de point de terminaison et recommandations](security-center-endpoint-protection.md).

| Protection du point de terminaison| Plateformes | Installation du centre de sécurité | Détection du centre de sécurité |
|------|------|-----|-----|
| Antivirus Microsoft Defender| Windows Server 2016 ou version ultérieure| Non, intégré au système d’exploitation| Oui |
| System Center Endpoint Protection (logiciel anti-programme malveillant de Microsoft) | Windows Server 2012 R2, 2012, 2008 R2 (voir la remarque ci-dessous) | Via l’extension | Oui |
| Trend Micro - Deep Security | Gamme Windows Server  | Non | Oui |
| Symantec v12.1.1100+| Gamme Windows Server  | Non | Oui |
| McAfee v10+ | Gamme Windows Server  | Non | Oui |
| McAfee v10+ | Famille de serveurs Linux  | Non | Oui * *\** _ |
| Sophos V9+| Famille de serveurs Linux  | Non | Oui _*\**_  |

 _ *\** * L’état de couverture et les données de prise en charge sont actuellement disponibles uniquement dans l’espace de travail Log Analytics associé à vos abonnements protégés. Il n’est pas reflété dans le portail Azure Security Center.

> [!NOTE]
> La détection de System Center Endpoint Protection (SCEP) sur une machine virtuelle Windows Server 2008 R2 nécessite l’installation de SCEP après celle de PowerShell (v3.0 ou ultérieure).


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [Security Center collecte des données avec l’agent Log Analytics](security-center-enable-data-collection.md).
- Découvrez comment [Security Center gère et protège les données](security-center-data-security.md).
- Passez en revue les [plateformes qui prennent en charge Security Center](security-center-os-coverage.md).
