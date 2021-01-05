---
title: Mettre à jour un cluster pour utiliser le nom commun du certificat
description: Découvrez comment convertir un certificat de cluster Azure Service Fabric de déclarations basées sur l’empreinte en noms communs.
ms.topic: conceptual
ms.date: 09/06/2019
ms.openlocfilehash: 013b8190390a4b05791b0a56072487f249956ec5
ms.sourcegitcommit: d6a739ff99b2ba9f7705993cf23d4c668235719f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92495200"
---
# <a name="convert-cluster-certificates-from-thumbprint-based-declarations-to-common-names"></a>Convertir des certificats de cluster de déclarations basées sur l’empreinte en noms communs

La signature d’un certificat (communément appelée empreinte) est unique. Un certificat de cluster déclaré par empreinte fait référence à une instance spécifique d’un certificat. Cette spécificité rend la substitution du certificat, et la gestion en général, difficile et explicite. Chaque modification nécessite l’orchestration des mises à niveau du cluster et des hôtes de calcul sous-jacents.

La conversion des déclarations de certificat d’un cluster Azure Service Fabric, basées sur l’empreinte, en déclarations basées sur le nom commun du sujet du certificat simplifie considérablement la gestion. En particulier, la substitution d’un certificat ne nécessite plus de mise à niveau du cluster. Cet article explique comment convertir un cluster existant en déclarations basées sur le nom commun sans temps d’arrêt.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="move-to-certificate-authority-signed-certificates"></a>Passer aux certificats signés par une autorité de certification

La sécurité d’un cluster dont le certificat est déclaré par empreinte repose sur le fait qu’il est impossible, ou irréalisable par calcul, de falsifier un certificat avec la même signature qu’un autre. Dans ce cas, la provenance du certificat est moins importante, de sorte que les certificats auto-signés sont adéquats.

En revanche, la sécurité d’un cluster dont les certificats sont déclarés par nom commun découle de la confiance implicite que le propriétaire du cluster a dans son fournisseur de certificats. Le fournisseur est le service d’infrastructure à clé publique (PKI) qui a émis le certificat. La confiance est basée, entre autres, sur les pratiques de certification de l’infrastructure à clé publique, sur le fait que leur sécurité opérationnelle soit auditée et approuvée par d’autres parties de confiance, etc.

Le propriétaire du cluster doit également avoir une connaissance détaillée des autorités de certification qui émettent ses certificats, car il s’agit d’un aspect fondamental de la validation des certificats par sujet. Cela implique également que les certificats auto-signés sont totalement inadaptés à cette fin. Littéralement, tout le monde peut générer un certificat avec un sujet donné.

Un certificat déclaré par nom commun est généralement considéré comme valide dans les cas suivants :

* Sa chaîne peut être générée correctement.
* Le sujet contient l’élément de nom commun attendu.
* Son émetteur (immédiat ou supérieur dans la chaîne) est approuvé par l’agent effectuant la validation.

Service Fabric prend en charge la déclaration de certificats par nom commun de deux manières :

* Avec des émetteurs *implicites* , ce qui signifie que la chaîne doit se terminer par une ancre d’approbation.
* Avec des émetteurs déclarés par empreinte, ce qui est connu sous le nom d’épinglage de l’émetteur.

Pour plus d’informations, consultez [Déclarations de validation de certificat basées sur un nom commun](cluster-security-certificates.md#common-name-based-certificate-validation-declarations).

Pour convertir un cluster à l’aide d’un certificat auto-signé déclaré par empreinte en nom commun, le certificat cible, signé par l’autorité de certification, doit d’abord être introduit dans le cluster par empreinte. Ce n’est qu’ensuite que la conversion de l’empreinte en nom commun est possible.

À des fins de test, un certificat auto-signé *peut* être déclaré par nom commun, mais uniquement si l’émetteur est épinglé à sa propre empreinte. Du point de vue de la sécurité, cette action est quasiment équivalente à la déclaration du même certificat par empreinte. Une conversion réussie de ce type ne garantit pas la réussite d’une conversion de l’empreinte en nom commun avec un certificat signé par une autorité de certification. Nous vous recommandons de tester la conversion avec un certificat approprié, signé par une autorité de certification. Des options gratuites existent pour ce test.

## <a name="upload-the-certificate-and-install-it-in-the-scale-set"></a>Charger le certificat et l’installer dans le groupe identique

Dans Azure, le mécanisme recommandé pour l’obtention et l’approvisionnement des certificats implique Azure Key Vault et ses outils. Un certificat correspondant à la déclaration de certificat de cluster doit être approvisionné sur chaque nœud des groupes de machines virtuelles identiques qui composent votre cluster. Pour plus d’informations, consultez [Secrets sur des groupes de machines virtuelles identiques](../virtual-machine-scale-sets/virtual-machine-scale-sets-faq.md#how-do-i-securely-ship-a-certificate-to-the-vm).

Il est important d’installer les certificats de cluster actuels et cibles sur les machines virtuelles de chaque type de nœud du cluster avant de modifier les déclarations de certificat du cluster. Le parcours entre l’émission de certificats et l’approvisionnement sur un nœud Service Fabric est abordé en détail dans [Parcours d’un certificat](cluster-security-certificate-management.md#the-journey-of-a-certificate).

## <a name="bring-the-cluster-to-an-optimal-starting-state"></a>Amener le cluster à un état initial optimal

La conversion d’une déclaration de certificat par empreinte en déclaration par nom commun a un impact sur :

- la façon dont chaque nœud du cluster trouve et présente ses informations d’identification à d’autres nœuds ;
- la façon dont chaque nœud valide les informations d’identification de son homologue lors de l’établissement d’une connexion sécurisée.

Passez en revue les [règles de présentation et de validation pour les deux configurations](cluster-security-certificates.md#certificate-configuration-rules) avant de continuer. Le point le plus important à prendre en compte lorsque vous effectuez une conversion d’empreinte en nom commun est que les nœuds mis à niveau et ceux qui ne sont pas encore mis à niveau (c’est-à-dire les nœuds appartenant à différents domaines de mise à niveau) doivent être en mesure d’effectuer une authentification mutuelle réussie à tout moment pendant la mise à niveau. La méthode recommandée pour obtenir ce comportement consiste à déclarer le certificat cible ou objectif par empreinte lors d’une mise à niveau initiale. Effectuez ensuite la transition vers le nom commun lors d’une mise à niveau ultérieure. Si le cluster est déjà dans un état initial recommandé, vous pouvez ignorer cette section.

Il existe plusieurs états initiaux valides pour une conversion. L’invariant est que le cluster utilise déjà le certificat cible (déclaré par empreinte) au début de la mise à niveau vers le nom commun. Nous utilisons `GoalCert`, `OldCert1` et `OldCert2` dans cet article.

#### <a name="valid-starting-states"></a>États initiaux valides

- `Thumbprint: GoalCert, ThumbprintSecondary: None`
- `Thumbprint: GoalCert, ThumbprintSecondary: OldCert1`, où `GoalCert` a une date `NotAfter` ultérieure à celle de `OldCert1`
- `Thumbprint: OldCert1, ThumbprintSecondary: GoalCert`, où `GoalCert` a une date `NotAfter` ultérieure à celle de `OldCert1`

Si votre cluster n’est pas dans l’un des états valides décrits précédemment, consultez les informations relatives à l’obtention de cet état dans la section à la fin de cet article.

## <a name="select-the-desired-cn-based-certificate-validation-scheme"></a>Sélectionner le modèle souhaité de validation de certificat par nom commun

Comme décrit précédemment, Service Fabric prend en charge la déclaration de certificats par nom commun avec une ancre d’approbation implicite ou l’épinglage explicite des empreintes de l’émetteur. Pour plus d’informations, consultez [Déclarations de validation de certificat basées sur un nom commun](cluster-security-certificates.md#common-name-based-certificate-validation-declarations).

Veillez à bien comprendre les différences et les implications du choix de l’un ou l’autre mécanisme. Syntaxiquement, cette différence ou ce choix est déterminé par la valeur du paramètre `certificateIssuerThumbprintList`. Une valeur vide signifie que l’on compte sur une autorité de certification racine de confiance (ancre d’approbation), tandis qu’un ensemble d’empreintes restreint les émetteurs directs autorisés des certificats de cluster.

   > [!NOTE]
   > Le champ `certificateIssuerThumbprint` vous permet de spécifier les émetteurs directs attendus des certificats déclarés par nom commun sujet. Les valeurs acceptables sont une ou plusieurs empreintes SHA-1 séparées par des virgules. Cette action renforce la validation du certificat.
   >
   > Si aucun émetteur n’est spécifié ou si la liste est vide, l’authentification du certificat est acceptée si sa chaîne peut être générée. Le certificat se retrouve alors dans une racine approuvée par le validateur. Si une ou plusieurs empreintes d’émetteur sont spécifiées, le certificat sera accepté si l’empreinte de son émetteur direct, telle qu’extraite de la chaîne, correspond à l’une des valeurs spécifiées dans ce champ. Le certificat sera accepté, que la racine soit approuvée ou non.
   >
   > Une infrastructure à clé publique peut utiliser différentes autorités de certification (également appelées *émetteurs* ) pour signer des certificats avec un sujet donné. C’est pourquoi il est important de spécifier toutes les empreintes d’émetteur attendues pour ce sujet. En d’autres termes, le renouvellement d’un certificat n’est pas garanti d’être signé par le même émetteur que le certificat en cours de renouvellement.
   >
   > La meilleure pratique consiste à spécifier l’émetteur. Le chaînage des certificats jusqu’à une racine approuvée continuera de fonctionner même si l’émetteur est omis, mais ce comportement présente des limitations et risque d’être progressivement supprimé dans un avenir proche. Les clusters déployés dans Azure, sécurisés à l’aide de certificats X509 émis par une infrastructure privée à clé publique et déclarés par sujet, risquent de ne pas pouvoir être validés par Service Fabric (pour la communication entre le cluster et le service). La validation nécessite que la stratégie de certificat de l’infrastructure à clé publique soit détectable, disponible et accessible.

## <a name="update-the-clusters-azure-resource-manager-template-and-deploy"></a>Mettre à jour et déployer le modèle Azure Resource Manager du cluster

Gérez vos clusters Service Fabric à l’aide de modèles Azure Resource Manager. Une alternative, qui utilise également des artefacts JSON, est [Azure Resource Explorer (préversion)](https://resources.azure.com). Une expérience équivalente n’est pas disponible dans le portail Azure pour le moment.

Si le modèle d’origine correspondant à un cluster existant n’est pas disponible, un modèle équivalent peut être obtenu dans le portail Azure. Accédez au groupe de ressources qui contient le cluster, puis sélectionnez **Exporter le modèle** dans le menu **Automatisation** à gauche. Sélectionnez ensuite les ressources souhaitées. Au minimum, le groupe de machines virtuelles identiques et les ressources de cluster, respectivement, doivent être exportés. Le modèle généré peut également être téléchargé. Ce modèle peut nécessiter des modifications avant de pouvoir être entièrement déployé. Il est également possible que le modèle ne corresponde pas exactement à celui d’origine. Il reflète l’état actuel de la ressource de cluster.

Les modifications nécessaires sont les suivantes :

- Mise à jour de la définition de l’extension de nœud Service Fabric (sous la ressource de la machine virtuelle). Si le cluster définit plusieurs types de nœuds, vous devez mettre à jour la définition de chaque groupe de machines virtuelles identiques correspondant.
- Mise à jour de la définition de ressource de cluster.

Des exemples détaillés sont inclus ici.

### <a name="update-the-virtual-machine-scale-set-resources"></a>Mettre à jour les ressources du groupe de machines virtuelles identiques
De :
```json
"virtualMachineProfile": {
        "extensionProfile": {
            "extensions": [
                {
                    "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
                    "properties": {
                        "type": "ServiceFabricNode",
                        "autoUpgradeMinorVersion": true,
                        "protectedSettings": {
                            ...
                        },
                        "publisher": "Microsoft.Azure.ServiceFabric",
                        "settings": {
                            ...
                            "certificate": {
                                "thumbprint": "[parameters('certificateThumbprint')]",
                                "x509StoreName": "[parameters('certificateStoreValue')]"
                            }
                        },
                        ...
                    }
                },
```
Par :
```json
"virtualMachineProfile": {
        "extensionProfile": {
            "extensions": [
                {
                    "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
                    "properties": {
                        "type": "ServiceFabricNode",
                        "autoUpgradeMinorVersion": true,
                        "protectedSettings": {
                            ...
                        },
                        "publisher": "Microsoft.Azure.ServiceFabric",
                        "settings": {
                            ...
                            "certificate": {
                                "commonNames": [
                                    "[parameters('certificateCommonName')]"
                                ],
                                "x509StoreName": "[parameters('certificateStoreValue')]"
                            }
                        },
                        ...
                    }
                },
```

### <a name="update-the-cluster-resource"></a>Mettre à jour la ressource de cluster

Dans la ressource **Microsoft.ServiceFabric/clusters** , ajoutez une propriété **certificateCommonNames** avec un paramètre **commonNames** et supprimez complètement la propriété **certificate** (tous ses paramètres).

De :
```json
    {
        "apiVersion": "2018-02-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        "dependsOn": [
            ...
        ],
        "properties": {
            "addonFeatures": [
                ...
            ],
            "certificate": {
              "thumbprint": "[parameters('certificateThumbprint')]",
              "x509StoreName": "[parameters('certificateStoreValue')]"
            },
        ...
```
Par :
```json
    {
        "apiVersion": "2018-02-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        "dependsOn": [
            ...
        ],
        "properties": {
            "addonFeatures": [
                ...
            ],
            "certificateCommonNames": {
                "commonNames": [
                    {
                        "certificateCommonName": "[parameters('certificateCommonName')]",
                        "certificateIssuerThumbprint": "[parameters('certificateIssuerThumbprintList')]"
                    }
                ],
                "x509StoreName": "[parameters('certificateStoreValue')]"
            },
        ...
```

Pour plus d’informations, consultez [Déployer un cluster Service Fabric utilisant un nom commun de certificat au lieu d’une empreinte](./service-fabric-create-cluster-using-cert-cn.md).

## <a name="deploy-the-updated-template"></a>Déployer le modèle mis à jour

Redéployez le modèle mis à jour après avoir apporté les modifications.

```powershell
$groupname = "sfclustertutorialgroup"

New-AzResourceGroupDeployment -ResourceGroupName $groupname -Verbose `
    -TemplateParameterFile "C:\temp\cluster\parameters.json" -TemplateFile "C:\temp\cluster\template.json" 
```

## <a name="achieve-a-valid-starting-state-for-converting-a-cluster-to-cn-based-certificate-declarations"></a>Obtenir un état initial valide pour la conversion d’un cluster en déclarations de certificat par nom commun

| État initial | Mise à niveau 1 | Mise à niveau 2 |
| :--- | :--- | :--- |
| `Thumbprint: OldCert1, ThumbprintSecondary: None` et `GoalCert` ont une date `NotAfter` ultérieure à celle de `OldCert1` | `Thumbprint: OldCert1, ThumbprintSecondary: GoalCert` | - |
| `Thumbprint: OldCert1, ThumbprintSecondary: None` et `OldCert1` ont une date `NotAfter` ultérieure à celle de `GoalCert` | `Thumbprint: GoalCert, ThumbprintSecondary: OldCert1` | `Thumbprint: GoalCert, ThumbprintSecondary: None` |
| `Thumbprint: OldCert1, ThumbprintSecondary: GoalCert`, où `OldCert1` a une date `NotAfter` ultérieure à celle de `GoalCert` | Mettre à niveau vers `Thumbprint: GoalCert, ThumbprintSecondary: None` | - |
| `Thumbprint: GoalCert, ThumbprintSecondary: OldCert1`, où `OldCert1` a une date `NotAfter` ultérieure à celle de `GoalCert` | Mettre à niveau vers `Thumbprint: GoalCert, ThumbprintSecondary: None` | - |
| `Thumbprint: OldCert1, ThumbprintSecondary: OldCert2` | Supprimer `OldCert1` ou `OldCert2` pour accéder à l’état `Thumbprint: OldCertx, ThumbprintSecondary: None` | Continuer à partir du nouvel état initial |

Pour obtenir des instructions sur la façon d’effectuer ces mises à niveau, consultez [Gérer des certificats dans un cluster Azure Service Fabric](service-fabric-cluster-security-update-certs-azure.md).

## <a name="next-steps"></a>Étapes suivantes

* Découvrez plus en détail la [sécurité des clusters](service-fabric-cluster-security.md).
* Découvrez comment [substituer un certificat de cluster par nom commun](service-fabric-cluster-rollover-cert-cn.md).
* Découvrez comment [configurer un cluster pour la substitution automatique sans intervention de l’utilisateur](cluster-security-certificate-management.md).

[image1]: ./media/service-fabric-cluster-change-cert-thumbprint-to-cn/PortalViewTemplates.png
