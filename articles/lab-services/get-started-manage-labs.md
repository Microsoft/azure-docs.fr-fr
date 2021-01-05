---
title: Prise en main d’Azure Lab Services
description: Cet article explique comment prendre en main Azure Lab Services.
ms.topic: article
ms.date: 11/18/2020
ms.openlocfilehash: 44afe13fb6f555b12dfce939ce8e88e3af8dc7ef
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602663"
---
# <a name="get-started-with-lab-services"></a>Prise en main de Lab Services 

Azure Lab Services permet aux étudiants et aux enseignants d'accéder à des laboratoires informatiques virtuels à partir de leur ordinateur.

Les enseignants doivent être en mesure d'apprendre aux élèves/parents à utiliser Lab Services dans le cadre de leur enseignement par le biais d'un matériel individuel fourni par l'étudiant. Ainsi, les étudiants pourront accéder aux logiciels standard nécessaires à leurs programmes d'études par le biais de machines virtuelles. 

Une machine virtuelle est un environnement virtuel qui agit comme un ordinateur virtuel. Les machines virtuelles ont leur propre processeur, mémoire et stockage. Les machines virtuelles remplacent une machine réelle et peuvent donner aux utilisateurs un accès aux systèmes d’exploitation et aux logiciels sans qu’ils aient besoin de les avoir sur leur propre appareil. Azure Lab Services fournit un outil permettant aux étudiants d’accéder à des machines virtuelles et de les parcourir et permettant au personnel de gérer ses labos d’ordinateurs virtuels. 

Cet article fournit des informations au personnel enseignant sur la manière d'accéder à Azure Lab Services, de gérer le service et d'apprendre aux étudiants/parents à l'utiliser.

## <a name="key-concepts"></a>Concepts clés

### <a name="quota-hours"></a>Heures de quota

Les étudiants peuvent accéder à leurs machines virtuelles à tout moment pendant les heures de cours planifiées sans que cela n’ait d’incidence sur leur heures de quota. Les heures de quota sont définies pour l’ensemble du semestre et déterminent le nombre d’heures pendant lesquelles un étudiant peut utiliser sa machine virtuelle en dehors des heures de cours normales.

Huit heures par semaine, réinitialisées le dimanche, non cumulables.

Pour plus d’informations, consultez [Définir le quota](how-to-configure-student-usage.md#set-quotas-for-users).

### <a name="automatic-shut-down"></a>Arrêt automatique

Afin de réduire les coûts et de ne pas gaspiller les heures associées au quota des étudiants, des arrêts automatiques sont activés pour les labos. Les arrêts automatiques désactiveront les machines virtuelles après une période d’inactivité (aucune entrée à la souris ou au clavier). Les arrêts automatiques fonctionnent en deux étapes : tout d’abord, un étudiant sera déconnecté de la machine virtuelle après une période d’inactivité. À ce stade, la machine virtuelle est toujours **en cours d’exécution** et les étudiants peuvent s’y connecter. Après une autre période d’inactivité faisant suite à la déconnexion, la machine virtuelle s’éteint d’elle-même.

Les arrêts automatiques sont un outil important qui permet d’économiser sur les coûts. Toutefois, ils représentent un défi pour les étudiants en ce qui concerne la sauvegarde de leur travail et le rendu de fichiers de projets volumineux. Si vos étudiants sont souvent déconnectés ou si les machines virtuelles s'éteignent trop rapidement, contactez votre administrateur CTE. 

Pour plus d’informations, consultez [Configurer l’arrêt automatique des machines virtuelles pour un compte lab](how-to-configure-lab-accounts.md).

### <a name="managing-virtual-machines"></a>Gestion des machines virtuelles

La gestion du labo permet aux enseignants de contrôler des éléments tels que la capacité du labo (le nombre de machines virtuelles disponibles pour les étudiants) et de démarrer, d’arrêter ou de réinitialiser manuellement les machines virtuelles. Les enseignants peuvent également se connecter aux machines virtuelles afin de découvrir l’interface avec les étudiants, d’accéder aux fichiers et de résoudre les problèmes liés aux logiciels ou à la machine virtuelle elle-même.

La chose la plus importante à retenir lors de la gestion des machines virtuelles est que chaque fois qu'une machine est **en cours d'exécution**, des frais vous sont facturés, même si personne n'y est connecté.

## <a name="lab-dashboards"></a>Tableaux de bord Lab

### <a name="overview"></a>Vue d’ensemble

Les tableaux de bord de labos dans Azure Lab Services fournissent un instantané des différents aspects d’un labo particulier, notamment les informations sur les machines virtuelles, le nombre de machines virtuelles attribuées ou non, le nombre d’utilisateurs inscrits ou non et les informations sur les planifications des labos. 

> [!NOTE]
> Bien que la plupart des aspects d’administration du tableau de bord et du [site web d’Azure Lab Services](https://labs.azure.com/) soient visibles aux enseignants, les autorisations spécifiques à votre rôle peuvent avoir un impact sur votre capacité à modifier certains critères du tableau de bord. Si vous rencontrez un problème avec la configuration spécifique de votre labo, contactez votre administrateur CTE.

:::image type="content" source="./media/use-dashboard/dashboard.png" alt-text="Portail Azure Lab Services":::

### <a name="examine-a-dashboard"></a>Examiner un tableau de bord

1. Accédez au [site web d’Azure Lab Services](https://labs.azure.com/) et connectez-vous.
1. Sélectionnez votre labo.
1. Vous verrez un **tableau de bord** sur le côté gauche de la fenêtre. Cliquez sur **Tableau de bord** pour afficher un certain nombre de vignettes dans votre tableau de bord.
1. Sous la vignette **Coûts et facturation**, il existe également des vignettes pour les modèles, les pools de machines virtuelles, les utilisateurs et les planifications, qui vous permettent de modifier certains aspects et d’afficher plus de détails sur le labo de classe.

    * Modèle : décrit la date à laquelle le modèle a été créé et la publication la plus récente. 
    * Pool de machines virtuelles : nombre de machines virtuelles attribuées et non attribuées.
    * Utilisateurs : nombre d’utilisateurs inscrits et d’utilisateurs qui ont été ajoutés au labo, mais qui ne sont pas inscrits.
    * Planifications : affiche les événements planifiés à venir pour le labo et un lien pour afficher plus d’événements.

Pour plus d’informations, consultez [Utiliser le tableau de bord](use-dashboard.md).

### <a name="manually-starting-vms"></a>Démarrage manuel des machines virtuelles

1. À partir de la page **Pool de machines virtuelles**, vous pouvez démarrer toutes les machines virtuelles d’un labo en cliquant sur le bouton **Démarrer tout** en haut de la page.

    :::image type="content" source="./media/how-to-set-virtual-machine-passwords/start-all-vms-button.png" alt-text="Démarrer vos machines virtuelles":::
1. Vous pouvez démarrer des machines virtuelles individuelles en cliquant sur le bouton bascule d’état. 

    Le bouton bascule indique **Démarrage** lorsque la machine virtuelle démarre, puis **En cours d’exécution** une fois que la machine virtuelle a démarré.
1. Vous pouvez également sélectionner un certain nombre de machines virtuelles en utilisant les cases à cocher à gauche de la colonne **Nom**. 

    Une fois que vous avez sélectionné les machines virtuelles souhaitées, cliquez sur le bouton **Démarrer** en haut de l’écran.
1. Une fois qu’elles ont démarré, vous pouvez cliquer sur le bouton **Arrêter tout** pour arrêter toutes les machines virtuelles.

    :::image type="content" source="./media/how-to-set-virtual-machine-passwords/stop-all-vms-button.png" alt-text="Arrêter vos machines virtuelles":::

### <a name="stopping-and-resetting-vms"></a>Arrêt et réinitialisation des machines virtuelles

* Vous pouvez arrêter des machines virtuelles individuelles en cliquant sur le bouton bascule d’état.
* Vous pouvez également arrêter plusieurs machines virtuelles en utilisant les cases à cocher et en cliquant sur le bouton « Arrêter » en haut de l’écran.

Si un étudiant rencontre des difficultés à se connecter à la machine virtuelle, ou si la machine virtuelle doit être réinitialisée pour une raison quelconque, vous pouvez utiliser la fonction de réinitialisation.
1. Pour réinitialiser une ou plusieurs machines virtuelles, sélectionnez-les à l’aide des cases à cocher, puis cliquez sur le bouton **Réinitialiser** en haut de la page.
1. Dans la fenêtre contextuelle, cliquez sur **Réinitialiser**.

    :::image type="content" source="./media/how-to-set-virtual-machine-passwords/reset-vms-dialog.png" alt-text="Réinitialiser votre machine virtuelle":::

    > [!NOTE]
    > L’activation d’une machine virtuelle d’étudiant n’aura pas d’incidence sur le quota de l’étudiant. Le quota des utilisateurs spécifie le nombre d’heures de labo disponibles pour l’utilisateur en dehors des heures de cours planifiées.

### <a name="connect-to-vms"></a>Se connecter aux machines virtuelles

Les enseignants peuvent se connecter à la machine virtuelle d’un étudiant tant qu’elle est sous tension et que l’étudiant n’est PAS connecté à la machine virtuelle. En vous connectant à la machine virtuelle, vous serez en mesure d’accéder aux fichiers locaux sur la machine virtuelle et d’aider les étudiants à résoudre les problèmes.

1. Pour vous connecter à la machine virtuelle de l’étudiant, pointez la souris sur la machine virtuelle dans la liste et cliquez sur le bouton **Connexion**. 
1. Suivez ensuite le guide de démarrage pour les étudiants pour Chromebook, Mac ou PC.

:::image type="content" source="./media/how-to-set-virtual-machine-passwords/connect-student-vm.png" alt-text="Se connecter au bouton de machines virtuelles d’étudiant":::

## <a name="manage-users-in-a-lab"></a>Gérer les utilisateurs dans un labo

Les enseignants sont en mesure d’ajouter des utilisateurs étudiants à un labo et de surveiller leurs heures de quota. 

### <a name="add-users-by-email-address"></a>Ajouter des utilisateurs par adresse e-mail

1. À partir du [site web d’Azure Lab Services](https://labs.azure.com/), cliquez sur **Utilisateurs** sur le côté gauche de la fenêtre.
1. En haut de la fenêtre, cliquez sur **Ajouter des utilisateurs** et sélectionnez **Ajouter par adresse e-mail**. 
1. Dans le volet **Ajouter des utilisateurs** qui s’affiche à droite, entrez les adresses e-mail des étudiants sur des lignes distinctes, ou sur une seule ligne en les séparant par des points-virgules.
1. Cliquez sur **Enregistrer**.
1. Votre liste d’utilisateurs sera désormais mise à jour avec les adresses e-mail, les états, les invitations et les heures de quota.

    Une fois les étudiants inscrits à un labo, leurs noms sont mis à jour avec le prénom et le nom figurant dans Azure Active Directory.

    > [!NOTE]
    > Gardez l’option « Restreindre l’accès » activée pour les utilisateurs. Cela signifie que seuls les utilisateurs que vous répertoriez peuvent s’inscrire au labo à l’aide du lien d’inscription que vous envoyez.

### <a name="add-users-using-a-spreadsheet"></a>Ajouter des utilisateurs à l’aide d’une feuille de calcul 

Vous pouvez également ajouter des utilisateurs en chargeant un fichier CSV qui contient leurs adresses e-mail.

1. Dans Microsoft Excel, créez un fichier CSV qui répertorie les adresses e-mail des étudiants dans une colonne.
1. Sur le [site web d’Azure Lab Services](https://labs.azure.com/), en haut de la page **Utilisateurs**, cliquez sur le bouton **Ajouter des utilisateurs**.
1. Sélectionnez **Charger un fichier CSV**.
1. Sélectionnez le fichier CSV qui contient les adresses e-mail des étudiants, puis cliquez sur **Ouvrir**.

    :::image type="content" source="./media/get-started-manage-labs/add-users-spreadsheet.png" alt-text="Ajouter des utilisateurs à l’aide d’une feuille de calcul":::
1. Les adresses e-mail s’affichent maintenant dans la fenêtre de droite. Cliquez sur **Enregistrer**.

    :::image type="content" source="./media/get-started-manage-labs/register-users.png" alt-text="Inscrire des utilisateurs":::

### <a name="register-users"></a>Inscrire des utilisateurs

Une fois que les utilisateurs ont été ajoutés au labo, ils doivent s’inscrire pour accéder aux machines virtuelles. Pour cela, vous pouvez inviter les utilisateurs à partir du portail, qui leur enverra un e-mail contenant le lien d'inscription au labo. Vous pouvez également copier-coller le lien d’inscription dans un e-mail ou toute autre forme de communication avec les étudiants.

1. Dans la page **Utilisateurs**, sélectionnez un ou plusieurs étudiants dans la liste.

    Dans la ligne correspondant à l’étudiant que vous avez sélectionné, sélectionnez l’icône d’enveloppe dans la liste ou cliquez sur **Inviter** en haut de l’écran.

    :::image type="content" source="./media/get-started-manage-labs/send-invitation.png" alt-text="Envoyer une invitation":::
    
    Dans la fenêtre **Envoyer une invitation par e-mail**, entrez un message facultatif (par exemple, un nom d’utilisateur et un mot de passe) pour les étudiants, puis cliquez sur **Envoyer**. 
    
    :::image type="content" source="./media/get-started-manage-labs/send-invitation-mail.png" alt-text="Envoyer une invitation par e-mail":::

    Sinon, à partir de la même page **Utilisateurs**, vous pouvez cliquer sur le bouton **Lien d’inscription** en haut de l’écran. 

    :::image type="content" source="./media/get-started-manage-labs/registration-link.png" alt-text="Lien d’inscription de l’utilisateur":::
    
    Copiez le lien d’inscription dans le champ de texte et collez-le dans un e-mail ou dans l’outil de messagerie sécurisé de votre choix.  
    
    :::image type="content" source="./media/get-started-manage-labs/user-registration.png" alt-text="Envoyer l’inscription de l’utilisateur":::

Après avoir invité des utilisateurs ou partagé le lien, vous pouvez identifier les utilisateurs qui se sont bien inscrits sur la page **Utilisateurs** dans la colonne **État**. 

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser les ressources que vous avez créées dans ce démarrage rapide, supprimez-les.

## <a name="next-steps"></a>Étapes suivantes

[Configurer un compte de laboratoire](tutorial-setup-lab-account.md)
