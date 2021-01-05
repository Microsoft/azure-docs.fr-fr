---
title: Activer la virtualisation imbriquée sur un modèle de machine virtuelle dans Azure Lab Services (IU) | Microsoft Docs
description: Découvrez comment créer un modèle de machine virtuelle contenant plusieurs machines virtuelles.  En d’autres termes, activez la virtualisation imbriquée sur un modèle de machine virtuelle dans Azure Lab Services.
author: emaher
ms.topic: article
ms.date: 06/26/2020
ms.author: enewman
ms.openlocfilehash: f8135e11fb7b7ddb588ab3a8ed01227712072fd2
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94647917"
---
# <a name="enable-nested-virtualization-on-a-template-virtual-machine-in-azure-lab-services-manually"></a>Activer la virtualisation imbriquée sur un modèle de machine virtuelle dans Azure Lab Services manuellement

Une virtualisation imbriquée vous permet de créer un environnement avec plusieurs machines virtuelles à l’intérieur d’une machine virtuelle de labo. La publication du modèle fournit à chaque utilisateur du labo une machine virtuelle configurée avec plusieurs machines virtuelles en son sein.  Pour plus d’informations sur la virtualisation imbriquée et Azure Lab Services, consultez [Activer la virtualisation imbriquée sur un modèle de machine virtuelle dans Azure Lab Services](how-to-enable-nested-virtualization-template-vm.md).

Cet article explique comment configurer la virtualisation imbriquée sur une machine modèle dans Azure Lab Services en utilisant directement des rôles et outils Windows.  Quelques actions sont nécessaires pour permettre à une classe d’utiliser la virtualisation imbriquée.  Les étapes ci-dessous décrivent comment configurer manuellement un modèle de machine Lab Services avec Hyper-V.  Elles conviennent pour Windows Server 2016 ou Windows Server 2019.  

>[!IMPORTANT]
>Sélectionnez **Grande (virtualisation imbriquée)** ou **Moyenne (virtualisation imbriquée)** pour la taille de la machine virtuelle lorsque vous créez le labo.  Sinon, la virtualisation imbriquée ne fonctionnera pas.  

## <a name="enable-hyper-v-role"></a>Activer le rôle Hyper-V

Les étapes suivantes décrivent les actions nécessaires pour activer Hyper-V sur Windows Server à l’aide d’un Gestionnaire de serveur.  Une fois l’installation réussie, un Gestionnaire Hyper-V est disponible pour ajouter, modifier et supprimer des machines virtuelles clientes.

1. Dans la page Tableau de bord du **Gestionnaire de serveur**, cliquez sur **Ajouter des rôles et fonctionnalités**.
2. Sur la page **Avant de commencer**, cliquez sur **Suivant**.
3. Dans la page **sélectionner le type d’installation**, conservez la sélection par défaut de l’installation basée sur un rôle ou une fonctionnalité, puis cliquez sur **Suivant**.
4. Dans la page **Sélectionner le serveur de destination**, choisissez Sélectionner un serveur du pool de serveurs.   Le serveur actif est déjà sélectionné.  Cliquez sur Suivant.
5. Dans la page **Sélectionner des rôles de serveurs**, choisissez **Hyper-V**.  
6. La fenêtre contextuelle de l’**Assistant Ajout de rôles et de fonctionnalités** s’affiche.  Sélectionnez **Inclure les outils de gestion (si applicable)** .  Cliquez sur le bouton **Ajouter des fonctionnalités**.
7. Sur la page **Sélectionner des rôles de serveurs**, cliquez sur **Suivant**.
8. Dans la page **Sélectionner les fonctionnalités**, cliquez sur **Suivant**.
9. Dans la page **Hyper-V**, cliquez sur **Suivant**.
10. Dans la page **Créer des commutateurs virtuels**, acceptez les valeurs par défaut, puis cliquez sur **Suivant**.
11. Dans la page **Migration d’ordinateur virtuel**, acceptez les valeurs par défaut, puis cliquez sur **Suivant**.
12. Dans la page **Emplacements par défaut**, acceptez les valeurs par défaut, puis cliquez sur **Suivant**.
13. Dans la page **Confirmer les sélections d’installation**, choisissez **Redémarrer automatiquement le serveur de destination, si nécessaire**.
14. Lorsque la fenêtre contextuelle de l’**Assistant Ajout de rôles et de fonctionnalités** s’affiche, cliquez sur **Oui**.
15. Cliquez sur **Installer**.
16. Attendez que la page **Progression de l’installation** indique que le rôle Hyper-V est complet.  Il se peut que la machine redémarre au milieu de l’installation.
17. Cliquez sur **Fermer**.

## <a name="enable-dhcp-role"></a>Activer le rôle DHCP

Toutes les machines virtuelles clients Hyper-V créées ont besoin d’une adresse IP dans le réseau NAT.  Nous allons créer le réseau NAT plus tard.  Une façon d’attribuer des adresses IP consiste à configurer l’hôte, en l’occurrence, le modèle de machine virtuelle lab, en tant que serveur DHCP.  Vous trouverez ci-dessous les étapes requises pour activer le rôle DHCP.

1. Dans la page **Tableau de bord** du **Gestionnaire de serveur**, cliquez sur **Ajouter des rôles et fonctionnalités**.
2. Sur la page **Avant de commencer**, cliquez sur **Suivant**.
3. Dans la page **Sélectionner le type d’installation**, sélectionnez **Installation basée sur un rôle ou une fonctionnalité**, puis cliquez sur **Suivant**.
4. Dans la page **Sélectionner le serveur de destination**, choisissez le serveur actif dans le pool de serveurs, puis cliquez sur **Suivant**.
5. Dans la page **Sélectionner des rôles de serveurs**, choisissez **Serveur DHCP**.  
6. La fenêtre contextuelle de l’**Assistant Ajout de rôles et de fonctionnalités** s’affiche.  Sélectionnez **Inclure les outils de gestion (si applicable)** .  Cliquez sur **Ajouter des fonctionnalités**.

    >[!NOTE]
    >Il se peut qu’une erreur de validation s’affiche, indiquant qu’aucune adresse IP statique n’a été trouvée.  Vous pouvez ignorer cet avertissement pour notre scénario.

7. Sur la page **Sélectionner des rôles de serveurs**, cliquez sur **Suivant**.
8. Dans la page **Sélectionner des fonctionnalités**, cliquez sur **Suivant**.
9. Dans la page **Serveur DHCP**, cliquez sur **Suivant**.
10. Sur la page **Confirmer les sélections pour l’installation**, cliquez sur **Installer**.
11. Attendez que la page **Progression de l’installation** indique que le rôle DHCP est complet.
12. Cliquez sur Fermer.

## <a name="enable-routing-and-remote-access-role"></a>Activer le rôle Routage et accès à distance

1. Dans la page **Tableau de bord** du **Gestionnaire de serveur**, cliquez sur **Ajouter des rôles et fonctionnalités**.
2. Sur la page **Avant de commencer**, cliquez sur **Suivant**.
3. Dans la page **Sélectionner le type d’installation**, sélectionnez **Installation basée sur un rôle ou une fonctionnalité**, puis cliquez sur **Suivant**.
4. Dans la page **Sélectionner le serveur de destination**, choisissez le serveur actif dans le pool de serveurs, puis cliquez sur **Suivant**.
5. Dans la page **Sélectionner des rôles de serveurs**, choisissez **Accès à distance**. Cliquez sur **OK**.
6. Dans la page **Sélectionner des fonctionnalités**, cliquez sur **Suivant**.
7. Dans la page **Accès à distance**, cliquez sur **Suivant**.
8. Dans la page **Services de rôle**, sélectionnez **Routage**.
9. La fenêtre contextuelle de l’**Assistant Ajout de rôles et de fonctionnalités** s’affiche.  Sélectionnez **Inclure les outils de gestion (si applicable)** .  Cliquez sur **Ajouter des fonctionnalités**.
10. Cliquez sur **Suivant**.
11. Sur la page **Rôle Web Server (IIS)** , cliquez sur **Suivant**.
12. Dans la page **Sélectionner les services de rôle**, cliquez sur **Suivant**.
13. Sur la page **Confirmer les sélections pour l’installation**, cliquez sur **Installer**.
14. Attendez que la page **Progression de l’installation** indique que le rôle Accès à distance est complet.  
15. Cliquez sur **Fermer**.

## <a name="create-virtual-nat-network"></a>Créer un réseau NAT virtuel

À présent que tous les rôles nécessaires ont été installés, il est temps de créer le réseau NAT.  Le processus de création implique la création d’un commutateur et du réseau NAT.  Un réseau NAT (traduction d’adresses réseau) attribue une adresse IP publique à un groupe de machines virtuelles sur un réseau privé pour permettre la connectivité à Internet.  Dans notre cas, le groupe de machines virtuelles privées comprendra les machines virtuelles imbriquées.  Le réseau NAT permettra aux machines virtuelles imbriquées de communiquer entre elles. Un commutateur est un périphérique réseau qui gère la réception et le routage du trafic dans un réseau.

### <a name="create-a-new-virtual-switch"></a>Créer un commutateur virtuel

1. Ouvrez le **Gestionnaire Hyper-V** à partir d’Outils d’administration Windows.
2. Sélectionnez le serveur actuel dans le menu de navigation de gauche.
3. Cliquez sur **Gestionnaire de commutateur virtuel...** dans le menu **Actions** sur le côté droit du **Gestionnaire Hyper-V**.
4. Dans la fenêtre contextuelle du **Gestionnaire de commutateur virtuel**, pour le type de commutateur à créer, sélectionnez **Interne**.  Cliquez sur **Créer un commutateur virtuel**.
5. Pour le commutateur virtuel nouvellement créé, définissez un nom mémorisable.  Pour cet exemple, nous allons utiliser « LabServicesSwitch ».  Cliquez sur **OK**.
6. Une nouvelle carte réseau est créée.  Son nom est similaire à « vEthernet (LabServicesSwitch) ».  Pour vérifier, ouvrez le **Panneau de configuration**, cliquez sur **Réseau et Internet**, puis cliquez sur **Afficher l’état et la gestion du réseau**.  Sur la gauche, cliquez sur **Modifier les paramètres de l’adaptateur**.

### <a name="create-a-nat-network"></a>Créer un réseau NAT

1. Ouvrez l’outil **Routage et accès à distance** à partir des Outils d’administration Windows.
2. Sélectionnez le serveur local dans la page de navigation de gauche.
3. Choisissez **Action** -> **Configurer et activer le routage et l’accès à distance**.
4. Quand l’**Assistant Installation d’un serveur Routage et accès distant** s’affiche, cliquez sur **Suivant**.
5. Dans la page **Configuration**, sélectionnez la configuration **Traduction d’adresses réseau (NAT)** .  Cliquez sur **Suivant**.

    >[!WARNING]
    >Ne choisissez pas l’option « Accès VPN (Virtual Private Network) et NAT ».

6. Dans la page **Connexion Internet NAT**, choisissez « Ethernet ».  Ne choisissez pas la connexion « vEthernet (LabServicesSwitch) » que nous avons créée dans le Gestionnaire Hyper-V. Cliquez sur **Suivant**.
7. Dans la dernière page de l’Assistant, cliquez sur **Terminer**.
8. Quand la boîte de dialogue **Démarrer le service** s’affiche, cliquez sur **Démarrer le service**.
9. Attendez que le service ait démarré.

## <a name="update-network-adapter-settings"></a>Mettre à jour les paramètres de carte réseau

La carte réseau est associée à l’adresse IP utilisée pour l’adresse IP de passerelle par défaut pour le réseau NAT créé précédemment.  Dans cet exemple, nous créons une adresse IP 192.168.0.1 avec un masque de sous-réseau 255.255.255.0.  Nous allons utiliser le commutateur virtuel créé précédemment.

1. Ouvrez le **Panneau de configuration**, cliquez sur **Réseau et Internet**, puis cliquez sur **Afficher l’état et la gestion du réseau**.
2. Sur la gauche, cliquez sur **Modifier les paramètres de l’adaptateur**.  
3. Dans la fenêtre **Connexions réseau**, double-cliquez sur « vEthernet (LabServicesSwitch) » pour afficher la boîte de dialogue détaillée **État de vEthernet (LabServicesSwitch)** .
4. Cliquez sur le bouton **Propriétés**.
5. Sélectionnez l’élément **Protocole Internet version 4 (TCP/IPv4)** , puis cliquez sur le bouton **Propriétés**.
6. Dans la boîte de dialogue **Propriétés du protocole Internet version 4 (TCP/IPv4)** , sélectionnez **Utiliser l’adresse IP suivante**.  Pour l’adresse IP, entrez 192.168.0.1. Pour le masque de sous-réseau, entrez 255.255.255.0.  Laissez la valeur de la passerelle par défaut vide.  Laissez également les serveurs DNS vides.

    >[!NOTE]
    > Notre plage pour le réseau NAT sera, en notation CIDR, 192.168.0.0/24.  Cela crée une plage d’adresses IP utilisables de 192.168.0.1 à 192.168.0.254.  Par convention, les passerelles ont la première adresse IP dans une plage de sous-réseau.

7. Cliquez sur OK.

## <a name="create-dhcp-scope"></a>Créer une étendue DHCP

Les étapes suivantes sont des instructions pour ajouter une étendue DHCP.  Dans cet article, notre réseau NAT est 192.168.0.0/24 en notation CIDR.  Cela crée une plage d’adresses IP utilisables de 192.168.0.1 à 192.168.0.254.  L’étendue créée doit être comprise dans cette plage d’adresses utilisables, à l’exception de l’adresse IP créée précédemment.

1. Ouvrez **Outils d’administration**, puis l’outil d’administration **DHCP**.
2. Dans l’outil **DHCP**, développez le nœud du serveur actif, puis sélectionnez **IPv4**.
3. Dans le menu Action, choisissez **Nouvelle étendue...**
4. Lorsque l’**Assistant Nouvelle étendue** s’affiche, dans la page **Bienvenue**, cliquez sur **Suivant**.
5. Dans la page **Nom de l’étendue**, entrez « LabServicesDhcpScope » ou un autre nom mémorisable.  Cliquez sur **Suivant**.
6. Dans la page **Plage d’adresses IP**, entrez les valeurs suivantes.

    - 192.168.0.100 pour Adresse IP de début
    - 192.168.0.200 pour Adresse IP de fin
    - 24 pour Longueur
    - 255.255.255.0 pour Masque de sous-réseau

7. Cliquez sur **Suivant**.
8. Dans la page **Ajout d’exclusions et de retard**, cliquez sur **Suivant**.
9. Dans la page **Durée du bail**, cliquez sur **Suivant**.
10. Dans la page **Configuration des paramètres DHCP**, sélectionnez **Oui, je veux configurer ces options maintenant**. Cliquez sur **Suivant**.
11. Sur le **Routeur (Passerelle par défaut)**
12. Ajoutez 192.168.0.1 si ce n’est déjà fait. Cliquez sur **Suivant**.
13. Dans la page **Nom de domaine et serveurs DNS**, si ce n’est déjà fait, ajoutez 168.63.129.16 en tant qu’adresse IP de serveur DNS.  168.63.129.16 est l’adresse IP d’un serveur DNS statique Azure. Cliquez sur **Suivant**.
14. Dans la page **Serveurs WINS**, cliquez sur **Suivant**.
15. Dans la page **Activer l’étendue**, sélectionnez **Oui, je veux activer cette étendue maintenant**.  Cliquez sur **Suivant**.
16. Dans la page **Fin de l’Assistant Nouvelle étendue**, cliquez sur **Terminer**.

## <a name="conclusion"></a>Conclusion

Votre machine modèle est maintenant prête pour la création de machines virtuelles Hyper-V.   Pour obtenir des instructions sur la création de machines virtuelles Hyper-V, consultez [Créer une machine virtuelle dans Hyper-V](/windows-server/virtualization/hyper-v/get-started/create-a-virtual-machine-in-hyper-v).  Visitez également le [Centre d’évaluation Microsoft](https://www.microsoft.com/evalcenter/) pour voir les systèmes d’exploitation et les logiciels disponibles.

## <a name="next-steps"></a>Étapes suivantes

Les étapes suivantes sont communes à la configuration de n’importe quel labo.

- [Ajout d'utilisateurs](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Définir un quota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Définir un calendrier](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Envoyer par mail des liens d’inscription aux étudiants](how-to-configure-student-usage.md#send-invitations-to-users)