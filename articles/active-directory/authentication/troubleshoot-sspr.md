---
title: Résoudre les problèmes relatifs à la réinitialisation de mot de passe libre-service - Azure Active Directory
description: Découvrir comment résoudre les problèmes courants et suivre les étapes de résolution pour la réinitialisation de mot de passe en libre-service dans Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: troubleshooting
ms.date: 08/26/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7099de23b825fa7af203dff9696bb53e23a099c8
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741114"
---
# <a name="troubleshoot-self-service-password-reset-in-azure-active-directory"></a>Résoudre les problèmes relatifs à la réinitialisation de mot de passe en libre-service dans Azure Active Directory

La réinitialisation de mot de passe en libre-service (SSPR) Azure Active Directory (Azure AD) permet aux utilisateurs de réinitialiser leur mot de passe dans le cloud.

Si vous rencontrez des problèmes avec SSPR, les étapes de résolution des problèmes et les erreurs courantes suivantes peuvent vous aider. Si vous ne trouvez pas la réponse à votre problème, [nos équipes du support technique sont toujours disponibles](#contact-microsoft-support) pour vous aider davantage.

## <a name="sspr-configuration-in-the-azure-portal"></a>Configuration de la solution SSPR sur le portail Azure

Si vous rencontrez des problèmes d’affichage ou de configuration des options SSPR sur le portail Azure, consultez les étapes de dépannage suivantes :

### <a name="i-dont-see-the-password-reset-section-under-azure-ad-in-the-azure-portal"></a>Je ne vois pas la section **Réinitialisation de mot de passe** sous Azure AD dans le portail Azure.

Vous ne voyez pas l’option de menu **Réinitialisation de mot de passe** si vous n’avez pas de licence Azure AD affectée à l’administrateur qui effectue l’opération.

Pour l’attribution d’une licence au compte administrateur en question, suivez les étapes pour [Affecter, vérifier et résoudre les problèmes des licences](../enterprise-users/licensing-groups-assign.md#step-1-assign-the-required-licenses).

### <a name="i-dont-see-a-particular-configuration-option"></a>Je ne vois pas d’option de configuration spécifique.

De nombreux éléments de l’interface utilisateur sont masqués jusqu’à ce qu’ils soient nécessaires. Assurez-vous que l’option est activée avant de rechercher des options de configuration particulières.

### <a name="i-dont-see-the-on-premises-integration-tab"></a>Je ne vois pas l’onglet **Intégration locale**.

La réécriture du mot de passe en local n’est visible que si vous avez téléchargé Azure AD Connect et configuré la fonctionnalité.

Pour plus d’informations, consultez [Bien démarrer avec Azure AD Connect](../hybrid/how-to-connect-install-express.md).

## <a name="sspr-reporting"></a>Rapports SSPR

Si vous rencontrez des problèmes avec les rapports SSPR sur le portail Azure, passez en revue les étapes de dépannage suivantes :

### <a name="i-dont-see-any-password-management-activity-types-in-the-self-service-password-management-audit-event-category"></a>Aucun type d’activité de gestion des mots de passe ne s’affiche dans la catégorie d’événement d’audit **Gestion des mots de passe en libre-service**.

Cela peut se produire si vous n’avez pas de licence Azure AD affectée à l’administrateur qui effectue l’opération.

Pour l’attribution d’une licence au compte administrateur en question, suivez les étapes pour [Affecter, vérifier et résoudre les problèmes des licences](../enterprise-users/licensing-groups-assign.md#step-1-assign-the-required-licenses).

### <a name="user-registrations-show-multiple-times"></a>Les inscriptions de l’utilisateur apparaissent plusieurs fois.

Quand un utilisateur s’enregistre, nous journalisons actuellement chaque élément de données individuel sous la forme d’un événement distinct.

Si vous souhaitez agréger ces données et bénéficier d’une plus grande souplesse pour les afficher, vous pouvez télécharger le rapport et ouvrir les données dans un tableau croisé dynamique dans Excel.

## <a name="sspr-registration-portal"></a>Portail d’inscription à SSPR

Si vos utilisateurs rencontrent des problèmes lors de l’inscription à SSPR, passez en revue les étapes de dépannage suivantes :

### <a name="the-directory-isnt-enabled-for-password-reset-the-user-may-see-an-error-that-reports-your-administrator-has-not-enabled-you-to-use-this-feature"></a>L’annuaire n’est pas activé pour la réinitialisation du mot de passe. L’utilisateur peut voir une erreur signalant que « votre administrateur ne vous a pas autorisé à utiliser cette fonctionnalité ».

Vous pouvez activer SSPR pour tous les utilisateurs, aucun utilisateur ou pour des groupes sélectionnés d’utilisateurs. Un seul groupe Azure AD peut actuellement être activé pour SSPR à l’aide du portail Azure. Dans le cadre d’un déploiement plus étendu de SSPR, les groupes imbriqués sont pris en charge. Vérifiez que les licences appropriées sont attribuées aux utilisateurs des groupes que vous choisissez.

Dans le portail Azure, définissez **Réinitialisation de mot de passe en libre-service activée** sur *Sélectionné* ou *Tout le monde*, puis sélectionnez **Enregistrer**.

### <a name="the-user-doesnt-have-an-azure-ad-license-assigned-the-user-may-see-an-error-that-reports-your-administrator-has-not-enabled-you-to-use-this-feature"></a>L’utilisateur n’a pas de licence Azure AD. L’utilisateur peut voir une erreur signalant que « votre administrateur ne vous a pas autorisé à utiliser cette fonctionnalité ».

Un seul groupe Azure AD peut actuellement être activé pour SSPR à l’aide du portail Azure. Dans le cadre d’un déploiement plus étendu de SSPR, les groupes imbriqués sont pris en charge. Vérifiez que les licences appropriées sont attribuées aux utilisateurs des groupes que vous choisissez. Passez en revue l’étape de dépannage précédente pour activer SSPR en fonction des besoins.

Consultez également les étapes de dépannage pour vous assurer que l’administrateur qui procède aux options de configuration dispose d’une licence. Pour l’attribution d’une licence au compte administrateur en question, suivez les étapes pour [Affecter, vérifier et résoudre les problèmes des licences](../enterprise-users/licensing-groups-assign.md#step-1-assign-the-required-licenses).

### <a name="theres-an-error-processing-the-request"></a>Une erreur s’est produite pendant le traitement de la demande.

Les erreurs génériques d’inscription à SSPR peuvent être provoquées par de nombreux problèmes, mais en général cette erreur est due à une panne du service ou à un problème de configuration. Si cette erreur générique continue de s’afficher lorsque vous renouvelez le processus d’inscription à SSPR, [contactez le support technique Microsoft](#contact-microsoft-support) pour obtenir une assistance supplémentaire.

## <a name="sspr-usage"></a>Utilisation de SSPR

Si vos utilisateurs ou vous-même rencontrez des problèmes lors de l’utilisation de SSPR, passez en revue les scénarios de dépannage et les étapes de résolution suivants :

| Error | Solution |
| --- | --- |
| L’annuaire n’est pas activé pour la réinitialisation du mot de passe. | Dans le portail Azure, définissez **Réinitialisation de mot de passe en libre-service activée** sur *Sélectionné* ou *Tout le monde*, puis sélectionnez **Enregistrer**. |
| L’utilisateur n’a pas de licence Azure AD. | Cela peut se produire si vous n’avez pas de licence Azure AD affectée à l’utilisateur souhaité. Pour l’attribution d’une licence au compte administrateur en question, suivez les étapes pour [Affecter, vérifier et résoudre les problèmes des licences](../enterprise-users/licensing-groups-assign.md#step-1-assign-the-required-licenses). |
| L’annuaire est activé pour la réinitialisation du mot de passe, mais il manque une information d’authentification ou celle-ci n’est pas bien formée. | Assurez-vous que les informations de contact de l’utilisateur sont correctes dans l’annuaire. Pour plus d’informations, consultez [Données utilisées par la réinitialisation du mot de passe en libre-service Azure AD](howto-sspr-authenticationdata.md). |
| L’annuaire est activé pour la réinitialisation du mot de passe. Toutefois, l’utilisateur n’a fourni qu’une seule partie des données de contact alors que la stratégie nécessite deux méthodes de vérification. | Vérifiez que l’utilisateur a configuré correctement au moins deux méthodes de contact. Par exemple, il peut combiner un téléphone mobile *et* un téléphone fixe. |
| L’annuaire est activé pour la réinitialisation du mot de passe et l’utilisateur est correctement configuré, mais l’utilisateur est injoignable. | Cela peut être dû à une erreur temporaire de service ou à des données de contact incorrectement configurées qui n’ont pas été signalées comme telles. <br> <br> Si l’utilisateur patiente pendant 10 secondes, un lien s’affiche l’invitant à « Réessayer » et à « Contacter votre administrateur ». Si l’utilisateur sélectionne « Réessayer », il renouvelle l’appel. S’il sélectionne « Contacter votre administrateur », un e-mail est envoyé aux administrateurs leur demandant d’effectuer une réinitialisation de mot de passe pour ce compte d’utilisateur. |
| L’utilisateur ne reçoit jamais l’appel téléphonique ou le SMS de réinitialisation du mot de passe. | Cela peut être dû à un numéro de téléphone mal formé dans l’annuaire. Assurez-vous que le numéro de téléphone est au format « +1 4251234567 ». <br> <br>La réinitialisation du mot de passe ne prend pas en charge les extensions, même si vous en spécifiez une dans l’annuaire. Les numéros d’extension sont supprimés avant le passage de l’appel. Utilisez un numéro sans extension ou intégrez le numéro de poste au numéro de téléphone dans votre autocommutateur privé (PBX). |
| L’utilisateur ne reçoit jamais l’e-mail de réinitialisation du mot de passe. | La cause la plus probable de ce problème est que le message est rejeté par un filtre antispam. Consultez votre dossier de courrier indésirable ou d’éléments supprimés pour y rechercher l’e-mail. <br> <br> Assurez-vous également que l’utilisateur vérifie l’exactitude du compte de messagerie tel qu’il est inscrit auprès de SSPR. |
| J’ai défini une stratégie de réinitialisation du mot de passe, mais quand un compte d’administrateur utilise la réinitialisation du mot de passe, cette stratégie n’est pas appliquée. | Microsoft gère et contrôle la stratégie de réinitialisation du mot de passe administrateur afin de garantir le plus haut niveau de sécurité possible. |
| L’utilisateur est empêché de tenter de réinitialiser son mot de passe un nombre de fois par jour trop élevé. | Un mécanisme de limitation automatique est utilisé pour empêcher les utilisateurs d’essayer de réinitialiser leurs mots de passe un nombre de fois trop élevé sur un court laps de temps. La limitation se produit dans les scénarios suivants : <br><ul><li>L’utilisateur tente de valider un numéro de téléphone cinq fois en une heure.</li><li>L’utilisateur tente d’utiliser la méthode des questions de sécurité cinq fois en une heure.</li><li>L’utilisateur tente de réinitialiser un mot de passe pour le même compte d’utilisateur cinq fois en une heure.</li></ul>Si un utilisateur rencontre ce problème, il doit attendre 24 heures après la dernière tentative. L’utilisateur peut ensuite réinitialiser son mot de passe. |
| L’utilisateur rencontre une erreur durant la validation de son numéro de téléphone. | Cette erreur se produit quand le numéro de téléphone entré ne correspond pas au numéro de téléphone enregistré. Assurez-vous que l’utilisateur entre le numéro de téléphone complet, y compris le code de zone et de pays, quand il essaie d’utiliser une méthode par téléphone pour la réinitialisation du mot de passe. |
| Une erreur s’est produite pendant le traitement de la demande. | Les erreurs génériques d’inscription à SSPR peuvent être provoquées par de nombreux problèmes, mais en général cette erreur est due à une panne du service ou à un problème de configuration. Si cette erreur générique continue de s’afficher lorsque vous renouvelez le processus d’inscription à SSPR, [contactez le support technique Microsoft](#contact-microsoft-support) pour obtenir une assistance supplémentaire. |
| Violation de la stratégie locale | Le mot de passe ne respecte pas la stratégie de mot de passe Active Directory locale. L’utilisateur doit définir un mot de passe qui respecte les exigences de complexité ou de puissance. |
| Le mot de passe n’est pas conforme à la stratégie approximative | Le mot de passe qui a été utilisé fait partie de la [liste des mots de passe interdits](./concept-password-ban-bad.md#how-are-passwords-evaluated) et ne peut-être pas être utilisé. L’utilisateur doit définir un mot de passe qui respecte ou dépasse la stratégie de la liste des mots de passe interdits. |

## <a name="sspr-errors-that-a-user-might-see"></a>Erreurs SSPR qu’un utilisateur peut voir

Les erreurs et les détails techniques suivants peuvent s’afficher pour un utilisateur lors du processus SSPR. Souvent, l’erreur n’est pas un problème qu’il peut résoudre lui-même, car la fonctionnalité SSPR doit être activée, configurée ou inscrite pour son compte.

Utilisez les informations suivantes pour comprendre le problème et trouver ce qui doit être corrigé sur le locataire Azure AD ou le compte d’utilisateur individuel.

| Error | Détails | Détails techniques |
| --- | --- | --- |
| TenantSSPRFlagDisabled = 9 | Nous sommes désolés, vous ne pouvez pas réinitialiser votre mot de passe pour l’instant, car votre administrateur a désactivé la réinitialisation de mot de passe pour votre organisation. Vous ne pouvez entreprendre aucune action supplémentaire pour résoudre ce problème. Veuillez contacter votre administrateur et demandez-lui d’activer cette fonctionnalité.<br /><br />Pour en savoir plus, consultez [J’ai oublié mon mot de passe Azure AD](../user-help/active-directory-passwords-update-your-own-password.md#common-problems-and-their-solutions). | SSPR_0009 : nous avons détecté que la réinitialisation de mot de passe n’a pas été activée par votre administrateur. Veuillez contacter votre administrateur et demandez-lui d’activer la réinitialisation de mot de passe pour votre organisation. |
| WritebackNotEnabled = 10 |Nous sommes désolés, vous ne pouvez pas réinitialiser votre mot de passe pour l’instant, car votre administrateur n’a pas activé le service nécessaire pour votre organisation. Vous ne pouvez entreprendre aucune action supplémentaire pour résoudre ce problème. Veuillez contacter votre administrateur et demandez-lui de vérifier la configuration de votre organisation.<br /><br />Pour en savoir plus sur ce service nécessaire, consultez [Configuration de l’écriture différée du mot de passe](./tutorial-enable-sspr-writeback.md). | SSPR_0010 : nous avons détecté que la réécriture du mot de passe n’a pas été activée. Veuillez contacter votre administrateur et demandez-lui d’activer la réécriture du mot de passe. |
| SsprNotEnabledInUserPolicy = 11 | Nous sommes désolés, vous ne pouvez pas réinitialiser votre mot de passe pour l’instant, car votre administrateur n’a pas configuré la réinitialisation de mot de passe pour votre organisation. Vous ne pouvez entreprendre aucune action supplémentaire pour résoudre ce problème. Contactez votre administrateur et demandez-lui de configurer la réinitialisation de mot de passe.<br /><br />Pour en savoir plus sur la configuration de la réinitialisation de mot de passe, consultez [Démarrage rapide : Réinitialisation de mot de passe en libre-service Azure AD](./tutorial-enable-sspr.md). | SSPR_0011 : votre organisation n’a pas défini de stratégie de réinitialisation de mot de passe. Veuillez contacter votre administrateur et demandez-lui de définir une stratégie de réinitialisation de mot de passe. |
| UserNotLicensed = 12 | Nous sommes désolés, vous ne pouvez pas réinitialiser votre mot de passe pour l’instant, car il manque les licences nécessaires pour votre organisation. Vous ne pouvez entreprendre aucune action supplémentaire pour résoudre ce problème. Veuillez contacter votre administrateur et demandez-lui de vérifier votre attribution des licences.<br /><br />Pour en savoir plus sur les licences, consultez [Conditions de licence pour la réinitialisation du mot de passe en libre-service Azure AD](./concept-sspr-licensing.md). | SSPR_0012 : votre organisation n’a pas les licences nécessaires pour effectuer la réinitialisation du mot de passe. Veuillez contacter votre administrateur et demandez-lui de vérifier l’attribution des licences. |
| UserNotMemberOfScopedAccessGroup = 13 | Nous sommes désolés, vous ne pouvez pas réinitialiser votre mot de passe pour l’instant, car votre administrateur n’a pas configuré votre compte pour qu’il utilise la réinitialisation de mot de passe. Vous ne pouvez entreprendre aucune action supplémentaire pour résoudre ce problème. Veuillez contacter votre administrateur et demandez-lui de configurer votre compte pour la réinitialisation de mot de passe.<br /><br />Pour en savoir plus sur la configuration du compte pour la réinitialisation de mot de passe, consultez [Lancer la réinitialisation du mot de passe des utilisateurs](./howto-sspr-deployment.md). | SSPR_0013: vous n’êtes pas un membre d’un groupe activé pour la réinitialisation de mot de passe. Contactez votre administrateur et demandez-lui de vous ajouter au groupe. |
| UserNotProperlyConfigured = 14 | Nous sommes désolés, vous ne pouvez pas réinitialiser votre mot de passe pour l’instant, car il manque des informations nécessaires pour votre compte. Vous ne pouvez entreprendre aucune action supplémentaire pour résoudre ce problème. Veuillez contacter votre administrateur et demandez-lui de réinitialiser votre mot de passe pour vous. Une fois que vous avez à nouveau accès à votre compte, vous devez enregistrer les informations nécessaires.<br /><br />Pour enregistrer les informations, suivez les étapes de l’article [S’inscrire à la réinitialisation de mot de passe en libre-service](../user-help/active-directory-passwords-reset-register.md). | SSPR_0014 : des informations de sécurité supplémentaires sont nécessaires pour la réinitialisation de votre mot de passe. Pour réaliser l’opération, veuillez contacter votre administrateur et demandez-lui de réinitialiser votre mot de passe. Une fois que vous avez accès à votre compte, vous pouvez enregistrer des informations de sécurité supplémentaires à l’adresse https://aka.ms/ssprsetup. Votre administrateur peut ajouter des informations de sécurité supplémentaires à votre compte en suivant les étapes décrites dans [Définir et lire les données d’authentification à l’aide de PowerShell](howto-sspr-authenticationdata.md). |
| OnPremisesAdminActionRequired = 29 | Nous sommes désolés, nous ne pouvons pas réinitialiser votre mot de passe pour l’instant en raison d’un problème avec la configuration de réinitialisation de mot de passe de votre organisation. Vous ne pouvez entreprendre aucune action supplémentaire pour résoudre ce problème. Veuillez contacter votre administrateur et demandez-lui d’étudier le problème. <br /><br />Ou<br /><br />Nous ne pouvons pas réinitialiser votre mot de passe pour l’instant en raison d’un problème avec la configuration de réinitialisation de mot de passe de votre organisation. Vous ne pouvez entreprendre aucune action supplémentaire pour résoudre ce problème. Veuillez contacter votre administrateur et demandez-lui d’étudier le problème.<br /><br />Pour plus d’informations sur le problème éventuel, consultez [Résoudre les problèmes d’écriture différée du mot de passe](troubleshoot-sspr-writeback.md). | SSPR_0029 : nous ne sommes pas en mesure de réinitialiser votre mot de passe en raison d’une erreur dans votre configuration locale. Veuillez contacter votre administrateur et demandez-lui d’étudier le problème. |
| OnPremisesConnectivityError = 30 | Nous sommes désolés, nous ne pouvons pas réinitialiser votre mot de passe pour l’instant en raison de problèmes de connectivité à votre organisation. Aucune action ne peut être entreprise maintenant, mais le problème sera éventuellement résolu si vous réessayez plus tard. Si le problème persiste, veuillez contacter votre administrateur et demandez-lui d’étudier le problème.<br /><br />Pour en savoir plus sur les problèmes de connectivité, consultez [Résolution des problèmes de connectivité de la réécriture du mot de passe](troubleshoot-sspr-writeback.md). | SSPR_0030 : nous ne pouvons pas réinitialiser votre mot de passe en raison d’une mauvaise connexion à votre environnement local. Contactez votre administrateur et demandez-lui d’étudier le problème.|

## <a name="azure-ad-forums"></a>Forums d’Azure AD

Si vous avez des questions générales sur Azure AD et la réinitialisation du mot de passe en libre-service, vous pouvez demander de l’aide à la communauté via la [Page des Q&R Microsoft pour Azure Active Directory](/answers/topics/azure-active-directory.html). Les membres de la communauté comprennent des ingénieurs, des chefs de produit, MVP et autres professionnels de l’informatique.

## <a name="contact-microsoft-support"></a>Contact Microsoft support

Si vous ne trouvez pas la réponse à un problème, nos équipes de support technique sont toujours disponibles pour vous aider davantage.

Pour que nous puissions mieux vous aider, nous vous demandons de fournir autant de détails que possible au moment de l’ouverture d’un dossier d’incident. Ces détails comprennent ce qui suit :

* **Description générale de l’erreur**. Quelle est l’erreur ? Quel était le comportement que vous avez remarqué ? Comment pouvons-nous reproduire l’erreur ? Fournissez autant de détails que possible.
* **Page**. Sur quelle page étiez-vous quand vous avez remarqué l’erreur ? Indiquez l’URL si possible, ainsi qu’une capture d’écran de la page.
* **Code de support**. Quel était le code de support généré quand l’utilisateur a vu l’erreur ?
   * Pour trouver ce code, reproduisez l’erreur, puis sélectionnez le lien **Code de support** en bas de l’écran et envoyez à l’ingénieur du support technique le GUID obtenu.

    :::image type="content" source="./media/troubleshoot-sspr-writeback/view-support-code.png" alt-text="Le code de support se trouve en bas à droite de la fenêtre du navigateur web.":::

  * Si vous êtes dans une page sans code de support dans la partie inférieure, appuyez sur F12 et recherchez le SID et le CID et envoyez ces deux résultats à l’ingénieur de support.
* **Date, heure et fuseau horaire**. Incluez la date et l’heure précises, *avec le fuseau horaire*, d’occurrence de l’erreur.
* **ID d’utilisateur**. Quel était l’utilisateur qui a vu l’erreur ? *user\@contoso.com* en est un exemple.
   * S’agit-il d’un utilisateur fédéré ?
   * S’agit-il d’un utilisateur de l’authentification directe ?
   * S’agit-il d’un utilisateur disposant de la synchronisation du hachage de mot de passe ?
   * S’agit-il d’un utilisateur cloud uniquement ?
* **Licences**. L’utilisateur a-t-il une licence Azure AD ?
* **Journal des événements de l’application**. Si vous utilisez la réécriture de mot de passe et que l’erreur se produit dans votre infrastructure locale, veuillez inclure une copie compressée du journal des événements de l’application provenant de votre serveur Azure AD Connect.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur SSPR, consultez [Fonctionnement : Réinitialisation de mot de passe en libre-service Azure AD](concept-sspr-howitworks.md) ou [Comment fonctionne la réécriture de la réinitialisation de mot de passe en libre-service dans Azure AD ?](concept-sspr-writeback.md).
