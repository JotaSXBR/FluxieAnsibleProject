# Projeto Ansible para Provisionamento Seguro de Servidor Debian/Ubuntu

Este projeto utiliza Ansible para automatizar a configuração e o hardening (reforço de segurança) de um servidor Debian ou Ubuntu, preparando-o para um ambiente de produção.

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

## Pós-Instalação: Acessando os Serviços

Após a execução bem-sucedida do playbook, os seguintes serviços estarão disponíveis:

*   **Traefik Dashboard:** `https://traefik.example.com`
*   **Portainer:** `https://portainer.example.com`

**Passos importantes:**

1.  **Configure seu DNS:** Antes de acessar, você **precisa** configurar os registros DNS dos seus domínios (neste exemplo, `traefik.example.com` e `portainer.example.com`) para apontar para o endereço IP do seu servidor.
2.  **Aguarde o SSL:** O Traefik irá gerar automaticamente os certificados SSL com Let's Encrypt no primeiro acesso. Isso pode levar um minuto.
3.  **Primeiro Login no Portainer:** Ao acessar o Portainer pela primeira vez, você será solicitado a criar um usuário administrador.

## Acesso SFTP aos Volumes

Conforme solicitado, os diretórios de configuração dos serviços estão localizados em `/opt` e pertencem ao usuário `deploy`. Você pode acessá-los via SFTP para gerenciar as configurações:

*   **Configuração do Traefik:** `/opt/traefik/config/`
*   **Dados do Portainer:** `/opt/portainer/data/`
*   **Arquivo Docker Compose:** `/opt/docker-compose.yml`

Lembre-se que a "fonte da verdade" é o seu projeto Ansible. Se você editar o `docker-compose.yml` manualmente no servidor, o Ansible irá sobrescrevê-lo na próxima vez que o playbook for executado. 