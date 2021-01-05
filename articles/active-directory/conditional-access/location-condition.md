---
title: Conditions d’emplacement dans l’accès conditionnel Azure Active Directory
description: Découvrez comment utiliser la condition d’emplacement pour contrôler l’accès à vos applications cloud basées sur un emplacement réseau de l’utilisateur.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 11/24/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.custom: contperf-fy20q4
ms.openlocfilehash: b647b5ee9dbd7a0447bbb69a47db76685daa1102
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97030193"
---
# <a name="using-the-location-condition-in-a-conditional-access-policy"></a>Utilisation de la condition d’emplacement dans une stratégie d’accès conditionnel 

Comme expliqué dans [l’article de vue d’ensemble](overview.md), une stratégie d’accès conditionnel est à la base constituée d’une instruction de type « if-then » combinant des signaux, qui permet de prendre des décisions et d’appliquer les stratégies de l’organisation. L’un des signaux pouvant être inclus dans le processus de prise de décision est l’emplacement réseau.

![Signal conditionnel conceptuel + Décision = Application](./media/location-condition/conditional-access-signal-decision-enforcement.png)

Les organisations peuvent utiliser cet emplacement réseau pour les tâches courantes, telles que : 

- Exiger l’authentification multifacteur pour les utilisateurs accédant à un service lorsqu’ils sont en dehors du réseau d’entreprise.
- Bloquer l’accès pour les utilisateurs accédant à un service à partir de pays ou régions spécifiques.

L’emplacement réseau est déterminé par l’adresse IP publique fournie par le client à Azure Active Directory. Par défaut, les stratégies d’accès conditionnel s’appliquent à toutes les adresses IPv4 et IPv6. 

> [!TIP]
> Les plages d’adresses IPv6 sont uniquement prises en charge dans l’interface **[Emplacement nommé (préversion)](#preview-features)** . 

## <a name="named-locations"></a>Emplacements nommés

Les emplacements sont désignés dans le Portail Azure sous **Azure Active Directory** > **Sécurité** > **Accès conditionnel** > **Emplacements nommés**. Ces emplacements réseau nommés peuvent inclure les plages réseau du siège de l’organisation ou du VPN, ou les plages que vous souhaitez bloquer. 

![Emplacements nommés dans le Portail Azure](./media/location-condition/new-named-location.png)

Pour configurer un emplacement, vous devez fournir au moins un **nom** et la plage d’adresses IP. 

Le nombre d’emplacements nommés que vous pouvez configurer est limité par la taille de l’objet associé dans Azure AD. Vous pouvez configurer des emplacements selon les limitations suivantes :

- Un emplacement nommé avec 1 200 plages d’adresses IPv4 maximum.
- Un maximum de 90 emplacements nommés avec une plage d’adresses IP assignée à chacun d’eux.

> [!TIP]
> Les plages d’adresses IPv6 sont uniquement prises en charge dans l’interface **[Emplacement nommé (préversion)](#preview-features)** . 

### <a name="trusted-locations"></a>Emplacements approuvés

Lors de la création d’un emplacement réseau, un administrateur a la possibilité de marquer un emplacement comme approuvé. 

![Emplacements approuvés dans le Portail Azure](./media/location-condition/new-trusted-location.png)

Cette option peut être prise en compte dans les stratégies d’accès conditionnel dans lesquelles vous pouvez, par exemple, exiger l’inscription à des fins d’authentification multifacteur à partir d’un emplacement réseau approuvé. Elle est également incluse le calcul des risques liés à Azure AD Identity Protection, qui réduit le risque que présente la connexion d’un utilisateur lorsqu’elle provient d’un emplacement signalé comme approuvé.

### <a name="countries-and-regions"></a>Pays et régions

Certaines organisations peuvent choisir de définir les limites IP de régions ou pays entiers sous la forme d’emplacements nommés pour les stratégies d’accès conditionnel. Elles peuvent utiliser ces emplacements lors du blocage du trafic inutile, si elles sont certaines que les utilisateurs acceptés ne seront jamais issus d’un lieu tel que la Corée du Nord. Ces mappages d’adresses IP à des pays sont régulièrement mis à jour. 

> [!NOTE]
> Les plages d’adresses IPv6 ne peuvent pas être mappées à des pays. Seules les adresses IPv4 sont mappées à des pays.

![Créer un emplacement basé sur le pays ou la région dans le Portail Azure](./media/location-condition/new-named-location-country-region.png)

#### <a name="include-unknown-areas"></a>Inclure les zones inconnues

Certaines adresses IP ne sont pas mappées à un pays ou une région spécifique, notamment toutes les adresses IPv6. Pour capturer ces emplacements d’adresses IP, cochez la case **Inclure les zones inconnues** lors de la définition d’un emplacement. Cette option vous permet de choisir si ces adresses IP doivent être incluses dans l’emplacement nommé. Utilisez ce paramètre lorsque la stratégie utilisant l’emplacement nommé doit s’appliquer à des emplacements inconnus.

### <a name="configure-mfa-trusted-ips"></a>Configurer des adresses IP approuvées MFA

Vous pouvez également configurer des plages d’adresses IP représentant l’intranet local de votre organisation dans les [paramètres du service d’authentification multifacteur](https://account.activedirectory.windowsazure.com/usermanagement/mfasettings.aspx). Cette fonctionnalité vous permet de configurer jusqu’à 50 plages d’adresses IP. Les plages d’adresses IP sont au format CIDR. Pour plus d’informations, consultez [Adresses IP approuvées](../authentication/howto-mfa-mfasettings.md#trusted-ips).  

Si vous avez approuvé des adresses IP configurées, elles apparaissent en tant qu’**adresses IP approuvées MFA** dans la liste des emplacements de la condition d’emplacement.

### <a name="skipping-multi-factor-authentication"></a>Ignorer l’authentification multifacteur

Sur la page des paramètres du service d’authentification multifacteur, vous pouvez identifier les utilisateurs de l’intranet d’entreprise en sélectionnant **Ignorer l’authentification multifacteur pour les demandes issues d’utilisateurs fédérés provenant de mon intranet**. Ce paramètre indique que la revendication de réseau d’entreprise interne, qui est émise par AD FS, doit être approuvée et utilisée pour identifier l’utilisateur comme étant sur le réseau d’entreprise. Pour plus d’informations, consultez [Activer la fonctionnalité Adresses IP approuvées à l’aide de l’accès conditionnel](../authentication/howto-mfa-mfasettings.md#enable-the-trusted-ips-feature-by-using-conditional-access).

Après avoir activé cette option, qui inclut l’emplacement nommé, les **Adresses IP approuvées MFA** s’appliqueront à toutes les stratégies contenant cette option sélectionnée.

Pour les applications mobiles et de bureau avec des durées de session active longues, l’accès conditionnel est réévalué régulièrement. La valeur par défaut est une fois une heure. Lorsque la revendication de réseau d’entreprise interne n’est émise qu’au moment de l’authentification initiale, Azure AD peut ne pas contenir de liste de plages d’adresses IP approuvées. Dans ce cas, il est plus difficile de déterminer si l’utilisateur est toujours sur le réseau d’entreprise :

1. Vérifiez si l’adresse IP de l’utilisateur se trouve dans l’une des plages d’adresses IP approuvées.
1. Vérifiez si les trois premiers octets de l’adresse IP de l’utilisateur correspondent aux trois premiers octets de l’adresse IP de l’authentification initiale. L’adresse IP est comparée à l’authentification initiale lorsque la revendication de réseau d’entreprise interne a été émise à l’origine et que l’emplacement de l’utilisateur a été validé.

Si les deux étapes échouent, un utilisateur est considéré comme n’étant plus sur une adresse IP approuvée.

## <a name="preview-features"></a>Fonctionnalités de préversion

En plus de la fonctionnalité d’emplacement nommé généralement disponible, il existe également un emplacement nommé en préversion. Vous pouvez accéder à la préversion de l’emplacement nommé à l’aide de la bannière située en haut du panneau de l’emplacement nommé actuel.

![Essayer la préversion de l’emplacement nommé](./media/location-condition/preview-features.png)

Avec la préversion de l’emplacement nommé, vous pouvez :

- Configurer jusqu’à 195 emplacements nommés
- Configurer jusqu’à 2 000 plages d’adresses IP par emplacement nommé
- Configurer des adresses IPv6 avec des adresses IPv4

Nous avons également ajouté des vérifications supplémentaires pour aider à limiter les changements suite à une configuration incorrecte.

- Les plages d’adresses IP privées ne peuvent plus être configurées.
- Le nombre d’adresses IP pouvant être incluses dans une plage est limité. Seuls les masques CIDR supérieurs à /8 sont autorisés lors de la configuration d’une plage d’adresses IP.

Avec la préversion, il existe désormais deux options de création : 

- **Emplacement des pays**
- **Emplacement des plages d’adresses IP**

> [!NOTE]
> Les plages d’adresses IPv6 ne peuvent pas être mappées à des pays. Seules les adresses IPv4 sont mappées à des pays.

![Interface d’emplacement nommé en préversion](./media/location-condition/named-location-preview.png)

## <a name="location-condition-in-policy"></a>Condition d’emplacement dans une stratégie

Lorsque vous configurez la condition d’emplacement, vous avez la possibilité de faire la distinction entre :

- N’importe quel emplacement
- Tous les emplacements approuvés
- Des emplacements sélectionnés

### <a name="any-location"></a>N’importe quel emplacement

Par défaut, la sélection de **N’importe quel emplacement** entraîne l’application d’une stratégie à toutes les adresses IP, à savoir n’importe quelle adresse sur Internet. Ce paramètre ne se limite pas aux adresses IP que vous avez configurées en tant qu’emplacement nommé. Lorsque vous sélectionnez **N’importe quel emplacement**, vous pouvez toujours exclure des emplacements spécifiques d’une stratégie. Par exemple, vous pouvez appliquer une stratégie à tous les emplacements, sauf des emplacements approuvés, afin de définir l’étendue à tous les emplacements, à l’exception du réseau d’entreprise.

### <a name="all-trusted-locations"></a>Tous les emplacements approuvés

Cette option s’applique à :

- Tous les emplacements qui ont été marqués comme emplacement approuvé
- **Adresses IP approuvées MFA** (si configurées)

### <a name="selected-locations"></a>Des emplacements sélectionnés

Avec cette option, vous pouvez sélectionner un ou plusieurs emplacements nommés. Pour une stratégie avec ce paramètre à appliquer, un utilisateur doit se connecter à partir de l’un des emplacements sélectionnés. Lorsque vous cliquez sur **Sélectionner**, le contrôle de sélection du réseau nommé qui affiche la liste des réseaux nommés s’ouvre. La liste indique également si l’emplacement réseau a été marqué comme approuvé. L’emplacement nommé appelé **Adresses IP approuvées MFA** est utilisé pour inclure les paramètres IP pouvant être configurés dans la page des paramètres du service d’authentification multifacteur.

## <a name="ipv6-traffic"></a>Trafic IPv6

Par défaut, les stratégies d’accès conditionnel s’appliquent à tout le trafic IPv6. Avec la [préversion d’emplacement nommé](#preview-features), vous pouvez exclure des plages d’adresses IPv6 spécifiques d’une stratégie d’accès conditionnel. Cette option est utile dans les cas où vous ne souhaitez pas que la stratégie soit appliquée pour des plages IPv6 spécifiques. Par exemple, si vous ne souhaitez pas appliquer une stratégie pour les utilisations sur votre réseau d’entreprise et que votre réseau d’entreprise est hébergé sur des plages IPv6 publiques.  

### <a name="when-will-my-tenant-have-ipv6-traffic"></a>Quand mon locataire aura-t-il un trafic IPv6 ?

Actuellement, Azure Active Directory (Azure AD) ne prend pas en charge les connexions réseau directes qui utilisent IPv6. Toutefois, dans certains cas, le trafic d’authentification est transféré par proxy via un autre service. Dans ces cas, l’adresse IPv6 est utilisée lors de l’évaluation de la stratégie.

La majeure partie du trafic IPv6 qui est transférée par proxy à Azure AD provient de Microsoft Exchange Online. Lorsqu’elles sont disponibles, Exchange privilégie les connexions IPv6. **Par conséquent, si vous avez des stratégies d’accès conditionnel pour Exchange, qui ont été configurées pour des plages IPv4 spécifiques, vous devez vous assurer que vous avez également ajouté les plages IPv6 de votre organisation.** Si vous n’incluez pas de plages IPv6, vous risquez de provoquer un comportement inattendu pour les deux cas suivants :

- Lorsqu’un client de messagerie est utilisé pour se connecter à Exchange Online avec l’authentification héritée, Azure AD peut recevoir une adresse IPv6. La demande d’authentification initiale est envoyée à Exchange et est ensuite transmise par proxy à Azure AD.
- Quand Outlook Web Access (OWA) est utilisé dans le navigateur, il vérifie périodiquement que toutes les stratégies d’accès conditionnel sont toujours respectées. Cette vérification permet d’intercepter les cas où un utilisateur est passé d’une adresse IP autorisée à un nouvel emplacement, comme un café en ville. Dans ce cas, si une adresse IPv6 est utilisée et si l’adresse IPv6 n’est pas dans une plage configurée, il se peut que la session de l’utilisateur soit interrompue et redirigée vers Azure AD pour une nouvelle authentification. 

Il s’agit des raisons les plus courantes pour lesquelles vous devrez peut-être configurer des plages IPv6 dans vos emplacements nommés. En outre, si vous utilisez des réseaux virtuels Azure, vous recevrez du trafic provenant d’une adresse IPv6. Si le trafic du réseau virtuel est bloqué par une stratégie d’accès conditionnel, vérifiez votre journal des connexions Azure AD. Une fois que vous avez identifié le trafic, vous pouvez obtenir l’adresse IPv6 utilisée et l’exclure de votre stratégie. 

> [!NOTE]
> Si vous souhaitez spécifier une plage CIDR IP pour une seule adresse, appliquez le masque de bits /128. Si vous indiquez que l’adresse IPv6 est 2607:fb90:b27a:6f69:f8d5:dea0:fb39:74a et si vous souhaitez exclure cette adresse unique en tant que plage, utilisez 2607:fb90:b27a:6f69:f8d5:dea0:fb39:74a/128.

### <a name="identifying-ipv6-traffic-in-the-azure-ad-sign-in-activity-reports"></a>Identification du trafic IPv6 dans les rapports d’activité de connexion Azure AD

Vous pouvez découvrir le trafic IPv6 dans votre locataire en accédant aux [rapports d’activité de connexion Azure AD](../reports-monitoring/concept-sign-ins.md). Une fois le rapport d’activité ouvert, ajoutez la colonne « Adresse IP ». Cette colonne vous permettra d’identifier le trafic IPv6.

Vous pouvez également trouver l’adresse IP du client en cliquant sur une ligne du rapport, puis en accédant à l’onglet « Emplacement » dans les détails de l’activité de connexion. 

## <a name="what-you-should-know"></a>Ce que vous devez savoir

### <a name="when-is-a-location-evaluated"></a>Quand un emplacement est-il évalué ?

Les stratégies d’accès conditionnel sont évaluées lorsque :

- Un utilisateur se connecte initialement à une application web, mobile ou de bureau.
- Une application mobile ou de bureau qui utilise une authentification moderne utilise un jeton d’actualisation pour acquérir un nouveau jeton d’accès. Par défaut, la vérification est effectuée une fois par heure.

Ce qui signifie que pour des applications mobiles et de bureau utilisant l’authentification moderne, un changement d’emplacement sera détecté dans l’heure du changement de l’emplacement réseau. Pour les applications mobiles et de bureau n’utilisant pas l’authentification moderne, la stratégie est appliquée sur chaque requête de jeton. La fréquence de la requête peut varier selon l’application. Il en va de même pour les applications web. La stratégie est appliquée à la connexion initiale et est valide pour toute la durée de la session dans l’application web. En raison des différences de durées de session dans les applications, le temps d’évaluation de la stratégie varie aussi. À chaque fois que l’application demande un nouveau jeton de connexion, la stratégie est appliquée.

Par défaut, Azure AD émet un jeton toutes les heures. Après une sortie du réseau d’entreprise, la stratégie est appliquée dans l’heure qui suit pour les applications utilisant l’authentification moderne.

### <a name="user-ip-address"></a>Adresse IP utilisateur

L’adresse IP qui est utilisée dans l’évaluation de la stratégie correspond à l’adresse IP publique de l’utilisateur. Pour les appareils sur un réseau privé, il ne s’agit pas de l’adresse IP cliente de l’appareil de l’utilisateur sur l’intranet ; il s’agit de l’adresse utilisée par le réseau pour se connecter à l’Internet public.

### <a name="bulk-uploading-and-downloading-of-named-locations"></a>Chargement et téléchargement en bloc d’emplacements nommés

Lorsque vous créez ou mettez à jour des emplacements nommés, pour effectuer des mises à jour en bloc, vous pouvez charger ou télécharger un fichier CSV contenant les plages d’adresses IP. Un chargement permet de remplacer les plages d’adresses IP de la liste par celles du fichier. Chaque ligne du fichier contient une plage d’adresses IP au format CIDR.

### <a name="cloud-proxies-and-vpns"></a>Proxies cloud et réseaux privés virtuels

Lorsque vous utilisez un proxy hébergé dans le cloud ou une solution VPN, l’adresse IP Azure AD utilisée pendant l’évaluation d’une stratégie correspond à l’adresse IP du proxy. L’en-tête X-Forwarded-For (XFF) qui contient l’adresse IP publique de l’utilisateur n’est pas utilisé car il n’existe aucune validation provenant d’une source approuvée, il pourrait donc permettre de falsifier une adresse IP.

Lorsqu’un proxy cloud est disponible, une stratégie utilisée pour exiger un appareil joint Azure AD hybride peut être utilisée, ou la revendication de réseau d’entreprise interne à partir d’AD FS.

### <a name="api-support-and-powershell"></a>Prise en charge des API et PowerShell

Une version préliminaire de l’API Graph pour les emplacements nommés est disponible. pour plus d’informations, consultez [API namedLocation](/graph/api/resources/namedlocation?view=graph-rest-beta).

## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment configurer une stratégie d’accès conditionnel, consultez l’article [Création d’une stratégie d’accès conditionnel](concept-conditional-access-policies.md).
- Vous recherchez un exemple de stratégie utilisant la condition d’emplacement ? Consultez l’article [Accès conditionnel : Bloquer l’accès par emplacement](howto-conditional-access-policy-location.md).
