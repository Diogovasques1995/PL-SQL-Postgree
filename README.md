# 🐘 PostgreSQL: Modelagem Relacional & Gestão de Inventário IoT

Este projeto demonstra o desenvolvimento de um modelo de banco de dados relacional utilizando o **PostgreSQL** (via pgAdmin 4). O cenário simula um sistema real de controle de inventário de hardware (Dispositivos/Ativos de Redes) integrado a uma estrutura de gerenciamento de clientes (Organizações).

## 📊 Paradigma e Modelo de Dados
* **Modelo de Banco de Dados:** Relacional (SQL)
* **Arquitetura Lógica:** Relacionamento Um-para-Muitos (1:N) com restrições rígidas de integridade.
* **Isolamento de Escopo:** Uso de **Schemas** personalizados para governança de dados.

---

## 🛠️ Engenharia e Evolução de Schema (DDL)

Diferente do padrão de mercado que concentra tudo no schema `public`, este projeto isola o contexto de negócio criando um espaço exclusivo chamado `dispositivos`. 

Abaixo estão os scripts estruturados que demonstram a criação sequencial das entidades e a evolução do banco em produção (inclusão de colunas e amarração de chaves estrangeiras via `ALTER TABLE` após a criação da tabela):

```sql
-- 1. Criação do Schema Organizacional
CREATE SCHEMA dispositivos;

-- 2. Criação da Tabela Base (Entidade Forte)
CREATE TABLE dispositivos.organizacoes (
    id SERIAL PRIMARY KEY,
    id_customer VARCHAR(50) NOT NULL,
    nome VARCHAR(100) NOT NULL,
    data_cadastro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 3. Criação da Tabela de Ativos (Entidade Dependente)
CREATE TABLE dispositivos.device (
    id SERIAL PRIMARY KEY,
    codigo_dispositivo VARCHAR(50) NOT NULL,
    nome VARCHAR(100) NOT NULL,
    status_armazenamento SMALLINT NOT NULL, -- Regra: 1 = Em Estoque | 2 = Em Cliente
    organizacao_id INT,
    data_registro TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 4. Evolução do Schema: Inclusão de Coluna de Conectividade (Sigfox)
ALTER TABLE dispositivos.device ADD COLUMN sigfox VARCHAR(50);

-- 5. Integridade Referencial: Vinculando a Chave Estrangeira (Foreign Key)
ALTER TABLE dispositivos.device 
ADD CONSTRAINT fk_device_organizacao 
FOREIGN KEY (organizacao_id) 
REFERENCES dispositivos.organizacoes(id)
ON DELETE SET NULL;
```

Para a extração de relatórios inteligentes, a consulta abaixo realiza o cruzamento (LEFT JOIN) das tabelas e faz uso da cláusula conditional CASE WHEN para traduzir os códigos numéricos de armazenamento em descrições textuais prontas para consumo de BI:

```SQL
SELECT 
    d.codigo_dispositivo,
    d.nome AS nome_dispositivo,
    d.sigfox AS tecnologia_sigfox,
    CASE 
        WHEN d.status_armazenamento = 1 THEN 'Em Estoque'
        WHEN d.status_armazenamento = 2 THEN 'Em Cliente'
        ELSE 'Status Inválido/Desconhecido'
    END AS status_atual,
    o.organizacao AS nome_organizacao,
    o.customer_id AS codigo_cliente
FROM dispositivos.device d
LEFT JOIN dispositivos.organizacoes o ON d.organizacao_id = o.id;
```

💡 Competências Demonstradas neste Módulo
Diferenciação de Dialetos: Uso prático de tipos nativos do Postgres como SERIAL (auto-incremento) e NUMERIC.

Tratamento de Sintaxe Complexa: Resolução de conflitos de DDL e aplicação de constraints de chave via linha de comando isolada.

Abstração de Dados: Separação de privilégios e tabelas através de Namespaces (Schemas).
