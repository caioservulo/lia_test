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

![image](https://github.com/user-attachments/assets/80eaf09a-4797-4df2-ac54-906fe17001aa)


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
      - Racional
         - Identificação de departamento: segui uma lógica de relacionar a lotacao_div (tabela empregado) com o cod_divisao (tabela divisao) para encontrar o nome da divissao (tabela divisao). COm esse mesmo racional de relacionamento de tabelas fui capaz de realizar a contagem do total de empregados/divisao
              ```sql
              WITH
                empregados AS (
                  SELECT
                    emp.matr,
                    emp.gerencia_cod_dep,
                    emp.lotacao_div,
                    emp.gerencia_div
                  FROM public.empregado emp
                ),
                divisao AS (
                  SELECT
                    div.cod_divisao,
                    div.cod_dep
                  FROM public.divisao div
                ),
                departamento AS (
                  SELECT
                    dep.cod_dep,
                    dep.nome
                  FROM public.departamento dep
                )
              SELECT
                dp.nome,
                COUNT(DISTINCT e.matr) AS qtd_empregados
              FROM departamento dp
              LEFT JOIN divisao d ON dp.cod_dep = d.cod_dep
              LEFT JOIN empregados e ON d.cod_divisao = e.lotacao_div
              GROUP BY
                dp.nome;
  
        -  Identificando salários: na consulta abaixo sou capar de ter visao de todos od descontos e vencimentos por funcionário.
              ```sql
                WITH
                	empregados AS (
                		SELECT
                			emp.matr,
                			emp.gerencia_cod_dep,
                			emp.lotacao_div,
                			emp.gerencia_div
                		FROM public.empregado emp
                	),
                	vencimento AS (
                		SELECT
                			ev.matr,
                			venc.nome,
                			venc.tipo,
                			venc.valor
                		FROM public.emp_venc ev
                		LEFT JOIN public.vencimento venc ON ev.cod_venc = venc.cod_venc
                	),
                	desconto AS (
                		SELECT
                			ed.matr,
                			dsct.nome,
                			dsct.tipo,
                			dsct.valor
                		FROM public.emp_desc ed
                		LEFT JOIN public.desconto dsct ON ed.cod_desc = dsct.cod_desc
                	)
                SELECT
                	e.matr,
                	e.gerencia_cod_dep,
                	e.lotacao_div,
                	e.gerencia_div,
                	COALESCE(desct.nome, 'Sem desconto') AS nome_desconto,
                	COALESCE(desct.tipo, '-') AS tipo_desconto,
                	COALESCE(desct.valor, 0)::NUMERIC AS valor_desconto,
                	vc.nome AS nome_vencimento,
                	vc.tipo AS tipo_vencimento,
                	vc.valor AS valor_vencimento
                FROM empregados e
                LEFT JOIN desconto desct ON e.matr = desct.matr
                LEFT JOIN vencimento vc ON e.matr = vc.matr;

          Entendento que o racional para o cálculo do salário do trabalhador seja: ***[somatório de todos os seus vencimentos - somatório de todos os seus descontos]***
          A consulta para determinar o saário de cada funcionário é:
        
                ```sql
                  WITH
                  	empregados AS (
                  		SELECT
                  			emp.matr,
                  			emp.gerencia_cod_dep,
                  			emp.lotacao_div,
                  			emp.gerencia_div
                  		FROM public.empregado emp
                  	),
                  	vencimento AS (
                  		SELECT
                  			ev.matr,
                  			venc.nome,
                  			venc.tipo,
                  			venc.valor
                  		FROM public.emp_venc ev
                  		LEFT JOIN public.vencimento venc ON ev.cod_venc = venc.cod_venc
                  	),
                  	desconto AS (
                  		SELECT
                  			ed.matr,
                  			dsct.nome,
                  			dsct.tipo,
                  			dsct.valor
                  		FROM public.emp_desc ed
                  		LEFT JOIN public.desconto dsct ON ed.cod_desc = dsct.cod_desc
                  	),
                  	salario AS (
                  		SELECT 
                  			e.matr,
                  			COALESCE(SUM(desct.valor), 0)::NUMERIC AS valor_desconto,
                  			COALESCE(SUM(vc.valor), 0)::NUMERIC AS valor_vencimento,
                  			ROUND ((COALESCE(SUM(vc.valor), 0)::NUMERIC - COALESCE(SUM(desct.valor), 0)::NUMERIC),2) AS salario
                  		FROM empregados e
                  		LEFT JOIN desconto desct ON e.matr = desct.matr
                  		LEFT JOIN vencimento vc ON e.matr = vc.matr
                  		GROUP BY
                  			e.matr
                  	)
                  SELECT
                  	e.matr,
                  	ROUND (AVG(valor_vencimento), 2) AS valor_vencimento,
                  	ROUND(AVG(s.valor_desconto), 2) AS valor_desconto,
                  	ROUND(AVG(salario), 2) AS salario
                  FROM empregados e
                  LEFT JOIN salario s ON e.matr = s.matr
                  GROUP BY
                  	e.matr;


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


