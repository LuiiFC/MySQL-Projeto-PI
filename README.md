🌿 Plataforma de Monitoramento e Denúncia de Manguezais

Equipe: Luis Felipe, Pedro Ruan, Pedro Henrique e Davyson da Silva
    
📋 ÍNDICE
Minimundo do Projeto

Modelo Conceitual (MER)

Modelo Lógico (MR)

Script SQL

Instalação

Como Contribuir

🎯 MINIMUNDO DO PROJETO
Contexto e Problema
Os manguezais são ecossistemas costeiros vitais que enfrentam graves ameaças como poluição, desmatamento ilegal e pesca predatória. A falta de um sistema centralizado para monitoramento e denúncia dificulta a proteção eficaz dessas áreas.

Solução Proposta
Uma plataforma digital colaborativa que permite o registro de condições ambientais, denúncia de irregularidades e comunicação entre diferentes atores envolvidos na preservação dos manguezais.

Atores/Usuários do Sistema
Tipo de Usuário	Função	Permissões
Pescadores	Trabalham nos manguezais	Registrar temperaturas, fazer denúncias, receber alertas
Comunidade Local	Moradores das áreas próximas	Denunciar irregularidades, acompanhar status
ONGs Ambientais	Organizações de preservação	Responder denúncias, analisar dados, enviar mensagens
Empresas de Pescado	Indústrias relacionadas	Acompanhar condições, comunicar-se com pescadores
Órgãos Governamentais	Fiscalização ambiental	Dar respostas oficiais, atualizar status de casos
Funcionalidades Principais
1. 🎣 Registro de Temperatura
2. 
Medições periódicas da temperatura da água

Histórico temporal por manguezal

Alertas de mudanças bruscas

2. ⚠️ Sistema de Denúncias
Tipos de denúncia:

Poluição (resíduos químicos, lixo)

Desmatamento ilegal

Pesca predatória/ilegal

Outros problemas ambientais

Anexos: Fotos georreferenciadas

Status: Pendente → Em análise → Resolvido/Arquivado

3. 💬 Sistema de Comunicação
Mensagens diretas entre usuários

Discussões específicas sobre denúncias

Notificações de atualizações

4. 🗺️ Gestão de Manguezais
Cadastro de áreas de mangue

Localização geográfica precisa (coordenadas)

Informações descritivas e administrativas


🎨 MODELO CONCEITUAL (DIAGRAMA MER)
Entidades Principais
1. USUARIO
Descrição: Pessoa ou organização que utiliza o sistema

Atributos:

id_usuario (Identificador único)

Nome (Nome completo)

Email (Endereço eletrônico)

Senha_usuario (Credencial de acesso)

CPF_Usuario (Cadastro de Pessoa Física)

Telefone (Contato)

created_at (Data de cadastro)

2. MANGUE
Descrição: Área de manguezal monitorada

Atributos:

id_mangues (Identificador único)

Nome_Mangues (Denominação da área)

Localização (Coordenadas geográficas)

Estado (Unidade federativa)

Cidade (Município)

Descrição (Características ambientais)

3. TEMPERATURA
Descrição: Registro de medição térmica

Atributos:

idTemperatura (Identificador único)

Temperatura_Registro (Valor em °C)

data_registro (Data e hora da medição)

Notes (Observações complementares)

4. DENÚNCIA
Descrição: Reporte de problema ambiental

Atributos:

idDenúncias (Identificador único)

Título (Resumo do problema)

Descrição (Detalhamento)

Tipo_Reclamação (Categoria)

Status (Situação atual)

Latitude/Longitude (Localização exata)

Fotos (Evidências visuais)

5. MENSAGEM
Descrição: Comunicação entre usuários

Atributos:

idMessages (Identificador único)

Subjetivo (Assunto)

Conteudo (Corpo da mensagem)

6. REGISTRO
Descrição: Histórico de atividades

Atributos:

idRegistros (Identificador único)

Histórico (Data/hora do registro)


Cardinalidades
Um USUARIO registra várias TEMPERATURAS (1:N)

Um USUARIO faz várias DENÚNCIAS (1:N)

Um USUARIO envia/recebe várias MENSAGENS (1:N)

Um MANGUE tem várias TEMPERATURAS registradas (1:N)

Um MANGUE recebe várias DENÚNCIAS (1:N)

Uma DENÚNCIA pode gerar várias MENSAGENS (1:N)

O sistema mantém REGISTROS de todas as atividades

💾 MODELO LÓGICO (DIAGRAMA MR)
Tabelas do Banco de Dados
1. TABELA usuarios

CREATE TABLE usuarios (
    id_usuario INT PRIMARY KEY AUTO_INCREMENT,
    Nome VARCHAR(100) NOT NULL,
    Email VARCHAR(100) UNIQUE NOT NULL,
    Senha_usuario VARCHAR(255) NOT NULL,
    CPF_Usuario VARCHAR(14) UNIQUE,
    Telefone VARCHAR(15),
    tipo_usuario ENUM('pescador', 'ong', 'empresa', 'comum', 'governo') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

2. TABELA mangues
sql
CREATE TABLE mangues (
    id_mangues INT PRIMARY KEY AUTO_INCREMENT,
    Nome_Mangues VARCHAR(100) NOT NULL,
    Localizacao POINT NOT NULL,
    Estado VARCHAR(50),
    Cidade VARCHAR(100),
    Descricao TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_estado_cidade (Estado, Cidade),
    SPATIAL INDEX idx_localizacao (Localizacao)
);
3. TABELA temperaturas
sql
CREATE TABLE temperaturas (
    id_temperatura INT PRIMARY KEY AUTO_INCREMENT,
    usuario_id INT NOT NULL,
    mangue_id INT NOT NULL,
    temperatura_registro DECIMAL(4,2) NOT NULL,
    data_registro DATETIME NOT NULL,
    notas TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    FOREIGN KEY (mangue_id) REFERENCES mangues(id_mangues) ON DELETE CASCADE,
    INDEX idx_data_registro (data_registro),
    INDEX idx_mangue_data (mangue_id, data_registro)
);
4. TABELA denuncias
sql
CREATE TABLE denuncias (
    id_denuncia INT PRIMARY KEY AUTO_INCREMENT,
    mangue_id INT NOT NULL,
    usuario_id INT NOT NULL,
    titulo VARCHAR(200) NOT NULL,
    descricao TEXT NOT NULL,
    tipo_denuncia ENUM('poluicao', 'desmatamento', 'pesca_ilegal', 'outros') NOT NULL,
    status ENUM('pendente', 'em_analise', 'resolvido', 'arquivado') DEFAULT 'pendente',
    latitude DECIMAL(10,8),
    longitude DECIMAL(11,8),
    fotos JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (mangue_id) REFERENCES mangues(id_mangues) ON DELETE CASCADE,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    INDEX idx_status (status),
    INDEX idx_usuario_denuncia (usuario_id, created_at DESC),
    INDEX idx_tipo_denuncia (tipo_denuncia)
);
5. TABELA mensagens
sql
CREATE TABLE mensagens (
    id_mensagem INT PRIMARY KEY AUTO_INCREMENT,
    remetente_id INT NOT NULL,
    destinatario_id INT NOT NULL,
    assunto VARCHAR(200),
    conteudo TEXT NOT NULL,
    denuncia_id INT NULL,
    lida BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (remetente_id) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    FOREIGN KEY (destinatario_id) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    FOREIGN KEY (denuncia_id) REFERENCES denuncias(id_denuncia) ON DELETE SET NULL,
    INDEX idx_destinatario (destinatario_id, lida, created_at DESC),
    INDEX idx_remetente (remetente_id, created_at DESC)
);
6. TABELA registros
sql
CREATE TABLE registros (
    id_registro INT PRIMARY KEY AUTO_INCREMENT,
    usuario_id INT NOT NULL,
    mangue_id INT NULL,
    acao VARCHAR(50) NOT NULL,
    detalhes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    FOREIGN KEY (mangue_id) REFERENCES mangues(id_mangues) ON DELETE SET NULL,
    INDEX idx_usuario_acao (usuario_id, acao),
    INDEX idx_data_acao (created_at, acao)

);

⚙️ SCRIPT SQL COMPLETO

     -- ============================================
-- BANCO DE DADOS: PLATAFORMA MANGUEZAIS
-- ============================================

CREATE DATABASE IF NOT EXISTS plataforma_manguezais;
USE plataforma_manguezais;

-- Desativar verificações temporariamente
SET FOREIGN_KEY_CHECKS = 0;

-- Tabela USUARIOS
DROP TABLE IF EXISTS usuarios;
CREATE TABLE usuarios (
    id_usuario INT PRIMARY KEY AUTO_INCREMENT,
    Nome VARCHAR(100) NOT NULL,
    Email VARCHAR(100) UNIQUE NOT NULL,
    Senha_usuario VARCHAR(255) NOT NULL,
    CPF_Usuario VARCHAR(14) UNIQUE,
    Telefone VARCHAR(15),
    tipo_usuario ENUM('pescador', 'ong', 'empresa', 'comum', 'governo') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    -- Índices
    INDEX idx_email (Email),
    INDEX idx_tipo_usuario (tipo_usuario),
    INDEX idx_cpf (CPF_Usuario)
);

-- Tabela MANGUES
DROP TABLE IF EXISTS mangues;
CREATE TABLE mangues (
    id_mangues INT PRIMARY KEY AUTO_INCREMENT,
    Nome_Mangues VARCHAR(100) NOT NULL,
    Localizacao POINT NOT NULL,
    Estado VARCHAR(50),
    Cidade VARCHAR(100),
    Descricao TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Índices
    SPATIAL INDEX idx_localizacao (Localizacao),
    INDEX idx_estado_cidade (Estado, Cidade),
    INDEX idx_nome (Nome_Mangues)
);

-- Tabela TEMPERATURAS
DROP TABLE IF EXISTS temperaturas;
CREATE TABLE temperaturas (
    id_temperatura INT PRIMARY KEY AUTO_INCREMENT,
    usuario_id INT NOT NULL,
    mangue_id INT NOT NULL,
    temperatura_registro DECIMAL(4,2) NOT NULL,
    data_registro DATETIME NOT NULL,
    notas TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Chaves estrangeiras
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    FOREIGN KEY (mangue_id) REFERENCES mangues(id_mangues) ON DELETE CASCADE,
    
    -- Índices
    INDEX idx_data_registro (data_registro DESC),
    INDEX idx_mangue_data (mangue_id, data_registro DESC),
    INDEX idx_usuario_data (usuario_id, data_registro DESC),
    INDEX idx_temperatura (temperatura_registro)
);

-- Tabela DENÚNCIAS
DROP TABLE IF EXISTS denuncias;
CREATE TABLE denuncias (
    id_denuncia INT PRIMARY KEY AUTO_INCREMENT,
    mangue_id INT NOT NULL,
    usuario_id INT NOT NULL,
    titulo VARCHAR(200) NOT NULL,
    descricao TEXT NOT NULL,
    tipo_denuncia ENUM('poluicao', 'desmatamento', 'pesca_ilegal', 'outros') NOT NULL,
    status ENUM('pendente', 'em_analise', 'resolvido', 'arquivado') DEFAULT 'pendente',
    latitude DECIMAL(10,8),
    longitude DECIMAL(11,8),
    fotos JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    -- Chaves estrangeiras
    FOREIGN KEY (mangue_id) REFERENCES mangues(id_mangues) ON DELETE CASCADE,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    
    -- Índices
    INDEX idx_status (status),
    INDEX idx_tipo_denuncia (tipo_denuncia),
    INDEX idx_usuario_denuncia (usuario_id, created_at DESC),
    INDEX idx_mangue_denuncia (mangue_id, created_at DESC),
    INDEX idx_data_criacao (created_at DESC),
    INDEX idx_location (latitude, longitude)
);

-- Tabela MENSAGENS
DROP TABLE IF EXISTS mensagens;
CREATE TABLE mensagens (
    id_mensagem INT PRIMARY KEY AUTO_INCREMENT,
    remetente_id INT NOT NULL,
    destinatario_id INT NOT NULL,
    assunto VARCHAR(200),
    conteudo TEXT NOT NULL,
    denuncia_id INT NULL,
    lida BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Chaves estrangeiras
    FOREIGN KEY (remetente_id) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    FOREIGN KEY (destinatario_id) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    FOREIGN KEY (denuncia_id) REFERENCES denuncias(id_denuncia) ON DELETE SET NULL,
    
    -- Índices
    INDEX idx_destinatario (destinatario_id, lida, created_at DESC),
    INDEX idx_remetente (remetente_id, created_at DESC),
    INDEX idx_denuncia (denuncia_id),
    INDEX idx_nao_lidas (destinatario_id, lida)
);

-- Tabela REGISTROS (Histórico)
DROP TABLE IF EXISTS registros;
CREATE TABLE registros (
    id_registro INT PRIMARY KEY AUTO_INCREMENT,
    usuario_id INT NOT NULL,
    mangue_id INT NULL,
    acao VARCHAR(50) NOT NULL,
    detalhes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Chaves estrangeiras
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    FOREIGN KEY (mangue_id) REFERENCES mangues(id_mangues) ON DELETE SET NULL,
    
    -- Índices
    INDEX idx_usuario_acao (usuario_id, acao),
    INDEX idx_data_acao (created_at, acao),
    INDEX idx_mangue_acao (mangue_id, acao)
);

-- Tabela RESPOSTAS_DENUNCIA (Adicional para melhor estrutura)
DROP TABLE IF EXISTS respostas_denuncia;
CREATE TABLE respostas_denuncia (
    id_resposta INT PRIMARY KEY AUTO_INCREMENT,
    denuncia_id INT NOT NULL,
    usuario_id INT NOT NULL,
    resposta TEXT NOT NULL,
    acao_tomada VARCHAR(500),
    status_update ENUM('em_analise', 'resolvido', 'arquivado'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Chaves estrangeiras
    FOREIGN KEY (denuncia_id) REFERENCES denuncias(id_denuncia) ON DELETE CASCADE,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    
    -- Índices
    INDEX idx_denuncia_resposta (denuncia_id, created_at),
    INDEX idx_usuario_resposta (usuario_id)
);

-- Reativar verificações
SET FOREIGN_KEY_CHECKS = 1;

-- ============================================
-- INSERIR DADOS DE EXEMPLO
-- ============================================

-- Usuários exemplo
INSERT INTO usuarios (Nome, Email, Senha_usuario, tipo_usuario, Telefone) VALUES
('João Silva', 'joao.pescador@email.com', '$2y$10$hashedpassword1', 'pescador', '(11) 99999-9999'),
('ONG Mangue Vivo', 'contato@manguevivo.org', '$2y$10$hashedpassword2', 'ong', '(11) 88888-8888'),
('Maria Santos', 'maria.comunidade@email.com', '$2y$10$hashedpassword3', 'comum', '(11) 77777-7777');

-- Manguezais exemplo
INSERT INTO mangues (Nome_Mangues, Localizacao, Estado, Cidade, Descricao) VALUES
('Manguezal de Santos', POINT(-23.9675, -46.3328), 'SP', 'Santos', 'Área de preservação ambiental com rica biodiversidade'),
('Mangue da Baía de Guanabara', POINT(-22.8125, -43.1544), 'RJ', 'Rio de Janeiro', 'Manguezal urbano em processo de recuperação');

-- Temperaturas exemplo
INSERT INTO temperaturas (usuario_id, mangue_id, temperatura_registro, data_registro, notas) VALUES
(1, 1, 28.5, '2024-01-15 10:30:00', 'Temperatura normal para o período'),
(1, 1, 30.2, '2024-01-20 14:00:00', 'Aumento preocupante na temperatura');

-- Denúncias exemplo
INSERT INTO denuncias (mangue_id, usuario_id, titulo, descricao, tipo_denuncia, status, latitude, longitude) VALUES
(1, 1, 'Vazamento de óleo', 'Encontrei manchas de óleo no rio principal do manguezal', 'poluicao', 'pendente', -23.9675, -46.3328),
(2, 3, 'Desmatamento ilegal', 'Árvores sendo cortadas durante a noite', 'desmatamento', 'em_analise', -22.8125, -43.1544);

-- Mensagens exemplo
INSERT INTO mensagens (remetente_id, destinatario_id, assunto, conteudo, denuncia_id) VALUES
(2, 1, 'Sobre sua denúncia', 'Olá João, estamos analisando sua denúncia sobre o vazamento de óleo.', 1),
(1, 2, 'Mais informações', 'Encontrei mais evidências do vazamento, posso enviar fotos?', 1);

-- Registros exemplo
INSERT INTO registros (usuario_id, mangue_id, acao, detalhes) VALUES
(1, 1, 'REGISTRO_TEMPERATURA', 'Registrou temperatura de 28.5°C no Manguezal de Santos'),
(1, 1, 'CRIACAO_DENUNCIA', 'Criou denúncia sobre vazamento de óleo');

-- ============================================
-- VISUALIZAR OS DADOS
-- ============================================

SELECT '✅ BANCO DE DADOS CRIADO COM SUCESSO!' AS Mensagem;

-- Mostrar todas as tabelas
SHOW TABLES;

-- Contar registros em cada tabela
SELECT 'usuarios' AS Tabela, COUNT(*) AS Quantidade FROM usuarios
UNION ALL
SELECT 'mangues', COUNT(*) FROM mangues
UNION ALL
SELECT 'temperaturas', COUNT(*) FROM temperaturas
UNION ALL
SELECT 'denuncias', COUNT(*) FROM denuncias
UNION ALL
SELECT 'mensagens', COUNT(*) FROM mensagens
UNION ALL
SELECT 'registros', COUNT(*) FROM registros
UNION ALL
SELECT 'respostas_denuncia', COUNT(*) FROM respostas_denuncia;

   
  
  

