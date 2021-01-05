---
title: Intégrer Azure Automation Update Management à Microsoft Endpoint Configuration Manager
description: Cet article explique comment configurer Microsoft Endpoint Configuration Manager avec Update Management pour déployer des mises à jour logicielles sur des clients gérés.
services: automation
ms.subservice: update-management
ms.date: 07/28/2020
ms.topic: conceptual
ms.openlocfilehash: 5bfa4bf8a9d20cc1a7baf1b5432e68501d65a509
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221651"
---
# <a name="integrate-update-management-with-microsoft-endpoint-configuration-manager"></a>Intégrer Update Management avec Microsoft Endpoint Configuration Manager

Les clients ayant investi dans Microsoft Endpoint Configuration Manager pour gérer des PC, des serveurs et d’autres appareils mobiles s’appuient aussi sur sa puissance et sa maturité pour gérer les mises à jour logicielles dans le cadre de leur cycle de gestion des mises à jour logicielles (SUM).

Vous pouvez mettre à jour des serveurs Windows gérés, et créer des rapports à partir de ceux-ci, en créant et en mettant en préproduction des déploiements de mises à jour logicielles dans Microsoft Endpoint Configuration Manager. En outre, vous pouvez obtenir l’état détaillé des déploiements de mises à jour terminés à l’aide d’[Update Management](overview.md). Si vous utilisez Microsoft Endpoint Configuration Manager pour générer des rapports de conformité de mises à jour et non pour gérer les déploiements de mises à jour avec vos serveurs Windows, vous pouvez continuer d’adresser des rapports à Microsoft Endpoint Configuration Manager tout en gérant les mises à jour de sécurité avec Azure Automation Update Management.

>[!NOTE]
>Bien que prenant en charge l’évaluation des mises à jour et les mises à jour correctives de Windows Server 2008 R2, Update Management ne prend pas en charge les clients gérés par Microsoft Endpoint Configuration Manager exécutant ce système d’exploitation.

## <a name="prerequisites"></a>Prérequis

* Vous devez avoir ajouté [Azure Automation Update Management](overview.md) à votre compte Automation.
* Les serveurs Microsoft actuellement gérés par votre environnement Windows Endpoint Configuration Manager doivent aussi adresser des rapports à l’espace de travail Log Analytics dans lequel la solution Update Management est aussi activée.
* Cette fonctionnalité est activée dans la branche actuelle de Microsoft Endpoint Configuration Manager, version 1606 et supérieure. Pour intégrer votre site d’administration centrale Microsoft Endpoint Configuration Manager ou un site principal autonome aux journaux Azure Monitor et importer des collections, consultez [Connecter Configuration Manager aux journaux Azure Monitor](../../azure-monitor/platform/collect-sccm.md).  
* Les agents Windows doivent être configurés pour communiquer avec un serveur WSUS (Windows Server Update Services) ou avoir accès à Microsoft Update s’ils ne reçoivent pas de mises à jour de sécurité de Microsoft Endpoint Configuration Manager.

La façon dont vous gérez les clients hébergés dans Azure IaaS avec votre environnement Microsoft Endpoint Configuration Manager existant dépend essentiellement de la connexion qui existe entre les centres de données Azure et votre infrastructure. Cette connexion a une incidence sur les changements de conception que vous pouvez être amené à apporter à votre infrastructure Microsoft Endpoint Configuration Manager, et sur les coûts liés à la prise en charge de ces changements. Pour comprendre les considérations de planification que vous devez évaluer avant de poursuivre, consultez [Configuration Manager sur Azure - Questions fréquentes (FAQ)](/configmgr/core/understand/configuration-manager-on-azure#networking).

## <a name="manage-software-updates-from-microsoft-endpoint-configuration-manager"></a>Gérer les mises à jour logicielles à partir de Microsoft Endpoint Configuration Manager

Si vous avez l’intention de continuer à gérer les déploiements de mises à jour à partir de Microsoft Endpoint Configuration Manager, effectuez les étapes suivantes. Azure Automation se connecte à Microsoft Endpoint Configuration Manager pour appliquer les mises à jour aux ordinateurs clients connectés à votre espace de travail Log Analytics. Le contenu des mises à jour est disponible dans le cache de l’ordinateur client comme si le déploiement était géré par Microsoft Endpoint Configuration Manager.

1. Créez un déploiement de mises à jour logicielles à partir du site de niveau supérieur dans votre hiérarchie Microsoft Endpoint Configuration Manager à l’aide du processus décrit dans [Déployer des mises à jour logicielles](/configmgr/sum/deploy-use/deploy-software-updates). Le seul paramètre qui doit être configuré différemment par rapport à un déploiement standard est l’option **Ne pas installer les mises à jour logicielles** pour contrôler le comportement de téléchargement du package de déploiement. Ce comportement est géré dans Update Management en créant un déploiement de mises à jour planifié à l’étape suivante.

2. Dans Azure Automation, sélectionnez **Update Management**. Créez un déploiement en suivant les étapes décrites dans [Création d’un déploiement de mises à jour](deploy-updates.md#schedule-an-update-deployment), puis sélectionnez **Groupes importés** dans la liste déroulante **Type** pour sélectionner le regroupement Microsoft Endpoint Configuration Manager approprié. Gardez à l’esprit les points importants suivants :

    a. Si une fenêtre de maintenance est définie dans le regroupement d’appareils Microsoft Endpoint Configuration Manager sélectionné, les membres du regroupement la respectent au détriment du paramètre **Durée** défini dans le déploiement planifié.

    b. Les membres du regroupement cible doivent disposer d’une connexion Internet (qu’elle soit directe, via un serveur proxy ou via la passerelle Log Analytics).

À l’issue du déploiement des mises à jour par le biais d’Azure Automation, les ordinateurs cibles membres du groupe d’ordinateurs sélectionné installent les mises à jour à l’heure planifiée à partir de leur cache client local. Vous pouvez [consulter l’état du déploiement des mises à jour](deploy-updates.md#check-deployment-status) pour surveiller les résultats de votre déploiement.

## <a name="manage-software-updates-from-azure-automation"></a>Gérer les mises à jour logicielles à partir d’Azure Automation

Pour gérer les mises à jour de machines virtuelles Windows Server qui sont des clients Microsoft Endpoint Configuration Manager, vous devez configurer la stratégie de configuration des clients afin de désactiver la fonctionnalité de gestion des mises à jour logicielles pour tous les clients gérés par Update Management. Par défaut, les paramètres client ciblent tous les appareils de la hiérarchie. Pour plus d’informations sur ce paramètre de stratégie et sur la façon de le configurer, consultez [Comment configurer les paramètres client dans Configuration Manager](/configmgr/core/clients/deploy/configure-client-settings).

Après avoir apporté cette modification à la configuration, vous créez un déploiement en suivant les étapes décrites dans [Création d’un déploiement de mises à jour](deploy-updates.md#schedule-an-update-deployment), puis vous sélectionnez **Groupes importés** dans la liste déroulante **Type** pour sélectionner le regroupement Microsoft Endpoint Configuration Manager approprié.

## <a name="next-steps"></a>Étapes suivantes

Pour configurer une planification d’intégration, consultez [Planifier un déploiement de mises à jour](deploy-updates.md#schedule-an-update-deployment).
