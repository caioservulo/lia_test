# Questao 1b:

Dadas as 3 tabelas:

students:
|**campo**| **type** | **comentário**|
|----------|---------|-------------------|
|id |int|-|
|name| text|-|
|enrolled_at |date| -|
|course_id |text|-|


courses: 
|**campo**| **type** |**comentário**|
|----------|---------|-------------------|
|id |int|-|
|name| text|-|
|price |numeric| configuraria o data type como FLOAT a depender do DB|
|school_id |text|  configuraria o data type como INT a depender do DB|


schools: 
|**campo**| **type** |**comentário**|
|----------|---------|-------------------|
|id |int|-|
|name| text|-|


Desafios:
- Utilizando a resposta do item a, escreva uma consulta para obter, por escola e por dia, a soma acumulada, a média móvel 7 dias e a média móvel 30 dias da quantidade de alunos.
---

## Passo 1: Ter visão da estrutura de dados:
  Utilizando a ferramenta [dbdiagram](https://dbdiagram.io/d/67e701104f7afba1849df321) podemos criar virtualmente as tabelas e visualizar seus relacionamentos.

![image](https://github.com/user-attachments/assets/fdd623ff-e704-4df4-97ba-f81f4bb95f7f)
___

## Passo 2: Construção do racional da consulta:
- Enxergar chaves e relacionamentos
- Definir estrutura da consultas (CET's, Agrupamentos, etc..)
- Desenvolvimento 
___

## Passo 3: Desenho da consulta:

  - Dataset simulado (utilizando ferramentas: runsql.com):
  ![image](https://github.com/user-attachments/assets/e9375c49-c50f-4889-b899-4abc86db8b86)
  ![image](https://github.com/user-attachments/assets/6b225a46-725c-4c2d-8155-a131d6a5ac63)
  ![image](https://github.com/user-attachments/assets/b7a824ca-8a23-4d87-9829-0839a645dae4)
  ![image](https://github.com/user-attachments/assets/b1ff7cfb-91d4-49b0-869a-9b650198654b)

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


