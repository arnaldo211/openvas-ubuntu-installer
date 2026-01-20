# Guia de Instalação do OpenVAS (GVM) com Docker Compose no Ubuntu 24.04

Este guia detalha o método oficial e mais confiável para instalar o Greenbone Vulnerability Manager (GVM/OpenVAS) usando o Docker Compose. Esta abordagem evita os problemas complexos de compilação e dependências.

### Pré-requisitos

*   Um sistema Ubuntu 24.04 (ou outra distribuição Linux moderna).
*   Acesso à internet com permissão para baixar imagens Docker (pode exigir liberação de firewall para `registry.community.greenbone.net`).
*   Acesso `sudo` ao sistema.

### Passo 1: Limpeza Completa (Opcional, mas Recomendado)

Se você já tentou instalar o GVM por outros métodos, é crucial remover todos os vestígios para evitar conflitos.

```bash
# Parar e remover quaisquer serviços antigos
sudo systemctl stop gsad gvmd ospd-openvas openvasd postgresql redis-server &>/dev/null
sudo apt-get purge -y gvm greenbone-security-assistant gsad gvmd notus-scanner openvas-scanner ospd-openvas
sudo apt-get autoremove -y
```
```
# Remover diretórios de dados e logs antigos
sudo rm -rf /var/lib/gvm /var/lib/openvas /var/lib/notus /var/log/gvm
```
```
# Remover contêineres Docker antigos, se existirem
docker rm -f gvmd &>/dev/null
```

### Passo 2: Instalar o Docker e o Docker Compose

O GVM será executado dentro de contêineres Docker, que fornecem um ambiente isolado e pré-configurado.

1.  **Atualize a lista de pacotes:**
    ```bash
    sudo apt-get update
    ```

2.  **Instale o Docker e suas dependências:**
    ```bash
    sudo apt-get install -y docker.io
    ```

3.  **Inicie e habilite o serviço do Docker:**
    Isso garante que o Docker inicie automaticamente com o sistema.
    ```bash
    sudo systemctl start docker
    sudo systemctl enable docker
    ```

4.  **Instale o Docker Compose (v2):**
    Esta ferramenta orquestra múltiplos contêineres, o que é essencial para o GVM.
    ```bash
    sudo apt-get install -y docker-compose-v2
    ```

5.  **Adicione seu usuário ao grupo `docker`:**
    Isso permite que você execute comandos Docker sem precisar usar `sudo` todas as vezes.
    ```bash
    sudo usermod -aG docker $USER
    ```

6.  **Aplique as alterações de grupo:**
    **IMPORTANTE:** Você precisa fazer **logout e login novamente** na sua sessão de terminal (ou reiniciar a máquina) para que a adição ao grupo `docker` tenha efeito.

### Passo 3: Baixar e Iniciar os Contêineres do GVM

Agora que o ambiente Docker está pronto, vamos usar o arquivo de configuração oficial da Greenbone para implantar a aplicação.

1.  **Crie um diretório para o seu projeto GVM:**
    É uma boa prática manter os arquivos de configuração organizados.
    ```bash
    mkdir ~/greenbone-gvm
    cd ~/greenbone-gvm
    ```

2.  **Baixe o arquivo `docker-compose.yml` oficial:**
    Este arquivo contém as instruções para o Docker sobre quais contêineres baixar e como configurá-los para se comunicarem.
    ```bash
    curl -f -L https://greenbone.github.io/docs/latest/_static/docker-compose-22.4.yml -o docker-compose.yml
    ```

3.  **Inicie a pilha de contêineres:**
    Este comando é o coração do processo. Ele irá:
    *   Baixar todas as imagens necessárias do registro da Greenbone.
    *   Criar e iniciar os contêineres em segundo plano (`-d` ).
    *   Configurar a rede interna para que eles se comuniquem.

    ```bash
    docker compose -f docker-compose.yml -p greenbone-community-edition up -d
    ```
    **Nota:** O primeiro download pode demorar bastante. Se você encontrar um erro de `timeout`, como vimos em nossa depuração, simplesmente execute o comando novamente. O Docker é inteligente e continuará de onde parou.

### Passo 4: Sincronização Inicial e Acesso

Após o comando `docker compose up` terminar, os contêineres estarão rodando, mas o GVM precisa de tempo para realizar sua primeira sincronização de feeds de segurança.

1.  **Aguarde a Sincronização:** A primeira inicialização pode levar de **30 a 90 minutos**. Durante esse tempo, o GVM está baixando e processando dezenas de milhares de testes de vulnerabilidade. Seja paciente.

2.  **Monitore o Progresso (Opcional):**
    Você pode acompanhar os logs do contêiner `gvmd` para ver o que ele está fazendo.
    ```bash
    docker logs -f greenbone-community-edition-gvmd-1
    ```
    Procure por mensagens sobre "updating" ou "loading". Pressione `Ctrl + C` para sair dos logs.

3.  **Acesse a Interface Web:**
    Após o período de espera, abra seu navegador e acesse a interface web. Por padrão, ela estará disponível em:
    *   **URL:** `http://SEU_IP_DA_MAQUINA:9392`
      *(Substitua `SEU_IP_DA_MAQUINA` pelo endereço IP da sua máquina virtual. Você pode encontrá-lo com o comando `ip a` )*.

4.  **Faça o Login:**
    As credenciais padrão para a primeira vez são:
    *   **Usuário:** `admin`
    *   **Senha:** `admin`

    É altamente recomendável alterar a senha do administrador imediatamente após o primeiro login em **Administration > Users**.

### Comandos Úteis de Gerenciamento

*   **Para parar todos os serviços do GVM:**
    (Execute de dentro do diretório `~/greenbone-gvm`)
    ```bash
    docker compose down
    ```

*   **Para iniciar todos os serviços do GVM novamente:**
    (Execute de dentro do diretório `~/greenbone-gvm`)
    ```bash
    docker compose up -d
    ```

*   **Para ver o status de todos os contêineres:**
    ```bash
    docker ps
    ```
