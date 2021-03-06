---
title: Configurar dispositivos do Azure IoT Edge para proxies de rede | Microsoft Docs
description: Como configurar o tempo de execução do Azure IoT Edge e quaisquer módulos de IoT Edge voltados para a Internet para comunicação por meio de um servidor proxy.
author: kgremban
manager: ''
ms.author: kgremban
ms.date: 09/13/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: cf8f6197c65b0169e2bc61f46ab4a22f212512a6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996716"
---
# <a name="configure-an-iot-edge-device-to-communicate-through-a-proxy-server"></a>Configurar um dispositivo IoT Edge para se comunicar por meio de um servidor proxy

Os dispositivos IoT Edge enviam solicitações HTTPS para se comunicarem com o Hub IoT. Se o seu dispositivo estiver conectado a uma rede que usa um servidor proxy, você precisará configurar o tempo de execução do IoT Edge para se comunicar através do servidor. Os servidores proxy também podem afetar os módulos individuais do IoT Edge se eles fizerem solicitações HTTP ou HTTPS que não são roteadas pelo hub do Edge. 

Configurar um dispositivo IoT Edge para funcionar com um servidor proxy segue estas etapas básicas: 

1. Configure o daemon do Docker e o daemon do IoT Edge no seu dispositivo para usar o servidor proxy.
2. Configure as propriedades edgeAgent no arquivo config.yaml no seu dispositivo.
3. Definir variáveis de ambiente para o tempo de execução do IoT Edge e outros módulos do IoT Edge no manifesto de implantação. 

## <a name="configure-the-daemons"></a>Configurar os daemons

Os daemons Docker e IoT Edge em execução no dispositivo IoT Edge precisam ser configurados para usar o servidor proxy. O daemon do Docker faz solicitações da Web para extrair imagens de contêiner de registros de contêiner. O daemon do IoT Edge faz solicitações da Web para se comunicar com o Hub IoT.

### <a name="docker-daemon"></a>Daemon do docker

Consulte a documentação do Docker para configurar o daemon do Docker com variáveis de ambiente. A maioria dos registros de contêiner (incluindo DockerHub e registros de contêiner do Azure) oferecem suporte a solicitações HTTPS, portanto, é a variável que você deve definir **HTTPS_PROXY**. Se você está obtendo imagens de um registro que não dão suporte ao protocolo TLS (TLS) e em seguida, você poderá deve definir a **HTTP_PROXY**. 

Escolha o artigo que se aplica à sua versão do Docker: 

* [Docker](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)
* [Docker para Windows](https://docs.docker.com/docker-for-windows/#proxies)

### <a name="iot-edge-daemon"></a>Daemon do IoT Edge

O daemon do IoT Edge é configurado de maneira semelhante ao daemon do Docker. Todas as solicitações que o IoT Edge envia ao Hub IoT usam HTTPS. Use as etapas a seguir para definir uma variável de ambiente para o serviço, com base em seu sistema operacional. 

#### <a name="linux"></a>Linux

Abra um editor no terminal para configurar o daemon do IoT Edge. 

```bash
sudo systemctl edit iotedge
```

Insira o texto a seguir, substituindo  **\<URL do proxy >** com seu endereço do servidor proxy e a porta. Em seguida, salve e saia. 

```text
[Service]
Environment="https_proxy=<proxy URL>"
```

Atualize o gerenciador de serviços para pegar a nova configuração do iotedge.

```bash
sudo systemctl daemon-reload
```

Reinicie o IoT Edge para que as alterações entrem em vigor.

```bash
sudo systemctl restart iotedge
```

Verifique se sua variável de ambiente foi criada e se a nova configuração foi carregada. 

```bash
systemctl show --property=Environment iotedge
```

#### <a name="windows"></a>Windows

Abra uma janela do PowerShell como administrador e execute o seguinte comando para editar o registro com a nova variável de ambiente. Substitua **\<URL do proxy>** pelo endereço e porta do servidor proxy.
 

```powershell
reg add HKLM\SYSTEM\CurrentControlSet\Services\iotedge /v Environment /t REG_MULTI_SZ /d https_proxy=<proxy URL>
```

Reinicie o IoT Edge para que as alterações entrem em vigor.

```powershell
Restart-Service iotedge
```

## <a name="configure-the-edge-agent"></a>Configurar o agente do Edge

O agente Edge é o primeiro módulo a ser iniciado em qualquer dispositivo IoT Edge. É iniciado pela primeira vez com base nas informações do arquivo config.yaml do IoT Edge. Em seguida, o agente do Edge se conecta ao Hub IoT para recuperar os manifestos de implantação, que declaram quais outros módulos devem ser implantados no dispositivo.

Abra o arquivo config.yaml no seu dispositivo IoT Edge. Nos sistemas Linux, esse arquivo está localizado em **/etc/iotedge/config.yaml**. Nos sistemas Windows, esse arquivo está localizado em **C: \ ProgramData \ iotedge \ config.yaml**. O arquivo de configuração é protegido, você precisa de privilégios administrativos para acessá-lo. Em sistemas Linux, isso significa usar o `sudo` comando antes de abrir o arquivo em seu editor de texto preferido. No Windows, isso significa abrir um editor de texto como o Bloco de Notas para ser executado como administrador e, em seguida, abrir o arquivo. 

No arquivo config.yaml, localize a seção **Especificação do módulo Edge Agent**. A definição do agente de Borda inclui um parâmetro **env**, no qual é possível incluir variáveis de ambiente. 

![definição de edgeAgent](./media/how-to-configure-proxy-support/edgeagent-unedited.png)

Remova as chaves que são espaços reservados para o parâmetro env e inclua a nova variável em uma nova linha. Lembre-se de que os recuos em YAML são dois espaços. 

```yaml
https_proxy: "<proxy URL>"
```

O tempo de execução do IoT Edge usa o AMQP por padrão para falar com o IoT Hub. Alguns servidores proxy bloqueiam as portas AMQP. Se for esse o caso, você também precisará configurar o edgeAgent para usar o AMQP em WebSocket. Adicione uma segunda variável de ambiente.

```yaml
UpstreamProtocol: "AmqpWs"
```

![definição de edgeAgent com variáveis de ambiente](./media/how-to-configure-proxy-support/edgeagent-edited.png)

Salve as alterações no config. YAML e feche o editor. Reinicie o IoT Edge para que as alterações entrem em vigor. 

* Linux: 

   ```bash
   sudo systemctl restart iotedge
   ```

* Windows:

   ```powershell
   Restart-Service iotedge
   ```

## <a name="configure-deployment-manifests"></a>Configurar manifestos de implantação  

Depois que o dispositivo IoT Edge estiver configurado para funcionar com o servidor proxy, você também precisará declarar as variáveis de ambiente em todos os manifestos de implantação futuros. Os dois módulos de tempo de execução, edgeAgent e edgeHub, devem sempre ter o servidor proxy configurado para manter a comunicação com o Hub IoT. Você pode configurar qualquer módulo do IoT Edge para se comunicar por meio de um servidor proxy, mas não é necessário para os módulos que roteiam suas mensagens por meio do edgeHub ou que se comunicam apenas com outros módulos no dispositivo. 

Você pode criar manifestos de implantação usando o portal do Azure ou manualmente editando um arquivo JSON. 

### <a name="azure-portal"></a>Portal do Azure

Quando você usa o assistente **Configurar módulos** para criar implantações para dispositivos IoT Edge, cada módulo tem uma seção **Variáveis de Ambiente** que você pode usar para configurar conexões do servidor proxy. 

Para configurar o agente de Borda e os módulos do hub de Borda, selecione **Definir configurações avançadas do Edge Runtime** na primeira etapa do assistente. 

![Definir configurações avançadas de tempo de execução do Edge](./media/how-to-configure-proxy-support/configure-runtime.png)

Adicione a variável de ambiente **https_proxy** às definições do módulo de agente Edge e do hub de borda. Se você incluiu a variável de ambiente **UpstreamProtocol** no arquivo config.yaml em seu dispositivo IoT Edge, inclua isso na definição do módulo do agente de Borda também. 

![Configurar variáveis de ambiente](./media/how-to-configure-proxy-support/edgehub-environmentvar.png)

Todos os outros módulos adicionados a um manifesto de implantação seguem o mesmo padrão. Na página em que você define o nome e a imagem do módulo, há uma seção de variáveis de ambiente.

### <a name="json-deployment-manifest-files"></a>Arquivos de manifesto de implantação de JSON

Se você criar implantações para dispositivos IoT Edge usando os modelos no Visual Studio Code ou criando manualmente arquivos JSON, poderá adicionar as variáveis de ambiente diretamente a cada definição de módulo. 

Use o seguinte formato JSON: 

```json
"env": {
    "https_proxy": {
        "value": "<proxy URL>"
    }
}
```

Com as variáveis de ambiente incluídas, sua definição de módulo deve ser semelhante ao seguinte exemplo edgeHub:

```json
"edgeHub": {
    "type": "docker",
    "settings": {
        "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
        "createOptions": ""
    },
    "env": {
        "https_proxy": {
            "value": "https://proxy.example.com:3128"
        }
    },
    "status": "running",
    "restartPolicy": "always"
}
```

Se você incluiu os **UpstreamProtocol** variável de ambiente no arquivo confige.yaml em seu dispositivo IoT Edge, adicioná-lo para a definição de módulo do agente do Edge muito. 

```json
"env": {
    "https_proxy": {
        "value": "<proxy URL"
    },
    "UpstreamProtocol": {
        "value": "AmqpWs"
    }
}
```

## <a name="next-steps"></a>Próximas etapas

Saiba mais sobre as funções do tempo de execução do [IoT Edge](iot-edge-runtime.md).

Solucione problemas de erros de instalação e configuração com [problemas comuns e resoluções para o Azure IoT Edge](troubleshoot.md)

