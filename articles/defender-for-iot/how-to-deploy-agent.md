---
title: Sélectionner et déployer des agents de sécurité
description: Découvrez comment sélectionner et déployer des agents de sécurité Defender pour IoT sur des appareils IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/23/2019
ms.author: mlottner
ms.openlocfilehash: 2634ed819b8818632c58b9e471b0f26190e5f16b
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96339963"
---
# <a name="select-and-deploy-a-security-agent-on-your-iot-device"></a>Sélectionner et de déployer un agent de sécurité sur votre appareil IoT

Defender pour IoT fournit des architectures de référence pour les agents de sécurité qui surveillent et collectent des données à partir d’appareils IoT.
Consultez [l’Architecture de référence d’agent de sécurité](security-agent-architecture.md) pour en savoir plus.

Les agents sont développés en tant que projets open source et sont disponibles en deux versions : <br> [C](https://aka.ms/iot-security-github-c) et [C#](https://aka.ms/iot-security-github-cs).

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Comparer les versions d’agent de sécurité
> * Découvrir les plateformes d’agent prises en charge
> * Choisir la bonne version de l’agent pour votre solution

## <a name="understand-security-agent-options"></a>Comprendre les options d’agent de sécurité

Chaque version de l’agent de sécurité Defender pour IoT offre le même ensemble de fonctionnalités et prend en charge des options de configuration similaires.

L’agent de sécurité basé sur C représente un encombrement mémoire moindre et est le choix idéal pour les appareils avec moins de ressources disponibles.

|     | Agent de sécurité C | Agent de sécurité C# |
| --- | ----------- | --------- |
| **Open source** | Disponible sous [licence MIT](https://en.wikipedia.org/wiki/MIT_License) dans [GitHub](https://aka.ms/iot-security-github-c) | Disponible sous [licence MIT](https://en.wikipedia.org/wiki/MIT_License) dans [GitHub](https://aka.ms/iot-security-github-cs) |
| **Langage de développement**    | C | C# |
| **Plateformes Windows prises en charge ?** | Non | Oui |
| **Configuration requise pour Windows** | --- | [WMI](/windows/desktop/wmisdk/) |
| **Plateformes Linux prises en charge ?** | Oui, x64 et x86 | Oui, x64 uniquement |
| **Configuration requise pour Linux** | libunwind8, libcurl3, uuid-runtime, auditd, audispd-plugins | libunwind8, libcurl3, uuid-runtime, auditd, audispd-plugins, sudo, netstat, iptables |
| **Empreinte mémoire du disque** | 10,5 Mo | 90 Mo |
| **Empreinte mémoire (en moyenne)** | 5,5 Mo | 33 Mo |
| **[Authentification](concept-security-agent-authentication-methods.md) auprès d’IoT Hub** | Oui | Oui |
| **[Collection](how-to-agent-configuration.md#supported-security-events) de données de sécurité** | Oui | Oui |
| **Agrégation d’événements** | Oui | Oui |
| **Configuration à distance via [jumeau de module de sécurité](concept-security-module.md)** | Oui | Oui |

## <a name="security-agent-installation-guidelines"></a>Instructions relatives à l’installation de l’agent de sécurité

Pour **Windows** : Le script Install SecurityAgent.ps1 doit être exécuté à partir d’une fenêtre PowerShell d’administrateur.

Pour **Linux** : Le InstallSecurityAgent.sh doit être exécuté en tant que super-utilisateur. Nous vous recommandons de préfixer la commande d’installation avec « sudo ».

## <a name="choose-an-agent-flavor"></a>Choisir une version de l’agent

Répondez aux questions suivantes concernant vos appareils IoT pour sélectionner l’agent approprié :

- Utilisez-vous _Windows Server_ ou _Windows IoT Core_ ?

    [Déployez un agent de sécurité basé sur C# pour Windows](how-to-deploy-windows-cs.md).

- Utilisez-vous une distribution Linux avec architecture x86 ?

    [Déployez un agent de sécurité basé sur C pour Linux](how-to-deploy-linux-c.md).

- Utilisez-vous une distribution Linux avec architecture x64 ?

    Les deux versions de l’agent peuvent être utilisées. <br>
    [Déployer un agent de sécurité basé sur C pour Linux](how-to-deploy-linux-c.md) et/ou [Déployer un agent de sécurité basé sur C# pour Linux](how-to-deploy-linux-cs.md).

Les deux versions de l’agent offrent le même ensemble de fonctionnalités et prennent en charge des options de configuration similaires.
Consultez [Comparaison des agents de sécurité](how-to-deploy-agent.md#understand-security-agent-options) pour en savoir plus.

## <a name="supported-platforms"></a>Plateformes prises en charge

La liste suivante inclut toutes les plateformes prises en charge actuellement.

|Agent Defender pour IoT |Système d’exploitation |Architecture |
|--------------|------------|--------------|
|C|Ubuntu 16.04 |    x64|
|C|Ubuntu 18.04 |    x64, ARMv7|
|C|Debian 9 |    x64, x86|
|C#|Ubuntu 16.04     |x64|
|C#|Ubuntu 18.04    |x64, ARMv7|
|C#|Debian 9    |x64|
|C#|Windows Server 2016|    X64|
|C#|Windows 10 IoT Core, build 17763    |x64|
|

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les options de configuration, allez au guide pratique de configuration de l’agent.
> [!div class="nextstepaction"]
> [Guide pratique de configuration de l’agent](./how-to-agent-configuration.md)