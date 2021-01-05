---
title: Basculement et récupération d’urgence pour StorSimple Virtual Array
description: Découvrez la récupération d’urgence pour votre Microsoft Azure StorSimple Virtual Array, notamment la procédure détaillée pour basculer vers un autre Virtual Array.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 3c1f9c62-af57-4634-a0d8-435522d969aa
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 509a3ac383cebd91821e9c4b872c253ab3b0a947
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005858"
---
# <a name="disaster-recovery-and-device-failover-for-your-storsimple-virtual-array-via-azure-portal"></a>Basculement d'appareil et récupération d'urgence pour votre StorSimple Virtual Array via le portail Azure

## <a name="overview"></a>Vue d’ensemble
Cet article décrit la récupération d’urgence pour votre Microsoft Azure StorSimple Virtual Array, notamment la procédure détaillée pour basculer vers un autre Virtual Array. Un basculement vous permet de déplacer vos données depuis un appareil *source* du centre de données vers un appareil *cible*. L’appareil cible peut se trouver dans le même ou un autre emplacement géographique. Le basculement de l'appareil s'applique à l'ensemble de l'appareil. Lors du basculement, les données de cloud pour l'appareil source deviennent la propriété de l'appareil cible.

Cet article s'applique aux StorSimple Virtual Arrays uniquement. Pour le basculement d’un appareil de la gamme 8000, consultez [Basculement d’appareil et récupération d’urgence pour votre appareil StorSimple](./storsimple-8000-device-failover-disaster-recovery.md).

## <a name="what-is-disaster-recovery-and-device-failover"></a>En quoi consistent la récupération d’urgence et le basculement d’appareil ?

Dans un scénario de récupération d’urgence, l’appareil principal cesse de fonctionner. Dans ce scénario, vous pouvez déplacer les données de cloud associées à l’appareil défaillant vers un autre appareil. Vous pouvez utiliser l’appareil principal en tant que *source* et spécifier un autre appareil en tant que *cible*. Ce processus est appelé le *basculement*. Pendant le basculement, la propriété de tous les volumes ou les partages de l'appareil source change et ceux-ci sont transférés vers l'appareil cible. Aucun filtrage des données n'est autorisé.

La récupération d'urgence est modelée comme une restauration complète de l'appareil à l'aide du suivi et de la hiérarchisation basés sur la carte thermique. Une carte thermique est définie par l'affectation d'une valeur de chaleur aux données selon les modèles de lecture et d'écriture. Cette carte thermique transmet les segments de données à chaleur faible vers le cloud d'abord tout en conservant les segments de données à chaleur élevée (plus utilisés) au niveau local. Pendant une récupération d’urgence, StorSimple utilise la carte thermique pour restaurer et rafraîchir les données à partir du cloud. L'appareil extrait tous les volumes/partages dans la dernière sauvegarde récente (comme déterminé en interne) et effectue une restauration à partir de cette sauvegarde. Le Virtual Array orchestre l’ensemble du processus de récupération d’urgence.

> [!IMPORTANT]
> L’appareil source étant supprimé à la fin du basculement d’appareil, aucune restauration automatique n’est prise en charge.
> 
> 

La récupération d’urgence est orchestrée via la fonctionnalité de basculement de l’appareil et lancée à partir du panneau **Appareils**. Ce panneau répertorie tous les appareils StorSimple connectés à votre service StorSimple Device Manager. Vous pouvez voir le nom convivial, le statut, la capacité maximale et d’approvisionnement, le type et le modèle de chaque appareil.

## <a name="prerequisites-for-device-failover"></a>Configuration requise pour le basculement d'appareil

### <a name="prerequisites"></a>Conditions préalables requises

Pour permettre le basculement d’un appareil, assurez-vous que les conditions préalables suivantes sont satisfaites :

* L'appareil source doit se trouver dans un état **Désactivé** .
* L’appareil cible doit s’afficher avec l’état **Prêt pour la configuration** dans le portail Azure. Approvisionnez un Virtual Array cible avec une capacité identique ou supérieure. Utilisez l’interface utilisateur web locale pour configurer et inscrire correctement le Virtual Array cible.
  
  > [!IMPORTANT]
  > N’essayez pas de configurer l’appareil virtuel inscrit via le service. Aucune configuration d'appareil ne doit être effectuée via le service.
  > 
  > 
* L’appareil cible ne peut pas porter le même nom que l’appareil source.
* Les appareils source et cible doivent être du même type. Vous pouvez uniquement basculer un Virtual Array configuré comme un serveur de fichiers vers un autre serveur de fichiers. Il en est de même pour un serveur iSCSI.
* Pour la récupération d’urgence d’un serveur de fichiers, nous vous recommandons de joindre l’appareil cible au même domaine que la source. Cette configuration permet de garantir la résolution automatique des autorisations de partage. Seul le basculement vers un appareil cible dans le même domaine est pris en charge.
* Les appareils cibles disponibles pour la récupération d'urgence sont des appareils ayant une capacité égale ou supérieure par rapport à l'appareil source. Les appareils connectés à votre service qui ne présentent pas un espace suffisant ne sont pas disponibles en tant qu’appareils cibles.

### <a name="other-considerations"></a>Autres considérations

* Pour un basculement planifié :
  
  * Nous vous recommandons de mettre hors connexion tous les volumes ou partages sur l'appareil source.
  * Nous vous recommandons d’effectuer une sauvegarde de l’appareil, puis de continuer le basculement pour minimiser la perte de données.
* Dans le cas d’un basculement non planifié, l’appareil utilise la sauvegarde la plus récente pour restaurer les données.

### <a name="device-failover-prechecks"></a>Vérifications préalables au basculement de l’appareil

Avant le début de la récupération d’urgence, l’appareil effectue des vérifications préalables. Ces vérifications permettent d’éliminer les risques de survenue d’erreurs une fois la récupération d’urgence commencée. Les vérifications préalables sont les suivantes :

* Validation du compte de stockage
* Vérification de la connectivité du cloud dans Azure
* Vérification de l’espace disponible sur l’appareil cible
* Vérification de la validité des noms ACR du volume de l’appareil source
  
  * d’un serveur iSCSI
  * Validité du nom IQN (moins de 220 caractères)
  * Validité des mots de passe CHAP (12 à 16 caractères)

Si l’une des vérifications préalables ci-dessus échoue, vous ne pouvez pas procéder à la récupération d’urgence. Vous devez résoudre ces problèmes, puis réessayer d’exécuter la récupération d’urgence.

Lorsque la récupération d'urgence est terminée, la propriété des données cloud sur l'appareil source est transférée vers l'appareil cible. L'appareil source n'est alors plus disponible dans le portail. L'accès à tous les volumes/partages sur l'appareil source est bloqué et l'appareil cible devient actif.

> [!IMPORTANT]
> Même si l'appareil n'est plus disponible, la machine virtuelle que vous avez configurée sur le système hôte continue de consommer des ressources. Une fois que la récupération d’urgence est terminée, vous pouvez supprimer cette machine virtuelle de votre système hôte.
> 
> 

## <a name="fail-over-to-a-virtual-array"></a>Basculement vers un tableau virtuel

Avant d’exécuter cette procédure, nous vous recommandons d’approvisionner, de configurer et d’inscrire un autre StorSimple Virtual Array auprès de votre service StorSimple Device Manager.

> [!IMPORTANT]
> 
> * Vous ne pouvez pas effectuer un basculement d’un appareil de la gamme StorSimple 8000 vers un appareil virtuel de la gamme 1200.
> * Vous pouvez effectuer un basculement d’un appareil virtuel avec mode FIPS (Federal Information Processing Standard) vers un autre appareil avec mode FIPS ou un appareil non FIPS déployé dans le portail Government.


Procédez comme suit pour restaurer votre appareil vers un appareil virtuel StorSimple cible.

1. Approvisionnez et configurez un appareil cible qui satisfait la [configuration requise pour le basculement d’appareil](#prerequisites). Finalisez la configuration de l’appareil via l’interface utilisateur web locale et inscrivez-le auprès de votre service StorSimple Device Manager. Si vous créez un serveur de fichiers, passez à l’étape 1 de [Configurer en tant que serveur de fichiers](storsimple-virtual-array-deploy3-fs-setup.md#step-1-complete-the-local-web-ui-setup-and-register-your-device). Si vous créez un serveur de fichiers, passez à l’étape 1 de [Configurer en tant que serveur iSCSI](storsimple-virtual-array-deploy3-iscsi-setup.md#step-1-complete-the-local-web-ui-setup-and-register-your-device).

2. Mettez les volumes/partages hors connexion sur l'ordinateur hôte. Pour mettre les volumes/partages hors connexion, reportez-vous aux instructions spécifiques au système d’exploitation de l’ordinateur hôte. Si ce n’est pas déjà fait, vous devez mettre tous les volumes ou partages hors connexion sur l’appareil en procédant comme suit.
   
    1. Accédez au panneau **Appareils** et sélectionnez votre appareil.
   
    2. Accédez à **Paramètres > Gérer > Partages** (ou **Paramètres > Gérer > Volumes**). 
   
    3. Sélectionnez un partage/volume, cliquez avec le bouton droit et sélectionnez **Mettre hors connexion**. 
   
    4. Lorsque vous êtes invité à confirmer l’opération, cochez la case **Je comprends l’impact de la mise hors connexion de ce partage**. 
   
    5. Cliquez sur **Mettre hors connexion**.

3. Dans votre service StorSimple Device Manager, accédez à **Gestion > Appareils**. Dans le panneau **Appareils**, sélectionnez l’appareil source et cliquez dessus.

4. Dans le **tableau de bord de votre appareil**, cliquez sur **Désactiver**.

5. Dans le panneau **Désactiver**, vous êtes invité à confirmer l’opération. La désactivation d’appareil est une opération *définitive* qui ne peut pas être annulée. Vous recevez également un rappel pour mettre vos partages/volumes hors connexion sur l’ordinateur hôte. Tapez le nom de l’appareil pour confirmer l’opération et cliquez sur **Désactivez**.
   
    ![Capture d’écran du panneau Désactiver. La zone de nom de l’appareil est renseignée, et le bouton Désactiver est mis en surbrillance.](./media/storsimple-virtual-array-failover-dr/failover1.png)
6. La désactivation démarre. Vous recevez une notification une fois que la désactivation s’est correctement terminée.
   
    ![Capture d’écran d’une barre de progression indiquant que l’appareil est en cours de désactivation.](./media/storsimple-virtual-array-failover-dr/failover2.png)
7. Dans la page Appareils, l’état de l’appareil passe à **Désactivé**.
    ![Capture d’écran de la page Appareils. Les propriétés de l’appareil désactivé s’affichent, y compris l’état, qui est listé comme Désactivé.](./media/storsimple-virtual-array-failover-dr/failover3.png)
8. Dans la page **Appareils**, sélectionnez l’appareil source désactivé et cliquez dessus pour le basculement. 
9. Dans le **tableau de bord de l’appareil**, cliquez sur **Effectuer un basculement**. 
10. Dans le panneau **Effectuer un basculement de l’appareil**, effectuez les opérations suivantes :
    
    1. Le champ de l’appareil source est rempli automatiquement. Notez la taille totale des données pour l’appareil source. La taille des données doit être inférieure à la capacité disponible sur l’appareil cible. Examinez les détails associés à l’appareil source tels que le nom de l’appareil, la capacité totale et les noms des partages concernés par le basculement.

    2. Dans la liste déroulante des appareils disponibles, choisissez un **Appareil cible**. Seuls les appareils possédant la capacité suffisante sont affichés dans la liste déroulante.

    3. Cochez la case **Je comprends que cette opération va effectuer un basculement des données sur l’appareil cible**. 

    4. Cliquez sur **Effectuer un basculement**.
    
        ![Capture d’écran du panneau de basculement de l’appareil, avec les appareils source et cible renseignés, l’option cochée et le bouton Effectuer un basculement en surbrillance.](./media/storsimple-virtual-array-failover-dr/failover4.png)
11. Un travail de basculement est lancé et vous recevez une notification. Accédez à **Appareils > Travaux** pour contrôler le basculement.
    
     ![Capture d’écran d’une barre de progression indiquant que l’appareil est en cours de basculement.](./media/storsimple-virtual-array-failover-dr/failover5.png)
12. Dans la page **Travaux**, vous pouvez voir un travail de basculement créé pour l’appareil source. Ce travail effectue les vérifications préalables pour la récupération d'urgence.
    
    ![Capture d’écran montrant qu’un travail de basculement a démarré avec succès.](./media/storsimple-virtual-array-failover-dr/failover6.png)
    
     Une fois les vérifications préalables pour la récupération d'urgence terminées avec succès, le travail de basculement génèrera des travaux de restauration pour chaque volume/partage qui existe sur l'appareil source.
    
    ![Capture d’écran montrant les détails du travail de basculement, tels que l’état, l’appareil et la durée.](./media/storsimple-virtual-array-failover-dr/failover7.png)
13. Une fois le basculement terminé, accédez au panneau **Appareils**.
    
    1. Sélectionnez l’appareil StorSimple qui a été utilisé en tant qu’appareil cible pour le processus de basculement et cliquez dessus.
    2. Accédez à **Paramètres > Administration > Partages** (ou **Volumes** dans le cas d’un serveur iSCSI). Dans le panneau **Partages**, vous pouvez afficher tous les partages (volumes) de l’ancien appareil.
        ![Capture d’écran du panneau Appareils. L’appareil cible est listé avec un état En ligne.](./media/storsimple-virtual-array-failover-dr/failover9.png)
14. Vous devez [créer un alias DNS](https://support.microsoft.com/kb/168322) afin que toutes les applications qui essaient de se connecter puissent être redirigées vers le nouvel appareil.

## <a name="errors-during-dr"></a>Erreurs lors de la récupération d'urgence

**Panne de connectivité cloud pendant la récupération d'urgence**

Si la connectivité cloud est interrompue après le démarrage de la récupération d’urgence et avant la fin de la restauration de l’appareil, la récupération d’urgence échoue. Vous recevez une notification d’échec. L’appareil cible pour la récupération d’urgence est marqué comme *inutilisable*. Vous ne pouvez pas utiliser le même appareil cible pour des récupérations d’urgence ultérieures.

**Aucun appareil cible compatible**

Si les appareils cibles disponibles n’ont pas suffisamment d’espace, une erreur s’affiche indiquant qu’il n’y a aucun appareil cible compatible.

**Échecs des vérifications préalables**

Si l’une des vérifications préalables n’est pas satisfaite, des erreurs s’affichent.

## <a name="business-continuity-disaster-recovery-bcdr"></a>Continuité d’activité et récupération d’urgence (Business Continuity Disaster Recovery - BCDR)

Un scénario de continuité d’activité et récupération d’urgence (BCDR) se produit lorsque l’ensemble du centre de données Azure cesse de fonctionner. Cela peut affecter votre service StorSimple Device Manager et les appareils StorSimple associés.

S'il existe des appareils StorSimple inscrits juste avant un incident, ces appareils StorSimple devront peut-être être réinitialisés. Après l'incident, vous pouvez recréer et configurer ces appareils.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la [gestion de StorSimple Virtual Array à l’aide de l’interface utilisateur web locale](storsimple-ova-web-ui-admin.md).