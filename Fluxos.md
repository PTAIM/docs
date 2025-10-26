# Diagramas de Sequência

Este documento apresenta os fluxos completos da plataforma de telemedicina através de diagramas de sequência.

---

## 1. Fluxo de Cadastro e Autenticação

### 1.1. Cadastro de Médico/Funcionário

```mermaid
sequenceDiagram
    actor Usuario as Médico/Funcionário
    participant Frontend
    participant Backend
    participant DB as Banco de Dados

    Usuario->>Frontend: Acessa página de cadastro
    Usuario->>Frontend: Preenche formulário (nome, email, CPF, senha, tipo)
    Frontend->>Backend: POST /auth/cadastro
    Backend->>Backend: Valida dados
    Backend->>Backend: Hash da senha
    Backend->>DB: Insere novo usuário
    DB-->>Backend: Confirmação
    Backend-->>Frontend: {id, nome, email, tipo_usuario}
    Frontend-->>Usuario: Cadastro realizado com sucesso
```

### 1.2. Login de Usuário

```mermaid
sequenceDiagram
    actor Usuario as Usuário (Médico/Funcionário/Paciente)
    participant Frontend
    participant Backend
    participant DB as Banco de Dados

    Usuario->>Frontend: Acessa página de login
    Usuario->>Frontend: Insere email/CPF e senha
    Frontend->>Backend: POST /auth/login
    Backend->>DB: Busca usuário por email/CPF
    DB-->>Backend: Dados do usuário
    Backend->>Backend: Valida senha
    Backend->>Backend: Gera JWT token
    Backend-->>Frontend: {access_token, usuario}
    Frontend->>Frontend: Armazena token
    Frontend-->>Usuario: Redireciona para dashboard
```

---

## 2. Fluxo de Solicitação de Exame

### 2.1. Médico Solicita Exame para Paciente Existente

```mermaid
sequenceDiagram
    actor Medico as Médico
    participant Frontend
    participant Backend
    participant DB as Banco de Dados
    participant Email as Serviço de Email

    Medico->>Frontend: Acessa tela de solicitação
    Medico->>Frontend: Busca paciente na barra de busca
    Frontend->>Backend: GET /pacientes?q=nome
    Backend->>DB: Busca pacientes
    DB-->>Backend: Lista de pacientes
    Backend-->>Frontend: {pacientes: [...]}
    Frontend-->>Medico: Exibe resultados

    Medico->>Frontend: Seleciona paciente
    Frontend-->>Medico: Exibe formulário

    Medico->>Frontend: Preenche dados do exame
    Note over Medico,Frontend: nome_exame, hipotese_diagnostica,<br/>detalhes_preparo

    Frontend->>Backend: POST /solicitacoes
    Backend->>Backend: Gera código UUID único
    Backend->>DB: Insere solicitação (status: AGUARDANDO_RESULTADO)
    DB-->>Backend: Confirmação

    Backend->>Email: Envia email ao paciente
    Note over Backend,Email: Inclui código da solicitação,<br/>login e senha
    Email-->>Backend: Email enviado

    Backend-->>Frontend: {id, codigo_solicitacao, status}
    Frontend-->>Medico: Solicitação criada com sucesso
```

### 2.2. Médico Solicita Exame e Cadastra Novo Paciente

```mermaid
sequenceDiagram
    actor Medico as Médico
    participant Frontend
    participant Backend
    participant DB as Banco de Dados
    participant Email as Serviço de Email

    Medico->>Frontend: Clica em "Cadastrar Novo Paciente"
    Frontend-->>Medico: Exibe formulário de cadastro

    Medico->>Frontend: Preenche dados do paciente
    Note over Medico,Frontend: nome, email, CPF, data_nascimento,<br/>telefone, sumário de saúde

    Frontend->>Backend: POST /pacientes
    Backend->>Backend: Gera senha temporária
    Backend->>DB: Insere novo paciente
    DB-->>Backend: {id, dados_paciente}

    Backend->>Email: Envia credenciais ao paciente
    Email-->>Backend: Email enviado

    Backend-->>Frontend: {id, nome, email, senha_temporaria}
    Frontend-->>Medico: Paciente cadastrado

    Note over Medico,Frontend: Médico prossegue para solicitar exame<br/>(fluxo 2.1 a partir da seleção)
```

---

## 3. Fluxo de Upload de Resultado pelo Funcionário

```mermaid
sequenceDiagram
    actor Paciente as Paciente
    actor Funcionario as Funcionário da Clínica
    participant Frontend
    participant Backend
    participant DB as Banco de Dados
    participant Storage as Armazenamento de Arquivos

    Paciente->>Paciente: Realiza exame na clínica
    Paciente->>Funcionario: Apresenta código da solicitação

    Funcionario->>Frontend: Acessa tela de envio
    Funcionario->>Frontend: Insere código da solicitação
    Funcionario->>Frontend: Preenche formulário
    Note over Funcionario,Frontend: data_realizacao, nome_laboratorio,<br/>observacoes, upload de arquivos

    Frontend->>Backend: POST /exames (multipart/form-data)
    Backend->>Storage: Faz upload dos arquivos
    Storage-->>Backend: URLs dos arquivos

    Backend->>DB: Insere registro de exame
    Backend->>DB: Atualiza status da solicitação<br/>(RESULTADO_ENVIADO)
    DB-->>Backend: Confirmação

    Backend-->>Frontend: {id, arquivos, message}
    Frontend-->>Funcionario: Resultado enviado com sucesso
```

---

## 4. Fluxo de Laudagem pelo Médico

### 4.1. Primeira Etapa - Seleção de Exames

```mermaid
sequenceDiagram
    actor Medico as Médico
    participant Frontend
    participant Backend
    participant DB as Banco de Dados

    Medico->>Frontend: Acessa tela de laudagem
    Frontend->>Backend: GET /pacientes
    Backend->>DB: Busca pacientes do médico
    DB-->>Backend: Lista de pacientes
    Backend-->>Frontend: {pacientes: [...]}
    Frontend-->>Medico: Exibe campo de seleção de paciente

    Medico->>Frontend: Filtra por paciente
    Frontend->>Backend: GET /exames?paciente_id=X
    Backend->>DB: Busca filtrada
    DB-->>Backend: Exames do paciente
    Backend-->>Frontend: {exames: [...]}

    Medico->>Frontend: Seleciona exames (checkboxes)
    Medico->>Frontend: Clica em "Próximo"
    Frontend->>Frontend: Armazena IDs selecionados
    Frontend-->>Medico: Redireciona para etapa 2
```

### 4.2. Segunda Etapa - Criação do Laudo

```mermaid
sequenceDiagram
    actor Medico as Médico
    participant Frontend
    participant Backend
    participant LLM as Serviço de LLM
    participant DB as Banco de Dados
    participant Email as Serviço de Email

    Frontend->>Backend: GET /exames/{id} (para cada exame selecionado)
    Backend->>DB: Busca detalhes dos exames
    DB-->>Backend: {id, nome_arquivo, url_arquivo, ...}
    Backend-->>Frontend: Dados completos dos exames
    Frontend-->>Medico: Renderiza carousel com imagens/PDFs

    alt Médico solicita análise de IA (opcional)
        Medico->>Frontend: Seleciona imagem e clica "Analisar com IA"
        Frontend->>Backend: POST /ai/analisar
        Backend->>LLM: Envia imagem para análise
        LLM-->>Backend: Resultado da análise
        Backend-->>Frontend: {analise: "..."}
        Frontend-->>Medico: Exibe análise da IA
    end

    Medico->>Frontend: Preenche formulário do laudo
    Note over Medico,Frontend: titulo, descricao

    Medico->>Frontend: Clica em "Salvar Rascunho"
    Frontend->>Backend: POST /laudos
    Note over Frontend,Backend: status: RASCUNHO
    Backend->>DB: Insere laudo
    Backend->>DB: Cria vínculos em LaudoExame
    DB-->>Backend: Confirmação
    Backend-->>Frontend: {id, status: "RASCUNHO"}
    Frontend-->>Medico: Rascunho salvo

    Medico->>Frontend: Revisa e clica em "Finalizar"
    Frontend->>Backend: PUT /laudos/{id}
    Note over Frontend,Backend: status: FINALIZADO
    Backend->>DB: Atualiza status do laudo
    DB-->>Backend: Confirmação

    Backend->>Email: Notifica paciente sobre laudo
    Email-->>Backend: Email enviado

    Backend-->>Frontend: {id, status: "FINALIZADO"}
    Frontend-->>Medico: Laudo finalizado com sucesso
```

---

## 5. Fluxo de Visualização pelo Paciente

### 5.1. Paciente Acessa suas Solicitações

```mermaid
sequenceDiagram
    actor Paciente as Paciente
    participant Frontend
    participant Backend
    participant DB as Banco de Dados


    Paciente->>Frontend: Acessa "Minhas solicitações"
    Frontend->>Backend: GET /solicitacoes
    Backend->>DB: Busca solicitações do paciente
    DB-->>Backend: Lista de solicitações
    Backend-->>Frontend: {solicitacoes: [...]}
    Frontend-->>Paciente: Exibe lista de solicitaçõee

    Paciente->>Frontend: Clica em uma solicitação
    Frontend->>Backend: GET /solicitacoes/{id}
    Backend->>DB: Busca detalhes
    DB-->>Backend: {id, codigo_solicitacao, exames, laudo}
    Backend-->>Frontend: Dados completos
    Frontend-->>Paciente: Exibe detalhes

    alt Solicitação ainda aguardando
        Frontend-->>Paciente: Exibe código único para levar à clínica
        Frontend-->>Paciente: Exibe status e instruções
    else Solicitação tem resultado enviado
        Frontend-->>Paciente: Exibe arquivos do exame (se disponível)
    end
```

### 5.2. Paciente Visualiza Laudo

```mermaid
sequenceDiagram
    actor Paciente as Paciente
    participant Frontend
    participant Backend
    participant DB as Banco de Dados

    Paciente->>Frontend: Acessa "Meus Laudos"
    Frontend->>Backend: GET /laudos
    Backend->>DB: Busca laudos do paciente
    DB-->>Backend: Lista de laudos
    Backend-->>Frontend: {laudos: [...]}
    Frontend-->>Paciente: Exibe lista de laudos

    Paciente->>Frontend: Clica em um laudo
    Frontend->>Backend: GET /laudos/{id}
    Backend->>DB: Busca detalhes completos
    DB-->>Backend: {titulo, descricao, exames, medico}
    Backend-->>Frontend: Dados completos do laudo

    Frontend-->>Paciente: Renderiza laudo formatado
    Note over Frontend,Paciente: Exibe: título, descrição,<br/>dados do médico, exames vinculados,<br/>imagens/PDFs dos resultados

    Paciente->>Frontend: Clica em "Baixar PDF" (opcional)
    Frontend->>Frontend: Gera PDF do laudo
    Frontend-->>Paciente: Download do arquivo
```

---

## 6. Fluxo do Dashboard do Médico

```mermaid
sequenceDiagram
    actor Medico as Médico
    participant Frontend
    participant Backend
    participant DB as Banco de Dados

    Medico->>Frontend: Acessa dashboard
    Frontend->>Backend: GET /dashboard/stats?periodo=30d

    Backend->>DB: Query: count solicitações
    DB-->>Backend: total_solicitacoes

    Backend->>DB: Query: count exames recebidos
    DB-->>Backend: exames_recebidos

    Backend->>DB: Query: count laudos emitidos
    DB-->>Backend: laudos_emitidos

    Backend->>DB: Query: count pacientes distintos
    DB-->>Backend: total_pacientes

    Backend->>DB: Query: solicitações agrupadas por mês
    DB-->>Backend: solicitacoes_por_mes[]

    Backend->>DB: Query: laudos agrupados por mês
    DB-->>Backend: laudos_por_mes[]

    Backend-->>Frontend: {resumo, solicitacoes_por_mes, laudos_por_mes}
    Frontend-->>Medico: Renderiza dashboard com gráficos
```

---

## 7. Fluxo de Atualização de Status

### 7.1. Médico Cancela Solicitação

```mermaid
sequenceDiagram
    actor Medico as Médico
    participant Frontend
    participant Backend
    participant DB as Banco de Dados
    participant Email as Serviço de Email

    Medico->>Frontend: Acessa lista de solicitações
    Medico->>Frontend: Seleciona solicitação
    Medico->>Frontend: Clica em "Cancelar"
    Frontend-->>Medico: Modal de confirmação

    Medico->>Frontend: Confirma cancelamento
    Frontend->>Backend: PUT /solicitacoes/{id}
    Note over Frontend,Backend: status: CANCELADO

    Backend->>DB: Atualiza status
    DB-->>Backend: Confirmação

    Backend->>Email: Notifica paciente (opcional)
    Email-->>Backend: Email enviado

    Backend-->>Frontend: {id, status: "CANCELADO"}
    Frontend-->>Medico: Solicitação cancelada
```

---

## Notas Técnicas

### Autenticação

- Todas as rotas protegidas requerem `Authorization: Bearer {token}` no header
- O token JWT contém: `user_id`, `email` `tipo`, `nome`, `exp`

### Status das Solicitações

1. **AGUARDANDO_RESULTADO**: Criada pelo médico, aguardando upload
2. **RESULTADO_ENVIADO**: Funcionário enviou o resultado
3. **CANCELADO**: Médico cancelou a solicitação

### Status dos Laudos

1. **RASCUNHO**: Laudo em elaboração
2. **FINALIZADO**: Laudo concluído e disponível ao paciente

### Observações de Segurança

- Pacientes só acessam suas próprias solicitações e laudos
- Funcionários não têm acesso a dados de médicos e pacientes
- Médicos só acessam pacientes vinculados às suas solicitações
- Arquivos médicos devem ser armazenados com controle de acesso rigoroso
