---
title: Connexion à Azure Notebooks (préversion)
description: Connectez-vous rapidement à Azure Notebooks (préversion) et définissez un ID utilisateur pour pouvoir accéder à des projets enregistrés et partager des notebooks avec d’autres utilisateurs.
ms.topic: quickstart
ms.date: 04/15/2019
ms.openlocfilehash: 168da5d4d36cd4cfadb4ef120ddfabbd261cafaf
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109451"
---
# <a name="quickstart-sign-in-and-set-a-user-id-for-azure-notebooks-preview"></a>Démarrage rapide : Se connecter et définir un identifiant utilisateur pour Azure Notebooks (préversion)

[!INCLUDE [notebooks-status](../../includes/notebooks-status.md)]

Bien que vous puissiez toujours afficher Azure Notebooks sans vous connecter, vous devez vous connecter pour exécuter des notebooks, accéder aux notebooks et projets enregistrés, et partager vos notebooks avec d’autres utilisateurs.

## <a name="sign-in"></a>Se connecter

1. Sélectionnez **Se connecter** dans le coin supérieur droit de [notebooks.azure.com](https://notebooks.azure.com/).

    ![Emplacement de la commande Se connecter dans Azure Notebooks](media/accounts/sign-in-command.png)

1. Quand vous y êtes invité, entrez l’adresse e-mail d’un compte Microsoft, ou d’un compte professionnel ou scolaire, puis sélectionnez **Suivant**. Les types de comptes sont décrits dans [Votre compte d’utilisateur pour Azure Notebooks](azure-notebooks-user-account.md). Si vous n’avez pas de compte Microsoft ou si vous voulez en créer un à utiliser spécifiquement avec Azure Notebooks, sélectionnez **Créer** :

    ![Commande Créer un compte Microsoft dans l’invite de connexion](media/accounts/create-new-microsoft-account.png)

    > [!Tip]
    > Si vous tentez de créer un compte avec une adresse e-mail déjà associée à un compte, le message « Vous ne pouvez pas vous connecter avec une adresse e-mail de compte professionnel ou scolaire. Utilisez une adresse e-mail personnelle comme Gmail ou Yahoo!, ou configurez une nouvelle adresse e-mail Outlook. » peut s’afficher. Dans ce cas, essayez de vous connecter avec l’adresse e-mail professionnelle sans créer de compte.

1. Entrez votre mot de passe lorsque vous y êtes invité.

1. Si vous vous connectez pour la première fois, Azure Notebooks demande l’autorisation d’accéder à votre compte. Sélectionnez **Oui** pour continuer :

    ![Invite relative aux autorisations du compte](media/accounts/account-permission-prompt.png)

## <a name="set-a-user-id"></a>Définir un ID utilisateur

1. Lors de la première connexion, un ID utilisateur temporaire comme « anon idrca3 » vous est attribué. Tant que votre ID utilisateur commence par « anon- », Azure Notebooks vous invite à créer votre propre ID. Votre ID utilisateur étant utilisé dans n’importe quelle URL que vous obtenez pour partager vos projets et les notebooks, choisissez-le de façon à ce qu’il soit unique et significatif pour vous.

    ![Invite à entrer un ID utilisateur pour Azure Notebooks](media/accounts/create-user-id.png)

    Si vous sélectionnez **Non merci**, Azure Notebooks continue de vous demander un ID utilisateur chaque fois que vous vous connectez. Votre ID utilisateur peut également être défini à tout moment dans votre [profil utilisateur](azure-notebooks-user-profile.md).

1. Après vous être connecté, Azure Notebooks vous permet d’accéder à votre page de profil public dans laquelle vous pouvez sélectionner **Modifier les informations de profil** pour remplir le reste de vos informations (pour plus d’informations, consultez [Vos profil et ID utilisateur](azure-notebooks-user-profile.md)) :

    ![Affichage initial d’une page de profil Azure Notebooks](media/accounts/profile-page-new.png)

> [!NOTE]
> Si le message « User ID is already in use » (ID d’utilisateur déjà utilisé) apparaît, essayez un autre ID. Les ID d’utilisateur sont uniques sur tous les comptes Azure Notebooks, et Azure Notebooks réserve également certains ID d’utilisateur, notamment les noms de marque Microsoft.

## <a name="sign-out"></a>Se déconnecter

Pour vous déconnecter, sélectionnez votre nom d’utilisateur dans le coin supérieur droit de la page, puis sélectionnez **Se déconnecter** :

![Emplacement de la commande Se déconnecter dans Azure Notebooks](media/accounts/sign-out-command.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Démarrage rapide : Exporter un projet Jupyter Notebook](quickstart-export-jupyter-notebook-project.md)
