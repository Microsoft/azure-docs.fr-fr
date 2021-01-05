---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 02/27/2020
ms.author: mimart
ms.openlocfilehash: f4944881e6eff3e9a56a74bc997c280f9fff6be8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78186989"
---
## <a name="ropc-flow-notes"></a>Notes relatives au flux ROPC
Les options suivantes sont prises en charge dans Azure Active Directory B2C (Azure AD B2C) :

- **Client natif** : une interaction de l’utilisateur lors de l’authentification se produit quand le code s’exécute sur un appareil côté utilisateur. L’appareil peut être une application mobile qui s’exécute dans un système d’exploitation natif, comme Android et iOS.
- **Flux client public** : seules les informations d’identification de l’utilisateur, collectées par une application, sont envoyées dans l’appel d’API. Les informations d’identification de l’application ne sont pas envoyées.
- **Ajouter de nouvelles revendications** : il est possible de changer le contenu du jeton de l’ID pour ajouter de nouvelles revendications.

Les flux suivants ne sont pas pris en charge :

- **Serveur à serveur** : le système de protection des identités a besoin d’une adresse IP fiable collectée auprès de l’appelant (le client natif) dans le cadre de l’interaction. Dans un appel d’API côté serveur, seule l’adresse IP du serveur est utilisée. Si un seuil dynamique d’échecs d’authentification est dépassé, le système de protection d’identité peut identifier une adresse IP répétée en tant qu’attaquant.
- **Flux client confidentiel** : l’ID du client d’application est validé, mais le secret d’application ne l’est pas.

Lors de l'utilisation du flux ROPC, tenez compte de ce qui suit :

- ROPC ne fonctionne pas en cas d'interruption du flux d'authentification nécessitant une interaction de l'utilisateur. Par exemple, lorsqu'un mot de passe a expiré ou doit être modifié et qu'une authentification multifacteur est nécessaire, ou lorsque des informations supplémentaires doivent être collectées au moment de la connexion (comme le consentement de l'utilisateur).
- Seuls les comptes locaux sont pris en charge par ROPC. Les utilisateurs ne peuvent pas se connecter à des fournisseurs d'identité fédérés comme Microsoft, Google+, Twitter, AD-FS ou Facebook.
- La gestion des sessions, y compris le maintien de la connexion, n'est pas applicable.
