---
title: Concepts - Contrôle d'accès en fonction du rôle vSphere (vSphere RBAC)
description: Familiarisez-vous avec les principales fonctionnalités du contrôle d'accès en fonction du rôle vSphere pour Azure VMware Solution
ms.topic: conceptual
ms.date: 10/23/2020
ms.openlocfilehash: fbd5b48ce30a6612c45ebd97f08812efd29c50b3
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888400"
---
# <a name="vsphere-role-based-access-control-vsphere-rbac-for-azure-vmware-solution"></a>Contrôle d'accès en fonction du rôle vSphere (vSphere RBAC) pour Azure VMware Solution

Dans Azure VMware Solution, vCenter a un utilisateur local intégré appelé cloudadmin et attribué au rôle CloudAdmin intégré. L’utilisateur local cloudadmin est utilisé pour configurer des utilisateurs dans AD. En général, le rôle CloudAdmin crée et gère les charges de travail dans votre cloud privé. Dans Azure VMware Solution, le rôle CloudAdmin possède des privilèges vCenter qui diffèrent des autres solutions cloud VMware.     

> [!NOTE]
> Actuellement, Azure VMware Solution ne propose pas de rôles personnalisés sur vCenter ou sur le portail Azure VMware Solution. 

Dans un déploiement local vCenter et ESXi, l’administrateur a accès au compte vCenter administrator@vsphere.local. Il peut également se voir attribuer d’autres utilisateurs/groupes Active Directory (AD). 

Dans un déploiement Azure VMware Solution, l’administrateur n’a pas accès au compte d’utilisateur administrateur. Il peut cependant attribuer des utilisateurs et groupes AD au rôle CloudAdmin sur vCenter.  

L’utilisateur de cloud privé n’a pas accès et ne peut pas configurer de composants de gestion spécifiques pris en charge et gérés par Microsoft. Par exemple, les clusters, les hôtes, les magasins de données et les commutateurs virtuels distribués.




## <a name="azure-vmware-solution-cloudadmin-role-on-vcenter"></a>Rôle CloudAdmin d’Azure VMware Solution sur vCenter

Vous pouvez afficher les privilèges accordés au rôle CloudAdmin d’Azure VMware Solution sur votre instance vCenter de cloud privé Azure VMware Solution.

1. Connectez-vous au client SDDC vSphere et accédez à **Menu** > **Administration**.
1. Sous **Access Control**, sélectionnez **Rôles**.
1. Dans la liste des rôles, sélectionnez **CloudAdmin**, puis **Privilèges**. 

   :::image type="content" source="media/role-based-access-control-cloudadmin-privileges.png" alt-text="Comment afficher les privilèges du rôle CloudAdmin dans le client vSphere":::

Le rôle CloudAdmin dans Azure VMware Solution dispose des privilèges suivants sur vCenter. Reportez-vous à la [documentation produit VMware](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-ED56F3C4-77D0-49E3-88B6-B99B8B437B62.html) pour obtenir une explication détaillée de chaque privilège.

| Privilège | Description |
| --------- | ----------- |
| **Alarmes** | Confirmer la réception de l’alerte<br />Créer une alerte<br />Désactiver l’action d’alarme<br />Modifier l’alarme<br />Supprimer l’alarme<br />Définir l’état de l’alarme |
| **autorisations** | Modifier les autorisations |
| **Bibliothèque de contenu** | Ajouter un élément de bibliothèque<br />Créer un abonnement pour une bibliothèque publiée<br />Créer une bibliothèque locale<br />Créer une bibliothèque abonnée<br />Supprimer un élément de bibliothèque<br />Supprimer une bibliothèque locale<br />Supprimer une bibliothèque abonnée<br />Supprimer un abonnement pour une bibliothèque publiée<br />Télécharger les fichiers<br />Expulser des éléments de bibliothèque<br />Expulser une bibliothèque abonnée<br />Importer un stockage<br />Informations d’abonnement de probe<br />Publier un élément de bibliothèque à ses abonnés<br />Publier une bibliothèque à ses abonnés<br />Lire le stockage<br />Synchroniser un élément de bibliothèque<br />Synchroniser une bibliothèque abonnée<br />Introspection de type<br />Mettre à jour les paramètres de configuration du système<br />Mettre à jour les fichiers<br />Mettre à jour la bibliothèque<br />Mettre à jour un élément de bibliothèque<br />Mettre à jour une bibliothèque locale<br />Mettre à jour une bibliothèque abonnée<br />Mettre à jour l’abonnement d’une bibliothèque publiée<br />Afficher les paramètres de configuration |
| **Opérations de chiffrement** | Accès direct |
| **Magasins de données** | Allouer de l’espace<br />Parcourir les magasins de données<br />Configurer le magasin de données<br />Opérations de fichiers de bas niveau<br />Supprimer les fichiers<br />Mettre à jour les métadonnées de machine virtuelle |
| **Folder** | Créer un dossier<br />Supprimer un dossier<br />Déplacer le dossier<br />Renommer le dossier |
| **Global** | Annuler la tâche<br />Balise globale<br />Intégrité<br />Journaliser les événements<br />Gérer des attributs personnalisés<br />Gestionnaires d’appareils<br />Définir un attribut personnalisé<br />Balise système |
| **Hôte** | Réplication vSphere<br />&#160;&#160;&#160;&#160;Gérer la réplication |
| **Balisage vSphere** | Attribuer ou annuler l’attribution d’une balise vSphere<br />Créer une balise vSphere<br />Créer une catégorie de balise vSphere<br />Supprimer une balise vSphere<br />Supprimer une catégorie de balise vSphere<br />Modifier une balise vSphere<br />Modifier une catégorie de balise vSphere<br />Modifier le champ UsedBy pour la catégorie<br />Modifier le champ UsedBy pour la balise |
| **Réseau** | Attribuer un réseau |
| **Ressource** | Appliquer une recommandation<br />Attribuer vApp à une liste de ressources partagées<br />Attribuer une machine virtuelle à une liste de ressources partagées<br />Créer une liste de ressources partagées<br />Migrer une machine virtuelle hors tension<br />Migrer une machine virtuelle sous tension<br />Modifier la liste de ressources partagées<br />Déplacer la liste de ressources partagées<br />Interroger vMotion<br />Supprimer la liste de ressources partagées<br />Renommer la liste de ressources partagées |
| **Tâche planifiée** | Créer une tâche<br />Modifier la tâche<br />Supprimer la tâche<br />Exécuter la tâche |
| **Sessions** | Message<br />Valider la session |
| **Profil** | Vue du stockage basé sur le profil |
| **Vue du stockage** | Affichage |
| **vApp** | Ajouter une machine virtuelle<br />Attribuer une liste de ressources partagées<br />Attribuer vApp<br />Clone<br />Créer<br />DELETE<br />Exporter<br />Importer<br />Déplacer<br />Mise hors tension<br />Mise sous tension<br />Renommer<br />Interrompre<br />Unregister<br />Afficher l’environnement OVF<br />Configuration de l’application vApp<br />Configuration de l'instance vApp<br />Configuration de vApp managedBy<br />Configuration de ressource vApp |
| **Machine virtuelle** | Changer la configuration<br />&#160;&#160;&#160;&#160;Acquérir le bail du disque<br />&#160;&#160;&#160;&#160;Ajouter un disque existant<br />&#160;&#160;&#160;&#160;Ajouter un nouveau disque<br />&#160;&#160;&#160;&#160;Ajouter ou supprimer un appareil<br />&#160;&#160;&#160;&#160;Configuration avancée<br />&#160;&#160;&#160;&#160;Modifier le nombre d’UC<br />&#160;&#160;&#160;&#160;Modifier la mémoire<br />&#160;&#160;&#160;&#160;Modifier les paramètres<br />&#160;&#160;&#160;&#160;Modifier le positionnement du fichier d’échange<br />&#160;&#160;&#160;&#160;Modifier la ressource<br />&#160;&#160;&#160;&#160;Configurer un périphérique USB hôte<br />&#160;&#160;&#160;&#160;Configurer un périphérique brut<br />&#160;&#160;&#160;&#160;Configurer managedBy<br />&#160;&#160;&#160;&#160;Afficher les paramètres de connexion<br />&#160;&#160;&#160;&#160;Étendre le disque virtuel<br />&#160;&#160;&#160;&#160;Modifier les paramètres de l’appareil<br />&#160;&#160;&#160;&#160;Interroger la compatibilité de la tolérance de pannes<br />&#160;&#160;&#160;&#160;Interroger les fichiers sans propriétaire<br />&#160;&#160;&#160;&#160;Recharger à partir de chemins d’accès<br />&#160;&#160;&#160;&#160;Retirer un disque<br />&#160;&#160;&#160;&#160;Renommer<br />&#160;&#160;&#160;&#160;Réinitialiser les informations de l’invité<br />&#160;&#160;&#160;&#160;Définir l’annotation<br />&#160;&#160;&#160;&#160;Basculer le suivi des modifications du disque<br />&#160;&#160;&#160;&#160;Basculer le parent fourche<br />&#160;&#160;&#160;&#160;Mettre à niveau la compatibilité de la machine virtuelle<br />Modifier l’inventaire<br />&#160;&#160;&#160;&#160;Créer à partir de ce qui existe<br />&#160;&#160;&#160;&#160;Créer<br />&#160;&#160;&#160;&#160;Déplacer<br />&#160;&#160;&#160;&#160;Inscrire<br />&#160;&#160;&#160;&#160;Supprimer<br />&#160;&#160;&#160;&#160;Annuler l’inscription<br />Opérations de l’invité<br />&#160;&#160;&#160;&#160;Modification des alias d’opération invité<br />&#160;&#160;&#160;&#160;Requête d’alias d'opération invité<br />&#160;&#160;&#160;&#160;Modification d’opération invité<br />&#160;&#160;&#160;&#160;Exécution du programme d’opération invité<br />&#160;&#160;&#160;&#160;Requêtes d’opération invité<br />Interaction<br />&#160;&#160;&#160;&#160;Répondre à la question<br />&#160;&#160;&#160;&#160;Opération de sauvegarde de machines virtuelles<br />&#160;&#160;&#160;&#160;Configurer un média CD<br />&#160;&#160;&#160;&#160;Configurer une disquette<br />&#160;&#160;&#160;&#160;Connecter des appareils<br />&#160;&#160;&#160;&#160;Interaction de la console<br />&#160;&#160;&#160;&#160;Créer une capture d’écran<br />&#160;&#160;&#160;&#160;Défragmenter tous les disques<br />&#160;&#160;&#160;&#160;Glisser-déposer<br />&#160;&#160;&#160;&#160;Gestion du système d’exploitation invité par API VIX<br />&#160;&#160;&#160;&#160;Injecter les codes d’analyse USB HID<br />&#160;&#160;&#160;&#160;Installer les outils VMware<br />&#160;&#160;&#160;&#160;Suspendre ou annuler la suspension<br />&#160;&#160;&#160;&#160;Effectuer des opérations de réinitialisation ou de réduction<br />&#160;&#160;&#160;&#160;Mise hors tension<br />&#160;&#160;&#160;&#160;Allumer<br />&#160;&#160;&#160;&#160;Enregistrer la session sur la machine virtuelle<br />&#160;&#160;&#160;&#160;Relire la session sur la machine virtuelle<br />&#160;&#160;&#160;&#160;Suspendre<br />&#160;&#160;&#160;&#160;Suspendre la tolérance de panne<br />&#160;&#160;&#160;&#160;Tester le basculement<br />&#160;&#160;&#160;&#160;Redémarrage du test de la machine virtuelle secondaire<br />&#160;&#160;&#160;&#160;Désactiver la tolérance de panne<br />&#160;&#160;&#160;&#160;Activer la tolérance de panne<br />Approvisionnement<br />&#160;&#160;&#160;&#160;Autoriser l’accès au disque<br />&#160;&#160;&#160;&#160;Autoriser l’accès au fichier<br />&#160;&#160;&#160;&#160;Autoriser l’accès au disque en lecture seule<br />&#160;&#160;&#160;&#160;Autoriser le téléchargement de machines virtuelles<br />&#160;&#160;&#160;&#160;Cloner le modèle<br />&#160;&#160;&#160;&#160;Cloner la machine virtuelle<br />&#160;&#160;&#160;&#160;Créer un modèle à partir de la machine virtuelle<br />&#160;&#160;&#160;&#160;Personnaliser l’invité<br />&#160;&#160;&#160;&#160;Déployer un modèle<br />&#160;&#160;&#160;&#160;Marquer comme modèle<br />&#160;&#160;&#160;&#160;Modifier la spécification de personnalisation<br />&#160;&#160;&#160;&#160;Promouvoir les disques<br />&#160;&#160;&#160;&#160;Lire les spécifications de personnalisation<br />Configuration de service<br />&#160;&#160;&#160;&#160;Autoriser les notifications<br />&#160;&#160;&#160;&#160;Autoriser l’interrogation des notifications d’événements globales<br />&#160;&#160;&#160;&#160;Gérer la configuration du service<br />&#160;&#160;&#160;&#160;Modifier la configuration du service<br />&#160;&#160;&#160;&#160;Interroger les configurations de service<br />&#160;&#160;&#160;&#160;Lire la configuration du service<br />Gestion des instantanés<br />&#160;&#160;&#160;&#160;Créer une capture instantanée<br />&#160;&#160;&#160;&#160;Supprimer une capture instantanée<br />&#160;&#160;&#160;&#160;Renommer une capture instantanée<br />&#160;&#160;&#160;&#160;Rétablir une capture instantanée<br />Réplication vSphere<br />&#160;&#160;&#160;&#160;Configurer la réplication<br />&#160;&#160;&#160;&#160;Gérer la réplication<br />&#160;&#160;&#160;&#160;Surveiller la réplication |
| **vService** | Créer une dépendance<br />Détruire la dépendance<br />Reconfigurer la configuration de la dépendance<br />Mettre à jour la dépendance |



## <a name="next-steps"></a>Étapes suivantes

Reportez-vous à la [documentation produit VMware](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-ED56F3C4-77D0-49E3-88B6-B99B8B437B62.html) pour obtenir une explication détaillée de chaque privilège.

<!-- LINKS - internal -->

<!-- LINKS - external-->
[VMware product documentation]: https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-ED56F3C4-77D0-49E3-88B6-B99B8B437B62.html

