---
title: Résoudre une erreur de connexion à Azure Dev Tools for Teaching
description: Cet article décrit les actions que doit effectuer un étudiant s’il reçoit un message d’erreur lors de la connexion à Azure Dev Tools for Teaching.
author: rymend
ms.author: rymend
ms.topic: troubleshooting
ms.service: azure-education
ms.subservice: education-hub
ms.date: 06/30/2020
ms.openlocfilehash: 276ab2cc1d79235c00ce1bb50ff9b648bb1de6ea
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094088"
---
# <a name="troubleshooting-student-login-issues"></a>Résolution des problèmes de connexion des étudiants
L’accès à Azure Dev Tools for Teaching requiert que l’utilisateur dispose d’un compte Microsoft (MSA). Les étudiants sont automatiquement dirigés vers la création d’un MSA si leur compte n’est pas déjà un MSA ou lié à un MSA. Si votre domaine est associé avec Active Directory, tous les comptes sous ce domaine sont déjà considérés comme des comptes MSA.

Si un étudiant tente de se connecter et reçoit le message d’erreur suivant, utilisez l’une des solutions décrites ci-dessous.

:::image type="content" source="media/troubleshoot-login/error-login-message.png" alt-text="Message d’erreur de connexion." border="false":::

Il existe deux solutions : créer un compte Microsoft ou utiliser un compte Microsoft existant.

## <a name="create-a-new-microsoft-account"></a>Créer un compte Microsoft
### <a name="use-a-university-email-address"></a>Utiliser une adresse e-mail universitaire
Si vous vous connectez avec une adresse e-mail universitaire (par exemple, `John.Smith@university.edu`), vous devez créer un compte Microsoft avec cette adresse. La création d’un compte est gratuite et ne prend que quelques minutes. Le fait de disposer d’un compte Microsoft vous permet de vous connecter automatiquement à tous vos produits Microsoft avec un nom d’utilisateur et un mot de passe uniques.

### <a name="use-a-personal-email-address"></a>Utiliser une adresse e-mail personnelle
Si vous vous connectez avec une adresse e-mail personnelle (par exemple, `John.Smith@email.com`) mais que vous avez également une adresse e-mail universitaire, essayez d’utiliser cette dernière. Si vous vous êtes déjà connecté au webstore de votre établissement à l’aide d’une adresse e-mail personnelle, ou si vous n’avez pas d’adresse e-mail universitaire, vous devez créer ou lier un compte Microsoft avec votre adresse e-mail personnelle.

## <a name="use-an-existing-microsoft-account"></a>Utiliser un compte Microsoft existant
Si un étudiant a un compte Microsoft (par exemple, Xbox), il peut connecter ce compte à un compte Azure Dev Tools.

1. Atteindre https://account.microsoft.com.
1. Connectez-vous avec vos informations d’identification de compte Microsoft.
1. Sélectionnez **Vos informations** dans le menu du ruban supérieur.

1. Cliquez sur **Gérer la manière dont vous vous connectez à Microsoft**. Vous êtes invité à confirmer votre identité. Vous recevrez par e-mail un code de sécurité.

    :::image type="content" source="media/troubleshoot-accounts/manage-sign-in.png" alt-text="Message d’erreur de connexion." border="false":::

1. Entrez le code de sécurité envoyé par e-mail.

    :::image type="content" source="media/troubleshoot-accounts/sign-in-enter-code.png" alt-text="Message d’erreur de connexion." border="false":::

1. Cliquez sur **Ajouter e-mail** pour votre compte, puis entrez votre adresse e-mail universitaire.
La prochaine fois que vous vous connecterez, vous pourrez utiliser votre adresse e-mail universitaire pour accéder à votre Azure Dev Tools for Teaching.

    :::image type="content" source="media/troubleshoot-accounts/manage-sign-in-add-email.png" alt-text="Message d’erreur de connexion." border="false":::

## <a name="next-steps"></a>Étapes suivantes
- [FORUM AUX QUESTIONS](program-faq.md)

- [Options de support](program-support.md)
