---
title: Fournir des commentaires relatifs au risque dans Azure Active Directory Identity Protection
description: Comment et pourquoi vous devez fournir des commentaires sur les détections de risques dans Azure AD Identity Protection.
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: how-to
ms.date: 06/05/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 939ffc9e2a8fb8ce3a25dc212e3df34dc6bb2ec1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88950355"
---
# <a name="how-to-give-risk-feedback-in-azure-ad-identity-protection"></a>Procédure : Indiquer des commentaires sur les risques dans Azure AD Identity Protection

Azure AD Identity Protection vous permet d’indiquer des commentaires sur l’évaluation de ses risques. Le document suivant répertorie les scénarios où vous souhaiterez sans doute envoyer vos commentaires sur l’évaluation des risques du service Azure AD Identity Protection. Il indique également comment nous intégrons vos commentaires.

## <a name="what-is-a-detection"></a>Qu’est-ce qu’une détection ?

Une détection Azure AD Identity Protection est un indicateur d’activité suspecte du point de vue des risques d’identité. Les activités suspectes sont appelées des détections de risques. Ces détections basées sur l’identité peuvent être fonction d’une heuristique, de l’apprentissage automatique, ou peuvent provenir encore des produits de partenaires. Ces détections permettent de déterminer les risques liées aux connexions et aux utilisateurs :

* Un risque utilisateur reflète la probabilité qu’une identité soit compromise.
* Un risque de connexion représente la probabilité qu’une connexion soit compromise (par exemple, elle n’est pas autorisée par le propriétaire de l’identité).

## <a name="why-should-i-give-risk-feedback-to-azure-ads-risk-assessments"></a>Pourquoi fournir des commentaires pour l’évaluation des risques d’Azure AD ? 

Plusieurs raisons motivent l’envoi de commentaires sur les risques dans Azure AD :

- **Vous avez jugé incorrecte l’évaluation des risques utilisateur ou de connexion d’Azure AD**. Par exemple, une connexion figurant dans le rapport « Connexions à risque » s’est révélée inoffensive, et toutes les détections correspondantes étaient des faux positifs.
- **Vous avez confirmé que l’évaluation des risques de connexion ou utilisateur d’Azure AD était correcte**. Par exemple, une connexion figurant dans le rapport « Connexions à risque » était en effet malveillante, et vous souhaitez que toutes les détections correspondantes soient considérées comme des vrais positifs dans Azure AD.
- **Vous avez réduit le risque pesant sur un utilisateur donné en dehors d’Azure AD Identity Protection**, et vous souhaitez mettre à jour le niveau de risque de cet utilisateur.

## <a name="how-does-azure-ad-use-my-risk-feedback"></a>Comment mes commentaires portant sur les risques sont-ils exploités dans Azure AD ?

Azure AD utilise vos commentaires pour mettre à jour le risque de l’utilisateur ou de la connexion sous-jacents et la précision de ces événements. Ces commentaires contribuent à protéger les utilisateurs finals. Par exemple, après que vous avez confirmé qu’une connexion est compromise, Azure AD fait immédiatement passer le niveau de risque de l’utilisateur et de risque agrégé de connexion (risque autre qu’en temps réel) sur Élevé. Si cet utilisateur est inclus dans votre stratégie de risque utilisateur qui vise à forcer les utilisateurs à risques élevés à réinitialiser en toute sécurité leur mot de passe, l’utilisateur corrige automatiquement le risque lors de sa prochaine connexion.

## <a name="how-should-i-give-risk-feedback-and-what-happens-under-the-hood"></a>Comment fournir des commentaires sur les risques ? Quels sont les mécanismes sous-jacents ?

Voici les scénarios et les mécanismes d’envoi des commentaires sur les risques à Azure AD.

| Scénario | Comment envoyer vos commentaires ? | Quel est le mécanisme sous-jacent ? | Notes |
| --- | --- | --- | --- |
| **Connexion non compromise (faux positif)** <br> Le rapport « Connexions à risque » indique une connexion à risque [État du risque = À risque], mais celle-ci n’était pas compromise. | Sélectionnez la connexion, puis cliquez sur « Confirmer que la connexion est sécurisée ». | Azure AD déplace le risque d’agrégation de la connexion sur Aucun [État du risque = Sécurité confirmée ; Niveau de risque (agrégat) =-] et inverse son impact sur le risque de l’utilisateur. | Actuellement, l’option « Confirmer que la connexion est sécurisée » est uniquement disponible dans le rapport « Connexions à risque ». |
| **Connexion compromise (vrai positif)** <br> Le rapport « Connexions à risque » indique une connexion à risque [État du risque = À risque] avec un risque faible [Niveau de risque (agrégat) = Faible], et cette connexion a été effectivement compromise. | Sélectionnez la connexion, puis cliquez sur « Confirmer que la connexion est compromise ». | Azure AD déplace le risque d’agrégation de la connexion et le risque utilisateur sur le niveau Élevé [État du risque = Compromission confirmée ; Niveau de risque = Élevé]. | Actuellement, l’option « Confirmer que la connexion est compromise » est uniquement disponible dans le rapport « Connexions à risque ». |
| **Utilisateur compromis (vrai positif)** <br> Le rapport « Utilisateurs à risque » indique un utilisateur à risque [État du risque = À risque] avec un risque faible [Niveau de risque = Faible], et cet utilisateur a été effectivement compromis. | Sélectionnez l’utilisateur, puis cliquez sur « Confirmer que l’utilisateur est compromis ». | Azure AD déplace le risque utilisateur sur Élevé [État du risque = Compromission confirmée ; Niveau de risque = Élevé] et ajoute une nouvelle détection « L’administrateur a confirmé que cet utilisateur est compromis ». | Actuellement, l’option « Confirmer que l’utilisateur est compromis » est uniquement disponible dans le rapport « Utilisateurs à risque ». <br> La détection « L’administrateur a confirmé que cet utilisateur est compromis » figure dans l’onglet « Détections de risques non liés à une connexion » dans le rapport « Utilisateurs à risque ». |
| **Utilisateur corrigé en dehors d’Azure AD Identity Protection (vrai positif + corrigé)** <br> Le rapport « Utilisateurs à risque » indique un utilisateur à risque, et j’ai corrigé par la suite l’utilisateur en dehors d’Azure AD Identity Protection. | 1. Sélectionnez l’utilisateur, puis cliquez sur « Confirmer que l’utilisateur est compromis ». (Ce processus confirme à Azure AD que l’utilisateur a été effectivement compromis.) <br> 2. Attendez que le « Niveau de risque » de l’utilisateur soit défini sur Élevé. (Ce délai d’attente laisse à Azure AD le temps nécessaire pour envoyer les commentaires ci-dessus au moteur de risque.) <br> 3. Sélectionnez l’utilisateur, puis cliquez sur « Ignorer le risque lié à l’utilisateur ». (Ce processus confirme à Azure AD que l’utilisateur n’est plus compromis.) |  Azure AD déplace le risque utilisateur sur Aucun [État du risque = Ignoré ; Niveau de risque =-] et ferme le risque sur toutes les connexions existantes présentant un risque actif. | Cliquez sur « Ignorer le risque lié à l’utilisateur » pour fermer tous les risques liés à l’utilisateur et aux connexions passées. Il est impossible d’annuler cette opération. |
| **Utilisateur non compromis (faux positif)** <br> Le rapport « Utilisateurs à risque » indique un utilisateur à risque, mais celui-ci n’est pas compromis. | Sélectionnez l’utilisateur, puis cliquez sur « Ignorer le risque lié à l’utilisateur ». (Ce processus confirme à Azure AD que l’utilisateur n’est pas compromis.) | Azure AD déplace le risque utilisateur sur Aucun [État du risque = Ignoré ; Niveau de risque =-]. | Cliquez sur « Ignorer le risque lié à l’utilisateur » pour fermer tous les risques liés à l’utilisateur et aux connexions passées. Il est impossible d’annuler cette opération. |
| Je souhaite fermer le risque utilisateur, mais je ne sais pas si l’utilisateur est compromis/sécurisé. | Sélectionnez l’utilisateur, puis cliquez sur « Ignorer le risque lié à l’utilisateur ». (Ce processus confirme à Azure AD que l’utilisateur n’est plus compromis.) | Azure AD déplace le risque utilisateur sur Aucun [État du risque = Ignoré ; Niveau de risque =-]. | Cliquez sur « Ignorer le risque lié à l’utilisateur » pour fermer tous les risques liés à l’utilisateur et aux connexions passées. Il est impossible d’annuler cette opération. Nous vous recommandons de corriger l’utilisateur en cliquant sur « Réinitialiser le mot de passe » ou de lui demander de réinitialiser/modifier en toute sécurité ses informations d’identification. |

Les commentaires sur les détections de risques utilisateur dans Azure AD Identity Protection sont traités hors connexion, et leur mise à jour peut prendre un certain temps. La colonne État de traitement du risque indique l’état actuel du traitement de vos commentaires.

![État de traitement du risque du rapport Utilisateurs à risque](./media/howto-identity-protection-risk-feedback/risky-users-provide-feedback.png)

## <a name="next-steps"></a>Étapes suivantes

- [Référence des détections de risques Azure Active Directory Identity Protection](./concept-identity-protection-risks.md)