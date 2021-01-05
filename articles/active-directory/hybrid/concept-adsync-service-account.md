---
title: 'Azure AD Connect : Compte de service ADSync | Microsoft Docs'
description: Cette rubrique décrit le compte de service ADSync et les meilleures pratiques qui s'y rapportent.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/27/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8dddfb8426b769c06cb5b7494431b7eee34dbf9e
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94410893"
---
# <a name="adsync-service-account"></a>Compte de service ADSync
Azure AD Connect installe un service local qui orchestre la synchronisation entre Active Directory et Azure Active Directory.  Le service de synchronisation Microsoft Azure AD Sync (ADSync) s’exécute sur un serveur de votre environnement local.  Les informations d’identification du service sont définies par défaut dans les installations Express, mais peuvent être personnalisées pour répondre aux exigences de sécurité de votre organisation.  Ces informations d’identification ne sont pas utilisées pour se connecter à vos forêts locales ou à Azure Active Directory.

En termes de planification, il est important de choisir le compte le service ADSync avant d'installer Azure AD Connect.  Toute tentative de modification des informations d'identification après l'installation entraînera un échec de démarrage du service, une perte d'accès à la base de synchronisation et un échec d'authentification avec vos répertoires connectés (Azure et AD DS).  Aucune synchronisation ne sera possible avant la restauration des informations d'identification d'origine.

## <a name="the-default-adsync-service-account"></a>Compte de service ADSync par défaut

Lorsqu'il est exécuté sur un serveur membre, le service AdSync s’exécute dans le contexte d’un compte de service virtuel (VSA).  En raison d'une limitation du produit, un compte de service personnalisé est créé lors d'une l'installation sur un contrôleur de domaine.  Si le compte de service des paramètres Express ne répond pas aux exigences de sécurité de votre organisation, déployez Azure AD Connect en sélectionnant l'option Personnaliser.  Puis, choisissez l’option de compte de service qui répond aux besoins de votre organisation.

>[!NOTE]
>Installé sur un contrôleur de domaine, le compte de service par défaut se présente comme suit : Domain\AAD_InstallationIdentifier.  Le mot de passe de ce compte est généré de façon aléatoire, ce qui peut être complexe en termes de récupération et rotation du mot de passe.  Microsoft recommande de personnaliser le compte de service lors de l'installation initiale sur un contrôleur de domaine pour utiliser un compte de service administré autonome ou en groupe (sMSA/gMSA)

|Emplacement d'Azure AD Connect|Compte de service créé|
|-----|-----|
|Serveur membre|NT SERVICE\ADSync|
|Contrôleur de domaine|Domain\AAD_74dc30c01e80 (voir la remarque)|

## <a name="custom-adsync-service-accounts"></a>Comptes de service ADSync personnalisés
Microsoft recommande l'exécution du service ADSync dans le contexte d'un compte de service virtuel ou d'un compte de service administré autonome ou en groupe.  Votre administrateur de domaine peut également choisir de créer un compte de service configuré pour répondre aux exigences de sécurité spécifiques de votre organisation.   Pour personnaliser le compte de service utilisé lors de l’installation, sélectionnez l’option Personnaliser sur la page Paramètres Express ci-dessous.   Les options suivantes sont disponibles :

- compte par défaut – Azure AD Connect configure le compte de service comme décrit ci-dessus
- compte de service administré - utilisez un compte de service administré autonome ou en groupe configuré par votre administrateur
- compte de domaine - utilisez un compte de service de domaine configuré par votre administrateur

![Capture d’écran de la page Azure AD Connect - Configuration rapide, avec les cases d’option « Personnaliser » ou « Utiliser la configuration rapide ».](media/concept-adsync-service-account/adsync1.png)

![Capture d’écran de la page Azure AD Connect « Installer les composants nécessaires » avec l’option permettant d’utiliser un compte de service administré existant sélectionné.](media/concept-adsync-service-account/adsync2.png)

## <a name="diagnosing-adsync-service-account-changes"></a>Diagnostic des modifications du compte de service ADSync
Modifier les informations d'identification du service ADSync après l'installation entraînera un échec de démarrage du service, une perte d'accès à la base de synchronisation et un échec d'authentification avec vos répertoires connectés (Azure et AD DS).  L'octroi d'un accès de base de données au nouveau compte de service ADSync ne suffit pas à résoudre ce problème. Aucune synchronisation ne sera possible avant la restauration des informations d'identification d'origine.

S'il ne parvient pas à démarrer, le service ADSync génère un message dans le journal des événements.  Le contenu de ce message varie selon l'utilisation de la base de données intégrée (localdb) ou de SQL complet.  Vous trouverez ci-dessous quelques exemples d’entrées du journal des événements.

### <a name="example-1"></a>Exemple 1

Les clés de chiffrement du service AdSync sont introuvables et ont été recréées.  La synchronisation n'interviendra pas tant que ce problème n'aura pas été résolu.

Lors de la résolution de ce problème, les clés de chiffrement Microsoft Azure AD Sync seront inaccessibles en cas de modification des informations d'identification de connexion au service AdSync.  Si les informations d’identification ont été modifiées, utilisez l’application Services pour rétablir la valeur d'origine du compte d'ouverture de session (par exemple, NT SERVICE\AdSync) et redémarrez le service.  Cela aura pour effet de rétablir le bon fonctionnement du service AdSync.

Pour plus d’informations, consultez l'[article](./whatis-hybrid-identity.md) suivant.

### <a name="example-2"></a>Exemple 2

Le service n’a pas pu démarrer en raison de l'impossibilité d'établir une connexion à la base de données locale (localdb).

Lors de la résolution de ce problème, le service Microsoft Azure AD Sync ne sera plus autorisé à accéder au fournisseur de base de données locale en cas de modification des informations d'identification de connexion au service AdSync.  Si les informations d’identification ont été modifiées, utilisez l’application Services pour rétablir la valeur d'origine du compte d'ouverture de session (par exemple, NT SERVICE\AdSync) et redémarrez le service.  Cela aura pour effet de rétablir le bon fonctionnement du service AdSync.

Pour plus d’informations, consultez l'[article](./whatis-hybrid-identity.md) suivant.

Autres détails Les informations d'erreur suivantes ont été renvoyées par le fournisseur :
 

``` 
OriginalError=0x80004005 OLEDB Provider error(s): 
Description  = 'Login timeout expired'
Failure Code = 0x80004005
Minor Number = 0 
Description  = 'A network-related or instance-specific error has occurred while establishing a connection to SQL Server. Server is not found or not accessible. Check if instance name is correct and if SQL Server is configured to allow remote connections. For more information see SQL Server Books Online.'
```
## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).