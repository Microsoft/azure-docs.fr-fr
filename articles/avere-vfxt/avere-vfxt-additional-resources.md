---
title: Liens vers des informations supplémentaires sur Avere vFXT pour Azure
description: Utilisez ces ressources pour obtenir des informations supplémentaires sur Avere vFXT pour Azure, notamment la documentation sur les clusters Avere et la documentation sur la gestion vFXT.
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 01/13/2020
ms.author: rohogue
ms.openlocfilehash: 6f9a9f8e6d2a279871ded3dff614be99dd2901df
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88271088"
---
# <a name="additional-documentation"></a>Documentation complémentaire

Cet article fournit des liens vers de la documentation complémentaire sur l’interface de gestion du Panneau de configuration Avere et sujets connexes.

## <a name="avere-cluster-documentation"></a>Documentation sur le cluster Avere

Vous trouverez de la documentation complémentaire sur le cluster Avere sur le site web <https://azure.github.io/Avere/>. Les documents suivants peuvent vous aider à comprendre comment utiliser les fonctionnalités du cluster et configurer ses paramètres.

* Le [guide de création d’un cluster FXT](<https://azure.github.io/Avere/#fxt_cluster>) s’applique principalement aux clusters composés de nœuds matériels physiques, mais certaines informations de ce guide s’appliquent également aux clusters vFXT. En particulier, les sections suivantes peuvent être utiles aux administrateurs de nouveaux clusters vFXT :
  * La section [Customizing Support and Monitoring Settings](<https://azure.github.io/Avere/legacy/create_cluster/4_8/html/config_support.html#config-support>) explique comment personnaliser les paramètres de chargement des données de support et comment activer la supervision à distance.
  * La section [Configuring VServers and Global Namespace](<https://azure.github.io/Avere/legacy/create_cluster/4_8/html/config_vserver.html#config-vserver>) fournit des informations sur la création d’un espace de noms accessible par les clients.
  * La section [Configuring DNS for the Avere cluster](<https://azure.github.io/Avere/legacy/create_cluster/4_8/html/config_network.html#dns-overview>) explique comment configurer un DNS en tourniquet (round-robin).
  * [Adding Back-end Storage](<https://azure.github.io/Avere/legacy/create_cluster/4_8/html/config_core_filer.html#add-core-filer>) décrit comment ajouter des systèmes de stockage back-end.

* Le [guide de configuration du cluster](<https://azure.github.io/Avere/#operations>) fournit des informations de référence complètes sur les paramètres et options d’un cluster Avere. Un cluster vFXT utilise seulement une partie de ces options, mais il partage la plupart des pages de configuration.

* Le [guide du tableau de bord](<https://azure.github.io/Avere/#operations>) explique comment utiliser les fonctionnalités de supervision du cluster situées dans le Panneau de configuration Avere.

## <a name="vfxt-creation-and-management-documentation"></a>Documentation sur la création et la gestion de vFXT

Un guide complet sur l’utilisation de vfxt.py, un utilitaire pour créer et gérer des clusters cloud basé sur un script, est fourni sur GitHub : [Gestion des clusters cloud avec vfxt.py](https://github.com/Azure/AvereSDK/blob/master/docs/README.md)
