---
title: Problèmes connus/limitations de migration avec le mode hybride
description: Découvrez les problèmes connus/limitations de migration d’Azure Database Migration Service en mode hybride.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: mvc
ms.topic: troubleshooting
ms.date: 02/20/2020
ms.openlocfilehash: 819fe5ced6c91819c817065305a31fca456ea5c0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91291841"
---
# <a name="known-issuesmigration-limitations-with-using-hybrid-mode"></a>Problèmes connus/limitations de migration avec le mode hybride

Les sections suivantes décrivent les problèmes connus et limitations associés à Azure Database Migration Service en mode hybride.

## <a name="installer-fails-to-authenticate"></a>Échec de l’authentification du programme d’installation

Après le chargement du certificat sur votre AdApp, il existe un délai de quelques minutes avant qu’il puisse s’authentifier auprès d’Azure. Le programme d’installation effectuera une nouvelle tentative après un certain délai, mais il est possible que le délai de propagation soit plus long que la nouvelle tentative, auquel cas vous verrez un message **FailedToGetAccessTokenException**. Si le certificat a été chargé sur le bon AdApp et que l’AppId fourni dans dmsSettings.json est correct, essayez d’exécuter à nouveau la commande d’installation.

## <a name="service-offline-after-successful-installation"></a>Service « hors connexion » après une installation réussie

Si le service s’affiche comme étant hors connexion une fois le processus d’installation terminé, essayez de suivre les étapes ci-dessous.

1. Sur le Portail Azure, dans votre instance Azure Database Migration Service, accédez à l’onglet de paramètres **Hybride**, puis vérifiez que le Worker est inscrit en consultant la grille des Workers inscrits.

    L’état de ce Worker doit être **En ligne**, mais il peut apparaître **Hors connexion** en cas de problème.

2. Sur l’ordinateur Worker, vérifiez l’état du service en exécutant la commande PowerShell suivante :

    ```
    Get-Service Scenario*
    ```

    Cette commande donne l’état du service Windows qui exécute le Worker. Il ne devrait y avoir qu’un seul résultat. Si le Worker est arrêté, vous pouvez essayer de le redémarrer avec la commande PowerShell suivante :

    ```
    Start-Service Scenario*
    ```

    Vous pouvez également vérifier le service dans l’interface utilisateur des services Windows.

3. Si le service Windows alterne entre En cours d’exécution et Arrêté, c’est le signe que le Worker a rencontré des problèmes de démarrage. Vérifiez les journaux du Worker hybride Azure Database Migration Service pour identifier le problème.

    - Les journaux du processus d’installation sont stockés dans le dossier « logs » du dossier à partir duquel l’exécutable du programme d’installation a été exécuté.
    - Les journaux du Worker hybride Azure Database Migration Service sont stockés dans le dossier **WorkerLogs** du dossier dans lequel le Worker est installé. L’emplacement par défaut des fichiers journaux du Worker hybride est **C:\Program Files\DatabaseMigrationServiceHybrid\WorkerLogs**.

## <a name="using-your-own-signed-certificate"></a>Utiliser son propre certificat signé

Le certificat généré par l’action GenerateCert est un certificat auto-signé, qui n’est pas forcément acceptable en fonction de vos stratégies de sécurité internes. Au lieu de l’utiliser, vous pouvez fournir votre propre certificat et indiquer l’empreinte numérique dans dmsSettings.json. Ce certificat doit être chargé sur votre AdApp et installé sur l’ordinateur d’installation du Worker hybride Azure Database Migration Service. Ensuite, installez ce certificat avec la clé privée dans le magasin de certificats de l’ordinateur local.

## <a name="running-the-worker-service-as-a-low-privilege-account"></a>Exécuter le service Worker en tant que compte à faibles privilèges

Par défaut, le service du Worker hybride Azure Database Migration Service s’exécute en tant que compte système local. Vous pouvez modifier le compte utilisé pour ce service tant que le compte que vous utilisez dispose d’autorisations réseau. Pour modifier le compte d’identification du service, suivez le processus ci-dessous.

1. Arrêtez le service, soit par le biais des services Windows, soit à l’aide de la commande Stop-Service dans PowerShell.

2. Mettez à jour le service de façon à utiliser un autre compte d’ouverture de session.

3. Dans certmgr pour les certificats d’ordinateur local, accordez des autorisations de clé privée au nouveau compte pour les certificats **DMS Hybrid App Key** and **DMS Scenario Engine Key Pair**.

    a. Ouvrez certmgr pour voir les clés suivantes :

    - DMS Hybrid App Key
    - DMS Hybrid Worker Setup Key
    - DMS Scenario Engine Key Pair

    b. Cliquez avec le bouton droit sur l’entrée **DMS Hybrid App Key**, pointez sur **Toutes les tâches**, puis sélectionnez **Gérer les clés privées**.

    c. Dans l’onglet **Sécurité**, sélectionnez **Ajouter**, puis entrez le nom du compte.

    d. Suivez les mêmes étapes pour accorder l’autorisation de clé privée pour le nouveau compte au certificat **DMS Scenario Engine Key Pair**.

## <a name="unregistering-the-worker-manually"></a>Annuler manuellement l’inscription du Worker

Si vous n’avez plus accès à l’ordinateur Worker, vous pouvez annuler l’inscription du Worker et réutiliser votre instance Azure Database Migration Service en suivant les étapes ci-dessous :

1. Sur le Portail Azure, accédez à votre instance Azure Database Migration Service, puis à la page de paramètres **Hybride**.

   Votre entrée Worker apparaît dans la liste, avec l’état **Hors connexion**.

2. Tout à droite de la liste de l’entrée Worker, sélectionnez les points de suspension, puis **Annuler l’inscription**.

## <a name="addressing-issues-for-specific-migration-scenarios"></a>Résoudre les problèmes dans des scénarios de migration spécifiques

Les sections ci-dessous décrivent des problèmes propres à certains scénarios d’utilisation du mode hybride d’Azure Database Migration Service pour effectuer une migration en ligne.

### <a name="online-migrations-to-azure-sql-managed-instance"></a>Migrations en ligne vers Azure SQL Managed Instance

**Utilisation élevée du processeur**

**Problème** : pour les migrations en ligne vers SQL Managed Instance, l’ordinateur exécutant le Worker hybride affiche une forte utilisation du processeur s’il y a trop de sauvegardes ou si elles sont trop volumineuses.

**Atténuation** : pour atténuer ce problème, utilisez des sauvegardes compressées, fractionnez la migration pour qu’elle utilise plusieurs partages ou effectuez un scale-up de l’ordinateur exécutant le Worker hybride.
