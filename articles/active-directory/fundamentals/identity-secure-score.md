---
title: Qu’est-ce qu’un degré de sécurisation Identity Secure Score ? - Azure Active Directory
description: Procédure d’utilisation du degré de sécurisation Identity Secure Score pour améliorer la méthode de sécurité de votre répertoire
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 02/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: tilarso
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8c319489fe4c884cd5de48ac2d3e47e7beb3026f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90705484"
---
# <a name="what-is-the-identity-secure-score-in-azure-active-directory"></a>Qu’est-ce que le degré de sécurisation Identity Secure Score dans Azure Active Directory ?

Dans quelle mesure votre locataire Azure AD est-il sécurisé ? Si vous ne savez pas comment répondre à cette question, cet article explique comment le degré de sécurisation Identity Secure Score vous aide à surveiller et améliorer votre méthode de sécurité relative aux identités.

## <a name="what-is-an-identity-secure-score"></a>Qu’est-ce qu’un degré de sécurisation Identity Secure Score ?

Le degré de sécurisation Identity Secure Score est un nombre compris entre 1 et 223 qui indique dans quelle mesure vous respectez les suggestions de meilleures pratiques de Microsoft en matière de sécurité. Chaque action d’amélioration du score d’identité sécurisée est adaptée à votre configuration spécifique.  

![Degré de sécurisation](./media/identity-secure-score/identity-secure-score-overview.png)

Le degré de sécurisation vous aide à accomplir les tâches suivantes :

- Évaluer votre méthode de sécurité relative aux identités de façon objective
- Planifier les améliorations à apporter à la sécurité des identités
- Évaluer la réussite de vos améliorations

Vous pouvez accéder au degré de sécurisation et aux informations correspondantes dans le tableau de bord du degré de sécurisation Identity Secure Score. Ce tableau de bord présente les informations suivantes :

- Votre degré de sécurisation Identity Secure Score
- Graphique de comparaison qui situe votre degré de sécurisation Identity Secure Score par rapport aux autres locataires de tailles similaires et du même secteur d’activité
- Graphique de tendance représentant l’évolution de votre score d’identité sécurisée au fil du temps
- Liste des améliorations possibles

La mise en œuvre des actions d’amélioration vous offre les possibilités suivantes :

- Améliorez votre méthode de sécurité et votre degré de sécurisation
- Tirez profit des fonctionnalités dont dispose votre entreprise dans le cadre de vos investissements d’identité

## <a name="how-do-i-get-my-secure-score"></a>Comment faire pour obtenir mon degré de sécurisation ?

Le degré de sécurisation Identity Secure Score est disponible avec toutes les éditions d’Azure AD. Les organisations peuvent accéder à leur degré de sécurisation de sécurité à partir du **portail Azure** > **Azure Active Directory** > **Sécurité** > **Degré de sécurisation d’identité**.

## <a name="how-does-it-work"></a>Comment cela fonctionne-t-il ?

Toutes les 48 heures, Azure examine votre configuration de sécurité et compare vos paramètres avec ceux des meilleures pratiques suggérées. Selon le résultat de cette évaluation, un nouveau degré de sécurisation est calculé pour votre répertoire. Il est possible que votre configuration de sécurité ne soit pas entièrement alignée sur les meilleures pratiques et que les actions d’amélioration ne soient que partiellement réalisées. Dans ces scénarios, vous ne recevrez qu’une partie de la note maximale disponible pour le témoin.

Chaque suggestion est évaluée en fonction de votre configuration Azure AD. Si vous utilisez des produits tiers pour mettre en œuvre une suggestion de meilleure pratique, vous pouvez indiquer cette configuration dans les paramètres d’une action d’amélioration. Vous avez la possibilité de définir les suggestions à ignorer si elles ne s’appliquent pas à votre environnement. Les suggestions ignorées ne sont pas prises en compte dans le calcul de votre degré de sécurisation.

![Ignorer ou marquer une action comme étant couverte par un tiers](./media/identity-secure-score/identity-secure-score-ignore-or-third-party-reccomendations.png)

## <a name="how-does-it-help-me"></a>En quoi cela peut-il m’aider ?

Le degré de sécurisation vous aide à accomplir les tâches suivantes :

- Évaluer votre méthode de sécurité relative aux identités de façon objective
- Planifier les améliorations à apporter à la sécurité des identités
- Évaluer la réussite de vos améliorations

## <a name="what-you-should-know"></a>Ce que vous devez savoir

### <a name="who-can-use-the-identity-secure-score"></a>Qui peut utiliser le degré de sécurisation Identity Secure Score ?

Le degré de sécurisation Identity Secure Score peut être utilisé par les rôles suivants :

- Administrateur général
- Administrateur de la sécurité
- Lecteurs Sécurité

### <a name="how-are-controls-scored"></a>Comment les témoins sont-ils notés ?

Les témoins peuvent être évalués de deux manières. Certains sont évalués de façon binaire - vous obtenez 100 % du score si vous disposez de la fonctionnalité ou du paramètre configuré conformément à notre recommandation. Les autres scores sont calculés sous forme de pourcentage de la configuration totale. Par exemple, si la recommandation d’amélioration indique que vous obtiendrez 30 points si vous protégez tous vos utilisateurs avec l’authentification multifacteur et que vous avez seulement 5 utilisateurs protégés sur 100, vous obtiendrez un score partiel autour de 2 points (5 protégés / 100 au total * 30 points maximum = 2 points de score partiel).

### <a name="what-does-not-scored-mean"></a>Que signifie la mention [Not Scored] (Degré non calculé) ?

Les actions accompagnées de la mention [Not Scored] (Degré non calculé) sont les actions que vous pouvez mettre en œuvre dans votre organisation, mais dont le degré de sécurisation n’est pas calculé, car elles ne sont pas (encore) intégrées à l’outil. Par conséquent, vous pouvez toujours améliorer votre sécurité, mais vous n’obtenez pas les points correspondant aux actions mises en œuvre pour le moment.

### <a name="how-often-is-my-score-updated"></a>Quelle est la fréquence de mise à jour de mon degré de sécurisation ?

Le degré de sécurisation est calculé une fois par jour (vers 1 h PST). Si vous apportez une modification à une action évaluée, le degré de sécurisation est automatiquement mis à jour le lendemain. La répercussion d’une modification dans le degré de sécurisation peut prendre jusqu’à 48 heures.

### <a name="my-score-changed-how-do-i-figure-out-why"></a>Mon degré de sécurisation a changé. Comment savoir pourquoi ?

Rendez-vous dans le [Centre de sécurité Microsoft 365](https://security.microsoft.com/) pour y retrouver votre score sécurisé Microsoft complet. L’onglet Historique permet de consulter aisément toutes les modifications apportées en profondeur à votre score sécurisé.

### <a name="does-the-secure-score-measure-my-risk-of-getting-breached"></a>Le degré de sécurisation mesure-t-il le risque de violation ?

Non. Le degré de sécurisation ne fournit pas une mesure absolue de la probabilité d’être victime d’une violation. Il indique dans quelle mesure vous appliquez des fonctionnalités qui peuvent atténuer le risque de violation. Aucun service ne peut garantir que vous ne serez pas victime d’une violation. Le degré de sécurisation ne doit en aucun cas être interprété comme une garantie.

### <a name="how-should-i-interpret-my-score"></a>Comment dois-je interpréter mon degré de sécurisation ?

Vous obtenez des points lorsque vous configurez des fonctionnalités de sécurité suggérées ou mettez en œuvre des tâches liées à la sécurité (par exemple, la lecture de rapports). Certaines actions sont évaluées dans le cadre d’une mise en œuvre partielle, par exemple lorsque vous activez l’authentification multifacteur (MFA) pour vos utilisateurs. Votre degré de sécurisation représente de façon directe les services de sécurité Microsoft que vous utilisez. N’oubliez pas que vous devez trouver le juste équilibre entre sécurité et facilité d’utilisation. Tous les contrôles de sécurité ont une forme d’impact sur l’utilisateur. Les contrôles ayant un faible impact sur l’utilisateur doivent avoir peu d’effet (voire aucun) sur les opérations quotidiennes de vos utilisateurs.

Pour consulter votre historique de score, rendez-vous dans le [Centre de sécurité Microsoft 365](https://security.microsoft.com/) et passez en revue votre score sécurisé Microsoft global. Vous pouvez examiner les modifications apportées à votre score sécurisé global en cliquant sur Afficher l’historique. Choisissez une date spécifique pour déterminer les contrôles qui ont été activés ce jour et les points que vous avez gagnés pour chacun d’eux.

### <a name="how-does-the-identity-secure-score-relate-to-the-microsoft-365-secure-score"></a>Quel est le lien entre les degrés de sécurisation Identity Secure Score et Microsoft 365 Secure Score ?

Le [score sécurisé Microsoft](/office365/securitycompliance/microsoft-secure-score) comporte cinq catégories distinctes de contrôle et de score :

- Identité
- Données
- Appareils
- Infrastructure
- Applications

Le degré de sécurisation Identity Secure Score représente la partie identité du degré de sécurisation Microsoft Secure Score. Ce chevauchement signifie que vos suggestions relatives au degré de sécurisation Identity Secure Score et au degré de sécurisation des identités dans Office 365 sont les mêmes.

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur le score sécurisé Microsoft](/office365/securitycompliance/microsoft-secure-score)