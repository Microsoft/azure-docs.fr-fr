---
title: Résolution des problèmes liés aux alertes et notifications Azure Monitor
description: Problèmes courants liés aux alertes Azure Monitor et solutions possibles.
author: ofirmanor
ms.author: ofmanor
ms.topic: reference
ms.date: 03/16/2020
ms.subservice: alerts
ms.openlocfilehash: d86c6d8b67546aede86a3fc3c905f6feb40d92db
ms.sourcegitcommit: dc68a2c11bae2e9d57310d39fbed76628233fd7f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2020
ms.locfileid: "91403413"
---
# <a name="troubleshooting-problems-in-azure-monitor-alerts"></a>Résolution des problèmes relatifs aux alertes Azure Monitor

Cet article décrit les problèmes courants liés aux alertes et notifications Azure Monitor.

Azure Monitor vous avertit de façon proactive lorsque des conditions significatives sont détectées dans vos données de surveillance. Elles permettent d’identifier et de résoudre les problèmes avant que les utilisateurs de votre système ne les remarquent. Pour plus d’informations sur les alertes, consultez [Vue d’ensemble des alertes dans Microsoft Azure](alerts-overview.md).

Si vous rencontrez un problème lors du déclenchement d’une alerte ou si vous ne parvenez pas à déclencher une alerte, consultez les articles ci-dessous. Vous pouvez consulter les alertes déclenchées dans le portail Azure.

- [Résolution des alertes de métrique Azure Monitor dans Microsoft Azure](alerts-troubleshoot-metric.md)  
- [Résolution des alertes de journal Azure Monitor dans Microsoft Azure](alerts-troubleshoot-metric.md)

Si l’alerte se déclenche comme prévu en fonction du portail Azure, mais que les notifications qui conviennent n’interviennent pas, utilisez les informations contenues dans la suite de cet article pour y remédier.

## <a name="action-or-notification-on-my-alert-did-not-work-as-expected"></a>L’action ou la notification sur mon alerte n’a pas fonctionné comme prévu

Si vous pouvez voir une alerte déclenchée dans le portail Azure, mais que vous rencontrez un problème avec certaines de ses actions ou notifications, consultez les sections suivantes.

## <a name="did-not-receive-expected-email"></a>L’e-mail attendu n’a pas été reçu

Si vous pouvez voir une alerte déclenchée dans le portail Azure, mais que vous n’avez pas reçu l’e-mail que vous avez configuré, effectuez les vérifications suivantes :

1. **L’e-mail a-t-il été supprimé par une [règle d’action](alerts-action-rules.md)** ?

    Vérifiez en cliquant sur l’alerte déclenchée dans le portail, puis examinez l’onglet Historique à la recherche des [groupes d’actions](action-groups.md) supprimés :

    ![Historique de suppression des règles d’action d’alerte](media/alerts-troubleshoot/history-action-rule.png)

1. **Le type d’action est-il « Envoyer un e-mail au rôle Azure Resource Manager » ?**

    Cette action examine uniquement les attributions de rôle Azure Resource Manager qui se trouvent dans l’étendue de l’abonnement et de type *Utilisateur*.  Assurez-vous que vous avez attribué le rôle au niveau de l’abonnement, et non au niveau de la ressource ou du groupe de ressources.

1. **Votre serveur de messagerie et votre boîte aux lettres acceptent-ils les e-mails externes ?**

    Vérifiez que les e-mails de ces trois adresses ne sont pas bloqués :
      - azure-noreply@microsoft.com  
      - azureemail-noreply@microsoft.com
      - alerts-noreply@mail.windowsazure.com

    Il est courant que les listes de distribution internes ou les listes de diffusion bloquent les e-mails des adresses e-mail externes. Vous devez autoriser le courrier à partir des adresses e-mail ci-dessus.  
    À des fins de test, ajoutez une adresse de messagerie professionnelle ordinaire (pas une liste de diffusion) au groupe d’actions et vérifiez si les alertes arrivent à cet e-mail.

1. **L’e-mail a-t-il été traité par des règles de boîte de réception ou un filtre antispam ?**

    Vérifiez qu’il n’existe pas de règles de boîte de réception qui suppriment ces messages ou les déplacent dans un autre dossier. Par exemple, les règles de boîte de réception peuvent intercepter des expéditeurs spécifiques ou des mots spécifiques dans l’objet.

    Vérifiez aussi :

   - les paramètres de courrier indésirable de votre client de messagerie (comme Outlook ou Gmail)
      - les limites sur l’expéditeur/paramètres de courrier indésirable/paramètres de mise en quarantaine de votre serveur de messagerie (comme Exchange, Microsoft 365 ou G-suite)
      - les paramètres de votre appliance de sécurité de messagerie, le cas échéant (par exemple, Barracuda ou Cisco).

1. **Avez-vous été désabonné par erreur du groupe d’actions ?**

    Les e-mails d’alerte fournissent un lien pour se désabonner du groupe d’actions. Pour vérifier si vous avez accidentellement annulé votre abonnement à ce groupe d’actions, vous pouvez :

    1. Ouvrir le groupe d’actions dans le portail et vérifier la colonne État :

    ![Colonne État du groupe d’actions](media/alerts-troubleshoot/action-group-status.png)

    2. Rechercher dans votre e-mail la confirmation de résiliation d’abonnement :

    ![Désabonné du groupe d’actions d’alerte](media/alerts-troubleshoot/unsubscribe-action-group.png)

    Pour vous abonner à nouveau : utilisez le lien figurant dans le message de confirmation de résiliation d’abonnement que vous avez reçu ou supprimez l’adresse de messagerie du groupe d’actions, puis rajoutez-la. 
 
1. **Avez-vous subi une restriction en raison d’un grand nombre d’e-mails allant à une seule adresse e-mail ?**

    L’adresse de messagerie est soumise à une [restriction](alerts-rate-limiting.md) de 100 e-mails maximum toutes les heures pour chaque adresse e-mail. Si vous dépassez ce seuil, les notifications par e-mail en trop sont supprimées.  Vérifiez si vous avez reçu un message indiquant que votre adresse e-mail a été temporairement limitée : 
 
   ![Taux d’envoi limité](media/alerts-troubleshoot/email-paused.png)

   Si vous souhaitez recevoir un grand nombre de notifications sans limitation du taux, envisagez d’utiliser une autre action, comme un webhook, une application logique, une fonction Azure ou un runbook Automation : aucune de ces solutions n’est limitée en volume. 

## <a name="did-not-receive-expected-sms-voice-call-or-push-notification"></a>Impossible de recevoir le SMS, l’appel vocal ou la notification Push

Si vous pouvez voir une alerte déclenchée dans le portail, mais que vous n’avez pas reçu le SMS, l’appel vocal ou la notification Push, effectuez les vérifications suivantes : 

1. **L’action a-t-elle été supprimée par une [règle d’action ](alerts-action-rules.md)?**

    Vérifiez en cliquant sur l’alerte déclenchée dans le portail, puis examinez l’onglet Historique à la recherche des [groupes d’actions](action-groups.md) supprimés : 

    ![Historique de suppression des règles d’action d’alerte](media/alerts-troubleshoot/history-action-rule.png)

   Si cela était involontaire, vous pouvez modifier, désactiver ou supprimer la règle d’action.
 
1. **SMS/voix :  Votre numéro de téléphone est-il correct ?**

   Vérifiez l’action SMS pour rechercher des fautes de frappe dans l’indicatif du pays ou le numéro de téléphone.
 
1. **SMS/voix : votre débit est-il limité ?**

    Les appels SMS et vocaux sont limités à une seule notification toutes les cinq minutes par numéro de téléphone. Si vous dépassez ce seuil, les notifications sont abandonnées.

      - Appel vocal : vérifiez l’historique des appels et regardez si vous avez un appel différent d’Azure au cours des cinq minutes précédentes.
      - SMS : vérifiez dans votre historique SMS s’ils contiennent un message indiquant que votre numéro de téléphone fait l’objet d’une restriction.

    Si vous souhaitez recevoir un grand nombre de notifications sans limitation du taux, envisagez d’utiliser une autre action, comme un webhook, une application logique, une fonction Azure ou un runbook Automation : aucune de ces solutions n’est limitée en volume. 
 
1. **SMS : Avez-vous été désabonné par erreur du groupe d’actions ?**

    Ouvrez votre historique de SMS et vérifiez si vous avez refusé la réception de SMS de ce groupe d’actions spécifique (avec la réponse DISABLE nom_court_groupe_actions) ou de tous les groupes d’actions (avec la réponse STOP). Pour vous abonner à nouveau, envoyez la commande SMS appropriée (ENABLE ou START nom_court_groupe_actions) ou supprimez l’action SMS du groupe d’actions, puis rajoutez-la à nouveau.  Pour plus d’informations, consultez [Comportement des alertes SMS dans les groupes d’actions](alerts-sms-behavior.md).

1. **Avez-vous bloqué accidentellement les notifications sur votre téléphone ?**

   La plupart des téléphones mobiles vous permettent de bloquer les appels ou SMS de numéros de téléphone ou de codes courts spécifiques, ou de bloquer les notifications Push émanant d’applications spécifiques (telles que les applications mobiles Azure). Pour vérifier si vous avez bloqué accidentellement les notifications sur votre téléphone, recherchez la documentation spécifique à votre système d’exploitation et votre modèle de téléphone, ou testez un autre appareil et un autre numéro de téléphone.

## <a name="expected-another-type-of-action-to-trigger-but-it-did-not"></a>Un autre type d’action à déclencher était attendu, mais cela ne s’est pas produit 
   
Si vous pouvez voir une alerte déclenchée dans le portail, mais que son action configurée ne s’est pas déclenchée, procédez comme suit :

1. **L’action a-t-elle été supprimée par une règle d’action ?**

    Vérifiez en cliquant sur l’alerte déclenchée dans le portail, puis examinez l’onglet Historique à la recherche des [groupes d’actions](action-groups.md) supprimés :

    ![Historique de suppression des règles d’action d’alerte](media/alerts-troubleshoot/history-action-rule.png)
 
    Si cela était involontaire, vous pouvez modifier, désactiver ou supprimer la règle d’action.

1. **Un webhook a-t-il été déclenché ?**

    1. **Les adresses IP source ont-elles été bloquées ?**
    
       Ajoutez les [adresses IP](action-groups.md#action-specific-information) à partir desquelles le webhook sera appelé à votre liste d’adresses IP autorisées.

    1. **Votre point de terminaison webhook fonctionne-t-il correctement ?**

       Vérifiez si le point de terminaison du webhook que vous avez configuré est correct et si le point de terminaison fonctionne correctement. Vérifiez vos journaux de webhook ou instrumentez son code pour pouvoir l’examiner (par exemple, enregistrez la charge utile entrante).

    1. **Appelez-vous Slack ou Microsoft Teams ?**  
    Chacun de ces points de terminaison attend un format JSON spécifique. Suivez [ces instructions](action-groups-logic-app.md) pour configurer une action d’application logique à la place.

    1. **Votre webhook a-t-il cessé de répondre ou a-t-il renvoyé des erreurs ?** 

        Notre délai d’expiration d’une réponse de webhook est de 10 secondes. L’appel de webhook sera retenté deux fois au maximum lorsque les codes d’état HTTP suivants sont retournés : 408, 429, 503, 504 ou lorsque le point de terminaison HTTP ne répond pas. La première nouvelle tentative se produit après 10 secondes. La deuxième et la dernière nouvelles tentatives se produisent après 100 secondes. Si la seconde tentative échoue, le point de terminaison ne sera pas rappelé pendant 30 minutes pour tous les groupes d’actions.

## <a name="action-or-notification-happened-more-than-once"></a>Une action ou une notification s’est produite plusieurs fois 

Si vous avez reçu une notification pour une alerte (par exemple, un e-mail ou un SMS) plusieurs fois, ou si l’action de l’alerte (par exemple, webhook ou Azure Function) a été déclenchée plusieurs fois, procédez comme suit : 

1. **S’agit-il vraiment de la même alerte ?** 

    Dans certains cas, plusieurs alertes similaires sont déclenchées à peu près en même temps. Par conséquent, vous pourriez avoir l’impression que la même alerte déclenche plusieurs fois ses actions. Par exemple, une règle d’alerte de journal d’activité peut être configurée pour se déclencher quand un événement a démarré et lorsqu’il est terminé (réussi ou échoué) en ne filtrant pas sur le champ État de l’événement. 

    Pour vérifier si ces actions ou notifications proviennent de différentes alertes, examinez les détails de l’alerte, comme son horodatage et l’ID d’alerte ou son ID de corrélation. Vous pouvez également consulter la liste des alertes déclenchées dans le portail. Si tel est le cas, vous devez adapter la logique de la règle d’alerte ou configurer autrement la source de l’alerte. 

1. **L’action se répète-t-elle dans plusieurs groupes d’actions ?** 

    Lorsqu’une alerte est déclenchée, chacun de ses groupes d’actions est traité indépendamment. Ainsi, si une action (sur une adresse e-mail, par exemple) apparaît dans plusieurs groupes d’actions déclenchés, elle est appelée une fois par groupe d’actions. 

    Pour vérifier les groupes d’actions qui ont été déclenchés, sélectionnez l’onglet Historique des alertes. Vous voyez que les deux groupes d’action sont définis dans la règle d’alerte et que les groupes d’actions ont été ajoutés aux règles d’alerte par action : 

    ![Action répétée dans plusieurs groupes d’actions](media/alerts-troubleshoot/action-repeated-multi-action-groups.png)

## <a name="action-or-notification-has-an-unexpected-content"></a>Le contenu de l’action ou de la notification est inattendu

Si vous avez reçu l’alerte, mais pensez que certains de ses champs sont manquants ou incorrects, procédez comme suit : 

1. **Avez-vous choisi le bon format pour l’action ?** 

    Chaque type d’action (e-mail, webhook, etc.) a deux formats : le format par défaut (hérité) et le [format de schéma commun, plus récent](alerts-common-schema.md). Lorsque vous créez un groupe d’actions, vous spécifiez le format souhaité pour chaque action : des actions différentes dans les groupes d’actions peuvent avoir des formats différents. 

    Par exemple, pour l’action de webhook : 

    ![Option de schéma d’action de webhook](media/alerts-troubleshoot/webhook.png)

    Vérifiez si le format spécifié au niveau de l’action correspond à ce que vous attendez. Par exemple, vous avez peut-être développé du code qui répond aux alertes (webhook, fonction, application logique, etc.), qui attendait un format, mais plus tard dans l’action, vous ou une autre personne avez spécifié un format différent.  

    Vérifiez également le format de charge utile (JSON) pour [les alertes de journal d’activité](activity-log-alerts-webhook.md), pour [les alertes de recherche de journal](alerts-log-webhook.md) (à la fois pour Application Insights et Log Analytics), pour [les alertes de métriques](alerts-metric-near-real-time.md#payload-schema), pour [le schéma d’alerte commun](alerts-common-schema-definitions.md), et pour les [alertes de métriques classiques](alerts-webhooks.md) dépréciées.

 
1. **Alertes de journal d’activité : Les informations sont-elles disponibles dans le journal d’activité ?** 

    [Les alertes du journal d’activité](activity-log-alerts.md) sont des alertes basées sur des événements écrits dans le journal des activités Azure, par exemple des événements sur la création, la mise à jour ou la suppression de ressources Azure, les événements d’intégrité du service et d’intégrité des ressources, ou les résultats d’Azure Advisor et Azure Policy. Si vous avez reçu une alerte basée sur le journal d’activité, mais que certains champs dont vous avez besoin sont manquants ou incorrects, vérifiez d’abord les événements dans le journal d’activité lui-même. Si la ressource Azure n’a pas écrit les champs que vous recherchez dans son événement du journal d’activité, ces champs ne sont pas inclus dans l’alerte correspondante. 

## <a name="action-rule-is-not-working-as-expected"></a>La règle d’action ne fonctionne pas comme prévu 

Si vous pouvez voir une alerte déclenchée dans le portail, mais qu’une règle d’action associée ne fonctionnait pas comme prévu, procédez comme suit : 

1. **La règle d’action est-elle activée ?** 

    Vérifiez la colonne État de la règle d’action pour vérifier que le rôle d’action associé est activé. 

    ![Graphique](media/alerts-troubleshoot/action-rule-status.png) 

    Si elle n’est pas activée, vous pouvez l’activer en la sélectionnant et en cliquant sur Activer. 

1. **Est-ce une alerte d’intégrité du service ?** 

    Les alertes d’intégrité du service (monitor service = « Service Health ») ne sont pas affectées par les règles d’action. 

1. **La règle d’action agit-elle sur votre alerte ?** 

    Vérifiez que la règle d’action a traité votre alerte en cliquant sur l’alerte déclenchée dans le portail et en examinant l’onglet Historique.

    Voici un exemple de règle d’action supprimant tous les groupes d’actions : 
 
     ![Historique de suppression des règles d’action d’alerte](media/alerts-troubleshoot/history-action-rule.png)

    Voici un exemple de règle d’action qui ajoute un autre groupe d’actions :

    ![Action répétée dans plusieurs groupes d’actions](media/alerts-troubleshoot/action-repeated-multi-action-groups.png)
 

1. **L’étendue de la règle d’action et le filtre correspondent-ils à l’alerte déclenchée ?** 

    Si vous pensez que la règle d’action devrait avoir été déclenchée mais ne l’a pas été ou vice versa, examinez attentivement l’étendue de la règle d’action et les conditions de filtre par rapport aux propriétés de l’alerte déclenchée. 


## <a name="how-to-find-the-alert-id-of-a-fired-alert"></a>Comment trouver l’ID d’alerte d’une alerte déclenchée

Lors de l’ouverture d’un incident sur une alerte déclenchée spécifique (par exemple, si vous n’avez pas reçu sa notification par courrier électronique), vous devez fournir l’ID d’alerte. 

Pour la localiser, procédez comme suit :

1. Dans le portail Azure, accédez à la liste des alertes déclenchées et recherchez cette alerte spécifique. Vous pouvez utiliser les filtres pour vous aider à la localiser. 

1. Cliquez sur l’alerte pour ouvrir ses détails. 

1. Faites défiler les champs d’alerte du premier onglet (onglet Résumé) jusqu’à ce que vous le trouviez, puis copiez-le. Ce champ comprend également un bouton d’assistance « Copier dans le Presse-papiers » que vous pouvez utiliser.  

    ![Rechercher un ID d’alerte](media/alerts-troubleshoot/get-alert-id.png)

## <a name="problem-creating-updating-or-deleting-action-rules-in-the-azure-portal"></a>Problème lors de la création, de la mise à jour ou de la suppression de règles d’action dans le portail Azure

Si vous avez reçu une erreur lors de la tentative de création, de mise à jour ou de suppression d’une [règle d’action](alerts-action-rules.md), procédez comme suit : 

1. **Avez-vous reçu une erreur d’autorisation ?**  

    Vous devez disposer du  [rôle intégré Contributeur d’analyse](../../role-based-access-control/built-in-roles.md#monitoring-contributor) ou des autorisations spécifiques liées aux règles et aux alertes d’action.

1. **Avez-vous vérifié les paramètres de la règle d’action ?**  

    Consultez la [documentation de la règle d’action](alerts-action-rules.md) ou la commande [PowerShell Set-AzActionRule de la règle d’action](/powershell/module/az.alertsmanagement/set-azactionrule?view=azps-3.5.0). 


## <a name="next-steps"></a>Étapes suivantes
- Si vous utilisez une alerte de journal, consultez la section [Résolution des alertes de journal](./alerts-troubleshoot-log.md).
- Revenez au [portail Azure](https://portal.azure.com) pour vérifier si vous avez résolu votre problème avec les conseils ci-dessus 
