# Questao 1:

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
-  Escreva uma consulta PostgreSQL para obter, por nome da escola e por dia, a quantidade de alunos matriculados e o valor total das matrículas, tendo como restrição os cursos que começam com a palavra “data”. Ordene o resultado do dia mais recente para o mais antigo.
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
-  Escreva uma consulta PostgreSQL para obter, por nome da escola e por dia, a quantidade de alunos matriculados e o valor total das matrículas, tendo como restrição os cursos que começam com a palavra “data”. Ordene o resultado do dia mais recente para o mais antigo.

