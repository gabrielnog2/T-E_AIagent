
# 1️⃣ Modelo – Arquitetura Técnica do Chatbot **Travall** 

---

## 1. Visão Geral da Solução

```mermaid

%%{init: {
  "flowchart": { "htmlLabels": true, "wrap": true, "nodeSpacing": 30, "rankSpacing": 25 },
  "themeVariables": { "fontSize": "12px" }
}}%%
graph TB
  classDef card fill:#E3F2FD,stroke:#1565C0,color:#0D47A1,stroke-width:1px,rx:8px,ry:8px;

  subgraph "🏢 Visão Geral"
    direction TB

    A["<b>Nome da Solução</b><br/>Travall – Assistente de T&amp;E"]

    B["<b>Escopo Funcional (alto nível)</b><br/>
       • Política de viagens<br/>
       • Hospedagem<br/>
       • Passagens<br/>
       • Adiantamento e VTM<br/>
       • Prestações de contas / reembolso<br/>
       • NEO KDS (acesso &amp; suporte)<br/>
       • Links úteis: segurança, riscos, EBTA, hotéis parceiros"]

    C["<b>Plataforma</b><br/>Microsoft Copilot Studio"]

    D["<b>Canal</b><br/>Microsoft Teams (South America)"]
  end

  class A,B,C,D card;

  ```
  ## 2. Arquitetura de Componentes

  ```mermaid


%%{init: {
  "flowchart": { "htmlLabels": true, "wrap": true, "nodeSpacing": 30, "rankSpacing": 30 },
  "themeVariables": { "fontSize": "12px" }
}}%%
graph TB
  %% ====== Classes de estilo ======
  classDef node fill:#E3F2FD,stroke:#1565C0,color:#0D47A1,stroke-width:1px,rx:8px,ry:8px;
  classDef agent fill:#E8EAF6,stroke:#283593,color:#1A237E,stroke-width:1px,rx:8px,ry:8px;
  classDef kb fill:#F3E5F5,stroke:#6A1B9A,color:#4A148C,stroke-width:1px,rx:8px,ry:8px;
  classDef auth fill:#FFEBEE,stroke:#C62828,color:#B71C1C,stroke-width:1px,rx:8px,ry:8px;
  classDef safety fill:#FFF3E0,stroke:#E65100,color:#BF360C,stroke-width:1px,rx:8px,ry:8px;
  classDef dlp fill:#FFFDE7,stroke:#F9A825,color:#F57F17,stroke-width:1px,rx:8px,ry:8px;

  %% ====== Subgrafo: Canais de Acesso ======
  subgraph CH["👥 Canais de Acesso"]
    direction TB
    TEAMS["Microsoft Teams (SA)"]
  end
  class TEAMS node;

  %% ====== Subgrafo: Camada do Copilot Studio ======
  subgraph AG["🤖 Camada do Copilot Studio"]
    direction TB
    AGENT["<b>Agente Travall (Copilot Studio)</b><br/>
           • Tópicos &amp; Intents<br/>
           • RAG em docs (KB)<br/>
           • Suporte multilíngue"]

    AUTH["<b>Autenticação</b><br/>Microsoft Entra ID (SSO)"]

    SAFETY["<b>Responsible AI / Segurança de Conteúdo</b><br/>
            (filtros, anti-jailbreak, etc)"]

    DLP["<b>Data Loss Prevention</b><br/>(por ambiente)"]

    KB["<b>Knowledge Base</b><br/>
        • Políticas; Diretrizes T&E<br/>
        • Manuais/Guias &amp; Troubleshooting NEO KDS"]
  end
  class AGENT agent;
  class AUTH auth;
  class SAFETY safety;
  class DLP dlp;
  class KB kb;

  %% ====== Subgrafo: Integrações ======
  subgraph INT["🔗 Integrações"]
    direction TB
    SPO["SharePoint<br/>(repositório de documentos)"]
    PA["Power Automate<br/>(rota de ajuda/chamado)"]
    DV["Dataverse<br/>(controle &amp; acompanhamento)"]
    MAIL["E-mail T&amp;E<br/>(fallback)"]
    LINKS["Links externos úteis<br/>(EBTA, segurança do viajante,<br/>hotéis parceiros, riscos, Flytour)"]
  end
 class SPO,PA,MAIL,LINKS,DV node;

  %% ====== Subgrafo: Observabilidade & Telemetria ======
  subgraph OBS["📈 Observabilidade &amp; Telemetria"]
    direction TB
    AI["Application Insights"]
    CA["Copilot Analytics"]
  end
  class AI,CA node;

  %% ====== Relações ======
  CH --> AGENT
  AGENT --> AUTH
  AGENT --> SAFETY
  AGENT --> DLP
  AGENT --> KB
  KB --> SPO
  AGENT --> PA
  PA --> MAIL
  AGENT --> DV
  AGENT --> LINKS
  AGENT --> AI
  AI --> CA

  ```
  ## 4. Conectores e Integrações

| Item | Conector / Mecanismo                 | Finalidade                                                            | Observações                                                     |
|------|--------------------------------------|----------------------------------------------------------------------|----------------------------------------------------------------|
| 1    | SharePoint Connector / upload nativo | Acesso a documentos normativos, políticas, manuais e guias (KB)     | Manter *single source of truth* no repositório corporativo     |
| 2    | Power Automate (flow simples)        | Rota de ajuda quando o bot não resolver (RF19)                       | Pode criar tarefa/chamado e/ou enviar e-mail T&E               |
| 3    | Teams Channel                        | Publicação para usuários SA                                          | Controle por Entra ID e DLP                                    |
| 4    | E-mail T&E                           | Contato humano de fallback                                           | Disclaimers e registro em telemetria                           |

## 5. Fontes de Dados e Conhecimento

  ```mermaid
  graph LR
  subgraph "📚 Fontes de Conhecimento"
    SP["SharePoint / Repositórios\n• Documentos normativos (PDF/Word)\n• Políticas de Viagem\n• Guias/Manuais / Troubleshooting NEO KDS"]
  end
  AGKB["🤖 Copilot Studio – KB"]
  SP --> AGKB
  style AGKB fill:#E8EAF6,stroke:#283593
   ```
   | # | Tipo de Fonte           | Local          | Conteúdo                                           | Owner                     | Mecanismo                 |
|---|--------------------------|----------------|----------------------------------------------------|----------------------------|----------------------------|
| 1 | Documentos (PDF/Word)    | SharePoint T&E | Política de Viagens, Diretrizes Internas           | Gestão de Viagens (T&E)    | Upload nativo / SP Connector |
| 2 | Manuais & Guias (PDF/Word)  | SharePoint T&E | Processos T&E SA, NEO KDS – erros comuns           | Gestão de Viagens (T&E)    | Upload nativo / SP Connector |

## 6. Ambientes de Pipeline e Publicação
  ```mermaid

graph LR
  DEV["🛠️ DEV – Desenvolvimento\nLógica e base de dados"]
  QA["🧪 QA/Homologação\nValidação de respostas"]
  PROD["🚀 PRODUÇÃO\nSuporte ao usuário interno"]
  DEV -->|CI/CD| QA -->|Aprovação T&E| PROD
  style DEV fill:#FFF3E0,stroke:#E65100
  style QA fill:#FFF9C4,stroke:#F9A825
  style PROD fill:#E8F5E9,stroke:#2E7D32
   ```
   ## 7. Fluxo de Dados (Data Flow)
   
```mermaid

   sequenceDiagram
  autonumber
  participant U as Usuário (Empregado SA)
  participant T as Microsoft Teams
  participant A as Agente Travall (Copilot Studio)
  participant E as Entra ID (SSO)
  participant KB as KB (SharePoint)
  participant P as Power Automate (Rota de ajuda)
  participant M as E-mail T&E

  U->>T: 1) Mensagem/pergunta sobre T&E / NEO KDS
  T->>A: 2) Encaminha conversa
  A->>E: 3) Autenticação (SSO)
  E-->>A: 4) Token válido
  A->>KB: 5) Recupera políticas, guias e troubleshooting
  KB-->>A: 6) Conteúdo relevante
  A-->>U: 7) Resposta estruturada (idioma do usuário)
  alt Bot não consegue responder (RF19)
    A->>P: 8) Aciona fluxo simples (coleta contexto)
    P->>M: 9) Envia e-mail/abre chamado para T&E
    P-->>U: 10) Confirmação e prazo de retorno
  end
```

