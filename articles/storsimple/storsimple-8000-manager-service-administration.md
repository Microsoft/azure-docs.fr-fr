---
title: Administration du service StorSimple Device Manager | Microsoft Docs
description: Découvrez comment gérer votre appareil StorSimple à l’aide du service StorSimple Device Manager dans le portail Azure.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/12/2017
ms.author: alkohli
ms.openlocfilehash: 69a36f38cc76693f596b45d4df6e232bc01584d5
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967973"
---
# <a name="use-the-storsimple-device-manager-service-to-administer-your-storsimple-device"></a>Utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple

## <a name="overview"></a>Vue d’ensemble

Cet article décrit l’interface du service StorSimple Device Manager, y compris la procédure pour s’y connecter et les options disponibles, et offre des liens vers les flux de travail spécifiques exécutables via cette interface utilisateur. Ce guide s’applique à la fois aux appareils physiques StorSimple et à StorSimple Cloud Appliance.

Cet article portera sur les éléments suivants :

* Connexion au service StorSimple Device Manager
* Gestion de votre appareil StorSimple via le service StorSimple Device Manager

## <a name="connect-to-storsimple-device-manager-service"></a>Connexion au service StorSimple Device Manager

Le service StorSimple Device Manager s’exécute dans Microsoft Azure et se connecte à plusieurs appareils StorSimple. Vous devez utiliser un portail Microsoft Azure central s’exécutant dans un navigateur pour gérer ces appareils. Pour vous connecter au service StorSimple Device Manager, procédez comme suit.

#### <a name="to-connect-to-the-service"></a>Connexion au service :
1. Accédez à [https://portal.azure.com/](https://portal.azure.com/).
2. À l’aide de votre compte Microsoft, connectez-vous au portail Microsoft Azure (situé dans l’angle supérieur droit du volet).
3. Faites défiler le volet de navigation de gauche jusqu’au service StorSimple Device Manager.


## <a name="administer-storsimple-device-using-storsimple-device-manager-service"></a>Gestion d’un appareil StorSimple via le service StorSimple Device Manager

Le tableau suivant récapitule toutes les tâches de gestion courantes et les flux de travail complexes pouvant être effectués dans l’interface utilisateur du service StorSimple Device Manager. Ces tâches sont ordonnées en fonction des panneaux de l’interface utilisateur d’où elles sont lancées.

Pour plus d'informations sur chaque flux de travail, cliquez sur la procédure appropriée dans le tableau.

#### <a name="storsimple-device-manager-workflows"></a>Flux de travail de StorSimple Device Manager

| Pour ce faire... | Suivez cette procédure. |
| --- | --- |
| Créer un service</br>Supprimer un service</br>Obtenir la clé d’inscription au service</br>Régénération d’une clé d’inscription de service |[Déployer un service StorSimple Device Manager](storsimple-8000-manage-service.md) |
| Afficher les journaux d’activité |[Utiliser le panneau de synthèse du service StorSimple Device Manager](storsimple-8000-service-dashboard.md) |
| Modifier la clé de chiffrement des données de service</br>Affichage des journaux d’activité des opérations |[Utilisation du tableau de bord du service StorSimple Device Manager](storsimple-8000-service-dashboard.md) |
| Désactiver un appareil</br>Supprimer une unité |[Désactivation ou suppression d’un appareil](storsimple-8000-deactivate-and-delete-device.md) |
| En savoir plus sur le basculement entre appareils et la récupération d’urgence</br>Basculement vers un appareil physique</br>Basculement vers un appareil virtuel</br>Continuité d’activité et récupération d’urgence (Business Continuity Disaster Recovery - BCDR) |[Basculement et récupération d’urgence pour votre appareil StorSimple](storsimple-8000-device-failover-disaster-recovery.md) |
| Liste des sauvegardes d’un volume</br>Sélectionner un jeu de sauvegarde</br>Supprimer un jeu de sauvegarde |[Gestion des sauvegardes](storsimple-8000-manage-backup-catalog.md) |
| Clonage d’un volume |[Clonage d’un volume](storsimple-8000-clone-volume-u2.md) |
| Restauration d'un jeu de sauvegarde |[Restauration d'un jeu de sauvegarde](storsimple-8000-restore-from-backup-set-u2.md) |
| À propos des comptes de stockage</br>Ajout d’un compte de stockage</br>Modification d’un compte de stockage</br>Suppression d'un compte de stockage</br>Rotation des clés de comptes de stockage |[Gestion des comptes de stockage](storsimple-8000-manage-storage-accounts.md) |
| À propos des modèles de bande passante</br>Ajouter un modèle de bande passante</br>Modifier un modèle de bande passante</br>Supprimer un modèle de bande passante</br>Utiliser un modèle de bande passante par défaut</br>Créer un modèle de bande passante sur une journée entière qui commence à une heure spécifiée |[Modèles de bande passante](storsimple-8000-manage-bandwidth-templates.md) |
| À propos des enregistrements de contrôle d’accès</br>Création d'un enregistrement de contrôle d’accès</br>Modifier un enregistrement de contrôle d’accès</br>Supprimer un enregistrement de contrôle d’accès |[Gestion d’enregistrements de contrôle d’accès](storsimple-8000-manage-acrs.md) |
| Affichage des détails d’une tâche</br>Annulation d’une tâche |[Gestion des travaux](storsimple-8000-manage-jobs-u2.md) |
| Réception de notifications d’alerte</br>Gérer les alertes</br>Consulter les alertes |[Affichage et gestion des alertes StorSimple](storsimple-8000-manage-alerts.md) |
| Création de graphiques d’analyse |[Surveillance de votre appareil StorSimple](./storsimple-8000-monitor-device.md) |
| Ajouter un conteneur de volumes</br>Modifier un conteneur de volumes</br>Supprimer un conteneur de volumes |[Gestion de conteneurs de volume](storsimple-8000-manage-volume-containers.md) |
| Ajout d’un volume</br>Modification d’un volume</br>Mise hors connexion d’un volume</br>Suppression d’un volume</br>Analyse d’un volume |[Gérer les volumes](storsimple-8000-manage-volumes-u2.md) |
| Modification des paramètres de l’appareil</br>Modification des paramètres d’heure</br>Modification des paramètres DNS.md</br>Configuration d’interfaces réseau |[Modification de la configuration de votre appareil StorSimple](storsimple-8000-modify-device-config.md) |
| Afficher les paramètres de proxy web |[Configuration du proxy web de votre appareil](storsimple-8000-configure-web-proxy.md) |
| Modification du mot de passe administrateur de votre appareil</br>Modification du mot de passe de StorSimple Snapshot Manager |[Modification des mots de passe StorSimple](storsimple-8000-change-passwords.md) |
| Configuration de la gestion à distance |[Connexion à distance à votre appareil StorSimple](storsimple-8000-remote-connect.md) |
| Configuration des paramètres d'alerte |[Affichage et gestion des alertes StorSimple](storsimple-8000-manage-alerts.md) |
| Configuration de CHAP pour votre appareil StorSimple |[Configuration de CHAP pour votre appareil StorSimple](./storsimple-8000-configure-chap.md) |
| Ajout d’une stratégie de sauvegarde</br>Ajouter ou modifier une planification</br>Supprimer une stratégie de sauvegarde</br>Exécuter une sauvegarde manuelle</br>Créer une stratégie de sauvegarde personnalisée comportant plusieurs volumes et planifications |[Gestion des stratégies de sauvegarde](storsimple-8000-manage-backup-policies-u2.md) |
| Arrêt des contrôleurs d’appareil</br>Redémarrage des contrôleurs de l’appareil</br>Arrêt des contrôleurs de l'appareil</br>Réinitialisation de votre appareil aux valeurs par défaut</br>(Les données ci-dessus s’appliquent aux appareils en local uniquement) |[Gestion du contrôleur d’appareil StorSimple](storsimple-8000-manage-device-controller.md) |
| En savoir plus sur les composants matériels de StorSimple</br>Surveillance de l'état du matériel</br>(Les données ci-dessus s’appliquent aux appareils en local uniquement) |[Surveillance des composants matériels](storsimple-8000-monitor-hardware-status.md) |
| Création d’un package de prise en charge |[Création et gestion d’un package de prise en charge](storsimple-8000-contact-microsoft-support.md#start-a-support-session-in-windows-powershell-for-storsimple) |
| Installer les mises à jour logicielles |[Mise à jour de votre appareil](storsimple-update-device.md) |

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez des problèmes de fonctionnement quotidien de votre appareil StorSimple ou avec l’un de ses composants matériels, consultez la rubriques suivantes :

* [Résolution des problèmes à l’aide de l’outil de diagnostic](storsimple-8000-diagnostics.md)
* [Utilisation des indicateurs de surveillance de StorSimple](storsimple-monitoring-indicators.md)

Si vous ne parvenez pas à résoudre les problèmes et vous devez générer une demande d’intervention, consultez [Contacter le support technique Microsoft](storsimple-8000-contact-microsoft-support.md).