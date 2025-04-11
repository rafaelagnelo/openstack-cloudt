## Objetivo

O presente relatório tem como alvo abstrair os seguintes conhecimentos:

•	Entendimento de conceitos básicos sobre gerenciamento de hardware (Bare Metal) e MaaS (Metal as a Service).

•	Entendimento de conceitos básicos sobre redes de computadores.


### Infraestrutura
Instalação do Ubuntu:

<!-- termynal -->


``` bash
sudo snap install maas --channel=3.5/Stable
```

![Tela do Dashboard do MAAS](./maas.png)
/// caption
Dashboard do MAAS
///

Conforme ilustrado acima, a tela inicial do MAAS apresenta um dashboard com informações sobre o estado atual dos servidores gerenciados. O dashboard é composto por diversos painéis, cada um exibindo informações sobre um aspecto específico do ambiente gerenciado. Os painéis podem ser configurados e personalizados de acordo com as necessidades do usuário.

![instalacao01-maas](image.png)

## App

### Tarefa 1

![alt text](imagem2.png)

```mermaid
architecture-beta
    group api(cloud)[API]

    service db(database)[Database] in api
    service disk1(disk)[Storage] in api
    service disk2(disk)[Storage] in api
    service server(server)[Server] in api

    db:L -- R:server
    disk1:T -- B:server
    disk2:T -- B:db
```

[Mermaid](https://mermaid.js.org/syntax/architecture.html){:target="_blank"}


### Tarefa 2

Dashboard do Maas:
![alt text](image-3.png)

Aba com imagens sincronizadas:

![alt text](image-4.png)

Aba da máquina 5 mostrando os testes de hardware e commissioning com Status "OK"

![alt text](image-5.png)


### Tarefa 3

1 - Print da tela do Dashboard do MAAS com as 2 Maquinas e seus respectivos IPs:

![alt text](image-1.png)

2 -  Print da aplicacao Django, provando que voce está conectado ao server:

![alt text](image-2.png)

3 - Configuração com deploy:

![alt text](image-3.png)

### Tarefa 4

1. De um print da tela do Dashboard do MAAS com as 3 Maquinas e seus respectivos IPs.

![alt text](image-6.png)

2. Aplicacao Django, provando que voce está conectado ao server2 

 ![alt text](image-7.png)


3. Explique qual diferenca entre instalar manualmente a aplicacao Django e utilizando o Ansible.

![alt text](image-8.png)

### Acerca do funcionamento do proxy: 



## Questionário, Projeto ou Plano

Esse seção deve ser preenchida apenas se houver demanda do roteiro.

## Discussões

Quais as dificuldades encontradas? O que foi mais fácil? O que foi mais difícil?

## Conclusão

O que foi possível concluir com a realização do roteiro?
