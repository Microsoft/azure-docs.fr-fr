---
title: Configurer votre solution de sécurité pour connecter des données CEF à Azure Sentinel en préversion | Microsoft Docs
description: Découvrez comment configurer votre solution de sécurité pour connecter des données CEF à Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: fec3f25c4b401ff7c3bc73d249b716b9c12e6529
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96548543"
---
# <a name="step-2-configure-your-security-solution-to-send-cef-messages"></a>ÉTAPE 2 : Configurer votre solution de sécurité pour envoyer des messages CEF

Au cours de cette étape, vous allez effectuer les modifications de configuration nécessaires sur votre solution de sécurité pour envoyer des journaux à l’agent CEF.

## <a name="configure-a-solution-with-a-connector"></a>Configurer une solution avec un connecteur

Si votre solution de sécurité possède déjà un connecteur existant, utilisez les instructions spécifiques au connecteur comme suit :

- [AI Vectra Detect](connect-ai-vectra-detect.md)
- [Check Point](connect-checkpoint.md)
- [Cisco](connect-cisco.md)
- [ExtraHop Reveal(x)](connect-extrahop.md)
- [F5 ASM](connect-f5.md)  
- [Produits Forcepoint](connect-forcepoint-casb-ngfw.md)
- [Fortinet](connect-fortinet.md)
- [Illusive Networks AMS](connect-illusive-attack-management-system.md)
- [One Identity Safeguard](connect-one-identity.md)
- [Palo Alto Networks](connect-paloalto.md)
- [Trend Micro Deep Security](connect-trend-micro.md)
- [Zscaler](connect-zscaler.md)
## <a name="configure-any-other-solution"></a>Configurer une autre solution

Si aucun connecteur n’existe pour votre solution de sécurité spécifique, utilisez les instructions génériques suivantes pour transférer les journaux à l’agent CEF.

1. Accédez à l’article de configuration spécifique pour connaître la procédure de configuration de votre solution pour envoyer des messages CEF. Si votre solution n’est pas répertoriée sur l’appliance, vous devez définir ces valeurs pour que l’appliance envoie les journaux nécessaires dans le format approprié à l’agent Azure Sentinel Syslog, en fonction de l’agent Log Analytics. Vous pouvez modifier ces paramètres dans votre appliance, à condition de les modifier également dans le démon Syslog sur l'agent Azure Sentinel.
    - Protocole = TCP
    - Port = 514
    - Format = CEF
    - Adresse IP : assurez-vous d’envoyer les messages CEF à l’adresse IP de la machine virtuelle dédiée à cet effet.

   Cette solution prend en charge Syslog RFC 3164 ou RFC 5424.

1. Pour rechercher des événements CEF dans Log Analytics, entrez `CommonSecurityLog` dans la fenêtre Requête.

1. Continuez jusqu’à l’ÉTAPE 3 : [valider la connectivité](connect-cef-verify.md).

> [!NOTE]
> **Modification de la source du champ TimeGenerated**
>
> - Par défaut, l’agent Log Analytics remplit le champ *TimeGenerated* du schéma en indiquant l’heure à laquelle il a reçu l’événement du démon Syslog. Par conséquent, l’heure à laquelle l’événement a été généré sur le système source n’est pas enregistrée dans Azure Sentinel.
>
> - Vous pouvez toutefois exécuter la commande suivante, qui permet de télécharger et d’exécuter le script `TimeGenerated.py`. Ce script configure l’agent Log Analytics de sorte qu’il remplisse le champ *TimeGenerated* en indiquant l’heure d’origine de l’événement sur son système source, et non l’heure à laquelle l’agent l’a reçu.
>
>    ```bash
>    wget -O TimeGenerated.py https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/CEF/TimeGenerated.py && python TimeGenerated.py {ws_id}
>    ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter les appliances CEF à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](./tutorial-detect-threats-built-in.md).