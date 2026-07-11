🤖 Orquestrador Inteligente de Atendimento via WhatsApp
Um sistema automatizado de atendimento ao cliente via WhatsApp, construído com Agentes de Inteligência Artificial e Google Sheets. Este projeto foi desenhado como uma solução em nuvem ágil e acessível para pequenos e médios negócios, utilizando uma arquitetura de múltiplos agentes para orquestrar pedidos, cadastros e dúvidas sem a necessidade de um banco de dados complexo neste estágio inicial.

📌 Visão Geral
O sistema recebe as mensagens do cliente pelo WhatsApp (incluindo suporte a transcrição de áudio), processa a intenção do usuário através de um Agente Orquestrador principal e direciona a solicitação para subagentes especializados. O banco de dados é mantido no Google Sheets, permitindo que os donos do negócio gerenciem produtos, preços, clientes e pedidos de forma visual e familiar.

🏗️ Arquitetura do Sistema
O fluxo é gerenciado por uma ferramenta de automação (como n8n) que orquestra as seguintes etapas:

Gatilho (WhatsApp): Recebe mensagens de texto e áudio.

Processamento de Áudio: Se for áudio, converte para texto.

Agente Orquestrador (IA): Analisa a intenção da mensagem e invoca a ferramenta (subagente) correta.

Subagentes: Executam tarefas específicas de leitura e escrita no Google Sheets ou no Documento de FAQ.

Resposta: O Orquestrador formula uma resposta amigável, clara e objetiva para o cliente no WhatsApp.

🧠 Agentes e Ferramentas
O coração do sistema é composto pelo Agente Orquestrador que delega tarefas para 4 subagentes especializados:

1. 🔍 Agente de Verificação de Cadastro
Função: Consulta a base de clientes no Google Sheets (por CPF, CNPJ ou telefone).

Segurança: Retorna estritamente o nome da empresa encontrada, preservando a privacidade do cliente. Solicitações de alteração de dados são sempre repassadas ao atendimento humano.

2. 📝 Agente de Cadastro
Função: Ativado quando um novo cliente chega ou quando a Verificação de Cadastro não encontra registros.

Ação: Coleta os dados faltantes de forma conversacional e insere (append) uma nova linha na aba de "Clientes Cadastrados" do Google Sheets.

3. 🛒 Agente de Pedidos
Função: Responsável por registrar as compras.

Ação: Lê os produtos disponíveis, pode calcular totais (usando a ferramenta de Calculadora) e insere o novo pedido na aba "Pedidos" do Google Sheets.

Regra: Só pode ser acionado se o cliente já possuir um cadastro confirmado.

4. ❓ Agente FAQ (Dúvidas Gerais)
Função: Responde a perguntas sobre a empresa (horários, localização, regras).

Ação: Consulta um documento de texto (Google Docs) contendo o FAQ da empresa e também lê a tabela de "Produtos" para informar preços e descrições atualizadas (ex: pães, doces e salgados).

🗄️ Banco de Dados Leve (Google Workspace)
Para manter o custo baixo e a facilidade de uso para os gestores, o banco de dados está estruturado diretamente no Google Drive:

Google Sheets (Digu Data): Possui abas como Produtos (com Nome, Categoria, Descrição e Preço), Clientes Cadastrados e Pedidos. O sistema tem permissão para ler catálogos e escrever novos cadastros/pedidos.

Google Docs (FAQ D'gusta): Um arquivo simples contendo as políticas, horários e informações institucionais lidas pela IA.

🛡️ Políticas e Segurança
Foco na Privacidade: Nenhuma informação sensível do cliente (histórico, endereço, telefones adicionais) é exposta no chat pela IA.

Atendimento Humano (Transbordo): Caso o cliente queira falar com um humano, ou necessite alterar dados sensíveis, a IA o instrui a digitar o comando exato: falar com atendente. Quando essa mensagem é detectada, a IA para de responder e o fluxo é repassado para a equipe da loja.

Prevenção de Alucinação: O Orquestrador é instruído a nunca inventar dados. Se a informação não estiver no FAQ ou na tabela, ele não tentará adivinhar.

🛠️ Tecnologias Utilizadas
Plataforma de Automação: n8n (para orquestrar os fluxos visuais).

Modelos de IA: LLMs para conversação, raciocínio (Agentes) e conversão de Áudio para Texto.

Banco de Dados / Armazenamento: APIs do Google Sheets e Google Drive.

Mensageria: API do WhatsApp Cloud.

Desenvolvido como uma solução prática, escalável e inteligente para o atendimento de pequenos e médios negócios.
