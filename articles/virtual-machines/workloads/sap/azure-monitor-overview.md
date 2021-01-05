---
title: Présentation et architecture d’Azure Monitor pour solutions SAP | Microsoft Docs
description: Cet article fournit des réponses aux questions fréquemment posées sur Azure Monitor pour solutions SAP
author: rdeltcheva
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 06/30/2020
ms.author: radeltch
ms.reviewer: cynthn
ms.openlocfilehash: bcb912a24dfb2a5e78719cf9010fd23afe0df185
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484394"
---
# <a name="azure-monitor-for-sap-solutions-preview"></a>Azure Monitor pour solutions SAP (préversion)

## <a name="overview"></a>Vue d’ensemble

Azure Monitor pour les solutions SAP est un produit de surveillance Azure natif pour les clients qui exécutent leurs paysages SAP sur Azure. Le produit fonctionne avec [SAP sur des machines virtuelles Azure](./hana-get-started.md) et [SAP sur des instances volumineuses Azure](./hana-overview-architecture.md).
Avec Azure Monitor pour les solutions SAP, les clients peuvent collecter des données de télémétrie à partir de l’infrastructure Azure et des bases de données dans un emplacement central et établir une corrélation visuelle entre les données de télémétrie pour accélérer le dépannage.

Les solutions Azure Monitor pour SAP sont proposées via la place de marché Azure. Elles offrent une expérience d’installation simple et intuitive et il suffit de quelques clics pour déployer la ressource pour Azure Monitor pour les solutions SAP (appelée **Ressources d’analyse SAP**).

Les clients peuvent surveiller différents composants d’un paysage SAP, tels que les machines virtuelles Azure, le cluster à haute disponibilité, la base de données SAP HANA, etc. en ajoutant le **fournisseur** de ce composant.

Infrastructure prise en charge :

- Machine virtuelle Azure
- Instance volumineuse Azure

Bases de données prises en charge :
- Base de données SAP HANA
- Microsoft SQL Server

Les solutions Azure Monitor pour solutions SAP tirent parti de la puissance des fonctionnalités [Azure Monitor](../../../azure-monitor/overview.md) existantes, telles que l’analytique des journaux d'activité et les [classeurs](../../../azure-monitor/platform/workbooks-overview.md) pour fournir des fonctionnalités de surveillance supplémentaires. Les clients peuvent créer des [visualisations personnalisées](../../../azure-monitor/platform/workbooks-overview.md#getting-started) en modifiant les classeurs par défaut fournis par Azure Monitor pour solutions SAP, écrire des [requêtes personnalisées](../../../azure-monitor/log-query/log-analytics-tutorial.md) et créer des [alertes personnalisées](../../../azure-monitor/learn/tutorial-response.md) à l’aide de l’espace de travail Azure Log Analytics, tirer parti d’une [période de rétention flexible](../../../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period) et connecter les données de surveillance à leur système de création de tickets.

## <a name="what-data-does-azure-monitor-for-sap-solutions-collect"></a>Quelles sont les données collectées par Azure Monitor pour solutions SAP ?

La collecte de données dans Azure Monitor pour solutions SAP dépend des fournisseurs configurés par les clients. La préversion publique collecte les données suivantes.

Télémétrie de cluster Pacemaker haute disponibilité :
- État du nœud, de la ressource et de l’appareil SBD
- Contraintes d’emplacement de Pacemaker
- État des voix et des anneaux du quorum
- [Autres](https://github.com/ClusterLabs/ha_cluster_exporter/blob/master/doc/metrics.md)

Télémétrie SAP HANA :
- Processeur, mémoire, disque et utilisation réseau
- Réplication du système HANA (HSR)
- Sauvegarde HANA
- État de l’hôte HANA
- Rôles de serveur d’index et de serveur de noms

Télémétrie du serveur Microsoft SQL :
- processeur, mémoire, utilisation du disque
- Nom d’hôte, nom de l’instance SQL, ID du système SAP
- Requêtes de lots, compilations et espérance de vie d’une page dans le temps
- Les 10 instructions SQL les plus onéreuses dans le temps
- Les 12 plus grandes tables dans le système SAP
- Problèmes enregistrés dans les journaux d’erreurs SQL Server
- Processus de blocage et statistiques d’attente SQL dans le temps

## <a name="data-sharing-with-microsoft"></a>Partage de données avec Microsoft

Azure Monitor pour solutions SAP collecte les métadonnées système pour offrir une meilleure prise en charge de nos clients SAP sur Azure. Aucune information d’identification personnelle (PII)/information d’identification de l’utilisateur final (EUII) n’est collectée.
Les clients peuvent activer le partage de données avec Microsoft au moment de la création de la ressource Azure Monitor pour solutions SAP en choisissant *Partager* dans la liste déroulante.
Il est vivement recommandé aux clients d’activer le partage de données, car ainsi les équipes de support et d’ingénierie de Microsoft ont plus d’informations sur l’environnement du client et fournissent une prise en charge améliorée à nos clients SAP stratégiques sur Azure.

## <a name="architecture-overview"></a>Vue d’ensemble de l’architecture

À un niveau élevé, le diagramme suivant explique comment les Azure Monitor pour solutions SAP collecte les données de télémétrie à partir de la base de données SAP HANA. L’architecture est agnostique si SAP HANA est déployée sur des machines virtuelles Azure ou des instances volumineuses Azure.

![Architecture d’Azure Monitor pour solutions SAP](./media/azure-monitor-sap/azure-monitor-architecture.png)

Les composants clés de l’architecture sont les suivants :
- Portail Azure : point de départ pour les clients. Les clients peuvent accéder à la place de marché au sein du portail Azure et découvrir Azure Monitor pour solutions SAP.
- Ressource Azure Monitor pour solutions SAP : un emplacement d’arrivée pour que les clients affichent la télémétrie de surveillance
- Groupe de ressources managées : déployé automatiquement dans le cadre d’Azure Monitor pour le déploiement de ressources pour solutions SAP. Les ressources déployées dans le groupe de ressources managées aident à collecter des données de télémétrie. Les ressources clés déployées et leurs objectifs sont les suivantes :
   - Machine virtuelle Azure : Également appelée *machine virtuelle du collecteur*. Il s’agit d’une machine virtuelle Standard_B2ms. L’objectif principal de cette machine virtuelle est d’héberger la *charge utile de surveillance*. La charge utile de surveillance fait référence à la logique de la télémétrie de collecte des données à partir des systèmes sources et du transfert des données collectées vers l’infrastructure de surveillance. Dans le diagramme ci-dessus, la charge utile de surveillance contient la logique permettant la connexion à la base de données SAP HANA via le port SQL.
   - [Azure Key Vault](../../../key-vault/general/basic-concepts.md) : cette ressource est déployée pour conserver en toute sécurité les informations d’identification de la base de données SAP HANA et pour stocker des informations sur les [fournisseurs](./azure-monitor-providers.md).
   - Espace de travail Log Analytics : destination où résident les données de télémétrie.
      - La visualisation est basée sur la télémétrie dans Log Analytics à l’aide de [classeurs Azure](../../../azure-monitor/platform/workbooks-overview.md). Les clients peuvent personnaliser la visualisation. Les clients peuvent également épingler leurs classeurs ou une visualisation spécifique des classeurs dans le tableau de bord Azure pour une fonctionnalité d’actualisation automatique avec une granularité minimale de 30 minutes.
      - Les clients peuvent utiliser leur espace de travail existant dans le même abonnement que la ressource de surveillance SAP en choisissant cette option au moment du déploiement.
      - Les clients peuvent utiliser le langage de requête Kusto (KQL) pour exécuter des [requêtes](../../../azure-monitor/log-query/log-query-overview.md) par rapport aux tables brutes dans l’espace de travail Log Analytics. Examinons les *journaux d’activité personnalisés*.

> [!Note]
> Les clients sont responsables de la mise à jour corrective et de la maintenance de la machine virtuelle déployée dans le groupe de ressources managées.

> [!Tip]
> Les clients peuvent choisir d’utiliser un espace de travail Log Analytics existant pour la collecte de données de télémétrie s’ils sont déployés dans le même abonnement Azure que la ressource pour Azure Monitor pour solutions SAP.

### <a name="architecture-highlights"></a>Points forts de l’architecture

Voici les principales caractéristiques de l’architecture :
 - **Multi-instance** : les clients peuvent créer des analyses pour plusieurs instances d’un type de composant donné (par exemple, base de données HANA, cluster HA, Microsoft SQL Server) sur plusieurs SID SAP au sein d’un réseau virtuel avec une seule ressource d’Azure Monitor pour solutions SAP.
 - **Multifournisseur** : le diagramme d’architecture ci-dessus montre le fournisseur SAP HANA à titre d’exemple. De même, les clients peuvent configurer des fournisseurs supplémentaires pour les composants correspondants (par exemple, base de données HANA, cluster HA, Microsoft SQL Server) pour collecter des données à partir de ces composants.
 - **Open source** : le code source d’Azure Monitor pour solutions SAP est disponible dans [GitHub](https://github.com/Azure/AzureMonitorForSAPSolutions). Les clients peuvent se référer au code du fournisseur et en apprendre plus sur le produit, contribuer ou partager des commentaires.
 - **Requêtes de framework extensibles** : les requêtes SQL pour la collecte des données de télémétrie sont écrites en [JSON](https://github.com/Azure/AzureMonitorForSAPSolutions/blob/master/sapmon/content/SapHana.json). Des requêtes SQL supplémentaires pour collecter plus de données de télémétrie peuvent facilement être ajoutées. Les clients peuvent demander l’ajout de données de télémétrie spécifiques à Azure Monitor pour solutions SAP en laissant un commentaire via le lien fourni à la fin de ce document, ou en contactant l’équipe chargée de leur compte.

## <a name="pricing"></a>Tarifs
Azure Monitor pour solutions SAP est un produit gratuit (pas de frais de licence). Le paiement des composants sous-jacents dans le groupe de ressources managées incombe aux clients.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les fournisseurs et créer votre première ressource Azure Monitor pour solutions SAP.
 - En savoir plus sur les [fournisseurs](./azure-monitor-providers.md)
 - [Déployer Azure Monitor pour les solutions SAP avec Azure PowerShell](azure-monitor-sap-quickstart-powershell.md)
 - Avez-vous des questions sur Azure Monitor pour solutions SAP ? Consultez la section [FAQ](./azure-monitor-faq.md)