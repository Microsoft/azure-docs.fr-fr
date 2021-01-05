---
title: Comment créer des alertes pour la fonctionnalité Suivi des modifications et inventaire d’Azure Automation
description: Cet article explique comment configurer des alertes Azure pour notifier l’état des modifications détectées par le Suivi des modifications et inventaire .
services: automation
ms.subservice: change-inventory-management
ms.date: 10/15/2020
ms.topic: conceptual
ms.openlocfilehash: 275e57e5dcf173e8d5f30f262641b02698910422
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92209060"
---
# <a name="how-to-create-alerts-for-change-tracking-and-inventory"></a>Comment créer des alertes pour le Suivi des modifications et inventaire

Les alertes dans Azure vous informent de façon proactive des résultats des travaux de runbook, des problèmes d’intégrité de service ou d’autres scénarios liés à votre compte Automation. Azure Automation n’inclut pas de règles d’alerte préconfigurées, mais vous pouvez créer vos propres règles en fonction des données générées. Cet article fournit des conseils sur la création de règles d’alerte basées sur les modifications identifiées par le Suivi des modifications et inventaire.

Avant de commencer, si vous n’êtes pas familiarisé avec les alertes d’Azure Monitor, consultez [Vue d’ensemble des alertes dans Microsoft Azure](../../azure-monitor/platform/alerts-overview.md). Pour en savoir plus sur les alertes utilisant des requêtes de journal, consultez [Alertes de journal dans Azure Monitor](../../azure-monitor/platform/alerts-unified-log.md).

## <a name="create-alert"></a>Créer une alerte

L’exemple suivant montre que le fichier **c:\windows\system32\drivers\etc\hosts** a été modifié sur une machine. Ce fichier est important car Windows l’utilise pour résoudre des noms d’hôtes en adresses IP. Cette opération est prioritaire par rapport à DNS et peut provoquer des problèmes de connectivité. Elle peut également entraîner une redirection du trafic vers des sites web malveillants ou dangereux.

![Graphique montrant la modification du fichier hosts](./media/configure-alerts/changes.png)

Utilisons cet exemple pour aborder les étapes de création d’alertes en cas de modification.

1. Sur la page **Suivi des modifications** à partir de votre compte Automation, sélectionnez **Log Analytics** .

2. Dans Recherche dans les journaux, recherchez les modifications apportées au contenu du fichier **hosts** à l’aide de la requête `ConfigurationChange | where FieldsChanged contains "FileContentChecksum" and FileSystemPath contains "hosts"`. Cette requête recherche des modifications de contenu pour les fichiers avec des noms de chemin complet contenant le mot `hosts`. Vous pouvez aussi rechercher un fichier spécifique en remplaçant le chemin par sa forme complète (par exemple `FileSystemPath == "c:\windows\system32\drivers\etc\hosts"`).

3. Une fois que la requête a renvoyé ses résultats, sélectionnez **Nouvelle règle d’alerte** dans la recherche dans les journaux pour ouvrir la page **Création d’alerte** . Vous pouvez également accéder à cette page par le biais d’ **Azure Monitor** dans le portail Azure.

4. Vérifiez à nouveau votre requête et modifiez la logique d’alerte. Pour cet exemple, vous souhaitez que l'alerte soit déclenchée à la moindre modification détectée sur l'ensemble des machines de l'environnement.

    ![Modification de la requête pour le suivi des modifications apportées au fichier hosts](./media/configure-alerts/change-query.png)

5. Une fois la logique d’alerte définie, attribuez des groupes d’actions pour effectuer des actions en réponse à l’alerte déclenchée. Ici, nous allons configurer des e-mails à envoyer et un ticket ITSM (IT Service Management) à créer.

Effectuez les étapes ci-dessous pour configurer des alertes afin de vous informer de l’état d’un déploiement de mises à jour. Si vous débutez avec les alertes Azure, consultez [Vue d’ensemble des alertes Azure](../../azure-monitor/platform/alerts-overview.md).

## <a name="configure-action-groups-for-your-alerts"></a>Configurer des groupes d’actions pour vos alertes

Une fois vos alertes configurées, vous pouvez configurer un groupe d’actions à exécuter quand plusieurs alertes sont déclenchées. Les actions peuvent inclure les notifications par e-mail, les runbooks, les webhooks et bien plus encore. Pour en savoir plus sur les groupes d’actions, consultez [Créer et gérer des groupes d’actions](../../azure-monitor/platform/action-groups.md).

1. Sélectionnez une alerte, puis sélectionnez **Créer** sous **Groupes d’actions** .

2. Entrez un nom complet et un nom abrégé pour le groupe d’actions. Update Management utilise le nom abrégé lors de l’envoi de notifications pour le groupe spécifié.

3. Sous **Actions** , entrez un nom qui spécifie l’action, par exemple **Notification par e-mail** .

4. Pour **Type d’action** , sélectionnez le type approprié, par exemple **E-mail/SMS/Push/Voix** .

5. Sélectionnez **Modifier les détails** .

6. Renseignez le volet correspondant à votre type d’action. Par exemple, si vous utilisez **E-mail/SMS/Push/Voix** , entrez un nom d’action, cochez la case **E-mail** , entrez une adresse e-mail valide, puis sélectionnez **OK** .

    ![Configurer l’e-mail du groupe d’actions](./media/configure-alerts/configure-email-action-group.png)

7. Dans le volet Ajouter un groupe d’actions, sélectionnez **OK** .

8. Pour un e-mail d’alerte, vous pouvez personnaliser l’objet de l’e-mail. Sélectionnez **Personnaliser les actions** sous **Créer une règle** , puis sélectionnez **Objet de l’e-mail** .

9. Lorsque vous avez terminé, cliquez sur **Créer une règle d’alerte** .

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [alertes dans Azure Monitor](../../azure-monitor/platform/alerts-overview.md).

* Découvrez les [requêtes de journal](../../azure-monitor/log-query/log-query-overview.md) pour récupérer et analyser les données d’un espace de travail Log Analytics.

* Gérer [l’utilisation et les coûts à l’aide des Journaux de Azure Monitor](../../azure-monitor/platform/manage-cost-storage.md) décrit comment contrôler vos coûts en modifiant la période de conservation des données, et comment analyser et alerter sur l’utilisation de vos données.