# Projeto-Integrador-Assistente-de-Administrador-de-Banco-de-Dados
-- criação do banco de dados
DROP DATABASE IF EXISTS SGE;
CREATE DATABASE SGE;
USE SGE;

-- criação da tabela evento 
CREATE TABLE evento (
    idEvento INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    dInicio DATE NOT NULL,
    dFim DATE NOT NULL,
    local VARCHAR(100) NOT NULL,
    descricao TEXT,
    status ENUM('planejado', 'ativo', 'cancelado', 'concluido') DEFAULT 'planejado',
    dCriacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    dAtualizacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- criação da tabela participante
CREATE TABLE participante (
    idParticipante INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cpf VARCHAR(14) UNIQUE NOT NULL,
    rg VARCHAR(12),
    telefone VARCHAR(15) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    dCriacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- criação da tabela atividade 
CREATE TABLE atividade (
    idAtividade INT AUTO_INCREMENT PRIMARY KEY,
    idEvento INT NOT NULL,
    nome VARCHAR(100) NOT NULL,
    descricao TEXT,
    dHora DATETIME NOT NULL,
    duracao INT NOT NULL COMMENT 'Duração em minutos',
    palestrante VARCHAR(100) NOT NULL,
    local VARCHAR(100) NOT NULL,
    vagas INT NOT NULL,
    FOREIGN KEY (idEvento) REFERENCES evento(idEvento) ON DELETE CASCADE
);

-- criação da tabela de relacionamento presença
CREATE TABLE presenca (
    idPresenca INT AUTO_INCREMENT PRIMARY KEY,
    idParticipante INT NOT NULL,
    idAtividade INT NOT NULL,
    status ENUM('presente', 'ausente', 'pendente') DEFAULT 'pendente',
    checkin TIMESTAMP NULL,
    FOREIGN KEY (idParticipante) REFERENCES participante(idParticipante) ON DELETE CASCADE,
    FOREIGN KEY (idAtividade) REFERENCES atividade(idAtividade) ON DELETE CASCADE
);

-- inserção de dados na tabela evento
INSERT INTO evento (nome, dInicio, dFim, local, descricao, status)
VALUES 
('Conferência de Tecnologia', '2024-03-27', '2024-03-28', 'Centro RJ', 'Evento de tecnologia', 'planejado'),
('Summit de Marketing', '2024-11-02', '2024-11-03', 'Hotel BH', 'Workshops de marketing', 'ativo'),
('Feira de Startups', '2023-07-15', '2023-07-16', 'Expo Center', 'Feira de startups', 'ativo'),
('Congresso Médico', '2024-10-19', '2024-10-20', 'Centro SP', 'Congresso médico', 'planejado'),
('Festival de Inovação', '2024-05-30', '2024-05-31', 'Parque', 'Festival de inovação', 'planejado');

-- inserção de dados na tabela participante
INSERT INTO participante (nome, cpf, rg, telefone, email)
VALUES 
('Carina Souza', '123.456.789-00', '12.345.678-9', '(11) 99999-9999', 'carina@email.com'),
('Gabriel Cruz', '987.654.321-00', '98.765.432-1', '(21) 98888-8888', 'gabriel@email.com'),
('Amanda Lopes', '111.222.333-44', '11.222.333-4', '(31) 97777-7777', 'amanda@email.com'),
('Ana Clara', '555.666.777-88', '55.666.777-8', '(41) 96666-6666', 'ana@email.com'),
('Mariana Barbosa', '999.888.777-66', '99.888.777-6', '(51) 95555-5555', 'mariana@email.com');

-- inserção de dados na tabela atividade
INSERT INTO atividade (idEvento, nome, descricao, dHora, duracao, palestrante, local, vagas)
VALUES 
(1, 'Inteligência Artificial', 'Palestra sobre IA', '2024-03-27 10:00:00', 90, 'Dr. Silva', 'Auditório A', 100),
(1, 'Blockchain', 'O futuro das criptomoedas', '2024-03-27 15:00:00', 60, 'Dr. Souza', 'Sala B', 80),
(2, 'Marketing Digital', 'Novas tendências', '2024-11-02 09:30:00', 120, 'Dra. Lima', 'Auditório', 50),
(3, 'Pitch para Investidores', 'Como captar recursos', '2023-07-15 11:00:00', 90, 'Sr. Investidor', 'Palco', 30),
(4, 'Medicina do Futuro', 'Inovações tecnológicas', '2024-10-19 09:00:00', 60, 'Dr. Santos', 'Auditório', 120);

-- inserção de dados na tabela presenca
INSERT INTO presenca (idParticipante, idAtividade, status, checkin)
VALUES 
(1, 1, 'presente', '2024-03-27 09:45:00'),
(2, 1, 'presente', '2024-03-27 09:50:00'),
(3, 3, 'ausente', NULL),
(4, 4, 'pendente', NULL),
(5, 2, 'presente', '2024-03-27 14:55:00');

-- consultas básicas
SELECT * FROM evento;
SELECT * FROM participante;
SELECT * FROM atividade;
SELECT * FROM presenca;

-- consulta combinando informações
SELECT 
    e.nome AS evento,
    a.nome AS atividade,
    p.nome AS participante,
    pr.status AS presenca
FROM 
    presenca pr
    JOIN participante p ON pr.idParticipante = p.idParticipante
    JOIN atividade a ON pr.idAtividade = a.idAtividade
    JOIN evento e ON a.idEvento = e.idEvento;

-- atualizações de exemplo
UPDATE evento SET status = 'ativo' WHERE idEvento = 1;
UPDATE participante SET telefone = '(11) 98888-7777' WHERE idParticipante = 1;
UPDATE atividade SET vagas = vagas - 1 WHERE idAtividade = 1;
UPDATE presenca SET status = 'presente' WHERE idPresenca = 4;

-- exclusões com verificação de relacionamentos
DELETE FROM presenca WHERE idPresenca = 5;
DELETE FROM atividade WHERE idAtividade = 5 AND NOT EXISTS (SELECT 1 FROM presenca WHERE idAtividade = 5);
DELETE FROM participante WHERE idParticipante = 5 AND NOT EXISTS (SELECT 1 FROM presenca WHERE idParticipante = 5);
DELETE FROM evento WHERE idEvento = 5 AND NOT EXISTS (SELECT 1 FROM atividade WHERE idEvento = 5);
