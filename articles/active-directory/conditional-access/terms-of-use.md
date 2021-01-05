---
title: Conditions d’utilisation - Azure Active Directory | Microsoft Docs
description: Découvrez comment utiliser les conditions d’utilisation Azure Active Directory pour présenter des informations aux employés ou aux utilisateurs invités avant de leur octroyer l’accès.
services: active-directory
ms.service: active-directory
ms.subservice: compliance
ms.topic: how-to
ms.date: 12/02/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: jocastel
ms.collection: M365-identity-device-management
ms.openlocfilehash: 399cbef4de91623ada1c38de7c6148d22fde94c7
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97672398"
---
# <a name="azure-active-directory-terms-of-use"></a>Conditions d’utilisation d’Azure Active Directory

Les conditions d’utilisation d’Azure AD offre aux organisations une méthode simple pour présenter des informations aux utilisateurs finaux. Cette présentation permet de garantir que les utilisateurs voient les clauses d’exclusion de responsabilité nécessaires au respect des conditions légales ou de conformité. Cet article vous montre comment démarrer avec les conditions d’utilisation.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

## <a name="overview-videos"></a>Vidéos de présentation

La vidéo suivante fournit une vue d’ensemble des conditions d’utilisation.

>[!VIDEO https://www.youtube.com/embed/tj-LK0abNao]

Pour voir d’autres vidéos, consultez :
- [How to deploy terms of use in Azure Active Directory](https://www.youtube.com/embed/N4vgqHO2tgY)
- [How to roll out terms of use in Azure Active Directory](https://www.youtube.com/embed/t_hA4y9luCY)

## <a name="what-can-i-do-with-terms-of-use"></a>Que permettent les conditions d’utilisation ?

Les conditions d’utilisation d’Azure AD vous permettent d’effectuer les opérations suivantes :

- Faire accepter vos conditions d’utilisation à vos employés ou à vos utilisateurs invités avant de leur octroyer l’accès
- Faire accepter vos conditions d’utilisation à vos employés ou à vos utilisateurs invités sur chaque appareil avant de leur octroyer l’accès
- Faire accepter vos conditions d’utilisation à vos employés ou à vos utilisateurs invités selon une planification périodique
- Faire accepter vos conditions d'utilisation à vos employés ou à vos utilisateurs invités avant d'inscrire les informations de sécurité dans Azure AD Multi-Factor Authentication (MFA)
- Faire accepter vos conditions d’utilisation à vos employés avant d’inscrire les informations de sécurité dans la réinitialisation de mot de passe en libre-service (SSPR) Azure AD
- Présenter les conditions d’utilisation générales à tous les utilisateurs de votre organisation
- Présenter les conditions d’utilisation spécifiques en fonction des attributs d’un utilisateur (par exemple, docteurs/infirmières ou employés nationaux/internationaux dans des [groupes dynamiques](../enterprise-users/groups-dynamic-membership.md)).
- Présenter les conditions d’utilisation spécifiques lors de l’accès aux applications à fort impact commercial, comme Salesforce
- Présenter les conditions d’utilisation dans différentes langues
- Lister qui a accepté ou refusé vos conditions d’utilisation
- Aider à faire face à la réglementation sur la confidentialité.
- Consulter le journal des conditions d’utilisation pour en vérifier la conformité et procéder à un audit
- Créer et gérer des conditions d’utilisation avec des [API Microsoft Graph](/graph/api/resources/agreement?view=graph-rest-beta) (actuellement en préversion)

## <a name="prerequisites"></a>Prérequis

Pour utiliser et configurer les conditions d’utilisation d’Azure AD, vous devez disposer des éléments suivants :

- Abonnement Azure AD Premium P1, P2, EMS E3 ou EMS E5.
   - Si vous ne disposez pas d’un de ces abonnements, vous pouvez [obtenir Azure AD Premium](../fundamentals/active-directory-get-started-premium.md) ou [activer un essai d’Azure AD Premium](https://azure.microsoft.com/trial/get-started-active-directory/).
- Un des comptes d’administrateur suivants pour le répertoire que vous souhaitez configurer :
   - Administrateur général
   - Security Administrator
   - Administrateur de l’accès conditionnel

## <a name="terms-of-use-document"></a>Document Conditions d’utilisation

Les conditions d’utilisation d’Azure AD sont au format PDF. Libre à vous de choisir le contenu de ce PDF. Vous pouvez par exemple utiliser le document d’un contrat existant, ce qui vous permet de collecter les contrats des utilisateurs finaux quand ils se connectent. Pour prendre en charge les utilisateurs sur les appareils mobiles, il est recommandé d’utiliser une taille de police de 24 points dans le fichier PDF.

## <a name="add-terms-of-use"></a>Ajouter des conditions d’utilisation

Après avoir finalisé le document relatif à vos conditions d’utilisation, effectuez les étapes suivantes pour les ajouter.

1. Connectez-vous à Azure en tant qu’Administrateur général, Administrateur de la sécurité ou Administrateur de l’accès conditionnel.
1. Accédez aux **conditions d’utilisation** sur [https://aka.ms/catou](https://aka.ms/catou).

    ![Accès conditionnel - Panneau Conditions d’utilisation](./media/terms-of-use/tou-blade.png)

1. Cliquez sur **Nouvelles conditions d’utilisation**.

    ![Nouveau panneau Conditions d’utilisation permettant de spécifier les paramètres de vos conditions d’utilisation](./media/terms-of-use/new-tou.png)

1. Dans la zone **Nom**, entrez le nom des conditions d’utilisation à utiliser dans le portail Azure.
1. Dans la zone **Nom d’affichage**, entrez le titre présenté aux utilisateurs quand ils se connectent.
1. Pour **Document Conditions d’utilisation**, accédez au fichier PDF finalisé de vos conditions d’utilisation et sélectionnez-le.
1. Sélectionnez la langue de votre document Conditions d’utilisation. L’option de langue vous permet de charger plusieurs conditions d’utilisation, chacune dans une langue différente. La version des conditions d’utilisation affichée pour l’utilisateur final dépend des préférences de son navigateur.
1. Pour demander aux utilisateurs finaux d’afficher les conditions d’utilisation avant de les accepter, définissez **Demander aux utilisateurs d’étendre les conditions d’utilisation** sur **Activé**.
1. Pour demander aux utilisateurs finaux d’accepter vos conditions d’utilisation sur chaque appareil à partir duquel l’accès leur est octroyé, définissez **Demander le consentement des utilisateurs sur chaque appareil** sur **Activé**. Les utilisateurs peuvent être amenés à installer des applications supplémentaires si cette option est activée. Pour plus d’informations, consultez [Conditions d’utilisation par appareil](#per-device-terms-of-use).
1. Si vous voulez faire expirer les consentements pour les conditions d’utilisation selon une planification, définissez **Faire expirer les consentements** sur **Activé**. Si la valeur Activé est définie, deux paramètres de planification supplémentaires sont affichés.

    ![Paramètres Faire expirer les consentements pour définir la date de début, la fréquence et la durée](./media/terms-of-use/expire-consents.png)

1. Utilisez les paramètres **Expiration commençant le** et **Fréquence** pour spécifier la planification régissant l’expiration des conditions d’utilisation. Le tableau suivant présente deux exemples de paramètres et leur résultat :

   | Expiration commençant le | Fréquence | Résultats |
   | --- | --- | --- |
   | Date du jour  | Mensuelle | À compter d’aujourd’hui, les utilisateurs doivent accepter les conditions d’utilisation et les réaccepter chaque mois. |
   | Date future  | Mensuelle | À compter d’aujourd’hui, les utilisateurs doivent accepter les conditions d’utilisation. Quand la date future survient, les consentements expirent. Les utilisateurs doivent alors réaccepter les conditions d’utilisation chaque mois.  |

   Par exemple, si vous définissez « Expiration commençant le » avec la date **1 jan** et la fréquence avec la valeur **Mensuelle**, voici comment les expirations peuvent se produire pour deux utilisateurs :

   | Utilisateur | Date d’acceptation initiale | Première date d’expiration | Deuxième date d’expiration | Troisième date d’expiration |
   | --- | --- | --- | --- | --- |
   | Alice | 1 jan | 1 fév | 1 mar | 1 avr |
   | Bob | 15 jan | 1 fév | 1 mar | 1 avr |

1. Utilisez le paramètre **Durée avant nouvelle acceptation requise (jours)** pour spécifier le nombre de jours au bout duquel l’utilisateur doit réaccepter les conditions d’utilisation. Les utilisateurs peuvent ainsi suivre leur propre calendrier. Par exemple, si vous définissez une durée de **30** jours, voici comment les expirations peuvent se produire pour deux utilisateurs :

   | Utilisateur | Date d’acceptation initiale | Première date d’expiration | Deuxième date d’expiration | Troisième date d’expiration |
   | --- | --- | --- | --- | --- |
   | Alice | 1 jan | 31 jan | 2 mar | 1 avr |
   | Bob | 15 jan | 14 fév | 16 mar | 15 avr |

   Il est possible d’utiliser les paramètres **Faire expirer les consentements** et **Durée avant nouvelle acceptation requise (jours)** ensemble, mais généralement, vous utilisez l’un ou l’autre.

1. Sous **Accès conditionnel**, utilisez la liste **Appliquer avec des modèles de stratégie d’accès conditionnel** afin de sélectionner le modèle à utiliser pour appliquer les conditions d’utilisation.

    ![Liste déroulante Accès conditionnel pour sélectionner un modèle de stratégie](./media/terms-of-use/conditional-access-templates.png)

   | Modèle | Description |
   | --- | --- |
   | **Accès aux applications cloud pour tous les invités** | Une stratégie d’accès conditionnel est créée pour tous les utilisateurs invités et toutes les applications cloud. Cette stratégie impacte le portail Azure. Une fois la stratégie créée, il peut vous être demandé de vous déconnecter et de vous connecter. |
   | **Accès aux applications cloud pour tous les utilisateurs** | Une stratégie d’accès conditionnel est créée pour tous les utilisateurs et toutes les applications cloud. Cette stratégie impacte le portail Azure. Une fois la stratégie créée, il vous est demandé de vous déconnecter et de vous connecter. |
   | **Stratégie personnalisée** | Sélectionnez les utilisateurs, groupes et applications auxquels ces conditions d’utilisation seront appliquées. |
   | **Créer la stratégie d’accès conditionnel plus tard** | Ces conditions d’utilisation apparaîtront dans la liste de contrôle d’autorisation lors de la création d’une stratégie d’accès conditionnel. |

   >[!IMPORTANT]
   >Les contrôles de stratégie d’accès conditionnel (y compris les conditions d’utilisation) ne peuvent pas être appliqués sur les comptes de service. Nous vous recommandons d’exclure tous les comptes de service de la stratégie d’accès conditionnel.

    Les stratégies d’accès conditionnel personnalisées permettent de définir des conditions d’utilisation plus précises, jusqu’au niveau d’une application cloud ou d’un groupe d’utilisateurs. Pour plus d’informations, consultez [Démarrage rapide : Exiger l’acceptation des conditions d’utilisation avant d’accorder l’accès à des applications cloud](require-tou.md).

1. Cliquez sur **Créer**.

    Si vous avez sélectionné un modèle d’accès conditionnel personnalisé, un nouvel écran s’affiche pour vous permettre de créer la stratégie d’accès conditionnel personnalisée.

   ![Nouveau volet Accès conditionnel si vous avez choisi le modèle de stratégie d’accès conditionnel personnalisé](./media/terms-of-use/custom-policy.png)

   Vos nouvelles conditions d’utilisation doivent désormais s’afficher.

   ![Nouvelles conditions d’utilisation listées dans le panneau Conditions d’utilisation](./media/terms-of-use/create-tou.png)

## <a name="view-report-of-who-has-accepted-and-declined"></a>Afficher le rapport des utilisateurs ayant accepté et refusé les conditions d’utilisation

Le panneau des conditions d’utilisation affiche le nombre d’utilisateurs ayant accepté et refusé les conditions d’utilisation. Ce nombre, ainsi que le nom des personnes ayant accepté ou refusé, sont stockés pour toute la durée de vie des conditions d’utilisation.

1. Connectez-vous à Azure et accédez à **Conditions d’utilisation** sur [https://aka.ms/catou](https://aka.ms/catou).

    ![Panneau Conditions d’utilisation montrant le nombre d’utilisateurs ayant accepté et le nombre d’utilisateurs ayant refusé les conditions](./media/terms-of-use/view-tou.png)

1. Cliquez sur les chiffres situés sous **Accepté** et **Refusé** pour voir qui a accepté et qui a refusé les conditions d’utilisation.

    ![Volet Consentements pour les conditions d’utilisation listant les utilisateurs qui ont accepté les conditions d’utilisation](./media/terms-of-use/accepted-tou.png)

1. Pour afficher l’historique d’un utilisateur individuel, cliquez sur le bouton de sélection ( **...** ), puis sur **Afficher l’historique**.

    ![Menu contextuel Afficher l’historique pour un utilisateur](./media/terms-of-use/view-history-menu.png)

   Le volet Afficher l’historique présente l’historique des acceptations, des refus et des expirations.

   ![Le volet Afficher l’historique liste les acceptations, les refus et les délais d’expiration associés à un utilisateur](./media/terms-of-use/view-history-pane.png)

## <a name="view-azure-ad-audit-logs"></a>Afficher les journaux d’audit Azure AD

Si vous souhaitez afficher d’autres activités, la fonctionnalité Conditions d’utilisation d’Azure AD inclut des journaux d’audit. Chaque consentement d’utilisateur déclenche un événement dans les journaux d’audit qui est stocké pendant **30 jours**. Vous pouvez afficher ces journaux d’activité dans le portail ou les télécharger sous forme de fichier .csv.

Pour démarrer avec les journaux d’audit Azure AD, effectuez les étapes suivantes :

1. Connectez-vous à Azure et accédez à **Conditions d’utilisation** sur [https://aka.ms/catou](https://aka.ms/catou).
1. Sélectionnez des conditions d’utilisation.
1. Cliquez sur **Afficher les journaux d’audit**.

    ![Panneau Conditions d’utilisation avec l’option Afficher les journaux d’audit mise en surbrillance](./media/terms-of-use/audit-tou.png)

1. Sur l’écran des journaux d’audit Azure AD, vous pouvez filtrer les informations à l’aide des listes fournies pour cibler des éléments spécifiques du journal d’audit.

    Vous pouvez également cliquer sur **Télécharger** pour télécharger les informations dans un fichier .csv en vue d’une utilisation locale.

   ![Écran Journaux d’audit Azure AD listant la date, la stratégie cible, l’activité et la personne ayant démarré l’activité](./media/terms-of-use/audit-logs-tou.png)

   Si vous cliquez sur un journal, un volet s’affiche avec des détails supplémentaires sur l’activité.

   ![Détails de l’activité dans un journal d’audit montrant l’activité, l’état de l’activité, la personne ayant démarré l’activité et la stratégie cible](./media/terms-of-use/audit-log-activity-details.png)

## <a name="what-terms-of-use-looks-like-for-users"></a>À quoi ressemblent les conditions d’utilisation pour les utilisateurs ?

Une fois les conditions d’utilisation créées et appliquées, les utilisateurs concernés voient l’écran suivant au cours de la connexion.

![Exemple de conditions d’utilisation qui s’affichent lorsqu’un utilisateur se connecte](./media/terms-of-use/user-tou.png)

Les utilisateurs peuvent afficher les conditions d’utilisation et, si nécessaire, utiliser les boutons pour effectuer un zoom avant ou arrière.

![Vue des conditions d’utilisation avec les boutons de zoom](./media/terms-of-use/zoom-buttons.png)

L’écran suivant montre comment se présentent les conditions d’utilisation sur les appareils mobiles.

![Exemple de conditions d’utilisation qui s’affichent lorsqu’un utilisateur se connecte sur un appareil mobile](./media/terms-of-use/mobile-tou.png)

Les utilisateurs n’ont à accepter les conditions d’utilisation qu’une seule fois et ils ne les verront plus lors des prochaines connexions.

### <a name="how-users-can-review-their-terms-of-use"></a>Comment les utilisateurs peuvent consulter leurs conditions d’utilisation

Vous pouvez consulter et vérifier les conditions d’utilisation qu’ils ont acceptées en procédant de la façon suivante.

1. Connectez-vous à [https://myapps.microsoft.com](https://myapps.microsoft.com).
1. Dans l’angle supérieur droit, cliquez sur votre nom, puis sélectionnez **Profil**.

    ![Site MyApps avec le volet de l’utilisateur ouvert](./media/terms-of-use/tou14.png)

1. Dans la page de votre profil, cliquez sur **Vérifier les conditions d’utilisation**.

    ![Page de profil d’un utilisateur montrant le lien Vérifier les conditions d’utilisation](./media/terms-of-use/tou13a.png)

1. C’est là que vous pouvez consulter les conditions d’utilisation que vous avez acceptées.

## <a name="edit-terms-of-use-details"></a>Modifier les détails des conditions d’utilisation

Vous pouvez modifier certains détails des conditions d’utilisation, mais il n’est pas possible de modifier un document existant. La procédure suivante explique comment modifier les détails.

1. Connectez-vous à Azure et accédez à **Conditions d’utilisation** sur [https://aka.ms/catou](https://aka.ms/catou).
1. Sélectionnez les conditions d’utilisation à modifier.
1. Cliquez sur **Modifier les conditions**.
1. Dans le volet Modifier les conditions d’utilisation, vous pouvez changer les éléments suivants :
    - **Nom** : nom interne des conditions d’utilisation qui n’est pas partagé avec les utilisateurs finaux
    - **Nom d’affichage** : nom que les utilisateurs finaux peuvent voir lors de l’affichage des conditions d’utilisation
    - **Demander aux utilisateurs d’étendre les conditions d’utilisation** : la définition de cette option sur **Activé** force l’utilisateur final à étendre les conditions d’utilisation avant de les accepter.
    - (Préversion) Vous pouvez **mettre à jour un document de conditions d’utilisation existant**
    - Vous pouvez ajouter une langue à des conditions d’utilisation existantes

   Si vous souhaitez changer d’autres paramètres, comme le document PDF, le fait de demander le consentement des utilisateurs sur chaque appareil, l’expiration des consentements, la durée avant nouvelle acceptation ou encore la stratégie d’accès conditionnel, vous devez établir de nouvelles conditions d’utilisation.

    ![Modifier les différentes options de langue ](./media/terms-of-use/edit-terms-use.png)

1. Une fois que vous avez terminé, cliquez sur **Enregistrer** pour enregistrer vos modifications.

## <a name="update-the-version-or-pdf-of-an-existing-terms-of-use"></a>Mettre à jour la version ou le fichier PDF de conditions d’utilisation existantes

1.  Connectez-vous à Azure et accédez à [Conditions d’utilisation](https://aka.ms/catou).
2.  Sélectionnez les conditions d’utilisation à modifier.
3.  Cliquez sur **Modifier les conditions**.
4.  Pour la langue dans laquelle vous souhaitez mettre à jour une nouvelle version, cliquez sur **Mettre à jour** sous la colonne Action.

    ![Volet Modifier les conditions d’utilisation montrant les options Nom et Développer](./media/terms-of-use/edit-terms-use.png)

5.  Dans le volet de droite, chargez le fichier PDF pour la nouvelle version.
6.  Il existe également une option à bascule **Exiger une nouvelle acceptation** si vous souhaitez obliger les utilisateurs à accepter cette nouvelle version la prochaine fois qu’ils se connectent. Si vous exigez une nouvelle acceptation de la part de vos utilisateurs, ceux-ci sont invités à accepter cette nouvelle version la prochaine fois qu’ils tentent d’accéder à la ressource définie dans votre stratégie d’accès conditionnel. Si vous n’exigez pas de nouvelle acceptation de la part de vos utilisateurs, leur consentement préalable demeure à jour et seuls les nouveaux utilisateurs qui n’ont pas fait part de leur consentement ou dont le consentement arrive à expiration voient la nouvelle version.

    ![Modifier les conditions d’utilisation - Mise en évidence de l’option permettant d’exiger une nouvelle acceptation](./media/terms-of-use/re-accept.png)

7.  Une fois que vous avez chargé votre nouveau fichier PDF et opéré votre choix quant à l’exigence d’une nouvelle acceptation, cliquez sur Ajouter en bas du volet.
8.  La version la plus récente apparaît alors dans la colonne Document.

## <a name="view-previous-versions-of-a-terms-of-use"></a>Afficher les versions précédentes de conditions d’utilisation

1.  Connectez-vous à Azure et accédez à **Conditions d’utilisation** sur https://aka.ms/catou.
2.  Sélectionnez les conditions d’utilisation dont vous souhaitez afficher un historique des versions.
3.  Cliquez sur **Langues et historique des versions**.
4.  Cliquez sur **Voir les versions précédentes**.

    ![Détails du document, y compris les versions linguistiques](./media/terms-of-use/document-details.png)

5.  Vous pouvez cliquer sur le nom du document pour télécharger cette version.

## <a name="see-who-has-accepted-each-version"></a>Voir qui a accepté chaque version

1.  Connectez-vous à Azure et accédez à **Conditions d’utilisation** sur https://aka.ms/catou.
2.  Pour voir qui a accepté les conditions d’utilisation, sous la colonne **Accepté**, cliquez sur le numéro des conditions d’utilisation de votre choix.
3.  Par défaut, la page suivante indique l’état actuel de l’acceptation des conditions d’utilisation par chaque utilisateur.
4.  Si vous souhaitez afficher les événements de consentement préalable, vous pouvez sélectionner **Tous** dans la liste déroulante **État actuel**. Vous pouvez maintenant voir les événements de chaque utilisateur dans les détails pour chaque version et ce qui s’est passé.
5.  Vous pouvez également sélectionner une version spécifique dans la liste déroulante **Version** pour voir qui l’a acceptée.


## <a name="add-a-terms-of-use-language"></a>Ajouter une langue aux conditions d’utilisation

La procédure suivante explique comment ajouter une langue aux conditions d’utilisation.

1. Connectez-vous à Azure et accédez à **Conditions d’utilisation** sur [https://aka.ms/catou](https://aka.ms/catou).
1. Sélectionnez les conditions d’utilisation à modifier.
1. Cliquez sur **Modifier les conditions**.
1. Cliquez sur **Ajouter une langue** en bas de la page.
1. Dans le volet Ajouter une langue aux conditions d’utilisation, chargez votre PDF localisé et sélectionnez la langue.

    ![Conditions d’utilisation sélectionnées, montrant l’onglet Langues dans le volet Détails](./media/terms-of-use/select-language.png)

1. Cliquez sur **Ajouter une langue**.
1. Cliquez sur **Enregistrer**

1. Cliquez sur **Ajouter** pour ajouter la langue.

## <a name="per-device-terms-of-use"></a>Conditions d’utilisation par appareil

Le paramètre **Demander le consentement des utilisateurs sur chaque appareil** vous permet de demander aux utilisateurs finaux d’accepter vos conditions d’utilisation sur chaque appareil à partir duquel l’accès leur est octroyé. L’utilisateur final doit inscrire son appareil dans Azure AD. Quand l’appareil est inscrit, l’ID d’appareil est utilisé pour appliquer les conditions d’utilisation sur chaque appareil.

Voici une liste des plateformes et des logiciels pris en charge.

> [!div class="mx-tableFixed"]
> |  | iOS | Android | Windows 10 | Autres |
> | --- | --- | --- | --- | --- |
> | **Application native** | Oui | Oui | Oui |  |
> | **Microsoft Edge** | Oui | Oui | Oui |  |
> | **Internet Explorer** | Oui | Oui | Oui |  |
> | **Chrome (avec extension)** | Oui | Oui | Oui |  |

Les conditions d’utilisation par appareil sont associées aux contraintes suivantes :

- Un appareil ne peut être joint qu’à un seul locataire.
- Un utilisateur doit disposer d’autorisations pour joindre son appareil.
- L’application d’inscription à Intune n’est pas prise en charge. Assurez-vous qu’elle est exclue de toute stratégie d’accès conditionnel nécessitant des conditions d’utilisation.
- Les utilisateurs Azure AD B2B ne sont pas pris en charge.

Si l’appareil de l’utilisateur n’est pas joint, il reçoit un message lui demandant de le faire. Son expérience varie selon la plateforme et le logiciel utilisés.

### <a name="join-a-windows-10-device"></a>Joindre un appareil Windows 10

Si un utilisateur utilise Windows 10 et Microsoft Edge, il reçoit un message semblable au suivant pour lui demander de [joindre son appareil](../user-help/user-help-join-device-on-network.md#to-join-an-already-configured-windows-10-device).

![Windows 10 et Microsoft Edge - Message indiquant que votre appareil doit être inscrit.](./media/terms-of-use/per-device-win10-edge.png)

S’il utilise Chrome, il est invité à installer l’[extension Windows 10 Accounts](https://chrome.google.com/webstore/detail/windows-10-accounts/ppnbnpeolgkicgegkbkbjmhlideopiji).

### <a name="register-an-ios-device"></a>Inscrire un appareil iOS

Si un utilisateur utilise un appareil iOS, il est invité à installer l’[application Microsoft Authenticator](https://apps.apple.com/us/app/microsoft-authenticator/id983156458).

### <a name="register-an-android-device"></a>Inscrire un appareil Android

Si un utilisateur utilise un appareil Android, il est invité à installer l’[application Microsoft Authenticator](https://play.google.com/store/apps/details?id=com.azure.authenticator).

### <a name="browsers"></a>Navigateurs

Si un utilisateur utilise un navigateur qui n’est pas pris en charge, il est invité à utiliser un autre navigateur.

![Message indiquant que votre appareil doit être inscrit, mais que le navigateur n’est pas pris en charge.](./media/terms-of-use/per-device-browser-unsupported.png)

## <a name="delete-terms-of-use"></a>Supprimer des conditions d’utilisation

Pour supprimer d’anciennes conditions d’utilisation, effectuez les étapes suivantes :

1. Connectez-vous à Azure et accédez à **Conditions d’utilisation** sur [https://aka.ms/catou](https://aka.ms/catou).
1. Sélectionnez les conditions d’utilisation à supprimer.
1. Cliquez sur **Supprimer les conditions d’utilisation**.
1. Dans le message qui s’affiche vous demandant si vous souhaitez continuer, cliquez sur **Oui**.

    ![Message demandant de confirmer la suppression des conditions d’utilisation](./media/terms-of-use/delete-tou.png)

   Désormais, vos conditions d’utilisation ne doivent plus s’afficher.

## <a name="deleted-users-and-active-terms-of-use"></a>Utilisateurs supprimés et conditions d’utilisation actives

Par défaut, un utilisateur supprimé conserve cet état dans Azure AD pendant 30 jours, au cours desquels il peut être restauré par un administrateur s’il y a lieu. Au bout de 30 jours, cet utilisateur est définitivement supprimé. En outre, à l’aide du portail Azure Active Directory, un Administrateur général peut explicitement [supprimer définitivement un utilisateur récemment supprimé](../fundamentals/active-directory-users-restore.md) avant que cette période arrive à son terme. Une fois qu’un utilisateur a été définitivement supprimé, les données le concernant sont supprimées des conditions d’utilisation actives. Les informations d’audit relatives aux utilisateurs supprimés sont conservées dans le journal d’audit.

## <a name="policy-changes"></a>Modifications de stratégie

Les stratégies d’accès conditionnel prennent effet immédiatement. Dans ce cas, l’administrateur commence à voir des icônes de nuages tristes ou des erreurs liées aux jetons Azure AD. L’administrateur doit se déconnecter puis se reconnecter pour satisfaire aux exigences de la nouvelle stratégie.

> [!IMPORTANT]
> Les utilisateurs dans l’étendue doivent se déconnecter et se reconnecter pour satisfaire aux exigences d’une nouvelle stratégie si :
>
> - Une stratégie d’accès conditionnel est activée dans les conditions d’utilisation
> - une deuxième instance des conditions d’utilisation est créée.

## <a name="b2b-guests"></a>Invités B2B

La plupart des organisations mettent en place un processus qui permet à leurs employés de consentir aux conditions d’utilisation et déclarations de confidentialité de leur organisation. Mais comment faire pour appliquer les mêmes consentements pour les invités Azure AD B2B quand ils sont ajoutés par le biais de SharePoint ou de Teams ? Grâce à l’accès conditionnel et aux conditions d’utilisation, vous pouvez appliquer une stratégie directement aux utilisateurs invités B2B. Durant le flux d’acceptation d’invitation, l’utilisateur voit les conditions d’utilisation s’afficher. Cette prise en charge est actuellement en mode préliminaire.

Les conditions d’utilisation sont seulement affichées quand l’utilisateur dispose d’un compte invité dans Azure AD. SharePoint Online propose actuellement une [expérience ad-hoc de partage avec des destinataires externes](/sharepoint/what-s-new-in-sharing-in-targeted-release) permettant de partager un document ou un dossier avec des utilisateurs n’ayant pas de compte invité. Dans ce cas, les conditions d’utilisation ne sont pas affichées.

![Volet Utilisateurs et groupes - Onglet Inclure avec l’option Tous les utilisateurs invités cochée](./media/terms-of-use/b2b-guests.png)

## <a name="support-for-cloud-apps"></a>Prise en charge des applications cloud

Les conditions d’utilisation peuvent être utilisées pour différentes applications cloud, notamment Azure Information Protection et Microsoft Intune. Cette prise en charge est actuellement en mode préliminaire.

### <a name="azure-information-protection"></a>Azure Information Protection

Vous pouvez configurer une stratégie d’accès conditionnel pour l’application Azure Information Protection et exiger l’affichage de conditions d’utilisation quand un utilisateur accède à un document protégé. L’affichage des conditions d’utilisation est alors déclenché avant l’accès initial de l’utilisateur au document protégé.

![Volet Applications cloud avec l’application Microsoft Azure Information Protection sélectionnée](./media/terms-of-use/cloud-app-info-protection.png)

### <a name="microsoft-intune-enrollment"></a>Inscription à Microsoft Intune

Vous pouvez configurer une stratégie d’accès conditionnel pour l’application d’inscription à Microsoft Intune et exiger l’affichage de conditions d’utilisation avant l’inscription d’un appareil dans Intune. Pour plus d’informations, consultez le [billet de blog sur le choix des conditions appropriées pour votre organisation](https://go.microsoft.com/fwlink/?linkid=2010506&clcid=0x409).

![Volet Applications cloud avec l’application Microsoft Intune sélectionnée](./media/terms-of-use/cloud-app-intune.png)

> [!NOTE]
> L’application d’inscription à Intune n’est pas prise en charge pour [Conditions d’utilisation par appareil](#per-device-terms-of-use).

## <a name="frequently-asked-questions"></a>Forum aux questions

**Q : Comment déterminer si un utilisateur a accepté les conditions d’utilisation et quand ?**<br />
A : Dans le panneau Conditions d’utilisation, cliquez sur le numéro sous **Accepté**. Vous pouvez également afficher ou rechercher l’activité d’acceptation dans les journaux d’audit Azure AD. Pour plus d’informations, consultez Afficher le rapport des utilisateurs ayant accepté et refusé les conditions d’utilisation et [Afficher les journaux d’audit Azure AD](#view-azure-ad-audit-logs).

**Q : Combien de temps sont stockées les informations ?**<br />
A : Le nombre d’utilisateurs ayant accepté et le nombre d’utilisateurs ayant refusé les conditions d’utilisation sont stockés pour la durée de vie des conditions d’utilisation. Les journaux d’audit Azure AD sont stockés pendant 30 jours.

**Q : Pourquoi le nombre de consentements est-il différent dans le rapport des conditions d’utilisation et dans les journaux d’audit Azure AD ?**<br />
A : Le rapport des conditions d’utilisation est stocké pendant toute la durée de vie de celles-ci, alors que les journaux d’audit Azure AD sont stockés pendant 30 jours. En outre, le rapport des conditions d’utilisation affiche uniquement l’état actuel du consentement utilisateur. Par exemple, si un utilisateur refuse, puis accepte les conditions d’utilisation, le rapport affichera uniquement son acceptation. Si vous avez besoin de consulter l’historique, vous pouvez utiliser les journaux d’audit Azure AD.

**Q : Si je modifie les détails des conditions d’utilisation, les utilisateurs doivent-ils les réaccepter ?**<br />
A : Non. Si un administrateur modifie les détails des conditions d’utilisation (nom, nom d’affichage, demander aux utilisateurs de développer les conditions ou ajouter une langue), les utilisateurs ne sont pas tenus de réaccepter les nouvelles conditions.

**Q : Puis-je mettre à jour un document de conditions d’utilisation existant ?**<br />
A : Il n’est pas possible à l’heure actuelle de mettre à jour un document de conditions d’utilisation existant. Il vous faudra pour cela créer une nouvelle instance de conditions d’utilisation.

**Q : Si des liens hypertexte se trouvent dans les conditions d’utilisation d’un document PDF, les utilisateurs finaux peuvent-ils cliquer dessus ?**<br />
A : Oui, les utilisateurs finaux sont en mesure de sélectionner des liens hypertexte vers des pages supplémentaires, mais les liens vers les sections du document ne sont pas pris en charge. En outre, les liens hypertexte dans les PDF de conditions d’utilisation ne fonctionnent pas quand vous y accédez à partir du portail MyAccount/Azure AD MyApps.

**Q : Les conditions d’utilisation prennent-elles en charge plusieurs langues ?**<br />
A : Oui. Actuellement, un administrateur peut configurer 108 langues différentes pour des conditions d’utilisation. Un administrateur peut charger plusieurs documents PDF et les étiqueter avec une langue correspondante (jusqu’à 108). Quand les utilisateurs finaux se connectent, nous examinons les préférences linguistiques de leur navigateur et affichons le document correspondant. En l’absence de correspondance, nous affichons le document par défaut (c’est-à-dire le premier qui a été chargé).

**Q : À quel moment les conditions d’utilisation sont-elles déclenchées ?**<br />
R : Les conditions d’utilisation sont déclenchées durant la procédure de connexion.

**Q : Quelles sont les applications pouvant être ciblées par des conditions d’utilisation ?**<br />
A : Vous pouvez créer une stratégie d’accès conditionnel pour les applications d’entreprise à l’aide de l’authentification moderne. Pour plus d’informations, consultez [Applications d’entreprise](./../manage-apps/view-applications-portal.md).

**Q : Puis-je ajouter plusieurs conditions d’utilisation à une application ou un utilisateur donné ?**<br />
A : Oui. Pour cela, créez plusieurs stratégies d’accès conditionnel ciblant ces groupes ou applications. Si un utilisateur est concerné par plusieurs conditions d’utilisation, il doit les accepter les unes après les autres.

**Q : Que se passe-t-il si un utilisateur refuse les conditions d’utilisation ?**<br />
R : L’utilisateur n’a pas accès à l’application. Il doit se reconnecter et accepter les conditions pour se voir octroyer des droits d’accès.

**Q : Est-il possible de refuser les conditions d’utilisation après les avoir acceptées ?**<br />
A : Vous pouvez [consulter les conditions d’utilisation précédemment acceptées](#how-users-can-review-their-terms-of-use), mais vous ne pouvez pas les refuser après les avoir acceptées.

**Q : Que se passe-t-il si j’utilise également les conditions générales d’Intune ?**<br />
A : Si vous avez configuré les conditions d’utilisation d’Azure AD et les [conditions d’utilisation d’Intune](/intune/terms-and-conditions-create), l’utilisateur doit accepter les deux. Pour plus d’informations, voir le [billet de blog sur le choix des conditions appropriées pour votre organisation](https://go.microsoft.com/fwlink/?linkid=2010506&clcid=0x409).

**Q : Quels sont les points de terminaison utilisés par le service Conditions d’utilisation pour l’authentification ?**<br />
A : Le service Conditions d’utilisation utilise les points de terminaison suivants pour l’authentification : https://tokenprovider.termsofuse.identitygovernance.azure.com et https://account.activedirectory.windowsazure.com. Si votre organisation dispose d’une liste verte d’URL pour l’inscription, vous devez ajouter ces points de terminaison à votre liste verte, ainsi que les points de terminaison Azure AD pour vous connecter.

## <a name="next-steps"></a>Étapes suivantes

- [Démarrage rapide : Exiger l’acceptation des conditions d’utilisation avant d’accorder l’accès à des applications cloud](require-tou.md)