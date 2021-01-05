---
title: Sauvegarde et restauration - Portail Azure - Azure Database pour MySQL
description: Cet article décrit comment restaurer un serveur Azure Database pour MySQL à l’aide du portail.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 6/30/2020
ms.openlocfilehash: 9bc31cf8fee2669634ff366caac77cb090baf075
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000298"
---
# <a name="how-to-backup-and-restore-a-server-in-azure-database-for-mysql-using-the-azure-portal"></a>Comment sauvegarder et restaurer un serveur dans Azure Database pour MySQL à l’aide du Portail Azure

## <a name="backup-happens-automatically"></a>La sauvegarde s’effectue automatiquement
Les serveurs Azure Database pour MySQL sont sauvegardés régulièrement pour activer les fonctionnalités de restauration. À l’aide de cette fonctionnalité, vous pouvez restaurer le serveur et toutes ses bases de données à un point dans le temps antérieur, sur un nouveau serveur.

## <a name="prerequisites"></a>Prérequis
Pour utiliser ce guide pratique, il vous faut :
- Un [serveur et une base de données Azure Database pour MySQL](quickstart-create-mysql-server-database-using-azure-portal.md)

## <a name="set-backup-configuration"></a>Définir la configuration de sauvegarde

Vous choisissez entre la configuration de votre serveur pour des sauvegardes redondantes localement ou géographiquement redondantes lors de la création du serveur, dans la fenêtre **Niveau tarifaire**.

> [!NOTE]
> Après la création d’un serveur, son type de redondance (géographique ou locale) ne peut pas être modifié.
>

Lors de la création d’un serveur via le portail Azure, la fenêtre **Niveau tarifaire** vous permet de sélectionner **Localement redondant** ou **Géographiquement redondant** pour les sauvegardes de votre serveur. Cette fenêtre vous permet également de sélectionner la **période de rétention de la sauvegarde**, c’est-à-dire la durée (en nombre de jours) pendant laquelle vous souhaitez conserver les sauvegardes de serveur.

   :::image type="content" source="./media/howto-restore-server-portal/pricing-tier.png" alt-text="Niveau tarifaire - Choisir la redondance de sauvegarde":::

Pour plus d’informations sur la définition de ces valeurs lors de la création, consultez [Démarrage rapide du serveur Azure Database pour MySQL](quickstart-create-mysql-server-database-using-azure-portal.md).

La période de rétention de sauvegarde peut être modifiée sur un serveur en suivant les étapes ci-après :
1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Sélectionnez votre serveur Azure Database pour MySQL. Cette action ouvre la page **Vue d’ensemble**.
3. Sélectionnez **Niveau tarifaire** dans le menu, sous **PARAMÈTRES**. Le curseur vous permet de modifier la **période de rétention de sauvegarde** selon vos préférences entre 7 et 35 jours.
Dans la capture d’écran ci-dessous, elle a été augmentée à 34 jours.
:::image type="content" source="./media/howto-restore-server-portal/3-increase-backup-days.png" alt-text="Période de rétention de sauvegarde augmentée":::

4. Cliquez sur **OK** pour confirmer la modification.

La période de rétention de sauvegarde détermine jusqu’à quelle date une restauration à un point dans le temps peut être récupérée, dans la mesure où elle est basée sur les sauvegardes disponibles. La restauration à un point dans le temps est décrite plus loin dans la section suivante. 

## <a name="point-in-time-restore"></a>Restauration dans le temps
Azure Database pour MySQL vous permet de restaurer le serveur à un moment donné et dans une nouvelle copie du serveur. Vous pouvez utiliser ce nouveau serveur pour récupérer vos données ou faire en sorte que vos applications client pointent vers ce nouveau serveur.

Par exemple, si une table a été accidentellement supprimée à midi aujourd'hui, vous pourrez restaurer à l’heure juste avant midi et retrouver la table et les données manquantes à partir de cette nouvelle copie du serveur. La restauration à un point dans le temps est effectuée au niveau du serveur, pas au niveau de la base de données.

Les étapes suivantes restaurent l’exemple de serveur à un point dans le temps :
1. Dans le portail Azure, sélectionnez votre serveur Azure Database pour MySQL. 

2. Dans la barre d’outils de la page **Vue d’ensemble** du serveur, sélectionnez **Restaurer**.

   :::image type="content" source="./media/howto-restore-server-portal/2-server.png" alt-text="Azure Database pour MySQL - Vue d’ensemble - Bouton Restaurer":::

3. Remplissez le formulaire Restaurer avec les informations requises :

   :::image type="content" source="./media/howto-restore-server-portal/3-restore.png" alt-text="Azure Database pour MySQL - Informations de restauration":::
   - **Point de restauration** : sélectionnez le point dans le temps vers lequel vous souhaitez restaurer.
   - **Serveur cible** : Entrez un nom pour le nouveau serveur.
   - **Emplacement** : vous ne pouvez pas sélectionner la région. Par défaut, elle est identique à celle du serveur source.
   - **Niveau tarifaire** : vous ne pouvez pas modifier ces paramètres lorsque vous effectuez une restauration à un point dans le temps. Elle est identique à celle du serveur source. 

4. Cliquez sur **OK** pour restaurer le serveur à un point dans le temps. 

5. Une fois la restauration terminée, recherchez le nouveau serveur créé pour vérifier que les données ont été restaurées correctement.

Le serveur créé par la restauration à un point dans le temps a les mêmes nom de connexion administrateur de serveur et mot de passe qui étaient valides pour le serveur existant au point dans le temps choisi. Vous pouvez modifier le mot de passe sur la page **Vue d’ensemble** du nouveau serveur.

En outre, une fois l’opération de restauration terminée, deux paramètres de serveur sont réinitialisés aux valeurs par défaut (et ne sont pas copiés à partir du serveur principal) après l’opération de restauration
*   time_zone : valeur à affecter à la valeur par défaut **SYSTEM**
*   event_scheduler : valeur définie sur **OFF** sur le serveur restauré

Vous devrez copier la valeur à partir du serveur principal et la définir sur le serveur restauré en reconfigurant le [paramètre de serveur](howto-server-parameters.md)

Le nouveau serveur créé lors d’une restauration ne dispose pas des points de terminaison de service de réseau virtuel qui se trouvaient sur le serveur d’origine. Ces règles doivent être définies séparément pour ce nouveau serveur. Les règles de pare-feu du serveur d’origine sont restaurées.

## <a name="geo-restore"></a>Restauration géographique
Si vous avez configuré votre serveur pour les sauvegardes redondantes géographiquement, un serveur peut être créé à partir de la sauvegarde de ce serveur existant. Ce nouveau serveur peut être créé dans toutes les régions dans lesquelles Azure Database pour MySQL est disponible.  

1. Cliquez sur le bouton **Créer une ressource** (+) dans le coin supérieur gauche du portail. Sélectionnez **Bases de données** > **Azure Database pour MySQL**.

   :::image type="content" source="./media/howto-restore-server-portal/1_navigate-to-mysql.png" alt-text="Accédez à Azure Database pour MySQL.":::
 
2. Indiquez l’abonnement, le groupe de ressources et le nom du nouveau serveur. 

3. Sélectionnez **Sauvegarde** comme **Source de données**. Cette action charge une liste déroulante des serveurs pour lesquels les sauvegardes géoredondantes sont activées.
   
   :::image type="content" source="./media/howto-restore-server-portal/3-geo-restore.png" alt-text="Sélectionnez la source de données.":::
    
   > [!NOTE]
   > Lorsqu’un serveur est créé, il est possible qu’il ne soit pas immédiatement disponible pour la restauration géographique. Le remplissage des métadonnées nécessaires peut prendre quelques heures.
   >

4. Sélectionnez la liste déroulante **Sauvegarde**.
   
   :::image type="content" source="./media/howto-restore-server-portal/4-geo-restore-backup.png" alt-text="Sélectionnez la liste déroulante de sauvegarde.":::

5. Sélectionnez le serveur source à partir duquel effectuer la restauration.
   
   :::image type="content" source="./media/howto-restore-server-portal/5-select-backup.png" alt-text="Sélectionnez une sauvegarde.":::

6. Le serveur utilisera par défaut les valeurs pour le nombre de **vCores**, la **période de conservation de sauvegarde**, l’**option de redondance de sauvegarde**, la **version du moteur** et les **informations d’identification d’administrateur**. Sélectionnez **Continuer**. 
   
   :::image type="content" source="./media/howto-restore-server-portal/6-accept-backup.png" alt-text="Poursuivez la sauvegarde.":::

7. Renseignez le reste du formulaire avec vos préférences. Vous pouvez sélectionner n’importe quel **Emplacement**.

    Après avoir sélectionné l’emplacement, vous pouvez sélectionner **Configurer le serveur** pour mettre à jour la **génération de calcul** (si disponible dans la région choisie), le nombre de **vCores**, la **période de conservation de sauvegarde** et l’**option de redondance de sauvegarde**. La modification du **niveau tarifaire** (De base, Usage général ou À mémoire optimisée) ou de la taille du **stockage** pendant la restauration n’est pas prise en charge.

   :::image type="content" source="./media/howto-restore-server-portal/7-create.png" alt-text="Remplissez le formulaire."::: 

8. Sélectionnez **Vérifier + créer** pour passer en revue vos sélections. 

9. Sélectionnez **Créer** pour approvisionner le serveur. Cette opération peut prendre quelques minutes.

Le serveur créé par la restauration géographique a les mêmes nom de connexion administrateur de serveur et mot de passe qui étaient valides pour le serveur existant au moment où la restauration a été initiée. Le mot de passe peut être modifié sur la page **Vue d’ensemble** du nouveau serveur.

Le nouveau serveur créé lors d’une restauration ne dispose pas des points de terminaison de service de réseau virtuel qui se trouvaient sur le serveur d’origine. Ces règles doivent être définies séparément pour ce nouveau serveur. Les règles de pare-feu du serveur d’origine sont restaurées.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [sauvegardes](concepts-backup.md) du service.
- En savoir plus sur les [réplicas](concepts-read-replicas.md).
- En savoir plus sur les options de [continuité d’activité](concepts-business-continuity.md).