---
title: 'Azure AD Connect : Résolution des problèmes d’ancre source pendant l’installation | Microsoft Docs'
description: Cette rubrique explique comment résoudre les problèmes d’ancre source pendant l’installation.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: troubleshooting
ms.date: 04/19/2019
ms.subservice: hybrid
ms.author: billmath
ms.openlocfilehash: 4e6460015430850c11fbf70a005d7440ce1b92fe
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91306002"
---
# <a name="troubleshooting-source-anchor-issues-during-installation"></a>Résolution des problèmes d’ancre source pendant l’installation
Cet article explique les différents problèmes d’ancre source qui peuvent se produire pendant l’installation et des moyens de résoudre ces problèmes.

## <a name="invalid-source-anchor-in-azure-active-directory"></a>Ancre source non valide dans Azure Active Directory

### <a name="custom-installation"></a>Installation personnalisée

Pendant l’installation personnalisée, Azure AD Connect lit la stratégie d’ancre source dans Azure Active Directory. Si la stratégie existe dans Azure Active Directory, Azure AD Connect applique la même stratégie, sauf si elle est remplacée par le client. L’Assistant vous indique quel attribut a été lu. En outre, l’Assistant vous avertit si vous essayez de remplacer la stratégie d’ancre source.

Au cours de cette opération de lecture, il est possible que la stratégie d’ancre source dans Azure Active Directory soit inattendue. Dans ce cas, Azure AD Connect ne sait pas quel ancre source utiliser et un remplacement manuel est nécessaire.</br>
![Capture d’écran montrant où remplacer manuellement l’ancre source.](media/tshoot-connect-source-anchor/source1.png)

Pour résoudre ce problème, vous pouvez remplacer manuellement l’ancre source en sélectionnant un attribut spécifique. N’utilisez cette option que si vous êtes certain de l’attribut à sélectionner. Si vous n’en êtes pas certain, contactez le [support Microsoft](https://support.microsoft.com/contactus/) pour obtenir des conseils. Si vous modifiez la stratégie d’ancre source, l’association entre vos utilisateurs locaux et leurs ressources Azure associées peut être rompue.</br>
![Capture d’écran montrant l’attribut spécifié qui remplace l’ancre source.](media/tshoot-connect-source-anchor/source2.png)

### <a name="express-installation"></a>Installation rapide
Pendant l’installation express, Azure AD Connect lit la stratégie d’ancre source dans Azure Active Directory. Si la stratégie existe dans Azure Active Directory, Azure AD Connect applique la même stratégie. Il n’y a aucune option pour effectuer le remplacement manuel.

Au cours de cette opération de lecture, il est possible que la stratégie d’ancre source dans Azure Active Directory soit inattendue. Dans ce cas, Azure AD Connect ne sait pas quelle est l’ancre source.</br>
![Capture d’écran montrant ce qui se passe quand l’ancre source dans Azure Active Directory est inattendue.](media/tshoot-connect-source-anchor/source3.png)

Pour résoudre ce problème, vous devez effectuer une réinstallation en mode personnalisé et remplacer manuellement l’ancre source en sélectionnant un attribut spécifique. N’utilisez cette option que si vous êtes certain de l’attribut à sélectionner. Si vous n’en êtes pas certain, contactez le [support Microsoft](https://support.microsoft.com/contactus/) pour obtenir des conseils. Si vous modifiez la stratégie d’ancre source, l’association entre vos utilisateurs locaux et leurs ressources Azure associées peut être rompue.

### <a name="invalid-source-anchor-in-sync-engine"></a>Ancre source non valide dans le moteur de synchronisation
Pendant l’installation, il est possible qu’Azure AD Connect tente de configurer le moteur de synchronisation à l’aide d’une ancre source non valide. Cette opération est très probablement liée à un problème de produit et l’installation d’Azure AD Connect échoue. Contactez le [support Microsoft](https://support.microsoft.com/contactus/) si vous être confronté à ce problème.</br>
![inattendu](media/tshoot-connect-source-anchor/source4.png)


## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
