# rabbit-mq-fault-tolerance

A motivação é mostrar como o **RabbitMQ** se comporta em situações de falhas como falhas na comunicação ou queda de algum dos nós, considerando-se um cluster. A idéia também é demonstrar quais os mecanismos podem ser utilizados (configurações no broker, comportamento dos consumidores/publicadores) para ter uma maior tolerância a falhas e avaliabilidade.

## Configuração do Ambiente

Para preparaçãod do ambiente foi feita a criação de 3 máquinas virtuais com o auxílio da ferramenta **Vagrant**.

- manager - 192.168.10.2
- worker1 - 192.168.10.3
- worker2 - 192.168.10.4

### Cluster

Criação do cluster usando o **Docker Swarm** em cada VM.

Utilizando o Docker Swarm, foi feito o deploy de instâncias do **Consul** no cluster para cuidar do _service discovery_ para o **RabbitMQ**.

Ainda com o auxílio do docker swarm, foi feito o deploy de réplicas do **RabbitMQ** com o plugin do **Consul** ativado em cada nó, sendo estes cada um com o seu próprio storage, e em uma mesma rede virtual.

Por último, o **HAProxy** foi usado como _load balancer_.

## Testes

Para testar a performance da aplicação, foi utilizado a ferramenta _rabbit-mq-stress-tester_, disponível no GitHub.

- https://github.com/agocs/rabbit-mq-stress-tester

Esta ferramenta permite enviar ou consumir mensagens em massa e de forma paralela para o **RabbitMQ**. Ela também permite configurar parâmetros importantes como tamanho das mensagens, tempo de espera enre mensagens e confirmação e reenvio de mensagens.

No broker, foram criadas 3 filas: **stress-queue**, **sd** e **two.stress-queue**. Sendo a última configurada para utilizar uma política de _espelhamento de fila_ em dois nós, já que por padrão, o RabbitMQ não replica o as filas.

Os seguintes comandos foram usados para geração de carga de trabalho:

### Produtor

    $ ./rabbit-mq-stress-tester --server 192.168.10.2 --user admin --password Passw0rd --producer 1000000 --bytes 10000 --wait 10000000 --concurrency 50 --quiet --wait-for-ack --queue-name stress-queue

    $ ./rabbit-mq-stress-tester --server 192.168.10.2 --user admin --password Passw0rd --producer 1000000 --bytes 10000 --wait 10000000 --concurrency 50 --quiet --wait-for-ack --queue-name sd

    $ ./rabbit-mq-stress-tester --server 192.168.10.2 --user admin --password Passw0rd --producer 1000000 --bytes 10000 --wait 10000000 --concurrency 50 --quiet --wait-for-ack --queue-name two.stress-queue

### Consumidor

    $ ./rabbit-mq-stress-tester --server 192.168.10.2 --user admin --password Passw0rd --consumer 0 --queue-name stress-queue

## Tolerância a Falhas

Para testar a tolerância a falhas uma das máquinas virtuais foi reiniciada utilizando o **Vagrant** com o comando:

    $ vagrant reload worker2

Os resultados podem ser conferidos no vídeo no YouTube: https://www.youtube.com/watch?v=lmc_ncQrZcU

## Autor

**Felipe Marinho - Sistemas Distribuídos 2019.2 | UFCG**
