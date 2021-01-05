---
title: Protection de mots de passe Azure AD – Azure Active Directory
description: Interdisez les mots de passe faibles dans les environnements Active Directory Domain Services en utilisant la fonctionnalité Protection de mots de passe Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/17/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8f5df1cb158821fb0cd85d90f9ba3b79d80adf45
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743919"
---
# <a name="enforce-on-premises-azure-ad-password-protection-for-active-directory-domain-services"></a>Appliquer la fonctionnalité Protection de mots de passe Azure AD localement pour Active Directory Domain Services

Protection de mots de passe d’Azure AD détecte et bloque les mots de passe faibles connus et leurs variantes, mais peut aussi bloquer d’autres termes faibles propres à votre organisation. Le déploiement local de Protection de mots de passe Azure AD utilise les mêmes listes globales et personnalisées de mots de passe interdits qui sont stockées dans Azure AD et effectue les mêmes vérifications des modifications de mot de passe localement que celles d’Azure AD pour les modifications informatiques. Ces vérifications sont effectuées lors des modifications de mot de passe et des événements de réinitialisation de mot de passe par les contrôleurs de domaine Active Directory Domain Services (AD DS) locaux.

## <a name="design-principles"></a>Principes de conception

Protection de mots de passe Azure AD est conçue dans le respect des principes suivants :

* Les contrôleurs de domaine ne doivent jamais communiquer directement avec Internet.
* Aucun nouveau port réseau n’est ouvert sur les contrôleurs de domaine.
* Aucune modification du schéma AD DS n’est requise. Le logiciel utilise le *conteneur* AD DS et les objets de schéma *serviceConnectionPoint* existants.
* Aucun niveau fonctionnel minimal de domaine ou de forêt (DFL\FFL) AD DS n’est requis.
* Le logiciel ne crée pas de compte ni n’en exige dans les domaines AD DS qu’il protège.
* Les mots de passe utilisateur en texte clair ne quittent jamais le contrôleur de domaine, que ce soit pendant les opérations de validation de mot de passe ou à tout autre moment.
* Le logiciel n’est pas dépendant d’autres fonctionnalités d’Azure AD. Par exemple, la synchronisation du hachage du mot de passe (PHS) d’Azure AD n’est pas liée à Protection de mots de passe Azure AD ni requise pour son fonctionnement.
* Le déploiement incrémentiel est pris en charge, sous réserve que la stratégie de mot de passe soit appliquée uniquement à l’endroit où l’agent du contrôleur de domaine est installé.

## <a name="incremental-deployment"></a>Déploiement incrémentiel

Protection de mots de passe Azure AD prend en charge le déploiement incrémentiel sur les différents contrôleurs de domaine dans un domaine AD DS. Il est important de comprendre ce que cela signifie réellement et quels sont les compromis.

Le logiciel de l’agent du contrôleur de domaine pour Protection de mots de passe Azure AD ne peut valider les mots de passe que s’il est installé sur un contrôleur de domaine et uniquement pour des changements de mot de passe envoyés à ce contrôleur de domaine. Il n’est pas possible de contrôler quels contrôleurs de domaine sont choisis par les ordinateurs clients Windows pour le traitement des changements de mot de passe utilisateur. Afin de garantir un comportement cohérent et l’application universelle de la sécurité de Protection de mots de passe Azure AD, le logiciel de l’agent du contrôleur de domaine doit être installé sur tous les contrôleurs de domaine au sein d’un domaine.

De nombreuses organisations souhaitent tester Protection de mots de passe Azure AD rigoureusement sur un sous-ensemble de leurs contrôleurs de domaine avant de procéder à un déploiement complet. Pour prendre en charge ce scénario, Protection de mots de passe Azure AD prend en charge le déploiement partiel. Le logiciel de l’agent du contrôleur de domaine sur un contrôleur de domaine donné valide activement les mots de passe, même si ce logiciel n’est pas installé sur les autres contrôleurs de domaine du domaine en question. Des déploiements partiels de ce type ne sont pas sécurisés, et ils ne sont donc pas recommandés à des fins autres que de test.

## <a name="architectural-diagram"></a>Diagramme architectural

Avant de déployer Protection de mots de passe Azure AD dans un environnement AD DS local, il est important d’en comprendre la conception et les concepts fonctionnels sous-jacents. Le schéma suivant montre comment les composants de la protection par mot de passe Azure AD opèrent ensemble :

![Comment les composants de protection par mot de passe Azure AD fonctionnent ensemble](./media/concept-password-ban-bad-on-premises/azure-ad-password-protection.png)

* Le service proxy de Protection de mots de passe Azure AD s’exécute sur toute machine jointe au domaine dans la forêt AD DS actuelle. L’objectif principal du service est de transférer les requêtes de téléchargement de stratégie de mot de passe des contrôleurs de domaine à Azure AD, puis de renvoyer les réponses d’Azure AD au contrôleur de domaine.
* La DLL de filtre de mot de passe de l’agent du contrôleur de domaine reçoit les demandes de validation de mot de passe utilisateur du système d’exploitation. Le filtre les transmet au service de l’agent du contrôleur de domaine qui s’exécute localement sur le contrôleur de domaine.
* Le service de l’agent du contrôleur de domaine de Protection de mots de passe Azure AD reçoit les demandes de validation de mot de passe en provenance de la DLL de filtre de mot de passe de l’agent du contrôleur de domaine. Le service de l’agent du contrôleur de domaine les traite en appliquant la stratégie de mot de passe en vigueur (disponible localement) et retourne le résultat *réussite* ou *échec*.

## <a name="how-azure-ad-password-protection-works"></a>Fonctionnement de Protection de mots de passe Azure AD

Les composants locaux de Protection de mots de passe Azure AD fonctionnent comme suit :

1. Chaque instance du service proxy de Protection de mots de passe Azure AD se signale aux contrôleurs de domaine de la forêt en créant un objet *serviceConnectionPoint* dans Active Directory.

    Chaque service de l’agent du contrôleur de domaine pour Protection de mots de passe Azure AD crée également un objet *serviceConnectionPoint* dans Active Directory. Cet objet est principalement utilisé pour la création de rapports et les diagnostics.

1. Le service de l’agent du contrôleur de domaine est chargé d’initier le téléchargement d’une nouvelle stratégie de mot de passe à partir d’Azure AD. La première étape consiste à localiser un service proxy de protection par mot de passe Azure AD en recherchant dans la forêt des objets *serviceConnectionPoint* proxy.

1. Quand un service proxy disponible est trouvé, l’agent du contrôleur de domaine envoie une demande de téléchargement de stratégie de mot de passe à ce service proxy. Le service proxy envoie à son tour la demande à Azure AD, puis renvoie la réponse au service de l’agent du contrôleur de domaine.

1. Après réception d’une nouvelle stratégie de mot de passe d’Azure AD, le service de l’agent du contrôleur de domaine stocke la stratégie dans un dossier dédié à la racine du partage de dossiers *sysvol* de son domaine. Le service de l’agent du contrôleur de domaine supervise également ce dossier, au cas où des stratégies plus récentes s’y répliqueraient à partir d’autres services de l’agent du contrôleur de domaine dans le domaine.

1. Le service de l’agent du contrôleur de domaine demande toujours une nouvelle stratégie au démarrage. Après démarrage, le service de l’agent du contrôleur de domaine vérifie toutes les heures l’âge de la stratégie en vigueur disponible localement. Si la stratégie a plus d’une heure, l’agent du contrôleur de domaine demande à Azure AD une nouvelle stratégie via le service proxy, comme décrit précédemment. Si la stratégie actuelle n’a pas plus d’une heure, l’agent du contrôleur de domaine continue de l’utiliser.

1. Lorsque des événements de modification de mot de passe sont reçus par un contrôleur de domaine, la stratégie mise en cache est utilisée pour déterminer si le nouveau mot de passe est accepté ou rejeté.

### <a name="key-considerations-and-features"></a>Considérations et fonctionnalités principales

* Chaque fois qu’une stratégie de mot de passe Protection de mots de passe Azure AD est téléchargée, elle est spécifique à un locataire. En d’autres termes, les stratégies de mot de passe sont toujours une combinaison de la liste globale des mots de passe interdits de Microsoft et de la liste des mot de passe interdits personnalisée par le client.
* L’agent du contrôleur de domaine communique avec le service proxy via RPC sur TCP. Le service proxy écoute ces appels sur un port RPC statique ou dynamique en fonction de la configuration.
* L’agent du contrôleur de domaine n’écoute jamais un port réseau disponible.
* Le service proxy n’appelle jamais le service de l’agent du contrôleur de domaine.
* Le service proxy est sans état. Il ne met jamais en cache les stratégies ou tout autre état téléchargé à partir d’Azure.
* Le service de l’agent du contrôleur de domaine utilise toujours la stratégie de mot de passe la plus récente disponible localement pour évaluer le mot de passe d’un utilisateur. Si aucune stratégie de mot de passe n’est disponible sur le contrôleur de domaine local, le mot de passe est accepté automatiquement. Lorsque cela se produit, un message d’événement est journalisé pour avertir l’administrateur.
* Protection de mots de passe Azure AD n’est pas un moteur d’application de stratégie en temps réel. Il peut exister un décalage entre le moment où la configuration de la stratégie de mot de passe est modifiée dans Azure AD et celui où cette modification atteint tous les contrôleurs de domaine et y est appliquée.
* Protection de mots de passe Azure AD complète les stratégies de mot de passe AD DS existantes. Elle ne les remplace pas. Cela englobe toutes les autres DLL de filtre de mot de passe tierces qui peuvent être installées. Avant d’accepter un mot de passe, AD DS exige toujours que tous les composants de validation de mot de passe marquent leur accord.

## <a name="forest--tenant-binding-for-azure-ad-password-protection"></a>Liaison forêt\locataire de Protection de mots de passe Azure AD

Le déploiement de Protection de mots de passe Azure AD dans une forêt AD DS nécessite l’inscription de cette forêt auprès d’Azure AD. Chaque service proxy déployé doit également être inscrit auprès d’Azure AD. Ces inscriptions de forêt et de proxy sont associées avec un locataire Azure AD spécifique, qui est implicitement identifié par les informations d’identification utilisées lors de l’inscription.

La forêt AD DS et tous les services proxy déployés au sein d’une forêt doivent être inscrits auprès du même locataire. Il n’est pas possible d’avoir une forêt AD DS ou des services proxy au sein de cette forêt inscrits auprès d’autres locataires Azure AD. La configuration erronée d’un déploiement se traduit notamment par l’impossibilité de télécharger des stratégies de mot de passe.

> [!NOTE]
> Les clients qui disposent de plusieurs locataires Azure AD doivent donc choisir un locataire unique pour inscrire chaque forêt à des fins de protection des mots de passe Azure AD.

## <a name="download"></a>Téléchargement

Les deux programmes d’installation d’agent requis pour Protection de mots de passe Azure AD sont disponibles dans le[Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=57071).

## <a name="next-steps"></a>Étapes suivantes

Pour vous familiariser avec l’utilisation locale de Protection de mots de passe Azure AD, procédez comme suit :

> [!div class="nextstepaction"]
> [Déployer Protection de mots de passe Azure AD localement](howto-password-ban-bad-on-premises-deploy.md)
