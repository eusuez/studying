# studying
repositorio feito para qualquer projeto que eu precise fazer para a UNIFAA 




Considerações para o diagrama dos cursos e o banco de dados  do mesmo:


Validação: É essencial garantir que um curso não seja oferecido em mais de um período e que um aluno não possa se matricular em mais de um curso ao mesmo tempo no mesmo período.

Cálculo de Aprovado/Reprovado: A lógica para calcular a nota final (NF) e determinar o resultado deve ser implementada no sistema.

resultado (Aprovado, Reprovado). Considere a informação de que o aluno é aprovado quando a NF (nota final) obitida pela média de nota 1 (N1) e nota 2 (N2) é igual ou superior a 7,0!

a) Criação de todas as tabelas, considerando todas as constraints:

CREATE TABLE professores (

    id SERIAL PRIMARY KEY,

    nome VARCHAR(100) NOT NULL,

    cpf VARCHAR(11) NOT NULL UNIQUE,

    titulacao VARCHAR(50),

    email VARCHAR(100) NOT NULL UNIQUE,

    CONSTRAINT ck_titulacao CHECK (titulacao IN ('Mestre', 'Doutor', 'Graduado'))

);



CREATE TABLE alunos (

    id SERIAL PRIMARY KEY,

    nome VARCHAR(100) NOT NULL,

    cpf VARCHAR(11) NOT NULL UNIQUE,

    email VARCHAR(100) NOT NULL UNIQUE,

    ra VARCHAR(20) NOT NULL UNIQUE,

    sexo CHAR(1) CHECK (sexo IN ('M', 'F')),

    data_matricula DATE NOT NULL

);



CREATE TABLE cursos (

    id SERIAL PRIMARY KEY,

    nome VARCHAR(100) NOT NULL,

    carga_horaria INT NOT NULL,

    valor DECIMAL(10, 2) NOT NULL

);



CREATE TABLE matriculas (

    id SERIAL PRIMARY KEY,

    aluno_id INT REFERENCES alunos(id),

    curso_id INT REFERENCES cursos(id),

    data_matricula DATE NOT NULL,

    nota_final DECIMAL(5, 2),

    resultado_final VARCHAR(20)

);



d) Criação de índices para os professores e alunos pelos seus CPFs:

CREATE INDEX idx_professor_cpf ON professores(cpf);

CREATE INDEX idx_aluno_cpf ON alunos(cpf);



e) Inserção de vários registros em todas as tabelas:

INSERT INTO professores (nome, cpf, titulacao, email) VALUES

('João Silva', '12345678901', 'Mestre', 'joao@example.com'),

('Maria Oliveira', '10987654321', 'Doutor', 'maria@example.com');



INSERT INTO alunos (nome, cpf, ra, sexo, email, data_matricula) VALUES

(Natan Roldão', '18511610740', 'E63348', 'M', 'eusuez@gmail.com', '2023-01-10'),



INSERT INTO cursos (nome, carga_horaria, valor) VALUES

('Curso de SQL', 60, 200.00),

('Curso de Python', 80, 300.00);



INSERT INTO matriculas (aluno_id, curso_id, data_matricula, nota_final, resultado_final) VALUES

(1, 1, '2023-01-10', 9.5, 'Aprovado')



f) Altere eventuais cadastros de professores cuja titulação esteja como “Mestre” para “Mestrado”:

UPDATE professores

SET titulacao = 'Mestrado'

WHERE titulacao = 'Mestre';



g) Exclua os períodos letivos de anos anteriores:

DELETE FROM matriculas

WHERE data_matricula < '2023-01-01';



h) Listar todas as alunas em ordem alfabética (ra, nome e e-mail):

SELECT ra, nome, email

FROM alunos

WHERE sexo = 'F'

ORDER BY nome;



i) Listar todos os professores que tenham sua titulação não preenchida:

SELECT *

FROM professores

WHERE titulacao IS NULL;



j) Listar ra, nome e cpf de todos os alunos matriculados em um curso específico:

SELECT a.ra, a.nome, a.cpf, c.nome AS curso, c.carga_horaria

FROM alunos a

JOIN matriculas m ON a.id = m.aluno_id

JOIN cursos c ON m.curso_id = c.id

WHERE c.nome = 'Curso de SQL';



k) Listar todos os cursos, carga horária e valor, nome do professor, titulação do professor:

SELECT c.nome AS curso, c.carga_horaria, c.valor, p.nome AS professor, p.titulacao

FROM cursos c

JOIN professores p ON p.id = (SELECT id FROM professores ORDER BY RANDOM() LIMIT 1)

WHERE m.data_matricula BETWEEN '2023-01-01' AND '2023-12-31';



l) Listar todos os alunos (nome e e-mail) de um professor específico:

SELECT a.nome, a.email

FROM alunos a

JOIN matriculas m ON a.id = m.aluno_id

JOIN cursos c ON m.curso_id = c.id

JOIN professores p ON p.id = (SELECT id FROM professores WHERE nome = 'João Silva')

WHERE m.curso_id = c.id;



m) Gerar o boletim do aluno, com ra, nome, cpf do aluno, nome do curso, nota final e resultado final de todos os alunos aprovados:

SELECT a.ra, a.nome, a.cpf, c.nome AS curso, m.nota_final, m.resultado_final

FROM alunos a

JOIN matriculas m ON a.id = m.aluno_id

JOIN cursos c ON m.curso_id = c.id

WHERE m.resultado_final = 'Aprovado';



n) Listar a quantidade de alunos por curso em um determinado período de oferta:

SELECT c.nome, COUNT(m.aluno_id) AS quantidade_alunos

FROM cursos c

LEFT JOIN matriculas m ON c.id = m.curso_id

WHERE m.data_matricula BETWEEN '2023-01-01' AND '2023-12-31'

GROUP BY c.nome;



o) Informar o valor do curso mais caro, do curso mais barato e o valor médio dos cursos:

SELECT

    MAX(valor) AS curso_mais_caro,

    MIN(valor) AS curso_mais_barato,

    AVG(valor) AS valor_medio

FROM cursos;



p) Informar o ticket médio dos cursos em um determinado período de oferta:

SELECT AVG(valor) AS ticket_medio

FROM cursos

WHERE id IN (SELECT curso_id FROM matriculas WHERE data_matricula BETWEEN '2023-01-01' AND '2023-12-31');



q) Listar nome e cpf de todos os professores que nunca foram alunos dos cursos ofertados:

SELECT p.nome, p.cpf

FROM professores p

WHERE NOT EXISTS (

    SELECT 1

    FROM matriculas m

    JOIN alunos a ON a.id = m.aluno_id

    WHERE a.cpf = p.cpf

);



r) Listar a quantidade de alunos do sexo masculino e feminino matriculados nos cursos cuja mensalidade está acima do valor médio de todos os cursos:

WITH valor_medio AS (

    SELECT AVG(valor) AS media FROM cursos

)

SELECT

    SUM(CASE WHEN sexo = 'M' THEN 1 ELSE 0 END) AS quantidade_masculino,

    SUM(CASE WHEN sexo = 'F' THEN 1 ELSE 0 END) AS quantidade_feminino

FROM alunos a

JOIN matriculas m ON a.id = m.aluno_id

JOIN cursos c ON m.curso_id = c.id

WHERE c.valor > (SELECT media FROM valor_medio);




