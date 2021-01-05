---
title: Évaluation de l’accès continu dans Azure AD
description: Réponse plus rapide aux modifications de l’état utilisateur grâce à l’évaluation de l’accès continu dans Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: jlu
ms.collection: M365-identity-device-management
ms.openlocfilehash: b7519b6c7e1f3381be77b9a0734ddda250228e7d
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860301"
---
# <a name="continuous-access-evaluation"></a>Évaluation de l’accès continu

L’expiration et l’actualisation des jetons sont un mécanisme standard du secteur. Lorsqu’une application cliente telle qu’Outlook se connecte à un service comme Exchange Online, les demandes d’API sont autorisées à l’aide de jetons d’accès OAuth 2.0. Par défaut, ces jetons d’accès sont valides pendant une heure. Quand ils expirent, le client est redirigé vers Azure AD pour les actualiser. Cette période d’actualisation offre la possibilité de réévaluer les stratégies d’accès utilisateur. Par exemple, il est possible de choisir de ne pas actualiser le jeton en raison d’une stratégie d’accès conditionnel ou parce que l’utilisateur a été désactivé dans l’annuaire. 

Des clients nous ont fait par de leur préoccupation concernant le décalage entre le moment où des conditions changent pour l’utilisateur, telles qu’un changement d’emplacement réseau ou un vol d’informations d’identification, et le moment où des stratégies peuvent être mises en œuvre en lien avec de tels changement. Nous avons fait l’expérience de l’approche radicale des durées de vie de jeton réduites, mais il s’est avéré qu’elles peuvent nuire à l’expérience utilisateur et à la fiabilité sans éliminer les risques.

Une réponse en temps utile aux violations de stratégie ou autres problèmes de sécurité nécessite vraiment une « conversation » entre l’émetteur du jeton, comme Azure AD, et la partie de confiance, comme Exchange Online. Cette conversation bidirectionnelle nous offre deux capacités importantes. La partie de confiance peut détecter des changements, comme un client provenant d’un nouvel emplacement, et en informer l’émetteur du jeton. Et l’émetteur du jeton peut demander à la partie de confiance d’arrêter de respecter les jetons pour un utilisateur donné en raison d’une compromission ou d’une désactivation de compte ou d’autres soucis. Le mécanisme de cette conversation est l’évaluation continue de l’accès. L’objectif est que la réponse soit en quasi-temps réel mais, dans certains cas, une latence pouvant atteindre 15 minutes peut être observée en raison du temps de propagation de l’événement.

L’implémentation initiale de l’évaluation continue de l’accès se concentre sur Exchange, Teams et SharePoint Online.

Pour préparer vos applications à utiliser l’évaluation continue de l’accès, consultez [Guide pratique pour utiliser les API d’évaluation continue de l’accès dans vos applications](/azure/active-directory/develop/app-resilience-continuous-access-evaluation).

### <a name="key-benefits"></a>Principaux avantages

- Démission d’utilisateur ou modification/réinitialisation du mot de passe : La révocation de session utilisateur sera appliquée en quasi-temps réel.
- Changement d’emplacement réseau : les stratégies d’emplacement d’accès conditionnel seront appliquées en quasi-temps réel.
- L’exportation de jetons vers une machine en dehors d’un réseau approuvé peut être évitée avec des stratégies d’emplacement d’accès conditionnel.

## <a name="scenarios"></a>Scénarios 

L’évaluation continue de l’accès se décline en deux scénarios : l’évaluation des événements critiques et l’évaluation de la stratégie d’accès conditionnel.

### <a name="critical-event-evaluation"></a>Évaluation des événements critiques

L’évaluation continue de l’accès est implémentée en permettant à des services tels qu’Exchange Online, SharePoint Online et Microsoft Teams, de s’abonner à des événements critiques dans Azure AD, de façon à ce que ces événements puissent être évalués et traités en quasi-temps réel. L’évaluation des événements critiques ne reposant pas sur des stratégies d’accès conditionnel, elle est disponible dans n’importe quel locataire. Les événements suivants sont actuellement évalués :

- Le compte d’utilisateur est supprimé ou désactivé
- Le mot de passe d’un utilisateur a été modifié ou réinitialisé
- L’authentification multifacteur est activée pour l’utilisateur
- Un administrateur révoque explicitement tous les jetons d’actualisation d’un utilisateur
- Risque utilisateur élevé détecté par Azure AD Identity Protection

Ce processus permet l’émergence du scénario dans lequel des utilisateurs perdent l’accès à des fichiers, à des e-mails, à des éléments du calendrier ou à des tâches SharePoint Online de l’organisation, ainsi qu’à Teams à partir d’applications clientes Microsoft 365 quelques minutes après la survenance de l’un de ces événements critiques. 

### <a name="conditional-access-policy-evaluation-preview"></a>Évaluation de la stratégie d’accès conditionnel (préversion)

Exchange et SharePoint sont en mesure de synchroniser des stratégies d’accès conditionnel clés afin qu’elles puissent être évaluées dans le service lui-même.

Ce processus permet l’émergence du scénario dans lequel des utilisateurs perdent l’accès à des fichiers, à des e-mails, à des éléments du calendrier ou à des tâches de l’organisation à partir d’applications clientes Microsoft 365 ou de SharePoint Online immédiatement après des changements d’emplacement réseau.

> [!NOTE]
> Les combinaisons de fournisseurs d’applications et de ressources ne sont pas toutes prises en charge. Consultez le tableau ci-dessous. Office fait référence à Word, Excel et PowerPoint

| | Outlook Web | Outlook Win32 | Outlook iOS | Outlook Android | Outlook Mac |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **SharePoint Online** | Prise en charge | Prise en charge | Non pris en charge | Non pris en charge | Prise en charge |
| **Exchange Online** | Prise en charge | Prise en charge | Prise en charge | Prise en charge | Prise en charge |

| | Applications web Office | Applications Win32 Office | Office pour iOS | Office pour Android | Office pour Mac |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **SharePoint Online** | Non pris en charge | Prise en charge | Prise en charge | Prise en charge | Prise en charge |
| **Exchange Online** | Non pris en charge | Prise en charge | Prise en charge | Prise en charge | Prise en charge |

### <a name="client-side-claim-challenge"></a>Contestation de revendication côté client

Avant l’évaluation de l’accès continu, les clients essayaient toujours de relire le jeton d’accès à partir de leur cache tant qu’il n’avait pas expiré. Avec l’EAC, nous introduisons un nouveau cas où un fournisseur de ressources peut rejeter un jeton même s’il n’a pas expiré. Pour informer les clients qu’ils doivent contourner leur cache, même si les jetons mis en cache n’ont pas expiré, nous introduisons un mécanisme appelé **contestation de revendication** destiné à indiquer que le jeton a été rejeté et qu’un nouveau jeton d’accès doit être émis par Azure AD. L’EAC requiert une mise à jour du client pour comprendre la constestation de revendication. La dernière version des applications suivantes prend en charge la contestation de revendication :

- Outlook Windows
- Outlook iOS
- Outlook Android
- Outlook Mac
- Outlook Web App
- Teams pour Windows (uniquement pour les ressources Teams)
- Teams iOS (uniquement pour les ressources Teams)
- Teams Android (uniquement pour les ressources Teams)
- Teams Mac (uniquement pour les ressources Teams)
- Word/Excel/PowerPoint pour Windows
- Word/Excel/PowerPoint pour iOS
- Word/Excel/PowerPoint pour Android
- Word/Excel/PowerPoint pour Mac

### <a name="token-lifetime"></a>Durée de vie des jetons

Étant donné que les risques et les stratégies sont évalués en temps réel, les clients qui négocient des sessions prenant en charge l’évaluation de l’accès continu s’appuient sur l’EAC au lieu des stratégies existantes de durée de vie des jetons d’accès statiques, ce qui signifie que la stratégie configurable de durée de vie des jetons n’est plus respectée pour les clients prenant en charge l’EAC.

La durée de vie du jeton est allongée jusqu’à 28 heures dans les sessions d’évaluation continue de l’accès. La révocation est motivée par des événements critiques et l’évaluation de la stratégie. Il ne s’agit pas d’une période arbitraire. Cette modification renforce la stabilité des applications sans compromettre la posture de sécurité. 

Si vous n’utilisez pas de clients compatibles avec l’évaluation continue de l’accès, la durée de vie par défaut de votre jeton d’accès restera d’une heure, sauf si vous l’avez configurée avec la fonctionnalité [Durée de vie du jeton configurable (CTL)](../develop/active-directory-configurable-token-lifetimes.md) en préversion.

## <a name="example-flows"></a>Exemples de flux

### <a name="user-revocation-event-flow"></a>Flux d’événements de révocation des utilisateurs :

![Flux d’événements de révocation des utilisateurs](./media/concept-continuous-access-evaluation/user-revocation-event-flow.png)

1. Un client compatible avec l’évaluation continue de l’accès présente à Azure AD des informations d’identification ou un jeton d’actualisation demandant un jeton d’accès pour une certaine ressource.
1. Un jeton d’accès est retourné au client avec d’autres artefacts.
1. Un administrateur [révoque explicitement tous les jetons d’actualisation d’un utilisateur](/powershell/module/azuread/revoke-azureaduserallrefreshtoken). Un événement de révocation sera envoyé au fournisseur de ressources à partir d’Azure AD.
1. Un jeton d’accès est présenté au fournisseur de ressources. Le fournisseur de ressources évalue la validité du jeton et vérifie s’il existe un événement de révocation pour l’utilisateur. Le fournisseur de ressources utilise ces informations pour décider d’accorder ou non l’accès à la ressource.
1. Dans ce cas, le fournisseur de ressources refuse l’accès et renvoie une contestation de revendication 401+ au client.
1. Le client compatible avec l’EAC comprend la contestation de revendication 401+. Il contourne les caches et revient à l’étape 1, en renvoyant à Azure AD son jeton d’actualisation avec la contestation de revendication. Azure AD réévalue ensuite toutes les conditions et invite l’utilisateur à se réauthentifier dans ce cas.

### <a name="user-condition-change-flow-preview"></a>Flux de changement de condition utilisateur (préversion) :

Dans l’exemple suivant, un administrateur d’accès conditionnel a configuré une stratégie d’accès conditionnel basée sur l’emplacement afin de n’autoriser l’accès qu’à partir de plages d’adresses IP spécifiques :

![Flux d’événements de condition utilisateur](./media/concept-continuous-access-evaluation/user-condition-change-flow.png)

1. Un client compatible avec l’évaluation continue de l’accès présente à Azure AD des informations d’identification ou un jeton d’actualisation demandant un jeton d’accès pour une certaine ressource.
1. Azure AD évalue toutes les stratégies d’accès conditionnel pour déterminer si l’utilisateur et le client remplissent les conditions.
1. Un jeton d’accès est retourné au client avec d’autres artefacts.
1. L’utilisateur quitte une plage d’adresses IP autorisées
1. Le client présente un jeton d’accès au fournisseur de ressources en dehors d’une plage d’adresses IP autorisées.
1. Le fournisseur de ressources évalue la validité du jeton et vérifie la synchronisation de la stratégie d’emplacement à partir d’Azure AD.
1. Dans ce cas, le fournisseur de ressources refuse l’accès et envoie une demande de revendication 401+ au client, car la tentative d’accès ne provient pas de la plage d’adresses IP autorisée.
1. Le client compatible avec l’EAC comprend la contestation de revendication 401+. Il contourne les caches et revient à l’étape 1, en renvoyant à Azure AD son jeton d’actualisation avec la contestation de revendication. Dans ce cas, Azure AD réévalue toutes les conditions et refuse l’accès.

## <a name="enable-or-disable-cae-preview"></a>Activer ou désactiver l’évaluation continue de l’accès (préversion)

1. Connectez-vous au **portail Azure** en tant qu’administrateur de l’accès conditionnel, administrateur de la sécurité ou administrateur général.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Évaluation continue de l’accès**.
1. Choisissez **Activer la préversion**.

À partir de cette page, vous pouvez limiter les utilisateurs et les groupes bénéficiaire de la préversion.

![Activation de la préversion de l’évaluation continue de l’accès dans le portail Azure](./media/concept-continuous-access-evaluation/enable-cae-preview.png)

## <a name="troubleshooting"></a>Dépannage

### <a name="supported-location-policies"></a>Stratégies d’emplacement prises en charge

Pour l’évaluation continue de l’accès, nous n’avons d’insights que sur les emplacements nommés sur la base de l’adresse IP. Nous n’avons pas d’insights sur d’autres paramètres d’emplacement, tels que des [adresses IP approuvées pour l’authentification multifacteur](../authentication/howto-mfa-mfasettings.md#trusted-ips) ou des emplacements basés sur le pays. Quand la provenance de l’utilisateur est une adresse IP approuvée pour l’authentification multifacteur ou un emplacement approuvé, à savoir une adresse IP approuvée pour l’authentification multifacteur ou un emplacement de pays, l’évaluation continue de l’accès n’est pas appliquée après le déplacement de l’utilisateur vers un autre emplacement. Dans ce cas, nous émettons un jeton d’évaluation continue de l’accès d’une heure sans vérification de l’application d’adresse IP instantanée.

> [!IMPORTANT]
> Quand vous configurez des emplacements pour l’évaluation continue de l’accès, utilisez uniquement la [condition d’emplacement d’accès conditionnel basé sur l’adresse IP](../conditional-access/location-condition.md#preview-features), et configurez toutes les adresses IP, **y compris IPv4 et IPv6**, que votre fournisseur d’identité et votre fournisseur de ressources peuvent voir. N’utilisez pas les conditions d’emplacement de pays ni la fonctionnalité d’adresses IP approuvées disponibles dans la page des paramètres de service Authentification multifacteur Azure AD.

### <a name="ip-address-configuration"></a>Configuration de l'adresse IP

Votre fournisseur d’identité et vos fournisseurs de ressources peuvent voir des adresses IP différentes. Cette incompatibilité peut être due à des implémentations de proxy réseau dans votre organisation ou à des configurations IPv4/IPv6 incorrectes entre votre fournisseur d’identité et votre fournisseur de ressources. Par exemple :

- Votre fournisseur d’identité voit une adresse IP du client.
- Votre fournisseur de ressources voit une adresse IP du client différente après un passage via un proxy.
- L’adresse IP que voit votre fournisseur d’identité fait partie d’une plage d’adresses IP autorisées dans la stratégie, mais pas l’adresse IP du fournisseur de ressources.

Si ce scénario existe dans votre environnement, afin d’éviter les boucles infinies, Azure AD émet un jeton d’évaluation continue de l’accès d’une heure, et n’applique pas le changement d’emplacement de client. Même dans ce cas, la sécurité est améliorée par rapport aux jetons d’une heure traditionnels, car nous évaluons encore les [autres événements](#critical-event-evaluation) à côté des événements de changement d’emplacement de client.

### <a name="office-and-web-account-manager-settings"></a>Paramètres du gestionnaire de comptes web et Office

| Canal de mise à jour Azure | DisableADALatopWAMOverride | DisableAADWAM |
| --- | --- | --- |
| Canal d’entreprise semi-annuel | Si la valeur est Activé ou 1, l’évaluation continue de l’accès n’est pas prise en charge. | Si la valeur est Activé ou 1, l’évaluation continue de l’accès n’est pas prise en charge. |
| Canal actuel <br> or <br> Canal d’entreprise mensuel | L’évaluation continue de l’accès est prise en charge, quel que soit le paramètre | L’évaluation continue de l’accès est prise en charge, quel que soit le paramètre |

Pour une explication concernant les canaux Office Update, consultez [Vue d’ensemble des canaux de mise à jour pour Microsoft 365 Apps](/deployoffice/overview-update-channels). Nous recommandons aux organisations de ne pas désactiver le Gestionnaire de comptes web (WAM).

### <a name="policy-change-timing"></a>Synchronisation des changements de stratégie

En raison du retard potentiel de réplication entre Azure AD et les fournisseurs de ressources, jusqu’à 2 heures peuvent s’écouler avant que les changements de stratégie opérés par les administrateurs prennent effet pour Exchange Online.

Exemple : À 11h00, un administrateur ajoute une stratégie pour empêcher une plage d’adresses IP d’accéder à la messagerie. Un utilisateur arrivé avant cette heure, dont l’adresse IP figure dans cette plage d’adresses, pourrait continuer à accéder au courrier jusqu’à 13h00.

### <a name="coauthoring-in-office-apps"></a>Co-création dans les applications Office

Quand plusieurs utilisateurs collaborent simultanément sur le même document, l’accès de l’utilisateur au document peut ne pas être immédiatement révoqué par l’évaluation continue de l’accès, en fonction d’événements de révocation d’utilisateur ou de changement de stratégie. Dans ce cas, l’utilisateur perd complètement l’accès par la suite, quand il ferme le document, quand il ferme Word, Excel ou PowerPoint, ou après une période de 10 heures.

Pour réduire cette durée, un administrateur SharePoint peut réduire la durée de vie maximale des sessions de cocréation pour les documents stockés dans SharePoint Online et OneDrive Entreprise, en [configurant une stratégie d’emplacement réseau dans SharePoint Online](/sharepoint/control-access-based-on-network-location). Une fois cette configuration modifiée, la durée de vie maximale des sessions de cocréation est réduite à 15 minutes, et peut être ajustée davantage à l’aide de la commande SharePoint Online PowerShell « Set-SPOTenant –IPAddressWACTokenLifetime ».

### <a name="enable-after-a-user-is-disabled"></a>Activer après la désactivation d’un utilisateur

Si vous activez un utilisateur juste après qu’il a été désactivé. Il y aura une certaine latence avant que le compte puisse être activé. SharePoint Online et Teams accuseront un retard de 15 minutes. Le retard est de 35-40 minutes pour Exchange Online.

## <a name="faqs"></a>FAQ

### <a name="how-will-cae-work-with-sign-in-frequency"></a>Comment l’EAC fonctionnera-t-elle avec la fréquence de connexion ?

La fréquence de connexion sera respectée avec ou sans EAC.

## <a name="next-steps"></a>Étapes suivantes

[Annonce de l’évaluation de l’accès continu](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/moving-towards-real-time-policy-and-security-enforcement/ba-p/1276933)
