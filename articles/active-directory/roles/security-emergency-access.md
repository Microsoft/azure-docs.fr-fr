---
title: Gérer des comptes administrateur d’accès en cas d’urgence – Azure AD | Microsoft Docs
description: Cet article explique comment utiliser des comptes d’accès d’urgence pour éviter de vous retrouver empêché par inadvertance d’accéder à votre organisation Azure Active Directory (Azure AD).
services: active-directory
author: markwahl-msft
manager: daveba
ms.author: curtand
ms.date: 11/05/2020
ms.topic: conceptual
ms.service: active-directory
ms.subservice: roles
ms.workload: identity
ms.custom: it-pro
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
ms.openlocfilehash: 10d93b92f3bb0adfe734ad439079afdfcaa6270e
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94834436"
---
# <a name="manage-emergency-access-accounts-in-azure-ad"></a>Gérer des comptes d’accès d’urgence dans Azure AD

Il est important d’éviter de bloquer par inadvertance votre organisation Azure Active Directory (Azure AD), car vous ne pouvez pas vous connecter ou activer un autre compte d’utilisateur en tant qu’administrateur. Vous pouvez pallier l’impact d’un défaut d’accès administratif en créant deux ou plusieurs *comptes d’accès d’urgence* dans votre organisation.

Les comptes d’accès d’urgence sont hautement privilégiés et ne sont pas affectés à des personnes spécifiques. Les comptes d’accès d’urgence sont limités à des cas d’urgence ou à des scénarios « de secours » où il est impossible d’utiliser des comptes d’administration normaux. Nous vous recommandons de vous fixer pour objectif de limiter l’utilisation du compte d’urgence uniquement lorsque cela est strictement nécessaire.

Cet article fournit des instructions pour la gestion des comptes d’accès d’urgence dans Azure AD.

## <a name="why-use-an-emergency-access-account"></a>Pourquoi utiliser un compte d’accès d’urgence ?

Une organisation peut avoir recours à un compte d’accès d’urgence dans les situations décrites ici.

- Les comptes d’utilisateurs sont fédérés, et la fédération est actuellement indisponible en raison d’un dysfonctionnement du réseau cellulaire ou d’une panne du fournisseur d’identité. Par exemple, si l’hôte du fournisseur d’identité dans votre environnement s’est arrêté de fonctionner, les utilisateurs risquent de ne pas pouvoir se connecter lors de la redirection d’Azure AD vers leur fournisseur d’identité.
- Les administrateurs sont inscrits par le biais de l’authentification multifacteur Azure AD. Tous leurs appareils individuels ou le service sont indisponibles. Les utilisateurs peuvent se retrouver dans l’incapacité de procéder à l’authentification multifacteur pour activer un rôle. Par exemple, une panne de réseau cellulaire les empêche de répondre aux appels téléphoniques ou de recevoir des SMS, les deux seuls mécanismes d’authentification à leur disposition qu’ils ont enregistrés pour leur appareil.
- La personne disposant de l’accès administrateur général le plus récent a quitté l’organisation. Azure AD empêche la suppression du dernier compte d’administrateur général, mais n’empêche pas ce compte d’être supprimé ou désactivé localement. Chacune de ces situations peut rendre impossible la récupération du compte par l’organisation.
- Un événement imprévu, comme une catastrophe naturelle, au cours duquel les téléphones mobiles ou d’autres réseaux sont indisponibles. 

## <a name="create-emergency-access-accounts"></a>Créer des comptes d’accès d’urgence

Créez plusieurs comptes d’accès d’urgence. Ces comptes doivent être des comptes uniquement cloud utilisant le domaine \*. onmicrosoft.com, et ne pas être fédérés ou synchronisés à partir d’un environnement local.

Lors de la configuration de ces comptes, les conditions suivantes doivent être remplies :

- Les comptes d’accès d’urgence ne doivent être associés à aucun utilisateur au sein de l’organisation. Veillez à ce que vos comptes d’accès d’urgence ne soient pas connectés à un matériel fourni à un employé et voyageant avec celui-ci, tel un téléphone mobile, un module de sécurité matériel, ou d’autres informations d’identification propres à l’employé. Cette précaution de sécurité couvre les cas où un employé n’est pas joignable alors que les informations d’identification doivent être fournies. Il est important de s’assurer que tous les appareils inscrits sont conservés dans un endroit sûr et connu, disposant de plusieurs moyens de communication avec Azure AD.
- Le mécanisme d’authentification utilisé pour un compte d’accès d’urgence doit être distinct de celui utilisé par d’autres comptes administratifs, y compris d’autres comptes d’accès d’urgence.  Par exemple, si votre administrateur normal se connecte par authentification MFA locale, l’authentification multifacteur Azure AD sera un mécanisme différent.  En revanche, si l’authentification multifacteur Azure AD est votre principal composant d’authentification pour vos comptes administratifs, envisagez une approche différente pour ces derniers, par exemple l’accès conditionnel avec un fournisseur MFA tiers au moyen de contrôles personnalisés.
- L’appareil ou les informations d’identification ne doivent pas expirer ou faire potentiellement l’objet d’un nettoyage automatisé en raison d’une utilisation insuffisante.  
- Vous devez rendre l’attribution de rôle d’administrateur général permanente pour vos comptes d’accès d’urgence. 

### <a name="exclude-at-least-one-account-from-phone-based-multi-factor-authentication"></a>Exclure au moins un compte de l’authentification multifacteur par téléphone

Afin de réduire le risque d’attaques résultant d’un mot de passe compromis, Azure AD recommande de généraliser l’authentification multifacteur pour tous les utilisateurs. Ce groupe inclut les administrateurs et tous les autres utilisateurs (par exemple, les responsables financiers) dont un compte compromis aurait un impact important.

Cependant, au moins un de vos comptes d’accès d’urgence ne doit pas avoir le même mécanisme d’authentification multifacteur que vos autres comptes non dédiés à l’accès d’urgence. Cela inclut les solutions d’authentification multifacteur tierces. Si vous avez une stratégie d’accès conditionnel qui impose l’[authentification multifacteur à chaque administrateur](../authentication/howto-mfa-userstates.md) pour Azure AD et d’autres applications SaaS connectées, vous devez soustraire les comptes d’accès d’urgence de cette exigence et configurer un autre mécanisme à la place. Vous devez en outre vous assurer que les comptes ne font pas l’objet d’une stratégie d’authentification multifacteur par utilisateur.

### <a name="exclude-at-least-one-account-from-conditional-access-policies"></a>Exclure au moins un compte des stratégies d’accès conditionnel

En cas d’urgence, vous ne souhaitez pas qu’une stratégie risque de vous empêcher d’accéder pour résoudre un problème. Au moins un compte d’accès d’urgence doit être exclu de toutes les stratégies d’accès conditionnel.

## <a name="federation-guidance"></a>Conseils sur la fédération

Une option supplémentaire pour les organisations qui utilisent Azure AD Domain Services et ADFS ou un fournisseur d’identité similaire pour fédérer sur Azure AD, consiste à configurer un compte d’accès d’urgence dont la revendication MFA pourrait être fournie par ce fournisseur d’identité.  Par exemple, le compte d’accès d’urgence pourrait s’appuyer sur un certificat et une paire de clés stockée sur une carte à puce.  Quand cet utilisateur est authentifié auprès d’AD, ADFS peut transmettre une revendication à Azure AD, indiquant que l’utilisateur a satisfait aux exigences de MFA.  Même avec cette approche, les organisations doivent toujours disposer de comptes d’accès d’urgence basés sur le cloud au cas où une fédération ne pourrait pas être établie. 

## <a name="store-account-credentials-safely"></a>Stocker les informations d’identification de compte en toute sécurité

Les organisations sont tenues de garantir la sécurisation des informations d’identification pour les comptes d’accès d’urgence, et une confidentialité limitée aux seules personnes autorisées à les utiliser. Certains clients utilisent une carte à puce, et d’autres des mots de passe. Un mot de passe pour un compte d’accès d’urgence se divise généralement en deux ou trois parties notées sur des feuilles de papier séparées, et stockées dans des coffres-forts ignifuges installés dans des endroits distincts et sécurisés.

Si vous utilisez des mots de passe, assurez-vous que les comptes ont des mots de passe forts qui n’expirent pas. Idéalement, les mots de passe doivent comprendre au moins 16 caractères générés de façon aléatoire.

## <a name="monitor-sign-in-and-audit-logs"></a>Surveiller les journaux de connexion et d’audit

Les organisations doivent surveiller l’activité de connexion et du journal d’audit à partir des comptes d’urgence et déclencher des notifications à d’autres administrateurs. Lorsque vous surveillez l’activité sur des comptes d’urgence, vous pouvez vérifier que ces comptes sont utilisés uniquement pour les tests ou les urgences réelles. Vous pouvez utiliser Azure Log Analytics pour surveiller les journaux de connexion et déclencher des alertes par e-mail et SMS à vos administrateurs à chaque fois que des comptes d’urgence se connectent.

### <a name="prerequisites"></a>Prérequis

1. [Envoyez les journaux de connexion Azure AD](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md) à Azure Monitor.

### <a name="obtain-object-ids-of-the-break-glass-accounts"></a>Obtenir les ID d’objet des comptes de secours

1. Connectez-vous au [portail Azure](https://portal.azure.com) en utilisant un compte attribué au rôle Administrateur d’utilisateurs.
1. Sélectionnez **Azure Active Directory** > **Utilisateurs**.
1. Recherchez le compte de secours et sélectionnez le nom de l’utilisateur.
1. Copiez et enregistrez l’attribut ID d’objet afin de pouvoir l’utiliser ultérieurement.
1. Répétez les étapes précédentes pour le deuxième compte de secours.

### <a name="create-an-alert-rule"></a>Création d'une règle d'alerte

1. Connectez-vous au [portail Azure](https://portal.azure.com) en utilisant un compte attribué au rôle Contributeur de surveillance dans Azure Monitor.
1. Sélectionnez **Tous les services**, entrez « log analytics » dans Rechercher, puis sélectionnez **Espaces de travail Log Analytics**.
1. Sélectionnez un espace de travail.
1. Dans votre espace de travail, sélectionnez **Alertes** > **Nouvelle règle d’alerte**.
    1. Sous **Ressource**, vérifiez que l’abonnement est celui auquel vous souhaitez associer la règle d’alerte.
    1. Sous **Condition**, sélectionnez **Ajouter**.
    1. Sélectionnez **Recherche de journal personnalisée** sous **Nom du signal**.
    1. Sous **Requête de recherche**, entrez la requête suivante, en insérant les ID d’objet des deux comptes de secours.
        > [!NOTE]
        > Pour chaque compte de secours supplémentaire que vous souhaitez inclure, ajoutez une autre valeur « or UserId == "ObjectGuid" » à la requête.

        ![Ajouter les ID d’objet des comptes de secours à une règle d’alerte](./media/security-emergency-access/query-image1.png)

    1. Sous **Logique d’alerte**, entrez ce qui suit :

        - Basé sur : Nombre de résultats
        - Opérateur : Supérieur à
        - Valeur de seuil : 0

    1. Sous **Évalué en fonction de**, sélectionnez **Durée (en minutes)** pour indiquer la durée pendant laquelle la requête doit s’exécuter et **Fréquence (en minutes)** pour indiquer à quelle fréquence la requête doit s’exécuter. La fréquence doit être inférieure ou égale à la durée.

        ![logique d’alerte](./media/security-emergency-access/alert-image2.png)

    1. Sélectionnez **Terminé**. Vous pouvez maintenant voir le coût mensuel estimé de cette alerte.
1. Sélectionnez un groupe d’actions d’utilisateurs devant être avertis par l’alerte. Si vous souhaitez en créer un, consultez [Créer un groupe d’actions](#create-an-action-group).
1. Pour personnaliser les notifications par e-mail envoyées aux membres du groupe d’actions, sélectionnez Actions sous **Personnaliser les actions**.
1. Sous **Détails de l’alerte**, spécifiez le nom de la règle d’alerte et ajoutez une description facultative.
1. Définissez le **Niveau de gravité** de l’événement. Nous vous recommandons de le définir sur **Critique(Grav 0)** .
1. Sous **Activer la règle lors de la création**, conserver la valeur **Oui**.
1. Pour désactiver les alertes pendant un certain temps, activez la case à cocher **Supprimer les alertes** et entrez la durée d’attente avant que les alertes ne soient à nouveau activées, puis sélectionnez **Enregistrer**.
1. Cliquez sur **Créer une règle d'alerte**.

### <a name="create-an-action-group"></a>Créer un groupe d’actions

1. Sélectionnez **Créer un groupe d’actions**.

    ![créer un groupe d’actions pour les actions de notification](./media/security-emergency-access/action-group-image3.png)

1. Entrez le nom du groupe d’actions et un nom court.
1. Vérifiez l’abonnement et le groupe de ressources.
1. Sous le type d’action, sélectionnez **E-mail/SMS/Push/Voix**.
1. Entrez un nom d’action, par exemple **Avertir l’administrateur général**.
1. Sélectionnez le **Type d’action** **E-mail/SMS/Push/Voix**.
1. Sélectionnez **Modifier les détails** pour sélectionner les méthodes de notification que vous souhaitez configurer, puis entrez les coordonnées requises et cliquez sur **OK** pour enregistrer les détails.
1. Ajoutez toutes les actions supplémentaires que vous souhaitez déclencher.
1. Sélectionnez **OK**.

## <a name="validate-accounts-regularly"></a>Valider régulièrement les comptes

Lorsque vous formez les membres du personnel à l’utilisation et à la validation des comptes d’accès d’urgence, suivez au moins les procédures suivantes à intervalles réguliers :

- Assurez-vous que les agents de la surveillance et de la sécurité sont informés que l’activité de vérification du compte est permanente.
- Assurez-vous que le processus de secours permettant d’utiliser ces comptes est documenté et à jour.
- Assurez-vous que les administrateurs et les responsables de la sécurité susceptibles de devoir suivre ces étapes en cas d’urgence sont formés au processus.
- Mettez à jour les informations d’identification de compte, en particulier les mots de passe, pour vos comptes d’accès d’urgence, puis confirmez que ceux-ci peuvent se connecter et effectuer des tâches administratives.
- Assurez-vous que les utilisateurs n’ont pas enregistré d’authentification multifacteur ou de réinitialisation de mot de passe en libre-service (SSPR) sur un appareil individuel ou dans des informations personnelles. 
- Si les comptes sont inscrits pour une authentification multifacteur sur un appareil, en vue de servir pendant la connexion ou l’activation de rôle, vérifiez que tous les administrateurs susceptibles d’utiliser l’appareil en cas d’urgence ont accès à celui-ci. Vérifiez également que l’appareil peut communiquer via au moins deux chemins d’accès réseau qui ne partagent pas un mode d’échec commun. Par exemple, l’appareil doit être capable de communiquer sur internet via le réseau sans fil d’une installation et le réseau d’un fournisseur de données cellulaires.

Ces étapes doivent être effectuées à intervalles réguliers et pour les principaux changements :

- Au moins tous les 90 jours
- En cas de remaniement récent de l’équipe informatique, par exemple suite à un changement de poste, à un départ ou à l’arrivée d’un nouvel employé
- Lorsque les abonnements Azure AD de l’organisation changent

## <a name="next-steps"></a>Étapes suivantes

- [Sécurisation de l’accès privilégié pour les déploiements hybrides et cloud dans Azure AD](security-planning.md)
- [Ajouter des utilisateurs d’AAD](../fundamentals/add-users-azure-active-directory.md) et [attribuer au nouvel utilisateur le rôle d’administrateur général](../fundamentals/active-directory-users-assign-role-azure-portal.md)
- [S’inscrire à Azure AD Premium](../fundamentals/active-directory-get-started-premium.md) si ce n’est déjà fait
- [Comment exiger la vérification en deux étapes pour un utilisateur](../authentication/howto-mfa-userstates.md)
- [Configurer des protections supplémentaires pour les administrateurs généraux dans Microsoft 365](/office365/enterprise/protect-your-global-administrator-accounts) si vous utilisez Microsoft 365
- [Commencer une vérification de l’accès des administrateurs généraux](../privileged-identity-management/pim-how-to-start-security-review.md) et [opérer une transition des administrateurs généraux existants vers des rôles d’administrateur plus spécifiques](permissions-reference.md)