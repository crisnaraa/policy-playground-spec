# Requirements Questions

## Question 1 of 10
O documento menciona que o MVP usa 'estudos atuais' como fonte de verdade do cálculo. Como esses estudos são expostos/acessados hoje? Qual é a interface de acesso?
1. APIs REST internas já existentes
2. Jobs/scripts SQL/Python executados sob demanda
3. Arquivos/datasets em S3 ou data lake
4. Banco de dados relacional acessível diretamente (RDS/Redshift)
5. Ainda não há interface — precisamos criar
X) Other — please describe
[Answer]: Jobs/scripts SQL/Python executados sob demanda

## Question 2 of 10
Como as regras/políticas de crédito são representadas hoje? O agente precisa lê-las de algum lugar.
1. Documentos textuais (Word, Confluence, Notion)
2. Código-fonte em repositório Git (SQL, Python, YAML, JSON)
3. Sistema de decisão proprietário com API (ex: FICO, Drools, Blaze)
4. Planilhas Excel/Google Sheets
5. Combinação de fontes — precisamos unificar
X) Other — please describe
[Answer]: As politicas de créditos são representadas a partir de versionamento no git.

## Question 3 of 10
Qual modelo de linguagem (LLM) deve ser usado pelo agente de IA do Policy Playground?
1. Amazon Bedrock (Claude, Titan ou outros modelos via Bedrock)
2. Amazon Bedrock com Agents (Bedrock Agents nativos)
3. Modelo open-source self-hosted no AWS (EC2/ECS)
4. A definir — quero receber uma recomendação
X) Other — please describe
[Answer]: A definir — quero receber uma recomendação

## Question 4 of 10
Como o especialista de políticas deve acessar o Policy Playground? Qual é a interface principal?
1. Aplicação web (browser) — SPA como React/Next.js
2. Integração dentro de uma ferramenta existente (Jira, Confluence, Slack)
3. Aplicação desktop interna
4. Interface de linha de comando (CLI) para usuários técnicos
X) Other — please describe
[Answer]: Precisa ser acessado por uma interface amigável.

## Question 5 of 10
Qual é o modelo de autenticação e autorização esperado para o Policy Playground?
1. SSO corporativo (SAML/OIDC) integrado ao AWS Cognito ou IAM Identity Center
2. Login com conta corporativa (Active Directory / Azure AD via federação)
3. Autenticação própria com usuário e senha gerenciados internamente
4. Sem autenticação no MVP — ambiente interno confiável
X) Other — please describe
[Answer]: Login com conta corporativa (Active Directory / Azure AD via federação)

## Question 6 of 10
Os dados usados nos experimentos contêm dados pessoais de clientes (CPF, renda, etc.)? Como tratar isso no MVP?
1. Sim — usar dados reais, com controles de LGPD (mascaramento, acesso por perfil)
2. Sim — usar dados anonimizados/sintéticos gerados a partir dos reais
3. Não — os experimentos usam apenas dados agregados e métricas, sem PII
4. A definir — precisamos mapear isso antes de lançar
X) Other — please describe
[Answer]: Sim — usar dados anonimizados/sintéticos gerados a partir dos reais

## Question 7 of 10
Qual é a estratégia de deployment/infraestrutura AWS preferida para o backend do Policy Playground?
1. Serverless: AWS Lambda + API Gateway
2. Containers: Amazon ECS (Fargate) ou EKS
3. Instâncias: EC2 com Auto Scaling
4. AWS App Runner (containers gerenciados, sem ops)
5. A definir — quero recomendação baseada nos requisitos
X) Other — please describe
[Answer]: Containers: Amazon ECS (Fargate) ou EKS

## Question 8 of 10
Como os resultados de experimentos e findings devem ser persistidos/armazenados?
1. Amazon DynamoDB (NoSQL — sessões, findings, hipóteses)
2. Amazon RDS PostgreSQL (relacional — histórico estruturado)
3. Amazon S3 + Athena (dados brutos de experimentos + queries ad hoc)
4. Combinação: DynamoDB para estado da sessão + S3 para evidências
5. A definir
X) Other — please describe
[Answer]: Amazon DynamoDB (NoSQL — sessões, findings, hipóteses) e athena para consulta dos resultados

## Question 9 of 10
O Policy Playground precisa de rastreabilidade/auditoria completa das decisões. Como esse log deve ser armazenado?
1. Amazon DynamoDB com TTL para registros antigos
2. Amazon RDS com tabela de auditoria imutável
3. Amazon S3 como data lake de auditoria (append-only, Parquet/JSON)
4. AWS CloudTrail + CloudWatch Logs para eventos de sistema
5. Combinação de S3 (evidências) + banco relacional (metadados de decisão)
X) Other — please describe
[Answer]: Amazon S3 como data lake de auditoria (append-only, Parquet/JSON)

## Question 10 of 10
Qual é o volume esperado de usuários e sessões simultâneas no MVP?
1. Pequeno: 5–20 especialistas, uso esporádico (< 10 sessões/dia)
2. Médio: 20–100 usuários, uso regular (10–100 sessões/dia)
3. Grande: 100+ usuários, uso intenso (100+ sessões/dia)
4. Não sei — quero que a arquitetura suporte escala futura
X) Other — please describe
[Answer]: Não sei — quero que a arquitetura suporte escala futura
