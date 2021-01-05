---
title: Fournisseurs Azure Monitor pour solutions SAP | Microsoft Docs
description: Cet article fournit des réponses aux questions fréquemment posées sur Azure Monitor pour fournisseurs de solutions SAP.
author: rdeltcheva
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 06/30/2020
ms.author: radeltch
ms.reviewer: cynthn
ms.openlocfilehash: 056eba8694d1727350809121f763181e3cdbdc64
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968602"
---
# <a name="azure-monitor-for-sap-solutions-providers-preview"></a>Fournisseurs Azure Monitor pour solutions SAP (préversion)

## <a name="overview"></a>Vue d’ensemble  

Dans le contexte de Azure Monitor pour solutions SAP, un *type fournisseur*  fait référence à un *fournisseur* spécifique. Par exemple *SAP HANA*, qui est configuré pour un composant spécifique au sein du paysage SAP, comme la base de données SAP HANA. Un fournisseur contient les informations de connexion pour le composant correspondant et permet de collecter des données de télémétrie à partir de ce composant. Une ressource Azure Monitor pour solutions SAP (également appelée ressource du moniteur SAP) peut être configurée avec plusieurs fournisseurs du même type fournisseur ou plusieurs fournisseurs de plusieurs types fournisseur.
   
Les clients peuvent choisir de configurer différents types fournisseur pour activer la collecte des données à partir du composant correspondant dans leur paysage SAP. Par exemple, les clients peuvent configurer un fournisseur pour le type fournisseur SAP HANA, un autre fournisseur pour le type fournisseur de cluster haute disponibilité, etc.  

Les clients peuvent également choisir de configurer plusieurs fournisseurs d’un type fournisseur spécifique pour réutiliser la même ressource du moniteur SAP et le groupe géré associé. Apprenez-en davantage sur le groupe de ressources managé. Pour la préversion publique, les types fournisseur suivants sont pris en charge :   
- SAP HANA
- Cluster haute disponibilité
- Microsoft SQL Server

![Fournisseurs Azure Monitor pour solutions SAP](./media/azure-monitor-sap/azure-monitor-providers.png)

Il est recommandé aux clients de configurer au moins un fournisseur à partir des types fournisseur disponibles au moment du déploiement de la ressource du moniteur SAP. En configurant un fournisseur, les clients lancent la collecte de données à partir du composant correspondant pour lequel le fournisseur est configuré.   

Si les clients ne configurent aucun fournisseur au moment du déploiement de la ressource du moniteur SAP, bien que la ressource du moniteur SAP soit déployée avec succès, aucune donnée de télémétrie n’est collectée. Les clients ont la possibilité d’ajouter des fournisseurs après le déploiement via la ressource du moniteur SAP au sein de Portail Azure. Les clients peuvent ajouter ou supprimer des fournisseurs de la ressource du moniteur SAP à tout moment.

> [!Tip]
> Si vous souhaitez que Microsoft implémente un fournisseur spécifique, laissez vos commentaires via le lien à la fin de ce document ou contactez votre équipe de compte.  

## <a name="provider-type-sap-hana"></a>Type fournisseur SAP HANA

Les clients peuvent configurer un ou plusieurs fournisseurs de type fournisseur *SAP HANA* pour activer la collecte des données à partir de la base de données SAP HANA. Le fournisseur SAP HANA se connecte à la base de données SAP HANA sur le port SQL, extrait les données de télémétrie de la base de données et les envoie (push) à l’espace de travail Log Analytics de l’abonnement client. Le fournisseur SAP HANA collecte des données toutes les minutes depuis la base de données SAP HANA.  

En préversion publique, les clients peuvent s’attendre à voir les données suivantes avec le fournisseur SAP HANA : Utilisation de l’infrastructure sous-jacente, état de l’hôte SAP HANA, réplication du système SAP HANA et données de télémétrie de Sauvegarde Microsoft Azure SAP HANA. Pour configurer le fournisseur SAP HANA, l’adresse IP de l’hôte, le numéro de port SQL HANA et le nom d’utilisateur et le mot de passe SYSTEMDB sont requis. Il est recommandé aux clients de configurer le fournisseur SAP HANA par rapport à SYSTEMDB. Toutefois, des fournisseurs supplémentaires peuvent être configurés pour d’autres locataires de base de données.

![Fournisseurs Azure Monitor pour solutions SAP : SAP HANA](./media/azure-monitor-sap/azure-monitor-providers-hana.png)

## <a name="provider-type-high-availability-cluster"></a>Type fournisseur de cluster haute disponibilité
Les clients peuvent configurer un ou plusieurs fournisseurs de type fournisseur de *cluster haute disponibilité* pour activer la collecte de données à partir du cluster Pacemaker dans le paysage SAP. Le fournisseur de cluster haute disponibilité se connecte à Pacemaker, à l’aide du point de terminaison [ha_cluster_exporter](https://github.com/ClusterLabs/ha_cluster_exporter), extrait les données de télémétrie de la base de données et les envoie (push) à l’espace de travail Log Analytics de l’abonnement client. Le fournisseur de cluster haute disponibilité collecte les données toutes les 60 secondes depuis Pacemaker.  

En préversion publique, les clients peuvent s’attendre à voir les données suivantes avec le fournisseur de cluster :   
 - État du cluster représenté sous la forme d’un regroupement du nœud et de l’état de la ressource 
 - [autres](https://github.com/ClusterLabs/ha_cluster_exporter/blob/master/doc/metrics.md) 

![Fournisseurs Azure Monitor pour solutions SAP : cluster haute disponibilité](./media/azure-monitor-sap/azure-monitor-providers-pacemaker-cluster.png)

Pour configurer un fournisseur de cluster haute disponibilité, deux étapes principales sont nécessaires :

1. Installer [ha_cluster_exporter](https://github.com/ClusterLabs/ha_cluster_exporter) dans *chaque* nœud dans le cluster Pacemaker.

   Vous avez deux options pour installer ha_cluster_exporter :
   
   - Utilisez des scripts Azure Automation pour déployer un cluster haute disponibilité. Les scripts installent [ha_cluster_exporter](https://github.com/ClusterLabs/ha_cluster_exporter) sur chaque nœud de cluster.  
   - Effectuez une [installation manuelle](https://github.com/ClusterLabs/ha_cluster_exporter#manual-clone--build). 

2. Configurez un fournisseur de cluster haute disponibilité dans *chaque* nœud dans le cluster Pacemaker.

   Pour configurer le fournisseur de cluster haute disponibilité, vous devez fournir les informations suivantes :
   
   - **Nom**. Nom de ce fournisseur. Il doit être unique pour cette instance de solutions Azure Monitor pour SAP.
   - **Point de terminaison Prometheus**. Généralement, http\://\<servername or ip address\>:9664/Metrics.
   - **SID**. Pour les systèmes SAP, utilisez le SID SAP. Pour les autres systèmes (par exemple, les clusters NFS), utilisez un nom à trois caractères pour le cluster. Le SID doit être différent des autres clusters qui sont surveillés.   
   - **Nom du cluster**. Nom du rôle utilisé lors de la création du cluster. Le nom du cluster se trouve dans la propriété du cluster `cluster-name`.
   - **Hostname**. Nom d'hôte Linux de la machine virtuelle.  

## <a name="provider-type-microsoft-sql-server"></a>Type fournisseur Microsoft SQL Server

Les clients peuvent configurer un ou plusieurs fournisseurs de type fournisseur *Microsoft SQL Server* pour activer la collecte des données à partir de [SQL Server sur les machines virtuelles](https://azure.microsoft.com/services/virtual-machines/sql-server/). Le fournisseur SQL Server se connecte à Microsoft SQL Server sur le port SQL, extrait les données de télémétrie de la base de données et les envoie (push) à l’espace de travail Log Analytics de l’abonnement client. Le SQL Server doit être configuré pour l’authentification SQL et un compte de connexion SQL Server, avec la base de données SAP comme base de données par défaut du fournisseur, doit être créé. Le fournisseur SQL Server collecte des données entre toutes les 60 secondes à toutes les heures, à partir de SQL Server.  

En préversion publique, les clients peuvent s’attendre à voir les données suivantes avec le fournisseur SQL Server : utilisation de l’infrastructure sous-jacente, instructions SQL principales, table la plus grande, problèmes enregistrés dans les journaux d’erreurs SQL Server, processus bloquants, etc.  

Pour configurer le fournisseur Microsoft SQL Server, l’ID du système SAP, l’adresse IP de l’hôte, le numéro de port SQL Server ainsi que le nom et le mot de passe du compte de connexion SQL Server sont requis.

![Fournisseurs Azure Monitor pour solutions SAP : SQL](./media/azure-monitor-sap/azure-monitor-providers-sql.png)

## <a name="next-steps"></a>Étapes suivantes

- Créez votre première ressource Azure Monitor pour solutions SAP.
- Avez-vous des questions sur Azure Monitor pour solutions SAP ? Consultez la section [FAQ](./azure-monitor-faq.md)
