---
title: Déployer le service StorSimple Device Manager dans Azure | Microsoft Docs
description: En savoir plus sur les étapes requises pour la création, la suppression, la migration du service et la gestion de la clé d’inscription au service.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/09/2018
ms.author: alkohli
ms.openlocfilehash: 66244bd8e24ff62be41df72f7a39c0ce0ed13135
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360709"
---
# <a name="deploy-the-storsimple-device-manager-service-for-storsimple-8000-series-devices"></a>Déployer le service StorSimple Device Manager pour les appareils de la gamme StorSimple 8000

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Vue d’ensemble

Le service StorSimple Device Manager s’exécute dans Microsoft Azure et se connecte à plusieurs appareils StorSimple. Après avoir créé le service, vous pouvez l’utiliser pour gérer tous les appareils qui y sont connectés à partir d’un emplacement central unique, ce qui réduit la charge administrative.

Ce didacticiel décrit les étapes requises pour la création, la suppression, la migration du service et la gestion de la clé d’inscription du service. Les informations contenues dans cet article s’appliquent uniquement aux appareils de la gamme StorSimple 8000. Pour plus d’informations sur StorSimple Virtual Arrays, accédez à [Déployer le service StorSimple Device Manager pour StorSimple Virtual Array](storsimple-virtual-array-manage-service.md).

> [!NOTE]
> -  Le portail Azure prend en charge les appareils exécutant Update 5.0 ou version ultérieure. Si votre appareil n’est pas à jour, installez la mise à jour Update 5.0 immédiatement. Pour plus d’informations, accédez à [Installer Update 5](storsimple-8000-install-update-5.md). 
> - Si vous utilisez une appliance StorSimple Cloud Appliance (8010/8020), vous ne pouvez pas mettre à jour une appliance cloud. Utilisez la version la plus récente du logiciel pour créer une appliance cloud avec Update 5.0 et basculez ensuite vers la nouvelle appliance cloud créée. 
> - Tous les appareils exécutant la mise à jour Update 4.0 ou une version antérieure auront des fonctionnalités de gestion réduites. 

## <a name="create-a-service"></a>Créer un service
Pour créer un service StorSimple Device Manager, vous avez besoin des éléments suivants :

* Un abonnement avec un contrat Entreprise
* Un compte de stockage Microsoft Azure actif
* Les informations de facturation utilisées pour la gestion des accès

Seuls les abonnements avec un contrat Entreprise sont autorisés. Vous pouvez également choisir de générer un compte de stockage par défaut lorsque vous créez le service.

Un seul service peut gérer plusieurs appareils. Cependant, un appareil ne peut pas couvrir plusieurs services. Une grande entreprise peut avoir plusieurs instances de service pour utiliser différents abonnements, organisations ou même emplacements de déploiement. 

> [!NOTE]
> Vous devez créer des instances distinctes du service StorSimple Device Manager pour gérer les appareils de la gamme StorSimple 8000 et les baies StorSimple Virtual Array.

Procédez comme suit pour créer un service.

[!INCLUDE [storsimple-create-new-service](../../includes/storsimple-8000-create-new-service.md)]


Pour chaque service StorSimple Device Manager, les attributs suivants existent :

* **Nom** : le nom assigné à votre service StorSimple Device Manager lors de sa création. **Impossible de modifier le nom du service une fois que le service a été créé. Cela vaut également pour les autres entités telles que les appareils, les volumes, les conteneurs de volumes et les stratégies de sauvegarde qui ne peuvent pas être renommés dans le portail Azure.**
* **État** : l’état du service, qui peut être **Actif**, **Création en cours** ou **En ligne**.
* **Emplacement** : l’emplacement géographique sur lequel l’appareil StorSimple sera déployé.
* **Abonnement** : l’abonnement de facturation associé à votre service.

## <a name="delete-a-service"></a>Supprimer un service

Avant de supprimer un service, assurez-vous qu’aucun appareil connecté ne l’utilise. Si le service est en cours d’utilisation, désactivez les appareils connectés. L’opération de désactivation rompt la connexion entre l’appareil et le service, mais conserve les données de l’appareil dans le cloud.

> [!IMPORTANT]
> Après qu’un service a été supprimé, l’opération ne peut pas être annulée. Un appareil qui utilisait le service doit être réinitialisé aux paramètres d’usine par défaut avant de pouvoir être utilisé avec un autre service. Dans ce scénario, les données locales de l’appareil, ainsi que la configuration, sont perdues.

Pour supprimer un service, procédez comme suit.

### <a name="to-delete-a-service"></a>Pour supprimer un service

1. Recherchez le service que vous souhaitez supprimer. Cliquez sur l’icône **Ressources** et entrez les termes appropriés à rechercher. Dans les résultats de la recherche, cliquez sur le service que vous souhaitez supprimer.

    ![Rechercher le service à supprimer](./media/storsimple-8000-manage-service/deletessdevman1.png)

2. Cela vous renvoie vers le panneau du service StorSimple Device Manager. Cliquez sur **Supprimer**.

    ![Suppression du service](./media/storsimple-8000-manage-service/deletessdevman2.png)

3. Cliquez sur **Oui** dans la notification de confirmation. La suppression du service peut nécessiter quelques minutes.

    ![Confirmer la suppression](./media/storsimple-8000-manage-service/deletessdevman3.png)

## <a name="get-the-service-registration-key"></a>Obtenir la clé d’inscription du service

Une fois que vous avez créé un service, vous devez inscrire votre appareil StorSimple auprès du service. Pour inscrire votre premier appareil StorSimple, vous avez besoin de la clé d’inscription du service. Pour inscrire des appareils supplémentaires auprès d’un service StorSimple existant, vous avez besoin de la clé d’inscription et de la clé de chiffrement des données de service (générée sur le premier appareil lors de l’inscription). Pour plus d’informations sur la clé de chiffrement des données du service, consultez la rubrique [Sécurité StorSimple](storsimple-8000-security.md). Vous pouvez obtenir la clé d’inscription en accédant à **Clés** sur le panneau de votre service StorSimple Device Manager.

Procédez comme suit pour obtenir la clé d’inscription du service.

[!INCLUDE [storsimple-8000-get-service-registration-key](../../includes/storsimple-8000-get-service-registration-key.md)]

Conservez la clé d’inscription du service dans un emplacement sûr. Vous aurez besoin de cette clé, ainsi que de la clé de chiffrement des données du service, pour enregistrer des appareils supplémentaires auprès du service. Après avoir obtenu la clé d’inscription du service, vous devez configurer votre appareil via l’interface Windows PowerShell pour StorSimple.

Pour plus d’informations sur la façon d’utiliser cette clé d’inscription, consultez l’[étape 3 : Configuration et inscription de l’appareil via Windows PowerShell pour StorSimple](storsimple-8000-deployment-walkthrough-u2.md#step-3-configure-and-register-the-device-through-windows-powershell-for-storsimple).

## <a name="regenerate-the-service-registration-key"></a>Régénérer la clé d’inscription du service
Vous devez régénérer une clé d’inscription du service si vous êtes amené à effectuer une rotation des clés ou si la liste des administrateurs de services fédérés a changé. Lorsque vous régénérez la clé, la nouvelle clé est utilisée uniquement pour l’enregistrement des appareils suivants. Les appareils déjà enregistrés ne sont pas affectés par ce processus.

Procédez comme suit pour régénérer une clé d’inscription du service.

### <a name="to-regenerate-the-service-registration-key"></a>Pour régénérer la clé d’inscription du service
1. Dans le panneau **StorSimple Device Manager**, accédez à **Gestion &gt;** **Clés**.
    
    ![Accédez au panneau Clés](./media/storsimple-8000-manage-service/regenregkey2.png)

2. Dans le panneau **Clés**, cliquez sur **Régénérer**.

    ![Cliquer sur Régénérer](./media/storsimple-8000-manage-service/regenregkey3.png)
3. Dans le panneau **Régénérer la clé d’inscription du service**, vérifiez l’action requise lors de la régénération des clés. Tous les appareils qui seront inscrits auprès de ce service par la suite utilisent la nouvelle clé d’inscription. Cliquez sur **Régénérer** pour confirmer l’opération. Une fois la régénération terminée, vous en êtes informé.

    ![Confirmer la régénération](./media/storsimple-8000-manage-service/regenregkey4.png)

4. Une nouvelle clé d’inscription du service s’affiche.

5. Copiez cette clé et sauvegardez-la pour enregistrer tout nouvel appareil auprès de ce service.



## <a name="change-the-service-data-encryption-key"></a>Modifier la clé de chiffrement des données de service
Les clés de chiffrement des données de service permettent de chiffrer les données client confidentielles, telles que les identifiants du compte de stockage, qui sont envoyées à partir votre service StorSimple Manager sur le périphérique StorSimple. Vous devrez modifier ces clés périodiquement si votre service informatique applique une politique de rotation des clés sur les périphériques de stockage. Le processus de modification de la clé peut être légèrement différent selon qu'il y a un ou plusieurs périphériques gérés par le service StorSimple Manager. Pour plus d’informations, accédez à [StorSimple security and data protection](storsimple-8000-security.md) (Sécurité et protection des données StorSimple).

La modification de la clé de chiffrement est un processus en 3 étapes :

1. À l’aide des scripts Windows PowerShell pour Azure Resource Manager, autorisez un appareil à modifier la clé de chiffrement des données de service.
2. Utiliser Windows PowerShell pour StorSimple pour démarrer la modification de la clé de chiffrement des données du service.
3. Si vous avez plusieurs périphériques StorSimple, mettez à jour la clé de chiffrement des données sur les autres périphériques.

### <a name="step-1-use-windows-powershell-script-to-authorize-a-device-to-change-the-service-data-encryption-key"></a>Étape 1 : Utiliser un script Windows PowerShell pour autoriser un appareil à modifier la clé de chiffrement des données de service
En règle générale, l’administrateur de l’appareil demande que l’administrateur du service autorise un appareil à modifier les clés de chiffrement des données du service. Ensuite, l’administrateur du service autorise l’appareil à modifier la clé.

Cette étape est effectuée en utilisant le script basé sur Azure Resource Manager. L’administrateur de services fédérés peut sélectionner un appareil pouvant être autorisé. Après cela, l’appareil est autorisé à démarrer le processus de modification de la clé de chiffrement des données du service. 

Pour plus d’informations sur l’utilisation du script, accédez à [Authorize-ServiceEncryptionRollover.ps1](https://github.com/anoobbacker/storsimpledevicemgmttools/blob/master/Authorize-ServiceEncryptionRollover.ps1).

#### <a name="which-devices-can-be-authorized-to-change-service-data-encryption-keys"></a>Quels appareils peuvent être autorisés à modifier les clés de chiffrement des données du service ?
Un appareil doit respecter les critères suivants avant d’être autorisé à démarrer des modifications de clé de chiffrement des données du service :

* L’appareil doit être en ligne pour bénéficier de l’autorisation de modification de clé de chiffrement des données du service.
* Si le changement de clé n’a pas été démarré, vous devez attendre 30 minutes avant d’autoriser à nouveau le même appareil.
* Vous pouvez autoriser un autre appareil, sous réserve que la modification de la clé n’a pas été démarrée par l’appareil précédemment autorisé. Une fois le nouvel appareil autorisé, l’ancien appareil ne peut plus démarrer la modification.
* Vous ne pouvez pas autoriser un appareil alors que la substitution de la clé de chiffrement des données du service est en cours.
* Vous pouvez autoriser un appareil lorsque certains appareils inscrits auprès du service ont substitué le chiffrement tandis que d’autres ne l’ont pas fait. 

### <a name="step-2-use-windows-powershell-for-storsimple-to-initiate-the-service-data-encryption-key-change"></a>Étape 2 : Utiliser Windows PowerShell pour StorSimple pour démarrer la modification de la clé de chiffrement des données du service
Cette étape s’effectue dans l’interface Windows PowerShell pour StorSimple, sur l’appareil StorSimple autorisé.

> [!NOTE]
> Aucune opération ne peut être effectuée dans le portail Azure de votre service StorSimple Manager avant la fin de la substitution de la clé.


Si vous utilisez la console série de l’appareil pour vous connecter à l’interface Windows PowerShell, procédez comme suit.

#### <a name="to-initiate-the-service-data-encryption-key-change"></a>Pour démarrer la modification de la clé de chiffrement des données du service
1. Sélectionnez Option 1 pour vous connecter avec un accès complet.
2. À l’invite de commandes, tapez :
   
     `Invoke-HcsmServiceDataEncryptionKeyChange`
3. Une fois l’applet de commande terminée, vous obtenez une nouvelle clé de chiffrement des données du service. Copiez et enregistrez cette clé pour l’utiliser lors de l’étape 3 de cette procédure. Cette clé permet de mettre à jour tous les appareils restants inscrits auprès du service StorSimple Manager.
   
   > [!NOTE]
   > Ce processus doit être démarré dans les quatre heures suivant l’autorisation d’un appareil StorSimple.
   > 
   > 
   
   Cette nouvelle clé est ensuite envoyée au service pour être transmise à tous les appareils inscrits auprès du service. Une alerte s’affiche alors sur le tableau de bord du service. Le service désactive toutes les opérations sur les appareils inscrits et l’administrateur de l’appareil doit alors mettre à jour la clé de chiffrement des données du service sur les autres appareils. Toutefois, les E/S (hôtes envoyant des données vers le cloud) ne sont pas interrompues.
   
   Si vous avez un seul appareil inscrit auprès de votre service, le processus de substitution est maintenant terminé et vous pouvez ignorer l’étape suivante. Si vous avez plusieurs appareils inscrits auprès de votre service, passez à l’étape 3.

### <a name="step-3-update-the-service-data-encryption-key-on-other-storsimple-devices"></a>Étape 3 : Mettre à jour la clé de chiffrement sur les autres appareils StorSimple
Si vous avez plusieurs appareils inscrits auprès du service StorSimple Manager, cette procédure s’effectue dans l’interface Windows PowerShell de votre appareil StorSimple. La clé que vous avez obtenue à l’étape 2 doit être utilisée pour mettre à jour tous les appareils StorSimple restant inscrits auprès du service StorSimple Manager.

Procédez comme suit pour mettre à jour le chiffrement des données du service sur votre appareil.

#### <a name="to-update-the-service-data-encryption-key-on-physical-devices"></a>Pour mettre à jour la clé de chiffrement des données de service sur des appareils physiques
1. Utilisez Windows PowerShell pour StorSimple pour vous connecter à la console. Sélectionnez Option 1 pour vous connecter avec un accès complet.
2. À l’invite de commandes, tapez : `Invoke-HcsmServiceDataEncryptionKeyChange – ServiceDataEncryptionKey`
3. Fournir la clé de chiffrement de données de service que vous avez obtenue à l’[étape 2 : Utiliser Windows PowerShell pour StorSimple pour démarrer la modification de la clé de chiffrement des données du service](#to-initiate-the-service-data-encryption-key-change).

#### <a name="to-update-the-service-data-encryption-key-on-all-the-80108020-cloud-appliances"></a>Pour mettre à jour la clé de chiffrement des données de service sur toutes les appliances cloud 8010/8020
1. Téléchargez et installez le script PowerShell [Update-CloudApplianceServiceEncryptionKey.ps1](https://github.com/anoobbacker/storsimpledevicemgmttools/blob/master/Update-CloudApplianceServiceEncryptionKey.ps1). 
2. Ouvrez PowerShell et, à l’invite de commandes, tapez : `Update-CloudApplianceServiceEncryptionKey.ps1 -SubscriptionId [subscription] -TenantId [tenantid] -ResourceGroupName [resource group] -ManagerName [device manager]`

Ce script permet de garantir que la clé de chiffrement des données de service est définie sur toutes les appliances cloud 8010/8020 sous Device Manager.

## <a name="supported-operations-on-devices-running-versions-prior-to-update-50"></a>Opérations prises en charge sur les appareils exécutant des versions antérieures à Update 5.0
Seuls les appareils StorSimple exécutant Update 5.0 et des versions ultérieures sont pris en charge dans le portail Azure. La prise en charge des appareils exécutant des versions antérieures est limitée. Une fois que vous avez effectué la migration vers le portail Azure, utilisez le tableau suivant pour connaître les opérations prises en charge sur les appareils exécutant des versions antérieures à Update 5.0.

| Opération                                                                                                                       | Prise en charge      |
|---------------------------------------------------------------------------------------------------------------------------------|----------------|
| Inscrire un appareil                                                                                                               | Oui            |
| Configurer les paramètres d’un appareil, tels que les paramètres généraux, réseau et de sécurité                                                                | Oui            |
| Analyser, télécharger et installer des mises à jour                                                                                             | Oui            |
| Désactiver un appareil                                                                                                               | Oui            |
| Supprimer un appareil                                                                                                                   | Oui            |
| Créer, modifier et supprimer un conteneur de volumes                                                                                   | Non             |
| Créer, modifier et supprimer un volume                                                                                             | Non             |
| Créer, modifier et supprimer une stratégie de sauvegarde                                                                                      | Non             |
| Exécuter une sauvegarde manuelle                                                                                                            | Non             |
| Exécuter une sauvegarde planifiée                                                                                                         | Non applicable |
| Restaurer à partir d’un jeu de sauvegarde                                                                                                        | Non             |
| Cloner vers un appareil exécutant Update 3.0 et versions ultérieures <br> L’appareil source exécute une version antérieure à Update 3.0.                                | Oui            |
| Cloner vers un appareil exécutant des versions antérieures à Update 3.0                                                                          | Non             |
| Basculer en tant qu’appareil source <br> (à partir d’un appareil exécutant une version antérieure à Update 3.0 vers un appareil exécutant Update 3.0 et versions ultérieures)                                                               | Oui            |
| Basculer en tant qu’appareil cible <br> (vers un appareil exécutant une version logicielle antérieure à Update 3.0)                                                                                   | Non             |
| Supprimer une alerte                                                                                                                  | Oui            |
| Afficher les stratégies de sauvegarde, le catalogue de sauvegarde, les volumes, les conteneurs de volumes, les graphiques de surveillance, les travaux et les alertes créés dans le portail classique | Oui            |
| Activer et désactiver des contrôleurs d’appareils                                                                                              | Oui            |


## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur le [processus de déploiement StorSimple](storsimple-8000-deployment-walkthrough-u2.md).
* En savoir plus sur la [gestion de votre compte de stockage StorSimple](storsimple-8000-manage-storage-accounts.md).
* En savoir plus sur [l’utilisation du service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).
