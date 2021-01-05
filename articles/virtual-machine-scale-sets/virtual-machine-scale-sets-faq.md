---
title: FAQ sur les groupes de machines virtuelles identiques Azure
description: Obtenez des réponses aux questions les plus fréquemment posées sur les groupes de machines virtuelles identiques dans Azure.
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: faq
ms.date: 06/30/2020
ms.reviewer: jushiman
ms.custom: mimckitt
ms.openlocfilehash: 8170cfcbbf200c6ba5030aff5716f46b537d8c97
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016707"
---
# <a name="azure-virtual-machine-scale-sets-faqs"></a>FAQ sur les groupes de machines virtuelles identiques Azure

Obtenez des réponses aux questions fréquemment posées sur les groupes de machines virtuelles identiques dans Azure.

## <a name="top-frequently-asked-questions-for-scale-sets"></a>Questions les plus fréquemment posées sur les groupes identiques

### <a name="how-many-vms-can-i-have-in-a-scale-set"></a>Combien de machines virtuelles peut-il y avoir dans un groupe identique ?

Un groupe identique peut contenir de 0 à 1 000 machines virtuelles basées sur des images de plateforme, ou de 0 à 600 machines virtuelles basées sur des images personnalisées.

### <a name="are-data-disks-supported-within-scale-sets"></a>Les disques de données sont-ils pris en charge dans les groupes identiques ?

Oui. Un groupe identique peut définir une configuration de disques de données attachés qui s’appliquera à toutes les machines virtuelles du groupe. Pour plus d’informations, consultez [Groupes identiques Azure et disques de données attachés](virtual-machine-scale-sets-attached-disks.md). Il existe d’autres options pour stocker les données :

* Fichiers Azure (lecteurs SMB partagés)
* Système d’exploitation de lecteur
* Lecteur temp (local, non sauvegardé par le stockage Azure)
* Service de données Azure (par exemple, tables Azure, objets blob Azure)
* Service de données externe (par exemple, base de données distante)

### <a name="which-azure-regions-support-scale-sets"></a>Quelles sont les régions Azure qui prennent en charge les groupes identiques ?

Toutes les régions prennent en charge les groupes identiques.

### <a name="how-do-i-create-a-scale-set-by-using-a-custom-image"></a>Comment créer un groupe identique à l’aide d’une image personnalisée ?

Créez et capturez une image de machine virtuelle, puis utilisez-la comme source de votre groupe identique. Pour un tutoriel sur la création et l’utilisation d’une image de machine virtuelle personnalisée, vous pouvez utiliser [Azure CLI](tutorial-use-custom-image-cli.md) ou [Azure PowerShell](tutorial-use-custom-image-powershell.md).

### <a name="if-i-reduce-my-scale-set-capacity-from-20-to-15-which-vms-are-removed"></a>Si je réduis ma capacité de groupe identique de 20 à 15, quelles sont les machines virtuelles qui seront supprimées ?

Par défaut, les machines virtuelles sont retirées du groupe identique de façon uniforme entre les zones de disponibilité (si le groupe identique est déployé dans une configuration zonale) et les domaines d’erreur pour optimiser la disponibilité. Les machines virtuelles avec les ID les plus élevés sont supprimées en premier.

Vous pouvez modifier l’ordre de suppression des machines virtuelles en spécifiant une [Stratégie de scale-in](virtual-machine-scale-sets-scale-in-policy.md) pour le groupe identique.

### <a name="what-if-i-then-increase-the-capacity-from-15-to-18"></a>Que se passe-t-il si j’augmente ensuite la capacité de 15 à 18 ?

Si vous augmentez la capacité à 18, 3 machines virtuelles sont créées. À chaque fois, l’ID d’instance de la machine virtuelle est incrémenté avec la valeur la plus élevée précédente (par exemple, 20, 21, 22). Les machines virtuelles sont équilibrées entre les domaines d’erreur.

### <a name="when-im-using-multiple-extensions-in-a-scale-set-can-i-enforce-an-execution-sequence"></a>Lorsque j’utilise plusieurs extensions dans un groupe identique, puis-je appliquer une séquence d’exécution ?

Oui, vous pouvez appliquer une [séquence d'exécution](virtual-machine-scale-sets-extension-sequencing.md) au groupe identique.

### <a name="do-scale-sets-work-with-azure-availability-sets"></a>Les groupes identiques fonctionnent-ils avec des ensembles haute disponibilité Azure ?

Un groupe identique régional (non zonal) utilise des *groupes de placement*, qui servent de groupe à haute disponibilité implicite avec cinq domaines d’erreur et cinq domaines de mise à jour. Les groupes identiques de plus de 100 machines virtuelles sont répartis entre plusieurs groupes de placement. Pour plus d’informations sur les groupes de placement, voir [Working with large virtual machine scale sets](virtual-machine-scale-sets-placement-groups.md) (Utilisation de grands groupes de machines virtuelles identiques). Un groupe de machines virtuelles à haute disponibilité peut figurer dans le même réseau virtuel qu’un groupe identique de machines virtuelles. Une configuration courante consiste à placer les machines virtuelles du nœud de contrôle qui nécessitent souvent une configuration unique dans un groupe à haute disponibilité, et les nœuds de données dans le groupe identique.

### <a name="do-scale-sets-work-with-azure-availability-zones"></a>Les groupes identiques fonctionnent-ils avec des zones de disponibilité Azure ?

Oui. Pour plus d’informations, consultez le [document relatif aux zones des groupes identiques](./virtual-machine-scale-sets-use-availability-zones.md).


## <a name="autoscale"></a>Mise à l’échelle automatique

### <a name="what-are-best-practices-for-azure-autoscale"></a>Quelles sont les meilleures pratiques pour la mise à l’échelle automatique d’Azure ?

Pour connaître les meilleures pratiques pour la mise à l’échelle automatique, consultez [Meilleures pratiques pour la mise à l’échelle automatique des machines virtuelles](../azure-monitor/platform/autoscale-best-practices.md).

### <a name="where-do-i-find-metric-names-for-autoscaling-that-uses-host-based-metrics"></a>Où trouver les noms des métriques pour la mise à l’échelle automatique utilisant des métriques basées sur les hôtes ?

Pour obtenir les noms des métriques pour la mise à l’échelle automatique utilisant des métriques basées sur les hôtes, consultez [Métriques prises en charge avec Azure Monitor](../azure-monitor/platform/metrics-supported.md).

### <a name="are-there-any-examples-of-autoscaling-based-on-an-azure-service-bus-topic-and-queue-length"></a>Existe-t-il des exemples de mise à l’échelle automatique basée sur une rubrique Azure Service Bus et une longueur de file d’attente ?

Oui. Pour obtenir des exemples de mise à l’échelle automatique basée sur une rubrique Azure Service Bus et une longueur de file d’attente, consultez [Métriques courantes pour la mise à l’échelle automatique d’Azure Monitor](../azure-monitor/platform/autoscale-common-metrics.md).

Pour une file d’attente Service Bus, utilisez le JSON suivant :

```json
"metricName": "MessageCount",
"metricNamespace": "",
"metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ServiceBus/namespaces/mySB/queues/myqueue"
```

Pour une file d’attente de stockage, utilisez le JSON suivant :

```json
"metricName": "ApproximateMessageCount",
"metricNamespace": "",
"metricResourceUri": "/subscriptions/s1/resourceGroups/rg1/providers/Microsoft.ClassicStorage/storageAccounts/mystorage/services/queue/queues/mystoragequeue"
```

Remplacez les exemples de valeurs par les URI (Uniform Resource Identifiers) de votre ressource.


### <a name="should-i-autoscale-by-using-host-based-metrics-or-a-diagnostics-extension"></a>Dois-je utiliser la mise à l’échelle automatique avec des métriques basées sur les hôtes ou une extension de diagnostics ?

Vous pouvez créer un paramètre de mise à l’échelle automatique sur une machine virtuelle pour utiliser les métriques au niveau de l’hôte ou les métriques basées sur le système d’exploitation invité.

Pour obtenir la liste des métriques prises en charge, consultez [Métriques courantes pour la mise à l’échelle automatique d’Azure Monitor](../azure-monitor/platform/autoscale-common-metrics.md).

Pour obtenir un exemple complet pour les groupes de machines virtuelles identiques, consultez [Configuration avancée de la mise à l’échelle automatique à l’aide de modèles Resource Manager pour les groupes de machines virtuelles identiques](../azure-monitor/platform/autoscale-virtual-machine-scale-sets.md).

L’exemple utilise la métrique du processeur au niveau de l’hôte et une métrique de comptage de messages.



### <a name="how-do-i-set-alert-rules-on-a-virtual-machine-scale-set"></a>Comment définir des règles d’alerte sur un groupe de machines virtuelles identiques ?

Vous pouvez créer des alertes sur des métriques pour les groupes de machines virtuelles identiques via PowerShell ou l’interface CLI Azure. Pour plus d’informations, consultez [Exemples de démarrage rapide Azure Monitor PowerShell](../azure-monitor/samples/powershell-samples.md#create-metric-alerts) et [Exemples de démarrage rapide de l’interface CLI multiplateforme pour Azure Monitor](../azure-monitor/samples/cli-samples.md#work-with-alerts).

La valeur TargetResourceId du groupe de machines virtuelles identiques ressemble à ceci :

/subscriptions/yoursubscriptionid/resourceGroups/yourresourcegroup/providers/Microsoft.Compute/virtualMachineScaleSets/yourvmssname

Vous pouvez choisir n’importe quel compteur de performances de machine virtuelle en tant que métrique sur laquelle définir une alerte. Pour plus d’informations, consultez [Métriques du système d’exploitation invité pour les machines virtuelles Windows Resource Manager](../azure-monitor/platform/autoscale-common-metrics.md#guest-os-metrics-for-resource-manager-based-windows-vms) et [Métriques du système d’exploitation invité pour les machines virtuelles Linux](../azure-monitor/platform/autoscale-common-metrics.md#guest-os-metrics-linux-vms) dans l’article[Métriques courantes pour la mise à l’échelle automatique d’Azure Monitor](../azure-monitor/platform/autoscale-common-metrics.md).

### <a name="how-do-i-set-up-autoscale-on-a-virtual-machine-scale-set-by-using-powershell"></a>Comment configurer la mise à l’échelle automatique sur un groupe de machines virtuelles identiques à l’aide de PowerShell ?

Pour configurer la mise à l’échelle automatique sur un groupe de machines virtuelles identiques à l’aide de PowerShell, consultez [Mettre à l’échelle automatiquement un groupe de machines virtuelles identiques dans le portail Azure](tutorial-autoscale-powershell.md). Vous pouvez également configurer la mise à l’échelle automatique avec [Azure CLI](tutorial-autoscale-cli.md) et les [modèles Azure](tutorial-autoscale-template.md).


### <a name="if-i-have-stopped-deallocated-a-vm-is-that-vm-started-as-part-of-an-autoscale-operation"></a>Si j’ai arrêté (libéré) une machine virtuelle, celle-ci démarre-t-elle en bénéficiant de l’opération de mise à l’échelle automatique ?

Non. Si la mise à l’échelle automatique nécessite des instances de machine virtuelle supplémentaires appartenant au groupe identique, une nouvelle instance de machine virtuelle doit être créée. Les instances de machine virtuelle arrêtées (libérées) ne démarrent pas sous l’événement de mise à l’échelle automatique. Toutefois, ces machines virtuelles arrêtées (libérées) risquent d’être supprimées dans le cadre d’un événement de mise à l’échelle qui diminue le nombre d’instances, de la même façon que n’importe quelle instance de machine virtuelle peut être supprimée en fonction de l’ordre de l’ID d’instance de machine virtuelle.



## <a name="certificates"></a>Certificats

### <a name="how-do-i-securely-ship-a-certificate-to-the-vm"></a>Comment expédier en toute sécurité un certificat sur la machine virtuelle ?

Pour expédier en toute sécurité un certificat sur la machine virtuelle, vous pouvez installer un certificat client directement dans un magasin de certificats Windows à partir du coffre de clés du client.

Utilisez le JSON suivant :

```json
"secrets": [
    {
        "sourceVault": {
            "id": "/subscriptions/{subscriptionid}/resourceGroups/myrg1/providers/Microsoft.KeyVault/vaults/mykeyvault1"
        },
        "vaultCertificates": [
            {
                "certificateUrl": "https://mykeyvault1.vault.azure.net/secrets/{secretname}/{secret-version}",
                "certificateStore": "certificateStoreName"
            }
        ]
    }
]
```

Le code prend en charge Windows et Linux.

Pour plus d’informations, consultez [Création ou mise à jour d’un groupe de machines virtuelles identiques](/rest/api/compute/virtualmachinescalesets/createorupdate).


### <a name="how-do-i-use-self-signed-certificates-provisioned-for-azure-service-fabric-clusters"></a>Comment faire pour utiliser des certificats auto-signés approvisionnés pour les clusters Azure Service Fabric
Pour obtenir l’exemple le plus récent, utilisez l’instruction Azure CLI suivante dans l’interpréteur de commandes Azure et lisez la documentation sur l’exemple de module CLI Service Fabrics (imprimée dans stdout) :

```azurecli
az sf cluster create -h
```

Vous ne pouvez pas utiliser de certificats auto-signés pour une approbation distribuée fournie par une autorité de certification. Vous ne pouvez pas non plus les utiliser avec un cluster Service Fabric destiné à héberger des solutions de production d’entreprise. Pour obtenir de l’aide supplémentaire sur la sécurité dans Service Fabric, passez en revue [Bonnes pratiques pour la sécurité Azure Service Fabric](../security/fundamentals/service-fabric-best-practices.md) et [Scénarios de sécurité d’un cluster Service Fabric](../service-fabric/service-fabric-cluster-security.md).

### <a name="can-i-specify-an-ssh-key-pair-to-use-for-ssh-authentication-with-a-linux-virtual-machine-scale-set-from-a-resource-manager-template"></a>Puis-je spécifier une paire de clés SSH à utiliser pour l’authentification SSH avec un groupe de machines virtuelles identiques Linux à partir d’un modèle Resource Manager ?

Oui. L’API REST pour **osProfile** est similaire à l’API REST de machine virtuelle standard.

Incluez **osProfile** dans votre modèle :

```json
"osProfile": {
    "computerName": "[variables('vmName')]",
    "adminUsername": "[parameters('adminUserName')]",
    "linuxConfiguration": {
        "disablePasswordAuthentication": "true",
        "ssh": {
            "publicKeys": [
                {
                    "path": "[variables('sshKeyPath')]",
                    "keyData": "[parameters('sshKeyData')]"
                }
            ]
        }
    }
}
```

Ce bloc JSON est utilisé dans le [modèle de démarrage rapide Azure](https://github.com/Azure/azure-quickstart-templates/blob/master/101-vm-sshkey/azuredeploy.json).

Pour plus d’informations, consultez [Création ou mise à jour d’un groupe de machines virtuelles identiques](/rest/api/compute/virtualmachinescalesets/createorupdate#linuxconfiguration).

### <a name="how-do-i-remove-deprecated-certificates"></a>Comment supprimer des certificats obsolètes ?

Pour supprimer les certificats obsolètes, supprimez l’ancien certificat dans la liste des certificats du coffre. Laissez tous les certificats que vous souhaitez garder sur votre ordinateur dans la liste. Cela ne supprime pas le certificat de toutes vos machines virtuelles. Cela n’ajoute pas non plus le certificat sur les nouvelles machines virtuelles créées dans le groupe de machines virtuelles identiques.

Pour supprimer le certificat sur des machines virtuelles existantes, utilisez une extension de script personnalisé afin de supprimer manuellement les certificats de votre magasin de certificats.

### <a name="how-do-i-inject-an-existing-ssh-public-key-into-the-virtual-machine-scale-set-ssh-layer-during-provisioning"></a>Comment injecter une clé publique SSH existante dans la couche SSH du groupe de machines virtuelles identiques lors de la configuration ?

Si vous fournissez uniquement une clé publique SSH aux machines virtuelles, vous n’avez pas besoin de placer les clés publiques dans Key Vault. Les clés publiques ne sont pas secrètes.

Vous pouvez fournir les clés publiques SSH en texte brut lorsque vous créez une machine virtuelle Linux :

```json
"linuxConfiguration": {
    "ssh": {
        "publicKeys": [
            {
                "path": "path",
                "keyData": "publickey"
            }
        ]
    }
}
```

nom d’élément linuxConfiguration | Obligatoire | Type | Description
--- | --- | --- | ---
ssh | Non | Collection | Spécifie la configuration de la clé SSH pour un système d’exploitation Linux
path | Oui | String | Spécifie le chemin d’accès du fichier Linux où les clés SSH ou le certificat doivent être placés
keyData | Oui | String | Spécifie une clé publique SSH encodée en base64

Pour obtenir un exemple, consultez [le modèle de démarrage rapide GitHub 101-vm-sshkey](https://github.com/Azure/azure-quickstart-templates/blob/master/101-vm-sshkey/azuredeploy.json).

### <a name="when-i-run-update-azvmss-after-adding-more-than-one-certificate-from-the-same-key-vault-i-see-the-following-message"></a>Lorsque j’exécute `Update-AzVmss` après l’ajout de plusieurs certificats à partir du même coffre de clés, je reçois le message suivant :

>Update-AzVmss : List secret contient des instances répétées de /subscriptions/\<my-subscription-id>/resourceGroups/internal-rg-dev/providers/Microsoft.KeyVault/vaults/internal-keyvault-dev, ce qui n’est pas autorisé.

Cela peut se produire si vous essayez d’ajouter à nouveau le même coffre au lieu d’utiliser un nouveau certificat de coffre pour le coffre source existant. La commande `Add-AzVmssSecret` ne fonctionne pas correctement si vous ajoutez des secrets supplémentaires.

Pour ajouter plus de secrets à partir du même coffre de clés, mettez à jour la liste $vmss.properties.osProfile.secrets[0].vaultCertificates.

Pour connaître la structure d’entrée attendue, consultez [Création ou mise à jour d’un groupe de machines virtuelles](/rest/api/compute/virtualmachinescalesets/createorupdate).

Recherchez le secret dans l’objet du groupe de machines virtuelles identiques qui se trouve dans le coffre de clés. Ensuite, ajoutez votre référence de certificat (l’URL et le nom du magasin des secrets) dans la liste associée au coffre.

> [!NOTE]
> Actuellement, vous ne pouvez pas supprimer des certificats des machines virtuelles à l’aide de l’API du groupe de machines virtuelles identiques.
>

Les nouvelles machines virtuelles ne disposent pas de l’ancien certificat. Toutefois, les machines virtuelles qui ont le certificat et qui sont déjà déployées disposent de l’ancien certificat.

### <a name="can-i-push-certificates-to-the-virtual-machine-scale-set-without-providing-the-password-when-the-certificate-is-in-the-secret-store"></a>Puis-je transmettre des certificats au groupe de machines virtuelles identiques sans fournir le mot de passe lorsque le certificat est dans le magasin des secrets ?

Il est inutile de coder en dur les mots de passe dans les scripts. Vous pouvez récupérer de manière dynamique les mots de passe avec les autorisations que vous utilisez pour exécuter le script de déploiement. Si vous avez un script qui déplace un certificat à partir du magasin des secrets vers le coffre de clés, la commande `get certificate` du magasin des secrets génère également le mot de passe du fichier .pfx.

### <a name="how-does-the-secrets-property-of-virtualmachineprofileosprofile-for-a-virtual-machine-scale-set-work-why-do-i-need-the-sourcevault-value-when-i-have-to-specify-the-absolute-uri-for-a-certificate-by-using-the-certificateurl-property"></a>Comment fonctionne la propriété des secrets de virtualMachineProfile.osProfile d’un groupe de machines virtuelles identiques ? Pourquoi ai-je besoin de la valeur sourceVault lorsque je dois spécifier l’URI absolu d’un certificat à l’aide de la propriété certificateUrl ?

Une référence de certificat WinRM (Windows Remote Management) doit être présente dans la propriété des secrets du profil du système d’exploitation.

L’objectif consistant à indiquer le coffre source est d’appliquer les stratégies de liste de contrôle d’accès (ACL) qui existent dans le modèle Azure Cloud Service d’un utilisateur. Si le coffre source n’est pas spécifié, les utilisateurs ne disposant pas des autorisations pour déployer ou accéder aux secrets d’un coffre de clés peuvent y parvenir via un CRP (Compute Resource Provider). Les listes ACL existent même pour les ressources qui n’existent pas.

Si vous fournissez un ID sourceVault incorrect mais une URL de coffre de clés valide, une erreur est signalée lorsque vous interrogez l’opération.

### <a name="if-i-add-secrets-to-an-existing-virtual-machine-scale-set-are-the-secrets-injected-into-existing-vms-or-only-into-new-ones"></a>Si j’ajoute des secrets à un groupe de machines virtuelles identiques existant, les secrets sont-ils injectés dans les machines virtuelles existantes ou uniquement les nouvelles ?

Les certificats sont ajoutés à toutes vos machines virtuelles, mêmes les préexistantes. Si la propriété upgradePolicy de votre groupe de machines virtuelles identiques est définie sur **manuelle**, le certificat est ajouté à la machine virtuelle lorsque vous effectuez une mise à jour manuelle sur celle-ci.

### <a name="where-do-i-put-certificates-for-linux-vms"></a>Où dois-je placer les certificats pour les machines virtuelles Linux ?

Pour savoir comment déployer des certificats pour les machines virtuelles Linux, consultez [Déployer des certificats sur les machines virtuelles à partir de coffres de clés gérés par les clients](/archive/blogs/kv/deploy-certificates-to-vms-from-customer-managed-key-vault).

### <a name="how-do-i-add-a-new-vault-certificate-to-a-new-certificate-object"></a>Comment ajouter un nouveau certificat de coffre à un nouvel objet de certificat ?

Pour ajouter un certificat de coffre à un secret existant, consultez l’exemple PowerShell suivant. Utilisez un seul objet secret.

```powershell
$newVaultCertificate = New-AzVmssVaultCertificateConfig -CertificateStore MY -CertificateUrl https://sansunallapps1.vault.azure.net:443/secrets/dg-private-enc/55fa0332edc44a84ad655298905f1809

$vmss.VirtualMachineProfile.OsProfile.Secrets[0].VaultCertificates.Add($newVaultCertificate)

Update-AzVmss -VirtualMachineScaleSet $vmss -ResourceGroup $rg -Name $vmssName
```

### <a name="what-happens-to-certificates-if-you-reimage-a-vm"></a>Que se passe-t-il pour les certificats en cas de réinitialisation d’une machine virtuelle ?

Si vous réinitialisez une machine virtuelle, les certificats sont supprimés. La réinitialisation efface le disque du système d’exploitation dans sa totalité.

### <a name="what-happens-if-you-delete-a-certificate-from-the-key-vault"></a>Que se passe-t-il si vous supprimez un certificat dans le coffre de clés ?

Si le secret est supprimé de Key Vault et si vous exécutez `stop deallocate` pour toutes vos machines virtuelles, puis les démarrez à nouveau, vous rencontrez une erreur. L’erreur se produit car le CRP, qui doit récupérer les secrets dans le coffre de clés, ne le peut pas. Dans ce scénario, vous pouvez supprimer les certificats à partir du modèle de groupe de machines virtuelles identiques.

Le composant CRP ne conserve pas les secrets du client. Si vous exécutez `stop deallocate` pour toutes les machines virtuelles dans le groupe de machines virtuelles identiques, le cache est supprimé. Dans ce scénario, les secrets sont récupérés à partir du coffre de clés.

Vous ne rencontrez pas ce problème lors de la montée en charge, car il existe une copie en cache du secret dans Azure Service Fabric (dans le modèle client à structure unique).

### <a name="why-do-i-have-to-specify-the-certificate-version-when-i-use-key-vault"></a>Pourquoi dois-je spécifier la version du certificat lorsque j’utilise Key Vault ?

L’objectif de la spécification Key Vault consistant à spécifier la version du certificat est d’indiquer clairement à l’utilisateur quel certificat est déployé sur ses machines virtuelles.

Si vous créez une machine virtuelle, puis mettez à jour votre secret dans le coffre de clés, ce nouveau certificat n’est pas téléchargé sur vos machines virtuelles. Mais vos machines virtuelles ont l’air d’y faire référence et les nouvelles machines virtuelles obtiennent le nouveau secret. Pour éviter cela, vous devez référencer une version pour le secret.

### <a name="my-team-works-with-several-certificates-that-are-distributed-to-us-as-cer-public-keys-what-is-the-recommended-approach-for-deploying-these-certificates-to-a-virtual-machine-scale-set"></a>Mon équipe utilise plusieurs certificats qui nous sont distribués en tant que clés publiques .cer. Quelle est l’approche recommandée pour le déploiement de ces certificats dans un groupe de machines virtuelles identiques ?

Pour déployer des clés publiques .cer dans un groupe de machines virtuelles identiques, vous pouvez générer un fichier .pfx qui contient uniquement des fichiers .cer. Pour ce faire, utilisez `X509ContentType = Pfx`. Par exemple, chargez le fichier .cer en tant qu’objet x509Certificate2 dans C# ou PowerShell, puis appelez la méthode.

Pour plus d’informations, consultez [Méthode X509Certificate.Export (X509ContentType, chaîne)](/dotnet/api/system.security.cryptography.x509certificates.x509certificate.export?view=netcore-3.1#system_security_cryptography_x509certificates_x509certificate_export_system_security_cryptography_x509certificates_x509contenttype_system_string_).

### <a name="how-do-i-pass-in-certificates-as-base64-strings"></a>Comment faire pour passer des certificats en tant que chaînes base64 ?

Pour émuler le transfert d’un certificat sous forme de chaîne en base64, vous pouvez extraire la dernière version d’URL dans un modèle Resource Manager. Incluez la propriété JSON suivante dans votre modèle Resource Manager :

```json
"certificateUrl": "[reference(resourceId(parameters('vaultResourceGroup'), 'Microsoft.KeyVault/vaults/secrets', parameters('vaultName'), parameters('secretName')), '2015-06-01').secretUriWithVersion]"
```

### <a name="do-i-have-to-wrap-certificates-in-json-objects-in-key-vaults"></a>Dois-je encapsuler les certificats dans des objets JSON dans les coffres de clés ?

Dans les groupes de machines virtuelles identiques et les machines virtuelles, les certificats doivent être encapsulés dans des objets JSON.

Nous prenons également en charge le type de contenu application/x-pkcs12.

Actuellement, nous ne prenons pas en charge les fichiers .cer. Pour utiliser des fichiers .cer, exportez-les dans des conteneurs .pfx.



## <a name="compliance-and-security"></a>Conformité et sécurité

### <a name="are-virtual-machine-scale-sets-pci-compliant"></a>Les groupes de machines virtuelles identiques sont-ils compatibles avec PCI ?

Les groupes de machines virtuelles identiques sont une fine couche API sur le composant CRP. Les deux composants font partie de la plateforme de calcul dans l’arborescence des services Azure.

Du point de vue de la conformité, les groupes de machines virtuelles identiques sont une composante essentielle de la plateforme de calcul Azure. Ils partagent une équipe, des outils, des processus, une méthodologie de déploiement, des contrôles de sécurité, la compilation JIT, la surveillance, les alertes et ainsi de suite, avec le CRP lui-même. Les groupes de machines virtuelles identiques sont conformes à PCI (Payment Card Industry), car le CRP fait partie de l’attestation PCI DSS (Data Security Standard) actuelle.

Pour plus d’informations, consultez le [Centre de gestion de la confidentialité de Microsoft](https://www.microsoft.com/TrustCenter/Compliance/PCI).

### <a name="does-managed-identities-for-azure-resources-work-with-virtual-machine-scale-sets"></a>Les [identités managées pour ressources Azure](../active-directory/managed-identities-azure-resources/overview.md) fonctionnent-elles avec des groupes de machines virtuelles identiques ?

Oui. Vous pouvez voir des exemples de modèles MSI dans les modèles de démarrage rapide Azure pour [Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-msi) et [Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-msi).

## <a name="deleting"></a>En cours de suppression

### <a name="will-the-locks-i-set-in-place-on-virtual-machine-scale-set-instances-be-respected-when-deleting-instances"></a>Les verrous que j’ai mis en place sur des instances de groupe de machines virtuelles identiques seront-ils respectés lors de la suppression des instances ?

Dans le portail Azure, vous avez la possibilité de supprimer une instance individuelle ou de supprimer en bloc en sélectionnant plusieurs instances. Si vous essayez de supprimer une instance individuelle avec un verrou en place, le verrou est respecté et vous ne pourrez pas supprimer l’instance. Cependant, si vous sélectionnez en bloc plusieurs instances et que l’une de ces instances a un verrou en place, le ou les verrous ne seront pas respectés et toutes les instances sélectionnées seront supprimées.

Dans Azure CLI, vous avez la possibilité de supprimer une instance individuelle. Si vous essayez de supprimer une instance individuelle avec un verrou en place, le verrou est respecté et vous ne pourrez pas supprimer cette instance.

## <a name="extensions"></a>Extensions

### <a name="how-do-i-delete-a-virtual-machine-scale-set-extension"></a>Comment supprimer une extension de groupe de machines virtuelles identiques ?

Pour supprimer une extension de groupe de machines virtuelles identiques, utilisez l’exemple PowerShell suivant :

```powershell
$vmss = Get-AzVmss -ResourceGroupName "resource_group_name" -VMScaleSetName "vmssName"

$vmss=Remove-AzVmssExtension -VirtualMachineScaleSet $vmss -Name "extensionName"

Update-AzVmss -ResourceGroupName "resource_group_name" -VMScaleSetName "vmssName" -VirtualMacineScaleSet $vmss
```

Vous trouverez la valeur extensionName dans `$vmss`.

### <a name="is-there-a-virtual-machine-scale-set-template-example-that-integrates-with-azure-monitor-logs"></a>Existe-t-il un exemple de modèle de groupe de machines virtuelles identiques qui s’intègre aux journaux Azure Monitor ?

Pour obtenir un exemple de modèle de groupe de machines virtuelles identiques qui s’intègre aux journaux Azure Monitor, consultez le deuxième exemple sous [Deploy an Azure Service Fabric cluster and enable monitoring by using Azure Monitor logs](https://github.com/krnese/AzureDeploy/tree/master/OMS/MSOMS/ServiceFabric) (Déployer un cluster Azure Service Fabric et activer le monitoring à l’aide des journaux Azure Monitor).

### <a name="how-do-i-add-an-extension-to-all-vms-in-my-virtual-machine-scale-set"></a>Comment ajouter une extension pour toutes les machines virtuelles de mon groupe de machines virtuelles identiques ?

Si la stratégie de mise à jour est définie sur **automatique**, le redéploiement du modèle avec les nouvelles propriétés d’extension met à jour toutes les machines virtuelles.

Si la stratégie de mise à jour est définie sur **manuelle**, mettez d’abord à jour l’extension, puis mettez manuellement à jour toutes les instances de vos machines virtuelles.

### <a name="if-the-extensions-associated-with-an-existing-virtual-machine-scale-set-are-updated-are-existing-vms-affected"></a>Si les extensions associées à un groupe de machines virtuelles identiques existant sont mises à jour, cela affecte-t-il les machines virtuelles existantes ?

Si la définition de l’extension dans le modèle de groupe de machines virtuelles identiques est mis à jour et la propriété upgradePolicy est définie sur **automatique**, les machines virtuelles sont mises à jour. Si la propriété upgradePolicy est définie sur **manuelle**, les extensions sont signalées comme ne correspondant pas au modèle.

### <a name="are-extensions-run-again-when-an-existing-machine-is-service-healed-or-reimaged"></a>Les extensions sont-elles exécutées à nouveau lorsqu’une machine existante est corrigée par service ou réinitialisée ?

Si une machine virtuelle existante est corrigée par service, cela s’apparente à un redémarrage et les extensions ne sont pas exécutées à nouveau. Si elle est réinitialisée, le processus est similaire au remplacement du disque du système d’exploitation par l’image source. Toutes les spécialisations du modèle le plus récent, telles que les extensions, sont exécutées.

### <a name="how-do-i-join-a-virtual-machine-scale-set-to-an-active-directory-domain"></a>Comment joindre un groupe de machines virtuelles identiques à un domaine Active Directory ?

Pour joindre un groupe de machines virtuelles identiques à un domaine Active Directory (AD), vous pouvez définir une extension.

Pour définir une extension, utilisez la propriété JsonADDomainExtension :

```json
"extensionProfile": {
    "extensions": [
        {
            "name": "joindomain",
            "properties": {
                "publisher": "Microsoft.Compute",
                "type": "JsonADDomainExtension",
                "typeHandlerVersion": "1.3",
                "settings": {
                    "Name": "[parameters('domainName')]",
                    "OUPath": "[variables('ouPath')]",
                    "User": "[variables('domainAndUsername')]",
                    "Restart": "true",
                    "Options": "[variables('domainJoinOptions')]"
                },
                "protectedsettings": {
                    "Password": "[parameters('domainJoinPassword')]"
                }
            }
        }
    ]
}
```

### <a name="my-virtual-machine-scale-set-extension-is-trying-to-install-something-that-requires-a-reboot"></a>Mon extension de groupe de machines virtuelles identiques tente d’installer un composant qui nécessite un redémarrage.

Si votre extension de groupe de machines virtuelles identiques tente d’installer un composant nécessitant un redémarrage, vous pouvez utiliser l’extension Azure Automation DSC. Si le système d’exploitation est Windows Server 2012 R2, Azure extrait la configuration Windows Management Framework (WMF) 5.0, redémarre, puis poursuit la configuration.

### <a name="how-do-i-turn-on-antimalware-in-my-virtual-machine-scale-set"></a>Comment activer le logiciel anti-programme malveillant dans mon groupe de machines virtuelles identiques ?

Pour activer le logiciel anti-programme malveillant dans votre groupe de machines virtuelles identiques, utilisez l’exemple PowerShell suivant :

```powershell
$rgname = 'autolap'
$vmssname = 'autolapbr'
$location = 'eastus'

# Retrieve the most recent version number of the extension.
$allVersions= (Get-AzVMExtensionImage -Location $location -PublisherName "Microsoft.Azure.Security" -Type "IaaSAntimalware").Version
$versionString = $allVersions[($allVersions.count)-1].Split(".")[0] + "." + $allVersions[($allVersions.count)-1].Split(".")[1]

$VMSS = Get-AzVmss -ResourceGroupName $rgname -VMScaleSetName $vmssname
echo $VMSS
Add-AzVmssExtension -VirtualMachineScaleSet $VMSS -Name "IaaSAntimalware" -Publisher "Microsoft.Azure.Security" -Type "IaaSAntimalware" -TypeHandlerVersion $versionString
Update-AzVmss -ResourceGroupName $rgname -Name $vmssname -VirtualMachineScaleSet $VMSS
```

### <a name="how-do-i-execute-a-custom-script-thats-hosted-in-a-private-storage-account"></a>Comment faire pour exécuter un script personnalisé hébergé dans un compte de stockage privé ?

Pour exécuter un script personnalisé qui est hébergé dans un compte de stockage privé, configurez les paramètres protégés avec le nom et la clé du compte de stockage. Pour plus d’informations, voir [Custom Script Extension](../virtual-machines/extensions/custom-script-windows.md?toc=/azure/virtual-machines/windows/toc.json#property-managedidentity) (Extension de script personnalisé).

## <a name="passwords"></a>Mots de passe

### <a name="how-do-i-reset-the-password-for-vms-in-my-virtual-machine-scale-set"></a>Comment réinitialiser le mot de passe des machines virtuelles dans mon groupe de machines virtuelles identiques ?

Il existe deux principales façons de changer le mot de passe pour les machines virtuelles dans les groupes identiques.

- Changez directement le modèle de groupe de machines virtuelles identiques. Disponible avec l’API version 2017-12-01 et plus.

    Mettez à jour les informations d’identification d’administrateur directement dans le modèle de groupe identique (par exemple en utilisant Azure Resource Explorer, PowerShell ou l’interface CLI). Une fois que le groupe identique est mis à jour, toutes les nouvelles machines virtuelles disposent des nouvelles informations d’identification. Les machines virtuelles existantes ont les nouvelles informations d’identification uniquement si elles sont réinitialisées.

- Réinitialisez le mot de passe à l’aide des extensions d’accès aux machines virtuelles. Veillez à respecter les exigences de mot de passe, telles qu’elles sont décrites [ici](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).

    Utilisez l’exemple PowerShell suivant :

    ```powershell
    $vmssName = "myvmss"
    $vmssResourceGroup = "myvmssrg"
    $publicConfig = @{"UserName" = "newuser"}
    $privateConfig = @{"Password" = "********"}

    $extName = "VMAccessAgent"
    $publisher = "Microsoft.Compute"
    $vmss = Get-AzVmss -ResourceGroupName $vmssResourceGroup -VMScaleSetName $vmssName
    $vmss = Add-AzVmssExtension -VirtualMachineScaleSet $vmss -Name $extName -Publisher $publisher -Setting $publicConfig -ProtectedSetting $privateConfig -Type $extName -TypeHandlerVersion "2.0" -AutoUpgradeMinorVersion $true
    Update-AzVmss -ResourceGroupName $vmssResourceGroup -Name $vmssName -VirtualMachineScaleSet $vmss
    ```

## <a name="networking"></a>Mise en réseau

### <a name="is-it-possible-to-assign-a-network-security-group-nsg-to-a-scale-set-so-that-it-applies-to-all-the-vm-nics-in-the-set"></a>Est-il possible d’affecter un groupe de sécurité réseau (NSG) à un groupe identique, afin de l’appliquer à toutes les cartes réseau de machines virtuelles du groupe ?

Oui. Un groupe de sécurité réseau peut être appliqué directement à un groupe identique en le référençant dans la section networkInterfaceConfigurations du profil de réseau. Exemple :

```json
"networkProfile": {
    "networkInterfaceConfigurations": [
        {
            "name": "nic1",
            "properties": {
                "primary": "true",
                "ipConfigurations": [
                    {
                        "name": "ip1",
                        "properties": {
                            "subnet": {
                                "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/virtualNetworks/', variables('vnetName'), '/subnets/subnet1')]"
                            },
                            "loadBalancerInboundNatPools": [
                                {
                                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('lbName'), '/inboundNatPools/natPool1')]"
                                }
                            ],
                            "loadBalancerBackendAddressPools": [
                                {
                                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/loadBalancers/', variables('lbName'), '/backendAddressPools/addressPool1')]"
                                }
                            ]
                        }
                    }
                ],
                "networkSecurityGroup": {
                    "id": "[concat('/subscriptions/', subscription().subscriptionId,'/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Network/networkSecurityGroups/', variables('nsgName'))]"
                }
            }
        }
    ]
}
```

### <a name="how-do-i-do-a-vip-swap-for-virtual-machine-scale-sets-in-the-same-subscription-and-same-region"></a>Comment effectuer un échange d’adresses IP virtuelles pour les groupes de machines virtuelles identiques dans le même abonnement et la même région ?

Si vous disposez de deux groupes de machines virtuelles identiques avec serveurs frontaux Azure Load Balancer, et qu’ils font partie d’une région et d’un abonnement identique, vous pouvez libérer l’adresse IP publique d’un groupe pour l’assigner à l’autre. Consultez [VIP Swap: Blue-green deployment in Azure Resource Manager](https://msftstack.wordpress.com/2017/02/24/vip-swap-blue-green-deployment-in-azure-resource-manager/) (Échange d’adresse IP virtuelle (VIP) : Déploiement Bleu/vert dans Azure Resource Manager) pour avoir un exemple. À noter que cela implique un délai, car les ressources doivent être libérées/allouées au niveau du réseau. Une option plus rapide consiste à utiliser la passerelle Azure Application Gateway avec deux pools backend et une règle de routage. Une autre option consiste à héberger votre application avec [Azure App service](https://azure.microsoft.com/services/app-service/), qui fournit une assistance pour basculer rapidement entre emplacements intermédiaires et emplacements de production.

### <a name="how-do-i-specify-a-range-of-private-ip-addresses-to-use-for-static-private-ip-address-allocation"></a>Comment spécifier une plage d’adresses IP privées à utiliser pour l’allocation d’adresse IP privée statique ?

Les adresses IP sont sélectionnées à partir d’un sous-réseau que vous spécifiez.

La méthode d’allocation d’adresses IP d’un groupe de machines virtuelles identiques est toujours « dynamique », mais cela ne signifie pas que ces adresses IP peuvent changer. Dans ce cas, « dynamique » signifie uniquement que vous ne spécifiez pas l’adresse IP dans une requête PUT. Spécifiez l’ensemble statique à l’aide du sous-réseau.

### <a name="how-do-i-deploy-a-virtual-machine-scale-set-to-an-existing-azure-virtual-network"></a>Comment déployer un groupe de machines virtuelles identiques sur un réseau virtuel Azure existant ?

Pour déployer un groupe de machines virtuelles identiques sur un réseau virtuel Azure existant, consultez [Déployer un groupe de machines virtuelles identiques sur un réseau virtuel existant](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-existing-vnet).

### <a name="can-i-use-scale-sets-with-accelerated-networking"></a>Puis-je me servir de groupes identiques lors d’une mise en réseau accélérée ?

Oui. Pour utiliser la mise en réseau accélérée, définissez enableAcceleratedNetworking sur True dans les paramètres networkInterfaceConfigurations du groupe identique. Par exemple

```json
"networkProfile": {
    "networkInterfaceConfigurations": [
        {
            "name": "niconfig1",
            "properties": {
                "primary": true,
                "enableAcceleratedNetworking" : true,
                "ipConfigurations": [
                ]
            }
        }
    ]
}
```

### <a name="how-can-i-configure-the-dns-servers-used-by-a-scale-set"></a>Comment puis-je configurer les serveurs DNS utilisés par un groupe identique ?

Pour créer un groupe de machines virtuelles identiques avec une configuration DNS personnalisée, ajoutez un paquet JSON dnsSettings à la section networkInterfaceConfigurations du groupe identique. Exemple :

```json
    "dnsSettings":{
        "dnsServers":["10.0.0.6", "10.0.0.5"]
    }
```

### <a name="how-can-i-configure-a-scale-set-to-assign-a-public-ip-address-to-each-vm"></a>Comment puis-je configurer un groupe identique afin qu’il attribue une adresse IP publique à chaque machine virtuelle ?

Pour créer un groupe de machines virtuelles identiques qui attribue une adresse IP publique à chaque machine virtuelle, vérifiez que la version API de la ressource Microsoft.Compute/virtualMachineScaleSets est datée du 30/03/2017, puis ajoutez un paquet JSON _publicipaddressconfiguration_ à la section ipConfigurations du groupe identique. Exemple :

```json
    "publicipaddressconfiguration": {
        "name": "pub1",
        "properties": {
        "idleTimeoutInMinutes": 15
        }
    }
```

### <a name="can-i-configure-a-scale-set-to-work-with-multiple-application-gateways"></a>Puis-je configurer un groupe identique à utiliser avec plusieurs passerelles Application Gateway ?

Oui. Vous pouvez ajouter les IDs de ressource de plusieurs pools d’adresses backend Application Gateway à la liste _applicationGatewayBackendAddressPools_ dans la section _ipConfigurations_ du profil réseau du groupe identique.

## <a name="scale"></a>Scale

### <a name="in-what-case-would-i-create-a-virtual-machine-scale-set-with-fewer-than-two-vms"></a>Dans quel cas dois-je créer un groupe de machines virtuelles identiques avec moins de deux machines virtuelles ?

L’une des raisons de créer un groupe de machines virtuelles identiques avec moins de deux machines virtuelles est d’utiliser les propriétés élastiques d’un groupe de machines virtuelles identiques. Vous pouvez, par exemple, déployer un groupe de machines virtuelles identiques avec zéro machine virtuelle afin de définir votre infrastructure sans payer les frais de fonctionnement de la machine virtuelle. Ensuite, lorsque vous êtes prêt à déployer des machines virtuelles, augmentez la « capacité » du groupe de machines virtuelles identiques en fonction du nombre d’instances de production.

Une autre raison de créer un groupe de machines virtuelles identiques avec moins de deux machines virtuelles est si vous vous souciez moins de la disponibilité par rapport à l’utilisation d’un groupe à haute disponibilité avec des machines virtuelles discrètes. Les groupes de machines virtuelles identiques vous permettent d’utiliser des unités de calcul indifférenciées qui sont fongibles. Cette uniformité est un atout pour les groupes de machines virtuelles identiques par rapport aux groupes à haute disponibilité. De nombreuses charges de travail sans état n’effectuent pas le suivi des unités individuelles. Si la charge de travail baisse, vous pouvez descendre en puissance à une unité de calcul, puis monter en puissance lorsque la charge de travail augmente.

### <a name="how-do-i-change-the-number-of-vms-in-a-virtual-machine-scale-set"></a>Comment modifier le nombre de machines virtuelles dans un groupe de machines virtuelles identiques ?

Pour modifier le nombre de machines virtuelles dans un groupe de machines virtuelles identiques dans le portail Azure, dans la section des propriétés du groupe de machines virtuelles identiques, cliquez sur le panneau de « mise à l’échelle » et utilisez la barre du curseur.

### <a name="how-do-i-define-custom-alerts-for-when-certain-thresholds-are-reached"></a>Comment définir des alertes personnalisées lorsque certains seuils sont atteints ?

Vous bénéficiez d’une certaine flexibilité dans la façon dont vous gérez les alertes pour les seuils spécifiés. Vous pouvez, par exemple, définir des webhooks personnalisés. L’exemple de webhook suivant est extrait d’un modèle Resource Manager :

```json
{
    "type": "Microsoft.Insights/autoscaleSettings",
    "apiVersion": "[variables('insightsApi')]",
    "name": "autoscale",
    "location": "[parameters('resourceLocation')]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachineScaleSets/', parameters('vmSSName'))]"
    ],
    "properties": {
        "name": "autoscale",
        "targetResourceUri": "[concat('/subscriptions/',subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Compute/virtualMachineScaleSets/', parameters('vmSSName'))]",
        "enabled": true,
        "notifications": [
            {
                "operation": "Scale",
                "email": {
                    "sendToSubscriptionAdministrator": true,
                    "sendToSubscriptionCoAdministrators": true,
                    "customEmails": [
                        "youremail@address.com"
                    ]
                },
                "webhooks": [
                    {
                        "serviceUri": "<service uri>",
                        "properties": {
                            "key1": "custommetric",
                            "key2": "scalevmss"
                        }
                    }
                ]
            }
        ]
    }
}
```


## <a name="patching-and-operations"></a>Application de correctifs et opérations

### <a name="can-i-create-a-scale-set-in-an-existing-resource-group"></a>Puis-je créer un groupe identique dans un groupe de ressources existant ?

Tout-à-fait.

### <a name="can-i-move-a-scale-set-to-another-resource-group"></a>Puis-je déplacer un groupe identique vers un autre groupe de ressources ?

Oui, vous pouvez déplacer des ressources d’un groupe identique vers un nouvel abonnement ou un nouveau groupe de ressources.

### <a name="how-to-i-update-my-virtual-machine-scale-set-to-a-new-image-how-do-i-manage-patching"></a>Comment mettre à jour mon groupe de machines virtuelles identiques sur une nouvelle image ? Comment gérer la mise à jour corrective ?

Pour mettre à jour votre groupe de machines virtuelles identiques sur une nouvelle image et gérer la mise à jour corrective, consultez [Mettre à niveau un groupe de machines virtuelles identiques](./virtual-machine-scale-sets-upgrade-scale-set.md).

### <a name="can-i-use-the-reimage-operation-to-reset-a-vm-without-changing-the-image-that-is-i-want-reset-a-vm-to-factory-settings-rather-than-to-a-new-image"></a>Puis-je utiliser l’opération de réinitialisation pour réinitialiser une machine virtuelle sans modifier l’image ? (Autrement dit, je veux réinitialiser une machine virtuelle sur les paramètres d’usine plutôt qu’une nouvelle image.)

Oui, vous pouvez utiliser l’opération de réinitialisation pour réinitialiser une machine virtuelle sans modifier l’image. Toutefois, si votre groupe de machines virtuelles identiques référence une image de plateforme avec `version = latest`, votre machine virtuelle peut se mettre à jour vers une image de système d’exploitation ultérieure lorsque vous appelez `reimage`.

### <a name="is-it-possible-to-integrate-scale-sets-with-azure-monitor-logs"></a>Est-il possible d’intégrer des groupes identiques aux journaux Azure Monitor ?

Oui, vous pouvez le faire en installant l’extension Azure Monitor sur les machines virtuelles du groupe identique. Voici un exemple d’interface de ligne de commande Azure :

```azurecli
az vmss extension set --name MicrosoftMonitoringAgent --publisher Microsoft.EnterpriseCloud.Monitoring --resource-group Team-03 --vmss-name nt01 --settings "{'workspaceId': '<your workspace ID here>'}" --protected-settings "{'workspaceKey': '<your workspace key here'}"
```

Vous trouverez les éléments workspaceId et workspaceKey requis dans l’espace de travail Log Analytics du portail Azure. Dans la page de présentation, cliquez sur la vignette Paramètres. Cliquez sur l’onglet Sources connectées, situé en haut de la page.

> [!NOTE]
> Si votre groupe identique _upgradePolicy_ est défini sur Manuel, vous devez appliquer l’extension à toutes les machines virtuelles du groupe, en appelant une mise à niveau. Dans l’interface de ligne de commande, cela se traduirait par _az vmss update-instances_.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="troubleshooting"></a>Dépannage

### <a name="how-do-i-turn-on-boot-diagnostics"></a>Comment activer les diagnostics de démarrage ?

Pour activer les diagnostics de démarrage, commencez par créer un compte de stockage. Puis, placez ce bloc JSON dans la propriété **virtualMachineProfile** de votre groupe de machines virtuelles identiques et mettez celui-ci à jour :

```json
"diagnosticsProfile": {
    "bootDiagnostics": {
        "enabled": true,
        "storageUri": "http://yourstorageaccount.blob.core.windows.net"
    }
}
```

Lorsqu’une nouvelle machine virtuelle est créée, la propriété InstanceView de la machine virtuelle affiche les détails de la capture d’écran, etc. Voici un exemple :

```json
"bootDiagnostics": {
    "consoleScreenshotBlobUri": "https://o0sz3nhtbmkg6geswarm5.blob.core.windows.net/bootdiagnostics-swarmagen-4157d838-8335-4f78-bf0e-b616a99bc8bd/swarm-agent-9574AE92vmss-0_2.4157d838-8335-4f78-bf0e-b616a99bc8bd.screenshot.bmp",
    "serialConsoleLogBlobUri": "https://o0sz3nhtbmkg6geswarm5.blob.core.windows.net/bootdiagnostics-swarmagen-4157d838-8335-4f78-bf0e-b616a99bc8bd/swarm-agent-9574AE92vmss-0_2.4157d838-8335-4f78-bf0e-b616a99bc8bd.serialconsole.log"
}
```

## <a name="virtual-machine-properties"></a>Propriétés de machine virtuelle

### <a name="how-do-i-get-property-information-for-each-vm-without-making-multiple-calls-for-example-how-would-i-get-the-fault-domain-for-each-of-the-100-vms-in-my-virtual-machine-scale-set"></a>Comment obtenir des informations sur les propriétés de chaque machine virtuelle sans avoir à effectuer plusieurs appels ? Par exemple, comment obtenir le domaine par défaut pour chacune des 100 machines virtuelles dans mon groupe de machines virtuelles identiques ?

Pour obtenir des informations sur les propriétés de chaque machine virtuelle sans effectuer plusieurs appels, vous pouvez appeler `ListVMInstanceViews` en effectuant une API REST `GET` sur l’URI de ressource suivant :

/subscriptions/<subscription_id>/resourceGroups/<resource_group_name>/providers/Microsoft.Compute/virtualMachineScaleSets/<scaleset_name>/virtualMachines?$expand=instanceView&$select=instanceView

### <a name="can-i-pass-different-extension-arguments-to-different-vms-in-a-virtual-machine-scale-set"></a>Puis-je transférer des arguments d’extension différents à des machines virtuelles différentes dans un groupe de machines virtuelles identiques ?

Non, vous ne pouvez pas transférer des arguments d’extension différents à des machines virtuelles différentes dans un groupe de machines virtuelles identiques. Toutefois, les extensions peuvent agir en fonction des propriétés uniques de la machine virtuelle où elles s’exécutent, comme le nom de la machine. Les extensions peuvent aussi interroger les métadonnées d’instance sur http://169.254.169.254 pour obtenir plus d’informations sur la machine virtuelle.

### <a name="why-are-there-gaps-between-my-virtual-machine-scale-set-vm-machine-names-and-vm-ids-for-example-0-1-3"></a>Pourquoi y a-t-il des écarts entre les noms de machines virtuelles de mon groupe de machines virtuelles identiques et les ID des machines virtuelles ? Par exemple : 0, 1, 3...

Il existe des écarts entre les noms des machines virtuelles de votre groupe de machines virtuelles identiques et les ID des machines virtuelles, car la propriété **overprovision** de votre groupe de machines virtuelles identiques est définie sur la valeur par défaut **true**. Si le sur-approvisionnement est défini sur **true**, le nombre de machines virtuelles créées est supérieur à ce qui est demandé. Les machines virtuelles supplémentaires sont ensuite supprimées. Dans ce cas, vous obtenez une fiabilité de déploiement accrue aux dépens d’une affectation de noms contigus et de règles NAT contiguës.

Vous pouvez définir cette propriété sur **false**. Pour les petits groupes de machines virtuelles identiques, cela n’affecte pas vraiment la fiabilité du déploiement.

### <a name="what-is-the-difference-between-deleting-a-vm-in-a-virtual-machine-scale-set-and-deallocating-the-vm-when-should-i-choose-one-over-the-other"></a>Quelle est la différence entre la suppression d’une machine virtuelle dans un groupe de machines virtuelles identiques et la libération de la machine virtuelle ? Quand choisir une option par rapport à l’autre ?

La principale différence entre la suppression d’une machine virtuelle dans un groupe de machines virtuelles identiques et la libération de la machine virtuelle est que `deallocate` ne supprime pas les disques durs virtuels (VHD). Il existe des coûts de stockage associés à l’exécution de `stop deallocate`. Vous pouvez choisir l’une ou l’autre option pour l’une des raisons suivantes :

- Vous souhaitez arrêter de payer des frais de calcul, mais conserver l’état des disques des machines virtuelles.
- Vous souhaitez démarrer un groupe de machines virtuelles plus rapidement par rapport au scale-out d’un groupe de machines virtuelles identiques.
  - En relation avec ce scénario, vous avez peut-être créé votre propre moteur de mise à l’échelle automatique et souhaitez obtenir une mise à l’échelle de bout en bout plus rapide.
- Vous avez un groupe de machines virtuelles identiques qui est distribué inégalement entre les domaines d’erreur ou les domaines de mise à jour. Cela peut être dû au fait que vous avez supprimé sélectivement des machines virtuelles, ou parce que des machines virtuelles ont été supprimées après le sur-approvisionnement. Exécutez `stop deallocate` suivi de `start` sur le groupe de machines virtuelles identiques pour distribuer uniformément les machines virtuelles entre les domaines d’erreur ou les domaines de mise à jour.

### <a name="how-do-i-take-a-snapshot-of-a-virtual-machine-scale-set-instance"></a>Comment faire pour prendre une capture instantanée d’une instance d’un groupe de machines virtuelles identiques ?
Créez une capture instantanée à partir d’une instance d’un groupe de machines virtuelles identiques.

```azurepowershell-interactive
$rgname = "myResourceGroup"
$vmssname = "myVMScaleSet"
$Id = 0
$location = "East US"

$vmss1 = Get-AzVmssVM -ResourceGroupName $rgname -VMScaleSetName $vmssname -InstanceId $Id     
$snapshotconfig = New-AzSnapshotConfig -Location $location -AccountType Standard_LRS -OsType Windows -CreateOption Copy -SourceUri $vmss1.StorageProfile.OsDisk.ManagedDisk.id
New-AzSnapshot -ResourceGroupName $rgname -SnapshotName 'mySnapshot' -Snapshot $snapshotconfig
```

Créez un disque managé à partir de la capture instantanée.

```azurepowershell-interactive
$snapshotName = "mySnapshot"
$snapshot = Get-AzSnapshot -ResourceGroupName $rgname -SnapshotName $snapshotName  
$diskConfig = New-AzDiskConfig -AccountType Premium_LRS -Location $location -CreateOption Copy -SourceResourceId $snapshot.Id
$osDisk = New-AzDisk -Disk $diskConfig -ResourceGroupName $rgname -DiskName ($snapshotName + '_Disk')
```
