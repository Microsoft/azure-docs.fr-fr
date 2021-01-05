---
title: Administration de Microsoft Azure StorSimple Manager Virtual Array | Microsoft Docs
description: Découvrez comment gérer votre StorSimple Virtual Array local à l’aide du service StorSimple Device Manager dans le portail Azure.
services: storsimple
documentationcenter: ''
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 958244a5-f9f5-455e-b7ef-71a65558872e
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/1/2016
ms.author: alkohli
ms.openlocfilehash: 2395e3c6f2026bc1024de38122bc20959fdcf9b7
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967089"
---
# <a name="use-the-storsimple-device-manager-service-to-administer-your-storsimple-virtual-array"></a>Utiliser le service StorSimple Device Manager pour gérer votre StorSimple Virtual Array
![flux du processus d'installation](./media/storsimple-virtual-array-manager-service-administration/manage4.png)

## <a name="overview"></a>Vue d’ensemble
Cet article décrit l’interface du service StorSimple Device Manager, y compris la connexion à ce service et les différentes options disponibles, et fournit des liens vers des flux de travail spécifiques exécutables par le biais de cette interface utilisateur.

À la fin de cet article, vous serez en mesure d’effectuer les opérations suivantes :

* Connexion au service StorSimple Device Manager
* Navigation dans l’interface utilisateur de StorSimple Device Manager
* Gestion de votre StorSimple Virtual Array par le biais du service StorSimple Device Manager

> [!NOTE]
> Pour afficher les options de gestion disponibles pour l’appareil StorSimple série 8000, consultez [Utiliser le service StorSimple Manager pour gérer votre appareil StorSimple](./storsimple-8000-manager-service-administration.md).
> 
> 

## <a name="connect-to-the-storsimple-device-manager-service"></a>Connexion au service StorSimple Device Manager
Le service StorSimple Device Manager s’exécute dans Microsoft Azure et se connecte à plusieurs StorSimple Virtual Arrays. Vous devez utiliser un portail Microsoft Azure central s’exécutant dans un navigateur pour gérer ces appareils. Pour vous connecter au service StorSimple Device Manager, procédez comme suit.

#### <a name="to-connect-to-the-service"></a>Connexion au service :
1. Accédez à [https://ms.portal.azure.com](https://ms.portal.azure.com).
2. À l’aide de votre compte Microsoft, connectez-vous au portail Microsoft Azure (situé dans l’angle supérieur droit du volet).
3. Accédez à Parcourir--> « Filtre » sur les gestionnaires d’appareils StorSimple pour afficher tous les gestionnaires d’appareils d’un abonnement.

## <a name="use-the-storsimple-device-manager-service-to-perform-management-tasks"></a>Utilisation du service StorSimple Device Manager pour effectuer des tâches de gestion
Le tableau suivant récapitule toutes les tâches de gestion courantes et les flux de travail complexes pouvant être effectués dans le panneau de synthèse du service StorSimple Device Manager. L’organisation de ces tâches dépend des panneaux sur lesquels elles sont basées.

Pour plus d'informations sur chaque flux de travail, cliquez sur la procédure appropriée dans le tableau.

#### <a name="storsimple-device-manager-workflows"></a>Flux de travail de StorSimple Device Manager
| Pour ce faire... | Suivez cette procédure |
| --- | --- |
| Créer un service</br>Supprimer un service</br>Obtenir la clé d’inscription du service</br>Régénérer la clé d’inscription du service |[Déployer le service StorSimple Device Manager](storsimple-virtual-array-manage-service.md) |
| Afficher les journaux d’activité |[Utiliser le résumé du service StorSimple](storsimple-virtual-array-service-summary.md) |
| Désactiver un Virtual Array</br>Supprimer un Virtual Array |[Désactiver ou supprimer un Virtual Array](storsimple-virtual-array-deactivate-and-delete-device.md) |
| Récupération d’urgence et basculement d’appareil</br>Conditions préalables de basculement</br>Continuité d’activité et récupération d’urgence (Business Continuity Disaster Recovery - BCDR)</br>Erreurs pendant une récupération d’urgence |[Basculement d'appareil et  récupération d'urgence pour votre StorSimple Virtual Array](storsimple-virtual-array-failover-dr.md) |
| Sauvegarder des partages et des volumes</br>Exécuter une sauvegarde manuelle</br>Modifier la planification de sauvegarde</br>Afficher les sauvegardes existantes |[Sauvegarder votre StorSimple Virtual Array](storsimple-virtual-array-backup.md) |
| Cloner des partages à partir d’un jeu de sauvegarde</br>Cloner des volumes à partir d’un jeu de sauvegarde</br>Récupération au niveau élément (serveur de fichiers uniquement) |[Cloner à partir d’une sauvegarde de votre instance StorSimple Virtual Array](storsimple-virtual-array-clone.md) |
| À propos des comptes de stockage</br>Ajout d’un compte de stockage</br>Modification d’un compte de stockage</br>Suppression d'un compte de stockage |[Gérer les comptes de stockage pour le StorSimple Virtual Array](storsimple-virtual-array-manage-storage-accounts.md) |
| À propos des enregistrements de contrôle d’accès</br>Ajouter ou modifier un enregistrement de contrôle d’accès </br>Supprimer un enregistrement de contrôle d’accès |[Gérer les enregistrements de contrôle d’accès pour le StorSimple Virtual Array](storsimple-virtual-array-manage-acrs.md) |
| Affichage des détails d’une tâche |[Gérer les tâches StorSimple Virtual Array](storsimple-virtual-array-manage-jobs.md) |
| Configuration des paramètres d'alerte</br>Réception de notifications d’alerte</br>Gérer les alertes</br>Consulter les alertes |[Afficher et gérer les alertes pour le StorSimple Virtual Array](storsimple-virtual-array-manage-alerts.md) |
| Modification du mot de passe d’administrateur de l’appareil |[Modifier le mot de passe administrateur de l’appareil StorSimple Virtual Array](storsimple-virtual-array-change-device-admin-password.md) |
| Installer les mises à jour logicielles |[Mettre à jour votre Virtual Array](storsimple-virtual-array-install-update.md) |

> [!NOTE]
> Vous devez utiliser [l’interface utilisateur web locale](storsimple-ova-web-ui-admin.md) pour les tâches suivantes :
> 
> * [Récupération de la clé de chiffrement des données de service](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key)
> * [Création d’un package de prise en charge](storsimple-ova-web-ui-admin.md#generate-a-log-package)
> * [Arrêt et redémarrage d’un Virtual Array](storsimple-ova-web-ui-admin.md#shut-down-and-restart-your-device)
> 
> 

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur l’interface utilisateur web et sur son utilisation, accédez à [Utiliser l’interface utilisateur web StorSimple pour gérer votre StorSimple Virtual Array](storsimple-ova-web-ui-admin.md).