## 👨‍🏫 Metadados e Informações Acadêmicas
- **Instituição:** Faculdade de Tecnologia de Ferraz de Vasconcelos (FATEC-FV)
- **Curso:** Análise e Desenvolvimento de Sistemas (ADS)
- **Disciplina:** Tópicos em Banco de Dados e Big Data
- **Professora Orientadora:** Prof. Francisco Abreu
- **Pesquisador / Desenvolvedor:** Josias Sobrinho
- **Projeto Base:** Atividade prática com Apache Cassandra

# 🚀 Atividade Prática: Introdução ao Apache Cassandra com Docker

> [!IMPORTANT]
> **Ambiente de Execução:** Os comandos deste roteiro foram elaborados e testados em ambiente Linux. Caso utilize Windows ou macOS, os comandos do Docker e CQL permanecem idênticos, com eventuais adaptações apenas nos comandos de terminal (PowerShell / WSL2).

---

## 🎯 Objetivos da Atividade
1. Provisionar uma instância local do **Apache Cassandra** utilizando **Docker Compose**.
2. Compreender a arquitetura de armazenamento do Cassandra (*Keyspaces*, *Tables*, *Partition Keys*).
3. Conectar e interagir com o cluster via **CQLSh** (*Cassandra Query Language Shell*).
4. Praticar operações de CRUD (*Create, Read, Update, Delete*) e entender os conceitos de *Upsert* e *Tombstones*.
5. Compreender as particularidades do modelo distribuído do Cassandra em comparação aos bancos relacionais (filtros, índices secundários e agregação com `COUNT`).

---

## 📋 1. Configuração do Docker Compose

O Apache Cassandra é um banco de dados NoSQL distribuído, altamente escalável e do tipo *Wide-Column Store*. Para o nosso laboratório, vamos instanciar um nó com persistência de dados.

Crie um arquivo chamado `docker-compose.yml` no seu diretório de trabalho com o seguinte conteúdo:

```yaml
version: '3.9'

services:
  cassandra:
    image: cassandra:latest
    container_name: cassandra
    ports:
      - "9042:9042" # Porta nativa do protocolo de transporte CQL
    volumes:
      - cassandra_data:/var/lib/cassandra # Persistência das SSTables e CommitLogs

volumes:
  cassandra_data:
```

> [!NOTE]
> **Explicação das diretivas:**
> - **Porta `9042`**: É a porta padrão usada pelo driver nativo CQL e pela ferramenta de linha de comando `cqlsh`.
> - **Volume `cassandra_data`**: Garante que os dados criados (Keyspaces, tabelas e registros) não sejam perdidos caso o container seja reiniciado ou recriado.

---

## 🐳 2. Execução e Verificação do Container

### 2.1. Inicializar o serviço em segundo plano (background)

Execute no terminal:

```bash
docker compose up -d
```

> [!TIP]
> **Aguarde a inicialização da JVM (30 a 60 segundos):**
> O Cassandra roda sobre a Máquina Virtual Java (JVM) e demora alguns segundos para inicializar os serviços internos (*Gossip*, *StorageService*, *Native Transport Daemon*).
> Se você tentar conectar imediatamente, poderá receber a mensagem: `Unable to connect to any servers`.

### 2.2. Validar o status do cluster com o `nodetool`

O `nodetool` é a principal ferramenta administrativa do Cassandra para monitorar os nós e a integridade do cluster. Verifique se o nó está ativo e saudável:

```bash
docker exec -it cassandra nodetool status
```

O status esperado para o nó é **`UN`** (*Up/Normal*):
```text
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  172.18.0.2  105.2 KiB  16      100.0%            a1b2c3d4-e5f6-7890-abcd-1234567890ab  rack1
```

### 2.3. Acessar o terminal CQL (cqlsh)

Com o nó em estado `UN`, conecte-se ao terminal interativo CQL:

```bash
docker exec -it cassandra cqlsh
```

*(Opcional: Caso a porta padrão do host esteja mapeada de forma diferente ou você queira forçar a conexão direta interna no container):*

```bash
docker exec -it cassandra cqlsh 127.0.0.1 9042
```

---

## 🏛️ 3. Criação de Keyspace e Tabela

No ecossistema Cassandra:
- **Keyspace**: É o agrupador de nível superior, equivalente a um *Database* ou *Schema* nos bancos relacionais. Nele são definidas as estratégias de replicação.
- **Table (ou Column Family)**: Estrutura que armazena as linhas de dados particionadas.

Execute dentro do prompt do `cqlsh`:

```cql
-- 1. Criar o Keyspace com estratégia de replicação local
CREATE KEYSPACE atividade_cassandra 
WITH replication = {
    'class': 'SimpleStrategy', 
    'replication_factor': 1
};

-- 2. Selecionar o Keyspace de trabalho
USE atividade_cassandra;

-- 3. Criar a tabela de alunos
CREATE TABLE alunos (
    matricula int PRIMARY KEY,
    nome text,
    curso text,
    semestre int,
    email text,
    status text
);
```

> [!NOTE]
> **Conceito Teórico - Replicação e Chave Primária:**
> - `SimpleStrategy`: Estratégia de replicação para datacenter único (adequada para desenvolvimento e testes). Em ambientes de produção multi-datacenter, utiliza-se `NetworkTopologyStrategy`.
> - `replication_factor: 1`: Indica que haverá 1 cópia de cada dado no cluster.
> - `matricula int PRIMARY KEY`: No Cassandra, a chave primária simples atua como **Partition Key** (Chave de Partição). O hash desse valor determina em qual nó do cluster a linha será fisicamente armazenada.

### 3.1. Inspecionar as estruturas criadas

Para visualizar as definições e metadados no `cqlsh`:

```cql
DESCRIBE KEYSPACE atividade_cassandra;
DESCRIBE TABLE alunos;
```

---

## 📝 4. Manipulação de Dados com CQL (CRUD)

### 4.1. Inserção de Registros (`INSERT`)

Insira os dados iniciais dos alunos:

```cql
INSERT INTO alunos (matricula, nome, curso, semestre, email, status) 
VALUES (345257484, 'Martim Afonso de Sousa', 'ADS', 1, 'martim.afonso@fatec.sp.gov.br', 'Ativo');

INSERT INTO alunos (matricula, nome, curso, semestre, email, status) 
VALUES (148791365, 'Teresa Cristina de Bourbon', 'Agro', 2, 'teresa.cristina@fatec.sp.gov.br', 'Formado');

INSERT INTO alunos (matricula, nome, curso, semestre, email, status) 
VALUES (642949538, 'Pedro de Alcântara', 'ADM', 3, 'pedro.alcantara@fatec.sp.gov.br', 'Ativo');

INSERT INTO alunos (matricula, nome, curso, semestre, email, status) 
VALUES (781582006, 'Isabel de Bragança', 'GTI', 4, 'isabel.braganca@fatec.sp.gov.br', 'Trancado');

INSERT INTO alunos (matricula, nome, curso, semestre, email, status) 
VALUES (307127774, 'Fernão de Magalhães', 'Logística', 5, 'fernao.magalhaes@fatec.sp.gov.br', 'Ativo');
```

> [!TIP]
> **No Cassandra, todo `INSERT` é essencialmente um `Upsert`!**
> Se você executar um `INSERT` com uma chave primária (`matricula`) já existente, o Cassandra não emitirá erro de chave duplicada: ele sobrescreverá os dados existentes com base no *timestamp* mais recente.

---

## 🔍 5. Consultas com CQL (`SELECT`)

### 5.1. Consulta Geral de Registros

```cql
SELECT * FROM alunos;
```

### 5.2. Consulta filtrando pela Chave Primária (Partition Key)

A busca por chave primária é a operação mais rápida e eficiente no Cassandra ($O(1)$), pois o hash aponta diretamente para o nó e partição onde o dado reside:

```cql
SELECT * FROM alunos WHERE matricula = 345257484;
```

### 5.3. ⚠️ O Desafio das Consultas sem Chave Primária

Tente executar uma busca filtrando por uma coluna que **não** é chave primária:

```cql
SELECT * FROM alunos WHERE curso = 'ADS';
```

O Cassandra retornará um erro:
> `InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING"`

#### Por que isso ocorre?
Diferente dos bancos relacionais (RDBMS), o Cassandra é distribuído. Para buscar por `curso`, ele precisaria varrer todas as partições em todos os nós do cluster (*Full Cluster Scan*), o que destruiria a performance em ambientes com milhões de dados.

#### Como resolver?
1. **Opção 1: Criar um Índice Secundário (Recomendado para filtros pontuais de baixa cardinalidade):**
   ```cql
   CREATE INDEX idx_alunos_curso ON alunos (curso);
   SELECT * FROM alunos WHERE curso = 'ADS';
   ```
2. **Opção 2: `ALLOW FILTERING` (Apenas para depuração pontual):**
   ```cql
   SELECT * FROM alunos WHERE status = 'Ativo' ALLOW FILTERING;
   ```
   > [!CAUTION]
   > Em produção, o uso frequente de `ALLOW FILTERING` é considerado um *anti-pattern*. A regra de ouro na arquitetura Cassandra é: **modele suas tabelas orientadas às consultas da aplicação** (*Query-Driven Modeling*), criando tabelas desnormalizadas se necessário.

---

## 🔄 6. Alterações de Dados (`UPDATE`)

No Cassandra, atualizações de valores exigem que a chave de partição seja informada na cláusula `WHERE`:

```cql
-- Promover o aluno Martim para o 2º semestre
UPDATE alunos SET semestre = 2 WHERE matricula = 345257484;

-- Atualizar o status do aluno Pedro para 'Formado'
UPDATE alunos SET status = 'Formado' WHERE matricula = 642949538;
```

Verifique se as alterações foram aplicadas:

```cql
SELECT matricula, nome, semestre, status FROM alunos WHERE matricula IN (345257484, 642949538);
```

---

## 🗑️ 7. Remoção de Registros (`DELETE`) e o conceito de Tombstones

Remova um aluno utilizando sua matrícula:

```cql
DELETE FROM alunos WHERE matricula = 345257484;
```

Confirme a exclusão:

```cql
SELECT * FROM alunos WHERE matricula = 345257484;
```

> [!NOTE]
> **Como o Cassandra lida com exclusões (Tombstones)?**
> Em arquiteturas distribuídas, apagar arquivos diretamente do disco pode ressuscitar dados antigos caso algum nó estivesse offline durante o delete. 
> Por isso, o Cassandra grava um marcador especial chamado **Tombstone** (lápide) indicando a exclusão com um timestamp. Os dados só são expurgados fisicamente do disco durante o processo periódico de **Compactação** (*Compaction*), respeitando a janela de segurança (`gc_grace_seconds`, padrão de 10 dias).

---

## 📊 8. Funções de Agregação e Alerta de Produção

Consulte o número total de alunos restantes:

```cql
SELECT COUNT(*) FROM alunos;
```

> [!WARNING]
> ### 🚨 O perigo do comando `COUNT(*)` em Produção
> Embora funcione perfeitamente com poucos registros na atividade acadêmica, se essa tabela tivesse milhões de linhas em produção, essa consulta causaria:
> - **Varredura global do cluster:** Todos os nós precisariam ler e consolidar suas partições.
> - **Alto consumo de I/O e CPU:** Queda de desempenho severa para outros clientes do banco.
> - **ReadTimeoutException:** A consulta muito provavelmente excederá o timeout e falhará.
> - **Alternativa recomendada:** Manter colunas de tipo `counter` em tabelas auxiliares, ou delegar agregações pesadas para motores analíticos (como Apache Spark ou Trino).

---

## ⏱️ 9. Recurso Prático: Inserção com TTL (*Time-To-Live*)

O Cassandra possui suporte nativo a expiração automática de dados por tempo, amplamente utilizado para logs temporários, tokens e carrinhos de compras.

Insira um aluno de teste que expira automaticamente em 60 segundos:

```cql
INSERT INTO alunos (matricula, nome, curso, semestre, email, status)
VALUES (999999999, 'Visitante Temporário', 'ADS', 1, 'visitante@fatec.sp.gov.br', 'Temporario')
USING TTL 60;
```

Consulte o registro e o tempo de vida restante (em segundos):

```cql
SELECT matricula, nome, TTL(nome) FROM alunos WHERE matricula = 999999999;
```

*(Aguarde 1 minuto e rode a consulta novamente para comprovar que o registro expirou e desapareceu sem necessidade de nenhum comando DELETE).*

---

## 🧩 10. Desafios Práticos (Para Fixação)

1. **Modelagem Orientada a Consultas:**
   Suponha que um requisito do sistema seja listar todos os alunos ordenados pelo **semestre** dentro de um determinado **curso**. No Cassandra, ordenações (`ORDER BY`) só são permitidas sobre **Clustering Keys**. Como seria a criação de uma nova tabela `alunos_por_curso` com chave primária composta:
   `PRIMARY KEY ((curso), semestre, matricula)`?
2. **Atualizações Condicionais (Lightweight Transactions - LWT):**
   O Cassandra suporta transações atômicas pontuais utilizando o protocolo Paxos com a cláusula `IF`. Teste uma atualização condicional:
   ```cql
   UPDATE alunos SET status = 'Formado' WHERE matricula = 148791365 IF status = 'Ativo';
   ```
   Analise o retorno booleano `[applied]` indicando se a condição foi satisfeita.
3. **Persistência dos Volumes:**
   Saia do `cqlsh`, pare o container com `docker compose down` e suba novamente com `docker compose up -d`. Conecte-se e verifique com `SELECT * FROM alunos;` se os dados permaneceram intactos graças ao volume configurado.

---

## 🛑 11. Encerramento e Limpeza

Para sair do terminal interativo CQL:

```cql
exit;
```

Para gerenciar os containers no terminal do host:

```bash
# Pausar/parar o container preservando os dados
docker compose stop

# Parar e remover o container mantendo os dados no volume
docker compose down

# ATENÇÃO: Caso queira apagar completamente os dados e reiniciar do zero
docker compose down -v
```

# Bonus

## Como criar a Chave de Agrupamento (Clustering Key)

**Para definir uma Chave de Agrupamento, você precisa passar uma tupla dentro do PRIMARY KEY (Chave_de_Partição, Chave_de_Agrupamento).**

Se quisermos buscar os alunos por curso (Chave de Partição) e mantê-los ordenados por nome ou matricula (Chave de Agrupamento), a estrutura ideal é esta:  

```cql
CREATE TABLE alunos_por_curso (
    curso text,
    matricula bigint,
    nome text,
    semestre int,
    email text,
    status text,
    PRIMARY KEY (curso, matricula)
) WITH CLUSTERING ORDER BY (matricula ASC);
```
-- Configuração adicional "CLUSTERING ORDER BY". Veja o motivo em [5.3](#53--o-desafio-das-consultas-sem-chave-primaria).

**O que muda nessa nova estrutura?**

* **curso (Chave de Partição):** Define em qual servidor do cluster o dado irá morar. Todos os alunos de 'ADS' ficarão guardados juntos na mesma máquina.
* **matricula (Chave de Agrupamento):** Organiza os alunos daquele curso fisicamente em disco. Eles ficam ordenados de forma crescente (ASC).
* **Consultas liberadas:** Agora você pode fazer buscas ultra rápidas filtrando pelo curso, e o COUNT(*) funcionará sem nenhum aviso de erro:
```sql
SELECT COUNT(*) FROM alunos_por_curso WHERE curso = 'ADS';
```