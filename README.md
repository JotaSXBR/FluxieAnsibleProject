# Projeto Ansible para Provisionamento Seguro de Servidor Debian/Ubuntu

Este projeto utiliza Ansible para automatizar a configuração e o hardening (reforço de segurança) de um servidor Debian ou Ubuntu, preparando-o para um ambiente de produção.

## Requisitos

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

# 3. (Opcional, mas recomendado) Mude para o novo usuário para criar o diretório .ssh
su - deploy

# 4. Crie o diretório para as chaves SSH autorizadas
mkdir ~/.ssh
chmod 700 ~/.ssh

# 5. Crie o arquivo authorized_keys
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# 6. Saia da sessão do usuário 'deploy'
exit
```

### 3. Configuração do Acesso SSH (Recomendado: Chaves SSH)

Para uma conexão segura e sem a necessidade de digitar senhas, configure o acesso para o usuário `deploy` usando sua chave SSH pública.

Copie sua chave SSH pública (geralmente `~/.ssh/id_rsa.pub` na sua máquina local) para o arquivo `/home/deploy/.ssh/authorized_keys` no servidor. Você pode usar o comando `ssh-copy-id`:

```bash
# Substitua 'user@seu_servidor_ip' pelo seu usuário de acesso inicial
ssh-copy-id -i ~/.ssh/id_rsa.pub deploy@seu_servidor_ip
```

## Como Executar o Playbook

Após concluir os pré-requisitos, você pode executar o playbook Ansible a partir da sua máquina local.

1.  **Atualize o Inventário:**
    Certifique-se de que o arquivo `inventory.ini` contém o endereço IP correto do seu servidor.

2.  **Execute o Comando Ansible:**
    Não é mais necessário especificar o usuário `root` ou pedir senha, pois a conexão será feita com o usuário `deploy` através da sua chave SSH.

    ```bash
    ansible-playbook -i inventory.ini playbook.yml
    ```

O Ansible se conectará como `deploy` e usará `sudo` (`become: yes`) para executar as tarefas que exigem privilégios de administrador. 