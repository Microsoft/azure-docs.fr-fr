---
title: Réécriture du mot de passe localement avec réinitialisation de mot de passe en libre-service - Azure Active Directory
description: Découvrir comment les événements de changement ou de réinitialisation de mot de passe dans Azure Active Directory peuvent être réécrits dans un environnement d’annuaire local
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 53f416a23dbb47660097c41ada09c8c135434bcb
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743647"
---
# <a name="how-does-self-service-password-reset-writeback-work-in-azure-active-directory"></a>Comment fonctionne la réécriture de la réinitialisation de mot de passe en libre-service dans Azure Active Directory ?

La technologie SSPR (réinitialisation de mot de passe en libre-service) d’Azure AD (Azure Active Directory) permet aux utilisateurs de réinitialiser leurs mots de passe dans le cloud. Toutefois, la plupart des entreprises disposent également d’un environnement AD DS (Active Directory Domain Services) local où sont situés leurs utilisateurs. La réécriture du mot de passe est une fonctionnalité activée avec [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) qui permet aux modifications de mot de passe dans le cloud d’être réécrites sur un annuaire local existant en temps réel. Dans cette configuration, quand les utilisateurs changent ou réinitialisent leurs mots de passe via SSPR dans le cloud, les mots de passe mis à jour sont également réécrits dans l’environnement AD DS local

> [!IMPORTANT]
> Cet article conceptuel explique à un administrateur le fonctionnement de la réécriture de la réinitialisation de mot de passe en libre-service. Si vous êtes un utilisateur final déjà inscrit pour la réinitialisation de mot de passe en libre-service et que vous devez récupérer votre compte, accédez à https://aka.ms/sspr.
>
> Si votre équipe informatique n’a pas activé la réinitialisation de votre propre mot de passe, contactez votre support technique pour obtenir une assistance supplémentaire.

La réécriture du mot de passe est prise en charge dans les environnements qui utilisent les modèles d’identité hybride suivants :

* [Synchronisation de hachage de mot de passe](../hybrid/how-to-connect-password-hash-synchronization.md)
* [Authentification directe](../hybrid/how-to-connect-pta.md)
* [Active Directory Federation Services](../hybrid/how-to-connect-fed-management.md)

La réécriture du mot de passe fournit les fonctionnalités suivantes :

* **Application de stratégies de mot de passe AD DS (Active Directory Domain Services) locales** : quand un utilisateur réinitialise son mot de passe, celui-ci fait l’objet d’une vérification pour garantir qu’il répond à votre stratégie AD DS locale avant d’être validé dans l’annuaire. Cette vérification porte sur l’historique, la complexité, l’âge, les filtres de mot de passe et toute autre restriction de mot de passe définie dans AD DS.
* **Le retour d’informations immédiat** :  L’écriture différée de mot de passe est une opération synchrone. Les utilisateurs sont notifiés immédiatement si leur mot de passe ne respecte pas la stratégie définie, ou s’il ne peut pas être réinitialisé ou changé pour une raison quelconque.
* **La prise en charge de la modification des mots de passe à partir du panneau d’accès et de Microsoft 365** : quand des utilisateurs fédérés ou disposant de la synchronisation du code de hachage de mot de passe changent leurs mots de passe (arrivés ou non à expiration), ces derniers sont réécrits dans AD DS.
* **La prise en charge de la réécriture du mot de passe lorsqu’un administrateur le réinitialise depuis le portail Azure** : quand un administrateur réinitialise le mot de passe d’un utilisateur dans le [portail Azure](https://portal.azure.com), si cet utilisateur est fédéré ou s’il dispose de la synchronisation du code de hachage de mot de passe, le mot de passe est réécrit localement. Actuellement, cette fonctionnalité n’est pas prise en charge dans le portail d’administration Office.
* **Absence de règles de pare-feu obligatoires entrantes** : la réécriture de mot de passe utilise un relais Microsoft Azure Service Bus comme canal de communication sous-jacent. Toutes les communications sont sortantes sur le port 443.

> [!NOTE]
> Les comptes d’administrateur qui existent dans des groupes protégés de votre annuaire AD local peuvent bénéficier de la réécriture du mot de passe. Les administrateurs peuvent changer leur mot de passe dans le cloud, mais ils ne peuvent pas utiliser la réinitialisation de mot de passe pour réinitialiser un mot de passe oublié. Pour plus d’informations sur les groupes protégés, consultez [Comptes et groupes protégés dans AD DS](/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--protected-accounts-and-groups-in-active-directory).

Pour bien démarrer avec la réécriture SSPR, suivez le tutoriel suivant :

> [!div class="nextstepaction"]
> [Tutoriel : Activer la réécriture SSPR (réinitialisation de mot de passe en libre-service)](./tutorial-enable-sspr-writeback.md)

## <a name="how-password-writeback-works"></a>Fonctionnement de la réécriture du mot de passe

Lorsqu’un utilisateur fédéré ou disposant de la synchronisation du hachage de mot de passe réinitialise ou modifie son mot de passe dans le cloud, les événements suivants ont lieu :

1. Vérification du type de mot de passe de l’utilisateur. Si le mot de passe de l’utilisateur est géré localement :
   * Vérification du bon fonctionnement du service de réécriture. Dans le cas, l’utilisateur peut continuer.
   * Si le service de réécriture n’est pas disponible, l’utilisateur est informé que son mot de passe ne peut pas être réinitialisé pour l’instant.
1. Ensuite, l’utilisateur transite par les portails d’authentification appropriés et atteint la page de **réinitialisation du mot de passe**.
1. L’utilisateur sélectionne un nouveau mot de passe et le confirme.
1. Lorsqu’il clique sur **Envoyer**, le mot de passe en texte brut est chiffré avec une clé de contenu créée durant le processus de configuration de réécriture.
1. Le mot de passe chiffré est inclus dans une charge utile qui est envoyée via un canal HTTPS à votre relais Service Bus spécifique à l’abonné (configuré pour vous lors du processus d’installation de la réécriture). Ce relais est protégé par un mot de passe généré de manière aléatoire, connu uniquement de votre installation locale.
1. Une fois que le message a atteint le Service Bus, le point de terminaison de réinitialisation de mot de passe sort automatiquement du mode veille et découvre qu’une demande de réinitialisation est en attente.
1. Ce service recherche alors l’utilisateur concerné à l’aide de l’attribut d’ancrage de cloud. Pour que cette recherche réussisse, les conditions suivantes doivent être remplies :

   * L’objet utilisateur doit exister dans l’espace connecteur AD DS.
   * L’objet utilisateur doit être lié à l’objet métaverse (MV) correspondant.
   * L’objet utilisateur doit être lié à l’objet connecteur Azure AD correspondant.
   * Le lien de l’objet connecteur AD DS vers le MV doit avoir la règle de synchronisation `Microsoft.InfromADUserAccountEnabled.xxx` sur le lien.

   Quand l’appel provient du cloud, le moteur de synchronisation utilise l’attribut **cloudAnchor** pour rechercher l’objet CS (Connector Space) Azure AD. Il suit ensuite le lien vers l’objet MV, puis le lien vers l’objet AD DS. Comme il peut exister plusieurs objets AD DS (multiforêts) pour le même utilisateur, le moteur de synchronisation s’appuie sur le lien `Microsoft.InfromADUserAccountEnabled.xxx` pour choisir celui qui convient.

1. Une fois le compte d’utilisateur trouvé, une tentative de réinitialisation du mot de passe directement dans la forêt AD DS appropriée est effectuée.
1. Si l’opération de définition du mot de passe réussit, l’utilisateur reçoit le message que son mot de passe a été modifié.

   > [!NOTE]
   > Si le code de hachage de mot de passe de l’utilisateur est synchronisé à Azure AD à l’aide de la synchronisation du code de hachage de mot de passe, il est possible que la stratégie de mot de passe locale soit plus faible que la stratégie de mot de passe cloud. Dans ce cas, la stratégie locale est appliquée. Cela garantit que votre stratégie locale est appliquée dans le cloud, que vous utilisiez ou non la synchronisation ou la fédération de hachage de mot de passe pour fournir une authentification unique.

1. Si l’opération de définition du mot de passe échoue, un message d’erreur invite l’utilisateur à réessayer. L’opération peut être un échec pour les raisons suivantes :
    * Le service était arrêté.
    * Le mot de passe sélectionné ne répond pas aux stratégies de l’organisation.
    * L’utilisateur est introuvable dans l’environnement AD DS local.

   Les messages d’erreur fournissent des conseils aux utilisateurs afin qu’ils tentent de résoudre leur problème sans intervention de l’administrateur.

## <a name="password-writeback-security"></a>Sécurité de réécriture du mot de passe

La réécriture du mot de passe est un service hautement sécurisé. Pour garantir la protection de vos informations, un modèle de sécurité à quatre niveaux est activé, comme indiqué ci-dessous :

* **Relais de Service Bus propre au client**
   * Lorsque vous configurez le service, un relais Service Bus spécifique au client est défini et protégé par un mot de passe fort généré de manière aléatoire, auquel Microsoft n’a jamais accès.
* **Clé de chiffrement de mot de passe forte et verrouillée**
   * Une fois le relais Service Bus créé, une paire de clés symétriques fortes est créée, permettant de chiffrer le mot de passe lorsqu’il arrive sur le réseau. Cette clé réside uniquement dans le magasin de secrets de votre entreprise dans le cloud, lequel est fortement verrouillé et audité, comme n’importe quel mot de passe de l’annuaire.
* **Norme TLS (Transport Layer Security)**
   1. Lorsqu’une opération de réinitialisation ou de modification de mot de passe a lieu dans le cloud, le mot de passe en clair est chiffré avec votre clé publique.
   1. Le mot de passe chiffré est placé dans un message HTTPS envoyé à votre instance de Service Bus Relay via un canal chiffré, à l’aide de certificats TLS/SSL Microsoft.
   1. Une fois le message arrivé dans Service Bus, votre agent local sort du mode veille et s’authentifie auprès de Service Bus en utilisant le mot de passe fort précédemment généré.
   1. L’agent local récupère le message chiffré et le déchiffre à l’aide de la clé privée.
   1. L’agent local tente ensuite de définir le mot de passe via l’API SetPassword AD DS. Cette étape est celle qui permet d’appliquer votre stratégie de mot de passe d’environnement AD DS local (complexité, âge, historique et filtres) dans le cloud.
* **Stratégies d’expiration de message**
   * Si étant donné que votre service local est arrêté, le message reste dans Service Bus, il expire et est supprimé après quelques minutes. Le délai d’expiration et la suppression du message renforcent ainsi la sécurité.

### <a name="password-writeback-encryption-details"></a>Informations détaillées sur le chiffrement de la réécriture du mot de passe

Lorsqu’un utilisateur soumet une réinitialisation de mot de passe, cette demande transite via plusieurs étapes de chiffrement avant son arrivée dans votre environnement local. Ces étapes de chiffrement garantissent une sécurité et une fiabilité maximales. En voici le détail :

1. **Chiffrement de mot de passe avec clé RSA de 2 048 bits** : Lorsqu’un utilisateur envoie un mot de passe en vue de sa réécriture en local, celui-ci est préalablement chiffré avec une clé RSA de 2 048 bits.
1. **Chiffrement au niveau du package avec AES-GCM** : La méthode AES-GCM permet de chiffrer la totalité du package, avec le mot de passe et les métadonnées nécessaires. Ce chiffrement empêche quiconque ayant un accès direct au canal Service Bus sous-jacent de visualiser ou de falsifier le contenu.
1. **Toutes les communications passent par une connexion TLS/SSL** : Toutes les communications avec Service Bus ont lieu dans un canal SSL/TLS. Ce chiffrement protège le contenu contre les tiers non autorisés.
1. **Substitution de clé automatique tous les six mois** : Tous les 6 mois ou chaque fois que la réécriture du mot de passe est désactivée puis réactivée sur Azure AD Connect, nous renouvelons automatiquement toutes les clés afin d’assurer la sécurité et la protection maximales du service.

### <a name="password-writeback-bandwidth-usage"></a>Utilisation de la bande passante de la réécriture du mot de passe

L’écriture différée des mots de passe est un service à bande passante faible qui renvoie les demandes à l’agent local, uniquement dans les circonstances suivantes :

* Deux messages sont envoyés lors de l’activation ou de la désactivation de la fonctionnalité via Azure AD Connect.
* Un message est envoyé une fois toutes les cinq minutes comme une pulsation du service tant que le service est en cours d’exécution.
* Deux messages sont envoyés à chaque fois qu’un nouveau mot de passe est proposé :
   * Le premier message est une requête visant à effectuer l’opération.
   * Le deuxième message contient le résultat de l’opération et est envoyé dans les circonstances suivantes :
      * Chaque fois qu’un nouveau mot de passe est soumis pendant une réinitialisation de mot de passe libre-service par l’utilisateur.
      * Chaque fois qu’un nouveau mot de passe est soumis pendant une opération de modification de mot de passe par l’utilisateur.
      * Chaque fois qu’un nouveau mot de passe est soumis au cours d’une réinitialisation des mots de passe des utilisateurs réalisée par un administrateur (depuis les portails d’administration Azure uniquement).

#### <a name="message-size-and-bandwidth-considerations"></a>Considérations relatives à la taille des messages et à la bande passante

La taille de chaque message décrit précédemment est généralement inférieure à 1 Ko. Même sous des charges extrêmes, le service d’écriture différée des mots de passe lui-même utilise quelques kilobits par seconde de bande passante. Étant donné que chaque message est envoyé en temps réel, uniquement lorsqu’une opération de mise à jour du mot de passe le requiert, et étant donné que la taille du message est très limitée, l’utilisation de la bande passante de la fonctionnalité d’écriture différée est effectivement trop petite pour produire un impact mesurable.

## <a name="supported-writeback-operations"></a>Opérations de réécriture prises en charge

Les mots de passe sont réécrits dans les situations suivantes :

* **Opérations de l’utilisateur final prises en charge**
   * Tout changement volontaire de mot de passe en libre-service de l’utilisateur final
   * Tout changement forcé de mot de passe en libre-service de l’utilisateur final (par exemple, expiration du mot de passe)
   * Toute réinitialisation de mot de passe en libre-service de l’utilisateur final émanant du [portail de réinitialisation de mot de passe](https://passwordreset.microsoftonline.com)

* **Opérations de l’administrateur prises en charge**
   * Tout changement volontaire de mot de passe en libre-service de l’administrateur
   * Tout changement forcé de mot de passe en libre-service de l’administrateur (par exemple, expiration du mot de passe)
   * Toute réinitialisation de mot de passe en libre-service de l’administrateur émanant du [portail de réinitialisation de mot de passe](https://passwordreset.microsoftonline.com)
   * Toute réinitialisation de mot de passe de l’utilisateur final lancée par l’administrateur depuis le [portail Azure](https://portal.azure.com)
   * Toute réinitialisation de mot de passe de l’utilisateur final lancée par l’administrateur depuis l’[API Microsoft Graph bêta](/graph/api/passwordauthenticationmethod-resetpassword?tabs=http&view=graph-rest-beta).

## <a name="unsupported-writeback-operations"></a>Opérations de réécriture non prises en charge

Les mots de passe ne sont pas réécrits dans les situations suivantes :

* **Opérations de l’utilisateur final non prises en charge**
   * Toute réinitialisation de mot de passe de l’utilisateur final lancée par ce dernier à l’aide de PowerShell (version 1 ou version 2) ou l’API Microsoft Graph
* **Opérations de l’administrateur non prises en charge**
   * Toute réinitialisation de mot de passe de l’utilisateur final lancée par l’administrateur à l’aide de PowerShell (version 1 ou version 2) ou l’API Microsoft Graph (l’[API Microsoft Graph bêta](/graph/api/passwordauthenticationmethod-resetpassword?tabs=http&view=graph-rest-beta) est prise en charge).
   * Toute réinitialisation de mot de passe de l’utilisateur final lancée par l’administrateur depuis le [Centre d’administration Microsoft 365](https://admin.microsoft.com)
   * N’importe quel administrateur ne peut pas utiliser l’outil de réinitialisation de mot de passe en vue de réinitialiser son propre mot de passe pour la réécriture du mot de passe.

> [!WARNING]
> L’utilisation de la case à cocher « L’utilisateur doit changer de mot de passe à la prochaine ouverture de session » dans les outils d’administration AD DS locaux, par exemple Utilisateurs et ordinateurs Active Directory ou le Centre d’administration Active Directory, est prise en charge en tant que fonctionnalité en préversion d’Azure AD Connect. Pour plus d’informations, consultez [Implémenter la synchronisation de hachage du mot de passe avec la synchronisation Azure AD Connect](../hybrid/how-to-connect-password-hash-synchronization.md).

## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer avec la réécriture SSPR, suivez le tutoriel suivant :

> [!div class="nextstepaction"]
> [Tutoriel : Activer la réécriture SSPR (réinitialisation de mot de passe en libre-service)](./tutorial-enable-sspr-writeback.md)