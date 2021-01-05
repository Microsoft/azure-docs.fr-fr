---
title: Méthode d’authentification par questions de sécurité – Azure Active Directory
description: Apprenez-en davantage sur les questions de sécurité dans Azure Active Directory pour contribuer à l’amélioration et à la sécurisation des événements de connexion
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 09/02/2020
ms.author: justinha
author: justinha
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7229dabd690e5932fbd297992e09782eda85a002
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96744140"
---
# <a name="authentication-methods-in-azure-active-directory---security-questions"></a>Méthodes d’authentification dans Azure Active Directory – Question de sécurité

Les questions de sécurité ne sont pas utilisées comme méthode d’authentification durant un événement de connexion. Au lieu de cela, vous pouvez utiliser les questions de sécurité durant le processus de réinitialisation de mot de passe en libre-service (SSPR) pour confirmer votre identité. Les comptes administrateurs ne peuvent pas utiliser les questions de sécurité comme méthode de vérification avec SSPR.

Quand les utilisateurs s’enregistrent pour SSPR, ils sont invités à choisir les méthodes d’authentification à utiliser. S’ils choisissent d’utiliser des questions de sécurité, ils les choisissent parmi un ensemble de questions et fournissent leurs propres réponses.

![Capture d’écran du Portail Azure affichant les méthodes d’authentification et les options de question de sécurité](media/concept-authentication-methods/security-questions-authentication-method.png)

> [!NOTE]
> Les questions de sécurité sont stockées de façon privée et sécurisée dans un objet utilisateur du répertoire et ne peuvent être posées qu’aux utilisateurs lors de l’inscription. L’administrateur ne peut pas lire ni modifier les questions ou les réponses des utilisateurs.

Les questions de sécurité peuvent être moins sécurisées que d’autres méthodes, car certaines personnes peuvent connaître les réponses aux questions d’un autre utilisateur. Si vous utilisez des questions de sécurité avec SSPR, il est recommandé de les utiliser conjointement avec une autre méthode. Un utilisateur peut être invité à utiliser l’application Microsoft Authenticator ou l’authentification par téléphone pour vérifier son identité pendant le processus SSPR, il peut également avoir à choisir des questions de sécurité, uniquement s’il n’a pas de téléphone ou d’appareil enregistré avec lui.

## <a name="predefined-questions"></a>Questions prédéfinies

Les questions de sécurité prédéfinies suivantes peuvent être utilisées comme méthode de vérification avec SSPR. Toutes ces questions de sécurité sont traduites et localisées dans l’ensemble des langues de Microsoft 365 en fonction des paramètres régionaux du navigateur de l’utilisateur :

* Dans quelle ville avez-vous rencontré votre premier(ère) époux/épouse ou compagnon/compagne ?
* Dans quelle ville vos parents se sont-ils rencontrés ?
* Dans quelle ville vit votre frère ou sœur le/la plus proche ?
* Dans quelle ville votre père est-il né ?
* Dans quelle ville était votre premier travail ?
* Dans quelle ville votre mère est-elle née ?
* Dans quelle ville étiez-vous pour le Nouvel an de l’année 2000 ?
* Quel était le nom de votre professeur préféré au lycée ?
* Quel est le nom d’un établissement d’enseignement supérieur auquel vous avez postulé mais auquel vous n’êtes finalement pas allé ?
* Quel est le nom de l’endroit où vous avez organisé la réception de votre premier mariage ?
* Quel est le deuxième prénom de votre père ?
* Quel est votre aliment préféré ?
* Quels sont les nom et prénom de votre grand-mère maternelle ?
* Quel est le deuxième prénom de votre mère ?
* Quel est le mois et l’année de naissance de votre frère/sœur aîné(e) ? (par exemple, novembre 1985)
* Quel est le deuxième prénom de votre frère/sœur aîné(e) ?
* Quels sont les nom et prénom de votre grand-père paternel ?
* Quel est le deuxième prénom de votre frère/sœur cadet(te) ?
* Où avez-vous effectué votre dernière année de l’école primaire ?
* Quels étaient les nom et prénom de votre meilleur ami quand vous étiez enfant ?
* Quels étaient les nom et prénom de votre premier(ère) petit(e) ami(e) ?
* Quel était le nom de votre instituteur préféré à l’école primaire ?
* Quels étaient la marque et le modèle de votre première voiture ou moto ?
* Quel était le nom de votre toute première école ?
* Quel était le nom de l’hôpital ou de la clinique où vous êtes né ?
* Quel était le nom de la rue de la première maison de votre enfance ?
* Quel était le nom de votre héros d’enfance ?
* Quel était le nom de votre peluche préférée ?
* Quel était le nom de votre premier animal domestique ?
* Quel était votre surnom quand vous étiez enfant ?
* Quel était votre sport préféré au lycée ?
* Quel a été votre premier travail ?
* Quels étaient les quatre derniers chiffres de votre numéro de téléphone quand vous étiez enfant ?
* Quand vous étiez petit, qu’auriez-vous voulu être ?
* Quelle est la personne la plus célèbre que vous ayez jamais rencontrée ?

## <a name="custom-security-questions"></a>Questions de sécurité personnalisées

Pour plus de flexibilité, vous pouvez définir vos questions de sécurité personnalisées. La longueur maximale d’une question de sécurité personnalisée est de 200 caractères.

Les questions de sécurité personnalisées ne sont pas automatiquement localisées, comme avec les questions de sécurité par défaut. Toutes les questions personnalisées s’affichent dans la même langue que celle dans laquelle elles sont entrées dans l’interface utilisateur d’administration, même si les paramètres régionaux du navigateur de l’utilisateur sont différents. Si vous avez besoin de questions localisées, vous devez utiliser les questions prédéfinies.

## <a name="security-question-requirements"></a>Conditions relatives aux questions de sécurité

Pour les questions de sécurité par défaut et personnalisées, les exigences et limitations suivantes s’appliquent :

* La limite minimale du nombre de caractères par réponse est de trois caractères.
* La limite maximale du nombre de caractères par réponse est de 40 caractères.
* L’utilisateur ne peut pas répondre plusieurs fois à la même question.
* L’utilisateur ne peut pas fournir la même réponse à plusieurs questions.
* N’importe quel jeu de caractères peut être utilisé pour définir les questions et les réponses, y compris les caractères Unicode.
* Le nombre de questions défini doit être supérieur ou égal au nombre de questions requises pour l’inscription.

## <a name="next-steps"></a>Étapes suivantes

Pour commencer, consultez le [tutoriel sur la réinitialisation du mot de passe en libre-service][tutorial-sspr].

Pour en savoir plus sur les concepts de SSPR, consultez [Fonctionnement de la réinitialisation de mot de passe en libre-service dans Azure AD][concept-sspr].

Apprenez-en plus sur la configuration des méthodes d’authentification à l’aide de l’[API REST Microsoft Graph bêta](/graph/api/resources/authenticationmethods-overview?view=graph-rest-beta).

<!-- INTERNAL LINKS -->
[tutorial-sspr]: tutorial-enable-sspr.md
[concept-sspr]: concept-sspr-howitworks.md
