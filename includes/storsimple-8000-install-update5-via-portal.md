---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 9832a961562930e61e94b28ed78976fd9f8c6f78
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93376114"
---
#### <a name="to-install-an-update-from-the-azure-portal"></a>Pour installer une mise à jour à partir du portail Azure

1. Dans la page du service StorSimple, sélectionnez votre appareil.

    ![Sélectionnez l’appareil](./media/storsimple-8000-install-update5-via-portal/update1.png)

2. Accédez à **Paramètres de l’appareil** > **Mises à jour de l’appareil**.

    ![Cliquez sur Mises à jour de l’appareil](./media/storsimple-8000-install-update5-via-portal/update2.png)

2. Une notification s’affiche si de nouvelles mises à jour sont disponibles. Dans le panneau **Mises à jour de l’appareil**, vous pouvez également cliquer sur **Rechercher les mises à jour**. Une tâche est créée pour rechercher les mises à jour disponibles. Un message s’affiche lorsque la tâche est terminée.

    ![Cliquez sur Mises à jour de l’appareil 2](./media/storsimple-8000-install-update5-via-portal/update3.png)

3. Nous vous recommandons de consulter les notes de publication avant d’appliquer une mise à jour sur votre appareil. Pour appliquer les mises à jour, cliquez sur **Installer les mises à jour**. Dans le panneau **Confirmer les mises à jour standard**, passez en revue les conditions préalables à remplir avant d’appliquer les mises à jour. Cochez la case pour indiquer que vous êtes prêt à mettre à jour l’appareil, puis cliquez sur **Installer**.

    ![Cliquez sur Mises à jour de l’appareil 3](./media/storsimple-8000-install-update5-via-portal/update4.png)

6. Un ensemble de vérifications préalables démarre. Ces vérifications incluent :
   
   * **Contrôles d’intégrité des contrôleurs** , destinés à vérifier que les contrôleurs d’appareil sont en bon état et en ligne.
   * **Contrôles d’intégrité des composants matériels** , utilisés pour vérifier l’état des composants matériels de votre appareil StorSimple.
   * **Contrôles DATA 0** , afin de s’assurer de l’activation de DATA 0 sur votre appareil. Si cette interface n’est pas activée, vous devez l’activer, puis réessayer.

     La mise à jour est téléchargée et installée uniquement si toutes les vérifications sont effectuées avec succès. Un message s’affiche lorsque les vérifications sont en cours. Si les pré-vérifications échouent, les causes de cet échec vous sont communiquées. Résolvez ces problèmes, puis retentez l’opération. Vous devrez peut-être contacter le support technique Microsoft si vous ne pouvez pas résoudre ces problèmes vous-même.

7. Une fois les vérifications préalables terminées, une tâche de mise à jour est créée. Un message s’affiche une fois la tâche de mise à jour créée.
   
    ![Création de la tâche de mise à jour](./media/storsimple-8000-install-update5-via-portal/update6.png)
   
    La mise à jour est ensuite appliquée à votre appareil.

9. La mise à jour prend quelques heures. Sélectionnez la tâche de mise à jour et cliquez sur **Détails** pour afficher les détails de la tâche à tout moment.

    ![Création de la tâche de mise à jour 2](./media/storsimple-8000-install-update5-via-portal/update8.png)

     Vous pouvez également surveiller la progression de la tâche de mise à jour à partir de **Paramètres de l’appareil > Travaux**. Le panneau **Travaux** indique la progression de la mise à jour.

     ![Création de la tâche de mise à jour 3](./media/storsimple-8000-install-update5-via-portal/update7.png)

10. Une fois la tâche terminée, accédez à **Paramètres de l’appareil > Mises à jour de l’appareil**. La version du logiciel doit maintenant être mise à jour.

