# 🤖 Assistente Virtual IA - Indústria de Panificação

Um sistema automatizado de atendimento ao cliente via WhatsApp, orquestrado no **n8n**, com inteligência artificial generativa. Este projeto foi desenvolvido para otimizar o fluxo de pedidos, responder dúvidas frequentes e gerenciar cadastros de uma indústria de panificações.

---

## 🏗️ Arquitetura e Infraestrutura

O ambiente de execução foi desenhado para rodar de forma contínua e segura:

*   **Docker:** O n8n é hospedado localmente via container Docker, garantindo isolamento e estabilidade da imagem da aplicação.
*   **Ngrok:** Utilizado para criar um túnel seguro (HTTPS), expondo o ambiente local para a internet, permitindo a comunicação perfeita com os webhooks da API Oficial do WhatsApp e outras integrações externas.
*   **Automação de Inicialização:** Um script `.bat` foi criado para garantir que a infraestrutura suba automaticamente ao ligar a máquina. Ele inicializa o Ngrok na mesma porta do n8n local e abre o navegador automaticamente no ambiente de trabalho do n8n.

---

## 🔄 Fluxo Central n8n e Lógica de Roteamento

O fluxo foi arquitetado para ser resiliente, rápido e inteligente, suportando tanto texto quanto áudio.

### 1. Recepção e Triagem (Input)
*   **Gatilho:** Webhook recebendo dados da **API Oficial do WhatsApp**.
*   **Ação Imediata:** Requisição HTTP POST para ativar o status de *Typing* (digitando) no WhatsApp do cliente.
*   **Roteamento (Switch Node):**
    *   **Caminho de Texto:** Passa por um nó `EditFields` ("JáTexto") e segue para o formatador central.
    *   **Caminho de Áudio:** O arquivo é baixado via nó `Download Media`. Um `HTTP Request` com o token do WhatsApp extrai o arquivo, que é enviado via `HTTP Request` para a **API do Groq** (modelo `whisper-large-v3-turbo`) para transcrição ultrarrápida. O texto transcrito passa pelo nó "AgoraTexto" e segue para o formatador central.
*   O nó `EditFields` ("Formatado") unifica as duas linhas, normalizando os dados para a lógica de negócio.

### 2. Regras de Negócio e Transbordo
*   **Transbordo Humano:** Um nó `IF` avalia se a intenção do cliente é "falar com atendente". 
    *   Se **SIM**: A conversa é imediatamente redirecionada, uma notificação é enviada ao atendente responsável e uma mensagem de confirmação de aguardo é enviada ao cliente.
    *   Se **NÃO**: O fluxo segue para o atendimento via IA.

### 3. Controle de Latência
*   **Subworkflow de Timeout:** Conectado antes da IA, este nó monitora o tempo de processamento. Se a resposta da IA levar mais de 20 segundos, ele dispara automaticamente uma mensagem amigável: *"Aguarde só mais um instante, estou processando seu pedido"*.

### 4. Inteligência Artificial e Orquestração
O cérebro do atendimento é o **Agente Orquestrador**, alimentado pelo modelo **Gemini flash 3.6**. Ele possui uma janela de contexto (memória) das últimas 20 mensagens.
Sua instrução principal exige que ele identifique se o cliente possui cadastro antes de qualquer pedido, orquestrando o fluxo através de **5 Subagentes Especializados**:

1.  **Agente de Verificação de Cadastro:** Consulta a base para validar clientes.
2.  **Agente de Cadastro:** Adiciona novos clientes ao banco de dados.
3.  **Agente de Perguntas Frequentes (FAQ):** Conectado ao Google Docs para ler as políticas e dúvidas comuns.
4.  **Agente de Anotação de Pedido:** Processa e registra a solicitação de produtos.
5.  **Agente de Alteração de Pedidos:** Gerencia modificações em pedidos previamente feitos.

*Nota: Todos os agentes possuem prompts com instruções rigorosas de segurança (nível de linha) e ferramentas (Tools) para interagir com o banco de dados.*

### 5. Finalização e Pós-Processamento
Após a resposta gerada pelo Orquestrador ser enviada ao cliente:
*   **Notificação de Alteração:** Se a IA alterou um pedido, um nó `IF` envia ao gestor: *"PEDIDO ALTERADO!"*
*   **Notificação de Novo Pedido:** Se a IA criou um pedido, um nó `IF` envia ao gestor: *"PEDIDO NOVO ANOTADO!"*
*   **Normalização de Dados:** Caso qualquer pedido seja criado/alterado, um `Call Subworkflow` é ativado. Ele acessa a tabela `Pedidos`, desmembra o conteúdo detalhado e reescreve na tabela `Itens Pedidos` (linha por linha, iterando a quantidade, subtotal e repetindo o `id_pedido` para garantir a atomicidade dos dados).

### 6. Sistema de Alerta de Erros
*   Um fluxo global de tratamento de erros está acoplado ao ambiente. Qualquer falha de execução nos nós dispara imediatamente um alerta via WhatsApp para o desenvolvedor, permitindo reações em tempo real.

---

## 🗄️ Modelagem do Banco de Dados

O banco de dados foi estruturado no **Google Sheets**, utilizando IDs de 7 dígitos para garantir eficiência no armazenamento, integridade relacional e escalabilidade para futuras análises de dados (Data Analysis). O acesso ocorre via API do Google em uma pasta privada compartilhada com a diretoria.

### Estrutura das Tabelas:

| Tabela | Permissão da IA | Colunas |
| :--- | :--- | :--- |
| **Biscoitos** | Leitura | `id_biscoito`, `nome biscoito`, `categoria`, `peso_gramas_un`, `qtd_fardo`, `qtd_atacado_minimo`, `preco_fardo`, `preco_atacado`, `preco_varejo` |
| **Pães** | Leitura | `id_pao`, `nome_pao`, `categoria`, `peso_gramas_un`, `qtd_atacado_minimo`, `preco_atacado`, `preco_varejo`, `observacao` |
| **Clientes Cadastrados** | Leitura / Escrita | `id_cliente`, `cpf/CNPJ`, `telefone contato`, `nome empresa`, `rua&numero`, `bairro&cidade`, `data_cadastro` |
| **Pedidos** | Leitura / Escrita | `id_pedido`, `cpf/CNPJ`, `data pedido`, `detalhamento pedido`, `total` |
| **Itens Pedidos** | Escrita (Subworkflow) | `id_pedido`, `nome produto`, `quantidade`, `data saída`, `subtotal` |
| **Análise** | Uso Analítico | Tabela reservada para cruzamento de dados, métricas financeiras e geração de relatórios de BI baseados nos IDs relacionais. |

*(O FAQ da empresa está armazenado de forma separada em um arquivo **Google Docs**, acessado exclusivamente pelo Agente de Perguntas Frequentes).*

---

## 🚀 Como iniciar o projeto localmente

1.  Verifique se o **Docker** está rodando em sua máquina.
2.  Execute o arquivo `start_env.bat` (Este script irá iniciar o container do n8n, criar o túnel ngrok na mesma porta e abrir o navegador).
3.  Atualize a URL do webhook no painel de desenvolvedores do WhatsApp Cloud API com o novo domínio gerado pelo Ngrok.
4.  Garanta que as credenciais do Google Cloud (para as Sheets e Docs) e da API do Groq e Gemini estejam ativas no gerenciador de credenciais do n8n.

---
*Desenvolvido por: Andrew Alves Rodrigues de Souza*
