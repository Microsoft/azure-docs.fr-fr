---
title: Gérer les stratégies d’arrêt automatique dans Azure DevTest Labs et les machines virtuelles de calcul | Microsoft Docs
description: Découvrez comment définir la stratégie d’arrêt automatique d’un labo afin que les machines virtuelles soient arrêtées automatiquement lorsqu’elles ne sont pas utilisées.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: cd7974580ea30c9d0591c88380a4e626711bad1e
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93318968"
---
# <a name="configure-autoshutdown-for-lab-and-compute-virtual-machines-in-azure-devtest-labs"></a>Configurer l’arrêt automatique pour les machines virtuelles lab et de calcul dans Azure DevTest Labs

Cet article explique comment configurer des paramètres d’arrêt automatique pour des machines virtuelles lab dans DevTest Labs et des machines virtuelles de calcul.

## <a name="configure-autoshutdown-for-lab-vms-devtest-labs"></a>Configurer l’arrêt automatique pour les machines virtuelles lab (DevTest Labs)

Azure DevTest Labs vous permet de contrôler les coûts et de réduire le gaspillage dans vos laboratoires en gérant les stratégies (paramètres) de chacun d’entre eux. Cet article explique comment configurer la stratégie d’arrêt automatique pour un laboratoire.  Il explique également comment configurer les paramètres d’arrêt automatique pour une machine virtuelle de laboratoire. Pour savoir comment définir chaque stratégie de laboratoire, consultez [Définir des stratégies de laboratoire dans Azure DevTest Labs](devtest-lab-set-lab-policy.md).  

### <a name="set-autoshutdown-policy-for-a-lab"></a>Définir une stratégie d’arrêt automatique pour un labo

En tant que propriétaire d’un laboratoire, vous pouvez configurer une planification d’arrêt pour toutes les machines virtuelles de votre laboratoire. Ceci vous permet de réaliser des économies en stoppant l’exécution de machines inutilisées (inactives). Vous pouvez appliquer une stratégie d’arrêt centralisée à toutes les machines virtuelles de votre laboratoire et, également, épargner aux utilisateurs de votre laboratoire l’établissement d’une planification pour leurs machines individuelles. Cette fonctionnalité vous permet de définir la stratégie de votre planification de laboratoire, allant de l’autorisation des utilisateurs de laboratoire à avoir un contrôle total sur la planification de l’arrêt de leur machine virtuelle jusqu’à l’absence de contrôle de l’arrêt de leur machine virtuelle. En tant que propriétaire de laboratoire, vous pouvez configurer cette stratégie en procédant comme suit :

1. Sur la page d’accueil de votre laboratoire, sélectionnez **Configuration et stratégies**.
2. Sélectionnez **Stratégie d'arrêt automatique** dans la section **Planifications** du menu de gauche.
3. Sélectionnez l'une des options. Les sections suivantes vous donnent plus de détails sur ces options :

    ![Options de stratégie d’arrêt automatique](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-options.png)

> [!IMPORTANT]
> Les modifications apportées à la stratégie d’arrêt s’appliquent seulement aux nouvelles machines virtuelles créées dans le labo et non pas aux machines virtuelles déjà existantes.

### <a name="configure-autoshutdown-settings"></a>Configurer les paramètres d’arrêt automatique

La stratégie d’arrêt automatique vous permet d’indiquer l’heure à laquelle les machines virtuelles du laboratoire doivent s’arrêter et contribue ainsi à réduire les pertes de laboratoire.

Pour afficher ou modifier les stratégies d’un laboratoire, procédez comme suit :

1. Sur la page d’accueil de votre laboratoire, sélectionnez **Configuration et stratégies**.
2. Sélectionnez **Arrêt automatique** dans la section **Planifications** du menu de gauche.
3. Sélectionnez **Activer** ou **Désactiver** pour activer ou désactiver cette stratégie.
     ![Détails de l’arrêt automatique](./media/devtest-lab-set-lab-policy/auto-shutdown.png)
4. Si vous activez cette stratégie, spécifiez l’heure (et le fuseau horaire) de l’arrêt pour toutes les machines virtuelles du laboratoire actif.
5. Spécifiez **Oui** ou **Non** pour l’option d’envoi de notification 30 minutes avant l’heure d’arrêt automatique spécifiée. Si vous choisissez **Oui** , saisissez un point de terminaison de l’URL de Webhook ou une adresse e-mail spécifiant où vous désirez publier ou envoyer la notification. L’utilisateur reçoit une notification et peut retarder l’arrêt. Pour en savoir plus, consultez la section [Notifications](#notifications).
6. Sélectionnez **Enregistrer**.

    Par défaut, une fois activée, cette stratégie s’applique à toutes les machines virtuelles dans le laboratoire en cours. Pour supprimer ce paramètre sur une machine virtuelle spécifique, ouvrez le volet de gestion de la machine virtuelle et modifiez son paramètre **Arrêt automatique**.

> [!NOTE]
> Si vous mettez à jour la planification de l’arrêt automatique pour votre laboratoire ou une machine virtuelle de laboratoire spécifique dans les 30 minutes de l’heure actuelle planifiée, l’heure d’arrêt mise à jour s’applique à la planification du jour suivant.

### <a name="user-sets-a-schedule-and-can-opt-out"></a>L'utilisateur définit une planification qu'il peut refuser

Si vous définissez cette stratégie pour votre laboratoire, les utilisateurs du laboratoire peuvent remplacer ou refuser la planification du laboratoire. Cette option accorde aux utilisateurs du laboratoire un contrôle total sur la planification d’arrêt automatique de leurs machines virtuelles. Les utilisateurs du laboratoire ne voient aucune modification de leur page de planification d’arrêt automatique des machine virtuelles.

![Option de stratégie d’arrêt automatique – 1](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-1.png)

### <a name="user-sets-a-schedule-and-cannot-opt-out"></a>L'utilisateur définit une planification qu'il ne peut pas refuser

Si vous définissez cette stratégie pour votre laboratoire, les utilisateurs du laboratoire peuvent remplacer la planification du laboratoire. Ils ne peuvent cependant pas renoncer à la stratégie d’arrêt automatique. Cette option permet de s’assurer que chaque machine de votre laboratoire fait l’objet d’une planification d’arrêt automatique. Les utilisateurs du laboratoire peuvent mettre à jour la planification d’arrêt automatique de leurs machines virtuelles et définir des notifications d’arrêt.

![Option de stratégie d’arrêt automatique – 2](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-2.png)

### <a name="user-has-no-control-over-the-schedule-set-by-lab-admin"></a>L'utilisateur n'a aucun contrôle sur la planification définie par l'administrateur du laboratoire

Si vous définissez cette stratégie pour votre laboratoire, les utilisateurs du laboratoire ne peuvent ni remplacer ni refuser la planification du laboratoire. Cette option offre à l’administrateur du laboratoire le contrôle total sur la planification de chaque machine du laboratoire. Les utilisateurs du laboratoire ne peuvent configurer de notifications d’arrêt automatique que pour leurs machines virtuelles.

![Option de stratégie d’arrêt automatique – 3](./media/devtest-lab-set-lab-policy/auto-shutdown-policy-option-3.png)

## <a name="configure-autoshutdown-for-compute-vms"></a>Configurer l’arrêt automatique pour des machines virtuelles de calcul

1. Sur la page **Machine virtuelle** , sélectionnez **Arrêt automatique** dans le menu de gauche de la section **Opérations**.
2. Dans la page **Arrêt automatique** , Sélectionnez **Activer** ou **Désactiver** pour activer ou désactiver cette stratégie.
3. Si vous activez cette stratégie, spécifiez l’ **heure** (et le **fuseau horaire** ) à laquelle la machine virtuelle doit être arrêtée.
4. Choisissez **Oui** ou **Non** pour l’option d’envoi de notification 30 minutes avant l’heure d’arrêt automatique spécifiée. Si vous choisissez **Oui** , saisissez un point de terminaison de l’URL de Webhook ou une adresse e-mail spécifiant où vous désirez publier ou envoyer la notification. L’utilisateur reçoit une notification et peut retarder l’arrêt. Pour en savoir plus, consultez la section [Notifications](#notifications).
5. Sélectionnez **Enregistrer**.

    ![Configurer l’arrêt automatique pour une machine virtuelle de calcul](./media/devtest-lab-auto-shutdown/comnpute-auto-shutdown.png)

### <a name="view-activity-logs-for-auto-shutdown-updates"></a>Afficher les journaux d'activité pour les mises à jour de l'arrêt automatique

Lorsque vous mettez à jour le paramètre d’arrêt automatique, vous pouvez consulter l’activité enregistrée dans le journal d’activité de la machine virtuelle.

1. Sur le [portail Azure](https://portal.azure.com), accédez à la page d'accueil de votre machine virtuelle.
2. Sélectionnez **Journal d'activité** dans le menu de gauche.
3. Supprimez **Ressource : mycomputevm** des filtres.
4. Vérifiez que l'opération **Ajouter ou modifier des planifications** figure dans le journal d'activité. Si elle n'y figure pas, attendez un peu et actualisez le journal d'activité.

    ![Entrée du journal d’activité](./media/devtest-lab-auto-shutdown/activity-log-entry.png)
5. Sélectionnez l'opération **Ajouter ou modifier des planifications** pour consulter les informations suivantes sur la page **Résumé**  :

    - Nom de l'opération (Ajouter ou modifier des planifications)
    - Date et heure auxquelles le paramètre d'arrêt automatique a été mis à jour.
    - Adresse e-mail de l'utilisateur qui a mis à jour le paramètre.

        ![Résumé des entrées du journal d'activité](./media/devtest-lab-auto-shutdown/activity-log-entry-summary.png)
6. Accédez à l'onglet **Historique des modifications** de la page **Ajouter ou modifier des planifications** pour consulter l'historique des modifications du paramètre. Dans l'exemple suivant, l'heure d'arrêt a été modifiée en remplaçant 19:00 par 18:00 le 10 avril 2020 à 15:18:47 EST. Et le paramètre a été désactivé à 15:25:09 EST.

    ![Journal d'activité - Historique des modifications](./media/devtest-lab-auto-shutdown/activity-log-entry-change-history.png)
7. Pour en savoir plus sur l'opération, accédez à l'onglet **JSON** de la page **Ajouter ou modifier des planifications**.

## <a name="notifications"></a>Notifications

Une fois l’arrêt automatique configuré, des notifications seront envoyées aux utilisateurs du laboratoire 30 minutes avant le déclenchement de l’arrêt automatique si l’une de leurs machines virtuelles est concernée. Cette option donne aux utilisateurs du laboratoire la possibilité d’enregistrer leur travail avant l’arrêt. La notification fournit également des liens pour chaque machine virtuelle pour les actions suivantes au cas où quelqu’un aurait besoin de continuer à travailler sur sa machine virtuelle.

- Pour l’instant, ignorez l’arrêt automatique
- Répéter l’arrêt automatique pendant une heure
- Répéter l’arrêt automatique pendant 2 heures

La notification est envoyée à l’URL du webhook si un webhook a été spécifié.  Si une adresse e-mail a été spécifiée dans les paramètres d’arrêt automatique, un e-mail est envoyé à cette adresse e-mail. Les webhooks vous permettent de créer ou de configurer des intégrations qui s’abonnent à certains événements. Lorsque l’un de ces événements se déclenche, DevTest Labs envoie une charge utile POST HTTP à l’URL configurée du webhook. Pour plus d’informations sur la réponse aux webhooks, consultez [Vue d’ensemble des déclencheurs et des liaisons HTTP Azure Functions](../azure-functions/functions-bindings-http-webhook.md) ou [Ajout d’un déclencheur HTTP pour Azure Logic Apps](../connectors/connectors-native-http.md#add-an-http-trigger).

Nous vous recommandons d’utiliser des webhooks, car ils sont largement pris en charge par diverses applications, telles que Azure Logic Apps et Slack.  Les webhooks vous permettent d’implémenter votre propre méthode d’envoi de notifications. À titre d’exemple, cet article vous explique comment configurer une notification d’arrêt automatique en vue d’envoyer un e-mail au propriétaire de machine virtuelle en utilisant les applications Azure Logic Apps. Tout d’abord, passons rapidement en revue les étapes de base pour activer la notification d’arrêt automatique dans votre laboratoire.

### <a name="create-a-logic-app-that-receives-email-notifications"></a>Créer une application logique qui reçoit des notifications par e-mail

[Azure Logic Apps](../logic-apps/logic-apps-overview.md) fournit de nombreux connecteurs qui facilitent l’intégration d’un service dans d’autres clients, comme Office 365 et Twitter. Au niveau supérieur, les étapes de configuration d’une application Logic App pour la notification par e-mail peuvent être divisées en quatre phases :

- Créez une application logique.
- Configurez le modèle intégré.
- Intégration avec votre client de courrier
- Obtenez l’URL du webhook.

### <a name="create-a-logic-app"></a>Créer une application logique

Pour commencer, créez une application logique dans votre abonnement Azure en procédant comme suit :

1. Sélectionnez **+ Créer une ressource** dans le menu de gauche, puis sélectionnez **Intégration** et **Application logique**.

    ![Menu Nouvelle application logique](./media/devtest-lab-auto-shutdown/new-logic-app.png)
2. Dans la page **Application logique - Créer** , procédez comme suit :
    1. Entrez le **nom** de l’application logique.
    2. Sélectionnez votre **abonnement** Azure.
    3. Créez un **groupe de ressources** ou sélectionnez-en un.
    4. Sélectionnez l’ **emplacement** de l’application logique.

        ![Nouvelle application logique - paramètres](./media/devtest-lab-auto-shutdown/new-logic-app-page.png)
3. Dans **Notifications** , sélectionnez **Accéder à la ressource** sur la notification.

    ![Accéder à la ressource](./media/devtest-lab-auto-shutdown/go-to-resource.png)
4. Sélectionnez **Concepteur d’application logique** dans la catégorie **Outils de déploiement**.

    ![Sélectionner la requête/réponse HTTP](./media/devtest-lab-auto-shutdown/select-http-request-response-option.png)
5. Dans la page **Demande-réponse HTTP** , sélectionnez **Utiliser ce modèle**.

    ![Sélectionner l’option Utiliser ce modèle](./media/devtest-lab-auto-shutdown/select-use-this-template.png)
6. Copiez le code JSON suivant dans la section **Schéma JSON du corps de la demande**  :

    ```json
    {
        "$schema": "http://json-schema.org/draft-04/schema#",
        "properties": {
            "delayUrl120": {
                "type": "string"
            },
            "delayUrl60": {
                "type": "string"
            },
            "eventType": {
                "type": "string"
            },
            "guid": {
                "type": "string"
            },
            "labName": {
                "type": "string"
            },
            "owner": {
                "type": "string"
            },
            "resourceGroupName": {
                "type": "string"
            },
            "skipUrl": {
                "type": "string"
            },
            "subscriptionId": {
                "type": "string"
            },
            "text": {
                "type": "string"
            },
            "vmName": {
                "type": "string"
            },
            "vmUrl": {
                "type": "string"
            },
            "minutesUntilShutdown": {
                "type": "string"
            }
        },
        "required": [
            "skipUrl",
            "delayUrl60",
            "delayUrl120",
            "vmName",
            "guid",
            "owner",
            "eventType",
            "text",
            "subscriptionId",
            "resourceGroupName",
            "labName",
            "vmUrl",
            "minutesUntilShutdown"
        ],
        "type": "object"
    }
    ```

    ![Capture d’écran montrant le « Schéma JSON du corps de la demande ».](./media/devtest-lab-auto-shutdown/request-json.png)
7. Sélectionnez **+ Nouvelle étape** dans le concepteur et procédez comme suit :
    1. Recherchez **Office 365 Outlook - Envoyer un courrier**.
    2. Sélectionnez **Envoyer un courrier** dans **Actions**.

        ![Option Envoyer un courrier](./media/devtest-lab-auto-shutdown/select-send-email.png)
    3. Sélectionnez **Se connecter** pour vous connecter à votre compte de courrier.
    4. Sélectionnez le champ **TO** (À), puis choisissez propriétaire.
    5. Sélectionnez **SUBJECT** (OBJET), et saisissez l’objet de la notification par e-mail. Par exemple : « Arrêt de la machine vmName du labo labName. »
    6. Sélectionnez **BODY** (CORPS), puis définissez le contenu du corps de la notification. Par exemple : « L’arrêt de vmName est prévu dans 15 minutes. Pour ignorer cet arrêt, cliquez sur : URL. Arrêt différé d’une heure : delayUrl60. Arrêt différé de deux heures : delayUrl120. »

        ![Schéma JSON du corps de la demande](./media/devtest-lab-auto-shutdown/email-options.png)
8. Sélectionnez **Enregistrer** dans la barre d’outils. Vous pouvez alors copiez l’ **URL HTTP POST**. Sélectionnez le bouton Copier pour copier l’URL dans le Presse-papiers.

    ![URL du webhook](./media/devtest-lab-auto-shutdown/webhook-url.png)

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir comment définir toutes les stratégies, consultez [Définir des stratégies de laboratoire dans Azure DevTest Labs](devtest-lab-set-lab-policy.md).
