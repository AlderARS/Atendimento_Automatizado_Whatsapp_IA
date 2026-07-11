### 4. ❓ Agente FAQ (Dúvidas Gerais)
* **Função:** Responde a perguntas sobre a empresa (horários, localização, regras).
* **Ação:** Consulta um documento de texto (Google Docs) contendo o FAQ da empresa e também lê a tabela de "Produtos" para informar preços e descrições atualizadas (ex: pães, doces e salgados).

## 🗄️ Banco de Dados Leve (Google Workspace)

Para manter o custo baixo e a facilidade de uso para os gestores, o banco de dados está estruturado diretamente no Google Drive:
* **Google Sheets (`Digu Data`):** Possui abas como `Produtos` (com Nome, Categoria, Descrição e Preço), `Clientes Cadastrados` e `Pedidos`. O sistema tem permissão para ler catálogos e escrever novos cadastros/pedidos.
* **Google Docs (`FAQ D'gusta`):** Um arquivo simples contendo as políticas, horários e informações institucionais lidas pela IA.

## 🛡️ Políticas e Segurança
* **Foco na Privacidade:** Nenhuma informação sensível do cliente (histórico, endereço, telefones adicionais) é exposta no chat pela IA.
* **Atendimento Humano (Transbordo):** Caso o cliente queira falar com um humano, ou necessite alterar dados sensíveis, a IA o instrui a digitar o comando exato: `falar com atendente`. Quando essa mensagem é detectada, a IA para de responder e o fluxo é repassado para a equipe da loja.
* **Prevenção de Alucinação:** O Orquestrador é instruído a nunca inventar dados. Se a informação não estiver no FAQ ou na tabela, ele não tentará adivinhar.

## 🛠️ Tecnologias Utilizadas
* **Plataforma de Automação:** n8n (para orquestrar os fluxos visuais).
* **Modelos de IA:** LLMs para conversação, raciocínio (Agentes) e conversão de Áudio para Texto.
* **Banco de Dados / Armazenamento:** APIs do Google Sheets e Google Drive.
* **Mensageria:** API do WhatsApp Cloud.

---
*Desenvolvido como uma solução prática, escalável e inteligente para o atendimento de pequenos e médios negócios.*
