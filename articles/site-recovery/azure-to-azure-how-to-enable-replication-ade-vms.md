---
title: Activer la réplication pour des machines virtuelles Azure chiffrées dans Azure Site Recovery
description: Cet article décrit comment configurer la réplication pour les machines virtuelles prenant en charge Azure Disk Encryption, d’une région Azure à l’autre à l’aide de Site Recovery.
author: asgang
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 08/08/2019
ms.author: sutalasi
ms.openlocfilehash: fa4d61599e102f9a2580e704ee7a02486067daa2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86135787"
---
# <a name="replicate-azure-disk-encryption-enabled-virtual-machines-to-another-azure-region"></a>Répliquer des machines virtuelles prenant en charge Azure Disk Encryption vers une autre région Azure

Cet article décrit comment répliquer des machines virtuelles Azure avec Azure Disk Encryption (ADE) activé, d’une région Azure vers une autre.

>[!NOTE]
> Site Recovery prend actuellement en charge ADE, avec et sans Azure Active Directory (AAD) pour les machines virtuelles sur Windows. Dans le cas des systèmes d’exploitation Linux, nous ne gérons ADE que sans AAD. Par ailleurs, les machines virtuelles exécutant ADE 1.1 (sans AAD) doivent utiliser des disques managés. Les machines virtuelles avec des disques non managés ne sont pas prises en charge. Si vous passez de ADE 0.1 (avec AAD) à 1.1, vous devez désactiver la réplication et activer la réplication pour une machine virtuelle après avoir activé 1.1.


## <a name="required-user-permissions"></a><a id="required-user-permissions"></a> Autorisations utilisateur requises
Site Recovery a besoin que l’utilisateur dispose d’une autorisation de création du coffre de clés dans la région cible et d’une autorisation de copie des clés du coffre de clés de la région source dans le coffre de la région cible.

Pour activer la réplication de machines virtuelles prenant en charge Azure Disk Encryption à partir du portail Azure, l’utilisateur a besoin des autorisations sur les coffres de clés de la **région source et de la région cible**.

- Autorisations d’accès au coffre de clés
    - Lister, créer et obtenir
    
- Autorisations d’accès au secret du coffre de clés
    - Opérations de gestion des secrets
        - Obtenir, lister et définir
    
- Autorisations de clé du coffre de clés (obligatoires uniquement si les machines virtuelles utilisent la clé de chiffrement à clé pour chiffrer les clés de chiffrement de disque)
    - Opérations de gestion des clés
        - Obtenir, lister et créer
    - Opérations de chiffrement
        - Déchiffrer et chiffrer

Pour gérer les autorisations, accédez à la ressource du coffre de clés dans le portail. Ajoutez les autorisations nécessaires pour l’utilisateur. L’exemple suivant montre comment activer des autorisations pour le coffre de clés *ContosoWeb2Keyvault*, qui se trouve dans la région source.

1. Accédez à **Accueil** > **Keyvaults** > **ContosoWeb2KeyVault > Stratégies d’accès**.

   ![Fenêtre des autorisations d’accès au coffre de clés](./media/azure-to-azure-how-to-enable-replication-ade-vms/key-vault-permission-1.png)

2. Vous pouvez voir qu’il n’y a aucune autorisation d’utilisateur. Sélectionnez **Ajouter**. Entrez les informations sur l’utilisateur et les autorisations.

   ![Autorisation d’accès au coffre de clés](./media/azure-to-azure-how-to-enable-replication-ade-vms/key-vault-permission-2.png)

Si l’utilisateur qui active la reprise d’activité ne dispose pas des autorisations nécessaires pour copier les clés, un administrateur de la sécurité qui dispose des autorisations appropriées peut utiliser le script suivant pour copier les secrets et clés de chiffrement dans la région cible.

Pour résoudre les problèmes liés aux autorisations, reportez-vous aux [problèmes liés aux autorisations d’accès au coffre de clés](#trusted-root-certificates-error-code-151066) plus loin dans cet article.

>[!NOTE]
>Pour activer la réplication de machines virtuelles prenant en charge Azure Disk Encryption à partir du portail, vous avez au minimum besoin des autorisations « Lister » sur les coffres de clés, les secrets et les clés.

## <a name="copy-disk-encryption-keys-to-the-dr-region-by-using-the-powershell-script"></a>Copier les clés Azure Disk Encryption dans la région de reprise d’activité en utilisant le script PowerShell

1. [Ouvrez le code de script brut « CopyKeys »](https://aka.ms/ade-asr-copy-keys-code).
2. Copiez le script dans un fichier que vous nommez **Copy-keys.ps1**.
3. Ouvrez l’application Windows PowerShell, puis accédez au dossier dans lequel vous avez enregistré le fichier.
4. Exécutez Copy-keys.ps1.
5. Indiquez des informations d’identification Azure pour vous connecter.
6. Sélectionnez l’**abonnement Azure** de vos machines virtuelles.
7. Attendez le chargement des groupes de ressources, puis sélectionnez le **groupe de ressources** de vos machines virtuelles.
8. Sélectionnez les machines virtuelles dans la liste qui s’affiche. Seules les machines virtuelles prenant en charge le chiffrement de disque figurent dans la liste.
9. Sélectionnez l’**emplacement cible**.

    - **Coffres de clés de chiffrement de disque**
    - **Coffres de clés de chiffrement de clé**

   Par défaut, Site Recovery crée un coffre de clés dans la région cible. Le nom du coffre porte un suffixe « asr » qui se base sur les clés de chiffrement de disque de machine virtuelle source. S’il existe déjà un coffre de clés créé par Site Recovery, il est réutilisé. Sélectionnez un autre coffre de clés dans la liste si nécessaire.

## <a name="enable-replication"></a>Activer la réplication

Pour cet exemple, la région principale Azure est Asie Est et la région secondaire Asie Sud-Est.

1. Dans le coffre, sélectionnez **+Répliquer**.
2. Notez les champs suivants.
    - **Source** : point d’origine des machines virtuelles, qui dans ce cas est **Azure**.
    - **Emplacement source** : région Azure où vous souhaitez protéger vos machines virtuelles. Dans cet exemple, l’emplacement source est « Asie Est ».
    - **Modèle de déploiement** : modèle de déploiement Azure des machines sources.
    - **Abonnement source** : abonnement auquel vos machines virtuelles sources appartiennent. Il peut s’agir de n’importe quel abonnement au sein du même locataire Azure Active Directory où se trouve votre coffre Recovery services.
    - **Groupe de ressources** : groupe de ressources auquel appartiennent vos machines virtuelles sources. Toutes les machines virtuelles du groupe de ressources sélectionné sont listées pour la protection à l’étape suivante.

3. Dans **Machines virtuelles** > **Sélectionner les machines virtuelles**, sélectionnez chaque machine à répliquer. Vous pouvez uniquement sélectionner les machines pour lesquelles la réplication peut être activée. Ensuite, sélectionnez **OK**.

4. Dans **Paramètres**, vous pouvez configurer les paramètres de site/cible suivants.

    - **Emplacement cible** : Emplacement où vos données de machines virtuelles sources sont répliquées. Site Recovery fournit la liste des régions cibles appropriées selon l’emplacement de la machine sélectionnée. Nous vous recommandons d’utiliser le même emplacement que celui du coffre Recovery Services.
    - **Abonnement cible** : abonnement cible utilisé pour la reprise d’activité. Par défaut, l’abonnement cible est identique à l’abonnement source.
    - **Groupe de ressources cible** : groupe de ressources auquel appartiennent toutes vos machines virtuelles répliquées. Par défaut, Site Recovery crée un groupe de ressources dans la région cible. Son nom porte un suffixe « asr ». S’il existe déjà un groupe de ressources créé par Azure Site Recovery, il est réutilisé. Vous pouvez également choisir de le personnaliser, comme indiqué dans la section suivante. L’emplacement du groupe de ressources cible peut être n’importe quelle région Azure à l’exception de la région dans laquelle les machines virtuelles sources sont hébergées.
    - **Réseau virtuel cible** : Par défaut, Site Recovery crée un réseau virtuel dans la région cible. Son nom porte un suffixe « asr ». Il est mappé à votre réseau source et utilisé pour toute protection ultérieure. [En savoir plus](./azure-to-azure-network-mapping.md) sur le mappage réseau.
    - **Comptes de stockage cibles (si votre machine virtuelle source n’utilise pas de disques managés)**  : par défaut, Site Recovery crée un compte de stockage cible avec la même configuration que celle du compte de stockage de machines virtuelles source. S’il existe déjà un compte de stockage, il est réutilisé.
    - **Disques managés de réplica (si votre machine virtuelle source utilise des disques managés)**  : Site Recovery crée des disques managés de réplica dans la région cible pour refléter les disques managés de la machine virtuelle source du même type de stockage (Standard ou Premium).
    - **Comptes de stockage de cache** : Site Recovery a besoin d’un compte de stockage supplémentaire appelé *stockage de cache* dans la région source. Toutes les modifications apportées sur les machines virtuelles sources sont suivies et envoyées au compte de stockage de cache. Elles sont ensuite répliquées sur l’emplacement cible.
    - **Groupe à haute disponibilité** : Par défaut, Site Recovery crée un groupe à haute disponibilité dans la région cible. Son nom porte un suffixe « asr ». S’il existe déjà un groupe à haute disponibilité créé par Site Recovery, il est réutilisé.
    - **Coffres de clés de chiffrement de disque** : Par défaut, Site Recovery crée un coffre de clés dans la région cible. Son nom porte un suffixe « asr » qui se base sur les clés de chiffrement de disque de machine virtuelle source. S’il existe déjà un coffre de clés créé par Azure Site Recovery, il est réutilisé.
    - **Coffres de clés de chiffrement de clé** : Par défaut, Site Recovery crée un coffre de clés dans la région cible. Son nom porte un suffixe « asr » basé sur les clés de chiffrement à clé de machine virtuelle source. S’il existe déjà un coffre de clés créé par Azure Site Recovery, il est réutilisé.
    - **Stratégie de réplication** : définit les paramètres de l’historique de conservation des points de récupération et la fréquence des instantanés de cohérence d’application. Par défaut, Site Recovery crée une stratégie de réplication avec des paramètres par défaut de *24 heures* pour la rétention des points de récupération et *60 minutes* pour la fréquence des instantanés de cohérence d’application.

## <a name="customize-target-resources"></a>Personnaliser les ressources cibles

Suivez ces étapes pour modifier les paramètres cibles par défaut de Site Recovery.

1. Sélectionnez **Personnaliser** en regard d’« Abonnement cible » pour modifier l’abonnement cible par défaut. Sélectionnez l’abonnement dans la liste des abonnements disponibles dans le locataire Azure AD.

2. Sélectionnez **Personnaliser** en regard de « Groupe de ressources, Réseau, Stockage et Groupes à haute disponibilité » pour modifier les paramètres par défaut suivants :
    - Pour **Groupe de ressources cible**, sélectionnez le groupe de ressources dans la liste des groupes de ressources à l’emplacement cible de l’abonnement.
    - Pour **Réseau virtuel cible**, sélectionnez le réseau dans la liste des réseaux virtuels à l’emplacement cible.
    - Pour **Groupe à haute disponibilité**, vous pouvez ajouter les paramètres de groupe à haute disponibilité de la machine virtuelle, s’ils font partie d’un groupe à haute disponibilité dans la zone source.
    - Pour **Comptes de stockage cible**, sélectionnez le compte à utiliser.

2. Sélectionnez **Personnaliser** en regard de « Paramètres de chiffrement » pour modifier les paramètres par défaut suivants :
   - Pour **Coffre de clés pour le chiffrement de disque cible**, sélectionnez le coffre de clés pour le chiffrement de disque cible dans la liste des coffres de clés figurant à l’emplacement cible de l’abonnement.
   - Pour **Coffre de clés pour le chiffrement de clé cible**, sélectionnez le coffre de clés pour le chiffrement de clé cible dans la liste des coffres de clés figurant à l’emplacement cible de l’abonnement.

3. Sélectionnez **Créer une ressource cible** > **Activer la réplication**.
4. Une fois que les machines virtuelles sont activées pour la réplication, vous pouvez vérifier leur état d’intégrité sous **Éléments répliqués**.

>[!NOTE]
>Pendant la réplication initiale, l’état peut prendre un certain temps à s’actualiser et sa progression peut ne pas apparaître. Cliquez sur **Actualiser** pour obtenir l’état le plus récent.

## <a name="update-target-vm-encryption-settings"></a>Mettre à jour les paramètres de chiffrement de machine virtuelle cible
Dans les scénarios suivants, vous devez mettre à jour les paramètres de chiffrement de machine virtuelle cible :
  - Vous avez activé la réplication Site Recovery sur la machine virtuelle. Plus tard, vous avez activé le chiffrement de disque sur la machine virtuelle source.
  - Vous avez activé la réplication Site Recovery sur la machine virtuelle. Plus tard, vous avez modifié la clé de chiffrement de disque ou une clé de chiffrement à clé sur la machine virtuelle source.

Vous pouvez utiliser [un script](#copy-disk-encryption-keys-to-the-dr-region-by-using-the-powershell-script) pour copier les clés de chiffrement dans la région cible, puis mettre à jour les paramètres de chiffrement cible dans **Coffre Recovery Services** > *Élémént répliqué* > **Propriétés** > **Calcul et Réseau**.

![Boîte de dialogue Mettre à jour les paramètres ADE](./media/azure-to-azure-how-to-enable-replication-ade-vms/update-ade-settings.png)

## <a name="troubleshoot-key-vault-permission-issues-during--azure-to-azure-vm-replication"></a><a id="trusted-root-certificates-error-code-151066"></a>Résoudre les problèmes d’autorisation du coffre de clés lors de la réplication d’une machine virtuelle Azure vers Azure

Pour pouvoir lire le secret et le copier dans le coffre de clés de la région cible, Azure Site Recovery requiert au minimum des autorisations de lecture dans le coffre de clés de la région source, ainsi que des autorisations de lecture dans le coffre de clés de la région cible. 

**Cause 1 :** Vous ne disposez pas d’autorisations de type GET sur le **coffre de clés de la région source** pour la lecture des clés. </br>
**Procédure de résolution :** Que vous soyez un administrateur de l’abonnement ou non, il est important de bénéficier d’une autorisation GET sur le coffre de clés.

1. Accédez au coffre de clés de la région source ; dans cet exemple, il s’agit de ContososourceKeyvault > **Stratégies d’accès**. 
2. Sous **Sélectionner le principal**, ajoutez votre nom d’utilisation (par exemple, « dradmin@contoso.com »).
3. Sous **Autorisations de clé**, sélectionnez GET. 
4. Sous **Autorisations de clé**, sélectionnez GET. 
5. Enregistrez la stratégie d’accès.

**Cause 2 :** Vous ne disposez pas de l’autorisation requise sur le **coffre de clés de la région cible** pour l’écriture des clés. </br>

*Par exemple* : Vous essayez de répliquer une machine virtuelle qui a le coffre de clés *ContososourceKeyvault* sur une région source.
Vous avez toutes les autorisations sur le coffre de clés de la région source. Mais pendant la protection, vous sélectionnez le coffre de clés déjà créé ContosotargetKeyvault, qui n’a pas les autorisations. Une erreur se produit.

Autorisation requise sur le [coffre de clés cible](#required-user-permissions)

**Procédure de résolution :** Accédez à **Accueil** > **Keyvaults** > **ContosotargetKeyvault** > **Stratégies d’accès** et ajoutez les autorisations appropriées.

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus](site-recovery-test-failover-to-azure.md) sur l’exécution d’un test de basculement.
