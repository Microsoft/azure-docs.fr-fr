---
title: Configurer le serveur de configuration pour la récupération d’urgence de serveurs physiques sur Azure avec Azure Site Recovery | Microsoft Docs
description: Cet article décrit la configuration du serveur de configuration local pour la récupération d’urgence de serveurs physiques locaux sur Azure.
services: site-recovery
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 07/03/2019
ms.author: ramamill
ms.openlocfilehash: 327e995a8fe2f66903548fba054804768d2538ab
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001549"
---
# <a name="set-up-the-configuration-server-for-disaster-recovery-of-physical-servers-to-azure"></a>Configurer le serveur de configuration pour la récupération d’urgence de serveurs physiques sur Azure

Cet article décrit comment configurer votre environnement local pour lancer la réplication des serveurs physiques exécutant Windows ou Linux dans Azure.

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous disposez déjà des éléments suivants :
- Un coffre Recovery Services dans le [portail Azure](https://portal.azure.com "Portail Azure").
- Un ordinateur physique sur lequel installer le serveur de configuration.
- Si vous avez désactivé TLS 1.0 sur l’ordinateur sur lequel vous installez le serveur de configuration, vérifiez que TLS 1.2 est activé et que .NET Framework version 4.6 ou ultérieure est installé sur l’ordinateur (et que le chiffrement fort est activé). [Plus d’informations](https://support.microsoft.com/help/4033999/how-to-resolve-azure-site-recovery-agent-issues-after-disabling-tls-1)

### <a name="configuration-server-minimum-requirements"></a>Configuration minimale requise du serveur
Le tableau suivant présente la configuration minimale requise pour le matériel, le logiciel et le réseau pour un serveur de configuration.
[!INCLUDE [site-recovery-configuration-server-requirements](../../includes/site-recovery-configuration-and-scaleout-process-server-requirements.md)]

> [!NOTE]
> Les serveurs proxy HTTPS ne sont pas pris en charge par le serveur de configuration.

## <a name="choose-your-protection-goals"></a>Sélectionner vos objectifs en matière de protection

1. Dans le portail Azure, accédez au panneau des coffres **Recovery Services**, puis sélectionnez votre coffre.
2. Dans le menu **Ressource** du coffre, cliquez sur **Prise en main** > **Site Recovery** > **Étape 1 : préparer l’infrastructure** > **Objectif de protection**.

    ![Capture d'écran montrant où sélectionner l'objectif de protection.](./media/physical-azure-set-up-source/choose-goals.png)
3. Dans la zone **Objectif de protection**, sélectionnez **To Azure** (Vers Azure) et **Non virtualisé/Autre**, puis cliquez sur **OK**.

    ![Sélectionner des objectifs](./media/physical-azure-set-up-source/physical-protection-goal.png)

## <a name="set-up-the-source-environment"></a>Configurer l’environnement source

1. Dans **Préparer la source**, si vous n’avez pas de serveur de configuration, cliquez sur **+Serveur de configuration** afin d’en ajouter un.

   ![Capture d'écran montrant comment sélectionner le serveur de configuration.](./media/physical-azure-set-up-source/plus-config-srv.png)
2. Dans le panneau **Ajouter un serveur**, vérifiez que **Serveur de configuration** s’affiche dans **Type de serveur**.
4. Téléchargez le fichier d’installation unifiée de Site Recovery.
5. Téléchargez la clé d’inscription du coffre. Vous avez besoin de la clé d’inscription lorsque vous exécutez le programme d’installation unifiée. Une fois générée, la clé reste valide pendant 5 jours.

    ![Configurer la source](./media/physical-azure-set-up-source/set-source2.png)
6. Sur la machine qui vous sert de serveur de configuration, exécutez le **programme d’installation unifiée Azure Site Recovery** afin d’installer le serveur de configuration, le serveur de processus et le serveur cible maître.

#### <a name="run-azure-site-recovery-unified-setup"></a>Exécuter le programme d’installation unifiée Azure Site Recovery

> [!TIP]
> L’inscription du serveur de configuration échoue si l’horloge système de vos ordinateurs diffère de l’heure locale de plus de cinq minutes. Synchronisez votre horloge système avec un [serveur de temps](/windows-server/networking/windows-time-service/windows-time-service-top) avant de commencer l’installation.

[!INCLUDE [site-recovery-add-configuration-server](../../includes/site-recovery-add-configuration-server.md)]

> [!NOTE]
> Le serveur de configuration peut être installé via une ligne de commande. [Plus d’informations](physical-manage-configuration-server.md#install-from-the-command-line)


## <a name="common-issues"></a>Problèmes courants

[!INCLUDE [site-recovery-vmware-to-azure-install-register-issues](../../includes/site-recovery-vmware-to-azure-install-register-issues.md)]


## <a name="next-steps"></a>Étapes suivantes

L’étape suivante consiste en la [configuration de votre environnement cible](physical-azure-set-up-target.md) dans Azure.
