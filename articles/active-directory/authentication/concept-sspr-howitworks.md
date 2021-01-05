---
title: Analyse approfondie de la réinitialisation de mot de passe en libre-service – Azure Active Directory
description: Comment fonctionne la réinitialisation de mot de passe en libre-service ?
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 12/07/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: e5bcc6503af5f5f685ee589ed7671b4715834fef
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779360"
---
# <a name="how-it-works-azure-ad-self-service-password-reset"></a>Fonctionnement : Réinitialisation de mot de passe en libre-service Azure AD

La réinitialisation de mot de passe en libre-service (SSPR) Azure Active Directory (Azure AD) permet aux utilisateurs de changer ou de réinitialiser leur mot de passe sans intervention d’un administrateur ou d’un agent du support technique. Si le compte d’un utilisateur est verrouillé ou si ce dernier oublie son mot de passe, il peut suivre des invites afin de se débloquer et de reprendre son travail. Cette fonctionnalité réduit les appels au support technique et la perte de productivité quand l’utilisateur ne parvient pas à se connecter à son appareil ou à une application.

> [!IMPORTANT]
> Cet article conceptuel explique à un administrateur le fonctionnement de la réinitialisation de mot de passe en libre-service. Si vous êtes un utilisateur final déjà inscrit pour la réinitialisation de mot de passe en libre-service, et si vous devez récupérer votre compte, accédez à [https://aka.ms/sspr](https://aka.ms/sspr).
>
> Si votre équipe informatique n’a pas activé la réinitialisation de votre propre mot de passe, contactez votre support technique pour obtenir une assistance supplémentaire.

## <a name="how-does-the-password-reset-process-work"></a>Fonctionnement du processus de réinitialisation de mot de passe

Un utilisateur peut réinitialiser ou changer son mot de passe à l’aide du [portail SSPR](https://aka.ms/sspr). Il doit d’abord avoir inscrit les méthodes d’authentification souhaitées. Quand un utilisateur accède au portail SSPR, la plateforme Azure prend en compte les facteurs suivants :

* Comment la page doit être localisée ?
* Le compte d’utilisateur est-il valide ?
* À quelle organisation l’utilisateur appartient-il ?
* Où le mot de passe de l’utilisateur est-il géré ?
* L’utilisateur a-t-il une licence pour utiliser la fonctionnalité ?

Quand un utilisateur sélectionne le lien **Impossible d’accéder à votre compte** à partir d’une application ou d’une page, ou qu’il accède directement à [https://aka.ms/sspr](https://passwordreset.microsoftonline.com), la langue utilisée dans le portail SSPR est basée sur les options suivantes :

* Par défaut, les paramètres régionaux du navigateur sont utilisés pour afficher le portail SSPR dans la langue appropriée. L’expérience de réinitialisation de mot de passe est localisée dans les mêmes langues que celles [prises en charge par Microsoft 365](https://support.microsoft.com/office/what-languages-is-office-available-in-26d30382-9fba-45dd-bf55-02ab03e2a7ec).
* Si vous souhaitez créer un lien vers le portail SSPR dans une langue localisée spécifique, ajoutez `?mkt=` à la fin de l’URL de réinitialisation de mot de passe avec les paramètres régionaux nécessaires.
    * Par exemple, pour spécifier les paramètres régionaux de la langue espagnole *es-us*, utilisez `?mkt=es-us` - [https://passwordreset.microsoftonline.com/?mkt=es-us](https://passwordreset.microsoftonline.com/?mkt=es-us).

Une fois le portail SSPR affiché dans la langue appropriée, l’utilisateur est invité à entrer un identifiant utilisateur et à réussir un captcha. Azure AD vérifie ensuite que l’utilisateur peut utiliser la fonctionnalité SSPR en procédant aux vérifications suivantes :

* Il vérifie que la fonctionnalité SSPR est activée pour l’utilisateur, et que celui-ci dispose d’une licence Azure AD.
  * Si la fonctionnalité SSPR n’est pas activée pour l’utilisateur ou si aucune licence ne lui a été affectée, l’utilisateur est invité à contacter son administrateur pour réinitialiser son mot de passe.
* Il vérifie que l’utilisateur a défini les méthodes d'authentification appropriées sur son compte, conformément à la stratégie de l’administrateur.
  * Si la stratégie nécessite une seule méthode, vérifiez que l’utilisateur dispose des données appropriées définies pour au moins une des méthodes d’authentification activées par la stratégie de l’administrateur.
    * Si les méthodes d’authentification ne sont pas configurées, l’utilisateur est invité à contacter l’administrateur pour réinitialiser son mot de passe.
  * Si la stratégie nécessite deux méthodes, vérifiez que l’utilisateur dispose des données appropriées définies pour au moins deux des méthodes d’authentification activées par la stratégie de l’administrateur.
    * Si les méthodes d’authentification ne sont pas configurées, l’utilisateur est invité à contacter l’administrateur pour réinitialiser son mot de passe.
  * Si un rôle d’administrateur Azure est affecté à l’utilisateur, la stratégie de mot de passe fort à deux verrous est appliquée. Pour plus d’informations, consultez [Différences en matière de stratégie de réinitialisation par l’administrateur](concept-sspr-policy.md#administrator-reset-policy-differences).
* Il vérifie si le mot de passe de l’utilisateur est géré localement, par exemple si le locataire Azure AD utilise l’authentification fédérée directe ou la synchronisation du code de hachage de mot de passe :
  * Si la réécriture SSPR est configurée et si le mot de passe de l’utilisateur est géré localement, ce dernier est autorisé à s’authentifier et à réinitialiser son mot de passe.
  * Si la réécriture SSPR n’est pas déployée et si le mot de passe de l’utilisateur est géré localement, ce dernier est invité à contacter son administrateur pour qu’il réinitialise son mot de passe.

Si toutes les vérifications précédentes sont correctement effectuées, l’utilisateur est guidé tout au long du processus de réinitialisation ou de changement de son mot de passe.

> [!NOTE]
> La SSPR peut envoyer des notifications par e-mail aux utilisateurs dans le cadre du processus de réinitialisation de mot de passe. Ces e-mails sont envoyés à l’aide du service de relais SMTP, qui fonctionne en mode actif/actif dans plusieurs régions.
>
> Les services de relais SMTP reçoivent et traitent le corps de l’e-mail, mais ne le stockent pas. Le corps de l’e-mail SSPR, qui peut éventuellement contenir des informations fournies par le client, n’est pas stocké dans les journaux du service de relais SMTP. Les journaux contiennent uniquement des métadonnées de protocole.

Pour bien démarrer avec SSPR, suivez le tutoriel suivant :

> [!div class="nextstepaction"]
> [Tutoriel : Activer SSPR (réinitialisation de mot de passe en libre-service)](tutorial-enable-sspr.md)


## <a name="require-users-to-register-when-they-sign-in"></a>Obliger les utilisateurs à s’inscrire durant la connexion

Vous pouvez activer l’option qui permet d’obliger un utilisateur à s’inscrire à SSPR, s’il se connecte à des applications via Azure AD. Ce workflow inclut les applications suivantes :

* Microsoft 365
* Portail Azure
* Volet d'accès
* Applications fédérées
* Applications personnalisées utilisant Azure AD

Quand vous ne rendez pas l’inscription obligatoire, les utilisateurs ne sont pas sollicités au moment de la connexion. Toutefois, ils peuvent s’inscrire manuellement. Ils peuvent visiter [https://aka.ms/ssprsetup](https://aka.ms/ssprsetup) ou sélectionner le lien **Réinitialiser mon mot de passe** sous l’onglet **Profil** dans le volet d’accès.

![Options d’inscription à SSPR dans le portail Azure][Registration]

> [!NOTE]
> Les utilisateurs peuvent ignorer le portail d’inscription SSPR en sélectionnant **annuler** ou en fermant la fenêtre. Toutefois, ils sont invités à s’inscrire chaque fois qu’ils se connectent jusqu’à ce qu’ils aient effectué leur inscription.
>
> L’interruption de l’inscription à SSPR ne rompt pas la connexion de l’utilisateur, s’il est déjà connecté.

## <a name="reconfirm-authentication-information"></a>Reconfirmer les informations d’authentification

Si vous souhaitez vérifier que les méthodes d’authentification sont correctes quand elles sont nécessaires pour réinitialiser ou changer un mot de passe, vous pouvez demander aux utilisateurs de confirmer leurs informations inscrites après un certain temps. Cette option est disponible uniquement si vous activez l’option **Obliger les utilisateurs à s’inscrire durant la connexion**.

Les valeurs valides pour inviter un utilisateur à confirmer ses méthodes inscrites sont comprises entre *0* et *730* jours. L’affectation de la valeur *0* signifie que les utilisateurs ne sont jamais invités à confirmer leurs informations d’authentification.

## <a name="authentication-methods"></a>Méthodes d’authentification

Quand SSPR est activé pour un utilisateur, celui-ci doit inscrire au moins une méthode d’authentification. Nous vous recommandons vivement de choisir au moins deux méthodes d’authentification pour que les utilisateurs disposent d’une plus grande flexibilité au cas où l’une des méthodes nécessaires ne serait pas accessible. Pour plus d'informations, consultez [Quelles sont les méthodes d'authentification ?](concept-authentication-methods.md).

Les méthodes d’authentification suivantes sont disponibles pour SSPR :

* Notification sur l’application mobile
* Code de l’application mobile
* E-mail
* Téléphone mobile
* Téléphone de bureau
* Questions de sécurité

Les utilisateurs peuvent uniquement réinitialiser leur mot de passe s’ils ont inscrit une méthode d’authentification que l’administrateur a activée.

> [!WARNING]
> Les rôles *Administrateur* Azure affectés aux comptes sont nécessaires pour utiliser les méthodes définies dans la section [Différences en matière de stratégie de réinitialisation par l’administrateur](concept-sspr-policy.md#administrator-reset-policy-differences).

![Sélection des méthodes d’authentification dans le portail Azure][Authentication]

### <a name="number-of-authentication-methods-required"></a>Nombre de méthodes d’authentification requises

Vous pouvez configurer le nombre de méthodes d’authentification disponibles qu’un utilisateur doit fournir pour réinitialiser ou déverrouiller son mot de passe. Cette valeur peut être définie à *1* ou *2*.

Les utilisateurs peuvent et doivent inscrire plusieurs méthodes d’authentification. Encore une fois, il est vivement recommandé aux utilisateurs d’inscrire au moins deux méthodes d’authentification pour disposer d’une plus grande flexibilité au cas où l’une des méthodes nécessaires ne serait pas accessible.

Si un utilisateur n’a pas le nombre minimal de méthodes nécessaires inscrites quand il essaie d’utiliser SSPR, une page d’erreur s’affiche et lui demande de contacter un administrateur pour qu’il réinitialise son mot de passe. Soyez prudent si vous augmentez le nombre de méthodes nécessaires en le portant à deux. En effet, s’il existe des utilisateurs déjà inscrits à SSPR, ils risquent de ne pas pouvoir utiliser la fonctionnalité. Pour plus d’informations, consultez la section [Changer les méthodes d’authentification](#change-authentication-methods).

#### <a name="mobile-app-and-sspr"></a>Application mobile et réinitialisation de mot de passe en libre-service

Quand vous utilisez une application mobile en tant que méthode de réinitialisation de mot de passe, par exemple l’application Microsoft Authenticator, tenez compte des considérations suivantes :

* Lorsque les administrateurs demandent d’utiliser une méthode de réinitialisation de mot de passe, le code de vérification est la seule option disponible.
* Quand les administrateurs imposent l’utilisation de deux méthodes de réinitialisation de mot de passe, les utilisateurs peuvent effectuer la réinitialisation à l’aide d’une notification **OU** à l’aide d’un code de vérification en plus des autres méthodes activées.

| Nombre de méthodes requises pour la réinitialisation | Une | Deux |
| :---: | :---: | :---: |
| Fonctionnalités d’application mobile disponibles | Code | Code ou notification |

Les utilisateurs ne peuvent pas inscrire leur application mobile quand ils s’inscrivent à la réinitialisation de mot de passe en libre-service depuis [https://aka.ms/ssprsetup](https://aka.ms/ssprsetup). Les utilisateurs peuvent inscrire leur application mobile sur [https://aka.ms/mfasetup](https://aka.ms/mfasetup) ou effectuer une inscription combinée des informations de sécurité sur [https://aka.ms/setupsecurityinfo](https://aka.ms/setupsecurityinfo).

> [!IMPORTANT]
> L’application Authenticator ne peut pas être la seule méthode d’authentification sélectionnée quand une seule méthode est imposée. De même, il est impossible de sélectionner l’application Authenticator et une seule méthode supplémentaire quand deux méthodes sont obligatoires.
>
> Durant la configuration de stratégies SSPR qui incluent l’application Authenticator en tant que méthode, au moins une méthode supplémentaire doit être sélectionnée quand une seule méthode est obligatoire, et au moins deux méthodes supplémentaires doivent être sélectionnées quand deux méthodes sont obligatoires.
>
> Cette exigence est due au fait que l’expérience d’inscription à SSPR n’inclut pas l’option d’inscription de l’application Authenticator. L’option d’inscription de l’application Authenticator est incluse dans la nouvelle [expérience d’inscription combinée](./concept-registration-mfa-sspr-combined.md).
>
> L’autorisation de stratégies qui utilisent uniquement l’application Authenticator (quand une seule méthode est obligatoire), ou l’application Authenticator et une seule méthode supplémentaire (quand deux méthodes sont obligatoires), peut empêcher les utilisateurs de s’inscrire à SSPR tant qu’ils ne sont pas configurés pour la nouvelle expérience d’inscription combinée.

### <a name="change-authentication-methods"></a>Changer les méthodes d’authentification

Si vous démarrez avec une stratégie qui présente une seule méthode d’authentification requise pour la réinitialisation ou le déverrouillage et que vous optez ensuite pour deux méthodes, que se passe-t-il ?

| Nombre de méthodes inscrites | Nombre de méthodes requises | Résultats |
| :---: | :---: | :---: |
| 1 ou plus | 1 | Réinitialisation ou déverrouillage **possible** |
| 1 | 2 | Réinitialisation ou déverrouillage **impossible** |
| 2 ou plus | 2 | Réinitialisation ou déverrouillage **possible** |

Le changement des méthodes d’authentification disponibles peut également entraîner des problèmes pour les utilisateurs. Si vous changez les types de méthodes d’authentification qu’un utilisateur peut employer, vous risquez d’empêcher par inadvertance les utilisateurs d’employer SSPR s’ils ne disposent pas de la quantité minimale de données.

Examinez l’exemple de scénario suivant :

1. La stratégie d’origine est configurée avec deux méthodes d’authentification requises. Elle utilise uniquement le numéro de téléphone professionnel et les questions de sécurité.
1. L’administrateur change la stratégie pour ne plus utiliser les questions de sécurité, mais permet l’utilisation d’un téléphone mobile et d’une autre adresse de messagerie.
1. Désormais, les utilisateurs dont les champs de téléphone mobile ou d’adresse e-mail de secours ne sont pas renseignés ne peuvent pas réinitialiser leur mot de passe.

## <a name="notifications"></a>Notifications

Pour améliorer l’accès aux événements relatifs aux mots de passe, SSPR vous permet de configurer des notifications destinées aux utilisateurs et aux administrateurs d’identité.

### <a name="notify-users-on-password-resets"></a>Notifier les utilisateurs lors des réinitialisations de mot de passe ?

Si cette option a la valeur **Oui**, les utilisateurs qui réinitialisent leur mot de passe reçoivent une notification par e-mail les avertissant que leur mot de passe a été changé. L’e-mail est envoyé via le portail SSPR à leur adresse e-mail principale et leur adresse e-mail de secours stockées dans Azure AD. Personne d’autre n’est informé de l’événement de réinitialisation.

### <a name="notify-all-admins-when-other-admins-reset-their-passwords"></a>Notifier tous les administrateurs quand d’autres administrateurs réinitialisent leur mot de passe ?

Si cette option a la valeur **Oui**, tous les autres administrateurs Azure reçoivent un e-mail à leur adresse e-mail principale stockée dans Azure AD. Cet e-mail les informe qu’un autre administrateur a changé son mot de passe à l’aide de SSPR.

Examinez l’exemple de scénario suivant :

* quatre administrateurs font partie d’un environnement.
* L’administrateur *A* réinitialise leur mot de passe à l’aide de SSPR.
* Les administrateurs *B*, *C* et *D* reçoivent un e-mail les informant de la réinitialisation de mot de passe.

## <a name="on-premises-integration"></a>Intégration locale

Si vous disposez d’un environnement hybride, vous pouvez configurer Azure AD Connect pour réécrire les événements de changement de mot de passe d’Azure AD vers un annuaire local.

![La validation de la réécriture du mot de passe est activée et opérationnelle][Writeback]

Azure AD vérifie votre connectivité hybride actuelle et fournit l’un des messages suivants dans le portail Azure :

* Votre client de réécriture local est opérationnel.
* Azure AD Connect est en ligne et connecté à votre client de réécriture local. Cependant, il semble que la version installée d’Azure AD Connect est obsolète. Pensez à [mettre à niveau Azure AD Connect](../hybrid/how-to-upgrade-previous-version.md) pour vous assurer que vous disposez des dernières fonctionnalités de connectivité et des correctifs de bogues importants.
* Malheureusement, nous ne pouvons pas vérifier l’état de votre client de réécriture local, car la version installée d’Azure AD Connect est obsolète. [Mettez à niveau Azure AD Connect](../hybrid/how-to-upgrade-previous-version.md) pour être en mesure de vérifier l’état de votre connexion.
* Malheureusement, nous ne sommes pas en mesure de nous connecter à votre client d’écriture différée local pour le moment. [Résolvez les problèmes avec Azure AD Connect](./troubleshoot-sspr-writeback.md) pour restaurer la connexion.
* Malheureusement, nous ne pouvons pas nous connecter à votre client de réécriture local, car la réécriture du mot de passe n’a pas été configurée correctement. [Configurez la réécriture du mot de passe](./tutorial-enable-sspr-writeback.md) pour restaurer la connexion.
* Malheureusement, nous ne sommes pas en mesure de nous connecter à votre client d’écriture différée local pour le moment. Cela peut être dû à des problèmes temporaires de notre côté. Si le problème persiste, [résolvez les problèmes avec Azure AD Connect](./troubleshoot-sspr-writeback.md) pour restaurer la connexion.

Pour bien démarrer avec la réécriture SSPR, suivez le tutoriel suivant :

> [!div class="nextstepaction"]
> [Tutoriel : Activer la réécriture SSPR (réinitialisation de mot de passe en libre-service)](./tutorial-enable-sspr-writeback.md)

### <a name="write-back-passwords-to-your-on-premises-directory"></a>Réécriture du mot de passe dans votre répertoire local

Vous pouvez activer la réécriture du mot de passe à l’aide du portail Azure. Vous pouvez également désactiver temporairement la réécriture du mot de passe sans avoir à reconfigurer Azure AD Connect.

* Si l’option a la valeur **Oui**, la réécriture est activée. Les utilisateurs fédérés, ou ceux qui ont recours à l’authentification directe ou à la synchronisation du code de hachage de mot de passe peuvent réinitialiser leurs mots de passe.
* Si l’option a la valeur **Non**, la réécriture est désactivée. Les utilisateurs fédérés, ou ceux qui ont recours à l’authentification directe ou à la synchronisation du code de hachage de mot de passe ne peuvent pas réinitialiser leurs mots de passe.

### <a name="allow-users-to-unlock-accounts-without-resetting-their-password"></a>Autoriser les utilisateurs à déverrouiller les comptes sans réinitialiser leur mot de passe

Par défaut, Azure AD déverrouille les comptes quand il effectue une réinitialisation de mot de passe. Pour plus de flexibilité, vous pouvez choisir d’autoriser les utilisateurs à déverrouiller leurs comptes locaux sans avoir à réinitialiser leur mot de passe. Utilisez ce paramètre pour séparer ces deux opérations.

* Si la valeur est **Oui**, les utilisateurs peuvent réinitialiser leur mot de passe et déverrouiller leur compte. Ils peuvent également déverrouiller leur compte sans devoir réinitialiser le mot de passe.
* Si la valeur est **Non**, les utilisateurs peuvent uniquement effectuer une opération combinée qui comprend la réinitialisation de mot de passe et le déverrouillage de compte.

### <a name="on-premises-active-directory-password-filters"></a>Filtres de mot de passe Active Directory locaux

SSPR effectue l’équivalent d’une réinitialisation de mot de passe lancée par l’administrateur dans Active Directory. Si vous utilisez un filtre de mot de passe tiers afin d’appliquer des règles de mot de passe personnalisées, et si vous avez besoin que ce filtre de mot de passe soit vérifié au cours de la réinitialisation de mot de passe en libre-service Azure AD, vérifiez que la solution de filtre de mot de passe tiers est configurée pour le scénario de réinitialisation de mot de passe administrateur. La [protection par mot de passe Azure AD pour Active Directory Domain Services](concept-password-ban-bad-on-premises.md) est prise en charge par défaut.

## <a name="password-reset-for-b2b-users"></a>Réinitialisation de mot de passe pour les utilisateurs B2B

La réinitialisation et la modification du mot de passe sont totalement prises en charge sur toutes les configurations B2B. La réinitialisation du mot de passe utilisateur B2B est prise en charge dans les trois cas suivants :

* **Utilisateurs d’une organisation partenaire disposant d’un locataire Azure AD** : si l’organisation avec laquelle vous avez un partenariat dispose d’un locataire Azure AD, nous respectons les stratégies de réinitialisation de mot de passe activées sur ce locataire. Pour que la réinitialisation de mot de passe fonctionne, l’organisation partenaire doit simplement vérifier qu’Azure AD SSPR est activé. Cela n’entraîne aucuns frais supplémentaires pour les clients Microsoft 365.
* **Utilisateurs qui s’inscrivent par le biais de l’inscription en libre-service** : si l’organisation avec laquelle vous avez un partenariat a utilisé la fonctionnalité d’[inscription en libre-service](../enterprise-users/directory-self-service-signup.md) pour accéder à un locataire, nous l’autorisons à réinitialiser le mot de passe en indiquant l’adresse e-mail qu’elle a utilisée pour l’inscription.
* **Utilisateurs B2B** : tous les utilisateurs B2B créés à l’aide des nouvelles [fonctionnalités B2B d’Azure AD](../external-identities/what-is-b2b.md) peuvent également réinitialiser leur mot de passe à l’aide de l’adresse e-mail indiquée durant l’inscription.

Pour tester ce scénario, accédez à https://passwordreset.microsoftonline.com avec l’un de ces utilisateurs partenaires. Si ces utilisateurs disposent d’une autre adresse de messagerie ou d’un e-mail d’authentification, la réinitialisation du mot de passe fonctionne comme prévu.

> [!NOTE]
> Les comptes Microsoft qui disposent d’un accès invité à votre locataire Azure AD, par exemple les comptes Hotmail.com, Outlook.com ou correspondant à d’autres adresses e-mail personnelles, ne peuvent pas utiliser Azure AD SSPR. Les utilisateurs de ces comptes doivent réinitialiser leur mot de passe à l’aide des informations fournies dans l’article [Lorsque vous n’avez pas accès à votre compte Microsoft](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant).

## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer avec SSPR, suivez le tutoriel suivant :

> [!div class="nextstepaction"]
> [Tutoriel : Activer SSPR (réinitialisation de mot de passe en libre-service)](tutorial-enable-sspr.md)

Les liens suivants fournissent des informations supplémentaires sur la réinitialisation de mot de passe à l’aide d’Azure AD :

[Authentication]: ./media/concept-sspr-howitworks/manage-authentication-methods-for-password-reset.png "Méthodes d’authentification disponibles dans Azure Active Directory et quantité requise"
[Registration]: ./media/concept-sspr-howitworks/configure-registration-options.png "Configurer les options d’inscription SSPR dans le portail Azure"
[Writeback]: ./media/concept-sspr-howitworks/on-premises-integration.png "Intégration locale pour SSPR dans le portail Azure"