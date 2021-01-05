---
title: Vue d’ensemble du déploiement - Avere vFXT pour Azure
description: Découvrez dans cette présentation comment déployer un cluster Avere vFXT pour Azure. Les articles connexes comportent des instructions de déploiement spécifiques.
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 01/13/2020
ms.author: rohogue
ms.openlocfilehash: 4c63fdf2164dd4dce12912669eec29c79755cc2a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88271224"
---
<!-- filename is linked to in the marketplace template, make sure it gets a redirect if we rename it -->

# <a name="avere-vfxt-for-azure---deployment-overview"></a>Avere vFXT pour Azure - Vue d’ensemble du déploiement

Cet article offre une vue d’ensemble de la procédure à suivre pour obtenir un cluster Avere vFXT pour Azure opérationnel.

Plusieurs tâches sont nécessaires avant et après la création du cluster vFXT à partir de la Place de marché Azure. Le fait de bien comprendre l’ensemble du processus va vous aider à mesurer l’ampleur des efforts nécessaires.

## <a name="deployment-steps"></a>Étapes du déploiement

Après avoir [planifié votre système](avere-vfxt-deploy-plan.md), vous pouvez commencer à créer votre cluster Avere vFXT.

Dans la Place de marché Azure, un modèle Azure Resource Manager collecte les informations nécessaires et déploie automatiquement le cluster entier.

Une fois que le cluster vFXT est opérationnel, il reste des étapes de configuration à effectuer avant de l’utiliser. Si vous avez créé un nouveau conteneur de stockage Blob, vous pouvez y déplacer vos données. Si vous utilisez un système de stockage NAS, vous devez l’ajouter une fois le cluster créé. Vous pouvez connecter des clients au cluster.

Voici une vue d’ensemble des étapes à suivre.

1. Configuration préalable requise

   Avant de créer une machine virtuelle, vous devez créer un abonnement pour le projet Avere vFXT, configurer la propriété de l’abonnement, vérifier les quotas et demander une augmentation si nécessaire, puis accepter les conditions d’utilisation du logiciel Avere vFXT. Pour obtenir des instructions détaillées, consultez [Se préparer à la création du système Avere vFXT](avere-vfxt-prereqs.md).

1. Créer le cluster Avere vFXT

   Utilisez la Place de marché Azure pour créer le cluster Avere vFXT pour Azure. Un modèle collecte les informations requises et exécute des scripts pour créer le produit final.

   La création du cluster implique ces étapes, toutes exécutées via le modèle de la Place de marché :

   * Création d’une infrastructure réseau et de groupes de ressources, si nécessaire
   * Créer un contrôleur de cluster

     Le contrôleur de cluster est une machine virtuelle simple qui réside dans le même réseau virtuel que le cluster Avere vFXT et dispose du logiciel personnalisé requis pour créer et gérer le cluster. Le contrôleur crée les nœuds vFXT et forme le cluster. Il fournit également une interface de ligne de commande pour gérer le cluster pendant toute sa durée de vie.

     Si vous créez un sous-réseau ou réseau virtuel lors du déploiement, votre contrôleur a une adresse IP publique. Cela signifie que le contrôleur peut servir d’hôte de saut pour la connexion au cluster depuis l’extérieur du réseau virtuel.

   * Création de machines virtuelles de nœud de cluster

   * Créer le cluster à partir des nœuds individuels

   * Création facultative d’un conteneur de blobs avec configuration en tant que stockage back-end pour le cluster

   La création du cluster est décrite en détail dans [Déployer le cluster vFXT](avere-vfxt-deploy.md).

1. Configurer le cluster

   Connectez-vous à l’interface de configuration Avere vFXT (panneau de configuration Avere) pour personnaliser les paramètres du cluster. Choisissez la supervision de la prise en charge et ajoutez votre système de stockage si vous utilisez un stockage matériel ou des conteneurs d’objets blob supplémentaires.

   * [Accéder au cluster vFXT](avere-vfxt-cluster-gui.md)
   * [Activer la prise en charge](avere-vfxt-enable-support.md)
   * [Configurer le stockage](avere-vfxt-add-storage.md) (si nécessaire)

1. Monter des clients

   Suivez les instructions dans [Monter le cluster Avere vFXT](avere-vfxt-mount-clients.md) pour découvrir l’équilibrage de charge et savoir comment les ordinateurs clients doivent monter le cluster.

1. Ajouter des données (si nécessaire)

   Dans la mesure où Avere vFXT est un cache multiclient scalable, la meilleure façon de déplacer les données vers un nouveau conteneur de stockage back-end consiste à utiliser une stratégie de copie de fichiers multiclient et multithread.

   Si vous avez besoin de déplacer des données de plage de travail vers un nouveau conteneur de blobs ou un autre système de stockage back-end, suivez les instructions figurant à la page [Déplacer des données vers le cluster vFXT](avere-vfxt-data-ingest.md).

## <a name="next-steps"></a>Étapes suivantes

Passez à [Préparation à la création d’Avere vFXT](avere-vfxt-prereqs.md) pour effectuer les tâches préalables.
