---
title: Activer la protection par mot de passe Azure AD en local
description: Découvrir comment activer la protection par mot de passe Azure AD dans un environnement Active Directory Domain Services local
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 03/05/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: f2e11fa1bb9f1d3a3986a19da8ed44229829ec81
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741777"
---
# <a name="enable-on-premises-azure-active-directory-password-protection"></a>Activer la protection par mot de passe Azure Active Directory en local

Les utilisateurs créent souvent des mots de passe basés sur des mots locaux courants, par exemple une école, une équipe de sport ou une personne célèbre. Ces mots de passe sont faciles à deviner et offrent une faible protection contre les attaques par dictionnaire. Pour appliquer des mots de passe forts au sein de votre organisation, la protection par mot de passe Azure Active Directory (Azure AD) fournit une liste globale et personnalisée de mots de passe interdits. Toute demande de changement de mot de passe échoue s’il existe une correspondance dans la liste personnalisée de mots de passe interdits.

Pour protéger votre environnement Active Directory Domain Services (AD DS) local, vous pouvez installer et configurer la protection par mot de passe Azure AD pour qu’elle fonctionne avec votre contrôleur de domaine local. Cet article explique comment activer la protection par mot de passe Azure AD pour votre environnement local.

Pour plus d’informations sur le fonctionnement de la protection par mot de passe Azure AD dans un environnement local, voir [Comment appliquer la protection par mot de passe Azure AD pour Windows Server Active Directory](concept-password-ban-bad-on-premises.md).

## <a name="before-you-begin"></a>Avant de commencer

Cet article explique comment activer la protection par mot de passe Azure AD pour votre environnement local. Avant de pouvoir effectuer ce qui est décrit dans cet article, [installez et inscrivez les agents Contrôleur de domaine et le service proxy de la protection par mot de passe Azure AD](howto-password-ban-bad-on-premises-deploy.md) dans votre environnement Active Directory Domain Services local.

## <a name="enable-on-premises-password-protection"></a>Activer la protection par mot de passe en local

1. Connectez-vous à la [Portail Azure](https://portal.azure.com) et accédez à **Azure Active Directory** > **Sécurité** > **Méthodes d’identification** > **Protection des mots de passe**.
1. Définissez l’option **Activer la protection par mot de passe sur Windows Server Active Directory** sur *Oui*.

    Lorsque ce paramètre est défini sur *Non*, tous les agents Contrôleur de domaine de la protection par mot de passe Azure AD déployés passent en mode inactif, mode dans lequel tous les mots de passe sont acceptés tels quels. Aucune activité de validation n’est effectuée et les événements d’audit ne sont pas générés.

1. Il est recommandé de définir initialement le **Mode** sur *Audit*. Une fois que vous êtes familiarisé avec la fonctionnalité et l’impact observé sur les utilisateurs de votre organisation, vous pouvez changer le **Mode** pour *Appliqué*. Pour plus d’informations, consultez la section suivante sur les [modes de fonctionnement](#modes-of-operation).
1. Quand vous êtes prêt, sélectionnez **Enregistrer**.

    [![Enable on-premises password protection under Authentication Methods in the Azure portal](media/howto-password-ban-bad-on-premises-operations/enable-configure-custom-banned-passwords-cropped.png)](media/howto-password-ban-bad-on-premises-operations/enable-configure-custom-banned-passwords.png#lightbox)

## <a name="modes-of-operation"></a>Modes de fonctionnement

Lorsque vous activez la protection par mot de passe Azure AD en local, vous pouvez utiliser le mode *Audit* ou le mode *Appliqué*. Nous suggérons que les déploiement et test initiaux soient toujours démarrés en mode d’audit. Les entrées dans le journal des événements doivent ensuite être supervisées pour anticiper les perturbations éventuelles d’un processus opérationnel existant une fois le mode *Appliqué* activé.

### <a name="audit-mode"></a>Mode Audit

Le mode *Audit* est conçu comme un moyen d’exécuter le logiciel dans un mode de « simulation ». Chaque service d’agent Contrôleur de domaine de la protection par mot de passe Azure AD évalue un mot de passe entrant en fonction de la stratégie active.

Si la stratégie actuelle est configurée pour être en mode d’audit, les « mauvais » mots de passe entraînent l’apparition de messages dans le journal des événements, mais ils sont traités et mis à jour. Ce comportement est la seule différence existant entre le mode d’audit et le mode d’application Toutes les autres opérations s’exécutent de la même façon.

### <a name="enforced-mode"></a>Mode Appliqué

Le mode *Appliqué* est destiné à être la configuration finale. Tout comme dans le mode d’audit, chaque service d’agent Contrôleur de domaine de la protection par mot de passe Azure AD évalue les mots de passe entrants en fonction de la stratégie active. Cependant, lorsque le mode d’application est activé, un mot de passe, considéré comme non sécurisé selon la stratégie, est rejeté.

Lorsqu’un mot de passe est refusé en mode d’application par l’agent Contrôleur de domaine de la protection par mot de passe Azure AD, l’utilisateur final affiche le même genre d’erreur que si son mot de passe était rejeté par la mise en œuvre en local de la stratégie de complexité du mot de passe habituelle. Par exemple, un utilisateur peut voir le message d’erreur classique suivant sur l’écran de changement de mot de passe ou d’ouverture de session de Windows :

*« Impossible de mettre à jour le mot de passe. La valeur fournie pour le nouveau mot de passe ne répond pas aux exigences de longueur, complexité ou historique du domaine. »*

Ce message n’est qu’un seul exemple de plusieurs résultats possibles. Le message d’erreur spécifique peut varier selon le logiciel ou le scénario qui tente de définir un mot de passe non sécurisé.

Les utilisateurs finals concernés devront peut-être se mettre en rapport avec leur service informatique pour comprendre les nouvelles conditions exigées et choisir des mots de passe sécurisés.

> [!NOTE]
> La protection par mot de passe Azure AD n’a aucun contrôle sur le message d’erreur spécifique affiché par l’ordinateur client lorsqu’un mot de passe faible est rejeté.

## <a name="next-steps"></a>Étapes suivantes

Pour personnaliser la liste des mots de passe interdits de votre organisation, consultez [Configurer la liste personnalisée des mots de passe interdits de la protection par mot de passe Azure AD](tutorial-configure-custom-password-protection.md).

Pour superviser les événements en local, consultez [Supervision de la protection par mot de passe Azure AD en local](howto-password-ban-bad-on-premises-monitor.md).
