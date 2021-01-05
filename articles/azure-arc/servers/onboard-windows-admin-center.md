---
title: Connecter des machines hybrides à Azure à partir de Windows Admin Center
description: Cet article explique comment installer l’agent et connecter des machines à Azure à l’aide de serveurs Azure Arc dans Windows Admin Center.
ms.date: 10/12/2020
ms.topic: conceptual
ms.openlocfilehash: 8fa118a254689feade2897f083bb3aa2dc9db02c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183411"
---
# <a name="connect-hybrid-machines-to-azure-from-windows-admin-center"></a>Connecter des machines hybrides à Azure à partir de Windows Admin Center

Vous pouvez activer des serveurs Azure Arc pour une ou plusieurs machines Windows dans votre environnement en exécutant manuellement un ensemble d’étapes. Vous pouvez utiliser [Windows Admin Center](/windows-server/manage/windows-admin-center/understand/what-is) pour déployer l’agent Connected Machine et inscrire vos serveurs locaux sans avoir à effectuer d’étape en dehors de cet outil.

## <a name="prerequisites"></a>Prérequis

* Serveurs Azure Arc : Consultez les [prérequis](agent-overview.md#prerequisites) et vérifiez que votre abonnement, votre compte Azure et vos ressources répondent aux exigences.

* Windows Admin Center : Passez en revue les conditions requises pour [préparer votre environnement](/windows-server/manage/windows-admin-center/deploy/prepare-environment) afin de déployer et de [configurer l’intégration Azure ](/windows-server/manage/windows-admin-center/azure/azure-integration).

* Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

* Les serveurs Windows cibles que vous souhaitez gérer doivent disposer d’une connexion Internet pour accéder à Azure.

### <a name="security"></a>Sécurité

Cette méthode de déploiement nécessite que vous disposiez des droits d’administrateur sur l’ordinateur ou le serveur Windows cible pour installer et configurer l’agent. Vous devez également être membre du rôle [**Utilisateurs de passerelle**](/windows-server/manage/windows-admin-center/plan/user-access-options#gateway-access-roles).

## <a name="deploy"></a>Déployer

Procédez comme suit pour configurer le serveur Windows avec des serveurs Azure Arc.

1. Connectez-vous à Windows Admin Center.

1. Dans la liste de connexion de la page **Vue d’ensemble**, sélectionnez un serveur dans la liste des serveurs Windows connectés pour vous y connecter.

1. Dans le volet gauche, sélectionnez **Services hybrides Azure**.

1. Dans la page **Services hybrides Azure**, sélectionnez **Découvrir les services Azure**.

1. Dans la page **Découvrir les services Azure**, sous **Tirer parti des stratégies et des solutions Azure pour gérer vos serveurs avec Azure Arc**, sélectionnez **Configurer**.

1. Dans la page **Paramètres\Azure Arc pour serveurs**, si vous y êtes invité, authentifiez-vous auprès d’Azure, puis sélectionnez **Démarrage**.

1. Dans la page **Connecter le serveur à Azure**, fournissez les informations suivantes :

    1. Dans la liste déroulante **Abonnement Azure**, sélectionnez l’abonnement Azure.
    1. Pour **Groupe de ressources**, sélectionnez **Nouveau** pour créer un groupe de ressources ou, sous la liste déroulante **Groupe de ressources**, sélectionnez un groupe de ressources existant dans lequel inscrire et gérer l’ordinateur.
    1. Dans la liste déroulante **Région**, sélectionnez la région Azure dans laquelle seront stockées les métadonnées des serveurs.
    1. Si la machine ou le serveur communique via un serveur proxy pour se connecter à Internet, sélectionnez l’option **Utiliser le serveur proxy**. Spécifiez l’adresse IP ou le nom du serveur proxy, ainsi que le numéro de port, que la machine doit utiliser pour communiquer avec le serveur proxy.

1. Sélectionnez **Configurer** pour poursuivre la configuration du serveur Windows avec des serveurs Azure Arc.

Le serveur Windows se connectera à Azure, téléchargera l’agent Connected Machine, l’installera et s’inscrira auprès des serveurs Azure Arc. Pour suivre la progression, sélectionnez **Notifications** dans le menu.

Pour confirmer l’installation de l’agent Connected Machine, dans Windows Admin Center, sélectionnez [**Événements**](/windows-server/manage/windows-admin-center/use/manage-servers#events) dans le volet gauche pour passer en revue les événements *MsiInstaller* dans le Journal des événements de l’application.

## <a name="verify-the-connection-with-azure-arc"></a>Vérifier la connexion avec Azure Arc

Une fois que vous avez installé l’agent et que vous l’avez configuré pour qu’il se connecte à Azure Arc enabled servers, accédez au portail Azure pour vérifier que le serveur s’est correctement connecté. Affichez vos machines dans le [portail Azure](https://portal.azure.com).

:::image type="content" source="./learn/media/quick-enable-hybrid-vm/enabled-machine.png" alt-text="Une connexion de machine réussie" border="false":::

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la résolution des problèmes, consultez le [guide Résoudre les problèmes de l’agent Connected Machine](troubleshoot-agent-onboard.md).

* Apprenez à gérer votre machine à l’aide d’[Azure Policy](../../governance/policy/overview.md), par exemple pour la [configuration invité](../../governance/policy/concepts/guest-configuration.md) des machines virtuelles, pour vérifier que l’ordinateur crée des rapports sur l’espace de travail Log Analytics prévu, pour activer l’analyse d’[Azure Monitor sur des machines virtuelles](../../azure-monitor/insights/vminsights-enable-policy.md) et bien plus encore.

* En savoir plus sur [l’agent Log Analytics](../../azure-monitor/platform/log-analytics-agent.md). L’agent Log Analytics pour Windows et Linux est nécessaire quand vous souhaitez collecter des données de supervision du système d’exploitation et de la charge de travail, gérer les ressources à l’aide de runbooks Automation ou de fonctionnalités comme Update Management, ou utiliser d’autres services Azure comme [Azure Security Center](../../security-center/security-center-introduction.md).