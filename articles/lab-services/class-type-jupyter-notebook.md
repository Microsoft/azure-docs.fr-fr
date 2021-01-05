---
title: Configurer un laboratoire pour enseigner la science des données avec Python et Jupyter Notebooks | Microsoft Docs
description: Découvrez comment configurer un laboratoire pour enseigner la science des données avec Python et Jupyter Notebooks.
author: emaher
ms.topic: article
ms.date: 09/29/2020
ms.author: enewman
ms.openlocfilehash: d4034f889334bcf1e4eaa3710a32db60b6a9936b
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94648019"
---
# <a name="set-up-a-lab-to-teach-data-science-with-python-and-jupyter-notebooks"></a>Configurer un laboratoire pour enseigner la science des données avec Python et Jupyter Notebooks
Cet article explique comment configurer un modèle de machine virtuelle dans Lab Services avec les outils nécessaires pour apprendre aux étudiants à utiliser [Jupyter Notebook](http://jupyter-notebook.readthedocs.io/) et à se connecter à leur notebook sur leur machine virtuelle.

Jupyter Notebooks est un projet open source qui permet de combiner facilement du texte enrichi et du code source Python exécutable sur un même canevas appelé notebook. L’exécution d’un notebook produit un enregistrement linéaire des entrées et des sorties. Ces sorties peuvent inclure du texte, des tables d’informations, des nuages de points, etc.

## <a name="set-up-the-lab"></a>Configuration du Lab

### <a name="lab-configuration"></a>Configuration du laboratoire
Pour pouvoir configurer ce Lab, vous devez avoir accès à un abonnement Azure et à un compte Lab. Demandez à l’administrateur de votre organisation si vous pouvez accéder à un abonnement Azure existant. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

Une fois que vous disposez d’un abonnement Azure, créez un compte Lab dans Azure Lab Services en suivant les instructions du tutoriel [Configuration d’un compte Lab](tutorial-setup-lab-account.md). Vous pouvez également utiliser un compte Lab existant.

### <a name="lab-account-settings"></a>Paramètres du compte Lab
Activez les paramètres décrits dans le tableau ci-dessous pour le compte Lab. Pour plus d’informations sur l’activation des images de la Place de marché, consultez [Spécification des images de la Place de Marché accessibles aux créateurs Lab](specify-marketplace-images.md).

| Paramètres du compte lab | Instructions |
| ------------------- | ------------ |
| Image de la Place de marché | Au sein de votre compte Lab, activez l’une des images de la Place de marché Azure en fonction de vos besoins en matière de système d’exploitation : <br/><ul><li>Data Science Virtual Machine – Windows Server 2019</li><li>Data Science Virtual Machine – Ubuntu 18.04</li></ul> |

> [!NOTE]
> Cet article utilise les images Data Science Virtual Machine disponibles sur la Place de marché Azure, car elles sont préconfigurées avec Jupyter Notebook. Cependant, elles incluent également de nombreux autres outils de développement et de modélisation pour la science des données. Si vous ne souhaitez pas ces outils supplémentaires et que vous préférez une installation légère limitée à Jupyter Notebook, créez une image de machine virtuelle personnalisée. Pour voir un exemple, consultez [Installation de JupyterHub sur Azure](http://tljh.jupyter.org/en/latest/install/azure.html). Une fois l’image personnalisée créée, vous pouvez la charger dans une bibliothèque Shared Image Gallery pour pouvoir l’utiliser au sein d’Azure Lab Services. Découvrez comment [utiliser Shared Image Gallery dans Azure Lab Services](how-to-attach-detach-shared-image-gallery.md). 

### <a name="lab-settings"></a>Paramètres du labo
Configurez les paramètres **Taille de machine virtuelle** et **Image de machine virtuelle** comme dans le tableau suivant lors de la configuration d’un Lab de classe. Pour obtenir des instructions sur la création d’un Lab de classe, consultez [Configuration d’un Lab de classe](tutorial-setup-classroom-lab.md).

| Paramètres du labo | Valeur/instructions |
| ------------ | ------------------ | 
| Taille de la machine virtuelle | <p>La taille dépend de la charge de travail à exécuter :</p><ul><li>Petite ou moyenne : adaptée à une configuration de base de l’accès à Jupyter Notebook</li><li>Peu de GPU (calcul) : optimisée pour les applications nécessitant beaucoup de ressources système et réseau, comme l’intelligence artificielle et le Deep Learning</li></ul> | 
| Image de machine virtuelle | <p>Choisissez l’une des images suivantes en fonction de vos besoins en matière de système d’exploitation :</p><ul><li>[Data Science Virtual Machine – Windows Server 2019](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.dsvm-win-2019)</li><li>[Data Science Virtual Machine – Ubuntu 18.04](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-1804?tab=Overview)</li></ul> |


### <a name="template-virtual-machine"></a>Modèle de machine virtuelle
Une fois le Lab créé, un modèle de machine virtuelle est établi en fonction de la taille et de l’image de machine virtuelle choisies. Vous configurez le modèle de machine virtuelle avec tout ce que vous souhaitez fournir à vos étudiants pour ce cours. Pour plus d’informations, consultez [Guide pratique pour gérer le modèle de machine virtuelle](how-to-create-manage-template.md). 

Par défaut, les images Data Science VM sont fournies avec la plupart des frameworks et des outils de science des données requis pour ce type de cours, par exemple :

- [Jupyter Notebooks](http://jupyter-notebook.readthedocs.io/) : une application web qui permet aux scientifiques des données de prendre des données brutes, d’exécuter des calculs et de voir les résultats dans le même environnement. Il s’exécute en local dans le modèle de machine virtuelle.  
- [Visual Studio Code](https://code.visualstudio.com/) : un environnement de développement intégré (IDE) qui offre une expérience interactive riche lors de l’écriture et des tests d’un notebook. Pour plus d’informations, consultez [Utilisation de Jupyter Notebooks dans Visual Studio Code](https://code.visualstudio.com/docs/python/jupyter-support).

### <a name="provide-notebooks-for-the-class"></a>Fourniture de notebooks pour le cours
La tâche suivante consiste à fournir aux étudiants les notebooks qu’ils devront utiliser. Pour donner vos propres notebooks, vous pouvez les enregistrer en local sur le modèle de machine virtuelle. 

Si vous souhaitez utiliser des exemples de notebooks issus d’Azure Machine Learning, consultez [Guide pratique pour configurer un environnement avec Jupyter Notebook](../machine-learning/how-to-configure-environment.md#jupyter). 

### <a name="optional-enable-graphical-desktop-for-linux"></a>Activation du bureau graphique pour Linux (facultatif) 
L’image **Data Science Virtual Machine – Ubuntu** est déjà provisionnée avec X2GO Server et prête à accepter les connexions clientes. Aucune procédure supplémentaire n’est nécessaire lors de la configuration du modèle de machine virtuelle. 

### <a name="publish-the-template-machine"></a>Publication du modèle de machine virtuelle
Lorsque vous publiez le modèle, chacun des étudiants inscrit à votre Lab reçoit une copie du modèle de machine virtuelle comportant tous les outils et notebooks locaux que vous y avez configurés.

## <a name="how-students-connect-to-jupyter-notebooks"></a>Procédure de connexion à Jupyter Notebook pour les étudiants
Une fois le modèle publié, chaque étudiant a accès à une machine virtuelle contenant tous les éléments que vous avez préconfigurés pour le cours, y compris Jupyter Notebook. Les sections suivantes montrent les différents moyens qu’ils ont de se connecter à Jupyter Notebook. 

### <a name="for-windows-vms"></a>Pour les machines virtuelles Windows
Si vous avez fourni aux étudiants des machines virtuelles Windows, ils doivent s’y connecter et utiliser les notebooks Jupyter qui sont disponibles en local sur ces machines virtuelles. 

Pour se connecter à une machine virtuelle Windows, ils peuvent utiliser une connexion Bureau à distance (RDP). Pour connaître la procédure détaillée, consultez [Guide pratique pour accéder à un Lab de classe](how-to-use-classroom-lab.md). 

Les étudiants qui utilisent des Mac ou des Chromebooks peuvent suivre les instructions des articles suivants pour se connecter à la machine virtuelle Windows Data Science VM. 

- [Se connecter à une machine virtuelle avec RDP sur un Mac](connect-virtual-machine-mac-remote-desktop.md)
- [Se connecter à une machine virtuelle avec RDP sur un Chromebook](connect-virtual-machine-chromebook-remote-desktop.md)

### <a name="for-linux-vms"></a>Pour les machines virtuelles Linux
Si vous avez fourni aux étudiants des machines virtuelles Linux, ils ont plusieurs possibilités pour se connecter à Jupyter Notebook dans les machines virtuelles :

- Accéder à Jupyter Notebook en local après connexion à la machine virtuelle :
    - Connexion SSH à la machine virtuelle pour les sessions sur terminal
     - Connexion X2Go à la machine virtuelle pour les sessions graphiques
- Utiliser le tunneling SSH pour se connecter directement au serveur Jupyter sur la machine virtuelle à partir de l’ordinateur local de l’étudiant. 

Les sections suivantes décrivent plus en détails ces moyens de se connecter à Jupyter Notebook. 

#### <a name="ssh-to-virtual-machine"></a>Connexion SSH à la machine virtuelle
Les étudiants peuvent se connecter via SSH à leur machine virtuelle Linux à partir d’une session de terminal. Pour connaître la procédure détaillée, consultez [Guide pratique pour accéder à un Lab de classe](how-to-use-classroom-lab.md). S’ils utilisent un ordinateur client Windows, ils doivent activer un client SSH en téléchargeant [PuTTY](https://www.putty.org/) ou en activant [OpenSSH sur Windows](/windows-server/administration/openssh/openssh_install_firstuse) pour établir une connexion SSH à partir de l’invite de commandes. 

1.  Démarrez la machine virtuelle.
2.  Une fois la machine virtuelle en cours d’exécution, cliquez sur **Se connecter**. La boîte de dialogue qui s’affiche fournit la chaîne de commande SSH, qui se présente ainsi :
    
     ```shell
    ssh -p 12345 student@ml-lab-00000000-0000-0000-0000-000000000000.eastus2.cloudapp.azure.com
     ```
3.  Accédez à l’invite de commandes ou au terminal, puis collez la commande suivante et appuyez sur **ENTRÉE**.
4.  Entrez le mot de passe de connexion à la machine virtuelle. 

Une fois connectés aux machines virtuelles, les étudiants peuvent accéder à Jupyter Notebook et l’exécuter en local.

#### <a name="x2go-to-virtual-machine"></a>Connexion X2Go à la machine virtuelle
L’image **Data Science Virtual Machine – Ubuntu** est déjà provisionnée avec X2GO Server et prête à accepter les connexions clientes. Pour se connecter au bureau graphique de l’ordinateur Linux, les étudiants doivent suivre cette procédure visant à configurer X2Go une fois pour toutes sur leur ordinateur client :

1.  Téléchargez et installez le [client X2Go](https://wiki.x2go.org/doku.php/doc:installation:x2goclient) de votre plateforme cliente.
2.  Sur le [portail Azure Lab Services](https://labs.azure.com), assurez-vous que la machine virtuelle Linux à laquelle vous souhaitez vous connecter est démarrée.
3.  Une fois la machine virtuelle en cours d’exécution, cliquez sur **Se connecter**. La boîte de dialogue qui s’affiche fournit la chaîne de commande SSH, qui se présente ainsi :

    ```
     ssh -p 12345 student@ml-lab-00000000-0000-0000-0000-000000000000.eastus2.cloudapp.azure.com
     ```    
4. Une fois que vous disposez de ces informations, ouvrez l’application cliente X2Go et créez une nouvelle session. 
5.  Renseignez les valeurs suivantes dans le volet **Préférences de session** :
    - **Nom de la session** : nom de votre choix. Nous vous recommandons d’utiliser celui de votre machine virtuelle Lab.
     - **Hôte** : `ml-lab-00000000-0000-0000-0000-000000000000.eastus2.cloudapp.azure.com`
     - **Connexion** : student
     - **Port SSH** : 12345
     - **Type de session** : XFCE
6. Sélectionnez **OK**. 

    > [!NOTE]
     > Lors de la création d’une nouvelle session X2Go, veillez à utiliser le port SSH et **non** le port RDP.

Pour vous connecter à la machine virtuelle, suivez maintenant cette procédure :    

1.  Dans le client X2Go, double-cliquez sur la machine virtuelle à laquelle vous souhaitez vous connecter. 

    ![Client X2Go](./media/class-type-jupyter-notebook/x2go-client.png)
2. Entrez le mot de passe de connexion à la machine virtuelle. (Vous devrez peut-être accorder à X2Go l’autorisation de contourner votre pare-feu pour terminer la connexion.)
3.  L’interface graphique de votre machine virtuelle Ubuntu Data Science VM apparaît.


#### <a name="ssh-tunnel-to-jupyter-server-on-the-vm"></a>Tunnel SSH vers le serveur Jupyter sur la machine virtuelle
Certains étudiants préfèrent se connecter directement à partir de leur ordinateur local au serveur Jupyter sur leur machine virtuelle. Le protocole SSH permet le réacheminement de port entre l’ordinateur local et un serveur distant (dans notre cas, la machine virtuelle Lab de l’étudiant), afin de **« tunneler »** une application exécutée sur un certain port sur le serveur vers le port de mappage sur l’ordinateur local. Les étudiants doivent suivre cette procédure pour établir un tunnel SSH vers le serveur Jupyter sur leur machine virtuelle Lab :

1.  Sur le [portail Azure Lab Services](https://labs.azure.com), assurez-vous que la machine virtuelle Linux à laquelle vous souhaitez vous connecter est démarrée.
2.  Une fois la machine virtuelle en cours d’exécution, cliquez sur **Se connecter**. La boîte de dialogue qui s’affiche fournit la chaîne de commande SSH, qui se présente ainsi :

    ```bash
     ssh -p 12345 student@ml-lab-00000000-0000-0000-0000-000000000000.eastus2.cloudapp.azure.com
     ```
3. Sur votre ordinateur local, lancez un terminal ou une invite de commandes, puis copiez-y la chaîne de connexion SSH. Ensuite, ajoutez `-L 8888:localhost:8888` à la chaîne de commande pour créer le **tunnel** entre les ports. La chaîne finale devrait se présenter ainsi :

    ```bash
     ssh –L 8888:localhost:8888 -p 12345 student@ml-lab-b720853e-570f-49ac-9cb2-bd0bd2aeec35.eastus.cloudapp.azure.com
     ```
4. Appuyez sur **ENTRÉE** pour exécuter la commande. 
5.  Lorsque cela vous est demandé, indiquez le mot de passe de connexion à la machine virtuelle Lab. 
6.  Après connexion à la machine virtuelle, démarrez le serveur Jupyter à l’aide de la commande suivante : 

    ```bash
     jupyter notebook
     ```
7. Cette commande vous fournit une URL dans le terminal ou l’invite de commandes. L’URL doit se présenter ainsi :

    ```bash
     http://localhost:8888/?token=8c09ecfc93e6a8cbedf9c66dffdae19670a64acc1d37
     ```
8. Collez cette URL dans un navigateur sur votre ordinateur local pour vous connecter à Jupyter Notebook et travailler dessus. 

    > [!NOTE]
    > Visual Studio Code offre également une excellente [expérience d’édition Jupyter Notebook](https://code.visualstudio.com/docs/python/jupyter-support). Vous pouvez suivre les instructions de [Connexion à un serveur Jupyter distant](https://code.visualstudio.com/docs/python/jupyter-support#_connect-to-a-remote-jupyter-server) et utiliser la même URL qu’à l’étape précédente pour vous connecter à partir de VS Code au lieu du navigateur. 


## <a name="cost-estimate"></a>Estimation du coût
Nous allons aborder une estimation de coût possible pour cette classe. Nous allons utiliser une classe de 25 élèves. Nous prévoyons 20 heures de temps de classe. En outre, chaque étudiant obtient un quota de 10 heures pour les devoirs ou travaux en dehors des heures de cours planifiées. La taille de la machine virtuelle que nous avons choisie était Peu de GPU (calcul), soit 139 unités Lab. Si vous souhaitez utiliser la petite taille (20 unités Lab) ou la taille moyenne (42 unités Lab), vous pouvez remplacer la partie Unité Lab de l’équation ci-dessous par le nombre correspondant.  

Voici un exemple d’estimation de coût possible pour cette classe : 25 étudiants * (20 heures planifiées + 10 heures de quota) * 139 unités Lab * 0,01 USD par heure = 1 042,5 USD.

Pour plus d’informations sur les tarifs, consultez [Tarification Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).

## <a name="conclusion"></a>Conclusion
Dans cet article, nous avons parcouru les étapes de création d’un laboratoire pour une classe Jupyter Notebooks. Vous pouvez utiliser une configuration similaire pour d’autres cours de Machine Learning.

## <a name="next-steps"></a>Étapes suivantes

Les étapes suivantes sont communes à la configuration de n’importe quel labo.

- [Créer et gérer un modèle](how-to-create-manage-template.md)
- [Ajout d'utilisateurs](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Définir un quota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Définir un calendrier](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Envoyer par mail des liens d’inscription aux étudiants](how-to-configure-student-usage.md#send-invitations-to-users)