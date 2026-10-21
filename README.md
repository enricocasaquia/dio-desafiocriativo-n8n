# Automação de Captação e Acompanhamento de Leads com n8n

## 1\. Objetivo do projeto

Atue como um especialista em n8n, automação de processos e integração de APIs.

Crie uma automação completa no **n8n para captar, organizar, classificar e acompanhar leads automaticamente**, desde o primeiro contato até a conversão em cliente.

O objetivo é reduzir tarefas manuais, agilizar o atendimento comercial, evitar a perda de oportunidades e melhorar a conversão de potenciais clientes em vendas.

A solução deve ser modular, confiável, escalável e fácil de manter, com tratamento de erros, prevenção de duplicidades e possibilidade de intervenção humana.

## 2\. Público

A automação será utilizada por uma pequena empresa que possui uma equipe comercial responsável por atender potenciais clientes, apresentar produtos ou serviços, enviar propostas e realizar acompanhamentos comerciais.

Os principais usuários serão:

- **Vendedores:** receberão notificações sobre novos leads e acompanharão as oportunidades.
- **Gestores comerciais:** monitorarão o desempenho do funil de vendas por meio de relatórios.
- **Administradores:** serão responsáveis pelas configurações, integrações e manutenção do workflow.

## 3\. Ferramentas envolvidas

### 3.1. n8n

Será a plataforma central de automação, responsável por conectar as ferramentas, executar as regras de negócio e controlar o fluxo de informações.

### 3.2. Google Forms

Será utilizado para captar leads por meio de formulários contendo informações como nome, telefone, e-mail, empresa e produto ou serviço de interesse.

A integração deverá utilizar um mecanismo confiável de detecção de novas respostas, como um gatilho de formulário ou Google Sheets, conforme a configuração disponível.

### 3.3. WhatsApp Business Platform

Será utilizado para receber mensagens, enviar boas-vindas e realizar follow-ups.

A integração deverá utilizar uma API oficial ou um provedor compatível com a WhatsApp Business Platform, respeitando as regras de envio, os modelos de mensagem aprovados e as permissões aplicáveis.

### 3.4. Google Sheets

Será utilizado como base inicial para armazenar e acompanhar os leads.

A planilha deverá conter, no mínimo, as seguintes colunas:

| Campo | Descrição |
| --- | --- |
| `lead_id` | Identificador único do lead |
| `nome` | Nome do contato |
| `telefone` | Telefone normalizado |
| `email` | E-mail do contato |
| `origem` | Formulário ou WhatsApp |
| `interesse` | Produto ou serviço desejado |
| `classificacao` | Frio, morno ou quente |
| `status` | Etapa atual do atendimento |
| `responsavel` | Vendedor responsável |
| `consentimento` | Situação da autorização para comunicação |
| `ultimo_contato` | Data e hora do último contato |
| `proximo_followup` | Data prevista para o próximo acompanhamento |
| `tentativas_contato` | Quantidade de tentativas realizadas |
| `ultima_interacao` | Data e hora da última resposta recebida |

Para volumes maiores ou operações com múltiplos usuários atualizando os mesmos registros, considerar a migração para um CRM ou banco de dados com garantias mais robustas de integridade.

### 3.5. Gmail

Será utilizado para notificar a equipe comercial sobre novos leads, oportunidades prioritárias, falhas importantes e resumos diários.

### 3.6. OpenAI API

Será utilizada para interpretar mensagens recebidas e sugerir uma classificação do interesse comercial.

A IA deverá retornar dados estruturados, como:

- Categoria do interesse.
- Resumo da necessidade.
- Nível de prioridade.
- Intenção de compra.
- Grau de confiança da classificação.

A classificação da IA deverá ser tratada como uma recomendação sujeita às regras de negócio, e não como uma decisão comercial definitiva.

---

## 4\. Arquitetura geral do workflow

O projeto deverá ser dividido em workflows menores para facilitar a manutenção e o diagnóstico de erros.

A arquitetura proposta contém cinco fluxos:

1. **Workflow A — Captação e cadastro de leads.**
2. **Workflow B — Classificação e primeiro atendimento.**
3. **Workflow C — Follow-up automático.**
4. **Workflow D — Atualização do funil comercial.**
5. **Workflow E — Relatório diário e monitoramento.**

Essa divisão evita concentrar toda a lógica em um único workflow e permite testar cada etapa separadamente.

## 5\. Workflow A — Captação e cadastro de leads

### Objetivo

Receber os dados dos leads, validar as informações, verificar duplicidades e registrar os contatos na base de dados.

### Nós do n8n

| Ordem | Nó | Função |
| --- | --- | --- |
| 1 | `Google Sheets Trigger` ou gatilho equivalente | Detectar novos registros oriundos do formulário. |
| 2 | `Edit Fields (Set)` | Padronizar os campos recebidos. |
| 3 | `Code` | Normalizar telefone, e-mail e outros dados. |
| 4 | `IF` | Validar os campos obrigatórios. |
| 5 | `Google Sheets` | Consultar registros existentes. |
| 6 | `IF` | Verificar se o lead já está cadastrado. |
| 7 | `Google Sheets` | Criar ou atualizar o registro conforme a regra de negócio. |
| 8 | `Execute Sub-workflow` | Acionar o workflow de classificação e atendimento. |

Os nomes de alguns nós podem variar conforme a versão do n8n e a integração instalada.

### Lógica de funcionamento

**Etapa 1 — Captura dos dados**

O workflow será iniciado quando uma nova resposta estiver disponível na fonte de captação.

Os dados recebidos deverão ser convertidos para um formato padronizado, por exemplo:

```
{
  "nome": "Maria Silva",
  "telefone": "+5511999999999",
  "email": "maria@example.com",
  "origem": "Google Forms",
  "interesse": "Consultoria comercial"
}
```

O exemplo acima representa dados fictícios para demonstrar a estrutura.

**Etapa 2 — Normalização**

O nó `Edit Fields (Set)` deverá organizar os campos e definir valores padrão quando apropriado.

O nó `Code` poderá ser utilizado para:

- Remover espaços desnecessários.
- Padronizar o telefone com código do país e DDD.
- Converter o e-mail para letras minúsculas.
- Eliminar caracteres inválidos.
- Gerar um identificador de processamento, quando necessário.

A normalização deverá preservar os dados originais relevantes para auditoria.

**Etapa 3 — Validação**

O nó `IF` verificará se os dados obrigatórios foram preenchidos corretamente.

Exemplos de condições:

- O nome está preenchido.
- Existe um telefone ou e-mail válido.
- A origem do lead foi identificada.
- O formato do contato é aceitável.

Se os dados forem inválidos, o workflow deverá encaminhá-los para um caminho de rejeição controlada, registrando o motivo e evitando o envio de mensagens.

**Etapa 4 — Verificação de duplicidade**

O workflow deverá pesquisar a base utilizando o telefone normalizado e, quando disponível, o e-mail.

A lógica será:

- Se não existir um registro correspondente, criar um novo lead.
- Se já existir, atualizar somente os campos pertinentes, sem sobrescrever dados comerciais importantes indiscriminadamente.
- Se houver conflito de identidade, encaminhar o registro para revisão humana.

O telefone ou e-mail, isoladamente, nem sempre é suficiente para resolver todos os conflitos. Por isso, a regra de atualização deverá considerar a origem do dado, a data de atualização e os identificadores disponíveis.

**Etapa 5 — Encaminhamento**

Após validar e registrar o lead, o workflow acionará o Workflow B.

### Resultado esperado

Todo lead válido deverá ser registrado uma única vez na base lógica do sistema e encaminhado para classificação e atendimento.

---

## 6\. Workflow B — Classificação e primeiro atendimento

### Objetivo

Classificar os leads, definir a prioridade comercial, enviar uma mensagem inicial quando permitido e notificar o vendedor responsável.

### Nós do n8n

| Ordem | Nó | Função |
| --- | --- | --- |
| 1 | `Execute Sub-workflow Trigger` | Receber os dados do Workflow A. |
| 2 | `OpenAI` ou `HTTP Request` | Analisar a mensagem e sugerir uma classificação. |
| 3 | `Structured Output Parser` ou validação equivalente | Garantir a estrutura esperada da resposta da IA. |
| 4 | `IF` | Avaliar a confiança e a validade da classificação. |
| 5 | `Switch` | Direcionar o lead conforme a prioridade. |
| 6 | `HTTP Request` | Enviar a mensagem pela integração oficial do WhatsApp. |
| 7 | `Gmail` | Notificar o responsável comercial. |
| 8 | `Google Sheets` | Atualizar classificação, status e data do contato. |

### Lógica de funcionamento

**Etapa 1 — Análise do lead**

A mensagem e os dados relevantes serão enviados ao modelo de IA para identificar a necessidade do potencial cliente.

Exemplo de instrução para a IA:

> Analise a mensagem do potencial cliente e retorne um objeto estruturado com a categoria de interesse, um resumo da necessidade, a prioridade comercial e o nível de confiança. Não invente informações ausentes. Se os dados forem insuficientes ou ambíguos, indique que é necessária uma análise humana.

A resposta esperada poderá seguir esta estrutura:

```
{
  "categoria": "consultoria",
  "prioridade": "alta",
  "resumo": "Busca melhorar o processo comercial.",
  "confianca": 0.92,
  "revisao_humana": false
}
```

Os valores são ilustrativos.

**Etapa 2 — Validação da resposta**

O workflow deverá verificar se a resposta:

- Contém todos os campos necessários.
- Utiliza categorias previamente definidas.
- Possui um nível de confiança dentro do intervalo permitido.
- Não contém informações incompatíveis com as regras comerciais.

Se a resposta estiver incompleta ou apresentar baixa confiança, o lead deverá ser encaminhado para análise humana.

**Etapa 3 — Definição da prioridade**

O nó `Switch` poderá separar os leads em três categorias:

- **Alta prioridade:** demonstrou intenção clara de compra ou solicitou contato comercial.
- **Média prioridade:** demonstrou interesse, mas precisa de mais informações.
- **Baixa prioridade:** ainda está explorando opções ou não forneceu informações suficientes.

A prioridade não deverá ser determinada apenas pela IA. Regras explícitas, como solicitação de proposta, prazo de compra e perfil de atendimento, também deverão ser consideradas.

**Etapa 4 — Envio da mensagem de boas-vindas**

Quando permitido pelas regras de consentimento e pela política do WhatsApp, o workflow enviará uma mensagem personalizada.

Exemplo:

> Olá, {{nome}}\! Obrigado por entrar em contato. Recebemos seu interesse em {{interesse}} e nossa equipe poderá ajudar você com os próximos passos.

A mensagem deverá utilizar os dados reais do lead por meio das expressões do n8n, sem inserir dados não validados.

O envio deverá respeitar as regras aplicáveis à janela de atendimento do WhatsApp e aos modelos de mensagem aprovados. O simples preenchimento de um formulário não deve ser tratado automaticamente como autorização irrestrita para qualquer comunicação.

**Etapa 5 — Notificação comercial**

O nó `Gmail` enviará uma notificação ao vendedor contendo:

- Nome do lead.
- Telefone e e-mail disponíveis.
- Origem do contato.
- Interesse declarado.
- Classificação e prioridade.
- Resumo da mensagem.
- Link para o registro na base, quando disponível.

Leads de alta prioridade poderão gerar notificações com maior urgência, de acordo com a política comercial definida.

**Etapa 6 — Atualização do cadastro**

O registro será atualizado com a classificação, o responsável, o status e as informações do primeiro contato.

### Resultado esperado

A equipe comercial receberá leads organizados e priorizados, enquanto o potencial cliente terá uma primeira resposta adequada quando o envio for permitido.

---

## 7\. Workflow C — Follow-up automático

### Objetivo

Identificar leads que ainda não responderam e programar novas tentativas de contato sem enviar mensagens indevidas ou repetidas.

### Nós do n8n

| Ordem | Nó | Função |
| --- | --- | --- |
| 1 | `Schedule Trigger` | Executar a verificação em intervalos definidos. |
| 2 | `Google Sheets` | Buscar leads com acompanhamento pendente. |
| 3 | `Loop Over Items` | Processar os leads individualmente ou em lotes controlados. |
| 4 | `IF` | Verificar status, consentimento e prazo do próximo contato. |
| 5 | `IF` | Verificar o limite de tentativas. |
| 6 | `HTTP Request` | Enviar o follow-up pelo provedor do WhatsApp. |
| 7 | `Google Sheets` | Registrar a tentativa e atualizar a próxima data. |
| 8 | `Gmail` | Notificar a equipe quando for necessária intervenção humana. |

### Lógica de funcionamento

**Etapa 1 — Execução programada**

O nó `Schedule Trigger` iniciará o workflow em horários definidos, por exemplo, a cada hora durante o expediente.

O intervalo deverá ser ajustado de acordo com o volume de leads, a capacidade da equipe e as regras de comunicação.

**Etapa 2 — Seleção dos leads elegíveis**

O workflow deverá buscar somente os leads que atendam simultaneamente aos critérios:

- O status está em uma etapa que permite follow-up.
- A data do próximo contato já foi atingida.
- O contato pode receber mensagens conforme o consentimento registrado e as regras aplicáveis.
- O lead não solicitou interrupção das comunicações.
- Não existe uma resposta recente que torne o follow-up desnecessário.
- O limite de tentativas ainda não foi alcançado.

**Etapa 3 — Limite de tentativas**

Como regra inicial, definir no máximo três tentativas de acompanhamento após o contato inicial, com intervalos configuráveis.

Exemplo de programação:

| Tentativa | Intervalo sugerido |
| --- | --- |
| Primeiro follow-up | 1 dia após o contato inicial |
| Segundo follow-up | 3 dias após o primeiro follow-up |
| Terceiro follow-up | 5 dias após o segundo follow-up |

Esses intervalos são parâmetros iniciais e deverão ser adaptados ao ciclo de vendas.

O workflow deverá verificar novamente a elegibilidade imediatamente antes de enviar cada mensagem.

**Etapa 4 — Mensagem de acompanhamento**

Exemplo de mensagem:

> Olá, {{nome}}\! Gostaria de saber se você conseguiu avaliar as informações que enviamos. Se ainda tiver interesse, podemos ajudar com os próximos passos.

O conteúdo deverá ser adequado ao contexto e não deverá afirmar que o cliente demonstrou interesse em algo que não foi efetivamente registrado.

**Etapa 5 — Registro da tentativa**

Após uma confirmação de envio do provedor, o workflow registrará:

- Data e hora da tentativa.
- Número da tentativa.
- Identificador da mensagem no provedor.
- Próxima data de acompanhamento.
- Status do envio.

Uma resposta de aceitação da API não garante, por si só, que a mensagem foi entregue ou lida. Se necessário, o sistema deverá utilizar webhooks de status para acompanhar a entrega.

**Etapa 6 — Encerramento do acompanhamento**

Quando o lead responder, solicitar interrupção das mensagens, for convertido, for marcado como perdido ou atingir o limite de tentativas, o workflow deverá interromper os follow-ups automáticos.

### Resultado esperado

O sistema realizará acompanhamentos de maneira organizada, com limites de frequência e sem insistir em contatos que já responderam ou não desejam continuar recebendo mensagens.

---

## 8\. Workflow D — Atualização do funil comercial

### Objetivo

Manter o status dos leads atualizado e impedir que mensagens automáticas sejam enviadas quando a oportunidade já estiver em outra etapa do processo comercial.

### Nós do n8n

| Nó | Função |
| --- | --- |
| `Webhook` | Receber atualizações de sistemas externos, quando aplicável. |
| `Google Sheets Trigger` | Detectar alterações na base, se suportado pela configuração utilizada. |
| `Edit Fields (Set)` | Padronizar os dados da atualização. |
| `Switch` | Identificar a nova etapa do funil. |
| `IF` | Verificar se a transição de status é permitida. |
| `Google Sheets` | Atualizar o registro do lead. |
| `Gmail` | Notificar eventos comerciais relevantes. |

### Status sugeridos

- **Novo:** lead cadastrado e ainda não atendido.
- **Em atendimento:** vendedor iniciou o contato.
- **Proposta enviada:** uma proposta comercial foi encaminhada.
- **Convertido:** venda concluída.
- **Perdido:** oportunidade encerrada sem conversão.
- **Aguardando retorno:** existe uma interação pendente do cliente.

### Lógica de funcionamento

Quando o status de um lead mudar, o workflow deverá validar a alteração e atualizar os campos relacionados.

Exemplos:

- Se o lead for marcado como **Convertido**, cancelar os follow-ups pendentes.
- Se for marcado como **Perdido**, interromper mensagens comerciais automáticas e registrar o motivo quando disponível.
- Se for marcado como **Proposta enviada**, programar uma tarefa de acompanhamento conforme o processo comercial.
- Se for marcado como **Em atendimento**, impedir que o workflow de follow-up envie mensagens incompatíveis com a conversa em andamento.

Se a planilha for utilizada como interface de atualização, será necessário definir como alterações manuais serão detectadas. Para operações com maior concorrência, auditoria ou necessidade de controle de permissões, recomenda-se utilizar um CRM como fonte principal dos estados comerciais.

### Resultado esperado

O status comercial será a referência para determinar quais automações podem ser executadas, evitando conflitos entre o atendimento humano e as mensagens automáticas.

---

## 9\. Workflow E — Relatório diário e monitoramento

### Objetivo

Gerar um resumo diário da operação comercial e identificar falhas de integração.

### Nós do n8n

| Ordem | Nó | Função |
| --- | --- | --- |
| 1 | `Schedule Trigger` | Executar o relatório em horário configurado. |
| 2 | `Google Sheets` | Consultar os leads registrados. |
| 3 | `Code` ou nós de agregação | Calcular os indicadores. |
| 4 | `Edit Fields (Set)` | Organizar o resumo. |
| 5 | `Gmail` | Enviar o relatório para os responsáveis. |

### Indicadores sugeridos

O relatório deverá apresentar:

- Quantidade de novos leads no período.
- Leads por origem.
- Leads classificados como alta, média e baixa prioridade.
- Contatos aguardando atendimento.
- Follow-ups pendentes.
- Propostas enviadas.
- Leads convertidos.
- Leads perdidos.
- Erros de processamento relevantes.

A taxa de conversão deverá ser calculada com um denominador claramente definido. Por exemplo, conversões sobre os leads captados em uma mesma coorte ou período, evitando comparar vendas de um período com entradas de outro sem explicitar essa diferença.

### Exemplo de relatório

**Resumo comercial diário — {{data}}**

- Novos leads: 25
- Leads de alta prioridade: 8
- Contatos pendentes: 4
- Propostas enviadas: 6
- Vendas concluídas: 3
- Leads perdidos: 2
- Falhas de integração: 1

Os valores são apenas ilustrativos.

O relatório deverá utilizar os registros efetivos do sistema, sem estimar ou inventar resultados ausentes.

---

## 10\. Tratamento de erros e confiabilidade

A automação deverá incluir mecanismos específicos para falhas.

### Nós recomendados

- `Error Trigger`: iniciar um workflow de tratamento quando ocorrer uma falha não tratada em outro workflow.
- `IF`: verificar o tipo de resposta recebida de uma integração.
- `Wait`: aguardar antes de uma nova tentativa, quando apropriado.
- `HTTP Request`: configurar timeout e tratamento de respostas de APIs.
- `Google Sheets` ou um banco de dados: registrar os eventos de erro.
- `Gmail`: notificar o administrador sobre falhas críticas.

### Estratégia de novas tentativas

Para falhas temporárias, como limite de requisições ou indisponibilidade momentânea de um serviço, implementar novas tentativas com intervalos progressivos.

Exemplo de política:

1. Primeira tentativa após uma falha temporária: aguardar 1 minuto.
2. Segunda tentativa: aguardar 5 minutos.
3. Terceira tentativa: aguardar 15 minutos.
4. Após o limite, registrar a falha e notificar o responsável.

Os intervalos são parâmetros iniciais e deverão respeitar os limites de cada API.

Erros permanentes, como dados inválidos ou falta de autorização, não deverão ser repetidos indefinidamente.

### Prevenção de duplicidade

Cada operação crítica deverá possuir um identificador de idempotência ou uma chave de negócio que permita identificar se o evento já foi processado.

Por exemplo, antes de enviar uma mensagem, o workflow deverá verificar se o mesmo evento já gerou um envio.

**Importante:** a simples consulta seguida de inserção no Google Sheets não garante exclusividade em execuções simultâneas. Para cenários com concorrência, utilizar uma base com restrições de unicidade, operações atômicas ou um mecanismo equivalente de controle.

---

## 11\. Regras de negócio e restrições

A implementação deverá respeitar as seguintes regras:

1. Não cadastrar o mesmo lead repetidamente.
2. Validar os dados antes de registrá-los.
3. Não enviar mensagens duplicadas para o mesmo evento.
4. Respeitar o consentimento, a LGPD e as políticas aplicáveis do WhatsApp.
5. Não enviar follow-ups para contatos que solicitaram parar de receber mensagens.
6. Limitar a quantidade de tentativas de acompanhamento.
7. Interromper mensagens automáticas quando o lead responder ou o status comercial exigir intervenção humana.
8. Encaminhar classificações ambíguas da IA para revisão humana.
9. Registrar erros e permitir novas tentativas controladas.
10. Manter um histórico das principais alterações e interações.
11. Não expor credenciais, tokens ou dados pessoais desnecessários nos logs.
12. Não considerar uma mensagem enviada com sucesso apenas porque o workflow chegou ao nó de envio; registrar a resposta do provedor e, quando necessário, acompanhar os eventos de entrega.

As credenciais deverão ser armazenadas no gerenciador de credenciais do n8n, nunca diretamente no código dos nós ou em arquivos exportados sem proteção.

---

## 12\. Configuração dos principais nós

### 12.1. `Schedule Trigger`

Configurar o fuso horário da operação e os horários de execução.

Será utilizado principalmente para follow-ups e relatórios diários.

### 12.2. `IF`

Utilizar para condições como:

- Dados obrigatórios válidos.
- Lead já cadastrado.
- Consentimento disponível.
- Lead elegível para follow-up.
- Classificação da IA suficientemente confiável.
- Status comercial compatível com a ação.

### 12.3. `Switch`

Utilizar para encaminhar os leads conforme categorias conhecidas, como prioridade, origem ou status comercial.

### 12.4. `Edit Fields (Set)`

Utilizar para criar um formato padronizado para os dados que circulam entre os workflows.

Exemplo de campos:

```
{
  "lead_id": "identificador_unico",
  "nome": "nome_do_lead",
  "telefone": "telefone_normalizado",
  "origem": "origem_do_contato",
  "prioridade": "media",
  "status": "Novo"
}
```

Os valores são exemplos de estrutura e deverão ser substituídos por expressões e dados reais dos nós anteriores.

### 12.5. `HTTP Request`

Utilizar quando for necessário integrar uma API que não possua um nó nativo adequado.

Configurar:

- Método HTTP.
- URL do endpoint oficial.
- Cabeçalhos de autenticação.
- Corpo da requisição.
- Timeout.
- Tratamento dos códigos de resposta.
- Política de repetição compatível com a API.

Os tokens de autenticação deverão ser armazenados nas credenciais do n8n.

### 12.6. `Google Sheets`

Utilizar para consultar, adicionar e atualizar registros.

É importante definir uma chave de busca consistente e mapear explicitamente os campos para evitar alterações incorretas nos registros.

### 12.7. `Gmail`

Utilizar para enviar notificações de novos leads, alertas e relatórios.

O conteúdo deverá ser construído com base nos dados efetivamente disponíveis, evitando incluir informações pessoais que não sejam necessárias para a finalidade da mensagem.

### 12.8. `OpenAI`

Utilizar para analisar o conteúdo do contato e produzir uma classificação estruturada.

A saída deverá ser validada antes de acionar outras etapas. O workflow também deverá funcionar de maneira segura quando a API de IA estiver indisponível.

---

## 13\. Testes antes de colocar em produção

A automação deverá passar pelos seguintes testes:

| Teste | Resultado esperado |
| --- | --- |
| Novo lead válido | Cadastro e encaminhamento realizados. |
| Lead duplicado | Registro existente atualizado conforme as regras, sem duplicação. |
| Telefone inválido | Lead bloqueado para envio e ocorrência registrada. |
| E-mail ausente | Processamento permitido apenas se os demais identificadores obrigatórios forem suficientes. |
| IA retorna dados inválidos | Encaminhamento para tratamento ou revisão humana. |
| Falha na API do WhatsApp | Erro registrado e nova tentativa controlada, quando apropriado. |
| Lead responde | Follow-ups futuros cancelados ou reavaliados. |
| Lead solicita interrupção | Bloqueio de novas mensagens comerciais automáticas. |
| Lead convertido | Encerramento dos acompanhamentos pendentes. |
| Execuções simultâneas | Ausência de duplicação de operações críticas. |
| Falha no envio do relatório | Registro do erro e notificação do responsável. |

Os testes deverão ser realizados inicialmente em ambiente controlado, com contatos de teste e credenciais apropriadas.

---

## 14\. Implantação gradual

Recomenda-se implementar a solução em quatro fases.

### Fase 1 — Captação e cadastro

Implementar o Workflow A, validar os dados e garantir que os leads sejam registrados corretamente.

### Fase 2 — Classificação e notificações

Implementar o Workflow B, incluindo a análise por IA, o envio de boas-vindas e as notificações comerciais.

### Fase 3 — Follow-ups e funil comercial

Implementar os Workflows C e D, com controle de status, limites de contato e interrupção automática das mensagens quando necessário.

### Fase 4 — Relatórios e monitoramento

Implementar o Workflow E, revisar os logs e validar o comportamento em cenários de falha.

A automação deverá ser ativada gradualmente, começando com um volume pequeno de contatos e ampliando a operação após a validação dos resultados.

---

## 15\. Resultado final esperado

Ao concluir a implementação, a empresa terá um sistema de automação comercial capaz de:

- Captar leads de diferentes canais.
- Validar e organizar os dados automaticamente.
- Reduzir cadastros duplicados.
- Classificar oportunidades com auxílio de IA.
- Enviar mensagens iniciais quando permitido.
- Notificar os vendedores responsáveis.
- Realizar follow-ups com limites e regras definidos.
- Atualizar o funil comercial.
- Interromper contatos automáticos quando necessário.
- Gerar relatórios periódicos.
- Registrar falhas e permitir sua investigação.

## 16\. Entregáveis solicitados

Com base nesta especificação, desenvolva:

1. A arquitetura final dos workflows no n8n.
2. A lista de nós, na ordem de execução, com suas configurações.
3. As expressões necessárias para mapear os dados entre os nós.
4. Os exemplos de requisições e respostas das APIs envolvidas.
5. O modelo da planilha com os campos necessários.
6. Os prompts utilizados para a classificação dos leads pela IA.
7. O mecanismo de idempotência e prevenção de duplicidades.
8. O tratamento de erros e a estratégia de novas tentativas.
9. Um roteiro de testes com dados fictícios.
10. Um procedimento de implantação e manutenção.

Quando houver dependência de credenciais, identificadores de conta, endpoints específicos ou configurações externas, indique claramente o que precisa ser preenchido pelo administrador.

**Não invente credenciais, endpoints, IDs de provedores ou configurações que dependam de acesso ao ambiente.** Diferencie as partes que podem ser configuradas imediatamente das que exigem informações adicionais.

Sempre que possível, forneça um workflow exportável em JSON compatível com a versão do n8n utilizada, acompanhado das instruções para importar, configurar as credenciais e validar cada integração antes de ativar a automação em produção.

## Observação importante

Esse documento funciona como uma especificação técnica para orientar a criação da automação. O JSON exportável do n8n deverá ser gerado e validado conforme as versões dos nós e as APIs efetivamente utilizadas, especialmente para o WhatsApp e a integração com IA.