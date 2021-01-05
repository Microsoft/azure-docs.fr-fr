---
title: Simuler la détection des risques dans Azure AD Identity Protection
description: Découvrez comment simuler la détection des risques dans Identity Protection.
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: how-to
ms.date: 06/05/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7eafeaf59757fcda978fa89b4bc2f9882b769e48
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94835898"
---
# <a name="simulating-risk-detections-in-identity-protection"></a>Simuler la détection des risques dans Identity Protection

Les administrateurs peuvent chercher à simuler les risques que présente leur environnement afin d’accomplir les tâches suivantes :

- remplir les données de l’environnement Identity Protection en simulant des détections de risques et des vulnérabilités ;
- définir des stratégies d’accès conditionnel en fonction des risques et tester l’impact de ces stratégies.

Cette section donne les étapes à suivre pour simuler les types de détection des risques suivants :

- adresse IP anonyme (facile) ;
- propriétés de connexion inhabituelles (modéré) ;
- voyage atypique (difficile).

Les autres détections d’événements à risque ne peuvent pas être simulées de manière sécurisée.

Pour plus d’informations sur chaque détection de risque, voir l’article [Qu’est-ce que le risque](concept-identity-protection-risks.md).

## <a name="anonymous-ip-address"></a>Adresse IP anonyme

L’exécution de la procédure ci-après requiert l’utilisation des éléments suivants :

- [Navigateur Tor](https://www.torproject.org/projects/torbrowser.html.en) afin de simuler des adresses IP anonymes. Vous devrez peut-être utiliser une machine virtuelle si votre organisation restreint l’utilisation du navigateur Tor.
- Compte de test non encore inscrit à Azure AD Multi-Factor Authentication.

**Pour simuler une connexion depuis une adresse IP anonyme, procédez comme suit**:

1. Avec le [navigateur Tor](https://www.torproject.org/projects/torbrowser.html.en), accédez à [https://myapps.microsoft.com](https://myapps.microsoft.com).   
2. Entrez les informations d’identification du compte que vous souhaitez voir apparaître dans le rapport **Connexions depuis des adresses IP anonymes** .

La connexion s’affiche dans le tableau de bord d’Identity Protection dans un délai de 10 à 15 minutes. 

## <a name="unfamiliar-sign-in-properties"></a>Propriétés de connexion inhabituelles

Pour simuler des emplacements non connus, vous devez vous connecter à partir d’un emplacement et d’un appareil depuis lesquels votre compte de test ne s’est jamais connecté.

La procédure ci-après utilise les éléments nouvellement créés suivants :

- connexion VPN pour simuler le nouvel emplacement ;
- machine virtuelle pour simuler un nouvel appareil.

L’exécution de la procédure ci-après requiert l’utilisation d’un compte d’utilisateur présentant :

- un historique de connexions d’au moins 30 jours ;
- Fonctionnalité Azure AD Multi-Factor Authentication activée.

**Pour simuler une connexion depuis un emplacement non connu, procédez comme suit**:

1. Lorsque vous vous connectez avec votre compte de test, ne vous soumettez pas à la demande d’authentification multifacteur (MFA) pour y échouer.
2. En utilisant votre nouveau VPN, accédez à [https://myapps.microsoft.com](https://myapps.microsoft.com) et entrez les informations d’identification de votre compte de test.

La connexion s’affiche dans le tableau de bord d’Identity Protection dans un délai de 10 à 15 minutes.

## <a name="atypical-travel"></a>Voyage inhabituel

Il est difficile de simuler la condition de voyage atypique, car l’algorithme utilise le Machine Learning pour éliminer les faux positifs, par exemple le voyage atypique sur des appareils connus ou les connexions depuis des VPN utilisés par d’autres utilisateurs du répertoire. De plus, l’algorithme nécessite un historique de connexions de 14 jours et 10 connexions de l’utilisateur avant de commencer à générer des détections d’événements à risque. En raison des modèles Machine Learning complexes et des règles ci-dessus, les étapes ci-après n’entraîneront probablement pas de détections d’événements à risque. Il peut être judicieux de répliquer ces étapes pour plusieurs comptes Azure AD afin de simuler cette détection.

**Pour simuler une détection des risques de voyage atypique, suivez les étapes ci-dessous** :

1. Avec votre navigateur standard, accédez à [https://myapps.microsoft.com](https://myapps.microsoft.com).  
2. Entrez les informations d’identification du compte pour lequel vous voulez générer une détection des risques de voyage atypique.
3. Changez votre agent utilisateur. Vous pouvez modifier l’agent utilisateur dans Microsoft Edge, dans Outils de développement (F12).
4. Changez votre adresse IP. Vous pouvez changer votre adresse IP en utilisant un VPN, un module complémentaire Tor ou en créant une machine au sein d’Azure dans un autre centre de données.
5. Connectez-vous à [https://myapps.microsoft.com](https://myapps.microsoft.com) à l’aide des mêmes informations d’identification que précédemment et quelques minutes seulement après la connexion précédente.

La connexion s’affiche dans le tableau de bord d’Identity Protection dans un délai de 2 à 4 heures.

## <a name="testing-risk-policies"></a>Tester des stratégies de risque

Cette section décrit les étapes à suivre pour tester les stratégies d’utilisateur de connexion à risque créées dans l’article [Guide pratique : Configurer et activer des stratégies de risque](howto-identity-protection-configure-risk-policies.md).

### <a name="user-risk-policy"></a>Stratégie de risque d’utilisateur

Pour tester une stratégie de sécurité d’utilisateur à risque, suivez les étapes ci-dessous :

1. Accédez au [portail Azure](https://portal.azure.com).
1. Accédez à **Azure Active Directory** > **Sécurité** > **Vue d’ensemble**.
1. Sélectionnez **Configurer la stratégie d’utilisateur à risque**.
   1. Sous **Affectations**
      1. **Utilisateurs** : choisissez **Tous les utilisateurs** ou **Sélectionner des personnes et des groupes** si vous limitez votre lancement.
         1. Si vous le souhaitez, vous pouvez exclure des utilisateurs de la stratégie.
      1. **Conditions** - **Risque utilisateur** : la recommandation de Microsoft consiste à définir cette option sur **Élevé**.
   1. Sous **Contrôles**
      1. **Accès** : la recommandation de Microsoft consiste à **Autoriser l’accès** et à **Exiger la modification du mot de passe**.
   1. **Appliquer la stratégie** - **Désactivé**.
   1. **Enregistrer** : cette action a pour effet de renvoyer à la page **Vue d’ensemble**.
1. Élevez le risque utilisateur d’un compte de test, par exemple, en simulant à plusieurs reprises l’une des détections d’événements à risque.
1. Patientez quelques minutes, puis vérifiez que le niveau de risque de votre utilisateur a augmenté. Dans le cas contraire, simulez d’autres détections d’événements à risque pour l’utilisateur.
1. Revenez à votre stratégie de risque, définissez **Appliquer la stratégie** sur **Activé** et cliquez sur **Enregistrer** pour enregistrer votre modification de stratégie.
1. Vous pouvez désormais tester l’accès conditionnel en fonction des risques utilisateur en vous connectant à l’aide d’un compte d’utilisateur présentant un niveau de risque élevé.

### <a name="sign-in-risk-security-policy"></a>Stratégie de sécurité en matière de risque à la connexion

Pour tester une stratégie de connexion à risque, suivez les étapes ci-dessous :

1. Accédez au [portail Azure](https://portal.azure.com).
1. Accédez à **Azure Active Directory** > **Sécurité** > **Vue d’ensemble**.
1. Sélectionnez **Configurer la stratégie de connexion à risque**.
   1. Sous **Affectations**
      1. **Utilisateurs** : choisissez **Tous les utilisateurs** ou **Sélectionner des personnes et des groupes** si vous limitez votre lancement.
         1. Si vous le souhaitez, vous pouvez exclure des utilisateurs de la stratégie.
      1. **Conditions** - **Risque connexion** : la recommandation de Microsoft consiste à définir cette option sur **Moyen ou plus**.
   1. Sous **Contrôles**
      1. **Accès** : la recommandation de Microsoft consiste à **Autoriser l’accès** et à **Exiger l’authentification multifacteur**.
   1. **Appliquer la stratégie** - **Activé**
   1. **Enregistrer** : cette action a pour effet de renvoyer à la page **Vue d’ensemble**.
1. Vous pouvez désormais tester l’accès conditionnel en fonction des risques à la connexion en vous connectant à l’aide d’une session à risque (par exemple, au moyen du navigateur Tor). 

## <a name="next-steps"></a>Étapes suivantes

- [Qu’est-ce que le risque ?](concept-identity-protection-risks.md)

- [Guide pratique pour Configurer et activer des stratégies de risque](howto-identity-protection-configure-risk-policies.md)

- [Azure Active Directory Identity Protection](overview-identity-protection.md)
