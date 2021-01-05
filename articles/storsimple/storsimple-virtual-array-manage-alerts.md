---
title: Afficher et gérer les alertes pour le StorSimple Virtual Array
description: Décrit les conditions et la gravité des alertes StorSimple Virtual Array et comment utiliser le service StorSimple Manager pour gérer les alertes.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 97ee25a1-0ec3-4883-9a0a-54b722598462
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/12/2018
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 12fcc9996697f3bbba35826d79bec238bfb0f8b3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993011"
---
# <a name="use-storsimple-device-manager-to-manage-alerts-for-the-storsimple-virtual-array"></a>Utiliser StorSimple Device Manager pour gérer les alertes liées au StorSimple Virtual Array

## <a name="overview"></a>Vue d’ensemble

La fonctionnalité Alertes dans le service StorSimple Device Manager vous permet d’examiner et d’effacer en temps réel les alertes relatives au StorSimple Virtual Array. Vous pouvez utiliser les alertes du panneau **Résumé de service** afin de surveiller de manière centralisée les problèmes d’état d’intégrité de vos instances StorSimple Virtual Array et de la solution globale Microsoft Azure StorSimple.

Ce didacticiel décrit comment configurer les notifications d’alerte, les conditions d’alerte courantes, les niveaux de gravité des alertes et comment afficher les alertes et en effectuer le suivi. En outre, il inclut les tables de référence rapide des alertes, ce qui vous permet de localiser rapidement une alerte spécifique et d’agir en conséquence.

![Page des alertes](./media/storsimple-virtual-array-manage-alerts/alerts1.png)

## <a name="configure-alert-settings"></a>Configuration des paramètres d'alerte

Vous pouvez choisir d’être averti par e-mail de la condition des alertes pour chacune des instances StorSimple Virtual Array. En outre, vous pouvez identifier d’autres destinataires de notification des alertes en entrant leurs adresses e-mail dans la zone **Destinataires d’e-mail supplémentaires** , séparées par des points-virgules.

> [!NOTE]
> Vous pouvez entrer un maximum de 20 adresses e-mail par tableau virtuel.

Après avoir activé la notification par e-mail pour un tableau virtuel, les membres de la liste de notification reçoivent un e-mail chaque fois qu’une alerte critique survient. Les messages seront envoyés à partir de *storsimple-alerts-noreply\@mail.windowsazure.com* et décriront la condition d’alerte. Les destinataires peuvent cliquer sur **Se désabonner** pour ne plus faire partie de la liste des notifications par e-mail.

#### <a name="to-enable-email-notification-for-alerts"></a>Pour activer la notification par e-mail des alertes

1. Accédez à votre service StorSimple Device Manager. Dans la section **Gestion**, sélectionnez (cliquez sur) **Appareils**. Dans la liste des appareils affichés, cliquez sur votre appareil.
   
    ![Paramètres d’alerte](./media/storsimple-virtual-array-manage-alerts/alerts2.png)
2. Le panneau **Paramètres** s’ouvre. Dans la section **Paramètres de l’appareil**, sélectionnez **Général**. Le panneau **Paramètres généraux** s’ouvre.
   
    ![Capture d’écran montrant le volet Paramètres de l’appareil avec la zone Paramètres d’alerte mise en évidence.](./media/storsimple-virtual-array-manage-alerts/alerts4.png)
3. Dans le panneau **Paramètres généraux**, accédez à la section **Paramètres d’alerte** puis définissez les éléments suivants :
   
   1. Dans le champ **Activer la notification par e-mail**, sélectionnez **OUI**.
   2. Dans le champ **Envoyer un e-mail aux administrateurs du service**, sélectionnez **OUI** si vous voulez que l’administrateur et tous les coadministrateurs de service reçoivent les notifications d’alerte.
   3. Dans le champ **Destinataires d’e-mail supplémentaires**, entrez les adresses e-mail de tous les autres destinataires qui doivent recevoir les notifications d’alerte. Entrez les noms au format *personne\@lieu.com*. Utilisez des points-virgules pour séparer les adresses e-mail. Vous pouvez configurer un maximum de 20 adresses e-mail par appareil virtuel.
      
       ![Capture d’écran montrant les détails des Paramètres d’alerte avec les paramètres décrits dans cette étape.](./media/storsimple-virtual-array-manage-alerts/alerts6.png)
   4. Pour envoyer un test de notification par e-mail, cliquez sur **Envoyer un e-mail de test**. Le service StorSimple Device Manager affiche des messages d’état lorsqu’il transfère la notification de test.
      
       ![Capture d’écran montrant une boîte de dialogue d’informations sur la vérification de l’e-mail de test.](./media/storsimple-virtual-array-manage-alerts/alerts7.png)
      
      > [!NOTE]
      > Si le message de notification de test ne peut pas être envoyé, le service StorSimple Device Manager affiche un message approprié. Cliquez sur **OK**, attendez quelques minutes, puis réessayez d'envoyer votre message de notification de test.
      >
      >
   5. En bas de la page, cliquez sur **Enregistrer** pour enregistrer votre configuration. Cliquez sur **Oui** lorsque vous êtes invité à confirmer l’opération.
      
      ![Capture d’écran montrant le volet des paramètres avec le bouton Enregistrer sélectionné.](./media/storsimple-virtual-array-manage-alerts/alerts10.png)

## <a name="common-alert-conditions"></a>Conditions d’alerte courantes

Votre StorSimple Virtual Array génère des alertes en réponse à différentes conditions. Les éléments suivants constituent les types de conditions d’alerte les plus courants :

* **Problèmes de connectivité** : ces alertes se produisent lorsque des difficultés surviennent dans le transfert de données. Des problèmes de communication peuvent survenir pendant le transfert des données vers et depuis le compte de stockage Azure ou en raison d’une absence de connectivité entre les appareils virtuels et le service StorSimple Device Manager. Les problèmes de communication sont les plus difficiles à résoudre, car il existe de nombreux points de défaillance. Avant toute chose, vous devez toujours vérifier que la connectivité au réseau et l'accès à Internet sont disponibles avant d’aller plus loin dans la résolution des problèmes. Pour plus d’informations sur les ports et les paramètres du pare-feu, consultez la page [Configuration système requise pour StorSimple Virtual Array](storsimple-ova-system-requirements.md). Pour obtenir de l’aide sur la résolution des problèmes, consultez la section [Dépannage avec l’applet de commande Test-Connection](./storsimple-8000-troubleshoot-deployment.md).
* **Problèmes de performances** : ces alertes sont dues aux performances de votre système qui ne sont pas optimales, notamment lorsqu’il est soumis à des conditions de surcharge.

De plus, vous pouvez afficher les alertes liées à la sécurité, aux mises à jour ou aux tâches qui ont échoué.

## <a name="alert-severity-levels"></a>Niveaux de gravité des alertes

Les alertes présentent différents niveaux de gravité, en fonction de l'impact de la situation d'alerte et de la nécessité d'une réponse à cette alerte. Les niveaux de gravité des alertes sont :

* **Critique** : cette alerte répond à une condition qui affecte les performances optimales de votre système. Il est nécessaire d’agir pour vous assurer que le service StorSimple n'est pas interrompu.
* **Avertissement** : cette condition peut devenir critique si elle n’est pas résolue. Vous devez examiner la situation et prendre toutes les dispositions nécessaires pour résoudre le problème.
* **Information** : cette alerte contient des informations qui peuvent être utiles pour le suivi et la gestion de votre système.

## <a name="view-and-track-alerts"></a>Afficher et effectuer le suivi des alertes

Le panneau de synthèse du service StorSimple Device Manager vous offre un aperçu rapide du nombre d’alertes sur vos appareils virtuels, classées par niveau de gravité.

![Tableau de bord des alertes](./media/storsimple-virtual-array-manage-alerts/alerts14.png)

Cliquez sur le niveau de gravité afin d’ouvrir le panneau **Alertes**. Les résultats incluent uniquement les alertes qui correspondent à ce niveau de gravité.

![Le rapport des alertes est limité au type d'alerte](./media/storsimple-virtual-array-manage-alerts/alerts15.png)

En cliquant sur une alerte dans la liste, vous obtenez des détails supplémentaires sur l’alerte, notamment la dernière fois que l’alerte a été signalée, le nombre d’occurrences de cette alerte sur l’appareil et l’action recommandée pour résoudre cette alerte.

![Liste des alertes et détails](./media/storsimple-virtual-array-manage-alerts/alerts16.png)

Si vous devez envoyer les informations au Support Microsoft, vous pouvez copier les détails de l’alerte dans un fichier texte. Une fois que vous avez suivi les recommandations et résolu les alertes en local, vous avez intérêt à supprimer l’alerte de la liste. Sélectionnez l’alerte dans la liste, puis cliquez sur **Effacer**. Pour effacer plusieurs alertes, sélectionnez chaque alerte, cliquez sur n’importe quelle colonne, sauf la colonne **Alerte**, puis cliquez sur **Effacer** après avoir sélectionné toutes les alertes à effacer.

Lorsque vous cliquez sur **Effacer**, vous avez la possibilité d’insérer des commentaires sur l'alerte et les étapes que vous avez suivies pour résoudre le problème.

![commentaires d’alerte](./media/storsimple-virtual-array-manage-alerts/alerts17.png)

Certains événements seront effacés par le système si un autre événement est déclenché avec de nouvelles informations.

## <a name="sort-and-review-alerts"></a>Tri et vérification des alertes

Le panneau **Alertes** peut afficher jusqu’à 250 alertes. Si vous avez dépassé ce nombre d'alertes, toutes les alertes ne seront pas affichées dans l’affichage par défaut. Vous pouvez combiner les champs suivants pour personnaliser les alertes qui s’affichent :

* **État** : vous pouvez afficher des alertes **Actives** ou **Effacées**. Les alertes actives sont toujours déclenchées dans votre système, tandis que les alertes effacées ont été effacées manuellement par un administrateur, ou elles ont été effacées par un programme, car le système a mis à jour la condition d'alerte avec de nouvelles informations.
* **Gravité** : vous pouvez afficher des alertes de tous les niveaux de gravité (critique, avertissement, information), ou seulement celles d’une certaine gravité, par exemple les alertes critiques uniquement.
* **Source** : vous pouvez afficher les alertes de toutes les sources, ou vous limiter aux alertes qui proviennent d’un service ou d’un ou de tous les appareils virtuels.
* **Intervalle de temps** : en spécifiant les champs d’horodatage **De** et **À**, vous pouvez afficher les alertes pour la période qui vous intéresse.

## <a name="alerts-quick-reference"></a>Référence rapide des alertes

Les tableaux suivants répertorient certaines des alertes StorSimple que vous pouvez rencontrer, ainsi que des informations supplémentaires et des recommandations lorsqu’elles sont disponibles. Les alertes de StorSimple Virtual Array appartiennent à l’une des catégories suivantes :

* [Alertes de connectivité au cloud](#cloud-connectivity-alerts)
* [Alertes de configuration](#configuration-alerts)
* [Alertes d'échec de tâche](#job-failure-alerts)
* [Alertes de performances](#performance-alerts)
* [Alertes de sécurité](#security-alerts)

### <a name="cloud-connectivity-alerts"></a>Alertes de connectivité au cloud

| Texte d'alerte | Événement | Plus d'informations/actions recommandées |
|:--- |:--- |:--- |
| L'appareil <*nom de l'appareil*> n'est pas connecté au cloud. |L’appareil nommé ne peut pas se connecter au cloud. |Impossible de se connecter au cloud. La raison peut être l’une des suivantes :<ul><li>Il peut y avoir un problème avec les paramètres réseau de votre appareil.</li><li>Il peut y avoir un problème avec les informations d’identification du compte de stockage.</li></ul>Pour plus d’informations sur la résolution des problèmes de connectivité, accédez à [l’interface utilisateur web locale](storsimple-ova-web-ui-admin.md) de l’appareil. |

### <a name="configuration-alerts"></a>Alertes de configuration

| Texte d'alerte | Événement | Plus d'informations/actions recommandées |
|:--- |:--- |:--- |
| La configuration d’un appareil virtuel local n’est pas prise en charge. |Performances lentes. |La configuration actuelle peut aboutir à une dégradation des performances. Vérifiez que votre serveur répond à la configuration minimale requise. Pour plus d’informations, accédez à [Configuration requise pour StorSimple Virtual Array](storsimple-ova-system-requirements.md). |
| Vous manquez d'espace disque alloué sur <*nom de l'appareil*\>. |Avertissement d’espace disque. |Vous n’avez presque plus d’espace disque approvisionné. Pour libérer de l’espace, envisagez de déplacer des charges de travail vers un autre volume ou partage ou de supprimer des données. |

### <a name="job-failure-alerts"></a>Alertes d'échec de tâche

| Texte d'alerte | Événement | Plus d'informations/actions recommandées |
|:--- |:--- |:--- |
| <*Nom de l'appareil*\> n'a pas pu être sauvegardé. |Échec du travail de sauvegarde. |Impossible de créer une sauvegarde. Envisagez l’une des possibilités suivantes :<ul><li>Des problèmes de connectivité peuvent empêcher la réalisation de l'opération de sauvegarde. Vérifiez qu’il n’existe aucun problème de connectivité. Pour plus d’informations sur la résolution des problèmes de connectivité, accédez à [l’interface utilisateur web locale](storsimple-ova-web-ui-admin.md) de votre appareil virtuel.</li><li>Vous avez atteint la limite de stockage disponible. Pour libérer de l’espace, envisagez de supprimer les sauvegardes devenues inutiles.</li></ul> Résolvez les problèmes, effacez l’alerte et recommencez l’opération. |
| <*Nom de l'appareil*\> n'a pas pu être cloné. |Échec du travail de clonage. |La création d’un clone a échoué. Envisagez l’une des possibilités suivantes :<ul><li>Votre liste de sauvegarde n’est peut-être pas valide. Actualisez-la pour vérifier qu’elle est toujours valide.</li><li>Des problèmes de connectivité peuvent empêcher la bonne exécution de l’opération de clonage. Vérifiez qu’il n’existe aucun problème de connectivité.</li><li>Vous avez atteint la limite de stockage disponible. Pour libérer de l’espace, envisagez de supprimer les sauvegardes devenues inutiles.</li></ul>Résolvez les problèmes, effacez l’alerte et recommencez l’opération. |

### <a name="networking-alerts"></a>Alertes de réseau

| Texte d'alerte | Événement | Plus d'informations/actions recommandées |
|:--- |:--- |:--- |
| Impossible de se connecter au service d’authentification. |Erreur de chemin d’accès des données |L’URL qui est utilisée pour l’authentification n’est pas accessible. Vérifiez que vos règles de pare-feu incluent les modèles d’URL spécifiés pour l’appareil StorSimple. Pour plus d’informations sur les modèles d’URL dans le portail Azure, accédez à [Configuration requise du réseau pour StorSimple Virtual Array](storsimple-ova-system-requirements.md#url-patterns-for-firewall-rules).|

### <a name="performance-alerts"></a>Alertes de performances

| Texte d'alerte | Événement | Plus d'informations/actions recommandées |
|:--- |:--- |:--- |
| Vous rencontrez des retards inattendus dans le transfert de données. |Transfert de données lent. |Les erreurs de limitation se produisent lorsque vous dépassez les valeurs cibles d’évolutivité d’un service de stockage. Le service de stockage effectue cette opération afin de s’assurer qu’aucun client ne peut utiliser le service au détriment des autres utilisateurs. Pour plus d’informations sur la résolution des problèmes de votre compte de stockage Azure, accédez à [Surveiller, diagnostiquer et résoudre les problèmes liés à Microsoft Azure Storage](../storage/common/storage-monitoring-diagnosing-troubleshooting.md). |
| Vous manquez d'espace de réservation sur le disque local de <*nom de l'appareil*\>. |Temps de réponse lent. |10 % de la taille totale allouée à <*nom de l'appareil*\> sont réservés sur l'appareil local et vous êtes maintenant à court d'espace réservé. La charge de travail de <*nom de l'appareil*\> génère un taux d'évolution élevé ou bien vous avez récemment migré une grande quantité de données. Les performances peuvent s’en trouver réduites. Envisagez l’une des actions suivantes pour résoudre ce problème :<ul><li>Augmenter la bande passante du cloud pour cet appareil.</li><li>Réduire ou déplacer des charges de travail vers un autre volume ou partage.</li></ul> |

### <a name="security-alerts"></a>Alertes de sécurité

| Texte d'alerte | Événement | Plus d'informations/actions recommandées |
|:--- |:--- |:--- |
| Le mot de passe de <*nom de l'appareil*\> expirera dans <*nombre*\> jours. |Avertissement de mot de passe. |Votre mot de passe expirera dans <*nombre*\> jours. Pensez à le changer. Pour plus d’informations, accédez à [Modifier le mot de passe administrateur de l’appareil StorSimple Virtual Array](storsimple-virtual-array-change-device-admin-password.md). |

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez le StorSimple Virtual Array](storsimple-ova-overview.md).