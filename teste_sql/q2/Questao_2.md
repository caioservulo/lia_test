# Questao 1a:

Dadas as tabelas:

departamento:
|campo|type|
|------|-----|
|cod_dep|int|
|nome|varchar(50)|
|endereco|varchar(50)|

dependente:
|campo|type|
|------|-----|
|matr|int|
|nome|varchar(50)|
|endereco|varchar(50)|

desconto:
|campo|type|
|------|-----|
|cod_desc|int|
|nome|varchar(50)|
|tipo|varchar(10)|
|valor|numeric|

divisao:
|campo|type|
|------|-----|
|cod_divisao|int|
|nome|varchar(50)|
|endereco|varchar(50)|
|cod_dep|int|

emp_desc:
|campo|type|
|------|-----|
|cod_desc|int|
|matr|int|

emp_venc:
|campo|type|
|------|-----|
|cod_venc|int|
|matr|int|

empregado:
|campo|type|
|------|-----|
|matr|int|
|nome|varchar(50)|
|endereco|varchar(50)|
|data_lotacao|timestamp|
|lotacao|int|
|gerencia_cod_dep|int|
|lotacao_div|int|
|gerencia_div|int|

vencimento:
|campo|type|
|------|-----|
|cod_venc|int|
|nome|varchar(50)|
|tipo|varchar(10)|
|valor|numeric|


Desafios:
-  Para cada departamento, realize uma consulta em PostgresSQL que mostre o nome do departamento, a quantidade de empregados, a média salarial, o maior e o menor salários. Ordene o resultado pela maior média salarial.
---

## Passo 1: Ter visão da estrutura de dados:
  Utilizando a ferramenta [dbdiagram](https://dbdiagram.io/d/Questao-2-67e701034f7afba1849df205) podemos criar virtualmente as tabelas e visualizar seus relacionamentos.

![image](https://github.com/user-attachments/assets/1fb9c5c5-f862-4e93-ac57-e6a3bc22cbc0)

___

## Passo 2: Construção do racional da consulta:
- Enxergar chaves e relacionamentos
- Definir estrutura da consultas (CET's, Agrupamentos, etc..)
- Desenvolvimento 
___

## Passo 3: Desenho da consulta:

  - Dataset estruturado localmente:
<img width="127" alt="image" src="https://github.com/user-attachments/assets/2857adff-8aff-4849-92d1-bee493f31f9f" />



  - Consulta desenvolvida:
      ```sql
      WITH
        cursos AS (
          SELECT
            co.id,
            co.name,
            co.price,
            co.school_id
          FROM courses co
        ),
        escolas AS (
          SELECT
            s.id,
            s.name
          FROM schools s
        ),
        alunos AS (
          SELECT
            st.enrolled_at,
            st.course_id,
            st.id
          FROM students st
          ORDER BY
            st.enrolled_at DESC
        )
      SELECT
        e.name,
        a.enrolled_at,
        COUNT(
          CASE
            WHEN c.name ~* '^data' -- nome inicia com a palavra data
            THEN a.id
            ELSE NULL
        END) AS qtd_alunos_matriculados,
        ROUND(SUM(
          CASE
            WHEN c.name ~* '^data' -- nome inicia com a palavra data
            THEN c.price
            ELSE 0
        END),2) AS valor_total_matriculas
      FROM alunos a
      LEFT JOIN cursos c ON a.course_id = c.id
      LEFT JOIN escolas e ON c.school_id = e.id
      GROUP BY
        e.name,
        a.enrolled_at
      ORDER BY
        a.enrolled_at DESC;

  - Retorno:

![image](https://github.com/user-attachments/assets/11bb76c9-bf6c-4284-a219-1adf35544992)

**Análise:** podemos ver, por dia, o volume de alunos e total de matrícula apenas dos cursos que iniciam dom o temro "data" (sem diferenciar minúculas e maiúsculas, prevenindo despadronizações). Como melhorias, poderíamos: 
  - Criar uma tabela d-calendario onde poderíamos realizar a análise em cima de dias corridos, no case acima analisei tomando como base a ata de inscrição. Seria basicamente implementar um código:
      ```sql
        WITH calendario AS (
    SELECT generate_series(
        '2024-01-01'::DATE,  
        '2024-12-31'::DATE,  
        '1 day'::INTERVAL
    )::DATE AS data
  e, após isso, realizar LEFT JOINS com as CETs desemnvolvidas para seguir co um análise a nível calendário.
  - Implementar os dados em uma dashboard onde o nome do curso poderia ser um filtro do memso, simoplificando a consulta e dando autonomia ao usuário.


