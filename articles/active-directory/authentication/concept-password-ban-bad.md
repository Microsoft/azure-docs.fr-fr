---
title: Protection de mots de passe dans Azure Active Directory
description: Apprenez à interdire de manière dynamique les mots de passe faibles de votre environnement à l’aide de la fonctionnalité Protection de mots de passe d’Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/16/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: b019c12103a0324e21ef7c06b0c7a5d7fe14e6ec
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743885"
---
# <a name="eliminate-bad-passwords-using-azure-active-directory-password-protection"></a>Éliminer les mauvais mots de passe à l’aide de Protection de mots de passe d’Azure Active Directory

De nombreux conseils de sécurité recommandent de ne pas utiliser le même mot de passe à plusieurs endroits, de le rendre complexe et d’éviter les mots de passe simples comme *Motdepasse123*. Vous pouvez fournir à vos utilisateurs des [conseils sur la manière de choisir des mots de passe](https://www.microsoft.com/research/publication/password-guidance), mais les mots de passe faibles ou non sécurisés restent souvent utilisés. Protection de mots de passe d’Azure AD détecte et bloque les mots de passe faibles connus et leurs variantes, mais peut aussi bloquer d’autres termes faibles propres à votre organisation.

Grâce à Protection de mots de passe d’Azure AD, les listes globales de mots de passe interdits par défaut sont automatiquement appliquées à tous les utilisateurs dans un locataire Azure AD. Pour répondre aux besoins de votre entreprise et de votre sécurité, vous pouvez définir des entrées dans une liste personnalisée de mots de passe interdits. Lorsque les utilisateurs modifient ou réinitialisent leurs mots de passe, ces listes de mots de passe interdits sont vérifiées pour imposer l’utilisation de mots de passe forts.

Vous devez utiliser des fonctionnalités supplémentaires comme [Azure AD Multi-Factor Authentication](concept-mfa-howitworks.md), et éviter de vous fier uniquement aux mots de passe forts imposés par Azure AD Password Protection. Pour plus d’informations sur l’utilisation de plusieurs couches de sécurité pour vos événements de connexion, consultez [Your Pa$$word doesn’t matter](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Your-Pa-word-doesn-t-matter/ba-p/731984).

> [!IMPORTANT]
> Cet article conceptuel explique à un administrateur le fonctionnement de Protection de mots de passe d’Azure AD. Si vous êtes un utilisateur final déjà inscrit pour la réinitialisation de mot de passe en libre-service, et si vous devez récupérer votre compte, accédez à [https://aka.ms/sspr](https://aka.ms/sspr).
>
> Si votre équipe informatique n’a pas activé la réinitialisation de votre propre mot de passe, contactez votre support technique pour obtenir une assistance supplémentaire.

## <a name="global-banned-password-list"></a>Liste globale de mots de passe interdits

L’équipe Azure AD Identity Protection analyse en permanence les données de télémétrie de sécurité Azure AD à la recherche de mots de passe faibles couramment utilisés ou compromis. Plus précisément, l’analyse recherche des termes basiques qui sont souvent utilisés comme base pour les mots de passe faibles. Si des termes faibles sont détectés, ils sont ajoutés à la *liste globale de mots de passe interdits*. Le contenu de la liste globale de mots de passe interdits n’est pas basé sur une source de données externe, mais sur les résultats de la télémétrie et de l’analyse de la sécurité d’Azure AD.

Lorsqu’un mot de passe est modifié ou réinitialisé pour tout utilisateur d’un locataire d’Azure AD, la version actuelle de la liste globale de mots de passe interdits est utilisée pour vérifier la force du mot de passe. Ce contrôle de validation permet à tous les clients Azure AD de disposer de mots de passe plus forts.

La liste globale de mots de passe interdits est automatiquement appliquée à tous les utilisateurs dans un locataire Azure AD. Il n’y a rien à activer ni à configurer, et elle ne peut pas être désactivée. Cette liste globale de mots de passe interdits est appliquée aux utilisateurs lorsqu’ils modifient ou réinitialisent leur propre mot de passe via Azure AD.

> [!NOTE]
> Les cybercriminels utilisent également des stratégies similaires dans leurs attaques pour identifier les mots de passe faibles courants et leurs variations. Afin d’améliorer la sécurité, Microsoft ne publie pas le contenu de la liste globale de mots de passe interdits.

## <a name="custom-banned-password-list"></a>Liste personnalisée de mots de passe interdits

Certaines organisations souhaitent améliorer leur sécurité et ajouter leurs propres personnalisations en plus de la liste globale de mots de passe interdits. Pour ajouter vos propres entrées, vous pouvez utiliser la *liste personnalisée de mots de passe interdits*. Les termes ajoutés à la liste personnalisée de mots de passe interdits doivent être axés sur des termes propres à l’organisation. En voici quelques exemples :

- Noms de marques
- Noms de produits
- Lieux, par exemple le siège social de l’entreprise
- Termes internes spécifiques à l’entreprise
- Abréviations ayant une signification spécifique dans l’entreprise

Une fois ajoutés à la liste personnalisée de mots de passe interdits, ces termes sont combinés à ceux de la liste globale de mots de passe interdits. Les événements de modification ou de réinitialisation de mot de passe sont ensuite validés par rapport à l’ensemble combiné de ces listes de mots de passe interdits.

> [!NOTE]
> La liste personnalisée de mots de passe interdits est limitée à un maximum de 1 000 termes. Elle n’est pas conçue pour bloquer des listes de mots de passe extrêmement longues.
>
> Pour tirer pleinement parti des avantages de la liste personnalisée de mots de passe interdits, vous devez d’abord comprendre [comment les mots de passe sont évalués](#how-are-passwords-evaluated) avant d’ajouter des termes à la liste personnalisée de mots de passe interdits. Cette approche vous permet de détecter et de bloquer efficacement un grand nombre de mots de passe faibles et leurs variantes.

![Modifier la liste de mots de passe interdits personnalisée sous Méthodes d’authentification](./media/tutorial-configure-custom-password-protection/enable-configure-custom-banned-passwords-cropped.png)

Prenons l’exemple d’un client nommé *Contoso*. La société est basée à Londres et crée un produit nommé *Widget*. Pour ce client fictif, il serait peu productif et peu sûr de chercher à bloquer des variantes spécifiques de ces termes, par exemple :

- "Contoso!1"
- "Contoso@London"
- "ContosoWidget"
- "!Contoso"
- "LondonHQ"

Au lieu de cela, il serait nettement plus efficace et sécurisé de bloquer uniquement les termes de base clés, comme les exemples suivants :

- "Contoso"
- "Londres"
- "Widget"

L’algorithme de validation des mots de passe bloque alors automatiquement les variantes et combinaisons faibles.

Pour commencer à utiliser une liste personnalisée de mots de passe interdits, suivez le didacticiel suivant :

> [!div class="nextstepaction"]
> [Tutoriel : Configurer les mots de passe interdits personnalisés](tutorial-configure-custom-password-protection.md)

## <a name="password-spray-attacks-and-third-party-compromised-password-lists"></a>Attaques par pulvérisation de mot de passe et listes de mots de passe compromis tierces

Protection de mots de passe d’Azure AD vous aide à vous défendre contre les attaques par pulvérisation de mot de passe. La plupart des attaques par pulvérisation de mot de passe ne tentent pas d’attaquer un compte donné plus de quelques fois. Ce comportement augmente la probabilité de détection, soit par le verrouillage des comptes, soit par d’autres moyens.

Au lieu de cela, la majorité des attaques par pulvérisation de mots de passe ne soumettent qu’un petit nombre des mots de passe les plus faibles connus à chacun des comptes d’une entreprise. Cette technique permet à l’attaquant de repérer rapidement un compte facile à compromettre et d’éviter les seuils de détection potentiels.

Protection de mots de passe d’Azure AD bloque efficacement tous les mots de passe faibles connus susceptibles d’être utilisés dans des attaques par pulvérisation de mot de passe. Cette protection est basée sur les données de télémétrie de sécurité du monde réel réunies par Azure AD afin de créer la liste globale de mots de passe interdits.

Certains sites web tiers qui énumèrent des millions de mots de passe compromis à la suite de violations de sécurité connues publiquement. Il est courant que les produits tiers de validation des mots de passe s’appuient sur une comparaison en force brute par rapport à ces millions de mots de passe. Toutefois, ces techniques ne constituent pas le meilleur moyen d’améliorer la force globale des mots de passe en raison des stratégies généralement utilisées par les attaquants par pulvérisation de mot de passe.

> [!NOTE]
> La liste globale de mots de passe interdits ne repose pas sur des sources de données tierces, pas même sur des listes de mots de passe compromis.

Bien que la liste globale de mots de passe interdits soit petite par rapport à certaines listes groupées tierces, elle provient de la télémétrie de sécurité du monde réel sur de véritables attaques par pulvérisation de mot de passe. Cette approche améliore la sécurité et l’efficacité globales, et l’algorithme de validation des mots de passe utilise également des techniques intelligentes de correspondance partielle. Par conséquent, Protection de mots de passe d’Azure AD détecte et bloque efficacement des millions de mots de passe faibles connus et empêche leur utilisation dans votre entreprise.

## <a name="on-premises-hybrid-scenarios"></a>Scénarios hybrides locaux

De nombreuses organisations disposent d’un modèle d’identité hybride qui comprend des environnements Active Directory Domain Services (AD DS) locaux. Pour étendre les avantages de sécurité de Protection de mots de passe d’Azure AD dans votre environnement AD DS, vous pouvez installer des composants sur vos serveurs locaux. Ces agents requièrent des événements de modification de mot de passe dans l’environnement AD DS local pour se conformer à la même stratégie de mot de passe que dans Azure AD.

Pour plus d’informations, consultez [Appliquer Protection de mots de passe d’Azure AD pour AD DS](concept-password-ban-bad-on-premises.md).

## <a name="how-are-passwords-evaluated"></a>Évaluation des mots de passe

Lorsqu’un utilisateur change ou réinitialise son mot de passe, la force et la complexité du nouveau mot de passe sont vérifiées par rapport à la liste combinée des termes des listes globale et personnalisée de mots de passe interdits.

Même si le mot de passe d’un utilisateur contient un mot interdit, il peut être accepté s’il est par ailleurs suffisamment fort dans son ensemble. Un mot de passe nouvellement configuré passe par les étapes suivantes pour évaluer sa force globale et déterminer s’il doit être accepté ou rejeté :

### <a name="step-1-normalization"></a>Étape 1 : Normalisation

Un nouveau mot de passe est d'abord soumis à un processus de normalisation. Cette technique permet de mapper un petit ensemble de mots de passe interdits à un ensemble beaucoup plus large de mots de passe potentiellement faibles.

La normalisation comporte les deux parties suivantes :

* Toutes les majuscules sont remplacées par des minuscules.
* Ensuite, des substitutions de caractères communs sont effectuées, comme dans l’exemple suivant :

   | Lettre d'origine | Lettre de substitution |
   |-----------------|--------------------|
   | 0               | o                  |
   | 1               | l                  |
   | $               | s                  |
   | \@              | a                  |

Prenons l’exemple suivant :

* Le mot de passe « blank » est interdit.
* Un utilisateur tente de modifier son mot de passe en « Bl@nK ».
* Même si « Bl@nk » n’est pas interdit, le processus de normalisation convertit ce mot de passe en « blank ».
* Ce mot de passe est rejeté.

### <a name="step-2-check-if-password-is-considered-banned"></a>Étape 2 : Déterminer si le mot de passe est interdit

Un mot de passe est ensuite examiné pour détecter d’autres comportements correspondants, et un score est généré. Ce score final détermine si la demande de modification du mot de passe est acceptée ou rejetée.

#### <a name="fuzzy-matching-behavior"></a>Correspondance approximative

La correspondance approximative est utilisée sur le mot de passe normalisé pour déterminer s'il contient un mot de passe figurant dans la liste globale ou la liste personnalisée des mots de passe interdits. Le processus de correspondance repose sur une distance de modification d'une (1) comparaison.

Prenons l’exemple suivant :

* Le mot de passe « abcdef » est interdit.
* Un utilisateur tente de remplacer son mot de passe par l’un des éléments suivants :

   * « abcdeg » : *dernier caractère modifié de « f » en « g »*
   * « abcdefg » : *« g » ajouté à la fin*
   * « abcde » : *« f » final a été supprimé de la fin*

* Aucun des mots de passe ci-dessus ne correspond précisément au mot de passe interdit « abcdef ».

    Cependant, comme chaque exemple ont une distance de modification de 1 par rapport au terme interdit « abcdef », ils sont tous considérés comme correspondant à « abcdef ».
* Ces mots de passe sont rejetés.

#### <a name="substring-matching-on-specific-terms"></a>Correspondance de sous-chaîne (sur des termes spécifiques)

La correspondance de sous-chaîne est utilisée sur le mot de passe normalisé pour vérifier le nom et le prénom de l’utilisateur, ainsi que le nom du locataire. La correspondance des noms de locataire n’est pas effectuée lors de la validation des mots de passe sur un contrôleur de domaine AD DS pour les scénarios hybrides locaux.

> [!IMPORTANT]
> La correspondance de sous-chaînes est uniquement appliquée aux noms, et à d’autres termes, qui ont au moins quatre caractères.

Prenons l’exemple suivant :

* Un utilisateur nommé Poll qui souhaite réinitialiser son mot de passe en « p0LL23fb ».
* Après la normalisation, ce mot de passe devient « poll23fb ».
* La correspondance de sous-chaînes détecte que le mot de passe contient le prénom de l’utilisateur « Poll ».
* Même si « poll23fb » ne figure pas spécifiquement sur l’une ou l’autre des listes de mots de passe interdits, la correspondance de sous-chaînes a trouvé « Poll » dans le mot de passe.
* Ce mot de passe est rejeté.

#### <a name="score-calculation"></a>Calcul du score

L'étape suivante consiste à détecter toutes les occurrences de mots de passe interdits dans le nouveau mot de passe normalisé de l'utilisateur. Des points sont attribués en fonction des critères suivants :

1. Chaque mot de passe interdit trouvé dans le mot de passe d’un utilisateur reçoit un point.
1. Chaque caractère unique restant reçoit un point.
1. Un mot de passe doit recevoir au moins cinq (5) points pour être accepté.

Pour les deux exemples de scénarios suivants, nous partons du principe que Contoso utilise Protection de mots de passe d’Azure AD et que « contoso » figure dans sa liste personnalisée de mots de passe interdits. Nous supposons également que « blank » figure sur la liste globale.

Dans l’exemple de scénario suivant, un utilisateur remplace son mot de passe par « C0ntos0Blank12 » :

* Après la normalisation, ce mot de passe devient « contosoblank12 ».
* Le processus de correspondance détecte que ce mot de passe contient deux mots interdits : « contoso » et « blank ».
* Ce mot de passe reçoit alors le score suivant :

    *[contoso] + [blank] + [1] + [2] = 4 points*

* Étant donné que ce mot de passe reçoit un score inférieur à cinq (5) points, il est rejeté.

Prenons un exemple légèrement différent pour montrer comment la complexité supplémentaire d’un mot de passe peut permettre d’obtenir le nombre de points requis pour être accepté. Dans l’exemple de scénario suivant, un utilisateur remplace son mot de passe par « ContoS0Bl@nkf9! » :

* Après la normalisation, ce mot de passe devient « contosoblankf9! ».
* Le processus de correspondance détecte que ce mot de passe contient deux mots interdits : « contoso » et « blank ».
* Ce mot de passe reçoit alors le score suivant :

    *[contoso] + [blank] + [f] + [9] + [!] = 5 points*

* Ce mot de passe recevant un score d’au moins cinq (5) points, il est accepté.

> [!IMPORTANT]
> La liste globale et l’algorithme des mots de passe interdits peuvent être modifiés à tout moment dans Azure, en fonction des analyses et des recherches de sécurité en cours.
>
> Pour le service d’agent DC local dans des scénarios hybrides, les algorithmes mis à jour ne prennent effet qu’après la mise à niveau du logiciel de l’agent DC.

## <a name="what-do-users-see"></a>Ce que voient les utilisateurs

Lorsqu’un utilisateur tente de réinitialiser un mot de passe pour le remplacer par un mot de passe interdit, le message d’erreur suivant s’affiche :

*« Malheureusement, votre mot de passe contient un mot, une expression ou un modèle qui le rend facile à deviner. Réessayez avec un autre mot de passe. »*

## <a name="license-requirements"></a>Conditions de licence :

| Utilisateurs | Protection de mots de passe d’Azure AD avec liste globale de mots de passe interdits | Protection de mots de passe d’Azure AD avec liste personnalisée de mots de passe interdits|
|-------------------------------------------|---------------------------|---------------------------|
| Utilisateurs du cloud uniquement                          | Azure AD Gratuit             | Azure AD Premium P1 ou P2 |
| Utilisateurs synchronisés à partir d’une instance AD DS locale | Azure AD Premium P1 ou P2 | Azure AD Premium P1 ou P2 |

> [!NOTE]
> Les utilisateurs AD DS locaux qui ne sont pas synchronisés à Azure AD bénéficient également de Protection de mots de passe d’Azure AD en fonction des licences existantes pour les utilisateurs synchronisés.

Vous trouverez des informations de licence supplémentaires, notamment les prix, sur le [site de tarification Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="next-steps"></a>Étapes suivantes

Pour commencer à utiliser une liste personnalisée de mots de passe interdits, suivez le didacticiel suivant :

> [!div class="nextstepaction"]
> [Tutoriel : Configurer les mots de passe interdits personnalisés](tutorial-configure-custom-password-protection.md)

Vous pouvez également [activer Protection de mots de passe d’Azure AD localement](howto-password-ban-bad-on-premises-deploy.md).
