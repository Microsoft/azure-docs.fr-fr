---
title: Autoriser l’accès avec une signature d’accès partagé dans Azure Event Hubs
description: Cet article fournit des informations sur l’autorisation de l’accès aux ressources Azure Event Hubs à l’aide de signatures d’accès partagé.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 6a2d7385f82864e8d378055333377fb9c3f73c19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85323120"
---
# <a name="authorizing-access-to-event-hubs-resources-using-shared-access-signatures"></a>Autorisation de l’accès aux ressources Event Hubs à l’aide de signatures d’accès partagé
Une signature d’accès partagé (SAS, Shared Access Signature) vous offre un moyen d’accorder un accès limité aux ressources de votre espace de noms Event Hubs. Une signature d’accès partagé protège l’accès aux ressources Event Hubs en fonction de règles d’autorisation. Ces règles sont configurées sur un espace de noms ou une entité (hub d’événements ou rubrique). Cet article offre une vue d’ensemble du modèle SAS et passe en revue les bonnes pratiques relatives aux signatures d’accès partagé.

## <a name="what-are-shared-access-signatures"></a>Que sont les signatures d’accès partagé ?
Une signature d’accès partagé fournit un accès délégué aux ressources Event Hubs en fonction de règles d’autorisation. Une règle d’autorisation a un nom, est associée à des droits spécifiques et comporte une paire de clés de chiffrement. Vous utilisez le nom et la clé de la règle par le biais des clients Event Hubs ou dans votre propre code pour générer des jetons SAS. Un client peut ensuite passer le jeton à Event Hubs pour prouver l’autorisation relative à l’opération demandée.

Une signature d’accès partagé est un mécanisme d’autorisation reposant sur des revendications et utilisant de simples jetons. À l’aide d’une SAP, les clés ne sont jamais transmises simultanément. Les clés sont utilisées pour signer par chiffrement des informations qui peuvent être vérifiées ultérieurement par le service. Une SAP peut être utilisée de façon similaire à un schéma de nom d’utilisateur et de mot de passe où le client est en possession immédiate d’un nom de règle d’autorisation et d’une clé correspondante. Une signature d’accès partagé peut être utilisée comme un modèle de sécurité fédéré, où le client reçoit un jeton d’accès signé et limité dans le temps d’un service d’émission de jeton de sécurité sans jamais être en possession de la clé de signature.

> [!NOTE]
> Azure Event Hubs prend en charge l’utilisation d’Azure Active Directory (Azure AD) pour autoriser l’accès aux ressources Event Hubs. L’autorisation des utilisateurs ou des applications avec un jeton OAuth 2.0 retourné par Azure AD assure une meilleure sécurité que les signatures d’accès partagé. De plus, elle offre une plus grande simplicité d’utilisation. Azure AD vous évite d’avoir à stocker les jetons dans votre code. Vous êtes ainsi moins exposé au risque de failles de sécurité.
>
> Microsoft recommande d’utiliser Azure AD avec les applications Azure Event Hubs dans la mesure du possible. Pour plus d’informations, consultez [Autoriser l’accès aux ressources Event Hubs à l’aide d’Azure Active Directory](authorize-access-azure-active-directory.md).

> [!IMPORTANT]
> Les jetons SAS sont essentiels à la protection de vos ressources. Ils accordent aux clients l’accès à vos ressources Event Hubs tout en offrant une certaine précision. Ils ne doivent pas être partagés publiquement. Quand un partage est nécessaire dans le cadre de dépannage, pensez à utiliser une version réduite de tous les fichiers journaux ou à supprimer les jetons SAS (le cas échéant) des fichiers journaux. Vérifiez également que les captures d’écran ne contiennent pas d’informations relatives aux signatures d’accès partagé.

## <a name="shared-access-authorization-policies"></a>Stratégies d’autorisation d’accès partagé
Chaque espace de noms Event Hubs et chaque entité Event Hubs (instance de hub d’événements ou rubrique Kafka) utilisent une stratégie d’autorisation d’accès partagé constituée de règles. La stratégie au niveau de l’espace de noms s’applique à toutes les entités à l’intérieur de l’espace de noms, quelle que soit leur configuration de stratégie individuelle.
Pour chaque règle de stratégie d’autorisation, vous choisissez trois éléments d’information : le nom, l’étendue et les droits. Le nom est unique dans cette étendue. L’étendue est l’URI de la ressource en question. Pour un espace de noms Event Hubs, l’étendue est le nom de domaine complet (FQDN, fully qualified domain name) tel que `https://<yournamespace>.servicebus.windows.net/`.

Les droits attribués par la règle de stratégie peuvent être une combinaison de :
- **Envoyer** : donne le droit d’envoyer des messages à l’entité
- **Écouter** : donne le droit d’écouter l’entité ou de recevoir des données de l’entité
- **Gérer** : donne le droit de gérer la topologie de l’espace de noms et notamment de créer et supprimer des entités

> [!NOTE]
> Le droit **Gérer** inclut les droits **Envoyer** et **Recevoir**.

Une stratégie d’entité ou d’espace de noms peut contenir jusqu’à 12 règles d’autorisation d’accès partagé, ce qui laisse de l’espace à trois ensembles de règles, chacun couvrant les droits basiques et la combinaison des droits Envoyer et Écouter. Cette limite souligne que le magasin de stratégies SAS n’est pas destiné à être un magasin de comptes de service ou d’utilisateur. Si votre application doit accorder l’accès aux ressources Event Hubs en fonction des identités de service ou d’utilisateur, elle doit implémenter un service d’émission de jeton de sécurité qui émet des jetons SAS après une authentification et une vérification de l’accès.

Une **clé primaire** et une **clé secondaire** sont attribuées à une règle d’autorisation. Il s’agit de clés de chiffrement fortes. Ne les perdez pas et ne les divulguez pas. Elles seront toujours disponibles sur le portail Azure. Vous pouvez utiliser n’importe laquelle des clés générées et vous pouvez les régénérer à tout moment. Si vous régénérez ou modifiez une clé dans la stratégie, tous les jetons précédemment émis en fonction de cette clé deviennent instantanément non valides. Toutefois, les connexions en cours créées en fonction de ces jetons continuent de fonctionner jusqu’à ce que le jeton expire.

Quand vous créez un espace de noms Event Hubs, une règle de stratégie nommée **RootManageSharedAccessKey** est automatiquement créée pour l’espace de noms. Cette stratégie dispose d’autorisations **Gérer** pour l’espace de noms complet. Il est recommandé de traiter cette règle comme un compte racine d’administration et de ne pas l’utiliser dans votre application. Vous pouvez créer des règles de stratégies supplémentaires sous l’onglet **Configurer** pour l’espace de noms dans le portail via PowerShell ou Azure CLI.

## <a name="best-practices-when-using-sas"></a>Bonnes pratiques lors de l’utilisation de SAP
Lorsque vous utilisez des signatures d'accès partagé dans vos applications, vous devez être conscient de deux risques potentiels :

- Si une signature d’accès partagé est divulguée, toute personne qui se la procure peut s’en servir, ce qui peut compromettre vos ressources Event Hubs.
- Si une signature d’accès partagé fournie à une application cliente expire et que l’application est incapable d’en récupérer une nouvelle à partir de votre service, le fonctionnement de votre application risque d’être entravé.

Les recommandations suivantes relatives à l’utilisation des signatures d’accès partagé peuvent aider à limiter ces risques :

- **Faites en sorte que les clients renouvellent automatiquement la signature d’accès partagé si nécessaire** : les clients doivent renouveler la signature d’accès partagé bien avant l’heure d’expiration pour laisser suffisamment de temps pour de nouvelles tentatives si le service qui fournit la signature est indisponible. Si votre signature d’accès partagé doit être utilisée pour un petit nombre d’opérations immédiates de courte durée, censées être terminées avant l’heure d’expiration, cela ne sera peut-être pas nécessaire, car il n’est pas prévu que la signature d’accès partagé soit renouvelée. Toutefois, si vous avez un client qui effectue régulièrement des demandes par le biais de signatures d'accès partagé, le risque d'expiration est à prendre en compte. La principale considération consiste à trouver un équilibre entre la nécessité que la signature d’accès partagé ait une durée de vie limitée (comme indiqué plus haut) et la nécessité de veiller à ce que le client demande le renouvellement suffisamment tôt pour éviter une interruption due à une expiration de la signature avant le renouvellement effectif.
- **Faites attention à la date de début de la signature d’accès partagé** : si vous définissez la date de début d’une signature d’accès partagé sur **maintenant**, en raison du décalage d’horloge (différences constatées dans l’heure actuelle sur des machines différentes), des défaillances peuvent être observées par intermittence pendant les premières minutes. En règle générale, définissez une heure de début située au moins 15 minutes avant l’heure courante ou ne la définissez pas du tout, et elle sera alors valide immédiatement dans tous les cas. Généralement, le même principe s’applique également à l’heure d’expiration. N’oubliez pas que vous pouvez observer jusqu’à 15 minutes de décalage d’horloge (dans un sens ou dans l’autre) sur une demande. 
- **Soyez précis quant à la ressource pour laquelle vous voulez configurer l’accès** : il est recommandé de fournir à l’utilisateur les privilèges minimaux requis, ce qui s’inscrit dans les bonnes pratiques de sécurité. Si un utilisateur a besoin d'un accès en lecture à une seule entité, accordez-lui un accès en lecture à cette seule entité, plutôt qu'un accès en lecture/écriture/suppression à toutes les entités. Cela permet également d’atténuer les dégâts si une signature d’accès partagé est compromise, car son pouvoir est moindre entre les mains d’un attaquant.
- **N’utilisez pas toujours une signature d’accès partagé** : parfois, les risques associés à une opération particulière sur vos hubs d’événements l’emportent sur les avantages offerts par la signature d’accès partagé. Pour ces opérations, créez un service de niveau intermédiaire qui écrit dans vos hubs d’événements après avoir effectué la validation des règles métier, l’authentification et un audit.
- **Utilisez toujours HTTPS** : utilisez toujours HTTPS pour créer ou distribuer une signature d’accès partagé. Si une signature d’accès partagé est transmise sur HTTP et interceptée, un attaquant qui lance une attaque de type « attaque de l’intercepteur » (man-in-the-middle) peut lire la signature et s’en servir exactement comme l’utilisateur concerné aurait pu le faire, d’où le risque que les données sensibles soient compromises ou que les données soient altérées par l’utilisateur malveillant.

## <a name="conclusion"></a>Conclusion
Les signatures d’accès partagé sont utiles pour fournir à vos clients des autorisations d’accès limitées aux ressources Event Hubs. Elles sont un élément crucial du modèle de sécurité pour toute application utilisant Azure Event Hubs. Si vous suivez les bonnes pratiques présentées dans cet article, vous pouvez utiliser une signature d’accès partagé pour offrir une plus grande souplesse d’accès à vos ressources sans compromettre la sécurité de votre application.

## <a name="next-steps"></a>Étapes suivantes
Consultez les articles associés suivants : 

- [Authentifier les requêtes adressées à Azure Event Hubs à partir d’une application à l’aide d’Azure Active Directory](authenticate-application.md)
- [Authentifier une identité managée avec Azure Active Directory pour accéder aux ressources Event Hubs](authenticate-managed-identity.md)
- [Authentifier les requêtes adressées à Azure Event Hubs à l’aide de signatures d’accès partagé](authenticate-shared-access-signature.md)
- [Autoriser l’accès aux ressources Event Hubs à l’aide d’Azure Active Directory](authorize-access-azure-active-directory.md)


