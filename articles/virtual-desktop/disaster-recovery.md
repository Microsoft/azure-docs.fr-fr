---
title: Configurer le plan de récupération d’urgence Windows Virtual Desktop - Azure
description: Comment configurer un plan de continuité d'activité et reprise d'activité pour le déploiement de Windows Virtual Desktop.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: how-to
ms.date: 10/09/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 968f82a143872fd282222539ab71a70db488a20d
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91935577"
---
# <a name="set-up-a-business-continuity-and-disaster-recovery-plan"></a>Configurer un plan de continuité d’activité et la reprise d’activité (BCDR)

Pour garantir la sécurité des données de votre organisation, vous devrez peut-être adopter une stratégie de continuité d'activité et reprise d'activité (BCDR). Une stratégie BCDR solide permet aux applications et à la charge de travail de fonctionner pendant les interruptions de service planifiées et non planifiées.

Windows Virtual Desktop propose BCDR pour le service Windows Virtual Desktop afin de préserver les métadonnées client pendant les pannes. Lorsqu’une panne se produit dans une région, les composants de l’infrastructure de service basculent vers l’emplacement secondaire et continuent de fonctionner normalement. Vous pouvez toujours accéder aux métadonnées relatives au service, et les utilisateurs peuvent toujours se connecter aux hôtes disponibles. Les connexions de l’utilisateur final resteront en ligne tant que l’environnement ou les hôtes du locataire resteront accessibles.

Pour vous assurer que les utilisateurs peuvent toujours se connecter pendant une panne de région, vous devez répliquer leurs machines virtuelles à un autre emplacement. En cas de panne, le site principal bascule vers les machines virtuelles répliquées dans l’emplacement secondaire. Les utilisateurs peuvent continuer à accéder aux applications à partir de l’emplacement secondaire sans interruption. En plus de la réplication de machine virtuelle, vous devez conserver les identités utilisateur accessibles à l’emplacement secondaire. Si vous utilisez des conteneurs de profils, vous devrez également les répliquer. Enfin, assurez-vous que vos applications d’entreprise qui reposent sur les données de l’emplacement principal peuvent basculer avec le reste des données.

Pour résumer, pour maintenir les utilisateurs connectés pendant une panne, vous devez effectuer les opérations suivantes dans cet ordre :

- Répliquez les machines virtuelles dans un emplacement secondaire.
- Si vous utilisez des conteneurs de profils, configurez la réplication des données dans l’emplacement secondaire.
- Vérifiez que les identités des utilisateurs que vous configurez dans l’emplacement principal sont disponibles dans l’emplacement secondaire.
- Assurez-vous que toutes les applications métier reposant sur des données de votre emplacement principal sont basculées vers l’emplacement secondaire.

## <a name="vm-replication"></a>Réplication de machines virtuelles

Tout d’abord, vous devez répliquer vos machines virtuelles vers l’emplacement secondaire. Vos options pour ce faire dépendent de la façon dont vos machines virtuelles sont configurées :

- Vous pouvez configurer toutes vos machines virtuelles pour les pools d’hôtes personnels et regroupés avec Azure Site Recovery. Avec cette méthode, il vous suffit de configurer un pool hôte et les groupes d’applications et espaces de travail associés.
- Vous pouvez créer un pool hôte dans la région de basculement tout en laissant toutes les ressources de votre emplacement de basculement désactivées. Pour cette méthode, vous devez configurer de nouveaux groupes d’applications et espaces de travail dans la région de basculement. Vous pouvez ensuite utiliser un plan Azure Site Recovery pour activer les pools d’hôtes.
- Vous pouvez créer un pool hôte rempli de machines virtuelles créées à la fois dans les régions principale et de basculement, tout en gardant les machines virtuelles dans la région de basculement désactivées. Dans ce cas, il vous suffit de configurer un pool hôte et les groupes d’applications et espaces de travail associés. Vous pouvez utiliser un plan Azure Site Recovery pour activer les pools d’hôtes avec cette méthode.

Nous vous recommandons d’utiliser [Azure Site Recovery](../site-recovery/site-recovery-overview.md) pour gérer la réplication de machines virtuelles dans d’autres emplacements Azure, comme décrit dans [Architecture de récupération d’urgence Azure vers Azure](../site-recovery/azure-to-azure-architecture.md). Nous vous recommandons d’utiliser Azure Site Recovery pour les pools d’hôtes personnels, car Azure Site Recovery prend en charge à la fois [les SKU serveur et ceux basés sur le client](../site-recovery/azure-to-azure-support-matrix.md#replicated-machine-operating-systems).

Si vous utilisez Azure Site Recovery, vous n’avez pas besoin d’inscrire ces machines virtuelles manuellement. L’agent Windows Virtual Desktop de la machine virtuelle secondaire utilise automatiquement le dernier jeton de sécurité pour se connecter à l’instance de service la plus proche. La machine virtuelle (hôte de session) dans l’emplacement secondaire devient automatiquement une partie du pool d’hôtes. L’utilisateur final doit se reconnecter en cours de route, mais en dehors de cela, aucune autre opération manuelle n’est nécessaire.

S’il existe des connexions utilisateur existantes pendant la panne, avant que l’administrateur puisse démarrer le basculement vers la région secondaire, vous devez mettre fin aux connexions utilisateur dans la région actuelle.

Pour déconnecter les utilisateurs dans Windows Virtual Desktop (classique), exécutez cette applet de commande :

```powershell
Invoke-RdsUserSessionLogoff
```

Pour déconnecter des utilisateurs dans la version intégrée à Azure de Windows Virtual Desktop, exécutez cette applet de commande :

```powershell
Remove-AzWvdUserSession
```

Une fois que vous avez déconnecté tous les utilisateurs de la région primaire, vous pouvez basculer les machines virtuelles dans la région primaire et permettre aux utilisateurs de se connecter aux machines virtuelles de la région secondaire. Pour plus d’informations sur le fonctionnement de ce processus, consultez [Répliquer des machines virtuelles Azure dans une autre région Azure](../site-recovery/azure-to-azure-how-to-enable-replication.md).

## <a name="virtual-network"></a>Réseau virtuel

Ensuite, envisagez la connectivité réseau pendant la panne. Vous devez vous assurer que vous avez configuré un réseau virtuel (VNET) dans votre région secondaire. Si vos utilisateurs doivent accéder à des ressources locales, vous devez configurer ce réseau virtuel pour y accéder. Vous pouvez établir des connexions locales avec un VPN, ExpressRoute ou un Virtual WAN.

Nous vous recommandons d’utiliser Azure Site Recovery pour configurer le réseau virtuel dans la région de basculement, car il conserve les paramètres de votre réseau principal et n’a pas besoin de peering.

## <a name="user-identities"></a>Identités utilisateur

Ensuite, assurez-vous que le contrôleur de domaine est disponible à l’emplacement secondaire. 

Il existe trois façons de maintenir la disponibilité du contrôleur de domaine :

   - Utiliser un contrôleur de domaine Active Directory à l’emplacement secondaire
   - Utiliser un contrôleur de domaine Active Directory local
   - Répliquer un contrôleur de domaine Active Directory à l’aide d’[Azure Site Recovery](../site-recovery/site-recovery-active-directory.md)

## <a name="user-and-app-data"></a>Données d’utilisateur et d’application

Si vous utilisez des conteneurs de profils, l’étape suivante consiste à répliquer les données dans l’emplacement secondaire. Vous avez le choix entre cinq options pour stocker les profils FSLogix :

   - Espaces de stockage directs (S2D)
   - Lecteurs réseau (machine virtuelle avec lecteurs supplémentaires)
   - Azure Files
   - Azure NetApp Files
   - Cloud Cache pour la réplication

Pour davantage d’informations, consultez [Options de stockage pour conteneurs de profil FSLogix dans Windows Virtual Desktop](store-fslogix-profile.md).

Si vous configurez la récupération d’urgence pour les profils, vous avez le choix entre les options suivantes :

   - Configurez la réplication Azure native (par exemple, réplication de compte de stockage Azure Files standard, réplication Azure NetApp Files ou Azure File Sync pour les serveurs de fichiers).
    
     >[!NOTE]
     >La réplication NetApp est automatique une fois que vous avez effectué la configuration initiale. Avec les plans Azure Site Recovery, vous pouvez ajouter des pré-scripts et post-scripts pour basculer des ressources autres que des machines virtuelles et répliquer des ressources de stockage Azure.

   - Configurez FSLogix Cloud Cache pour les données d’application et les données utilisateur.
   - Configurez la récupération d’urgence pour les données d’application uniquement pour garantir l’accès à tout moment aux données critiques de l’entreprise. Avec cette méthode, vous pouvez récupérer les données utilisateur une fois la panne terminée.

Voyons comment configurer FSLogix pour configurer la récupération d’urgence pour chaque option.

### <a name="fslogix-configuration"></a>Configuration de FSLogix

L’agent FSLogix peut prendre en charge plusieurs emplacements de profil si vous configurez les entrées du Registre pour FSLogix.

Pour configurer les entrées du Registre :

1. Ouvrez l’**Éditeur du Registre**.
2. Accédez à **Computer** > **HKEY_LOCAL_MACHINE** > **SOFTWARE** > **FSLogix** > **Profiles**.
   
     > [!div class="mx-imgBorder"]
     > ![Capture d’écran de la fenêtre Profils dans l’éditeur du Registre. VHDLocation est sélectionné.](media/regedit-profiles.png)

3. Cliquez avec le bouton droit sur **VHDLocations**, puis sélectionnez **Modifier les chaînes multiples**.

     > [!div class="mx-imgBorder"]
     > ![Capture d’écran de la fenêtre Modifier les chaînes multiples. Les données de la valeur répertorient les emplacements USA Centre et USA Est.](media/multi-string-edit.png)

4. Dans le champ **Données de valeur**, entrez les emplacements que vous souhaitez utiliser.
5. Quand vous avez terminé, sélectionnez **OK**.

Si le premier emplacement n’est pas disponible, l’agent FSLogix bascule automatiquement vers le deuxième, et ainsi de suite.

Nous vous recommandons de configurer l’agent FSLogix avec un chemin d’accès vers l’emplacement secondaire dans la région principale. Une fois l’emplacement principal arrêté, l’agent FLogix est répliqué dans le cadre de la réplication Azure Site Recovery de la machine virtuelle. Une fois que les machines virtuelles répliquées sont prêtes, l’agent tente automatiquement d’établir le chemin d’accès vers la région secondaire.

Par exemple, supposons que vos machines virtuelles d’hôte de session principales se trouvent dans la région USA Centre, mais que votre conteneur de profil se trouve dans la région USA Centre pour des raisons de performances.

Dans ce cas, vous configurez l’agent FSLogix avec un chemin d’accès au stockage dans USA Centre. Vous configurez les machines virtuelles d’hôte de session pour qu’elles soient répliquées dans USA Ouest. Une fois que le chemin vers USA Centre échoue, l’agent tente de créer un nouveau chemin pour le stockage dans USA Ouest à la place.

### <a name="s2d"></a>S2D

Étant donné que S2D gère la réplication entre les régions en interne, vous n’avez pas besoin de configurer manuellement le chemin d’accès secondaire.

### <a name="network-drives-vm-with-extra-drives"></a>Lecteurs réseau (machine virtuelle avec lecteurs supplémentaires)

Si vous répliquez les machines virtuelles de stockage réseau à l’aide d’Azure Site Recovery comme machines virtuelles d’hôte de session, la récupération conserve le même chemin d’accès, ce qui signifie que vous n’avez pas besoin de reconfigurer FSlogix.

### <a name="azure-files"></a>Azure Files

Azure Files prend en charge la réplication asynchrone entre les régions que vous pouvez spécifier lors de la création du compte de stockage. Si la nature asynchrone d’Azure Files couvre déjà vos objectifs de récupération d’urgence, vous n’avez pas besoin de procéder à une configuration supplémentaire.

Si vous avez besoin d’une réplication synchrone pour réduire la perte de données, nous vous recommandons d’utiliser FSLogix Cloud Cache à la place.

>[!NOTE]
>Cette section ne couvre pas le mécanisme d’authentification par basculement pour Azure Files.

### <a name="azure-netapp-files"></a>Azure NetApp Files

Pour en savoir plus sur Azure NetApp Files, consultez [Créer un peering de réplication pour Azure NetApp Files](../azure-netapp-files/cross-region-replication-create-peering.md).

## <a name="app-dependencies"></a>Dépendances d’application

Enfin, assurez-vous que toutes les applications d’entreprise qui reposent sur les données situées dans la région primaire peuvent basculer vers l’emplacement secondaire. Veillez également à configurer les paramètres que les applications doivent utiliser dans le nouvel emplacement. Par exemple, si une des applications dépend du serveur principal SQL, veillez à répliquer SQL dans l’emplacement secondaire. Vous devez configurer l’application pour qu’elle utilise l’emplacement secondaire dans le cadre du processus de basculement ou de sa configuration par défaut. Vous pouvez modéliser des dépendances d’application sur des plans Azure Site Recovery. Pour plus d’informations, consultez [À propos des plans de récupération](../site-recovery/recovery-plan-overview.md).

## <a name="disaster-recovery-testing"></a>Tests de récupération d'urgence

Une fois que vous avez fini de configurer la récupération d’urgence, vous pouvez tester votre plan pour vous assurer qu’il fonctionne.

Voici quelques suggestions pour tester votre plan :

- Si les machines virtuelles de test ont accès à Internet, elles prennent le relais des hôtes de session existants pour les nouvelles connexions, mais toutes les connexions existantes à l’hôte de session d’origine restent actives. Assurez-vous que l’administrateur exécutant le test déconnecte tous les utilisateurs actifs avant de tester le plan. 
- Vous ne devriez effectuer des tests de récupération d’urgence complets qu’au cours d’une fenêtre de maintenance pour ne pas perturber vos utilisateurs. Vous pouvez également utiliser un pool d’hôtes dans l’environnement de validation pour le test. 
- Assurez-vous que votre test couvre toutes les applications critiques.
- Nous vous recommandons de basculer uniquement jusqu’à 100 machines virtuelles à la fois. Si vous avez plus de machines virtuelles, nous vous recommandons de les faire basculer par lots séparés de 10 minutes.

## <a name="next-steps"></a>Étapes suivantes

Si vous avez des questions sur la sécurisation de vos données en plus de la planification en prévision des pannes, consultez notre [guide de sécurité](security-guide.md).