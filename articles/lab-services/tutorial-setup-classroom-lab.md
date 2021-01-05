---
title: Configurer un laboratoire de classe à l’aide d’Azure Lab Services | Microsoft Docs
description: Dans ce tutoriel, vous allez utiliser Azure Lab Services pour configurer un laboratoire de classe composé de machines virtuelles utilisées par les étudiants de votre classe.
ms.topic: tutorial
ms.date: 12/03/2020
ms.openlocfilehash: 3abbf5221382b46dbf4e73f9f4dc3b639bc5ecbd
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602491"
---
# <a name="tutorial-set-up-a-classroom-lab"></a>Tutoriel : Configurer un laboratoire de salle de classe 
Dans ce tutoriel, vous allez configurer un laboratoire de classe avec des machines virtuelles utilisées par les étudiants dans la classe.  

Dans ce tutoriel, vous allez effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un laboratoire de classe
> * Ajouter des utilisateurs au laboratoire
> * Définir un calendrier pour le labo
> * Envoyer un e-mail d’invitation aux étudiants

## <a name="prerequisites"></a>Prérequis
Dans ce tutoriel, vous allez configurer un labo avec des machines virtuelles pour votre classe. Pour configurer un lab de classe dans un compte lab, vous devez être membre de l’un des rôles suivants dans le compte lab : Propriétaire, Créateur de laboratoire ou Contributeur. Le compte que vous avez utilisé pour créer un compte lab est automatiquement ajouté au rôle Propriétaire. Vous pouvez donc utiliser le compte d’utilisateur que vous avez utilisé lors de la création d’un compte de laboratoire pour créer un laboratoire de classe. 

Voici le workflow standard quand vous utilisez Azure Lab Services :

1. Un créateur de compte de laboratoire ajoute d’autres utilisateurs au rôle **Créateur de laboratoire**. Par exemple, le créateur/administrateur du compte lab ajoute des formateurs au rôle **Créateur de laboratoire** afin qu’ils puissent créer des laboratoires pour leurs classes. 
2. Les formateurs créent ensuite des laboratoires avec des machines virtuelles pour leurs classes et envoient des liens d’inscription aux étudiants de la classe. 
3. Les étudiants utilisent le lien d’inscription qu’ils reçoivent des formateurs pour s’inscrire au laboratoire. Une fois inscrits, ils peuvent utiliser des machines virtuelles dans les laboratoires pour faire le travail en classe et leurs devoirs. 

## <a name="create-a-classroom-lab"></a>Créer un laboratoire de classe
Dans cette étape, vous créez un laboratoire pour votre classe dans Azure. 

1. Accédez au [site web Azure Lab Services](https://labs.azure.com). Notez qu’Internet Explorer 11 n’est pas encore pris en charge. 
2. Sélectionnez **Se connecter** et entrez vos informations d’identification. Azure Lab Services prend en charge les comptes professionnels et les comptes Microsoft. 
3. Sélectionnez **Nouveau labo**. 
    
    ![Capture d’écran montrant « Azure Lab Services » avec le bouton « Nouveau labo » sélectionné.](./media/tutorial-setup-classroom-lab/new-lab-button.png)
4. Dans la fenêtre **New Lab** (Nouveau laboratoire), effectuez les actions suivantes : 
    1. Spécifiez un **nom** pour votre labo et sélectionnez **Suivant**.  

        ![Créer un laboratoire de classe](./media/tutorial-setup-classroom-lab/new-lab-window.png)
    2. Dans la page **Informations d’identification de machine virtuelle**, spécifiez les informations d’identification par défaut pour toutes les machines virtuelles du labo. Spécifiez le **nom** et le **mot de passe** de l’utilisateur, puis sélectionnez **Suivant**.  

        ![Fenêtre Nouveau labo](./media/tutorial-setup-classroom-lab/virtual-machine-credentials.png)

        > [!IMPORTANT]
        > Notez le nom d'utilisateur et le mot de passe que vous créez. Ils ne s’afficheront plus.
    3. Dans la page **Stratégies du lab**, sélectionnez **Terminer**. 

        ![Quota pour chaque utilisateur](./media/tutorial-setup-classroom-lab/quota-for-each-user.png)
5. Vous devez voir l’écran suivant qui indique l’état de la création du modèle de machine virtuelle. Cette opération prend jusqu’à 20 minutes. 

    ![État de la création du modèle de machine virtuelle](./media/tutorial-setup-classroom-lab/create-template-vm-progress.png)
8. Dans la page **Modèle**, effectuez les étapes suivantes : Ces étapes sont **facultatives** pour le tutoriel.

    1. Connectez-vous au modèle de machine virtuelle en sélectionnant **Se connecter**. S’il s’agit d’un modèle de machine virtuelle Linux, choisissez si vous voulez vous connecter avec SSH ou RDP (si RDP est activé).
    3. Installez et configurez les logiciels nécessaires pour votre classe sur le modèle de machine virtuelle. 
    4. **Arrêtez** le modèle de machine virtuelle.  

    > [!NOTE]
    > Étant donné que les modèles de machines virtuelles engendrent des **coûts** lors de leur exécution, prenez soin de vérifier que le modèle de machine virtuelle est arrêté lorsque vous n’avez pas besoin de l’exécuter. 

## <a name="publish-the-template-vm"></a>Publier le modèle de machine virtuelle
Au cours de cette étape, vous allez publier le modèle de machine virtuelle. Quand vous publiez le modèle de machine virtuelle, Azure Lab Services crée les machines virtuelles dans le labo à l’aide du modèle. Toutes les machines virtuelles ont la même configuration que le modèle.

1. Dans la page **Modèle**, sélectionnez **Publier** dans la barre d’outils. 

    ![Bouton Publier le modèle](./media/tutorial-setup-classroom-lab/template-page-publish-button.png)

    > [!WARNING]
    > Une fois que vous publiez, vous ne pouvez pas annuler la publication. 
2. Dans la page **Publier le modèle**, entrez le nombre de machines virtuelles que vous souhaitez créer dans le labo, puis sélectionnez **Publier**. 

    ![Publier le modèle - nombre de machines virtuelles](./media/tutorial-setup-classroom-lab/publish-template-number-vms.png)
3. Vous pouvez voir l’**état de publication** du modèle sur la page. Ce processus peut prendre jusqu’à une heure. 

    ![Publier un modèle - progression](./media/tutorial-setup-classroom-lab/publish-template-progress.png)
4. Attendez la fin de la publication, puis passez à la page **Pool de machines virtuelles** en sélectionnant **Machines virtuelles** dans le menu de gauche ou en sélectionnant la vignette **Machines virtuelles**. Vérifiez que des machines virtuelles se trouvent à l’état **Non affectée**. Ces machines virtuelles ne sont pas encore affectées aux étudiants. Elles doivent être à l’état **Arrêtée**. Vous pouvez démarrer la machine virtuelle d’un étudiant, vous y connecter, l’arrêter et la supprimer dans cette page. Vous pouvez démarrer les machines virtuelles dans cette page ou laisser les étudiants le faire. 

    ![Machines virtuelles à l’état Arrêtée](./media/tutorial-setup-classroom-lab/virtual-machines-stopped.png)   

    > [!NOTE]
    > Lorsqu’un formateur active la machine virtuelle d’un étudiant, le quota de l’étudiant n’est pas affecté. Le quota d’un utilisateur spécifie le nombre d’heures de labo disponibles pour l’utilisateur en dehors des heures de classe planifiées. Pour plus d’informations sur les quotas, consultez [Définir des quotas pour les utilisateurs](how-to-configure-student-usage.md?#set-quotas-for-users).

## <a name="set-a-schedule-for-the-lab"></a>Définir un calendrier pour le laboratoire
Créez un événement planifié pour le labo de sorte que les machines virtuelles du labo démarrent ou s’arrêtent automatiquement à des moments précis. Le quota utilisateur (par défaut : 10 heures) que vous avez spécifié précédemment est le temps supplémentaire attribué à chaque utilisateur en dehors de ces heures planifiées. 

1. Basculez vers la page **Planifications**, puis, dans la barre d’outils, sélectionnez **Ajouter un événement planifié**. 

    ![Capture d’écran montrant le bouton « Ajouter un événement planifié » sur la page « Planifications ».](./media/how-to-create-schedules/add-schedule-button.png)
2. Dans la page **Ajouter un événement planifié**, effectuez les étapes suivantes :
    1. Vérifiez que **Standard** est sélectionné pour **Type d’événement**.  
    2. Sélectionnez la **date de démarrage** pour la classe. 
    4. Sélectionnez l’**heure de démarrage** des machines virtuelles.
    5. Sélectionnez l’**heure d’arrêt** des machines virtuelles. 
    6. Sélectionnez le **fuseau horaire** pour les heures de démarrage et d’arrêt que vous avez spécifiées. 
3. Dans la même page **Ajouter un événement planifié**, sélectionnez la planification active à la section **Répéter**.  

    ![Bouton Ajouter une planification dans la page Planifications](./media/how-to-create-schedules/select-current-schedule.png)
5. Dans la boîte de dialogue **Répéter**, effectuez les étapes suivantes :
    1. Vérifiez que le champ **Répéter** est défini sur **chaque semaine**. 
    2. Sélectionnez les jours auxquels vous voulez appliquer la planification. Dans l’exemple suivant, l’option Monday-Friday (Lundi-vendredi) est sélectionnée. 
    3. Sélectionnez une **date de fin** pour la planification.
    8. Sélectionnez **Enregistrer**. 

        ![Définir la répétition de la planification](./media/how-to-create-schedules/set-repeat-schedule.png)

3. À présent, dans la page **Ajouter un événement planifié**, sous **Notes (facultatif)** , entrez une description ou des remarques pour la planification. 
4. Dans la page **Ajouter un événement planifié**, sélectionnez **Enregistrer**. 

    ![Planification hebdomadaire](./media/how-to-create-schedules/add-schedule-page-weekly.png)
5. Accédez à la date de démarrage dans le calendrier pour vérifier que la planification y est configurée.
    
    ![Planification dans le calendrier](./media/how-to-create-schedules/schedule-calendar.png)

    Pour plus d’informations sur la création et la gestion des planifications d’une classe, consultez [Créer et gérer une planification pour les labos](how-to-create-schedules.md).


## <a name="add-users-to-the-lab"></a>Ajouter des utilisateurs au laboratoire

Quand vous ajoutez des utilisateurs, par défaut, l’option **Restreindre l’accès** est activée et, à moins qu’ils ne soient dans la liste des utilisateurs, les étudiants ne peuvent pas s’inscrire au labo même s’ils disposent d’un lien d’inscription. Seuls les utilisateurs figurant dans la liste peuvent s’inscrire au labo à l’aide du lien d’inscription que vous envoyez. Vous pouvez désactiver l’option **Restreindre l’accès** afin de permettre aux étudiants disposant du lien d’inscription de s’inscrire au labo. 

### <a name="add-users-from-an-azure-ad-group"></a>Ajouter des utilisateurs d’un groupe Azure AD

Vous pouvez synchroniser une liste d’utilisateurs du labo avec un groupe Azure AD (Azure Active Directory) existant, ce qui vous évite d’avoir à ajouter ou supprimer manuellement les utilisateurs. 

Vous pouvez créer un groupe Azure AD au sein de la structure Azure Active Directory de votre organisation pour gérer l’accès aux ressources organisationnelles et aux applications cloud. Pour plus d’informations, consultez [Groupes Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-manage-groups). Si votre organisation utilise des services Microsoft Office 365 ou Azure, elle a déjà des administrateurs qui gèrent votre Azure Active Directory. 

> [!IMPORTANT]
> Assurez-vous que la liste des utilisateurs est vide. Si un labo a déjà des utilisateurs que vous aviez ajoutés manuellement ou en important un fichier CSV, l’option de synchronisation du labo avec un groupe existant n’est pas proposée. 

1. Dans le volet gauche, sélectionnez **Utilisateurs**. 
1. Cliquez sur **Synchroniser à partir du groupe**. 

    :::image type="content" source="./media/how-to-configure-student-usage/add-users-sync-group.png" alt-text="Ajouter des utilisateurs par une synchronisation à partir d’un groupe Azure AD":::
    
1. Vous êtes invité à sélectionner un groupe Azure AD existant avec lequel synchroniser votre labo. 
    
    Si vous ne voyez pas de groupe Azure AD dans la liste, cela peut être dû aux raisons suivantes :

    -   Vous êtes un utilisateur invité pour Azure Active Directory (généralement si vous êtes extérieur à l’organisation propriétaire d’Azure AD) et vous n’êtes pas en mesure de rechercher des groupes à l’intérieur d’Azure AD. Dans ce cas, vous ne pouvez pas ajouter un groupe Azure AD au labo. 
    -   Les groupes Azure AD créés par le biais de Teams ne figurent pas dans cette liste. Vous pouvez ajouter l’application Azure Lab Services dans Teams si vous souhaitez créer et gérer des labos directement à partir de Teams. Pour plus d’informations sur la [gestion de la liste des utilisateurs d’un labo dans Teams](how-to-manage-user-lists-within-teams.md). 
1. Une fois que vous avez choisi le groupe Azure AD à utiliser pour synchroniser votre labo, cliquez sur **Ajouter**.
1. Quand un labo est synchronisé, il tire (pull) tous les utilisateurs présents dans le groupe Azure AD en tant qu’utilisateurs du labo, et la liste des utilisateurs est mise à jour. Seules les personnes de ce groupe Azure AD auront accès à votre labo. La liste des utilisateurs est actualisée toutes les 24 heures en fonction de la dernière appartenance du groupe Azure AD. Vous pouvez également cliquer sur le bouton Synchroniser dans l’onglet Utilisateurs pour effectuer une synchronisation manuelle avec les dernières modifications apportées au groupe Azure AD.
1. Invitez les utilisateurs à votre labo en cliquant sur le bouton **Tous les inviter**. Tous les utilisateurs recevront un e-mail contenant le lien d’inscription au labo. 

### <a name="add-users-manually-from-emails-or-csv-file"></a>Ajouter des utilisateurs manuellement à l’aide des e-mails ou d’un fichier CSV

Dans cette section, vous allez ajouter des étudiants manuellement (par adresse e-mail ou en chargeant un fichier CSV). 

#### <a name="add-users-by-email-address"></a>Ajouter des utilisateurs par adresse e-mail

1. Dans le volet gauche, sélectionnez **Utilisateurs**. 
1. Cliquez sur **Ajouter des utilisateurs manuellement**. 

    :::image type="content" source="./media/how-to-configure-student-usage/add-users-manually.png" alt-text="Ajouter des utilisateurs manuellement":::
1. Sélectionnez **Ajouter par adresse e-mail** (option par défaut), entrez les adresses e-mail des étudiants sur des lignes distinctes, ou sur une seule ligne en les séparant par des points-virgules. 

    :::image type="content" source="./media/how-to-configure-student-usage/add-users-email-addresses.png" alt-text="Ajouter les adresses e-mail des utilisateurs":::
1. Sélectionnez **Enregistrer**. 

    La liste affiche les adresses e-mail et les états des utilisateurs actuels, qu’ils soient inscrits ou non au labo. 

    :::image type="content" source="./media/how-to-configure-student-usage/list-of-added-users.png" alt-text="Liste des utilisateurs":::

    > [!NOTE]
    > Une fois les étudiants inscrits au labo, leur nom apparaît dans la liste. Le nom affiché dans la liste est construit en utilisant le prénom et le nom des étudiants dans Azure Active Directory. 

#### <a name="add-users-by-uploading-a-csv-file"></a>Ajouter des utilisateurs en chargeant un fichier CSV

Vous pouvez également ajouter des utilisateurs en chargeant un fichier CSV qui contient leurs adresses e-mail. 

Un fichier texte CSV est utilisé pour stocker des données tabulaires séparées par des virgules (CSV) [nombres et texte]. Au lieu de stocker des informations dans des champs de colonnes (par exemple, dans des feuilles de calcul), un fichier CSV stocke des informations séparées par des virgules. Chaque ligne d’un fichier CSV a le même nombre de « champs » séparés par des virgules. Vous pouvez utiliser Excel pour créer et modifier facilement des fichiers CSV.

1. Dans Microsoft Excel, créez un fichier CSV qui répertorie les adresses e-mail des étudiants dans une colonne.

    :::image type="content" source="./media/how-to-configure-student-usage/csv-file-with-users.png" alt-text="Liste des utilisateurs dans un fichier CSV":::
1. En haut du volet **Utilisateurs**, sélectionnez **Ajouter des utilisateurs**, puis **Charger un fichier CSV**.
1. Sélectionnez le fichier CSV qui contient les adresses e-mail des étudiants, puis sélectionnez **Ouvrir**.

    La fenêtre **Ajouter des utilisateurs** affiche la liste d’adresses e-mail provenant du fichier CSV. 
1. Sélectionnez **Enregistrer**. 
1. Dans le volet **Utilisateurs**, affichez la liste des étudiants ajoutés. 

    :::image type="content" source="./media/how-to-configure-student-usage/list-of-added-users.png" alt-text="Liste des utilisateurs ajoutés dans le volet Utilisateurs"::: 

## <a name="send-invitation-emails-to-users"></a>Envoyer des e-mails d’invitation aux utilisateurs

1. Basculez vers la vue **Utilisateurs** si vous n’y êtes pas déjà. Sélectionnez **Inviter tout le monde** dans la barre d’outils. 

    ![Sélectionner des étudiants](./media/tutorial-setup-classroom-lab/invite-all-button.png)
1. Dans la page **Envoyer l’invitation par e-mail**, entrez un message si vous le souhaitez, puis sélectionnez **Envoyer**. L’e-mail inclut automatiquement le lien d’inscription. Pour obtenir ce lien d’inscription, sélectionnez **...** (points de suspension) dans la barre d’outils, puis sélectionnez **Lien d’inscription**. 

    ![Envoyer un lien d’inscription par e-mail](./media/tutorial-setup-classroom-lab/send-email.png)
4. L’état de l’**invitation**  est indiqué dans la liste **Utilisateurs**. L’état doit passer à **Envoi**, puis à **Envoyée le &lt;date&gt;** . 

Pour plus d’informations sur l’ajout d’élèves à une classe et sur la gestion de leur utilisation du laboratoire, consultez [Comment configurer l’utilisation des étudiants](how-to-configure-student-usage.md).

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez créé un laboratoire pour votre classe dans Azure. Pour savoir comment un étudiant peut accéder à une machine virtuelle dans le laboratoire à l’aide du lien d’inscription, passez au didacticiel suivant :

> [!div class="nextstepaction"]
> [Se connecter à une machine virtuelle dans le laboratoire de classe](tutorial-connect-virtual-machine-classroom-lab.md)

