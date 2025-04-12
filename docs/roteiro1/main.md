# Roteiro 1 - MAAS
## 1. Objetivo

O presente relatório tem como alvo abstrair os seguintes conhecimentos:

•	Entendimento de conceitos básicos sobre gerenciamento de hardware (Bare Metal) e MaaS (Metal as a Service).

•	Entendimento de conceitos básicos sobre redes de computadores.


## 2. Infraestrutura

O kit utilizado para esta atividade foi composto pelos seguintes itens:

- 1 NUC (main) com 10 GB de RAM e 1 SSD de 120 GB.

- 1 NUC (server1) com 12 GB de RAM e 1 SSD de 120 GB.

- 1 NUC (server2) com 16 GB de RAM e 2 SSDs de 120 GB.

- 3 NUCs (server3, server4, server5) com 32 GB de RAM e 2 SSDs de 120 GB.

- 1 Switch D-Link DSG-1210-28 com 28 portas.

- 1 Roteador TP-Link TL-R470T+.

Para a relização desse roteiro contamos com um ponto de rede (cabo) próprio conectado à rede do Insper e um IP de entrada configurado diretamente no roteador.




### 2.1 Instalação do Ubuntu Server na máquina principal (main)

A primeira etapa da configuração envolveu a instalação do sistema operacional Ubuntu Server 22.04 LTS na máquina principal (NUC main). O procedimento foi realizado via pendrive bootável, configurado previamente com a imagem ISO do Ubuntu.

As definições aplicadas durante a instalação foram:
Instalação do Ubuntu:

Hostname: main

Usuário: cloud

Senha: cloudt

### 2.2 Instalação do MAAS

Com o sistema operacional instalado, foi realizada a instalação do MAAS (Metal as a Service) na versão estável 3.5.3. Antes disso, foram feitos testes de conectividade com os comandos ping para verificar roteamento e resolução de DNS.

Com a conectividade confirmada, os seguintes comandos foram executados:

<!-- termynal -->


``` bash
$ sudo apt update && sudo apt upgrade -y
$ sudo snap install maas --channel=3.5/stable
$ sudo snap install maas-test-db
```


### 2.3 Inicialização do MAAS e criação do administrador

A inicialização do MAAS foi feita com os seguintes comandos:

``` bash
$ sudo maas init region+rack --maas-url http://172.16.0.3:5240/MAAS --database-uri maas-test-db:///
$ sudo maas createadmin
```
Durante a criação do administrador, utilizamos o login cloud, a senha padrão da disciplina e deixamos o campo de chave SSH vazio.

Depois, geramos um par de chaves SSH com senha vazia:

``` bash
$ ssh-keygen -t rsa
$ cat ./.ssh/id_rsa.pub
```

A chave pública gerada foi copiada e registrada no dashboard do MAAS.

### 2.4 Configuração inicial do Dashboard do MAAS e configuração do DHCP

Acessamos o dashboard do MAAS via navegador em http://172.16.0.3:5240/MAAS.

As configurações aplicadas nessa etapa incluíram: Upload da chave SSH gerada; Definição do DNS Forwarder para 172.20.129.131; Importação das imagens do Ubuntu 22.04 LTS e 20.04 LTS; Atualização do parâmetro global net.ifnames=0 em Settings > General.

Também, foi habilitado o serviço de DHCP diretamente no MAAS Controller, com os seguintes ajustes:

- Faixa reservada: de **172.16.11.1** até **172.16.14.255**

- DNS da subrede: **172.20.129.131**

**Obs:**

A integridade do ambiente foi confirmada por meio do painel de controladores do MAAS, onde todos os serviços essenciais, incluindo **regiond**, **rackd** e **dhcpd**, apresentaram status verde.

### 2.5 Comissionamento das máquinas

As NUCs server1 a server5 foram registradas como hosts no MAAS. Todas as máquinas realizaram o boot via PXE e foram comissionadas com sucesso. Após o processo, passaram a apresentar status Ready, com todas as informações de hardware corretamente detectadas.

### 2.6 Criação da OVS bridge (br-ex) e configuração do NAT

Foi criada uma Open vSwitch bridge (OVS) chamada **br-ex** em cada nó de nuvem, utilizando a interface física enp1s0. Essa ponte foi configurada no painel do MAAS, na aba Network de cada máquina. A configuração de **br-ex** é essencial para o funcionamento futuro do OVN e para suportar redes overlay sem exigir múltiplas interfaces físicas por nó.

Finalmente, configuramos um NAT no roteador do kit, liberando acesso externo à máquina principal (main) pela porta 22. Isso permitiu o uso de SSH mesmo fora da rede local.

Também foi criada uma regra de gestão para o endereço 0.0.0.0/0, permitindo acesso remoto à interface de gerenciamento do próprio roteador.

## 3. Aplicação

Fizemos a realização de um deploy manual para uma aplicação simples na nuvem MaaS da nossa dupla.

### 3.1 Ajuste de DNS para deploy Bare Metal

Ajustamos o DNS da nossa rede bare-metal diretamente pelo MAAS. Na aba Subnets, acessamos a subnet 172.16.0.0/20, editamos o campo Subnet summary e substituímos o DNS para o do Insper: **172.20.129.131**.

### 3.2 Deploy manual do banco de dados PostgreSQL no server1

A partir do dashboard do MAAS, realizamos o deploy do Ubuntu 22.04 no server1. Após o deploy, acessamos o terminal via SSH:

``` bash
$ ssh cloud@172.16.15.1
```

No terminal, executamos a instalação do PostgreSQL:

``` bash
$ sudo apt update
$ sudo apt install postgresql postgresql-contrib -y
```

Em seguida, configuramos o banco:

``` bash
$ sudo su - postgres
$ createuser -s cloud -W          
$ createdb -O cloud tasks
```

Alteramos os arquivos de configuração para permitir conexões externas:

``` bash
$ nano /etc/postgresql/14/main/postgresql.conf
# linha: listen_addresses = '*'

$ nano /etc/postgresql/14/main/pg_hba.conf
# linha adicionada: host all all 172.16.0.0/20 trust
```

Por fim, reiniciamos o serviço e liberamos a porta padrão do PostgreSQL:

``` bash
$ sudo ufw allow 5432/tcp
$ sudo systemctl restart postgresql
```

### 3.3 Deploy da aplicação Django no server2 via MAAS CLI

Acessamos o terminal do main e realizamos login no MAAS:

``` bash
$ maas login cloud http://172.16.0.3:5240/MAAS/
```

Solicitamos a alocação do server2:

``` bash
$ maas cloud machines allocate name=server2
```

Com o system_id retornado, iniciamos o deploy:

``` bash
$ maas cloud machine deploy [system_id]
```

Acessamos o server2 via SSH e iniciamos o processo de instalação da aplicação:

``` bash
$ git clone https://github.com/raulikeda/tasks.git
$ cd tasks
$ ./install.sh
$ sudo reboot
```

### 3.4 Acesso externo via túnel SSH

Com o serviço Django rodando na porta 8080 do server2, realizamos o acesso externo usando um túnel SSH a partir do main:

``` bash
$ ssh cloud@10.103.0.X -L 8001:172.16.15.2:8080
```

Abrimos o navegador e acessamos:


``` bash
http://localhost:8001/admin/
```

## Tarefa 2

### 1. Lista de Imagens Importadas

Vemos que a imagem do Ubuntu 22.04 LTS foi baixada e sincronizada com sucesso no ambiente MAAS, com arquitetura amd64 e status Synced. A sincronização é essencial para que os servidores possam ser deployados com essa versão.

![alt text](tarefa2.1..png)

### 2. Estado atual das máquinas no MAAS 