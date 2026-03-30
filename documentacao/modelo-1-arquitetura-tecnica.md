
# 1️⃣ Modelo – Arquitetura Técnica do Chatbot **Travall** 

> **Objetivo**: descrever a arquitetura técnica, integrações, segurança, observabilidade e fluxo de implantação do assistente virtual *Travall*, garantindo alinhamento com a especificação principal do projeto.  
> **Contexto**: o Travall centraliza e padroniza orientações de **Travel & Expenses (T&E) South America**, publicado no **Microsoft Teams (SA)**, autenticado por **Microsoft Entra ID**, com um **fluxo mínimo no Power Automate** para rota de ajuda/abertura de chamado.  

---

## 1. Visão Geral da Solução

```mermaid
graph TB
  subgraph "🏢 Visão Geral"
    A["**Nome da Solução**\nTravall – Assistente de T&E"]
    B["**Escopo Funcional (alto nível)**\nConsultas: política de viagens, hospedagem, passagens, adiantamento, VTM,\nprestações de contas/reembolso, NEO KDS (acesso & troubleshooting),\nlinks úteis (segurança, riscos, EBTA, hotéis parceiros)"]
    C["**Plataforma**\nMicrosoft Copilot Studio"]
    D["**Canal**\nMicrosoft Teams (South America)"]
  end
  style A fill:#E3F2FD,stroke:#1565C0
  style B fill:#E3F2FD,stroke:#1565C0
  style C fill:#E3F2FD,stroke:#1565C0
  style D fill:#E3F2FD,stroke:#1565C0
  ```
  ## 2. Arquitetura de Componentes

  ```mermaid

  graph TB
  subgraph CH["👥 Canais de Acesso"]
    TEAMS["Microsoft Teams (SA)"]
  end

  subgraph AG["🤖 Camada do Copilot Studio"]
    AGENT["Agente Travall (Copilot Studio)\n• Tópicos & Intents\n• RAG em documentos (KB)\n• Suporte multilíngue"]
    AUTH["Autenticação\nMicrosoft Entra ID (SSO)"]
    SAFETY["Responsible AI / Segurança de Conteúdo\n(filtros, anti-jailbreak, notas de transparência)"]
    DLP["Data Loss Prevention (por ambiente)"]
    KB["Knowledge Base\n• Políticas & Diretrizes T&E\n• Manuais/Guias & Troubleshooting NEO KDS"]
  end

  subgraph INT["🔗 Integrações"]
    SPO["SharePoint (repositório de documentos)"]
    PA["Power Automate\n(rota de ajuda/chamado)"]
    MAIL["E-mail T&E (fallback)"]
    LINKS["Links externos úteis\n(EBTA, segurança do viajante, hotéis parceiros, riscos, Flytour)"]
  end

  subgraph OBS["📈 Observabilidade & Telemetria"]
    AI["Application Insights"]
    CA["Copilot Analytics"]
  end

  CH --> AGENT
  AGENT --> AUTH
  AGENT --> SAFETY
  AGENT --> DLP
  AGENT --> KB
  KB --> SPO
  AGENT --> PA
  PA --> MAIL
  AGENT --> LINKS
  AGENT --> AI
  AI --> CA

  style AGENT fill:#E8EAF6,stroke:#283593
  style KB fill:#F3E5F5,stroke:#6A1B9A
  style AUTH fill:#FFEBEE,stroke:#C62828
  style SAFETY fill:#FFF3E0,stroke:#E65100
  style DLP fill:#FFFDE7,stroke:#F9A825
  ```
  ## 4. Conectores e Integrações

| Item | Conector / Mecanismo                 | Finalidade                                                            | Observações                                                     |
|------|--------------------------------------|----------------------------------------------------------------------|----------------------------------------------------------------|
| 1    | SharePoint Connector / upload nativo | Acesso a documentos normativos, políticas, manuais e guias (KB)     | Manter *single source of truth* no repositório corporativo     |
| 2    | Power Automate (flow simples)        | Rota de ajuda quando o bot não resolver (RF19)                       | Pode criar tarefa/chamado e/ou enviar e-mail T&E               |
| 3    | Teams Channel                        | Publicação para usuários SA                                          | Controle por Entra ID e DLP                                    |
| 4    | E-mail T&E                           | Contato humano de fallback                                           | Disclaimers e registro em telemetria                           |
| –    | (Fora do escopo)                     | Integrações transacionais (NEO KDS, etc.)                           | Não implementado nesta fase                                    |

## 5. Fontes de Dados e Conhecimento

  ```mermaid
  graph LR
  subgraph "📚 Fontes de Conhecimento"
    SP["SharePoint / Repositórios\n• Documentos normativos (PDF/Word)\n• Políticas de Viagem\n• Guias/Manuais / Troubleshooting NEO KDS"]
  end
  AGKB["🤖 Copilot Studio – Knowledge"]
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
  DEV["🛠️ DEV – Desenvolvimento\nCriação de tópicos e ingestão de documentos"]
  QA["🧪 QA/Homologação\nValidação de respostas e idiomas"]
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

