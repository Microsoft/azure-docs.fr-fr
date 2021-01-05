---
title: Classeur Insights et rapports sur l’accès conditionnel - Azure Active Directory
description: Découvrez comment utiliser le classeur Insights et rapports sur l’accès conditionnel Azure AD pour résoudre les problèmes relatifs aux stratégies.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 08/27/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: dawoo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3c2364eae0d04da8f8e6fe38ae80db7adb8666ce
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89049415"
---
# <a name="conditional-access-insights-and-reporting"></a>Insights et rapports sur l’accès conditionnel

Le classeur Insights et rapports sur l’accès conditionnel vous permet de comprendre l’impact des stratégies d’accès conditionnel sur votre organisation dans la durée. Lors d’une connexion, une ou plusieurs stratégies d’accès conditionnel peuvent s’appliquer. Celles-ci peuvent accorder l’accès si certains critères d’octroi sont satisfaits ou le refuser dans le cas contraire. Étant donné que plusieurs stratégies d’accès conditionnel peuvent être évaluées lors de chaque connexion, le classeur Insights et rapports vous permet de voir l’impact d’une seule stratégie ou d’un sous-ensemble de stratégies.  

## <a name="prerequisites"></a>Prérequis

Pour activer le classeur Insights et rapports, votre locataire doit disposer d’un espace de travail Log Analytics dans lequel conserver les données des journaux de connexion. Pour utiliser l’accès conditionnel, les utilisateurs doivent disposer de licences Azure AD Premium P1 ou P2.

Les rôles suivants peuvent accéder aux insights et aux rapports :  

- Administrateur de l’accès conditionnel 
- Lecteur de sécurité 
- Administrateur de sécurité 
- Lecteur général 
- Administrateur général 

Les utilisateurs doivent également disposer de l’un des rôles d’espace de travail Log Analytics suivants :  

- Contributeur  
- Propriétaire 

### <a name="stream-sign-in-logs-from-azure-ad-to-azure-monitor-logs"></a>Diffuser en continu les journaux de connexion entre les journaux Azure AD et les journaux Azure Monitor 

Si vous n’avez pas intégré les journaux Azure AD aux journaux Azure Monitor, vous devrez effectuer les étapes suivantes avant le chargement du classeur :  

1. [Créer un espace de travail Log Analytics dans Azure Monitor](../../azure-monitor/learn/quick-create-workspace.md).
1. [Intégrer les journaux Azure AD aux journaux Azure Monitor](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md).

## <a name="how-it-works"></a>Fonctionnement 

Pour accéder au classeur Insights et rapports :  

1. Connectez-vous au **portail Azure**.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel** > **Insights et rapports**.

### <a name="get-started-select-parameters"></a>Bien démarrer : Sélectionner les paramètres 

Le tableau de bord Insights et rapports vous permet de voir l’impact d’une ou plusieurs stratégies d’accès conditionnel sur une période donnée. Commencez par définir chacun des paramètres situés en haut du classeur. 

![Tableau de bord Insights et rapports sur l’accès conditionnel dans le portail Azure](./media/howto-conditional-access-insights-reporting/conditional-access-insights-and-reporting-dashboard.png)

**Stratégie d’accès conditionnel** : sélectionnez une ou plusieurs stratégies d’accès conditionnel pour évaluer leur impact. Les stratégies sont classées en deux groupes : les stratégies activées et les stratégies de rapport seul. Par défaut, toutes les stratégies activées sont sélectionnées. Ces stratégies activées correspondent aux stratégies qui sont actuellement appliquées dans votre locataire.  

**Intervalle de temps** : sélectionnez une période comprise entre 4 heures et 90 jours. Même si vous sélectionnez une plage de temps antérieure au moment auquel vous avez intégré les journaux Azure AD à Azure Monitor, seules les connexions effectuées après l’intégration s’afficheront.  

**Utilisateur** : Par défaut, le tableau de bord affiche l’impact des stratégies sélectionnées pour tous les utilisateurs. Pour filtrer par utilisateur, tapez le nom de l’utilisateur en question dans le champ de texte. Pour voir l’impact sur l’ensemble des utilisateurs, tapez « Tous les utilisateurs » dans le champ de texte ou laissez le paramètre vide. 

**Application** : par défaut, le tableau de bord affiche l’impact des stratégies sélectionnées pour toutes les applications. Pour filtrer par application, tapez le nom de l’application en question dans le champ de texte. Pour voir l’impact sur l’ensemble des applications, tapez « Toutes les applications » dans le champ de texte ou laissez le paramètre vide. 

**Vue de données** : indiquez si vous souhaitez que le tableau de bord affiche le nombre d’utilisateurs ou le nombre de connexions. Un même utilisateur peut s’être connecté plusieurs centaines de fois à de nombreuses applications et avoir obtenu des résultats très différents au cours d’une période donnée. Si vous sélectionnez la vue de données Utilisateurs, vous verrez qu’un utilisateur peut être comptabilisé aussi bien dans les connexions réussies que dans les échecs de connexion (par exemple, s’il y a 10 utilisateurs, 8 d’entre eux peuvent avoir réussi à se connecter au cours des 30 derniers jours et 9 d’entre peuvent avoir échoué à se connecter au cours de la même période).

## <a name="impact-summary"></a>Résumé de l’impact 

Une fois les paramètres définis, le résumé de l’impact est chargé. Ce résumé indique, sur une période donnée, le nombre d’utilisateurs (ou connexions) ayant obtenu comme résultat « Réussite », « Échec », « Action de l’utilisateur nécessaire » ou « Non appliqué », lors de l’évaluation des stratégies sélectionnées.  

![Résumé de l’impact dans le classeur Accès conditionnel](./media/howto-conditional-access-insights-reporting/workbook-impact-summary.png)

**Total** : Au cours d’une période donnée, nombre d’utilisateurs ou de connexions pour lesquels au moins une des stratégies sélectionnées a été évaluée.

**Réussite** : Au cours d’une période donnée, nombre d’utilisateurs ou de connexions ayant obtenu comme résultat des stratégies sélectionnées une combinaison de « Réussite » ou « Rapport seul : Réussite ».

**Échec** : Au cours d’une période donnée, nombre d’utilisateurs ou de connexions ayant obtenu, pour au moins une stratégie sélectionnée, le résultat « Réussite » ou « Rapport seul : Échec».

**Action de l’utilisateur nécessaire** : Au cours d’une période donnée, nombre d’utilisateurs ou de connexions ayant obtenu, pour les stratégies sélectionnées, le résultat « Rapport seul : Action de l’utilisateur nécessaire ». Une action utilisateur est nécessaire lorsqu’un contrôle d’octroi interactif, tel qu’une authentification multifacteur, est exigé par une stratégie d’accès conditionnel de type Rapport seul. Étant donné que les contrôles d’octroi interactifs ne sont pas appliqués par les stratégies Rapport seul, il est impossible de déterminer si la connexion a réussi ou échoué.  

**Non applicable** : Au cours d’une période donnée, nombre d’utilisateurs ou de connexions pour lesquels aucune des stratégies sélectionnées n’a été appliquée.

### <a name="understanding-the-impact"></a>Comprendre l’impact 

![Décomposition du classeur par condition et par état](./media/howto-conditional-access-insights-reporting/workbook-breakdown-condition-and-status.png)

Affichez la répartition des utilisateurs (ou connexions) pour chacune des conditions. Vous pouvez filtrer les connexions d’un résultat donné (par exemple, Réussite ou Échec) en sélectionnant l’une des vignettes récapitulatives situées en haut du classeur. Vous pouvez voir la répartition des connexions pour chacune des conditions d’accès conditionnel : état de l’appareil, plateforme de l’appareil, application cliente, emplacement, application et risque de connexion.  

## <a name="sign-in-details"></a>Détails de la connexion 

![Détails des connexions dans le classeur](./media/howto-conditional-access-insights-reporting/workbook-sign-in-details.png)

Vous pouvez également examiner les connexions d’un utilisateur en particulier, en recherchant des connexions en bas du tableau de bord. La requête située sur la gauche affiche les utilisateurs les plus fréquents. La sélection d’un utilisateur va filtrer la requête sur la droite.  

> [!NOTE]
> Lorsque vous téléchargez les journaux de connexion, choisissez le format JSON pour inclure les résultats en mode rapport uniquement de la stratégie d’accès conditionnel.

## <a name="configure-a-conditional-access-policy-in-report-only-mode"></a>Configurer une stratégie d’accès conditionnel en mode Rapport seul

Pour configurer une stratégie d’accès conditionnel en mode Rapport uniquement :

1. Connectez-vous au **portail Azure** en tant qu’administrateur de l’accès conditionnel, administrateur de la sécurité ou administrateur général.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
1. Sélectionnez une stratégie existante ou créez-en une.
1. Sous **Activer la stratégie**, définissez le mode **Rapport uniquement**.
1. Sélectionnez **Enregistrer**.

> [!TIP]
> La modification de l’état **Activer la stratégie** d’une stratégie d’**Activé** en **Rapport seul** désactive l’application de la stratégie. 

## <a name="troubleshooting"></a>Dépannage

### <a name="why-are-queries-failing-due-to-a-permissions-error"></a>Pourquoi les requêtes échouent-elles en raison d’une erreur d’autorisation ?

Pour accéder au classeur, vous devez disposer des autorisations Azure AD appropriées, ainsi que des autorisations de l’espace de travail Log Analytics. Pour vérifier si vous disposez des autorisations appropriées pour l’espace de travail, exécutez un exemple de requête Log Analytics :

1. Connectez-vous au **portail Azure**.
1. Accédez à **Azure Active Directory** > **Journaux**.
1. Saisissez `SigninLogs` dans la zone de requête et sélectionnez **Exécuter**.
1. Si la requête ne retourne aucun résultat, votre espace de travail n’a peut-être pas été configuré correctement. 

![Résoudre les échecs de requêtes](./media/howto-conditional-access-insights-reporting/query-troubleshoot-sign-in-logs.png)

Pour plus d’informations sur la diffusion en continu des journaux de connexion Azure AD vers un espace de travail Log Analytics, consultez l’article [Intégrer des journaux Azure AD avec aux journaux d’activité Azure Monitor](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md).

### <a name="why-are-the-queries-in-the-workbook-failing"></a>Pourquoi les requêtes dans le classeur échouent-elles ?

Des clients ont remarqué que des requêtes échouent parfois si un espace de travail erroné ou plusieurs espaces de travail sont associés au classeur. Pour résoudre ce problème, cliquez sur **Modifier** en haut du classeur, puis sur l’engrenage paramètres. Sélectionnez et supprimez les espaces de travail qui ne sont pas associés au classeur. Il ne doit y avoir qu’un seul espace de travail associé à chaque classeur.

### <a name="why-is-the-conditional-access-policies-parameter-is-empty"></a>Pourquoi le paramètre des stratégies d’accès conditionnel est-il vide ?

La liste des stratégies est générée en examinant les stratégies évaluées pour l’événement de connexion le plus récent. S’il n’existe aucune connexion récente dans votre locataire, vous devrez peut-être attendre pendant quelques minutes que le classeur charge la liste des stratégies d’accès conditionnel. Cela peut se produire immédiatement après la configuration de Log Analytics ou peut prendre plus de temps si un locataire n’a aucune activité de connexion récente.

### <a name="why-is-the-workbook-taking-a-long-time-to-load"></a>Pourquoi le chargement du classeur est-il si long ?  

Selon l’intervalle de temps sélectionné et la taille de votre locataire, le classeur peut avoir à évaluer un nombre important d’événements de connexion. Pour les locataires de grande taille, le volume des connexions peut dépasser la capacité de requête de Log Analytics. Essayez de limiter l’intervalle de temps à 4 heures pour voir si le classeur parvient à être chargé.  

### <a name="after-loading-for-a-few-minutes-why-is-the-workbook-returning-zero-results"></a>Une fois le chargement terminé, après plusieurs minutes d’attente, pourquoi le classeur retourne-il aucun résultat ? 

Lorsque le volume des connexions dépasse la capacité de requête de Log Analytics, le classeur ne retourne aucun résultat. Essayez de limiter l’intervalle de temps à 4 heures pour voir si le classeur parvient à être chargé.  

### <a name="can-i-save-my-parameter-selections"></a>Puis-je enregistrer mes sélections de paramètres ?  

Vous pouvez enregistrer vos sélections de paramètres en haut du classeur, en accédant à **Azure Active Directory** > **Classeurs** > **Insights et rapports sur l’accès conditionnel**. Vous y trouverez le modèle du classeur. Celui-ci vous permet de modifier le classeur et d’en enregistrer une copie dans votre espace de travail (y compris les sélections de paramètres), dans **Mes rapports** ou **Rapports partagés**. 

### <a name="can-i-edit-and-customize-the-workbook-with-additional-queries"></a>Puis-je modifier et personnaliser le classeur en y ajoutant des requêtes ? 

Vous pouvez modifier et personnaliser le classeur en accédant à **Azure Active Directory** > **Classeurs** > **Insights et rapports sur l’accès conditionnel**. Vous y trouverez le modèle du classeur. Celui-ci vous permet de modifier le classeur et d’en enregistrer une copie dans votre espace de travail (y compris les sélections de paramètres), dans **Mes rapports** ou **Rapports partagés**. Pour modifier les requêtes, cliquez sur **Modifier** en haut du classeur.  
 
## <a name="next-steps"></a>Étapes suivantes

- [Mode Rapport seul de l’accès conditionnel](concept-conditional-access-report-only.md)

- Pour plus d’informations sur les classeurs Azure AD, voir l’article[Comment utiliser des classeurs Azure Monitor pour créer des rapports Azure Active Directory](../reports-monitoring/howto-use-azure-monitor-workbooks.md).

- [Stratégies d’accès conditionnel courantes](concept-conditional-access-policy-common.md)
