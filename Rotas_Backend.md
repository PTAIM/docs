# Documentação das Rotas - Backend

## Autenticação

1. `POST /auth/login`

Efetuar o login na plataforma

**Request:**

```json
{
  "email": "string",
  "senha": "string"
}
```

**Response:**

```json
{
  "access_token": "string",
  "token_type": "bearer",
  "usuario": {
    "id": "number",
    "nome": "string",
    "email": "string",
    "tipo": "medico | funcionario | paciente"
  }
}
```

2. `POST /auth/cadastro`

Cadastro de usuários dos tipos `medico` e `funcionario`

**Request:**

```json
{
  "nome": "string",
  "email": "string",
  "telefone": "string",
  "cpf": "string",
  "senha": "string",
  "tipo": "medico | funcionario",
  "crm": "string (opcional, apenas para médicos)",
  "especialidade": "string (opcional, apenas para médicos)",
  "local_trabalho": "string (opcional, para funcionários)"
}
```

**Response:**

```json
{
  "id": "number",
  "nome": "string",
  "email": "string",
  "tipo": "medico | funcionario"
}
```

3. `GET /auth/me`

Ler dados básicos do usuário autenticado

**Headers:**

```
Authorization: Bearer {access_token}
```

**Response:**

```json
{
  "id": "number",
  "nome": "string",
  "email": "string",
  "tipo": "medico | funcionario"
}
```

## Usuários

Atualizar dados básicos

1. `PUT /usuarios/:usuarioId`

**Headers:**

```
Authorization: Bearer {access_token}
```

**Request:**

```json
{
  "nome": "string (opcional)",
  "email": "string (opcional)",
  "senha": "string (opcional)",
  "especialidade": "string (opcional, apenas médicos)",
  "local_trabalho": "string (opcional, apenas funcionários)"
}
```

**Response:**

```json
{
  "id": "number",
  "nome": "string",
  "email": "string",
  "message": "Dados atualizados com sucesso"
}
```

## Pacientes

1. `POST /pacientes/`

Criar novo paciente. Apenas usuários do tipo `medico`

**Headers:**

```
Authorization: Bearer {access_token}
```

**Request:**

```json
{
  "nome": "string",
  "email": "string",
  "telefone": "string",
  "cpf": "string",
  "senha": "string",
  "data_nascimento": "string",
  "sumario_saude": {
    "alergias": "string (opcional)",
    "medicacoes": "string (opcional)",
    "historico_doencas": "string (opcional)"
  }
}
```

**Response:**

```json
{
  "message": "Paciente cadastrado com sucesso. Email enviado com credenciais de acesso."
}
```

2. `PUT /pacientes/{paciente_id}`

Atualizar dados do paciente. Apenas usuários do tipo `medico`

3. `GET /pacientes/`

Listar pacientes de um médico (Pacientes se ligam a médicos através de **Solicitações de Exame**)

**Headers:**

```
Authorization: Bearer {access_token}
```

**Query Parameters:**

```
?search=string (busca por nome, email ou CPF)
&page=number
&limit=number
```

**Response:**

```json
{
  "pacientes": [
    {
      "id": "number",
      "nome": "string",
      "email": "string",
      "cpf": "string",
      "data_nascimento": "string",
      "telefone": "string"
    }
  ],
  "total": "number",
  "page": "number",
  "limit": "number"
}
```

4. `GET /pacientes/{paciente_id}`

Ler dados básicos de um paciente. Apenas usuários do tipo `medico`

**Headers:**

```
Authorization: Bearer {access_token}
```

**Response:**

```json
{
  "id": "number",
  "nome": "string",
  "email": "string",
  "telefone": "string",
  "cpf": "string",
  "senha": "string",
  "data_nascimento": "string",
  "sumario_saude": {
    "alergias": "string (opcional)",
    "medicacoes": "string (opcional)",
    "historico_doencas": "string (opcional)"
  }
}
```

## Solicitações

1. `POST /exames/solicitacoes`

Criar solicitação de exame. Apenas usuários do tipo `medico`.

**Headers:**

```
Authorization: Bearer {access_token}
```

**Request:**

```json
{
  "paciente_id": "number",
  "nome_exame": "string",
  "hipotese_diagnostica": "string",
  "detalhes_preparo": "string"
}
```

**Response:**

```json
{
  "id": "number",
  "codigo_solicitacao": "string (UUID)",
  "paciente_id": "number",
  "medico_id": "number",
  "nome_exame": "string",
  "status": "AGUARDANDO_RESULTADO",
  "data_solicitacao": "datetime",
  "message": "Solicitação criada com sucesso. Email enviado ao paciente."
}
```

2. `PUT /exames/solicitacoes/{solicitacao_id}`

Atualizar solicitação. Apenas usuários do tipo `medico`

**Headers:**

```
Authorization: Bearer {access_token}
```

**Request:**

```json
{
  "nome_exame": "string (opcional)",
  "hipotese_diagnostica": "string (opcional)",
  "detalhes_preparo": "string (opcional)",
  "status": "AGUARDANDO_RESULTADO | CANCELADO (opcional)"
}
```

**Response:**

```json
{
  "id": "number",
  "codigo_solicitacao": "string",
  "status": "string",
  "message": "Solicitação atualizada com sucesso"
}
```

3. `GET /exames/solicitacoes`

Listar solicitações. Usuários dos tipos `médico` e `paciente`

- **Médico**: Solicitações criadas por ele
- **Paciente**: Solicitações associadas a ele

**Headers:**

```
Authorization: Bearer {access_token}
```

**Query Parameters:**

```
?status=AGUARDANDO_RESULTADO | RESULTADO_ENVIADO | CANCELADO
&paciente_id=number
&data_inicio=string (YYYY-MM-DD)
&data_fim=string (YYYY-MM-DD)
&search=string (busca por código único ou nome do paciente)
&page=number
&limit=number
```

**Response:**

```json
{
  "solicitacoes": [
    {
      "id": "number",
      "codigo_solicitacao": "string",
      "paciente_id": "number",
      "paciente_nome": "string",
      "medico_id": "number",
      "medico_nome": "string",
      "nome_exame": "string",
      "status": "AGUARDANDO_RESULTADO | RESULTADO_ENVIADO | CANCELADO",
      "data_solicitacao": "datetime"
    }
  ],
  "total": "number",
  "page": "number",
  "limit": "number"
}
```

4. `GET /exames/solicitacoes/{solicitacao_id}`

Detalhes da solicitação. Usuários do tipo `médico` e `paciente`

**Headers:**

```
Authorization: Bearer {access_token}
```

**Response:**

```json
{
  "id": "number",
  "codigo_solicitacao": "string",
  "paciente_id": "number",
  "paciente_nome": "string",
  "medico_id": "number",
  "medico_nome": "string",
  "medico_crm": "string",
  "nome_exame": "string",
  "hipotese_diagnostica": "string",
  "detalhes_preparo": "string",
  "status": "AGUARDANDO_RESULTADO | RESULTADO_ENVIADO | CANCELADO",
  "data_solicitacao": "datetime",
  "exames": [
    {
      "id": "number",
      "data_realizacao": "datetime",
      "nome_laboratorio": "string"
    }
  ],
  "laudo": {
    "id": "number",
    "titulo": "string",
    "data_emissao": "datetime"
  }
}
```

## Exames

1. `POST /exames/resultados`

Upload do resultado do exame solicitado. Apenas usuários do tipo `funcionario`

**Headers:**

```
Authorization: Bearer {access_token}
Content-Type: multipart/form-data
```

**Request:**

```
codigo_solicitacao: string
data_realizacao: datetime
nome_laboratorio: string
observacoes: string (opcional)
arquivo: File
```

**Response:**

```json
{
  "message": "Exame enviado com sucesso. Status da solicitação atualizado para RESULTADO_ENVIADO."
}
```

2. `GET /exames/resultados`

Listar exames. Todos os usuários podem acessar

**Headers:**

```
Authorization: Bearer {access_token}
```

**Query Parameters:**

```
?solicitacao_id=number
&paciente_id=number
&status=pendente_laudo | laudado
&data_inicio=string (YYYY-MM-DD)
&data_fim=string (YYYY-MM-DD)
&page=number
&limit=number
```

**Response:**

```json
{
  "exames": [
    {
      "id": "number",
      "solicitacao_id": "number",
      "codigo_solicitacao": "string",
      "paciente_id": "number",
      "paciente_nome": "string",
      "nome_exame": "string",
      "data_realizacao": "datetime",
      "nome_laboratorio": "string",
      "tem_laudo": "boolean" (não acessivel para `funcionario`)
    }
  ],
  "total": "number",
  "page": "number",
  "limit": "number"
}
```

3. Detalhes (preview) exame (em dúvida como fazer isso)

4. `DELETE /exames/resultados/{exame_id}`

Deletar exames enviados. Apenas usuários do tipo `funcionario`

**Headers:**

```
Authorization: Bearer {access_token}
Content-Type: multipart/form-data
```

**Response:**

```json
{
  "message": "Exame deletado com sucesso."
}
```

## Laudos

1. `POST /laudos`

Criar laudo. Apenas usuários do tipo `medico`

**Headers:**

```
Authorization: Bearer {access_token}
```

**Request:**

```json
{
  "paciente_id": "number",
  "medico_id": "number",
  "titulo": "string",
  "descricao": "string",
  "exames_ids": ["number[]"]
}
```

**Response:**

```json
{
  "id": "number",
  "paciente_id": "number",
  "medico_id": "number",
  "titulo": "string",
  "status": "RASCUNHO",
  "data_emissao": "datetime",
  "exames_vinculados": ["number[]"],
  "message": "Laudo criado com sucesso."
}
```

2. `PUT /laudos/{id}`

Atualizar laudo. Apenas usuários do tipo `medico`

**Headers:**

```
Authorization: Bearer {access_token}
```

**Request:**

```json
{
  "titulo": "string (opcional)",
  "descricao": "string (opcional)",
  "status": "RASCUNHO | FINALIZADO (opcional)"
}
```

**Response:**

```json
{
  "id": "number",
  "status": "string",
  "message": "Laudo atualizado com sucesso"
}
```

3. `GET /laudos`

Listar laudos. Usuários dos tipos `medico` e `paciente`

**Headers:**

```
Authorization: Bearer {access_token}
```

**Query Parameters:**

```
?paciente_id=number
&medico_id=number
&status=RASCUNHO | FINALIZADO
&data_inicio=string (YYYY-MM-DD)
&data_fim=string (YYYY-MM-DD)
&search=string (busca por título)
&page=number
&limit=number
```

**Response:**

```json
{
  "laudos": [
    {
      "id": "number",
      "paciente_id": "number",
      "paciente_nome": "string",
      "medico_id": "number",
      "medico_nome": "string",
      "titulo": "string",
      "status": "RASCUNHO | FINALIZADO",
      "data_emissao": "datetime",
      "total_exames": "number"
    }
  ],
  "total": "number",
  "page": "number",
  "limit": "number"
}
```

4. `GET /laudos/{id}`

Detalhes laudo. Usuários dos tipos `medico` e `paciente`

**Headers:**

```
Authorization: Bearer {access_token}
```

**Response:**

```json
{
  "id": "number",
  "paciente_id": "number",
  "paciente_nome": "string",
  "paciente_cpf": "string",
  "medico_id": "number",
  "medico_nome": "string",
  "medico_crm": "string",
  "titulo": "string",
  "descricao": "string",
  "status": "RASCUNHO | FINALIZADO",
  "data_emissao": "datetime",
  "exames": [
    {
      "id": "number",
      "solicitacao_id": "number",
      "codigo_solicitacao": "string",
      "nome_exame": "string",
      "data_realizacao": "datetime",
      "nome_laboratorio": "string",
      "nome_arquivo": "string",
      "url_arquivo": "string"
    }
  ]
}
```

## Dashboard

1. `GET /dashboard/stats`

Stats dashboard. Apenas usuários do tipo `medico`

**Headers:**

```

Authorization: Bearer {access_token}

```

**Query Parameters:**

```

?periodo=7d | 30d | 90d | 1y | all

```

**Response:**

```json
{
  "resumo": {
    "total_solicitacoes": "number",
    "exames_recebidos": "number",
    "laudos_emitidos": "number",
    "total_pacientes": "number"
  },
  "solicitacoes_por_mes": [
    {
      "mes": "string (YYYY-MM)",
      "total": "number"
    }
  ],
  "laudos_por_mes": [
    {
      "mes": "string (YYYY-MM)",
      "total": "number"
    }
  ]
}
```
