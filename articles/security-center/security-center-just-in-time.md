---
title: Accès Juste à temps à la machine virtuelle dans Azure Security Center | Microsoft Docs
description: Ce document montre comment l’accès juste-à-temps (JAT) à la machine virtuelle dans Azure Security Center peut vous aider à contrôler l’accès à vos machines virtuelles Azure.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 07/12/2020
ms.author: memildin
ms.openlocfilehash: 55e9d2d1af863084b080c2de7833712413221050
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96445635"
---
# <a name="secure-your-management-ports-with-just-in-time-access"></a>Sécuriser vos ports de gestion avec un accès juste-à-temps

Verrouillez le trafic entrant sur vos machines virtuelles Azure grâce à la fonctionnalité d’accès à la machine virtuelle juste-à-temps (JAT) d’Azure Security Center. Cela réduit l’exposition aux attaques tout en offrant un accès facile lorsque vous devez vous connecter à une machine virtuelle.

Pour obtenir une explication complète sur le fonctionnement de l’accès JAT et la logique sous-jacente, consultez [Explication de l’option juste-à-temps](just-in-time-explained.md).

Cette page vous explique comment inclure l’accès JAT dans votre programme de sécurité. Vous découvrirez comment effectuer les actions suivantes : 

- **Activer l’accès JAT sur vos machines virtuelles** : vous pouvez activer l’accès JAT avec vos propres options personnalisées pour une ou plusieurs machines virtuelles à l’aide de Security Center, de PowerShell ou de l’API REST. Vous pouvez également activer l’accès JAT avec les paramètres codés en dur par défaut à partir des machines virtuelles Azure. Lorsqu’il est activé, l’accès JAT verrouille le trafic entrant sur vos machines virtuelles Azure en créant une règle dans votre groupe de sécurité réseau.
- **Demander l’accès à une machine virtuelle pour laquelle l’accès JAT est activé** : l’objectif de l’accès JAT est de garantir que, même si votre trafic entrant est verrouillé, Security Center offre toujours un accès facile pour se connecter aux machines virtuelles en cas de besoin. Vous pouvez demander l’accès à une machine virtuelle prenant en charge l’accès JAT à partir de Security Center, des machines virtuelles Azure, de PowerShell ou de l’API REST.
- **Auditer l’activité** : pour vous assurer que vos machines virtuelles sont correctement sécurisées, examinez les accès à vos machines virtuelles prenant en charge l’accès JAT dans le cadre de vos vérifications de sécurité régulières.   



## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|Disponibilité générale (GA)|
|Prix :|Nécessite [Azure Defender pour les serveurs](defender-for-servers-introduction.md)|
|Machines virtuelles prises en charge :|![Oui](./media/icons/yes-icon.png) Machines virtuelles déployées via Azure Resource Manager.<br>![Non](./media/icons/no-icon.png) Machines virtuelles déployées avec des modèles de déploiement classiques. [En savoir plus sur ces modèles de déploiement](../azure-resource-manager/management/deployment-models.md).<br>![Non](./media/icons/no-icon.png) Machines virtuelles protégées par des pare-feu Azure contrôlés par [Azure Firewall Manager](../firewall-manager/overview.md)|
|Rôles et autorisations obligatoires :|Les rôles **Lecteur** et **SecurityReader** peuvent tous deux afficher l’état et les paramètres de l’accès JAT.<br>Pour créer des rôles personnalisés pouvant fonctionner avec l’accès JAT, consultez [Quelles autorisations sont nécessaires pour configurer et utiliser l’accès JAT ?](just-in-time-explained.md#what-permissions-are-needed-to-configure-and-use-jit).<br>Pour créer un rôle de moindre privilège pour les utilisateurs qui doivent demander un accès JAT à une machine virtuelle et n’exécuter aucune autre opération JAT, utilisez le script [Set-JitLeastPrivilegedRole](https://github.com/Azure/Azure-Security-Center/tree/master/Powershell%20scripts/JIT%20Custom%20Role) à partir des pages de la communauté GitHub Security Center.|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Oui](./media/icons/yes-icon.png) National/souverain (US Gov, Chine Gov, autres Gov)|
|||


## <a name="enable-jit-vm-access"></a>Activer l’accès JAT aux machines virtuelles <a name="jit-configure"></a>

Vous pouvez activer l’accès JAT aux machines virtuelles avec vos propres options personnalisées pour une ou plusieurs machines virtuelles à l’aide de Security Center ou par programmation. 

Vous pouvez également activer l’accès JAT avec les paramètres codés en dur par défaut à partir des machines virtuelles Azure.

Chacune de ces options est expliquée dans un onglet séparé ci-dessous.

### <a name="azure-security-center"></a>[**Azure Security Center**](#tab/jit-config-asc)

### <a name="enable-jit-on-your-vms-from-azure-security-center"></a>Activer l’accès JAT sur vos machines virtuelles à partir d’Azure Security Center <a name="jit-asc"></a>

:::image type="content" source="./media/security-center-just-in-time/jit-config-security-center.gif" alt-text="Configuration d’un accès aux machines virtuelles juste-à-temps dans Azure Security Center":::

À partir de Security Center, vous pouvez activer et configurer l’accès JAT à la machine virtuelle.

1. Ouvrez le tableau de bord Azure Defender et, dans la zone de protection avancée, sélectionnez **Accès juste-à-temps aux machines virtuelles**.

    La page **Accès juste-à-temps aux machines virtuelles** s’ouvre avec vos machines virtuelles regroupées sous les onglets suivants :

    - **Configuré** : machines virtuelles configurées pour prendre en charge l’accès juste-à-temps aux machines virtuelles. Pour chaque machine virtuelle, l’onglet Configuré affiche :
        - le nombre de demandes d’accès JAT approuvées au cours des sept derniers jours
        - la date et l’heure du dernier accès
        - la configuration des détails de la connexion
        - le dernier utilisateur
    - **Non configuré** : machines virtuelles sans l’accès JAT activé, mais qui peuvent prendre en charge l’accès JAT. Nous vous recommandons d’activer l’accès JAT pour ces machines virtuelles.
    - **Non pris en charge** : machines virtuelles sans l’accès JAT activé et qui ne prennent pas en charge la fonctionnalité. Votre machine virtuelle peut se trouver dans cet onglet pour les raisons suivantes :
      - Groupe de sécurité réseau (NSG) manquant : l’accès JAT requiert la configuration d’un NSG
      - Machine virtuelle classique : l’accès JAT prend en charge les machines virtuelles déployées via Azure Resource Manager, et non un « déploiement classique ». [En savoir plus sur les modèles de déploiement classiques vs Azure Resource Manager](../azure-resource-manager/management/deployment-models.md).
      - Autre : votre machine virtuelle peut figurer dans cet onglet si la solution JAT est désactivée dans la stratégie de sécurité de l’abonnement ou du groupe de ressources.

1. Dans l’onglet **Non configuré**, marquez les machines virtuelles à protéger avec l’accès JAT et sélectionnez **Activer l’accès JAT sur les machines virtuelles**. 

    La page Accès JAT aux machines virtuelles s’ouvre et répertorie les ports que Security Center recommande de protéger :
    - 22 - SSH
    - 3389 - RDP
    - 5985 - WinRM 
    - 5986 - WinRM

    Pour accepter les paramètres par défaut, sélectionnez **Enregistrer**.

1. Pour personnaliser les options JAT :

    - Ajoutez des ports personnalisés avec le bouton **Ajouter**. 
    - Modifiez l’un des ports par défaut en le sélectionnant dans la liste.

    Pour chaque port (personnalisé et par défaut), le volet **Ajouter une configuration de port** offre les options suivantes :

    - **Protocole** : protocole autorisé sur ce port lorsqu’une requête est approuvée
    - **Adresses IP sources autorisées** : plages d’adresses IP autorisées sur ce port lorsqu’une requête est approuvée
    - **Durée maximale de la requête** : fenêtre de temps maximale pendant laquelle un port spécifique peut être ouvert

     1. Définissez la sécurité des ports en fonction de vos besoins.

     1. Sélectionnez **OK**.

1. Sélectionnez **Enregistrer**.



### <a name="edit-the-jit-configuration-on-a-jit-enabled-vm-using-security-center"></a>Modifier la configuration de l’accès JAT sur une machine virtuelle prenant en charge l’accès JAT à l’aide de Security Center <a name="jit-modify"></a>

Vous pouvez modifier la configuration juste-à-temps d’une machine virtuelle en ajoutant et en configurant un nouveau port à protéger pour cette machine virtuelle, ou en modifiant tout autre paramètre relatif à un port déjà protégé.

Pour modifier les règles JAT existantes pour une machine virtuelle :

1. Ouvrez le tableau de bord Azure Defender et, dans la zone de protection avancée, sélectionnez **Contrôles d’application adaptatifs**.

1. Dans l’onglet **Configuré**, cliquez avec le bouton droit sur la machine virtuelle à laquelle vous souhaitez ajouter un port, puis sélectionnez Modifier. 

    ![Modification d’une configuration d’accès JAT à la machine virtuelle dans Azure Security Center](./media/security-center-just-in-time/jit-policy-edit-security-center.png)

1. Sous **JIT VM access configuration** (Configuration de l’accès juste-à-temps à la machine virtuelle), vous pouvez soit modifier les paramètres existants d’un port déjà protégé, soit ajouter un nouveau port personnalisé.

1. Lorsque vous avez fini de modifier les ports, sélectionnez **Enregistrer**.
 


### <a name="azure-virtual-machines"></a>[**Machines virtuelles Azure**](#tab/jit-config-avm)

### <a name="enable-jit-on-your-vms-from-azure-virtual-machines"></a>Activer l’accès JAT sur vos machines virtuelles à partir des machines virtuelles Azure

Vous pouvez activer l’accès JAT sur une machine virtuelle à partir des pages des machines virtuelles Azure sur le portail Azure.

![Configuration d’un accès JAT aux machines virtuelles dans les machines virtuelles Azure](./media/security-center-just-in-time/jit-config-virtual-machines.gif)

> [!TIP]
> Si l’accès juste-à-temps est déjà activé sur une machine virtuelle, vous pouvez le voir quand vous accédez à sa page de configuration, et vous pouvez utiliser le lien pour ouvrir la page Accès JAT aux machines virtuelles dans Security Center afin d’afficher et de modifier les paramètres.

1. Dans le **portail Azure**, recherchez et sélectionnez [Machines virtuelles](https://ms.portal.azure.com). 

1. Sélectionnez la machine virtuelle que vous souhaitez protéger à l’aide de l’accès JAT.

1. Dans le menu, sélectionnez **Configuration**.

1. Sous **Accès juste-à-temps**, sélectionnez **Activer l’accès juste-à-temps**. 

    Cela permet un accès juste-à-temps pour la machine virtuelle en utilisant les paramètres par défaut suivants :

    - Machines Windows :
        - Port RDP 3389
        - Trois heures d’accès autorisé maximum
        - L’option Adresses IP sources autorisées est définie sur Toutes
    - Machines Linux :
        - Port SSH 22
        - Trois heures d’accès autorisé maximum
        - L’option Adresses IP sources autorisées est définie sur Toutes

1. Pour modifier l’une de ces valeurs ou ajouter d’autres ports à votre configuration JAT, utilisez la page Juste-à-temps d’Azure Security Center :

    1. Dans le menu de Security Center, sélectionnez **Accès JAT aux machines virtuelles**.

    1. Dans l’onglet **Configuré**, cliquez avec le bouton droit sur la machine virtuelle à laquelle vous souhaitez ajouter un port, puis sélectionnez Modifier. 

        ![Modification d’une configuration d’accès JAT à la machine virtuelle dans Azure Security Center](./media/security-center-just-in-time/jit-policy-edit-security-center.png)

    1. Sous **JIT VM access configuration** (Configuration de l’accès juste-à-temps à la machine virtuelle), vous pouvez soit modifier les paramètres existants d’un port déjà protégé, soit ajouter un nouveau port personnalisé.

    1. Lorsque vous avez fini de modifier les ports, sélectionnez **Enregistrer**.


### <a name="powershell"></a>[**PowerShell**](#tab/jit-config-powershell)

### <a name="enable-jit-on-your-vms-using-powershell"></a>Activer l’accès JAT sur vos machines virtuelles à l’aide de PowerShell

Pour activer l’accès juste-à-temps aux machines virtuelles à partir de PowerShell, utilisez la cmdlet officielle d’Azure Security Center PowerShell `Set-AzJitNetworkAccessPolicy`.

**Exemple** : activer l’accès juste-à-temps sur une machine virtuelle spécifique avec les règles suivantes :

* Fermez les ports 22 et 3389.
* Définissez une fenêtre de temps maximale de trois heures pour chaque afin qu’ils puissent être ouverts pour chaque demande approuvée.
* Autorisez l’utilisateur qui demande l’accès à contrôler les adresses IP sources.
* Autorisez l’utilisateur qui demande l’accès à établir une session après une demande d’accès juste-à-temps approuvée.

Les commandes PowerShell suivantes créent cette configuration JAT :

1. Attribuez une variable qui conserve les règles d’accès juste-à-temps pour une machine virtuelle :

    ```azurepowershell
    $JitPolicy = (@{
        id="/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Compute/virtualMachines/VMNAME";
        ports=(@{
             number=22;
             protocol="*";
             allowedSourceAddressPrefix=@("*");
             maxRequestAccessDuration="PT3H"},
             @{
             number=3389;
             protocol="*";
             allowedSourceAddressPrefix=@("*");
             maxRequestAccessDuration="PT3H"})})
    ```

1. Insérez les règles d’accès juste-à-temps d’une machine virtuelle dans un tableau :
    
    ```azurepowershell
    $JitPolicyArr=@($JitPolicy)
    ```

1. Configurez les règles d’accès juste-à-temps de la machine virtuelle sélectionnée :
    
    ```azurepowershell
    Set-AzJitNetworkAccessPolicy -Kind "Basic" -Location "LOCATION" -Name "default" -ResourceGroupName "RESOURCEGROUP" -VirtualMachine $JitPolicyArr
    ```

    Utilisez le paramètre -Name pour spécifier une machine virtuelle. Par exemple, pour établir la configuration JAT pour deux machines virtuelles différentes, VM1 et VM2, utilisez : ```Set-AzJitNetworkAccessPolicy -Name VM1``` et ```Set-AzJitNetworkAccessPolicy -Name VM2```.


### <a name="rest-api"></a>[**API REST**](#tab/jit-config-api)

### <a name="enable-jit-on-your-vms-using-the-rest-api"></a>Activer l’accès JAT sur vos machines virtuelles à l’aide de l’API REST

La fonctionnalité d’accès aux machines virtuelles juste-à-temps peut être utilisée via l’API Azure Security Center. Utilisez cette API pour obtenir des informations sur les machines virtuelles configurées, en ajouter de nouvelles, demander l’accès à une machine virtuelle, et bien plus encore. 

Pour en savoir plus, consultez [Stratégies d’accès réseau JAT](/rest/api/securitycenter/jitnetworkaccesspolicies).


--- 










## <a name="request-access-to-a-jit-enabled-vm"></a>Demander l’accès à une machine virtuelle prenant en charge l’accès JAT

Vous pouvez demander l’accès à une machine virtuelle prenant en charge l’accès JAT à partir du portail Azure (dans Security Center ou les machines virtuelles Azure) ou par programmation.

Chacune de ces options est expliquée dans un onglet séparé ci-dessous.

### <a name="azure-security-center"></a>[**Azure Security Center**](#tab/jit-request-asc)

### <a name="request-access-to-a-jit-enabled-vm-from-azure-security-center"></a>Demander l’accès à une machine virtuelle prenant en charge l’accès JAT à partir d’Azure Security Center 

Quand l’accès JAT est activé pour une machine virtuelle, vous devez demander l’accès pour vous y connecter. Vous pouvez demander l’accès selon l’une des méthodes prises en charge, quelle que soit la façon dont vous avez activé l’accès JAT.

:::image type="content" source="./media/security-center-just-in-time/jit-request-security-center.gif" alt-text="Demande d’accès JAT à partir de Security Center":::

1. Dans la page **Accès JAT aux machines virtuelles**, sélectionnez l’onglet **Configuré**.

1. Marquez les machines virtuelles auxquelles vous souhaitez accéder.

    - L’icône dans la colonne **Détails de la connexion** indique si l’accès juste-à-temps est activé sur le groupe de sécurité réseau ou le pare-feu. S’il est activé sur les deux, seule l’icône de pare-feu s’affiche.

    - La colonne **Détails de la connexion** fournit les informations nécessaires pour se connecter à la machine virtuelle et à ses ports ouverts.

1. Sélectionnez **Demander l’accès**. La fenêtre **Demander l’accès** s’ouvre.

1. Sous **Demander l’accès**, configurez pour chaque machine virtuelle les ports que vous voulez ouvrir, les adresses IP sources sur lesquelles le port est ouvert, ainsi que la fenêtre de temps pendant laquelle le port est ouvert. Il est uniquement possible de demander l’accès aux ports configurés. Chaque port dispose d’un délai maximal autorisé issu de la configuration juste-à-temps que vous avez créée.

1. Sélectionnez **Ports ouverts**.

> [!NOTE]
> Si un utilisateur demande l’accès alors qu’il se trouve derrière un proxy, l’option **Mon IP** risque de ne pas fonctionner. Il peut se révéler nécessaire de définir la plage complète d’adresses IP de l’organisation.



### <a name="azure-virtual-machines"></a>[**Machines virtuelles Azure**](#tab/jit-request-avm)

### <a name="request-access-to-a-jit-enabled-vm-from-the-azure-virtual-machines-connect-page"></a>Demander l’accès à une machine virtuelle prenant en charge l’accès JAT à partir de la page de connexion de la machine virtuelle Azure

Quand l’accès JAT est activé pour une machine virtuelle, vous devez demander l’accès pour vous y connecter. Vous pouvez demander l’accès selon l’une des méthodes prises en charge, quelle que soit la façon dont vous avez activé l’accès JAT.

  >![demande juste-à-temps JAT](./media/security-center-just-in-time/jit-request-vm.png)


Pour demander l’accès à partir de machines virtuelles Azure :

1. Dans le portail Azure, ouvrez la page des machines virtuelles.

1. Sélectionnez la machine virtuelle à laquelle vous souhaitez vous connecter, puis ouvrez la page **Se connecter**.

    Azure vérifie si l’accès JAT est activé sur cette machine virtuelle.

    - S’il n’est pas activé pour la machine virtuelle, vous êtes invité à l’activer.

    - S’il est activé, sélectionnez **Demander l’accès** pour transmettre une demande d’accès avec l’adresse IP, la plage de temps et les ports de la requête qui ont été configurés pour cette machine virtuelle.

> [!NOTE]
> Une fois la demande approuvée pour une machine virtuelle protégée par le pare-feu Azure, Security Center fournit à l’utilisateur les informations de connexion appropriées (mappage de ports provenant de la table DNAT) à utiliser pour se connecter à la machine virtuelle.



### <a name="powershell"></a>[**PowerShell**](#tab/jit-request-powershell)

### <a name="request-access-to-a-jit-enabled-vm-using-powershell"></a>Demander l’accès à une machine virtuelle prenant en charge l’accès JAT à l’aide de PowerShell

Dans l’exemple suivant, vous pouvez voir une demande d’accès juste-à-temps à une machine virtuelle pour une machine virtuelle spécifique. Dans cette dernière, l’ouverture du port 22 est requise pour une adresse IP et une durée spécifiques :

Exécutez la commande suivante dans PowerShell :

1. Configurez les propriétés de la demande d’accès à une machine virtuelle :

    ```azurepowershell
    $JitPolicyVm1 = (@{
        id="/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Compute/virtualMachines/VMNAME";
        ports=(@{
           number=22;
           endTimeUtc="2020-07-15T17:00:00.3658798Z";
           allowedSourceAddressPrefix=@("IPV4ADDRESS")})})
    ```

1. Insérez les paramètres de la demande d’accès à la machine virtuelle dans un tableau :

    ```azurepowershell
    $JitPolicyArr=@($JitPolicyVm1)
    ```
        
1. Envoyez la demande d’accès (utilisez l’ID de ressource obtenu à l’étape 1) :

    ```azurepowershell
    Start-AzJitNetworkAccessPolicy -ResourceId "/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Security/locations/LOCATION/jitNetworkAccessPolicies/default" -VirtualMachine $JitPolicyArr
    ```

Pour en savoir plus, consultez la [documentation relative aux cmdlets PowerShell](/powershell/scripting/developer/cmdlet/cmdlet-overview).



### <a name="rest-api"></a>[**API REST**](#tab/jit-request-api)

### <a name="request-access-to-a-jit-enabled-vms-using-the-rest-api"></a>Demander l’accès à une machine virtuelle prenant en charge l’accès JAT à l’aide de l’API REST

La fonctionnalité d’accès aux machines virtuelles juste-à-temps peut être utilisée via l’API Azure Security Center. Utilisez cette API pour obtenir des informations sur les machines virtuelles configurées, en ajouter de nouvelles, demander l’accès à une machine virtuelle, et bien plus encore. 

Pour en savoir plus, consultez [Stratégies d’accès réseau JAT](/rest/api/securitycenter/jitnetworkaccesspolicies).

---








## <a name="audit-jit-access-activity-in-security-center"></a>Auditer l’activité d’accès JIT dans Security Center

Vous pouvez obtenir des informations sur les activités des machines virtuelles à l’aide de la recherche dans les journaux. Comment afficher les journaux d’activité :

1. Dans **Accès JAT aux machines virtuelles**, sélectionnez l’onglet **Configuré**.

1. Pour la machine virtuelle que vous souhaitez auditer, ouvrez le menu de sélection au bout de la ligne.
 
1. Sélectionnez **Journal d’activité** dans le menu.

   ![Sélectionner le journal d’activité JAT juste-à-temps](./media/security-center-just-in-time/jit-select-activity-log.png)

   Le journal d’activité fournit une vue filtrée des opérations précédentes pour cette machine virtuelle, ainsi que l’abonnement, la date et l’heure.

1. Pour télécharger les informations du journal, sélectionnez **Télécharger au format CSV**.








## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à configurer et à utiliser l’accès juste-à-temps aux machines virtuelles. Pour en savoir plus sur les raisons pour lesquelles l’accès JAT doit être utilisé, consultez l’article conceptuel expliquant les menaces contre lesquelles il défend les machines virtuelles :

> [!div class="nextstepaction"]
> [Explication de JAT](just-in-time-explained.md)