# Épicos do MVP

## Épico 1: Gestão de Perfis

#### Feature 1: Autenticação e Cadastro

- **História 1.1**: Como Médico ou Funcionário (Staff), quero poder me cadastrar na plataforma fornecendo meus dados profissionais para ter acesso ao sistema.
- **História 1.2**: Como Médico, Funcionário ou Paciente, quero poder fazer login de forma segura (email/cpf e senha) para acessar minhas funcionalidades personalizadas.

#### Feature 2: Gestão de Pacientes

- **História 2.1:** Como Médico, quero ter acesso a uma funcionalidade para cadastrar novos pacientes (com dados mínimos) para que eu possa criar solicitações de exames para eles.
- **História 2.2:** Como Médico, quero poder buscar e selecionar pacientes já existentes na base de dados para criar novas solicitações.

#### Feature 3: Perfil do Médico

- **História 3.1:** Como médico, quero preencher meu perfil com biografia, especialidades e demais dados, para que a plataforma tenha meu registro profissional.

#### Feature 4: Sumário de Saúde do Paciente

- **História 4.1:** Como Paciente, quero registrar (ou que o médico registre por mim) minha lista de alergias, medicamentos e condições preexistentes, para que o médico tenha um contexto clínico rápido durante a laudagem.

## Épico 2: Gestão de Exames

#### Feature 1: Criação de Solicitação de Exame

- **História 1.1:** Como Médico, quero, após selecionar um paciente, criar uma solicitação de exame (preenchendo os dados do exame), para que um código único seja gerado e o paciente notificado por email (com seus dados de acesso).

#### Feature 2: Acompanhamento de Solicitações

- **História 2.1:** Como Paciente, quero acessar a plataforma e visualizar uma lista das minhas solicitações de exame (com o código único) e seus status (Solicitado, Enviado, Cancelado).
- **História 2.2:** Como Médico, quero visualizar uma lista de minhas solicitações de exames (com filtros e busca) para poder gerenciar e acompanhar o status de cada uma.

#### Feature 3: Upload de Exames por Código

- **História 3.1:** Como Funcionário (Staff), quero acessar uma tela de envio onde eu possa inserir o código único da solicitação (fornecido pelo paciente).
- **História 3.2:** Como Funcionário, após inserir o código, quero poder preencher dados e fazer o upload dos arquivos de resultado (imagens/PDF) referentes àquela solicitação.
- **História 3.3:** Como Funcionário, ao confirmar o envio, quero que o status da solicitação mude para "Enviado", para que o exame entre na fila de laudagem do médico.

## Épico 3: Análise e Emissão de Laudos

#### Feature 1: Fila de Trabalho

- **História 1.1:** Como Médico, quero ter uma "caixa de entrada" de exames pendentes (status "Enviado"), onde possa visualizar os resultados enviados, para que eu possa iniciar meu processo de análise.

#### Feature 2: Tela de Laudagem

- **História 2.1:** Como Médico, quero selecionar um ou mais resultados (da fila) para acessar uma tela de laudagem, onde posso visualizar os uploads (imagens/PDFs) e redigir um laudo estruturado.
- **História 2.2:** Como Médico, dentro da tela de laudagem, quero ter a opção de solicitar uma análise de IA (LLM) sobre as imagens para auxiliar no diagnóstico (conforme prompt anterior).

#### Feature 3: Acesso ao Laudo

- **História 3.1:** Como Médico, quero acessar a plataforma e visualizar os laudos emitidos por mim para poder editar ou excluir quando necessário
- **História 3.2:** Como Paciente, quero acessar a plataforma e visualizar os laudos finalizados associados às minhas solicitações de exame.
