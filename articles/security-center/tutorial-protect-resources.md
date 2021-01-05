---
title: Didacticiel sur les contrôles d'accès et d'applications - Azure Security Center
description: Ce tutoriel montre comment configurer une stratégie d’accès juste-à-temps aux machines virtuelles et une stratégie de contrôle d’applications.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security-center
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/03/2018
ms.author: memildin
ms.openlocfilehash: 89d04588d07407be5dbd7ddbafd6543ed461462e
ms.sourcegitcommit: ba7fafe5b3f84b053ecbeeddfb0d3ff07e509e40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91945717"
---
# <a name="tutorial-protect-your-resources-with-azure-security-center"></a>Tutoriel : Protéger vos ressources avec Azure Security Center
Security Center limite votre exposition aux menaces en utilisant des contrôles d’accès et d’applications pour bloquer les activités malveillantes. L’accès juste-à-temps (JIT) aux machines virtuelles réduit votre exposition aux attaques en vous permettant de refuser l’accès persistant aux machines virtuelles. À la place, vous fournissez un accès contrôlé et audité aux machines virtuelles uniquement en cas de besoin. Les contrôles d’applications adaptatifs permettent de renforcer la protection contre les logiciels malveillants en contrôlant les applications qui peuvent s’exécuter sur les machines virtuelles. Security Center utilise le machine learning pour analyser les processus en cours d’exécution sur la machine virtuelle et exploite ces informations pour vous aider à appliquer les règles de mise en liste d’autorisation.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Configurer une stratégie d’accès juste-à-temps aux machines virtuelles
> * Configurer une stratégie de contrôle d’applications

## <a name="prerequisites"></a>Prérequis
Pour parcourir les fonctionnalités couvertes dans ce tutoriel, vous devez avoir activé Azure Defender. Vous pouvez essayer gratuitement Azure Defender. Pour plus d’informations, consultez [Essayer Azure Defender](security-center-pricing.md).

## <a name="manage-vm-access"></a>Gérer l’accès aux machines virtuelles
L’accès JIT aux machines virtuelles peut être utilisé pour verrouiller le trafic entrant vers vos machines virtuelles Azure, réduire l’exposition aux attaques et faciliter la connexion aux machines virtuelles si nécessaire.

Les ports de gestion n’ont pas besoin d’être toujours ouverts. Ils doivent uniquement être ouverts lorsque vous êtes connecté à la machine virtuelle, par exemple pour effectuer des tâches de maintenance ou de gestion. Quand la fonctionnalité juste-à-temps est activée, Security Center utilise des règles de groupe de sécurité réseau (NSG) qui limitent l’accès aux ports de gestion pour qu’ils ne soient pas la cible d’attaquants.

Suivez les instructions fournies dans [Sécuriser vos ports de gestion avec un accès juste-à-temps](security-center-just-in-time.md).

## <a name="harden-vms-against-malware"></a>Renforcer la protection des machines virtuelles contre les logiciels malveillants
Les contrôles d’applications adaptatifs vous aident à définir un ensemble d’applications autorisées à s’exécuter sur des groupes de ressources configurés, ce qui permet, entre autres, de renforcer la protection de vos machines virtuelles contre les logiciels malveillants. Security Center utilise le machine learning pour analyser les processus en cours d’exécution sur la machine virtuelle et exploite ces informations pour vous aider à appliquer les règles de mise en liste d’autorisation.

Suivez les instructions fournies dans [Utiliser des contrôles d’application adaptatifs pour réduire les surfaces d’attaque de vos machines](security-center-adaptive-application.md).

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à limiter votre exposition aux menaces en effectuant les tâches suivantes :

> [!div class="checklist"]
> * Configuration d’une stratégie d’accès juste-à-temps aux machines virtuelles pour fournir un accès contrôlé et audité aux machines virtuelles uniquement en cas de besoin
> * Configuration d’une stratégie de contrôles d’applications adaptatifs pour contrôler les applications qui peuvent s’exécuter sur vos machines virtuelles

Passez au didacticiel suivant pour en savoir plus sur la façon de répondre aux incidents de sécurité.

> [!div class="nextstepaction"]
> [Tutoriel : Répondre à des incidents de sécurité](tutorial-security-incident.md)

<!--Image references-->
[1]: ./media/tutorial-protect-resources/just-in-time-vm-access.png
[2]: ./media/tutorial-protect-resources/add-port.png
[3]: ./media/tutorial-protect-resources/adaptive-application-control-options.png
[4]: ./media/tutorial-protect-resources/recommended-resource-groups.png
