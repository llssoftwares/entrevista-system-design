*Enunciado original do desafio:*

>Requisitos Funcionais
>
> O governo anunciou a abertura de uma licitação para o desenvolvimento e implementação de um sistema informatizado voltado à geração de relatórios detalhados de faturamento das unidades de pedágio do país. Como vencedor dessa licitação, você será responsável por projetar e implementar uma solução eficiente e escalável, capaz de receber dados sobre as utilizações de cada unidade e consolidá-los em um relatório no formato especificado pelo edital. De acordo com informações do UOL, o Brasil conta com mais de 1.800 praças de pedágio distribuídas pelas 27 unidades federativas, o que evidencia a magnitude e a importância do projeto. Este software deverá não apenas atender aos requisitos técnicos, mas também ser capaz de lidar como grande volume de dados gerado diariamente, garantindo a precisão e a agilidade necessárias para a tomada de decisões administrativas e estratégicas.
> 
> Os dados de utilização devem ser unitários e conter minimamente os atributos a seguir:
> 
> * Data e hora de utilização
> * Praça
> * Cidade
> * Estado
> * Valor pago
> * Tipo de veículo (Moto, Carro ou Caminhão)
> 
> Os relatórios a seguir foram solicitados e, assim que gerados, devem ser enviados para os gestores cadastrados via email:
> 
> * Valor total por hora por cidade
> * As praças que mais faturaram por mês (a quantidade a ser processada deve ser configurável)
> * Quantos tipos de veículos passaram em uma determinada praça

---

*Solução proposta*

# Sistema de Relatórios de Faturamento de Praças de Pedágio

## Visão Geral do Projeto

Este documento descreve a arquitetura técnica proposta para o desenvolvimento de um sistema escalável e eficiente voltado à geração de relatórios de faturamento das praças de pedágio do Brasil. O projeto tem como objetivo consolidar os dados de utilização das praças e disponibilizar relatórios automatizados para os gestores via e-mail.

## Requisitos Funcionais Atendidos

* Recebimento de dados unitários de utilização, contendo:

  * Data e hora de utilização
  * Praça
  * Cidade
  * Estado
  * Valor pago
  * Tipo de veículo (Moto, Carro ou Caminhão)
* Geração dos seguintes relatórios:

  * Valor total por hora por cidade
  * Praças que mais faturaram por mês (quantidade configurável)
  * Quantidade de tipos de veículos por praça
* Envio automatizado dos relatórios para gestores cadastrados via e-mail

## Arquitetura Proposta

A solução está baseada em uma arquitetura orientada a eventos, com separação entre escrita e leitura (CQRS), processamento assíncrono e componentes desacoplados por meio de um message broker.

### 1. Registro de Utilização (PedagioService)

* **Endpoint:** `POST /utilizacao`
* **Responsabilidade:** Registrar utilizações nas praças de pedágio
* **Banco:** `PedagioDB` (otimizado para escrita)
* **Evento gerado:** `UtilizacaoRegistrada`
* **Tecnologias sugeridas:** ASP.NET Core Minimal APIs, PostgreSQL

### 2. Message Broker

* **Responsabilidade:** Encaminhar eventos entre serviços
* **Eventos:** `UtilizacaoRegistrada`, `RelatorioProcessado`
* **Tecnologia sugerida:** RabbitMQ ou Apache Kafka

### 3. Processador de Relatórios (RelatorioService)

* **Endpoint:** `POST /relatorios`
* **Responsabilidade:** Solicitar a geração de relatórios
* **Componente:** `RelatorioAsyncProcessor`
* **Banco:** `RelatorioDB` (otimizado para leitura)
* **Armazenamento:** `RelatorioStorage` (ex: MinIO, Azure Blob Storage)
* **Evento gerado:** `RelatorioProcessado`
* **Observação:** Aceita parâmetros configuráveis (ex: top-N praças)

### 4. Envio de E-mail (EmailService)

* **Evento escutado:** `RelatorioProcessado`
* **Responsabilidade:** Montar e disparar e-mails com a URL do relatório
* **Componentes:**

  * `EmailBuilder`: Monta o conteúdo do e-mail
  * `EmailDB`: Contém dados dos gestores cadastrados
  * `SendGridAPI`: Envia o e-mail
* **Tecnologias sugeridas:** SendGrid, Entity Framework, .NET Background Services

## Escalabilidade e Resiliência

* **Separacão de responsabilidades:** cada serviço é independente e escalável isoladamente
* **Fila de mensagens:** garante desacoplamento, resiliência e possibilita retries
* **Processamento assíncrono:** evita bloqueios e melhora o throughput
* **Armazenamento separado:** uso de storage externo reduz carga do banco

## Sugestões de Extensões Futuras

* Dashboard em tempo real (ex: com Power BI conectado ao `RelatorioDB`)
* CRUD completo para gestão de usuários/gestores
* API para download manual de relatórios
* Auditoria e logging centralizado
* Suporte a autenticação/autorizacão (ex: JWT + IdentityServer)

## Considerações Finais

A arquitetura apresentada atende às necessidades funcionais do edital e está preparada para lidar com alto volume de dados de maneira escalável, eficiente e segura. O uso de padrões modernos como CQRS, processamento assíncrono e arquitetura orientada a eventos contribui para a manutenção e evolução futura do sistema.
