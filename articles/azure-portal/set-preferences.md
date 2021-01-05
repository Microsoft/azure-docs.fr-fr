---
title: Gérer les paramètres et les préférences du portail Azure
description: Vous pouvez changer les paramètres par défaut du portail Azure pour appliquer vos propres préférences. Les paramètres incluent le délai d’expiration des sessions inactives, la vue par défaut, le mode de menu, le contraste, le thème, les notifications et les formats de langue et de région
keywords: paramètres, délai d’expiration, langue, région
ms.date: 08/05/2020
ms.topic: how-to
ms.openlocfilehash: 07dd3eb5cb132adf87852f29d867be772294d189
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745636"
---
# <a name="manage-azure-portal-settings-and-preferences"></a>Gérer les paramètres et les préférences du portail Azure

Vous pouvez changer les paramètres par défaut du portail Azure pour appliquer vos propres préférences. La plupart des paramètres sont disponibles dans le menu **Paramètres** de l’en-tête de page global.

![Capture d’écran montrant les icônes de l’en-tête de page global avec mise en évidence des paramètres](./media/set-preferences/header-settings.png)


## <a name="choose-your-default-subscription"></a>Choisissez votre abonnement par défaut

Vous pouvez modifier l’abonnement qui s’ouvre par défaut lorsque vous vous connectez au portail Azure. Cela est utile si vous avez un abonnement principal avec lequel vous travaillez, mais que vous en utilisez parfois d’autres. 

:::image type="content" source="media/set-preferences/filter-subscription-default-view.png" alt-text="Filtrez la liste des ressources par abonnement.":::

1. Sélectionnez l’icône Filtre de répertoire et d’abonnement dans l’en-tête global de la page.

1. Sélectionnez les abonnements que vous souhaitez en tant qu’abonnements par défaut lorsque vous lancez le portail. 

    :::image type="content" source="media/set-preferences/default-directory-subscription-filter.png" alt-text="Sélectionnez les abonnements que vous souhaitez en tant qu’abonnements par défaut lorsque vous lancez le portail."::: 


## <a name="choose-your-default-view"></a>Choisir votre vue par défaut 

Vous pouvez changer la page qui s’ouvre par défaut lorsque vous vous connectez au portail Azure.

![Capture d’écran montrant les paramètres du portail Azure avec mise en évidence de la vue par défaut](./media/set-preferences/default-view.png)

- La **page d’accueil** ne peut pas être personnalisée.  Elle affiche des raccourcis vers les services Azure populaires et liste les ressources que vous avez utilisées en dernier. Nous vous proposons également des liens utiles vers des ressources comme Microsoft Learn et la feuille de route Azure.

- Vous pouvez personnaliser les tableaux de bord pour créer un espace de travail conçu uniquement pour vous. Par exemple, vous pouvez créer un tableau de bord qui est axé sur un projet, une tâche ou un rôle. Si vous sélectionnez **Tableau de bord**, votre vue par défaut est celle du dernier tableau de bord utilisé. Pour plus d’informations, consultez [Créer et partager des tableaux de bord dans le portail Azure](azure-portal-dashboards.md).

## <a name="choose-a-portal-menu-mode"></a>Choisir un mode de menu pour le portail

Le mode par défaut pour le menu du portail contrôle l’espace occupé par le menu du portail dans la page.

![Capture d’écran montrant comment définir le mode par défaut pour le menu du portail.](./media/set-preferences/menu-mode.png)

- Quand le menu du portail est en mode **volant**, il est masqué jusqu’à ce que vous en ayez besoin. Sélectionnez l’icône de menu pour ouvrir ou fermer le menu.

- Si vous choisissez le mode **ancré** pour le menu du portail, il est toujours visible. Vous pouvez réduire le menu pour libérer davantage d’espace de travail.

## <a name="choose-a-theme-or-enable-high-contrast"></a>Choisir un thème ou activer le contraste élevé

Le thème que vous choisissez affecte les couleurs d’arrière-plan et de police qui s’affichent dans le portail Azure. Vous pouvez sélectionner l’un des quatre thèmes de couleur prédéfinis. Sélectionnez chaque miniature pour trouver le thème qui vous convient le mieux.

Vous pouvez aussi choisir l’un des thèmes à contraste élevé à la place. Les thèmes de contraste élevé facilitent la lecture du portail Azure pour les utilisateurs malvoyants et remplacent toutes les autres sélections de thèmes.

![Capture d’écran montrant les paramètres du portail Azure avec mise en évidence des thèmes](./media/set-preferences/theme.png)

## <a name="enable-or-disable-pop-up-notifications"></a>Activer ou désactiver les notifications contextuelles

Les notifications sont des messages système liés à votre session active. Elles fournissent des informations telles que votre solde de crédit, la disponibilité des ressources que vous venez de créer ou la confirmation de votre dernière action. Quand les notifications contextuelles sont activées, les messages s’affichent brièvement dans le coin supérieur de l’écran. 

Pour activer ou désactiver les notifications contextuelles, cochez ou décochez **Activer les notifications contextuelles**.

![Capture d’écran montrant les paramètres du portail Azure avec mise en évidence des notifications contextuelles](./media/set-preferences/popup-notifications.png)

Pour lire toutes les notifications reçues pendant votre session active, sélectionnez **Notifications** dans l’en-tête global.

![Capture d’écran montrant l’en-tête global du portail Azure avec mise en évidence des notifications](./media/set-preferences/read-notifications.png)

Si vous souhaitez lire les notifications liées à des sessions antérieures, recherchez les événements dans le journal d’activité. Pour plus d’informations, consultez [Voir le journal d’activité](../azure-monitor/platform/activity-log.md#view-the-activity-log). 

## <a name="change-the-inactivity-timeout-setting"></a>Modifier le paramètre de délai d’inactivité

Le paramètre de délai d’inactivité vous permet de protéger les ressources contre tout accès non autorisé si vous oubliez de sécuriser votre station de travail. Une fois que vous êtes resté inactif pendant un certain temps, vous êtes déconnecté automatiquement de votre session du portail Azure. En tant qu’individu, vous pouvez modifier le paramètre de délai d’attente pour vous-même. Si vous êtes un administrateur, vous pouvez le définir au niveau de l’annuaire pour tous vos utilisateurs dans le répertoire.

### <a name="change-your-individual-timeout-setting-user"></a>Modifier le paramètre de délai d’attente individuel (utilisateur)

Sélectionnez la liste déroulante sous **Me déconnecter lorsque je suis inactif**. Choisissez la durée au-delà de laquelle votre session du portail Azure est déconnectée si vous êtes inactif.

![Capture d’écran montrant les paramètres du portail avec mise en évidence des paramètres de délai d’inactivité](./media/set-preferences/inactive-signout-user.png)

La modification est enregistrée automatiquement. Si vous êtes inactif, votre session du portail Azure se déconnecte au terme de la durée que vous avez définie.

Si votre administrateur a activé une stratégie de délai d’expiration d’inactivité, vous pouvez toujours définir votre propre stratégie, à condition qu’elle soit inférieure au paramètre au niveau du répertoire. Sélectionnez **Remplacer la stratégie de délai d’inactivité du répertoire**, puis définissez un intervalle de temps.

![Capture d’écran montrant les paramètres du portail avec mise en évidence du paramètre Remplacer la stratégie de délai d’inactivité du répertoire](./media/set-preferences/inactive-signout-override.png)

### <a name="change-the-directory-timeout-setting-admin"></a>Modifier le paramètre de délai d’attente du répertoire (admin)

Les administrateurs dans le [rôle administrateur général](../active-directory/roles/permissions-reference.md#global-administrator--company-administrator) peuvent appliquer la durée d’inactivité maximale avant qu’une session soit déconnectée. Le paramètre de délai d’inactivité s’applique au niveau du répertoire. Le paramètre prend effet lors des nouvelles sessions. Il ne s’applique pas immédiatement à tous les utilisateurs qui sont déjà connectés. Pour plus d’informations sur les répertoires, voir [Présentation des services de domaine Active Directory](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview).

Si vous êtes administrateur général et que vous voulez appliquer un paramètre de délai d’inactivité pour tous les utilisateurs du portail Azure, effectuez ces étapes :

1. Sélectionnez le texte du lien **Configurer le délai d’inactivité au niveau du répertoire**.

    ![Capture d’écran montrant les paramètres du portail avec mise en évidence du texte de lien](./media/set-preferences/settings-admin.png)

1. Sur la page **Configurer le délai d’inactivité au niveau du répertoire**, sélectionnez **Activer le délai d’inactivité au niveau du répertoire pour le Portail Azure** pour activer le paramètre.

1. Ensuite, entrez les **heures** et les **minutes** correspondant à la durée maximale durant laquelle un utilisateur peut être inactif avant que sa session ne soit déconnectée automatiquement.

1. Sélectionnez **Appliquer**.

    ![Capture d’écran montrant la page permettant de définir un délai d’inactivité au niveau du répertoire](./media/set-preferences/configure.png)

Pour confirmer que la stratégie de délai d’inactivité est correctement définie, sélectionnez **Notifications** à partir de l’en-tête de page globale. Vérifiez qu’une notification de réussite est indiquée.

![Capture d’écran montrant un message de notification de réussite pour le délai d’inactivité au niveau du répertoire](./media/set-preferences/confirmation.png)

## <a name="restore-default-settings"></a>Restaurer les paramètres par défaut

Si vous avez apporté des modifications aux paramètres du portail Azure et que vous souhaitez les ignorer, sélectionnez **Restaurer les paramètres par défaut**. Toutes les modifications que vous avez apportées aux paramètres du portail seront perdues. Cette option n’affecte pas les personnalisations des tableaux de bord.

![Capture d’écran montrant la restauration des paramètres par défaut](./media/set-preferences/useful-links-restore-defaults.png)

## <a name="export-user-settings"></a>Exporter les paramètres utilisateur

Les informations sur vos paramètres personnalisés sont stockées dans Azure. Vous pouvez exporter les données utilisateur suivantes :

* Les tableaux de bord privés sur le portail Azure
* Les paramètres utilisateur, comme les abonnements ou les répertoires de favoris et le répertoire de la dernière ouverture de session
* Les thèmes et autres paramètres personnalisés du portail

Nous vous recommandons d’exporter et de vérifier vos paramètres si vous prévoyez de les supprimer. Le recréation des tableaux de bord ou des paramètres peut prendre du temps.

Pour exporter vos paramètres de portail, sélectionnez **Exporter tous les paramètres**.

![Capture d’écran montrant l’exportation des paramètres](./media/set-preferences/useful-links-export-settings.png)

Exporter les paramètres crée un fichier *.json* contenant vos paramètres utilisateur, tels que votre thème, vos favoris et vos tableaux de bord privés. En raison de la nature dynamique des paramètres utilisateur et des risques d’altération des données, vous ne pouvez pas importer de paramètres à partir du fichier *.json*.

## <a name="delete-user-settings-and-dashboards"></a>Supprimer les paramètres utilisateur et les tableaux de bord

Les informations sur vos paramètres personnalisés sont stockées dans Azure. Vous pouvez supprimer les données utilisateur suivantes :

* Les tableaux de bord privés sur le portail Azure
* Les paramètres utilisateur, comme les abonnements ou les répertoires de favoris et le répertoire de la dernière ouverture de session
* Les thèmes et autres paramètres personnalisés du portail

Nous vous recommandons d’exporter et de vérifier vos paramètres avant de les supprimer. En effet, le recréation des tableaux de bord ou des paramètres personnalisés peut prendre du temps.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

Pour supprimer vos paramètres de portail, sélectionnez **Supprimer tous les paramètres et les tableaux de bord privés**.

![Capture d’écran montrant la suppression des paramètres](./media/set-preferences/useful-links-delete-settings.png)

## <a name="change-language-and-regional-settings"></a>Changer la langue et les paramètres régionaux

Il existe deux paramètres qui contrôlent le mode d’affichage du texte du portail Azure : 
- Le paramètre **Langue** contrôle la langue que vous voyez pour le texte dans le portail Azure. 

- Le paramètre **Format de région** contrôle la manière dont les dates, les heures, les nombres et les devises sont affichés.

Pour changer la langue utilisée dans le portail Azure, utilisez la liste déroulante afin d’effectuer une sélection dans la liste des langues disponibles.

La sélection du format de région change afin d’afficher les options régionales uniquement pour la langue que vous avez sélectionnée. Pour changer cette sélection automatique, utilisez la liste déroulante afin de choisir le format de région souhaité.

Par exemple, si vous sélectionnez Anglais comme langue, puis que vous sélectionnez États-Unis comme format de région, la devise est indiquée en dollars américains. De même, si vous sélectionnez Anglais comme langue, puis que vous sélectionnez Europe comme format de région, la devise est indiquée en euros.

Sélectionnez **Appliquer** pour mettre à jour vos paramètres de langue et de format de région.

   ![Capture d’écran montrant les paramètres de langue et de format de région](./media/set-preferences/language.png)

>[!NOTE]
>Ces paramètres linguistiques et régionaux affectent uniquement le portail Azure. Les liens de documentation qui s’ouvrent dans un nouvel onglet ou une nouvelle fenêtre utilisent les paramètres linguistiques de votre navigateur pour déterminer la langue à afficher.
>

## <a name="next-steps"></a>Étapes suivantes

- [Raccourcis clavier du portail Azure](azure-portal-keyboard-shortcuts.md)
- [Navigateurs et appareils pris en charge](azure-portal-supported-browsers-devices.md)
- [Ajouter, supprimer et réorganiser des favoris](azure-portal-add-remove-sort-favorites.md)
- [Créer et partager des tableaux de bord personnalisés](azure-portal-dashboards.md)
- [Série de vidéos de procédures sur le portail Azure](azure-portal-video-series.md)
