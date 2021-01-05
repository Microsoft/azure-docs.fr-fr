---
title: Intégrer un serveur activé pour Azure Arc à Azure Sentinel
description: Découvrez comment ajouter vos serveurs activés pour Azure Arc à Azure Sentinel et comment superviser leur état de sécurité de manière proactive.
ms.date: 11/16/2020
ms.topic: conceptual
ms.openlocfilehash: 60018e710613a27bcb5c00f38ee90be6ff46c61d
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94810890"
---
# <a name="onboard-azure-arc-enabled-servers-to-azure-sentinel"></a>Intégrer des serveurs activés pour Azure Arc à Azure Sentinel

Cet article est destiné à vous aider à intégrer votre serveur activé pour Azure Arc à [Azure Sentinel](../../sentinel/overview.md) et à commencer à collecter des événements liés à la sécurité. Azure Sentinel fournit une solution unique pour la détection des alertes, la visibilité des menaces, le repérage proactif et la réponse aux menaces dans l’entreprise.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous respectez les exigences suivantes :

- Un [espace de travail Log Analytics](../../azure-monitor/platform/data-platform-logs.md). Pour plus d’informations sur les espaces de travail Log Analytics, consultez [Conception de votre déploiement de journaux Azure Monitor](../../azure-monitor/platform/design-logs-deployment.md).

- Azure Sentinel [activé dans votre abonnement](../../sentinel/quickstart-onboard.md).

- Votre machine ou votre serveur est connecté à des serveurs activés pour Azure Arc.

## <a name="onboard-azure-arc-enabled-servers-to-azure-sentinel"></a>Intégrer des serveurs activés pour Azure Arc à Azure Sentinel

Azure Sentinel est fourni avec plusieurs connecteurs pour les solutions Microsoft, prêts à l’emploi et offrant une intégration en temps réel. Pour les machines physiques et virtuelles, vous pouvez installer l’agent Log Analytics qui collecte les journaux et les transfère à Azure Sentinel. Les serveurs activés pour Arc prennent en charge le déploiement de l’agent Log Analytics en utilisant les méthodes suivantes :

- Utilisation de l’infrastructure des extensions de machine virtuelle.

    Cette fonctionnalité des serveurs activés pour Azure Arc vous permet de déployer l’extension de machine virtuelle de l’agent Log Analytics sur un serveur Windows et/ou Linux non-Azure. Les extensions de machine virtuelle peuvent être gérées à l’aide des méthodes suivantes sur vos machines ou serveurs hybrides gérés par des serveurs compatibles Arc :

    - [Portail Azure](manage-vm-extensions-portal.md)
    - [Azure CLI](manage-vm-extensions-cli.md)
    - [Azure PowerShell](manage-vm-extensions-powershell.md)
    - [Modèles Azure Resource Manager](manage-vm-extensions-template.md)

- Utilisation d’Azure Policy.

    Avec cette approche, vous utilisez la stratégie intégrée [Déployer l’agent Log Analytics sur des machines Linux ou Windows Azure Arc](../../governance/policy/samples/built-in-policies.md#monitoring) d’Azure Policy si l’agent Log Analytics est installé sur le serveur activé pour Arc. Si l’agent n’est pas installé, elle le déploie automatiquement en utilisant une tâche de remédiation. Autrement, si vous envisagez de superviser les machines avec Azure Monitor pour machines virtuelles, utilisez plutôt l’initiative [Activer Azure Monitor pour machines virtuelles](../../governance/policy/samples/built-in-initiatives.md#monitoring) pour installer et configurer l’agent Log Analytics.

Nous vous recommandons d’installer l’agent Log Analytics pour Windows ou Linux en utilisant Azure Policy.

Une fois que vos serveurs activés pour Arc sont connectés, le streaming de vos données commence dans Azure Sentinel et vous pouvez commencer à les utiliser. Vous pouvez afficher les journaux dans les [classeurs intégrés](../../sentinel/quickstart-get-visibility.md) et commencer à créer des requêtes dans Log Analytics pour [examiner les données](../../sentinel/tutorial-investigate-cases.md).

## <a name="next-steps"></a>Étapes suivantes

Prise en main de la [détection des menaces avec Azure Sentinel](../../sentinel/tutorial-detect-threats-built-in.md).