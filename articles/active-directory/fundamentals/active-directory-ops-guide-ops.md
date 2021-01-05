---
title: Guide de référence des opérations générales Azure Active Directory
description: Ce guide de référence des opérations décrit les vérifications et les actions à entreprendre pour sécuriser les opérations générales.
services: active-directory
author: martincoetzer
manager: daveba
tags: azuread
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: fundamentals
ms.date: 10/31/2019
ms.author: martinco
ms.openlocfilehash: d5a8fe4192c3778e259ed18239a4198398d8807b
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94836833"
---
# <a name="azure-active-directory-general-operations-guide-reference"></a>Guide de référence des opérations générales Azure Active Directory

Cette section du [guide de référence des opérations Azure AD](active-directory-ops-guide-intro.md) décrit les vérifications et les actions à entreprendre pour optimiser les opérations générales d’Azure Active Directory (Azure AD).

> [!NOTE]
> Ces suggestions sont valables à la date de publication mais peuvent évoluer. Les organisations doivent évaluer leurs pratiques opérationnelles en continu à mesure que les produits et services Microsoft évoluent.

## <a name="key-operational-processes"></a>Processus opérationnels clés

### <a name="assign-owners-to-key-tasks"></a>Affecter les propriétaires à des tâches clés

La gestion d’Azure Active Directory nécessite l’exécution continue de tâches et de processus opérationnels clés, qui peuvent ne pas faire partie d’un projet de déploiement. Il est cependant important de configurer ces tâches pour optimiser votre environnement. Les tâches clés et leurs propriétaires recommandés sont listés ci-après :

| Tâche | Propriétaire |
| :- | :- |
| Apporter des améliorations au score d’identité sécurisée | Équipe des opérations InfoSec |
| Entretenir les serveurs Azure AD Connect | Équipe des opérations IAM |
| Exécuter et trier régulièrement les rapports IdFix | Équipe des opérations IAM |
| Trier les alertes d’Azure AD Connect Health pour la synchronisation et AD FS | Équipe des opérations IAM |
| Si vous n’utilisez pas Azure AD Connect Health, le client dispose de processus et d’outils équivalents pour surveiller l’infrastructure personnalisée | Équipe des opérations IAM |
| Si vous n’utilisez pas AD FS, le client dispose de processus et d’outils équivalents pour surveiller l’infrastructure personnalisée | Équipe des opérations IAM |
| Surveiller les journaux hybrides : Connecteurs de proxy d’application Azure AD | Équipe des opérations IAM |
| Surveiller les journaux hybrides : Agents de l’authentification directe | Équipe des opérations IAM |
| Surveiller les journaux hybrides : Service de réécriture des mots de passe | Équipe des opérations IAM |
| Surveiller les journaux hybrides : Passerelle de protection des mots de passe locale | Équipe des opérations IAM |
| Surveiller les journaux hybrides : Extension NPS Azure AD MFA (le cas échéant) | Équipe des opérations IAM |

Quand vous passerez votre liste en revue, vous devrez peut-être affecter un propriétaire à des tâches qui en sont dépourvues, ou modifier la propriété des tâches avec propriétaires qui ne sont pas conformes aux suggestions ci-dessus.

#### <a name="owners-recommended-reading"></a>Lectures recommandées pour les propriétaires

- [Attribution de rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md)
- [Gouvernance dans Azure](../../governance/index.yml)

## <a name="hybrid-management"></a>Gestion hybride

### <a name="recent-versions-of-on-premises-components"></a>Versions récentes de composants locaux

Lorsqu’un client dispose des versions les plus récentes des composants locaux, il a accès aux dernières mises à jour de sécurité, aux améliorations en matière de performances ainsi qu’aux fonctionnalités permettant de simplifier davantage l’environnement. La plupart des composants ont un paramètre de mise à niveau automatique, qui permet d’automatiser le processus de mise à niveau.

Ces composants sont les suivants :

- Azure AD Connect
- Connecteurs de proxy d’application Azure AD
- Agents d’authentification directe Azure AD
- Agents Azure AD Connect Health

Si aucun processus de mise à niveau des composants n’est établi, nous vous recommandons d’en définir un et d’utiliser la fonctionnalité de mise à niveau automatique autant que possible. Si certains composants datent de six mois ou plus, vous devez les mettre à niveau dès que possible.

#### <a name="hybrid-management-recommended-reading"></a>Lectures recommandées pour la gestion hybride

- [Azure AD Connect : mise à niveau automatique](../hybrid/how-to-connect-install-automatic-upgrade.md)
- [Présentation des connecteurs de proxy d’application Azure AD | Mises à jour automatiques](../manage-apps/application-proxy-connectors.md#automatic-updates)

### <a name="azure-ad-connect-health-alert-baseline"></a>Base de référence des alertes Azure AD Connect Health

Les organisations doivent déployer [Azure AD Connect Health](../hybrid/whatis-azure-ad-connect.md#what-is-azure-ad-connect-health) pour surveiller et générer des rapports d’Azure AD Connect et AD FS. Azure AD Connect et AD FS sont des composants critiques qui peuvent interrompre la gestion du cycle de vie et l’authentification, et ainsi entraîner des pannes. Azure AD Connect Health vous aide à superviser et à obtenir des insights concernant votre infrastructure d’identité locale, garantissant ainsi la fiabilité de votre environnement.

![Architecture d’Azure AD Connect Health](./media/active-directory-ops-guide/active-directory-ops-img16.png)

Lorsque vous surveillez l’intégrité de votre environnement, vous devez traiter immédiatement les alertes avec un niveau de gravité élevé, puis celles avec un niveau de gravité faible.

#### <a name="azure-ad-connect-health-recommended-reading"></a>Lectures recommandées pour Azure AD Connect Health

- [Installation de l’agent Azure AD Connect Health](../hybrid/how-to-connect-health-agent-install.md)

### <a name="on-premises-agents-logs"></a>Journaux des agents locaux

Certains services de gestion des accès et des identités nécessitent que des agents locaux activent des scénarios hybrides. Il s’agit notamment de la réinitialisation de mot de passe, de l’authentification directe (PTA), du Proxy d’application Azure AD et de l’extension NPS pour Azure AD MFA. Il est essentiel que l’équipe des opérations établisse un point de référence et surveille l’intégrité de ces composants en archivant et en analysant les journaux des agents des composants à l’aide de solutions telles que System Center Operations Manager ou SIEM. Il est tout aussi important que votre équipe des opérations Infosec ou de support technique sache comment résoudre les modèles d’erreurs.

#### <a name="on-premises-agents-logs-recommended-reading"></a>Lectures recommandées pour les journaux des agents locaux

- [Résoudre les problèmes du proxy d’application](../manage-apps/application-proxy-troubleshoot.md)
- [Résolution des problèmes relatifs à la réinitialisation de mot de passe libre-service - Azure Active Directory](../authentication/troubleshoot-sspr.md)
- [Présentation des connecteurs de proxy d’application Azure AD](../manage-apps/application-proxy-connectors.md)
- [Azure AD Connect : Résoudre les problèmes liés à l’authentification directe](../hybrid/tshoot-connect-pass-through-authentication.md#collecting-pass-through-authentication-agent-logs)
- [Résoudre les codes d’erreur liés à l’extension NPS pour Azure AD MFA](../authentication/howto-mfa-nps-extension-errors.md)

### <a name="on-premises-agents-management"></a>Gestion des agents locaux

L’adoption de meilleures pratiques peut contribuer à optimiser le fonctionnement des agents locaux. Envisagez les meilleures pratiques suivantes :

- Il est recommandé d’avoir plusieurs connecteurs de proxy d’application Azure AD par groupe de connecteurs pour fournir un équilibrage de charge fluide et une haute disponibilité en évitant les points de défaillance uniques lors de l’accès aux applications de proxy. Si vous disposez actuellement d’un seul connecteur dans un groupe de connecteurs qui gère les applications de production, vous devez déployer au moins deux connecteurs pour la redondance.
- Il peut être utile de créer et d’utiliser un groupe de connecteurs de proxy d’application à des fins de débogage dans le cadre de la résolution des problèmes liés aux scénarios et de l’intégration de nouvelles applications locales. Nous vous recommandons également d’installer des outils de mise en réseau tels que Message Analyzer et Fiddler sur les ordinateurs des connecteurs.
- Il est recommandé d’avoir plusieurs agents d’authentification directe pour fournir un équilibrage de charge fluide et une haute disponibilité en évitant les points de défaillance uniques lors du processus d’authentification. Veillez à déployer au moins deux agents d’authentification directe pour la redondance.

#### <a name="on-premises-agents-management-recommended-reading"></a>Lectures recommandées pour la gestion des agents locaux

- [Présentation des connecteurs de proxy d’application Azure AD](../manage-apps/application-proxy-connectors.md)
- [Authentification directe Azure AD - Démarrage rapide](../hybrid/how-to-connect-pta-quick-start.md#step-4-ensure-high-availability)

## <a name="management-at-scale"></a>Gestion à grande échelle

### <a name="identity-secure-score"></a>Identity Secure Score

Le [score d’identité sécurisée](./identity-secure-score.md) fournit une mesure quantifiable de la position de sécurité de votre organisation. Il est essentiel de passer en revue et de traiter en permanence les constats signalés et de mettre tout en œuvre pour obtenir le meilleur score possible. Le degré de sécurisation vous aide à accomplir les tâches suivantes :

- Évaluer votre méthode de sécurité relative aux identités de façon objective
- Planifier les améliorations à apporter à la sécurité des identités
- Évaluer la réussite de vos améliorations

![Degré de sécurisation](./media/active-directory-ops-guide/active-directory-ops-img17.png)

Si votre organisation n’a pas mis en place de programme pour surveiller les modifications du score d’identité sécurisée, nous vous recommandons d’implémenter un plan et d’affecter des propriétaires pour surveiller et mener à bien des actions d’amélioration. Les organisations doivent remédier dès que possible aux actions d’amélioration dont l’impact du degré est supérieur à 30.

### <a name="notifications"></a>Notifications

Microsoft envoie des e-mails aux administrateurs pour les informer des diverses modifications apportées au service, des mises à jour de configuration requises et des erreurs qui nécessitent une intervention de l’administrateur. Il est important que les clients définissent une adresse e-mail pour que les notifications soient envoyées aux membres d’équipe qui peuvent les comprendre et agir en conséquence. Nous vous recommandons d’ajouter plusieurs destinataires au [Centre de messages](/office365/admin/manage/message-center) et d’exiger que les notifications (y compris les notifications d’Azure AD Connect Health) soient envoyées à une liste de distribution ou une boîte aux lettres partagée. Si vous ne disposez que d’un compte d’administrateur général avec une adresse e-mail, veillez à configurer au moins deux comptes pouvant recevoir des e-mails.

Azure AD utilise deux adresses « De » : <o365mc@email2.microsoft.com>, qui envoie les notifications du Centre de messages, et <azure-noreply@microsoft.com>, qui envoie les notifications concernant :

- [Révisions d’accès Azure AD](../governance/access-reviews-overview.md)
- [Azure AD Connect Health](../hybrid/how-to-connect-health-operations.md#enable-email-notifications)
- [Azure AD Identity Protection](../identity-protection/howto-identity-protection-configure-notifications.md)
- [Azure AD Privileged Identity Management](../privileged-identity-management/pim-email-notifications.md)
- [Notifications de l’expiration prochaine des certificats des applications d’entreprise](../manage-apps/manage-certificates-for-federated-single-sign-on.md#add-email-notification-addresses-for-certificate-expiration)
- Notifications du service de provisionnement des applications d’entreprise

Reportez-vous au tableau suivant pour connaître le type de notifications qui sont envoyées et savoir où les trouver :

| Source de la notification | Contenu | Emplacement |
|:-|:-|:-|
| Contact technique | Erreurs de synchronisation | Portail Azure - Panneau Propriétés |
| Centre de messages | Avis d’incidents et de dégradation des services d’identité et des services backend Microsoft 365 | Portail Office |
| Synthèse hebdomadaire Identity Protection | Synthèse Identity Protection | Panneau Azure AD Identity Protection |
| Azure AD Connect Health | Notifications d’alerte | Portail Azure - Panneau Azure AD Connect Health |
| Notifications des applications d’entreprise | Notifications signalant l’expiration prochaine des certificats et les erreurs d’approvisionnement | Portail Azure - Panneau Application (chaque application possède son propre paramètre d’adresse e-mail) |

#### <a name="notifications-recommended-reading"></a>Notifications sur les lectures recommandées

- [Modifier l’adresse, le contact technique et d’autres informations](/office365/admin/manage/change-address-contact-and-more)

## <a name="operational-surface-area"></a>Surface d’exposition opérationnelle

### <a name="ad-fs-lockdown"></a>Verrouillage AD FS

Les organisations qui configurent des applications pour s’authentifier directement auprès d’Azure AD bénéficient du [verrouillage intelligent Azure AD](../authentication/concept-sspr-howitworks.md). Si vous utilisez AD FS dans Windows Server 2012 R2, implémentez la [protection par verrouillage extranet](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-soft-lockout-protection) AD FS. Si vous utilisez AD FS sur Windows Server 2016 ou version ultérieure, implémentez le [verrouillage extranet intelligent](https://support.microsoft.com/help/4096478/extranet-smart-lockout-feature-in-windows-server-2016). Nous vous recommandons au minimum d’activer le verrouillage extranet pour limiter le risque d’attaque par force brute contre les annuaires Active Directory locaux. Cependant, si vous utilisez AD FS sur Windows 2016 ou version ultérieure, nous vous recommandons d’activer également le verrouillage extranet intelligent, qui vous aidera à atténuer les attaques [par pulvérisation de mots de passe](https://www.microsoft.com/microsoft-365/blog/2018/03/05/azure-ad-and-adfs-best-practices-defending-against-password-spray-attacks/).

Si AD FS est utilisé uniquement pour la fédération Azure AD, certains points de terminaison peuvent être désactivés pour réduire la surface d’exposition aux attaques. Par exemple, si AD FS est utilisé uniquement pour Azure AD, il est préférable de désactiver les points de terminaison WS-Trust autres que ceux activés pour **usernamemixed** et **windowstransport**.

### <a name="access-to-machines-with-on-premises-identity-components"></a>Accès aux ordinateurs avec des composants d’identité locaux

Les organisations doivent verrouiller l’accès aux ordinateurs avec des composants hybrides locaux de la même façon que votre domaine local. Par exemple, un opérateur de sauvegarde ou un administrateur Hyper-V ne doit pas être en mesure de se connecter au serveur Azure AD Connect pour modifier les règles.

Le modèle de niveau administratif Active Directory a été conçu pour protéger les systèmes d’identité à l’aide d’un ensemble de zones de mémoire tampon entre le contrôle total de l’environnement (niveau 0) et les composants de station de travail à haut risque auxquels les attaques nuisent souvent. ![Diagramme montrant les trois couches du modèle de niveau](./media/active-directory-ops-guide/active-directory-ops-img18.png)

Le [modèle de niveau](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material) se compose de trois niveaux et inclut uniquement des comptes d’administrateur, pas des comptes d’utilisateur standard.

- **Niveau 0** : contrôle direct des identités d’entreprise dans l’environnement. Le niveau 0 inclut les comptes, groupes et autres ressources qui ont un contrôle administratif direct ou indirect de la forêt Active Directory, des domaines ou des contrôleurs de domaine et de toutes les ressources qui y sont contenues. Le degré de sécurité de toutes les ressources du niveau 0 est équivalent car elles se contrôlent toutes efficacement les unes les autres.
- **Niveau 1** : contrôle des serveurs et applications d’entreprise. Les ressources du niveau 1 incluent des systèmes d’exploitation serveur, des services cloud et des applications d’entreprise. Les comptes d’administrateur du niveau 1 ont un contrôle administratif d’une valeur commerciale significative sur ces ressources. Les administrateurs de serveur qui gèrent ces systèmes d’exploitation avec la possibilité d’exercer un impact sur tous les services d’entreprise sont un exemple de rôle courant.
- **Niveau 2** : contrôle des stations de travail et appareils des utilisateurs. Les comptes d’administrateur du niveau 2 ont un contrôle administratif d’une valeur commerciale significative sur les stations de travail et appareils des utilisateurs. Exemples : les administrateurs du support technique des ordinateurs et de l’assistance, car ils peuvent exercer un impact sur l’intégrité de presque toutes les données utilisateur.

Verrouillez l’accès aux composants d’identité locaux tels qu’Azure AD Connect, AD FS et les services SQL de la même manière que pour les contrôleurs de domaine.

## <a name="summary"></a>Résumé

Une infrastructure d’identité sécurisée repose sur sept aspects. Cette liste vous permet de définir les actions à entreprendre pour optimiser les opérations pour Azure Active Directory (Azure AD).

- Affectez des propriétaires aux tâches clés.
- Automatiser le processus de mise à niveau pour les composants hybrides locaux.
- Déployer Azure AD Connect Health pour surveiller et générer des rapports d’Azure AD Connect et AD FS.
- Surveiller l’intégrité des composants hybrides locaux en archivant et en analysant les journaux de l’agent du composant à l’aide de System Center Operations Manager ou d’une solution SIEM.
- Implémenter des améliorations en matière de sécurité en mesurant votre niveau de sécurité avec le score d’identité sécurisée.
- Verrouiller AD FS.
- Verrouiller l’accès aux ordinateurs avec des composants d’identité locaux.

## <a name="next-steps"></a>Étapes suivantes

Reportez-vous aux [plans de déploiement d’Azure AD](active-directory-deployment-plans.md) pour connaître les détails de l’implémentation des fonctionnalités que vous n’avez pas déployées.