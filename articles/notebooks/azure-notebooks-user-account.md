---
title: Connexion à Azure Notebooks (préversion)
description: Configurez votre compte d’utilisateur pour Azure Notebooks à l’aide d’un compte Microsoft ou d’un compte professionnel ou scolaire.
ms.topic: conceptual
ms.date: 04/15/2019
ms.openlocfilehash: 929a21785b5b48c81c2d2016874feed61021d3c0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85832183"
---
# <a name="your-user-account-for-azure-notebooks-preview"></a>Votre compte d’utilisateur pour Azure Notebooks (préversion)

[!INCLUDE [notebooks-status](../../includes/notebooks-status.md)]

Vous pouvez utiliser Azure Notebooks en vous connectant à un compte d'utilisateur ou non :

- Si vous ne souhaitez pas vous connecter, vous pouvez créer et exécuter des blocs-notes, mais vous ne pouvez pas conserver de blocs-notes ni de fichiers de données dans le cadre de projets. Par exemple, les utilisateurs qui reçoivent un lien vers un bloc-notes Azure peuvent s’en servir sans avoir à se connecter.
- Lorsque vous vous connectez, Azure Notebooks conserve tous vos projets au sein de votre compte. Les utilisateurs connectés bénéficient également d’un identifiant utilisateur qui leur permet de partager leurs projets et blocs-notes avec d’autres utilisateurs.
- Si le compte que vous utilisez avec Azure Notebooks est également associé à un abonnement Azure, vous bénéficiez d’avantages supplémentaires. Par exemple, vous pouvez exécuter des blocs-notes sur des serveurs plus puissants, créer des blocs-notes privés et accorder des privilèges d’accès aux blocs-notes à des utilisateurs individuels.

Vous devez disposez d’un compte Microsoft ou d’un compte professionnel ou scolaire pour vous connecter à Azure Notebooks. Lorsque vous sélectionnez la commande **Se connecter** dans l’angle supérieur droit de la page Azure Notebooks, vous êtes invité à entrer votre compte :

![Commande de connexion pour Azure Notebooks](media/accounts/sign-in-command.png)

Toutes les tâches que vous effectuez dans Azure Notebooks sont associées au compte auquel vous êtes connecté. Chaque compte doit également disposer d’un identifiant utilisateur unique, disponible dans votre [profil utilisateur](azure-notebooks-user-profile.md). Par conséquent, vous pouvez utiliser plusieurs comptes pour vous connecter à Azure Notebooks lorsque vous devez travailler sur des ensembles de projets différents qui sont associés à des identités distinctes. Par exemple, il se peut que chaque membre d’une équipe de science des données dispose à la fois d’un compte individuel et du compte de groupe partagé qui est utilisé pour présenter des travaux aux personnes extérieures à l’entreprise. De même, il se peut que les formateurs disposent d’un compte dédié à l’enseignement et d’un autre compte qu’ils utilisent dans le cadre d’expertises extérieures ou de travail en open source.

## <a name="microsoft-accounts"></a>Comptes Microsoft

Les comptes Microsoft permettent de se connecter à de nombreux produits et services Microsoft, tels que Windows, Azure, outlook.com, OneDrive et XBox Live. Si vous utilisez un de ces services, il est fort probable que vous disposiez déjà d’un compte Microsoft. Vous pouvez alors l’utiliser avec Azure Notebooks.

En cas de doute, sélectionnez la commande de **création** dans l’invite du compte. Vous pouvez créer un compte Microsoft à l’aide d’une adresse e-mail quelconque, peu importe le fournisseur.

![Commande de création d’un compte Microsoft](media/accounts/create-new-microsoft-account.png)

> [!Note]
> Si vous tentez de créer un compte avec une adresse e-mail déjà associée à un compte, le message « Vous ne pouvez pas vous connecter avec une adresse e-mail de compte professionnel ou scolaire. Utilisez une adresse e-mail personnelle comme Gmail ou Yahoo!, ou configurez une nouvelle adresse e-mail Outlook. » peut s’afficher. Dans ce cas, essayez de vous connecter avec l’adresse e-mail professionnelle sans créer de compte.

Par défaut, l’accès à Azure Notebooks est bloqué pour les comptes enfant. Si vous tentez de vous connecter avec un compte enfant, l’erreur suivante s’affiche :

![Erreur lors de la tentative de connexion avec un compte enfant. Un problème est survenu, vos parents ont bloqué l’accès.](media/accounts/child-account-error.png)

Pour activer l’accès, le parent doit suivre les instructions suivantes :

1. Accédez à `https://account.live.com/mk` et connectez-vous avec un compte parent.
1. Dans la section relative à l’enfant dont il est question, sélectionnez **Gérer l’accès de cet enfant aux applications tierces**.
1. Sur la page suivante, cliquez sur **Activer l’accès**.
1. Ensuite, lorsque le compte enfant est utilisé pour se connecter à Azure Notebooks, cliquez sur **Oui** dans l’invite des autorisations qui s’affiche.

> [!Warning]
> Si vous activez l’accès à des applications tierces pour Azure Notebooks, vous autorisez également l’accès à toutes les autres applications de tiers. Les parents sont invités à faire preuve de discrétion lorsqu’ils activent l’accès et à surveiller attentivement l’activité de leur enfant.

## <a name="work-or-school-accounts"></a>Comptes professionnels ou scolaires

Un compte professionnel ou scolaire est créé par l’administrateur d’une organisation afin de permettre à un membre d’accéder aux services de cloud computing Microsoft (par exemple, Microsoft 365) et de se connecter à Windows sur un ordinateur joint à un domaine. Un compte professionnel ou scolaire utilise généralement une adresse e-mail associée à l’organisation, comme any-user@contoso.com.

Il se peut que le consentement de l’administrateur soit nécessaire pour se connecter à Azure Notebooks avec un compte professionnel ou scolaire, car Azure Notebooks collecte ou utilise des informations (sans les divulguer) telles que de l’adresse e-mail du compte et les informations de navigation de l’utilisateur. (Les données du navigateur sont utilisées pour optimiser certaines fonctionnalités en fonction de l’usage courant.)

L’administrateur d’un compte professionnel doit donner son consentement pour le compte d’utilisateurs si ces derniers ne peuvent pas le faire individuellement. Dans ce cas, le message « Vous ne pouvez pas accéder à cette application » est affiché auprès des utilisateurs :

![Message « Vous ne pouvez pas accéder à cette application » lors de l’utilisation d’un compte professionnel ou scolaire](media/accounts/consent-permissions-denied.png)

Pour donner votre consentement en tant qu’administrateur, utilisez la [page de consentement administrateur](https://notebooks.azure.com/account/adminConsent) afin de vous guider dans ce processus.

## <a name="next-steps"></a>Étapes suivantes  

> [!div class="nextstepaction"]
> [Modifier votre profil et votre identifiant utilisateur](azure-notebooks-user-profile.md)
