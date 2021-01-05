---
title: Règles d’accès du pare-feu
description: Configurez des règles pour accéder à un registre de conteneurs Azure derrière un pare-feu, en autorisant l’accès à des noms de domaine d’API REST et de point de terminaison de données, ou à des plages d’adresses IP spécifiques du service (« mise en liste verte »).
ms.topic: article
ms.date: 05/18/2020
ms.openlocfilehash: b9ecd5f802176cdc6881294f5dedefd3dd467244
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148511"
---
# <a name="configure-rules-to-access-an-azure-container-registry-behind-a-firewall"></a>Configurer des règles pour accéder à un registre de conteneurs Azure derrière un pare-feu

Cet article explique comment configurer des règles sur votre pare-feu pour autoriser l’accès à un registre de conteneurs Azure. Par exemple, un appareil Azure IoT Edge situé derrière un pare-feu ou un serveur proxy peut devoir accéder à un registre de conteneurs pour extraire une image de conteneur. Un serveur verrouillé sur un réseau local peut aussi devoir y accéder pour envoyer (push) une image.

Si, au lieu de cela, vous souhaitez configurer l’accès réseau entrant à un registre de conteneurs uniquement au sein d’un réseau virtuel Azure, consultez [Configurer Azure Private Link pour un registre de conteneurs Azure](container-registry-private-link.md).

## <a name="about-registry-endpoints"></a>À propos des points de terminaison de registre

Pour extraire ou envoyer (push) des images ou d’autres artefacts à un registre de conteneurs Azure, un client (démon Docker, par exemple) doit interagir sur HTTPS avec deux points de terminaison distincts. Pour les clients qui accèdent à un registre derrière un pare-feu, vous devez configurer des règles d’accès pour les deux points de terminaison. Les deux points de terminaison sont atteints sur le port 443.

* **Point de terminaison d'API REST de registre** – Les opérations d'authentification et de gestion de registre sont gérées via le point de terminaison de l’API REST publique du registre. Ce point de terminaison correspond au nom du serveur de connexion du registre. Exemple : `myregistry.azurecr.io`

* **Point de terminaison (de données) de stockage** – Azure [alloue du stockage d’objets blob](container-registry-storage.md) aux comptes de Stockage Azure pour le compte de chaque registre, afin de gérer les données d’images conteneur et d’autres artefacts. Lorsqu’un client accède à des couches d’image dans un registre de conteneurs Azure, il fait des requêtes à l’aide d’un point de terminaison de compte de stockage fourni par le registre.

Si votre registre est [géorépliqué](container-registry-geo-replication.md), un client peut devoir interagir avec les points de terminaison de données dans une région spécifique ou dans plusieurs régions répliquées.

## <a name="allow-access-to-rest-and-data-endpoints"></a>Autoriser l’accès à des points de terminaison REST et de données

* **Point de terminaison REST** – Autorisez l’accès au nom complet du serveur de connexion au registre, `<registry-name>.azurecr.io`, ou à une plage d’adresses IP associée.
* **Point de terminaison (de données) de stockage** – Autorisez l’accès à tous les comptes de stockage Blob Azure à l’aide du caractère générique `*.blob.core.windows.net`, ou d’une plage d’adresses IP associée.
> [!NOTE]
> Azure Container Registry introduit des [points de terminaison de données dédiés](#enable-dedicated-data-endpoints), ce qui vous permet de définir précisément l’étendue des règles de pare-feu client pour votre stockage de registre. Activez éventuellement des points de terminaison de données dans toutes les régions où le registre est localisé ou répliqué, en utilisant le formulaire `<registry-name>.<region>.data.azurecr.io`.

## <a name="allow-access-by-ip-address-range"></a>Autoriser l’accès en fonction de la plage d’adresses IP

Si votre organisation dispose de stratégies pour autoriser l’accès uniquement à des adresses IP ou à des plages d’adresses spécifiques, téléchargez [Plages d’adresses IP et étiquettes des services Azure – Cloud public](https://www.microsoft.com/download/details.aspx?id=56519).

Afin de rechercher les plages d’adresses IP du point de terminaison ACR REST pour lesquelles vous devez autoriser l’accès, recherchez **AzureContainerRegistry** dans le fichier JSON.

> [!IMPORTANT]
> Les plages d’adresses IP des services Azure peuvent changer et des mises à jour sont publiées chaque semaine. Téléchargez régulièrement le fichier JSON et effectuez les mises à jour nécessaires dans vos règles d’accès. Si votre scénario implique la configuration de règles de groupe de sécurité réseau dans un réseau virtuel Azure ou si vous utilisez le Pare-feu Azure, utilisez plutôt l’[étiquette de service](#allow-access-by-service-tag) **AzureContainerRegistry**.
>

### <a name="rest-ip-addresses-for-all-regions"></a>Adresses IP REST pour toutes les régions

```json
{
  "name": "AzureContainerRegistry",
  "id": "AzureContainerRegistry",
  "properties": {
    "changeNumber": 10,
    "region": "",
    "platform": "Azure",
    "systemService": "AzureContainerRegistry",
    "addressPrefixes": [
      "13.66.140.72/29",
    [...]
```

### <a name="rest-ip-addresses-for-a-specific-region"></a>Adresses IP REST pour une région spécifique

Recherchez la région spécifique, par exemple **AzureContainerRegistry.AustraliaEast**.

```json
{
  "name": "AzureContainerRegistry.AustraliaEast",
  "id": "AzureContainerRegistry.AustraliaEast",
  "properties": {
    "changeNumber": 1,
    "region": "australiaeast",
    "platform": "Azure",
    "systemService": "AzureContainerRegistry",
    "addressPrefixes": [
      "13.70.72.136/29",
    [...]
```

### <a name="storage-ip-addresses-for-all-regions"></a>Adresses IP de stockage pour toutes les régions

```json
{
  "name": "Storage",
  "id": "Storage",
  "properties": {
    "changeNumber": 19,
    "region": "",
    "platform": "Azure",
    "systemService": "AzureStorage",
    "addressPrefixes": [
      "13.65.107.32/28",
    [...]
```

### <a name="storage-ip-addresses-for-specific-regions"></a>Adresses IP de stockage pour des régions spécifiques

Recherchez la région spécifique, par exemple **Storage.AustraliaCentral**.

```json
{
  "name": "Storage.AustraliaCentral",
  "id": "Storage.AustraliaCentral",
  "properties": {
    "changeNumber": 1,
    "region": "australiacentral",
    "platform": "Azure",
    "systemService": "AzureStorage",
    "addressPrefixes": [
      "52.239.216.0/23"
    [...]
```

## <a name="allow-access-by-service-tag"></a>Autoriser l’accès par balise de service

Dans un réseau virtuel Azure, utilisez les règles de sécurité réseau pour filtrer le trafic d’une ressource, telle qu’une machine virtuelle, vers un registre de conteneurs. Pour simplifier la création des règles de réseau Azure, utilisez l’**étiquette de service** [AzureContainerRegistry](../virtual-network/network-security-groups-overview.md#service-tags). Une balise de service représente un groupe de préfixes d’adresses IP permettant d'accéder à un service Azure globalement ou par région Azure. La balise est automatiquement mise à jour lorsque les adresses changent. 

Par exemple, créez une règle de groupe de sécurité réseau sortante avec la destination **AzureContainerRegistry** pour autoriser le trafic vers un registre de conteneurs Azure. Pour autoriser l’accès à la balise de service uniquement dans une région spécifique, indiquez la région au format suivant : **AzureContainerRegistry**.[*nom de la région*].

## <a name="enable-dedicated-data-endpoints"></a>Activer des points de terminaison de données dédiés 

> [!WARNING]
> Si vous avez précédemment configuré l’accès du pare-feu client aux points de terminaison `*.blob.core.windows.net` existants, le basculement vers des points de terminaison de données dédiés aura un impact sur la connectivité client, entraînant des échecs de tirage. Pour vous assurer que les clients disposent d’un accès cohérent, ajoutez les nouvelles règles de point de terminaison de données aux règles de pare-feu du client. Lorsque vous avez terminé, activez des points de terminaison de données dédiés pour vos registres à l’aide d’Azure CLI ou d’autres outils.

Les points de terminaison de données dédiés sont une fonctionnalité facultative du niveau de service **Premium** du registre de conteneurs. Pour plus d’informations sur les niveaux de service et les limites de registre, consultez [Niveaux de service Azure Container Registry](container-registry-skus.md). 

Vous pouvez activer des points de terminaison de données dédiés à l’aide du portail Azure ou d’Azure CLI. Les points de terminaison de données suivent un modèle régional, `<registry-name>.<region>.data.azurecr.io`. Dans un registre géo-répliqué, l’activation de points de terminaison de données active des points de terminaison dans toutes les régions de réplica.

### <a name="portal"></a>Portail

Pour activer des points de terminaison de données à l’aide du portail :

1. Accédez à votre registre de conteneurs.
1. Sélectionnez **Mise en réseau** > **Accès public**.
1. Activez la case à cocher **Activer le point de terminaison de données dédié**.
1. Sélectionnez **Enregistrer**.

Le ou les points de terminaison de données s’affichent dans le portail.

:::image type="content" source="media/container-registry-firewall-access-rules/dedicated-data-endpoints-portal.png" alt-text="Points de terminaison de données dédiés dans le portail":::

### <a name="azure-cli"></a>Azure CLI

Pour activer des points de terminaison de données à l’aide d’Azure CLI, utilisez Azure CLI version 2.4.0 ou ultérieure. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).

La commande [az acr update][az-acr-update] active des points de terminaison de données dédiés sur un registre *myregistry*. 

```azurecli
az acr update --name myregistry --data-endpoint-enabled
```

Pour afficher les points de terminaison de données, utilisez la commande [az acr show-endpoints][az-acr-show-endpoints] :

```azurecli
az acr show-endpoints --name myregistry
```

La sortie à des objectifs de démonstration montre deux points de terminaison régionaux

```
{
    "loginServer": "myregistry.azurecr.io",
    "dataEndpoints": [
        {
            "region": "eastus",
            "endpoint": "myregistry.eastus.data.azurecr.io",
        },
        {
            "region": "westus",
            "endpoint": "myregistry.westus.data.azurecr.io",
        }
    ]
}
```

Une fois que vous avez configuré des points de terminaison de données dédiés pour votre registre, vous pouvez activer des règles d’accès du pare-feu client pour les points de terminaison de données. Activez des règles d’accès de point de terminaison de données pour toutes les régions de registre requises.

## <a name="configure-client-firewall-rules-for-mcr"></a>Configurer les règles de pare-feu client pour MCR

Si vous devez accéder à Microsoft Container Registry (MCR) derrière un pare-feu, consultez les instructions de configuration des [règles de pare-feu du client MCR](https://github.com/microsoft/containerregistry/blob/master/client-firewall-rules.md). MCR est le registre principal pour toutes les images Docker publiées par Microsoft, comme les images de Windows Server.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [meilleures pratiques Azure pour la sécurité réseau](../security/fundamentals/network-best-practices.md)

* En savoir plus sur les [groupes de sécurité](../virtual-network/network-security-groups-overview.md) dans un réseau virtuel Azure

* En savoir plus sur la configuration de [liaison privée](container-registry-private-link.md) pour un registre de conteneurs

* En savoir plus sur les [points de terminaison de données dédiés](https://azure.microsoft.com/blog/azure-container-registry-mitigating-data-exfiltration-with-dedicated-data-endpoints/) pour Azure Container Registry



<!-- IMAGES -->

<!-- LINKS - External -->

<!-- LINKS - Internal -->

[az-acr-update]: /cli/azure/acr#az-acr-update
[az-acr-show-endpoints]: /cli/azure/acr#az-acr-show-endpoints