---
title: Données à collecter à l’ouverture d’un cas pour Microsoft Azure Automation | Microsoft Docs
description: Cet article répertorie les informations que vous devez rassembler avant d’ouvrir un cas pour Azure Automation auprès du support Microsoft Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: top-support-issue, azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 09/23/2019
ms.author: v-miegge
ms.openlocfilehash: d2f97d0b889186324aef9613847e3eddbfe1eb02
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83684844"
---
# <a name="data-to-collect-when-opening-a-case-for-microsoft-azure-automation"></a>Données à collecter à l’ouverture d’un cas pour Microsoft Azure Automation

Cet article décrit certaines informations que vous devez rassembler avant d’ouvrir un cas pour Azure Automation auprès du support Microsoft Azure. Ces informations ne sont pas nécessaires pour ouvrir le cas. Toutefois, elles peuvent aider Microsoft à résoudre votre problème plus rapidement. En outre, l’ingénieur de support peut vous demander ces données une fois le cas ouvert.

## <a name="basic-data"></a>Données de base

Collectez les données de base décrites dans l’article de la base de connaissances [4034605 – Comment capturer des scripts de diagnostics Azure Automation](https://support.microsoft.com/help/4034605/how-to-capture-azure-automation-scripted-diagnostics).

## <a name="data-for-update-management-issues-on-linux"></a>Données pour les problèmes rencontrés avec Update Management sur Linux

1. Collectez les données décrites dans l’article [4034605](https://support.microsoft.com/help/4034605/how-to-capture-azure-automation-scripted-diagnostics) de la base de connaissances et exécutez l’outil de collecte de journaux suivant :

   [Collecteur de journaux de l’agent OMS pour Linux](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/tools/LogCollector/OMS_Linux_Agent_Log_Collector.md)
 
2. Compressez le contenu du dossier **/var/opt/Microsoft/omsagent/Run/automationworker/** , puis envoyez le fichier compressé au support Azure.
 
3. Vérifiez que l’ID de l’espace de travail que l’agent de Log Analytics pour Linux signale est identique à l’ID de l’espace de travail surveillé pour les mises à jour.

## <a name="data-for-update-management-issues-on-windows"></a>Données pour les problèmes rencontrés avec Update Management sur Windows

1. Collectez les données pour les éléments répertoriés dans [4034605](https://support.microsoft.com/help/4034605/how-to-capture-azure-automation-scripted-diagnostics).

2. Exportez les journaux d’événements suivants au format EVTX :

   * Système
   * Application
   * Sécurité
   * Operations Manager
   * Microsoft-SMA/journal des opérations

3. Vérifiez que l’ID de l’espace de travail auquel l’agent est associé correspond à l’ID de l’espace de travail surveillé pour les mises à jour Windows.

## <a name="data-for-job-issues"></a>Données pour les problèmes liés à des travaux

1. Collectez les données pour les éléments répertoriés dans [4034605](https://support.microsoft.com/help/4034605/how-to-capture-azure-automation-scripted-diagnostics).

2. Collectez l’ID de la tâche qui pose problème :

   1. Dans le portail Azure, accédez à **Comptes Automation**.
   2. Sélectionnez le compte Automation affecté par le problème et notez son nom.
   3. Sélectionnez **Travaux**.
   4. Choisissez le travail affecté par le problème.
   5. Dans le volet Résumé de la tâche, recherchez la valeur GUID dans **ID de tâche**.

   ![ID de travail dans le volet Résumé du travail](media/collect-data-microsoft-azure-automation-case/job-summary-job-id.png)

3. Collectez un exemple du script que vous exécutez.

4. Collectez les fichiers journaux :

   1. Dans le portail Azure, accédez à **Comptes Automation**.
   2. Sous Compte Automation, sélectionnez le compte affecté par le problème.
   3. Sélectionnez **Travaux**.
   4. Choisissez le travail affecté par le problème.
   5. Sélectionnez **Tous les journaux**.
   6. Collectez les données fournies dans le volet qui s’affiche.

   ![Données listées sous Tous les journaux](media/collect-data-microsoft-azure-automation-case/all-logs-data.png)

## <a name="data-for-module-issues"></a>Données pour les problèmes de modules

En plus des [éléments des données de base](#basic-data), rassemblez les informations suivantes :

* Les étapes que vous avez suivies, de sorte que le problème puisse être reproduit.
* Les captures d’écran des messages d’erreur, le cas échéant.
* Les captures d’écran des modules actuels et leurs numéros de version.

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’une aide supplémentaire :

* Obtenez des réponses de la part d’experts Azure via les [Forums Azure](https://azure.microsoft.com/support/forums/).
* Connectez-vous avec [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client en connectant la communauté Azure aux ressources appropriées (réponses, support et experts).
* Signaler un incident au support Azure Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.
