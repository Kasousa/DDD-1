# Dinâmica: Design Estratégico do Projeto

## Objetivo
Identificar os subdomínios do projeto, classificá-los (Core, Supporting, Generic) e desenhar os bounded contexts, incluindo suas interações. Esse exercício ajudará a criar uma visão clara e estratégica do domínio.

---

## 1. Nome do Projeto
**Plataforma de Detecção Precoce e Monitoramento de Risco para Doença Renal Crônica**

---

## 2. Objetivo Principal do Projeto
Detectar precocemente risco de DRC e apoiar o acompanhamento longitudinal de pacientes na atenção básica, integrando dados clínicos e laboratoriais para priorizar triagem, disparar alertas e coordenar encaminhamentos antes de evolução para terapia renal substitutiva.

---

## 3. Identificação dos Subdomínios

| Subdomínio                                  | Descrição                                                                                                          | Tipo            |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | --------------- |
| **Gestão de Risco & Triagem**               | Calcula risco de DRC, identifica população elegível para rastreio (TFG/eGFR, UACR, comorbidades) e prioriza casos. | Core Domain     |
| **Modelos Preditivos de Progressão**        | Prediz progressão/agravamento (queda de TFG, necessidade de nefro/diálise) e janela ótima de intervenção.          | Core Domain     |
| **Orquestração de Rastreamento**            | Agenda janelas de rastreio, solicita exames (creatinina, EAS, UACR), acompanha pendências e resultados.            | Core Domain     |
| **Gestão de Casos & Plano de Cuidado**      | Registra plano individual (metas, medicação, retornos), consolida alertas clínicos para equipe.                    | Supporting      |
| **Integração com Laboratórios/EHR**         | Ingestão de exames e eventos clínicos (FHIR/HL7), conciliação de identidades.                                      | Supporting      |
| **Cadastro, Consentimento & LGPD**          | Identificação de paciente/profissional, consentimento para uso de dados e trilhas de auditoria.                    | Supporting      |
| **Notificações & Engajamento**              | Lembretes de coleta/consulta, nudges de adesão (SMS, app, WhatsApp, e-mail).                                       | Supporting      |
| **Relatórios Epidemiológicos & KPIs**       | Métricas de cobertura de rastreio, tempo até diagnóstico, redução de estágio tardio, custos evitados.              | Supporting      |
| **Terminologias & Catálogo Clínico**        | Mapas de códigos (LOINC, CID-10, SIGTAP), faixas de referência, regras de interpretação.                           | Generic         |
| **Pagamentos/Autorização de Procedimentos** | Integração com regras locais (ex.: SIGTAP/SUS) para autorizar/registrar exames.                                    | Generic         |
| **Identidade & Acesso**                     | Autenticação/autorização, perfis e RBAC.                                                                           | Generic         |
| **Observabilidade & Plataforma**            | Logs, métricas, tracing, feature flags, CI/CD/MLops.                                                               | Generic         |

---

## 4. Desenho dos Bounded Contexts

| **Bounded Context**                | **Responsabilidade**                                                                                       | **Subdomínios Relacionados**                  |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| **Contexto de Risco e Triagem**    | Centraliza a lógica de cálculo de risco de DRC e prioriza pacientes que precisam de investigação precoce.  | Gestão de Risco & Triagem                     |
| **Contexto Preditivo**             | Treina, valida e serve modelos de progressão da DRC, apoiando decisões de encaminhamento antecipado.       | Modelos Preditivos de Progressão              |
| **Contexto de Rastreamento**       | Agenda e controla a execução de exames periódicos (creatinina, EAS, UACR) e garante janelas de rastreio.   | Orquestração de Rastreamento                  |
| **Contexto de Casos Clínicos**     | Mantém o plano individual do paciente, metas de cuidado, alertas e acompanhamento clínico pela equipe.     | Gestão de Casos & Plano de Cuidado            |
| **Contexto de Integração Clínica** | Concilia dados de diferentes laboratórios e EHRs, normaliza terminologias (LOINC, CID) e garante fluxo.   | Integração com Laboratórios/EHR, Terminologias|
| **Contexto de Cadastro & LGPD**    | Administra identidade de pacientes/profissionais, consentimentos e trilhas de auditoria conforme LGPD.    | Cadastro, Consentimento & LGPD, Identidade & Acesso |
| **Contexto de Engajamento**        | Gera notificações multicanal (SMS, WhatsApp, e-mail, push) para lembrar exames, consultas e adesão.       | Notificações & Engajamento                    |
| **Contexto de Relatórios**         | Fornece dashboards e KPIs epidemiológicos para gestores e órgãos de saúde pública.                        | Relatórios Epidemiológicos & KPIs             |
| **Contexto de Plataforma**         | Suporte técnico, observabilidade, CI/CD, logs, monitoramento e infraestrutura compartilhada.               | Observabilidade & Plataforma                  |
| **Contexto de Procedimentos SUS**  | Integra com fluxos de autorização e cobrança de procedimentos (ex.: SIGTAP, SIA/SUS).                      | Pagamentos/Autorização de Procedimentos       |

---

## 5. Comunicação entre os Bounded Contexts

| **De (Origem)**                 | **Para (Destino)**          | **Forma de Comunicação**    | **Exemplo de Evento/Chamada**                                       |
| -------------------------------- | --------------------------- | --------------------------- | ------------------------------------------------------------------- |
| Contexto de Risco e Triagem      | Contexto de Rastreamento    | Mensageria (Evento)         | Evento: `PacienteIdentificadoAltoRisco` para disparar solicitação de exames |
| Contexto de Rastreamento         | Contexto de Casos Clínicos  | Mensageria (Evento)         | Evento: `ExameRealizado` atualiza plano de cuidado                   |
| Contexto Preditivo               | Contexto de Casos Clínicos  | API (síncrono)              | Chamada: `GET /score-risco/{pacienteId}` para incluir score no plano |
| Contexto de Casos Clínicos       | Contexto de Engajamento     | Mensageria (Evento)         | Evento: `AlertaClinicoGerado` dispara lembrete ao paciente           |
| Contexto de Integração Clínica   | Contexto de Rastreamento    | API (síncrono)              | Chamada: `GET /exames/{pacienteId}` retorna resultados normalizados  |
| Contexto de Integração Clínica   | Contexto Preditivo          | Batch/Mensageria            | Envio periódico de features derivadas de exames para reprocessamento |
| Contexto de Cadastro & LGPD      | Todos os demais             | API (síncrono)              | Chamada: `GET /paciente/{id}` com garantias de consentimento e LGPD  |
| Contexto de Relatórios           | Contexto de Plataforma      | API / Pipeline ETL          | Chamada: consulta de logs, métricas e eventos para geração de KPIs   |
| Contexto de Procedimentos SUS    | Contexto de Rastreamento    | API (síncrono)              | Chamada: validação de autorização de exame junto ao SUS              |

- **Eventos assíncronos (Kafka/RabbitMQ)** garantem baixo acoplamento entre os bounded contexts principais (ex.: risco → rastreamento → casos).
- **APIs síncronas (REST/GraphQL)** são usadas quando há necessidade de resposta imediata (ex.: consultar score de risco, buscar cadastro).
- **Batch/Pipelines** apoiam a alimentação de modelos preditivos e relatórios epidemiológicos.

---

## 6. Definição da Linguagem Ubíqua

| Termo                         | Descrição                                                                           |
| ----------------------------- | ----------------------------------------------------------------------------------- |
| **Paciente**                  | Pessoa sob cuidado/monitoramento no programa de DRC.                                |
| **População-Alvo**            | Conjunto elegível para rastreio (ex.: DM2, HAS, >60 anos).                          |
| **Risco DRC**                 | Probabilidade estimada de ter DRC ou progredir de estágio.                          |
| **TFG/eGFR**                  | Estimativa da taxa de filtração glomerular (ex.: CKD-EPI).                          |
| **Albuminúria (UACR)**        | Relação albumina/creatinina urinária para estratificar risco.                       |
| **Janela de Rastreamento**    | Período recomendado para coletar exames de triagem/controle.                        |
| **Alerta Clínico**            | Sinal gerado por regra/modelo (ex.: “queda rápida de TFG”).                         |
| **Caso Ativo**                | Paciente com plano de cuidado aberto e ações pendentes.                             |
| **Plano de Cuidado**          | Conjunto de metas, exames, consultas e intervenções.                                |
| **Encaminhamento Nefro**      | Direcionamento para especialista com critérios clínicos.                            |
| **Adesão**                    | Cumprimento de exames, consultas e recomendações.                                   |
| **Evento-Sentinela**          | Evento crítico (ex.: início de diálise, internação por IRA).                        |
| **Regras de Estratificação**  | Lógica clínica (KDIGO) para classificar risco por TFG/UACR.                         |
| **Interoperabilidade (FHIR)** | Troca de dados via recursos FHIR (Patient, Observation, Encounter, ServiceRequest). |
| **KPIs de Programa**          | Cobertura de rastreio, tempo até diagnóstico, % estágio avançado evitado.           |


---

## 7. Estratégia de Desenvolvimento

| Subdomínio                              | Estratégia                                       | Ferramentas ou Serviços (se aplicável)                               |
| --------------------------------------- | ------------------------------------------------ | -------------------------------------------------------------------- |
| **Gestão de Risco & Triagem (Core)**    | Desenvolvimento interno; regras clínicas + priorização; versionamento de regras. | Motor de regras (custom), feature store (Feast), persistência relacional. |
| **Modelos Preditivos de Progressão (Core)** | Desenvolvimento interno com MLOps; explicabilidade e monitoramento de drift. | MLflow, Kubeflow/SageMaker/Vertex, SHAP, feature store, batch + online serving. |
| **Orquestração de Rastreamento (Core)** | Desenvolvimento interno; orquestrador de tarefas, SLA de janelas, reconciliação de exames. | Orquestrador (Temporal/Camunda), filas (Kafka/RabbitMQ). |
| **Gestão de Casos & Plano de Cuidado (Supporting)** | Interno com co-criação com equipes clínicas; UX centrada no fluxo do cuidado. | App web (React), backend (Spring Boot/.NET), GraphQL/REST. |
| **Integração com Laboratórios/EHR (Supporting)** | Interno ou parceria; conectores padronizados FHIR/HL7; reprocessamento idempotente. | HAPI FHIR Server, Mirth/NextGen Connect, ETL (Airbyte/Dbt). |
| **Cadastro, Consentimento & LGPD (Supporting)** | Interno com padrões de privacidade por design; auditoria e trilhas. | Keycloak/Auth0, vault de segredos (HashiCorp Vault), audit logs. |
| **Notificações & Engajamento (Supporting)** | Parcialmente terceirizado; multicanal com template e reenvio. | Twilio/Infobip/FCM/WhatsApp Business API. |
| **Relatórios Epidemiológicos & KPIs (Supporting)** | Composição: lake/warehouse + BI; camadas semânticas e métricas validadas. | Data Lake/Warehouse (BigQuery/Redshift), Metabase/Superset/Power BI. |
| **Terminologias & Catálogo Clínico (Generic)** | Adotar de mercado; manter mapeamentos atualizados. | LOINC, CID-10, SIGTAP; dicionário via FHIR `CodeSystem/ValueSet`. |
| **Pagamentos/Autorização de Procedimentos (Generic)** | Usar serviços existentes; integração com fluxos SUS/privados. | Integração SIGTAP/SUS local, conectores/ETL. |
| **Identidade & Acesso (Generic)**       | SaaS/IdP com RBAC/ABAC e MFA.                    | Keycloak/Auth0/Okta. |
| **Observabilidade & Plataforma (Generic)** | Padrões de plataforma; IaC; SRE.                 | Prometheus/Grafana, OpenTelemetry, ELK/CloudWatch, Terraform/ArgoCD. |

## 8. Diagrama Visual (Opcional, mas Recomendado)
Desenhe um diagrama que mostre:
- Os bounded contexts.
- Como eles se comunicam.
- A relação com os subdomínios.

Use ferramentas como **Miro**, **Lucidchart** ou mesmo papel e caneta para criar seu diagrama e adicionar ao projeto.

---

## Dicas para Apresentação
- Explique cada parte do design, focando no **Core Domain** (o coração do negócio).
- Justifique por que certos subdomínios foram classificados como Supporting ou Generic.
- Destaque como a comunicação entre bounded contexts foi pensada para ser escalável.

---

Boa sorte com a dinâmica! 🚀
