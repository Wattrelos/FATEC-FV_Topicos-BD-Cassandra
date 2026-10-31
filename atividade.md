# Atividade 3 - Apache Cassandra
Due November 3, 2026 11:59 

## Objetivo

* Compreender os conceitos fundamentais do Apache Cassandra, banco de dados NoSQL orientado a colunas largas (wide-column), utilizando Docker Compose para configurar e executar um ambiente local.

* Aplicar os conceitos de Keyspace, tabelas, chave primária e manipulação de dados utilizando Cassandra Query Language (CQL).

* Realizar operações de inserção, consulta, atualização e remoção de dados, além de verificar a persistência das informações.

### A atividade deverá ser realizada individualmente, em sua máquina pessoal, contemplando as etapas a seguir.

1. Configuração do Docker Compose

Crie um arquivo chamado docker-compose.yml contendo um serviço para o Apache Cassandra.
2. Execução e verificação do container

* Inicie o ambiente e execute os comandos necessários para verificar o funcionamento do Cassandra.

* Após a inicialização, acesse o Cassandra utilizando o terminal CQL (cqlsh).

* Comprove que o banco de dados está disponível para receber comandos CQL.

3. Criação de Keyspace e tabela

### Utilizando CQL, realize as seguintes operações:

* Crie um Keyspace chamado atividade_cassandra, utilizando SimpleStrategy e fator de replicação igual a 1.

* Crie uma tabela chamada alunos, contendo, no mínimo, os seguintes atributos:

| Atributo | Tipo de dado |Finalidade|
|:--|:--|:--|
|matricula|int|Identificador único do aluno (PRIMARY KEY)|
|nome|text|Nome completo do aluno|
|curso|text|Curso em que está matriculado|
|semestre|int|Semestre atual|
|email|text|Endereço de e-mail|
|status|text|Situação acadêmica do aluno|


## 3. Manipulação de dados com CQL

### Realize as operações a seguir utilizando a linguagem CQL.

|Operação|Requisito obrigatório|
|:--|:--|
|INSERT|Inserir cinco alunos na tabela.|
|SELECT|Consultar todos os alunos cadastrados.|
|SELECT com WHERE|Consultar um aluno específico por sua matrícula.|
|UPDATE|Atualizar o semestre de um aluno.|
|UPDATE|Alterar o status acadêmico de outro aluno.|
|DELETE|Remover um aluno utilizando sua matrícula.|
|COUNT|Consultar a quantidade total de alunos após a remoção.|

### Todas as operações deverão ser executadas e seus resultados comprovados por meio de capturas de tela ou gravação.

## Entrega

### Envie obrigatoriamente os seguintes arquivos:

* docker-compose.yml contendo a configuração do Apache Cassandra.

* atividade.cql contendo todos os comandos CQL utilizados na atividade.

### Além desses arquivos, escolha uma das formas de comprovação:

* Vídeo de até 5 minutos: apresente sua identificação, o arquivo Docker Compose, o container em execução, a criação do Keyspace e da tabela, as operações CQL e a comprovação da persistência dos dados.

* Relatório com capturas de tela: inclua nome completo e matrícula, evidências de todas as etapas realizadas e uma breve explicação de cada imagem apresentada.

* Relatório com capturas de tela: inclua nome completo e matrícula, evidências de todas as etapas realizadas e uma breve explicação de cada imagem apresentada.


