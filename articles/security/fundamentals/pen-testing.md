---
title: Test d'intrusion | Microsoft Docs
description: Cet article fournit une vue d'ensemble du processus de test d'intrusion et explique comment l'appliquer à une application exécutée dans l'infrastructure Azure.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 695d918c-a9ac-4eba-8692-af4526734ccc
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/24/2020
ms.author: terrylan
ms.openlocfilehash: af61b6ee1e69d175f47170df30f75832033d61d5
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96489715"
---
# <a name="penetration-testing"></a>Test d’intrusion

L’un des avantages liés à l’utilisation d’Azure pour tester et déployer des applications est que vous pouvez créer rapidement des environnements. Vous n’avez pas à vous soucier des aspects liés à la demande, à l’acquisition, à la mise en rack et à l’empilage de votre propre matériel en local.

Créer rapidement des environnements, c'est très bien, mais vous devez toujours effectuer les contrôles de sécurité standard. Vous souhaiterez notamment soumettre les applications que vous déployez dans Azure à un test de pénétration.

Vous savez sans doute déjà que Microsoft effectue le [test de pénétration de l’environnement Azure](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e). Ce test permet de dynamiser les améliorations d'Azure.

Nous n'effectuons pas les tests d'intrusion à votre place, mais nous savons parfaitement qu'il est important pour vous d'exécuter ces tests sur vos propres applications. Et nous ne pouvons que vous en remercier car, lorsque vous améliorez la sécurité de vos applications, vous renforcez la sécurité de l’ensemble de l’écosystème Azure.

Depuis le 15 juin 2017, une approbation préalable n’est plus exigée par Microsoft pour réaliser un test d’intrusion sur les ressources Azure. Ce processus concerne uniquement Microsoft Azure et ne s’applique à aucun autre service cloud de Microsoft.

>[!IMPORTANT]
>Même s’il n’est plus nécessaire de prévenir Microsoft d’une activité de test d’intrusion, les clients doivent toujours respecter les [Règles d’engagement pour les tests d’intrusion unifiés du cloud Microsoft](https://technet.microsoft.com/mt784683).

Vous avez la possibilité d’effectuer plusieurs tests :

* Tests sur vos points de terminaison visant à détecter les [10 principales vulnérabilités de l’OWASP (Open Web Application Security Project)](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)
* [Fuzzing](https://cloudblogs.microsoft.com/microsoftsecure/2007/09/20/fuzz-testing-at-microsoft-and-the-triage-process/) de vos points de terminaison
* [Balayage des ports](https://en.wikipedia.org/wiki/Port_scanner) de vos points de terminaison

En revanche, vous ne pouvez effectuer aucune forme d'[attaque par déni de service (DoS)](https://en.wikipedia.org/wiki/Denial-of-service_attack) en guise de test d'intrusion. Cela signifie que vous ne pouvez ni initier une attaque par déni de service proprement dite, ni effectuer les tests associés permettant de déterminer, de démontrer ou de simuler un type d'attaque par déni de service.

>[!Note]
>Microsoft travaille en collaboration avec BreakingPoint Cloud pour créer une interface à l’aide de laquelle vous pouvez générer du trafic sur les adresses IP publiques dotées de DDoS Protection à des fins de simulation. Pour en savoir plus sur la simulation BreakPoint Cloud, consultez [Valider la détection d'une attaque DDoS](../../ddos-protection/manage-ddos-protection.md#validate-and-test).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [Règles d'engagement des tests d'intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=2).