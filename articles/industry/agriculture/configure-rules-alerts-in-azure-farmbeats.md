---
title: Configurer des règles et gérer des alertes
description: Décrit comment configurer des règles et gérer des alertes dans FarmBeats
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.openlocfilehash: 5e6a5d414c341f482c3fddf95a2f8bb8e55a3ca2
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92168543"
---
# <a name="configure-rules-and-manage-alerts"></a>Configurer des règles et gérer des alertes

Azure FarmBeats vous permet de créer des règles basées sur la logique métier et sur les données de capteur transmises par les capteurs et appareils déployés dans votre ferme. Les règles déclenchent des alertes dans le système dès lors que des valeurs de capteur dépassent un certain seuil. En visualisant et en analysant les alertes créées en fonction de valeurs de seuil, vous pouvez rapidement réagir aux différents problèmes et déterminer les solutions requises.

## <a name="create-rule"></a>Créer une règle

1. Dans la page d’accueil, accédez à **Rules** .
2. Sélectionnez **New Rule** . La fenêtre New Rule s’affiche.

    ![Capture d’écran sur laquelle le bouton Nouvelle règle et la section Nouvelle règle sont encadrés en rouge.](./media/configure-rules-and-alerts-in-azure-farmbeats/new-rule-1.png)

3. Renseignez les champs **Rule Name** et **Rule Description** , puis sélectionnez une ferme dans le menu déroulant **Select Farm** .
4. Tapez le nom de votre ferme pour la sélectionner. La section **Conditions** s’affiche dans la même fenêtre.  

    ![Capture d’écran sur laquelle le nom de la section Condition1 est encadré en rouge.](./media/configure-rules-and-alerts-in-azure-farmbeats/new-rule-condition-1.png)

5. Sous **Conditions** , entrez des valeurs pour **Measure** , **Operator** et **Value** .
6. Tapez le nom de la mesure dans le menu déroulant **Measure** .
7. Sélectionnez **+Add Condition** pour ajouter des conditions à la règle.
8. Sélectionnez le niveau de gravité sous **Severity level** .
9. Sous **Action** , activez le bouton bascule **Email enabled** pour activer les alertes par e-mail.

    ![Capture d’écran sur laquelle les boutons Ajouter une condition et Appliquer sont encadrés en rouge.](./media/configure-rules-and-alerts-in-azure-farmbeats/new-rule-email-1.png)

10. Entrez les adresses e-mail vers lesquelles vous souhaitez envoyer l’alerte sous **Email addresses** et renseignez les champs **Email Subject** et **Additional Notes** .  
11. Sous **Rule Status** , activez ou désactivez la règle à l’aide du bouton bascule **Enabled** .
    Vous pouvez voir le nombre d’appareils qui seront affectés par la règle.
12. Sélectionnez **Apply** pour créer la règle.

## <a name="view-rule"></a>Voir une règle

La page **Farm** présente la liste des règles disponibles. Sélectionnez le nom d’une règle sous **Rule Name** . Une fenêtre affiche les informations suivantes, applicables à la règle sélectionnée :
 - Nom de la règle
 - Lien vers la ferme à laquelle la règle est associée
 - Date de création
 - Date de la dernière mise à jour
 - Niveau de gravité
 - État de la règle
 - Liste de conditions  
 - Nombre d’appareils affectés par la règle

    ![Capture d’écran sur laquelle le titre Détails de la règle est encadré en rouge.](./media/configure-rules-and-alerts-in-azure-farmbeats/view-rule-1.png)

## <a name="edit-rule"></a>Modifier une règle

Pour modifier une règle, effectuez les étapes suivantes :

1. Dans la page d’accueil, sélectionnez **Rules** dans le menu de navigation de gauche.
   La fenêtre Rules s’affiche.
2. Sélectionnez la règle que vous souhaitez modifier.

    ![Capture d’écran sur laquelle le titre Règles et le bouton Modifier sont encadrés en rouge.](./media/configure-rules-and-alerts-in-azure-farmbeats/edit-rule-action-bar-1.png)

3. Sélectionnez **Edit** dans la barre d’action. La fenêtre **Edit Rule** s’affiche.

    ![Capture d’écran sur laquelle le titre Modifier la règle est encadré en rouge.](./media/configure-rules-and-alerts-in-azure-farmbeats/edit-rule-one-1.png)

4. Modifiez les champs **Rule Name** et **Rule Description** , puis sélectionnez une ferme dans le menu déroulant **Select Farm** .
5. Tapez le nom de votre ferme pour la sélectionner. La section **Conditions** s’affiche dans la même fenêtre.  
6. Sous **Conditions** , modifiez les champs **Measure** , **Operator** et **Value** .
7. Tapez le nom de la mesure dans le menu déroulant **Measure** .
8. Sélectionnez **+Add Condition** pour ajouter des conditions aux règles ou pour en modifier.

    ![Capture d’écran sur laquelle le bouton Ajouter une condition, le bouton bascule Activé et le bouton Appliquer sont encadrés en rouge.](./media/configure-rules-and-alerts-in-azure-farmbeats/edit-rule-two-1.png)

9.  Sélectionnez le niveau de gravité sous **Severity level** .  
10. Sous **Action** , activez le bouton bascule **Email enabled** pour activer les alertes par e-mail.
11. Modifiez les adresses e-mail vers lesquelles vous souhaitez envoyer l’alerte sous **Email addresses** et renseignez les champs **Email Subject** et **Additional Notes** .  
12. Sous **Rule Status** , activez ou désactivez la règle à l’aide du bouton bascule **Enabled** .
Vous pouvez voir le nombre d’appareils qui seront affectés par cette règle.
13. Sélectionnez **Apply** pour modifier la règle.

## <a name="change-rule-status"></a>Modifier l’état d’une règle

Pour modifier l’état d’une règle, effectuez les étapes suivantes :

1. Dans la page d’accueil, sélectionnez **Rules** dans le menu de navigation de gauche. La fenêtre Rules s’affiche.
2. Sélectionnez la règle dont vous souhaitez modifier l’état.

    ![Capture d’écran sur laquelle le titre Règles et le bouton Modifier l’état sont encadrés en rouge.](./media/configure-rules-and-alerts-in-azure-farmbeats/change-status-rule-action-bar-1.png)

3. Sélectionnez **Change Status** dans la barre d’action. La fenêtre **Change Status** s’affiche.

    ![Capture d’écran sur laquelle le titre Modifier l’état et le bouton Appliquer sont encadrés en rouge.](./media/configure-rules-and-alerts-in-azure-farmbeats/rule-change-status-1.png)

3. Modifiez l’état de la règle à l’aide du bouton bascule **Change Status** .
   Vous pouvez voir le nombre d’appareils qui seront affectés par la règle.
4. Sélectionnez **Apply** pour modifier l’état de la règle.

## <a name="delete-rule"></a>Supprimer une règle

Pour supprimer une règle, effectuez les étapes suivantes :

1. Dans la page d’accueil, sélectionnez **Rules** dans le menu de navigation de gauche. La fenêtre Rules s’affiche.
2. Sélectionnez la règle que vous souhaitez supprimer.

    ![Capture d’écran sur laquelle le titre Règles et le bouton Supprimer sont encadrés en rouge.](./media/configure-rules-and-alerts-in-azure-farmbeats/delete-rule-action-bar-1.png)

3. Sélectionnez **Delete** dans la barre d’action.

    ![Projet FarmBeats](./media/configure-rules-and-alerts-in-azure-farmbeats/delete-rule-1.png)

4. La boîte de dialogue **Delete Rule** s’affiche. Sélectionnez **Supprimer** .
