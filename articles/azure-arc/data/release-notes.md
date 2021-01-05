---
title: Services de données activés par Azure Arc – Notes de publication
description: Dernières notes de publication
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.date: 10/29/2020
ms.topic: conceptual
ms.openlocfilehash: 94074c2c5e11187252084832e5a20a197f6723fd
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93359814"
---
# <a name="release-notes---azure-arc-enabled-data-services-preview"></a>Notes de publication – Services de données activés par Azure Arc (préversion)

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="october-2020"></a>Octobre 2020 

Numéro de version d’Azure Data CLI (`azdata`) : 20.2.3. Téléchargez à l’adresse [https://aka.ms/azdata](https://aka.ms/azdata).

### <a name="breaking-changes"></a>Changements cassants

Cette version introduit les changements cassants suivants : 

* Dans la définition de ressource personnalisée PostgreSQL, le terme `shards` est renommé en `workers`. Ce terme (`workers`) correspond au nom du paramètre de ligne de commande.

* `azdata arc postgres server delete` demande une confirmation avant de supprimer une instance postgres.  Utilisez `--force` pour ignorer l’invite.

### <a name="additional-changes"></a>Modifications supplémentaires

* Un nouveau paramètre facultatif a été ajouté à `azdata arc postgres server create` appelé `--volume-claim mounts`. La valeur est une liste de montages de revendications de volume séparés par des virgules. Un montage de revendication de volume est une paire de type de volume et de nom PVC. Le seul type de volume actuellement pris en charge est `backup`.  Dans PostgreSQL, lorsque le type de volume est `backup`, le PVC est monté sur `/mnt/db-backups`.  Cela permet de partager des sauvegardes entre instances PostgresSQL afin que la sauvegarde d’une instance PostgresSQL puisse être restaurée dans une autre instance.

* Nouveaux noms courts pour les définitions de ressources personnalisées PostgresSQL : 

  * `pg11` 

  * `pg12`

* Le chargement de la télémétrie fournit à l’utilisateur l’un des éléments suivants :

   * Nombre de points chargés sur Azure

     ou 

   * Si aucune donnée n’a été chargée sur Azure, une invite pour essayer à nouveau.

* `azdata arc dc debug copy-logs` lit maintenant aussi dans le dossier `/var/opt/controller/log` et collecte les journaux du moteur PostgreSQL sur Linux.

*   Affichage d’un indicateur de travail lors de la création et de la restauration de la sauvegarde avec PostgreSQL Hyperscale.

* `azdata arc postrgres backup list` comprend désormais des informations sur la taille de la sauvegarde.

* La propriété Nom de l’administrateur SQL Managed Instance a été ajoutée à la colonne de droite du panneau Vue d’ensemble dans le portail Azure.

* Azure Data Studio prend en charge la configuration du nombre de nœuds Worker, de vCore et des paramètres de mémoire pour PostgreSQL Hyperscale. 

* La préversion prend en charge la sauvegarde/restauration pour les versions 11 et 12 de Postgres.

## <a name="september-2020"></a>Septembre 2020

Les services de données activés par Azure Arc sont mis en production pour la préversion publique. Les services de données activés par Arc permettent de gérer les services de données en tout lieu.

- Instance managée SQL
- PostgreSQL Hyperscale

Pour obtenir des instructions, consultez [Présentation des services de données avec Azure Arc](overview.md)

## <a name="known-limitations-and-issues"></a>Limites et problèmes connus

- Les noms d’instance ne peuvent pas dépasser 13 caractères.
- Aucune mise à niveau sur place du contrôleur de données ou des instances de base de données Azure Arc.
- Les images conteneurs de services de données activés par Arc ne sont pas signées.  Il se peut que vous deviez configurer vos nœuds Kubernetes pour permettre l’extraction d’images de conteneur non signées.  Par exemple, si vous utilisez Docker en tant que runtime de conteneurs, vous pouvez définir la variable d’environnement DOCKER_CONTENT_TRUST=0 et redémarrer.  D’autres runtimes de conteneurs offrent des options similaires, par exemple dans [OpenShift](https://docs.openshift.com/container-platform/4.5/openshift_images/image-configuration.html#images-configuration-file_image-configuration).
- Impossible de créer des instances gérées SQL activées par Azure Arc, ou des groupes de serveurs PostgreSQL Hyperscale à partir du portail Azure.
- Pour le moment, si vous utilisez NFS, vous devez définir `allowRunAsRoot` sur `true` dans votre fichier de profil de déploiement avant de créer le contrôleur de données Azure Arc.
- Authentification de connexion SQL et PostgreSQL uniquement.  Aucune prise en charge d’Azure Active Directory ou d’Active Directory.
- La création d’un contrôleur de données sur OpenShift impose des contraintes de sécurité peu exigeantes.  Pour plus d'informations, reportez-vous à la documentation.
- Si vous utilisez un moteur Azure Kubernetes Service (moteur AKS) sur Azure Stack Hub avec un contrôleur de données Azure Arc et des instances de base de données, la mise à niveau vers une version plus récente de Kubernetes n’est pas prise en charge. Avant de mettre à niveau le cluster Kubernetes, désinstallez le contrôleur de données Azure Arc et toutes les instances de base de données.
- Pour les clusters Azure Kubernetes Service (AKS) qui s'étendent sur [plusieurs zones de disponibilité](../../aks/availability-zones.md), les services de données avec Azure Arc ne sont actuellement pas pris en charge. Pour contourner ce problème, lorsque vous créez le cluster AKS sur le portail Azure, si vous sélectionnez une région dans laquelle des zones sont disponibles, effacez toutes les zones du contrôle de sélection. Consultez le graphique suivant :

   :::image type="content" source="media/release-notes/aks-zone-selector.png" alt-text="Décochez les cases des différentes zones pour n'en spécifier aucune.":::


### <a name="known-issues-for-azure-arc-enabled-postgresql-hyperscale"></a>Problèmes connus pour PostgreSQL Hyperscale ave Azure Arc   

- La préversion ne prend pas en charge la sauvegarde/restauration pour le moteur de la version 11 de PostgreSQL. Elle prend uniquement en charge la sauvegarde/restauration pour la version 12 de PostgreSQL.
- `azdata arc dc debug copy-logs` ne collecte pas les journaux du moteur PostgreSQL sur Windows.
- La recréation d’un groupe de serveurs avec le nom d’un groupe de serveurs qui vient d’être supprimé peut échouer ou cesser de répondre. 
   - **Solution de contournement** : Ne réutilisez pas le même nom lorsque vous recréez un groupe de serveurs ou attendez l’équilibreur de charge ou le service externe du groupe de serveurs précédemment supprimé. En supposant que le nom du groupe de serveurs que vous avez supprimé était `postgres01` et qu’il était hébergé dans un espace de noms `arc`, avant de recréer un groupe de serveurs portant le même nom, attendez que `postgres01-external-svc` disparaisse de la sortie de la commande kubectl `kubectl get svc -n arc`.
 - Le chargement de la page de présentation et de la page de configuration Calcul + stockage dans Azure Data Studio est lent. 



## <a name="next-steps"></a>Étapes suivantes
  
> **Vous voulez juste essayer ?**  
> Démarrez rapidement avec [Démarrage rapide d’Azure Arc](https://github.com/microsoft/azure_arc#azure-arc-enabled-data-services) sur Azure Kubernetes Service (AKS), AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) ou sur une machine virtuelle Azure.

[Installer les outils clients](install-client-tools.md)

[Créer le contrôleur de données Azure Arc](create-data-controller.md) (nécessite l’installation préalable des outils clients)

[Créer une instance gérée Azure SQL sur Azure Arc](create-sql-managed-instance.md) (nécessite la création préalable d’un contrôleur de données Azure Arc)

[Créer un groupe de serveurs Azure Database pour PostgreSQL Hyperscale sur Azure Arc](create-postgresql-hyperscale-server-group.md) (nécessite la création préalable d’un contrôleur de données Azure Arc)
