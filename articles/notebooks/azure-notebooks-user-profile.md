---
title: Profil et identifiant utilisateur pour Azure Notebooks (préversion)
description: Guide pratique pour créer et gérer, avec Azure Notebooks, un profil utilisateur ainsi qu’un identifiant utilisateur qui fera partie de l’URL des notebooks partagés.
ms.topic: conceptual
ms.date: 02/25/2019
ms.openlocfilehash: 30d70365fcc0c72df01b4dc059b6e0f4cc607bba
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109502"
---
# <a name="your-profile-and-user-id-for-azure-notebooks-preview"></a>Profil et identifiant utilisateur pour Azure Notebooks (préversion)

[!INCLUDE [notebooks-status](../../includes/notebooks-status.md)]

Dans le puissant espace collaboratif d’Azure Notebooks, votre profil utilisateur présente aux autres votre image publique :

[![Une page de profil Azure Notebooks](media/accounts/profile-page.png)](media/accounts/profile-page.png#lightbox)

Votre ID utilisateur fait partie des URL utilisées pour partager des projets et des notebooks. La liste suivante décrit les différents modèles d’URL :

- `https://notebooks.azure.com/<user_id>`: page de profil.
- `https://notebooks.azure.com/<user_id>/projects`: projets. Vous voyez tous les projets ; les autres utilisateurs ne voient que vos projets publics.
- `https://notebooks.azure.com/<user_id>/projects/<project_id>`: fichiers projet.
- `https://notebooks.azure.com/<user_id>/projects/<project_id>/clones`: clones d’un projet donné.
- `https://notebooks.azure.com/<user_id>/projects/<project_id>/html/<notebook>.ipynb`: aperçu HTML d’un bloc-notes ou d’un fichier donné.

## <a name="your-user-id"></a>L’identifiant utilisateur

Lorsque vous vous connectez à Azure Notebooks pour la première fois, un identifiant utilisateur temporaire (par exemple, « anon-idr3ca ») est automatiquement affecté à votre compte. Tant que votre identifiant commence par « anon- », Azure Notebooks vous invite à le modifier chaque fois que vous vous connectez :

![Message invitant à créer un identifiant utilisateur lors de la connexion à Azure Notebooks](media/accounts/create-user-id.png)

Une commande **Configurer l’identifiant utilisateur** s’affiche également à côté du nom d’utilisateur temporaire :

![Commande Configurer l’identifiant utilisateur qui s’affiche lorsqu’un identifiant temporaire est utilisé](media/accounts/configure-user-id-command.png)

Vous pouvez également modifier votre identifiant utilisateur à tout moment sur votre page de profil.

Un identifiant utilisateur doit contenir entre quatre et seize lettres, chiffres et traits d’union. Aucun autre caractère n’est autorisé. Par ailleurs, l’identifiant utilisateur ne peut ni commencer ou se terminer par un trait d’union ni comporter plusieurs traits d’union à la suite. Étant donné que les identifiants utilisateur sont uniques pour tous les comptes Azure Notebooks, le message « Identifiant utilisateur déjà utilisé » est susceptible de s’afficher. (Le message s’affiche également si vous essayez d’utiliser une marque déposée Microsoft comme identifiant utilisateur.) Dans ces cas de figure, choisissez un identifiant utilisateur différent.

> [!Important]
> Toute modification de l’identifiant invalide les URL partagées avec le précédent identifiant. Vous pouvez revenir à l’identifiant précédent pour revalider les liens. Toutefois, il est possible qu’un autre utilisateur revendique entre-temps l’identifiant inutilisé.

## <a name="your-profile"></a>Le profil

Un profil se compose d’informations publiques à l’URL `https://notebooks.azure.com/<user_id>`. La page de profil affiche également les projets utilisés récemment et tous les projets marqués d’une étoile.

Pour modifier votre profil, utilisez la commande **Modifier les informations de profil** sur votre page de profil. Les sections de votre profil sont les suivantes :

| Section | Contents |
| --- | --- |
| Photo de profil | Image qui s’affiche sur la page de profil. |
| Informations sur le compte | Nom d’affichage, identifiant utilisateur et compte de courrier public. Le compte de courrier représente pour les autres utilisateurs un moyen de vous contacter. Il peut être différent du [compte](azure-notebooks-user-account.md) utilisé pour vous connecter à Azure Notebooks. |
| Informations de profil | Localisation, entreprise, poste, site web et brève description de vous-même. |
| Profils sociaux | Vos identifiants GitHub, Twitter et Facebook, si vous souhaitez les partager. |
| Paramètres de confidentialité | Deux commandes :<ul><li>**Exporter mon profil** : crée et télécharge un fichier *.zip* contenant toutes les informations qu’Azure Notebooks enregistre dans votre profil, notamment votre photo, vos informations de profil et les journaux d’activité de sécurité.</li><li>**Supprimer mon compte** : supprime définitivement toutes vos informations personnelles stockées dans Azure Notebooks.</li></ul> |
| Activer les fonctionnalités du site | Permet de contrôler les différents aspects du comportement d’Azure Notebooks :<ul><li>**Front-end unifié pour les blocs-notes** : permet d’accélérer le démarrage des blocs-notes et d’améliorer la persistance.</li><li>**Exécuter dans JupyterLab par défaut** : par défaut, Azure Notebooks propose une interface utilisateur simple, adaptée à la plupart des utilisateurs. JupyterLab offre aux utilisateurs expérimentés une interface plus riche, mais plus compliquée.</li><li>**Site web VNext** : active la mise en page web modernisée indiquée dans cette documentation.</li></ul> |

## <a name="next-steps"></a>Étapes suivantes  

> [!div class="nextstepaction"]
> [Démarrage rapide : Exporter un projet Jupyter Notebook](quickstart-export-jupyter-notebook-project.md)
