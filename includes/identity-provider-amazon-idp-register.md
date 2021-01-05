---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 05/04/2020
ms.author: mimart
ms.openlocfilehash: de1b67265c5c3b3f7247b7f21506eed88abfd550
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82900358"
---
1. Connectez-vous à la [console de développement Amazon](https://developer.amazon.com/dashboard) avec les informations d’identification de votre compte Amazon.
1. Si ce n’est déjà fait, cliquez sur **Sign Up**, suivez la procédure d’inscription pour développeur et acceptez la politique d’utilisation.
1. Dans le tableau de bord, sélectionnez **connexion avec Amazon**.
1. Sélectionnez **Create a New Security Profile** (Créer un profil de sécurité).
1. Entrez un **nom de profil de sécurité**, une **description du profil de sécurité** et l’**URL de l’avis de confidentialité et de consentement**, par exemple `https://www.contoso.com/privacy` L’URL de l’avis de confidentialité est une page gérée qui fournit aux utilisateurs des informations sur la confidentialité. Ensuite, cliquez sur **Enregistrer**.
1. Dans la section **connexion avec les configurations Amazon**, sélectionnez le **nom du profil de sécurité** que vous avez créé, cliquez sur l’icône **Gérer** et sélectionnez les **paramètres web**.
1. Dans la section **Paramètres web**, copiez les valeurs **ID client**. Sélectionnez **Montrer le secret** pour obtenir la clé secrète client et copiez-la. Vous aurez besoin de ces deux valeurs pour configurer un compte Amazon en tant que fournisseur d’identité dans votre locataire. **Client Secret** est une information d’identification de sécurité importante.
1. Dans les **paramètres web**, sélectionnez **Modifier**. Dans **Origines autorisées** et **URL de retour autorisées**, entrez les URL appropriées (mentionnées ci-dessus). 
1. Cliquez sur **Enregistrer**.
