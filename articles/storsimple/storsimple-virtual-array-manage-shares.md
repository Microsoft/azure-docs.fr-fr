---
title: Gérer des partages StorSimple Virtual Array | Microsoft Docs
description: Décrit le service StorSimple Device Manager et vous explique comment l’utiliser pour gérer des partages sur votre instance StorSimple Virtual Array.
services: storsimple
documentationcenter: ''
author: manuaery
manager: syadav
editor: ''
ms.assetid: 0a799c83-fde5-4f3f-af0e-67535d1882b6
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2016
ms.author: manuaery
ms.openlocfilehash: 340b910319cabe3379bdb1bad1c09bc71c17f072
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994942"
---
# <a name="use-the-storsimple-device-manager-service-to-manage-shares-on-the-storsimple-virtual-array"></a>Utiliser le service StorSimple Device Manager pour gérer des partages sur l’instance StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel explique comment utiliser le service StorSimple Device Manager pour créer et gérer des partages sur votre instance StorSimple Virtual Array.

Le service StorSimple Device Manager est une extension dans le portail Azure qui vous permet de gérer votre solution StorSimple à partir d’une seule interface web. Outre la gestion des partages et des volumes, le service StorSimple Device Manager prend en charge l’affichage et la gestion des appareils, l’affichage des alertes, la gestion des stratégies de sauvegardes et la gestion du catalogue de sauvegarde.

## <a name="share-types"></a>Types de partages

Les partages StorSimple peuvent être les suivants :

* **Attaché localement** : les données de ces partages demeurent à tout moment sur le tableau ; elles ne débordent pas sur le cloud.
* **Hiérarchisé** : les données de ces partages peuvent déborder sur le cloud. Lorsque vous créez un partage à plusieurs niveaux, environ 10 % de l’espace est configuré au niveau local et 90 % dans le cloud. Par exemple, si vous avez configuré un partage de 1 To, 100 Go résident dans l’espace local et 900 Go sont utilisés dans le cloud lorsque les données sont stockées en niveaux. Cela implique que si vous n’avez plus d’espace local sur l’appareil, vous ne pouvez pas configurer un partage à plusieurs niveaux (car les 10 % requis sur l’espace local ne seront pas disponibles).

### <a name="provisioned-capacity"></a>Capacité allouée

Reportez-vous au tableau suivant pour connaître la capacité maximale allouée pour chaque type de partage.

| **Identificateur de la limite** | **Limite** |
| --- | --- |
| Taille minimale d’un partage hiérarchisé |500 Go |
| Taille maximale d’un partage hiérarchisé |20 To |
| Taille minimale d'un partage épinglé localement |50 Go |
| Taille maximale d'un partage épinglé localement |2 To |

## <a name="the-shares-blade"></a>Le panneau Partages

Sur le panneau de synthèse du service StorSimple, le menu **Partages** affiche la liste des partages de stockage sur un tableau StorSimple considéré et vous donne les moyens de les gérer.

![Panneau Partages](./media/storsimple-virtual-array-manage-shares/shares-blade.png)

Un partage est constitué d’une série d’attributs :

* **Nom du partage** : nom descriptif qui doit être unique et vous aide à identifier le partage.
* **État** : peut être en ligne ou hors connexion. Si un partage est hors connexion, ses utilisateurs ne seront pas en mesure d’y accéder.
* **Type** : indique si le partage est **Hiérarchisé** (valeur par défaut) ou **Attaché localement**.
* **Capacité** : spécifie la quantité de données utilisées par rapport au volume total des données pouvant être stockées sur le partage.
* **Description** : un paramètre facultatif décrivant le partage.
* **Autorisations** : les autorisations NTFS d’accès au partage pouvant être gérées via l’Explorateur Windows.
* **Sauvegarde** : dans le cas d’une instance StorSimple Virtual Array, l’ensemble des partages sont automatiquement activés pour la sauvegarde.

![Détails des partages](./media/storsimple-virtual-array-manage-shares/share-details.png)

Suivez les instructions de ce didacticiel pour effectuer les tâches suivantes :

* Ajouter un partage
* Modifier un partage
* Mettre un partage hors connexion
* Supprimer un partage

## <a name="add-a-share"></a>Ajouter un partage

1. À partir du panneau de synthèse du service StorSimple, cliquez sur **+ Ajouter un partage** dans la barre de commandes. Le panneau **Ajouter un partage** s’ouvre.

    ![Ajouter un partage](./media/storsimple-virtual-array-manage-shares/add-share.png)

2. Dans le panneau **Ajouter un partage**, procédez comme suit :
   
   1. Dans le champ **Nom du partage**, entrez un nom unique pour votre partage. Le nom doit être une chaîne contenant entre 3 et 127 caractères.

   2. Une **Description** facultative pour le partage. La description permet d'identifier les propriétaires du partage.

   3. Dans la liste déroulante **Type**, spécifiez si vous souhaitez créer un partage **Hiérarchisé** ou **Attaché localement**. Pour les charges de travail qui nécessitent des garanties locales, une faible latence et les meilleures performances possibles, sélectionnez un **Partage attaché localement**. Pour toutes les autres données, sélectionnez un partage **Hiérarchisé**.

   4. Dans le champ **Capacité**, spécifiez la taille du partage. Un partage hiérarchisé doit être compris entre 500 Go et 20 To, tandis qu’un partage attaché doit être compris entre 50 Go et 2 To.

   5. Dans le champ **Affecter une autorisation d’accès total par défaut**, attribuez les autorisations à l’utilisateur ou au groupe qui doivent accéder à ce partage. Spécifiez le nom de l’utilisateur ou du groupe d’utilisateurs au format _john@contoso.com_ . Nous vous recommandons d'utiliser un groupe d'utilisateurs (plutôt qu'un seul utilisateur) lorsque vous accordez des privilèges d'administrateur pour accéder à ces partages. Après avoir attribué les autorisations à cet emplacement, vous pouvez utiliser l’Explorateur de fichiers par la suite pour modifier ces autorisations.
3. Lorsque vous avez terminé de configurer votre partage, cliquez sur **Créer**. Un partage est créé avec les paramètres spécifiés ; une notification s’affiche. Par défaut, la sauvegarde est activée pour le partage.
4. Pour vérifier que le partage a été créé, accédez au panneau **Partages**. Le partage devrait y apparaître.
   
    ![Le partage stimule la réussite.](./media/storsimple-virtual-array-manage-shares/share-success.png)

## <a name="modify-a-share"></a>Modifier un partage

Modifiez un partage lorsque vous devez modifier sa description. Une fois le partage créé, aucune autre propriété de partage ne peut être modifiée.

#### <a name="to-modify-a-share"></a>Pour modifier un partage

1. À partir du paramètre **Partages** du panneau de synthèse du service StorSimple, sélectionnez le tableau virtuel sur lequel est hébergé le partage à modifier.
2. **Sélectionnez** le partage pour afficher la description actuelle et le modifier.
3. Pour enregistrer les modifications, cliquez sur la barre de commandes **Enregistrer**. Vos paramètres spécifiés sont appliqués ; une notification s’affiche.
   
    ![ Modifier un partage](./media/storsimple-virtual-array-manage-shares/share-edit.png)

## <a name="take-a-share-offline"></a>Mettre un partage hors connexion

Si vous envisagez de modifier ou de supprimer un partage, il vous faudra éventuellement le déplacer hors connexion. Lorsqu’un partage est hors connexion, il n’est pas disponible pour l’accès en lecture-écriture. Ce déplacement hors connexion s’effectue sur l’hôte et sur l’appareil.

#### <a name="to-take-a-share-offline"></a>Pour déplacer un partage hors connexion

1. Assurez-vous que le partage en question n’est pas utilisé avant de le mettre hors connexion.
2. Pour déplacer le partage sur le tableau hors connexion, procédez comme suit :
   
    1. À partir du paramètre **Partages** du panneau de synthèse du service StorSimple, sélectionnez le panneau virtuel sur lequel est hébergé le partage que vous souhaitez déplacer hors connexion.

    2. **Sélectionnez** le partage, puis cliquez sur **...** (sinon cliquez avec le bouton droit sur cette ligne), puis dans le menu contextuel, sélectionnez **Mettre hors connexion**.
     
        ![Partage hors connexion](./media/storsimple-virtual-array-manage-shares/shares-offline.png)

    3. Passez en revue les informations du panneau **Mettre hors connexion**, puis confirmez votre acceptation de l’opération. Cliquez sur **Mettre hors connexion** pour déplacer le partage. Une notification de l’opération en cours s’affiche.

    4. Pour vérifier que le partage a été mis hors connexion, accédez au panneau **Partages**. Le partage doit apparaître comme élément hors connexion.

## <a name="delete-a-share"></a>Supprimer un partage

> [!IMPORTANT]
> Un partage peut être supprimé uniquement s’il est hors connexion.


Pour supprimer un partage, procédez comme suit.

#### <a name="to-delete-a-share"></a>Pour supprimer un partage

1. À partir du paramètre **Partages** du panneau de synthèse du service StorSimple, sélectionnez le tableau virtuel sur lequel est hébergé le partage à supprimer.
2. **Sélectionnez** le partage, puis cliquez sur **...** (sinon cliquez avec le bouton droit sur cette ligne), puis dans le menu contextuel, sélectionnez **Supprimer**.
   
    ![Supprimer un partage](./media/storsimple-virtual-array-manage-shares/share-delete.png)
3. Vérifiez l’état du partage à supprimer. Si le partage que vous souhaitez supprimer n’est pas hors connexion, déplacez-le dans un premier temps. Suivez les étapes de [Mettre un partage hors connexion](#take-a-share-offline).
4. Lorsque vous êtes invité à confirmer l’opération dans le panneau **Supprimer**, validez la confirmation, puis cliquez sur **Supprimer**. Le partage est mis à jour, et le panneau **Partages** représente la liste mise à jour des partages dans le tableau virtuel.

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [cloner un partage StorSimple](storsimple-virtual-array-clone.md).

