# MediScan: Telemedicina para Análise de Imagens Médicas

![Lading Page](https://github.com/PTAIM/docs/blob/main/src/landing.png)

O Mínimo Produto Viável (MVP) da plataforma de telemedicina MediScan consiste em um sistema web focado na gestão do ciclo de vida de exames de imagem, conectando três perfis distintos: Médicos, Pacientes e Funcionários de clínicas. O fluxo principal se inicia com o Médico, que pode gerenciar pacientes (cadastrando novos ou buscando existentes) e criar solicitações de exames. O Paciente recebe automaticamente credenciais de acesso por e-mail para visualizar suas solicitações e um código único para o exame. De posse desse código, o Funcionário da clínica acessa uma tela restrita de upload para enviar os resultados do exame (imagens ou PDFs). Uma vez enviado, o status da solicitação é atualizado, e o Médico pode iniciar o processo de laudagem, que ocorre em duas etapas: primeiro, selecionando os resultados pertinentes e, segundo, acessando uma interface que exibe os arquivos do exame e um formulário para a elaboração do laudo, com um recurso opcional de análise por LLM. O Paciente, por fim, utiliza seu acesso para visualizar tanto os resultados brutos enviados pela clínica quanto os laudos finalizados pelo médico.

![Como Funciona](https://github.com/PTAIM/docs/blob/main/src/como_funciona.png)
