🔍 Análise Completa do seu Projeto PI - Plataforma Manguezal

📖 1. MINIMUNDO DO PROJETO (Descrição Detalhada)
Contexto:
Uma plataforma digital para monitoramento, denúncia e proteção de manguezais. O sistema conecta diferentes atores envolvidos na preservação desses ecossistemas costeiros.

Atores/Usuários:
Pescadores - Usuários que dependem do mangue para sustento

Comunidade local - Moradores das áreas próximas aos manguezais

ONGs Ambientais - Organizações de preservação

Empresas de Pescado - Indústrias relacionadas à pesca

Órgãos Governamentais - Fiscalização ambiental

Funcionalidades Principais:
A. Monitoramento Ambiental:
Registro de temperatura da água em diferentes pontos do manguezal

Histórico de medições para análise temporal

Alertas de mudanças bruscas

B. Sistema de Denúncias:
Reportar poluição (resíduos químicos, lixo)

Denunciar desmatamento ilegal

Registrar pesca predatória/ilegal

Outros problemas ambientais

C. Comunicação e Resposta:
Mensagens entre usuários

Respostas oficiais às denúncias

Atualização de status das denúncias

Notificações sobre ações tomadas

D. Gestão de Manguezais:
Cadastro de áreas de mangue

Localização geográfica precisa

Informações sobre estado, cidade, características

Fluxo Típico:
text
1. Pescador registra temperatura alta no Mangue X
2. Comunidade denuncia poluição no mesmo mangue  
3. ONG recebe notificação e analisa
4. Órgão ambiental responde com ações tomadas
5. Todos os envolvidos são notificados
6. Status da denúncia é atualizado
🎨 2. MODELO CONCEITUAL (Diagrama MER - Entidade-Relacionamento)
ENTIDADES PRINCIPAIS:
1. USUARIO
Atributos: id, nome, email, senha_hash, telefone, tipo_usuario

Cardinalidade:

Um USUARIO registra várias TEMPERATURAS (1:N)

Um USUARIO faz várias DENÚNCIAS (1:N)

Um USUARIO envia/recebe várias MENSAGENS (1:N)

Um USUARIO escreve várias RESPOSTAS (1:N)

2. MANGUE
Atributos: id, nome, localizacao, estado, cidade, descricao

Cardinalidade:

Um MANGUE tem várias TEMPERATURAS registradas (1:N)

Um MANGUE recebe várias DENÚNCIAS (1:N)

3. TEMPERATURA
Atributos: id, valor, data_hora, notas

Cardinalidade:

Uma TEMPERATURA pertence a um USUARIO (N:1)

Uma TEMPERATURA é de um MANGUE específico (N:1)

4. DENÚNCIA
Atributos: id, titulo, descricao, tipo, status, latitude, longitude, fotos

Cardinalidade:

Uma DENÚNCIA é feita por um USUARIO (N:1)

Uma DENÚNCIA se refere a um MANGUE (N:1)

Uma DENÚNCIA pode gerar várias RESPOSTAS (1:N)

Uma DENÚNCIA pode ser discutida em MENSAGENS (1:N)

5. RESPOSTA_DENUNCIA
Atributos: id, resposta, acao_tomada, status_update

Cardinalidade:

Uma RESPOSTA pertence a uma DENÚNCIA (N:1)

Uma RESPOSTA é escrita por um USUARIO (N:1)

6. MENSAGEM
Atributos: id, assunto, conteudo, lida

Cardinalidade:

Uma MENSAGEM tem um REMETENTE (usuário) (N:1)

Uma MENSAGEM tem um DESTINATÁRIO (usuário) (N:1)

Uma MENSAGEM pode referenciar uma DENÚNCIA (N:1) (opcional)

DIAGRAMA CONCEITUAL VISUAL:
text
     ┌─────────────┐
     │   USUARIO   │
     │  (Entidade) │
     └──────┬──────┘
            │
    ┌───────┼───────┐
    ▼       ▼       ▼
┌─────────┐ ┌─────────┐ ┌──────────┐
│TEMPERA- │ │ DENÚN-  │ │ MENSAGEM │
│ TURA    │ │  CIA    │ │          │
│(Entidade)│ │(Entidade)│ │(Entidade)│
└────┬────┘ └────┬────┘ └────┬─────┘
     │           │           │
     │           └─────┬─────┘
     │                 │
     ▼                 ▼
┌─────────┐     ┌──────────────┐
│ MANGUE  │     │ RESPOSTA_    │
│(Entidade)│     │ DENUNCIA    │
└─────────┘     │  (Entidade)  │
                └──────────────┘
💾 3. MODELO LÓGICO (Diagrama MR - Modelo Relacional)
TABELAS E ATRIBUTOS:
TABELA 1: usuarios
sql
CREATE TABLE usuarios (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    senha_hash VARCHAR(255) NOT NULL,
    telefone VARCHAR(15),
    tipo_usuario ENUM('pescador', 'ong', 'empresa', 'comum', 'governo') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
TABELA 2: mangues
sql
CREATE TABLE mangues (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    localizacao POINT NOT NULL,
    estado VARCHAR(50),
    cidade VARCHAR(100),
    descricao TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
TABELA 3: temperaturas
sql
CREATE TABLE temperaturas (
    id INT PRIMARY KEY AUTO_INCREMENT,
    usuario_id INT NOT NULL,
    mangue_id INT NOT NULL,
    temperatura DECIMAL(4,2) NOT NULL,
    data_registro DATETIME NOT NULL,
    notas TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id) ON DELETE CASCADE,
    FOREIGN KEY (mangue_id) REFERENCES mangues(id) ON DELETE CASCADE
);
TABELA 4: denuncias
sql
CREATE TABLE denuncias (
    id INT PRIMARY KEY AUTO_INCREMENT,
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
    
    FOREIGN KEY (mangue_id) REFERENCES mangues(id) ON DELETE CASCADE,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id) ON DELETE CASCADE
);
TABELA 5: resposta_denuncia
sql
CREATE TABLE resposta_denuncia (
    id INT PRIMARY KEY AUTO_INCREMENT,
    denuncia_id INT NOT NULL,
    usuario_id INT NOT NULL,
    resposta TEXT NOT NULL,
    acao_tomada VARCHAR(500),
    status_update ENUM('em_analise', 'resolvido', 'arquivado'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (denuncia_id) REFERENCES denuncias(id) ON DELETE CASCADE,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id) ON DELETE CASCADE
);
TABELA 6: mensagens
sql
CREATE TABLE mensagens (
    id INT PRIMARY KEY AUTO_INCREMENT,
    remetente_id INT NOT NULL,
    destinatario_id INT NOT NULL,
    assunto VARCHAR(200),
    conteudo TEXT NOT NULL,
    denuncia_id INT NULL,
    lida BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (remetente_id) REFERENCES usuarios(id) ON DELETE CASCADE,
    FOREIGN KEY (destinatario_id) REFERENCES usuarios(id) ON DELETE CASCADE,
    FOREIGN KEY (denuncia_id) REFERENCES denuncias(id) ON DELETE SET NULL
);
DIAGRAMA LÓGICO (MR) - RELACIONAMENTOS:
text
┌─────────────────────────────────────────────────────┐
│                    usuarios                         │
│  PK: id                                             │
│      nome, email, senha_hash, tipo_usuario, etc.    │
└─────────┬───────────────────────────────────────────┘
          │
          ├─────────────────┐
          │                 │
          ▼                 ▼
┌─────────────────┐ ┌─────────────────┐
│  temperaturas   │ │   denuncias     │
│  PK: id         │ │  PK: id         │
│  FK: usuario_id │ │  FK: usuario_id │
│  FK: mangue_id  │ │  FK: mangue_id  │
└────────┬────────┘ └────────┬────────┘
         │                   │
         │                   ├─────────────────┐
         │                   │                 │
         ▼                   ▼                 ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│    mangues      │ │ resposta_denuncia│ │   mensagens    │
│   PK: id        │ │    PK: id       │ │   PK: id       │
│   localizacao   │ │ FK: denuncia_id │ │ FK: remetente_id│
└─────────────────┘ │ FK: usuario_id  │ │ FK:destinatario_id│
                    └─────────────────┘ │ FK: denuncia_id │
                                        └─────────────────┘
CARDINALIDADES NO MODELO RELACIONAL:
text
usuarios (1) ──────┐
                   ├──▶ temperaturas (N)
                   │        │
                   │        └──▶ mangues (1)
                   │
                   ├──▶ denuncias (N)
                   │        │
                   │        ├──▶ mangues (1)
                   │        │
                   │        └──▶ resposta_denuncia (N)
                   │                 │
                   │                 └──▶ usuarios (1)
                   │
                   └──▶ mensagens (N) como remetente
                            │
                            ├──▶ usuarios (1) como destinatario
                            │
                            └──▶ denuncias (0 ou 1)
