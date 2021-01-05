---
title: Azure VMware Solution by CloudSimple – Modèle d’autorisation de cloud privé
description: Décrit le modèle d’autorisation, les groupes et les catégories de cloud privé CloudSimple
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 28c4dc7831f97d66eb4d47f08e640344d5cca0d1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77014944"
---
# <a name="cloudsimple-private-cloud-permission-model-of-vmware-vcenter"></a>Modèle d’autorisation de cloud privé CloudSimple de VMware vCenter

CloudSimple conserve un accès administratif complet à l’environnement de cloud privé. Chaque client CloudSimple dispose de privilèges Administrateur suffisants pour pouvoir déployer et gérer les machines virtuelles dans son environnement.  Si nécessaire, vous pouvez élever temporairement vos privilèges pour effectuer des fonctions d’administration.

## <a name="cloud-owner"></a>Propriétaire de cloud

Lorsque vous créez un Cloud privé, un utilisateur **CloudOwner**  est créé dans le domaine d’authentification unique vCenter avec un accès **de rôle de propriétaire Cloud** pour gérer des objets dans le cloud privé. Cet utilisateur peut également configurer des [sources d’identité vCenter](set-vcenter-identity.md) supplémentaires et d’autres utilisateurs sur le cloud privé vCenter.

> [!NOTE]
> L’utilisateur par défaut de votre cloud privé CloudSimple vCenter est cloudowner@cloudsimple.local.

## <a name="user-groups"></a>Groupes d’utilisateurs

Un groupe nommé **Cloud-owner-Group** est créé lors du déploiement d’un cloud privé. Les utilisateurs de ce groupe peuvent administrer diverses parties de l’environnement vSphere sur le cloud privé. Ce groupe obtient automatiquement des privilèges **Cloud-Owner-Role** et l’utilisateur **CloudOwner** est ajouté en tant que membre de ce groupe.  CloudSimple crée des groupes supplémentaires disposant de privilèges limités pour faciliter la gestion.  Vous pouvez ajouter n'importe quel utilisateur à ces groupes préalablement créés. Les privilèges définis ci-dessous sont automatiquement attribués aux utilisateurs de ces groupes.

### <a name="pre-created-groups"></a>Groupes précréés

| Nom du groupe | Objectif | Role |
| -------- | ------- | ------ |
| Cloud-owner-Group | Les membres de ce groupe disposent de privilèges Administrateur sur le cloud privé vCenter | [Cloud-Owner-Role](#cloud-owner-role) |
| Cloud-Global-Cluster-Admin-Group | Les membres de ce groupe disposent de privilèges Administrateur sur le cluster vCenter | [Cloud-Cluster-Admin-Role](#cloud-cluster-admin-role) |
| Cloud-Global-Storage-admin-Group | Les membres de ce groupe peuvent gérer le stockage sur le cloud privé vCenter | [Cloud-Storage-Admin-Role](#cloud-storage-admin-role) |
| Cloud-Global-Network-Admin-Group | Les membres de ce groupe peuvent gérer les groupes de ports réseau et distribués sur le cloud privé vCenter | [Cloud-Network-Admin-Role](#cloud-network-admin-role) |
| Cloud-Global-VM-Admin-Group | Les membres de ce groupe peuvent gérer des machines virtuelles sur le cloud privé vCenter | [Cloud-VM-Admin-Role](#cloud-vm-admin-role) |

Pour accorder à des utilisateurs individuels des autorisations leur permettant de gérer le cloud privé, créez des comptes d’utilisateur à ajoutez-les aux groupes appropriés.

> [!CAUTION]
> Les nouveaux utilisateurs doivent être ajoutés uniquement à *Cloud-Owner-Group*, à *Cloud-Global-Cluster-Admin-Group*, à *Cloud-Global-Storage-Admin-Group*, à *Cloud-Global-Network-Admin-Group* ou à *Cloud-Global-VM-Admin-Group*.  Les utilisateurs ajoutés au groupe *Administrateurs* seront automatiquement supprimés.  Seuls des comptes de service doivent être ajoutés au groupe *Administrateurs*, et les comptes de service en doivent pas être utilisés pour se connecter à l’interface utilisateur web de vSphere.

## <a name="list-of-vcenter-privileges-for-default-roles"></a>Liste des privilèges vCenter pour les rôles par défaut

### <a name="cloud-owner-role"></a>Cloud-Owner-Role

| **Catégorie** | **Privilège** |
|----------|-----------|
| **Alarmes** | Confirmer la réception de l’alerte <br> Créer une alerte <br> Désactiver l’action d’alarme <br> Modifier l’alarme <br> Supprimer l’alarme <br> Définir l’état de l’alarme |
| **Autorisations** | Modifier l’autorisation |
| **Bibliothèque de contenu** | Ajouter un élément de bibliothèque <br> Créer une bibliothèque locale <br> Créer une bibliothèque abonnée <br> Supprimer un élément de bibliothèque <br> Supprimer une bibliothèque locale <br> Supprimer une bibliothèque abonnée <br> Télécharger les fichiers <br> Expulser un élément de bibliothèque <br> Expulser une bibliothèque abonnée <br> Importer un stockage <br> Informations d’abonnement de probe <br> Lire le stockage <br> Synchroniser un élément de bibliothèque <br> Synchroniser une bibliothèque abonnée <br> Introspection de type <br> Mettre à jour les paramètres de configuration du système <br> Mettre à jour les fichiers <br> Mettre à jour la bibliothèque <br> Mettre à jour un élément de bibliothèque <br> Mettre à jour une bibliothèque locale <br> Mettre à jour une bibliothèque abonnée <br> Afficher les paramètres de configuration |
| **Opérations de chiffrement** | Ajouter un disque <br> Clone <br> Decrypt (Déchiffrer) <br> Accès direct <br> Encrypt (Chiffrer) <br> Chiffrer nouveau <br> Gérer KMS <br> Gérer les stratégies de chiffrement <br> Gérer les clés <br> Migrer <br> Chiffrer à nouveau <br> Inscrire la machine virtuelle <br> Inscrire l’hôte |
| **groupe dvPort** | Créer <br> DELETE <br> Modifier <br> Opération de stratégie <br> Étendue de l’opération |
| **Magasins de données** | Allouer de l’espace <br> Parcourir les magasins de données <br> Configurer le magasin de données <br> Opérations de fichiers de bas niveau <br> Déplacer le magasin de données <br> Supprimer le magasin de données <br> Supprimer le fichier <br> Renommer le magasin de données <br> Mettre à jour les fichiers de machine virtuelle <br> Mettre à jour les métadonnées de machine virtuelle |
| **Agent Manager ESX** | Config <br> Modifier <br> Affichage |
| **Extension** | Inscrire l’extension <br> Annuler l’inscription de l’extension <br> Mettre à jour l'extension |
| **Fournisseur de statistiques externes**| Inscrire <br> Unregister <br> Update |
| **Folder** | Créer un dossier <br> Supprimer un dossier <br> Déplacer le dossier <br> Renommer le dossier |
| **Global** | Annuler la tâche <br> planification de la capacité <br> Diagnostics <br> Désactiver les méthodes <br> Activer les méthodes <br> Balise globale <br> Intégrité <br> Licences <br> Journaliser les événements <br> Gérer des attributs personnalisés <br> Proxy <br> Action de script <br> Gestionnaires d’appareils <br> Définir un attribut personnalisé <br> Balise système |
| **Fournisseur de mises à jour d’intégrité** | Inscrire <br> Unregister <br> Update |
| **Hôte > Configuration** | Configuration de partition de stockage |
| **Hôte > Inventaire** | Modifier le cluster |
| **Catégorisation vSphere** | Attribuer ou annuler l’attribution d’une balise vSphere <br> Créer une balise vSphere <br> Créer une catégorie de balise vSphere <br> Supprimer une balise vSphere <br> Supprimer une catégorie de balise vSphere <br> Modifier une balise vSphere <br> Modifier une catégorie de balise vSphere <br> Modifier le champ UsedBy pour la catégorie <br> Modifier le champ UsedBy pour la balise |
| **Réseau** | Attribuer un réseau <br> Configurer <br> Déplacer le réseau <br> Supprimer |
| **Performances** | Modifier les intervalles |
| **Profil de l’hôte** | Affichage |
| **Ressource** | Appliquer une recommandation <br> Attribuer vApp à une liste de ressources partagées <br> Attribuer une machine virtuelle à une liste de ressources partagées <br> Créer une liste de ressources partagées <br> Migrer une machine virtuelle hors tension <br> Migrer une machine virtuelle sous tension <br> Modifier la liste de ressources partagées <br> Déplacer la liste de ressources partagées <br> Interroger vMotion <br> Supprimer la liste de ressources partagées <br> Renommer la liste de ressources partagées |
| **Tâche planifiée** | Créer des tâches <br> Modifier la tâche <br> Supprimer la tâche <br> Exécuter la tâche |
| **Sessions** | Emprunter l'identité d'un utilisateur <br> Message <br> Valider la session <br> Afficher et arrêter des sessions |
| **Cluster du magasin de données** | Configurer un cluster de magasin de données |
| **Stockage basé sur les profils** | Mise à jour de stockage basé sur profil <br> Affichage de stockage basé sur profil |
| **Affichage de stockage** | Configurer le service <br> Affichage |
| **Tâches :** | Créer une tâche <br> Mettre à jour la tâche |
| **Transférer le service**| Gérer <br> Moniteur |
| **vApp** | Ajouter une machine virtuelle <br> Attribuer une liste de ressources partagées <br> Attribuer vApp <br> Clone <br> Créer <br> DELETE <br> Exporter <br> Importer <br> Déplacer <br> Mise hors tension <br> Mise sous tension <br> Renommer <br> Interrompre <br> Unregister <br> Afficher l’environnement OVF <br> Configuration de l’application vApp <br> Configuration de l'instance vApp <br> Configuration de vApp managedBy <br> Configuration de ressource vApp |
| **VRMPolicy** | Interroger VRMPolicy <br> Mettre à jour VRMPolicy |
| **Machine virtuelle -> Configuration** | Ajouter un disque existant <br> Ajouter un nouveau disque <br> Ajouter ou supprimer un appareil <br> Avancé <br> Modifier le nombre d’UC <br> Modifier la ressource <br> Configurer managedBy <br> Suivi des modifications du disque <br> Bail de disque <br> Afficher les paramètres de connexion <br> Étendre l’unité virtuelle <br> Héberger l’appareil USB <br> Mémoire <br> Modification des paramètres de l’appareil <br> Interroger la compatibilité de la tolérance de pannes <br> Interroger les fichiers sans propriétaire <br> Appareil brut <br> Recharger à partir du chemin d’accès <br> Supprimer le disque <br> Renommer <br> Réinitialiser les informations de l’invité <br> Définir l’annotation <br> Paramètres <br> Positionnement du fichier d’échange <br> Basculer le parent fourche <br> Déverrouiller la machine virtuelle <br> Mettre à niveau la compatibilité de la machine virtuelle |
| **Machine virtuelle > Opérations invité** | Modification des alias d’opération invité <br> Requête d'alias d'opération invité <br> Modification d’opération invité <br> Exécution du programme d’opération invité <br> Requêtes d’opération invité |
| **Machine virtuelle > Interaction** | Répondre à la question <br> Opération de sauvegarde de machines virtuelles <br> Configurer un média CD <br> Configurer une disquette <br> Interaction de la console <br> Créer une capture d’écran <br> Défragmenter tous les disques <br> Connexion de l’appareil <br> Glisser-déposer <br> Gestion du système d’exploitation invité par API VIX <br> Injecter les codes d’analyse USB HID <br> Suspendre ou annuler la suspension <br> Effectuer des opérations de réinitialisation ou de réduction <br> Mise hors tension <br> Mise sous tension <br> Enregistrer la session sur la machine virtuelle <br> Relire la session sur la machine virtuelle <br> Réinitialiser <br> Reprendre la tolérance de panne <br> Interrompre <br> Suspendre la tolérance de panne <br> Test de basculement <br> Redémarrage du test de la machine virtuelle secondaire <br> Désactiver la tolérance de panne <br> Activer la tolérance de panne <br> Installation des outils VMware |
| **Machine virtuelle > Inventaire** | Créer à partir de ce qui existe <br> Création <br> Déplacer <br> Inscrire <br> Supprimer <br> Unregister |
| **Machine virtuelle > Approvisionnement** | Autoriser l’accès au disque <br> Autoriser l’accès au fichier <br> Autoriser l’accès au disque en lecture seule <br> Autoriser le téléchargement de machines virtuelles <br> Autoriser le téléchargement de machines virtuelles <br> Cloner le modèle <br> Cloner la machine virtuelle <br> Créer un modèle à partir de la machine virtuelle <br> Personnaliser <br> Déployer un modèle <br> Marquer comme modèle <br> Marquer come machine virtuelle <br> Modifier la spécification de personnalisation <br> Promouvoir les disques <br> Lire les spécifications de personnalisation |
| **Machine virtuelle -> Configuration de service** | Autoriser les notifications <br> Autoriser l’interrogation des notifications d’événements globales <br> Gérer les configurations de service <br> Modifier la configuration du service <br> Interroger les configurations de service <br> Lire la configuration du service |
| **Machine virtuelle > Gestion des instantanés** | Créer l’instantané <br> Supprimer l’instantané <br> Renommer l'instantané <br> Restaurer l’instantané |
| **Machine virtuelle > Replication vSphere** | Configurer la réplication <br> Gérer la réplication <br> Superviser la réplication |
| **vService** | Créer une dépendance <br> Détruire la dépendance <br> Reconfigurer la configuration de la dépendance <br> Mettre à jour la dépendance |

### <a name="cloud-cluster-admin-role"></a>Cloud-Cluster-Admin-Role

| **Catégorie** | **Privilège** |
|----------|-----------|
| **Magasins de données** | Allouer de l’espace <br> Parcourir les magasins de données <br> Configurer le magasin de données <br> Opérations de fichiers de bas niveau <br> Supprimer le magasin de données <br> Renommer le magasin de données <br> Mettre à jour les fichiers de machine virtuelle <br> Mettre à jour les métadonnées de machine virtuelle |
| **Folder** | Créer un dossier <br> Supprimer un dossier <br> Déplacer le dossier <br> Renommer le dossier |
| **Hôte > Configuration**  | Configuration de partition de stockage |
| **Catégorisation vSphere** | Attribuer ou annuler l’attribution d’une balise vSphere <br> Créer une balise vSphere <br> Créer une catégorie de balise vSphere <br> Supprimer une balise vSphere <br> Supprimer une catégorie de balise vSphere <br> Modifier une balise vSphere <br> Modifier une catégorie de balise vSphere <br> Modifier le champ UsedBy pour la catégorie <br> Modifier le champ UsedBy pour la balise |
| **Réseau** | Attribuer un réseau |
| **Ressource** | Appliquer une recommandation <br> Attribuer vApp à une liste de ressources partagées <br> Attribuer une machine virtuelle à une liste de ressources partagées <br> Créer une liste de ressources partagées <br> Migrer une machine virtuelle hors tension <br> Migrer une machine virtuelle sous tension <br> Modifier la liste de ressources partagées <br> Déplacer la liste de ressources partagées <br> Interroger vMotion <br> Supprimer la liste de ressources partagées <br> Renommer la liste de ressources partagées |
| **vApp** | Ajouter une machine virtuelle <br> Attribuer une liste de ressources partagées <br> Attribuer vApp <br> Clone <br> Créer <br> DELETE <br> Exporter <br> Importer <br> Déplacer <br> Mise hors tension <br> Mise sous tension <br> Renommer <br> Interrompre <br> Unregister <br> Afficher l’environnement OVF <br> Configuration de l’application vApp <br> Configuration de l'instance vApp <br> Configuration de vApp managedBy <br> Configuration de ressource vApp |
| **VRMPolicy** | Interroger VRMPolicy <br> Mettre à jour VRMPolicy |
| **Machine virtuelle -> Configuration** | Ajouter un disque existant <br> Ajouter un nouveau disque <br> Ajouter ou supprimer un appareil <br> Avancé <br> Modifier le nombre d’UC <br> Modifier la ressource <br> Configurer managedBy <br> Suivi des modifications du disque <br> Bail de disque <br> Afficher les paramètres de connexion <br> Étendre l’unité virtuelle <br> Héberger l’appareil USB <br> Mémoire <br> Modification des paramètres de l’appareil <br> Interroger la compatibilité de la tolérance de pannes <br> Interroger les fichiers sans propriétaire <br> Appareil brut <br> Recharger à partir du chemin d’accès <br> Supprimer le disque <br> Renommer <br> Réinitialiser les informations de l’invité <br> Définir l’annotation <br> Paramètres <br> Positionnement du fichier d’échange <br> Basculer le parent fourche <br> Déverrouiller la machine virtuelle <br> Mettre à niveau la compatibilité de la machine virtuelle |
| **Machine virtuelle > Opérations invité** | Modification des alias d’opération invité <br> Requête d'alias d'opération invité <br> Modification d’opération invité <br> Exécution du programme d’opération invité <br> Requêtes d’opération invité |
| **Machine virtuelle > Interaction** | Répondre à la question <br> Opération de sauvegarde de machines virtuelles <br> Configurer un média CD <br> Configurer une disquette <br> Interaction de la console <br> Créer une capture d’écran <br> Défragmenter tous les disques <br> Connexion de l’appareil <br> Glisser-déposer <br> Gestion du système d’exploitation invité par API VIX <br> Injecter les codes d’analyse USB HID <br> Suspendre ou annuler la suspension <br> Effectuer des opérations de réinitialisation ou de réduction <br> Mise hors tension <br> Mise sous tension <br> Enregistrer la session sur la machine virtuelle <br> Relire la session sur la machine virtuelle <br> Réinitialiser <br> Reprendre la tolérance de panne <br> Interrompre <br> Suspendre la tolérance de panne <br> Test de basculement <br> Redémarrage du test de la machine virtuelle secondaire <br> Désactiver la tolérance de panne <br> Activer la tolérance de panne <br> Installation des outils VMware
| **Machine virtuelle > Inventaire** | Créer à partir de ce qui existe <br> Création <br> Déplacer <br> Inscrire <br> Supprimer <br> Unregister |
| **Machine virtuelle > Approvisionnement** | Autoriser l’accès au disque <br> Autoriser l’accès au fichier <br> Autoriser l’accès au disque en lecture seule <br> Autoriser le téléchargement de machines virtuelles <br> Autoriser le téléchargement de machines virtuelles <br> Cloner le modèle <br> Cloner la machine virtuelle <br> Créer un modèle à partir de la machine virtuelle <br> Personnaliser <br> Déployer un modèle <br> Marquer comme modèle <br> Marquer come machine virtuelle <br> Modifier la spécification de personnalisation <br> Promouvoir les disques  <br> Lire les spécifications de personnalisation |
| **Machine virtuelle -> Configuration de service** | Autoriser les notifications <br> Autoriser l’interrogation des notifications d’événements globales <br> Gérer les configurations de service <br> Modifier la configuration du service <br> Interroger les configurations de service <br> Lire la configuration du service
| **Machine virtuelle > Gestion des instantanés** | Créer l’instantané <br> Supprimer l’instantané <br> Renommer l'instantané <br> Restaurer l’instantané |
| **Machine virtuelle > Replication vSphere** | Configurer la réplication <br> Gérer la réplication <br> Superviser la réplication |
| **vService** | Créer une dépendance <br> Détruire la dépendance <br> Reconfigurer la configuration de la dépendance <br> Mettre à jour la dépendance |

### <a name="cloud-storage-admin-role"></a>Cloud-Storage-Admin-Role

| **Catégorie** | **Privilège** |
|----------|-----------|
| **Magasins de données** | Allouer de l’espace <br> Parcourir les magasins de données <br> Configurer le magasin de données <br> Opérations de fichiers de bas niveau <br> Supprimer le magasin de données <br> Renommer le magasin de données <br> Mettre à jour les fichiers de machine virtuelle <br> Mettre à jour les métadonnées de machine virtuelle |
| **Hôte > Configuration** | Configuration de partition de stockage |
| **Cluster du magasin de données** | Configurer un cluster de magasin de données |
| **Stockage basé sur les profils** | Mise à jour de stockage basé sur profil <br> Affichage de stockage basé sur profil |
| **Affichage de stockage** | Configurer le service <br> Affichage |

### <a name="cloud-network-admin-role"></a>Cloud-Network-Admin-Role

| **Catégorie** | **Privilège** |
|----------|-----------|
| **groupe dvPort** | Créer <br> DELETE <br> Modifier <br> Opération de stratégie <br> Étendue de l’opération |
| **Réseau** | Attribuer un réseau <br> Configurer <br> Déplacer le réseau <br> Supprimer |
| **Machine virtuelle -> Configuration** | Modification des paramètres de l’appareil |

### <a name="cloud-vm-admin-role"></a>Cloud-VM-Admin-Role

| **Catégorie** | **Privilège** |
|----------|-----------|
| **Magasins de données** | Allouer de l’espace <br> Parcourir les magasins de données |
| **Réseau** | Attribuer un réseau |
| **Ressource** | Attribuer une machine virtuelle à une liste de ressources partagées <br> Migrer une machine virtuelle hors tension <br> Migrer une machine virtuelle sous tension
| **vApp** | Exporter <br> Importer |
| **Machine virtuelle -> Configuration** | Ajouter un disque existant <br> Ajouter un nouveau disque <br> Ajouter ou supprimer un appareil <br> Avancé <br> Modifier le nombre d’UC <br> Modifier la ressource <br> Configurer managedBy <br> Suivi des modifications du disque <br> Bail de disque <br> Afficher les paramètres de connexion <br> Étendre l’unité virtuelle <br> Héberger l’appareil USB <br> Mémoire <br> Modification des paramètres de l’appareil <br> Interroger la compatibilité de la tolérance de pannes <br> Interroger les fichiers sans propriétaire <br> Appareil brut <br> Recharger à partir du chemin d’accès <br> Supprimer le disque <br> Renommer <br> Réinitialiser les informations de l’invité <br> Définir l’annotation <br> Paramètres <br> Positionnement du fichier d’échange <br> Basculer le parent fourche <br> Déverrouiller la machine virtuelle <br> Mettre à niveau la compatibilité de la machine virtuelle |
| **Machine virtuelle > Opérations invité** | Modification des alias d’opération invité <br> Requête d'alias d'opération invité <br> Modification d’opération invité <br> Exécution du programme d’opération invité <br> Requêtes d’opération invité    |
| **Machine virtuelle >Interaction** | Répondre à la question <br> Opération de sauvegarde de machines virtuelles <br> Configurer un média CD <br> Configurer une disquette <br> Interaction de la console <br> Créer une capture d’écran <br> Défragmenter tous les disques <br> Connexion de l’appareil <br> Glisser-déposer <br> Gestion du système d’exploitation invité par API VIX <br> Injecter les codes d’analyse USB HID <br> Suspendre ou annuler la suspension <br> Effectuer des opérations de réinitialisation ou de réduction <br> Mise hors tension <br> Mise sous tension <br> Enregistrer la session sur la machine virtuelle <br> Relire la session sur la machine virtuelle <br> Réinitialiser <br> Reprendre la tolérance de panne <br> Interrompre <br> Suspendre la tolérance de panne <br> Test de basculement <br> Redémarrage du test de la machine virtuelle secondaire <br> Désactiver la tolérance de panne <br> Activer la tolérance de panne <br> Installation des outils VMware |
| **Machine virtuelle >Inventaire** | Créer à partir de ce qui existe <br> Création <br> Déplacer <br> Inscrire <br> Supprimer <br> Unregister |
| **Machine virtuelle > Approvisionnement** | Autoriser l’accès au disque <br> Autoriser l’accès au fichier <br> Autoriser l’accès au disque en lecture seule <br> Autoriser le téléchargement de machines virtuelles <br> Autoriser le téléchargement de machines virtuelles <br> Cloner le modèle <br> Cloner la machine virtuelle <br> Créer un modèle à partir de la machine virtuelle <br> Personnaliser <br> Déployer un modèle <br> Marquer comme modèle <br> Marquer come machine virtuelle <br> Modifier la spécification de personnalisation <br> Promouvoir les disques <br> Lire les spécifications de personnalisation |
| **Machine virtuelle > Configuration de service** | Autoriser les notifications <br> Autoriser l’interrogation des notifications d’événements globales <br> Gérer les configurations de service <br> Modifier la configuration du service <br> Interroger les configurations de service <br> Lire la configuration du service
| **Machine virtuelle > Gestion des instantanés** | Créer l’instantané <br> Supprimer l’instantané <br> Renommer l'instantané <br> Restaurer l’instantané |
| **Machine virtuelle > Replication vSphere** | Configurer la réplication <br> Gérer la réplication <br> Superviser la réplication |
| **vService** | Créer une dépendance <br> Détruire la dépendance <br> Reconfigurer la configuration de la dépendance <br> Mettre à jour la dépendance |
