# Projeto Ansible para Provisionamento Seguro de Servidor Debian/Ubuntu

## Visão Geral do Projeto

Este projeto utiliza Ansible para automatizar a configuração e o hardening (reforço de segurança) de um servidor Debian ou Ubuntu. Ele implanta uma stack de observabilidade moderna e robusta, incluindo:

*   **Traefik:** Como reverse proxy e load balancer, com geração automática de certificados SSL via Let's Encrypt.
*   **Docker:** Para containerização das aplicações.
*   **Portainer:** Para gerenciamento facilitado dos containers Docker.
*   **Prometheus:** Para coleta e armazenamento de métricas.
*   **Grafana:** Para visualização de métricas através de dashboards.
*   **Jaeger:** Para tracing distribuído.

O objetivo é preparar um servidor mínimo e seguro para produção, com foco em boas práticas e utilizando uma estrutura de roles Ansible para modularidade e fácil manutenção.

## Arquitetura Visual (Sugestão)

*Seria muito benéfico adicionar um diagrama aqui mostrando como os diferentes componentes (Traefik, Docker, Portainer, Prometheus, Grafana, Jaeger) interagem entre si e com o tráfego externo. Isso ajudaria os usuários a entender rapidamente a topologia do sistema implantado.*

*Exemplo de componentes a incluir no diagrama:*
*   *Internet / Usuários*
*   *DNS*
*   *Servidor Host (Debian/Ubuntu)*
*   *Traefik (como ponto de entrada, lidando com SSL)*
*   *Rede Docker para os serviços*
*   *Containers: Portainer, Prometheus, Grafana, Jaeger, e potencialmente outras aplicações do usuário.*
*   *Interações: Prometheus coletando métricas do Traefik e outros containers, Grafana buscando dados do Prometheus.*

## Índice

*   [Visão Geral do Projeto](#visão-geral-do-projeto)
*   [Arquitetura Visual (Sugestão)](#arquitetura-visual-sugestão)
*   [Estrutura do Projeto](#estrutura-do-projeto)
*   [Configuração do Ambiente Local (Sua Máquina)](#configuração-do-ambiente-local-sua-máquina)
    *   [Instalar Python e Pip](#1-instalar-python-e-pip)
    *   [Instalar o Ansible](#2-instalar-o-ansible)
    *   [Instalar Coleções do Ansible Galaxy](#3-instalar-coleções-do-ansible-galaxy)
    *   [Como Atualizar o Ambiente](#como-atualizar-o-ambiente)
*   [Requisitos do Servidor Remoto](#requisitos-do-servidor-remoto)
    *   [Acesso Inicial ao Servidor](#1-acesso-inicial-ao-servidor)
    *   [Criação do Usuário `deploy`](#2-criação-do-usuário-deploy)
    *   [Configuração do Acesso SSH](#3-configuração-do-acesso-ssh-recomendado-chaves-ssh)
*   [Como Executar o Playbook](#como-executar-o-playbook)
*   [Customização](#customização)
*   [Pós-Instalação: Acessando os Serviços](#pós-instalação-acessando-os-serviços)
*   [Acesso SFTP aos Volumes](#acesso-sftp-aos-volumes)
*   [Considerações de Segurança](#considerações-de-segurança)
*   [Contribuindo](#contribuindo)
*   [Licença](#licença)

## Estrutura do Projeto

```
.
├── README.md                # Este arquivo
├── grafana/                 # Configurações e dashboards para o Grafana
│   ├── dashboards/
│   │   └── traefik.json
│   └── provisioning/
│       ├── dashboards/
│       │   └── default.yml
│       └── datasources/
│           └── prometheus.yml
├── inventory.ini            # Arquivo de inventário Ansible (define os hosts)
├── playbook.yml             # Playbook principal do Ansible
├── requirements.yml         # Dependências de coleções Ansible Galaxy
├── roles/                   # Diretório contendo os roles Ansible
│   ├── common/              # Role para configurações comuns do servidor (segurança, pacotes básicos)
│   ├── docker/              # Role para instalação e configuração do Docker e Docker Compose
│   └── observability/       # Role para deploy da stack de observabilidade (Traefik, Prometheus, Grafana, etc.)
└── templates/               # Templates Jinja2 utilizados pelo Ansible
    ├── docker-compose.yml.j2  # Template para o arquivo docker-compose.yml
    └── prometheus.yml.j2    # Template para o arquivo de configuração do Prometheus
```

*   **`playbook.yml`**: O playbook principal que orquestra todas as tarefas e roles.
*   **`inventory.ini`**: Define os servidores onde o playbook será executado. Você **precisa** editar este arquivo com o IP ou hostname do seu servidor.
*   **`requirements.yml`**: Lista as coleções Ansible Galaxy necessárias para o projeto.
*   **`roles/`**: Contém os diferentes roles que modularizam as tarefas:
    *   **`common`**: Configurações base do servidor, como usuários, firewall, atualizações e pacotes essenciais.
    *   **`docker`**: Instala o Docker e o Docker Compose.
    *   **`observability`**: Implanta os serviços da stack de observabilidade (Traefik, Portainer, Prometheus, Grafana, Jaeger) usando Docker Compose.
*   **`templates/`**: Arquivos de template (Jinja2) que são processados pelo Ansible para gerar arquivos de configuração.
    *   **`docker-compose.yml.j2`**: Template principal para a configuração dos serviços Docker. As versões das imagens e domínios são definidos como variáveis no `playbook.yml`.
    *   **`prometheus.yml.j2`**: Template para a configuração do Prometheus.
*   **`grafana/`**: Contém arquivos para provisionamento automático de datasources e dashboards no Grafana.

## Configuração do Ambiente Local (Sua Máquina)

Antes de executar o playbook, você precisa ter o Ansible e suas dependências instaladas na sua máquina local.

### 1. Instalar Python e Pip

O Ansible é baseado em Python. Certifique-se de que você tem o Python 3.8 ou superior instalado. Você pode verificar com:
```bash
python3 --version
```
O `pip` (gerenciador de pacotes do Python) geralmente já vem com as versões mais recentes do Python.

### 2. Instalar o Ansible
A forma recomendada de instalar a versão mais recente do Ansible é usando `pip`:
```bash
python3 -m pip install ansible
```

### 3. Instalar Coleções do Ansible Galaxy
Nosso playbook utiliza módulos de coleções da comunidade. A maneira mais fácil de instalar todas as dependências é usando o arquivo `requirements.yml` fornecido:
```bash
ansible-galaxy collection install -r requirements.yml
```
Isso garantirá que você tenha as coleções `community.general` e `community.docker` necessárias.

### Como Atualizar o Ambiente

Para manter seu ambiente atualizado, você pode executar os seguintes comandos:

*   **Para atualizar o Ansible (o motor):**
    ```bash
    python3 -m pip install --upgrade ansible
    ```
*   **Para atualizar as Coleções (módulos):**
    ```bash
    ansible-galaxy collection install -r requirements.yml --upgrade
    ```

Após esses passos, seu ambiente local estará pronto para executar o playbook.

## Requisitos do Servidor Remoto

Antes de executar este playbook, você precisa garantir que os seguintes pré-requisitos foram atendidos no seu servidor de destino.

### 1. Acesso Inicial ao Servidor

Você deve ter acesso inicial ao servidor como `root` ou um usuário com permissões `sudo`. Geralmente, provedores de nuvem fornecem acesso `root` via SSH com chave ou senha.

### 2. Criação do Usuário `deploy`

O playbook foi projetado para ser executado pelo usuário `deploy`, e não por `root`. Portanto, você deve criar este usuário e conceder a ele privilégios `sudo`.

Conecte-se ao seu servidor e execute os seguintes comandos:

```bash
# 1. Crie o novo usuário (você será solicitado a definir uma senha)
adduser deploy

# 2. Adicione o usuário ao grupo 'sudo' para permitir a elevação de privilégios
usermod -aG sudo deploy

# 3. (Opcional, mas recomendado) Crie a estrutura para chaves SSH
su - deploy
mkdir ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
exit
```

### 3. Configuração do Acesso SSH (Recomendado: Chaves SSH)

Para uma conexão segura e sem a necessidade de digitar senhas, configure o acesso para o usuário `deploy` usando sua chave SSH pública.

Se você não possui uma chave SSH, pode criar uma com o comando `ssh-keygen -t rsa -b 4096`.

## Como Executar o Playbook

Após concluir a configuração do ambiente local e os pré-requisitos do servidor remoto, você pode executar o playbook Ansible.

1.  **Atualize o Inventário:**
    Certifique-se de que o arquivo `inventory.ini` contém o endereço IP correto do seu servidor.

2.  **Execute o Comando Ansible:**
    Para máxima segurança e compatibilidade, execute o playbook solicitando as senhas de conexão (SSH) e de privilégio (`sudo`) em tempo de execução. Isso evita armazenar senhas e funciona mesmo que as chaves SSH ou o `sudo` sem senha ainda não estejam configurados.

    ```bash
    ansible-playbook -i inventory.ini playbook.yml --ask-pass --ask-become-pass
    ```

    O Ansible solicitará duas senhas:
    - **SSH password:** A senha do usuário `deploy` para conectar ao servidor.
    - **BECOME password:** A senha de `sudo` do usuário `deploy` (geralmente, é a mesma senha do usuário).

    **Nota:** Após a primeira execução bem-sucedida, o playbook configura o `sudo` sem senha para o usuário `deploy`. Em execuções futuras, a opção `--ask-become-pass` não será mais estritamente necessária. Se você configurou o acesso com chaves SSH (conforme recomendado), a opção `--ask-pass` também pode ser omitida.

O Ansible se conectará como `deploy` e usará `sudo` (`become: yes`) para executar as tarefas que exigem privilégios de administrador.

## Customização

A principal forma de customizar esta implantação é através das variáveis definidas no arquivo `playbook.yml`.

### Variáveis Chave para Customização:

Localizadas na seção `vars` do `playbook.yml`:

```yaml
  vars:
    # --- Variáveis Gerais ---
    deploy_user: deploy
    swap_file_path: /swapfile
    swap_size_mb: "{{ ([1024, (ansible_memtotal_mb / 2) | int, 4096] | sort)[1] }}"

    # --- Variáveis das Aplicações (Imagens Docker) ---
    traefik_version: "v3.1.0"
    portainer_version: "2.20.1"
    prometheus_version: "v2.53.0"
    grafana_version: "11.1.0"
    jaeger_version: "1.57"

    # --- Variáveis de Domínio e Email (TROCAR ESTES VALORES) ---
    traefik_domain: "traefik.example.com"
    portainer_domain: "portainer.example.com"
    prometheus_domain: "prometheus.example.com"
    grafana_domain: "grafana.example.com"
    jaeger_domain: "jaeger.example.com"
    letsencrypt_email: "seu-email@example.com"
```

*   **Versões das Aplicações:**
    *   `traefik_version`, `portainer_version`, `prometheus_version`, `grafana_version`, `jaeger_version`: Permitem especificar as versões (tags) das imagens Docker para cada serviço. Recomenda-se verificar a compatibilidade entre versões antes de alterar.

*   **Domínios e Email para SSL:**
    *   `traefik_domain`, `portainer_domain`, `prometheus_domain`, `grafana_domain`, `jaeger_domain`: Defina os subdomínios que serão usados para acessar cada serviço. **É crucial que você configure os registros DNS correspondentes para apontar para o IP do seu servidor.**
    *   `letsencrypt_email`: O endereço de e-mail usado pelo Let's Encrypt para notificações sobre seus certificados SSL (por exemplo, avisos de expiração). **Substitua `"seu-email@example.com"` pelo seu e-mail real.**

*   **Outras Variáveis:**
    *   `deploy_user`: Nome do usuário de deploy no servidor remoto.
    *   `swap_size_mb`: Controla o tamanho do arquivo de swap. Por padrão, é configurado para 50% da RAM, com um mínimo de 1GB e máximo de 4GB.

Após modificar estas variáveis no `playbook.yml`, basta executar o playbook novamente para aplicar as alterações.

## Pós-Instalação: Acessando os Serviços

Após a execução bem-sucedida do playbook, os seguintes serviços estarão disponíveis:

*   **Traefik Dashboard:** `https://traefik.example.com` (protegido por senha)
*   **Portainer:** `https://portainer.example.com`
*   **Prometheus:** `https://prometheus.example.com` (protegido por senha)
*   **Grafana:** `https://grafana.example.com` (protegido por senha, login padrão: `admin`/`admin`)
*   **Jaeger:** `https://jaeger.example.com` (protegido por senha)

**Passos importantes:**

1.  **Configure seu DNS:** Antes de acessar, você **precisa** configurar os registros DNS dos seus domínios para apontar para o endereço IP do seu servidor.
2.  **Aguarde o SSL:** O Traefik irá gerar automaticamente os certificados SSL com Let's Encrypt no primeiro acesso. Isso pode levar um minuto.
3.  **Primeiro Login nos Serviços:**
    *   **Portainer:** Ao acessar pela primeira vez, você será solicitado a criar um usuário administrador.
    *   **Grafana:** O login inicial é `admin` com a senha `admin`. Você será solicitado a trocá-la no primeiro acesso.

## Acesso SFTP aos Volumes

Conforme solicitado, os diretórios de configuração dos serviços estão localizados em `/opt` e pertencem ao usuário `deploy`. Você pode acessá-los via SFTP para gerenciar as configurações:

*   **Configuração do Traefik:** `/opt/traefik/config/`
*   **Configuração do Prometheus:** `/opt/prometheus/config/`
*   **Dados do Portainer:** `/opt/portainer/data/`
*   **Dados do Grafana:** `/opt/grafana/data/`
*   **Arquivo Docker Compose:** `/opt/docker-compose.yml`

Lembre-se que a "fonte da verdade" é o seu projeto Ansible. Se você editar o `docker-compose.yml` ou `prometheus.yml` manualmente no servidor, o Ansible irá sobrescrevê-los na próxima vez que o playbook for executado.

## Considerações de Segurança

Este playbook implementa diversas medidas de segurança e hardening através da role `common`. É importante estar ciente delas:

*   **Atualizações Automáticas e Manuais:** O sistema é atualizado durante a execução do playbook, e o pacote `unattended-upgrades` é instalado para atualizações de segurança automáticas.
*   **Firewall (UFW):** O UFW (Uncomplicated Firewall) é configurado para bloquear todas as conexões de entrada por padrão, permitindo explicitamente apenas:
    *   SSH (porta 22)
    *   HTTP (porta 80)
    *   HTTPS (porta 443)
*   **Proteção contra Brute-Force (Fail2Ban):** O Fail2Ban é configurado para monitorar logs do SSH e banir IPs que apresentarem comportamento suspeito (ex: múltiplas tentativas de login falhas).
*   **Hardening do SSH:**
    *   Login como `root` é desabilitado.
    *   Autenticação por senha é mantida (para o primeiro acesso e setup), mas o acesso é restrito ao `deploy_user`.
    *   Número máximo de tentativas de autenticação é limitado.
    *   Um banner de aviso é exibido antes do login.
*   **Hardening do Kernel:** Diversas configurações `sysctl` são aplicadas para proteger o kernel contra ataques comuns (ex: IP spoofing, source routing, SYN floods).
*   **Minimização da Superfície de Ataque:** Pacotes desnecessários e potencialmente inseguros (como `telnetd`, `rsh-server`) são removidos.
*   **Usuário de Deploy Dedicado:** Todas as operações são feitas através de um usuário `deploy` com privilégios `sudo` (configurável para NOPASSWD após o primeiro run), em vez de usar `root` diretamente para operações do Ansible.
*   **Auditoria:** O pacote `auditd` é instalado, permitindo um registro detalhado das atividades do sistema.

**Recomendações Adicionais (Não cobertas por este playbook):**

*   **Backups Regulares:** Configure uma rotina de backup para os dados das aplicações e configurações críticas.
*   **Monitoramento de Logs:** Centralize e monitore os logs do sistema e das aplicações.
*   **Autenticação de Dois Fatores (2FA):** Considere adicionar 2FA para o acesso SSH e para os serviços expostos, se aplicável.
*   **Segurança de Aplicação:** Garanta que as aplicações que você implanta sobre esta infraestrutura também sigam boas práticas de segurança.
*   **Gerenciamento de Segredos:** Para senhas e chaves de API usadas em produção, considere o uso de uma solução de gerenciamento de segredos como HashiCorp Vault ou Ansible Vault.

## Contribuindo

Contribuições são bem-vindas! Se você tem sugestões para melhorar este projeto, sinta-se à vontade para abrir uma *issue* ou enviar um *pull request*.

Possíveis áreas para contribuição:
*   Adicionar mais opções de configuração.
*   Melhorar a documentação.
*   Implementar testes para os roles Ansible.
*   Adicionar suporte para outros serviços ou configurações.

## Licença

Recomenda-se adicionar um arquivo `LICENSE` ao projeto para definir como outros podem usá-lo ou contribuir para ele. Escolha uma licença open source apropriada (ex: MIT, Apache 2.0, GPL) de acordo com seus objetivos para o projeto.

Você pode adicionar um placeholder como:

```markdown
## Licença

Este projeto é licenciado sob a Licença MIT - veja o arquivo [LICENSE.md](LICENSE.md) para detalhes.
```
(E então criar um arquivo `LICENSE.md` com o texto da licença escolhida).
