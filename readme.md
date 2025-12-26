# Terraform WorkFlow: CI/CD para Infraestrutura AWS

Este repositório contém uma pipeline de CI/CD reutilizável baseada em GitHub Actions para automação de provisionamento de infraestrutura utilizando Terraform. A solução foca em segurança, utilizando OpenID Connect (OIDC) para autenticação na AWS, eliminando a necessidade de gerenciar AWS Access Keys de longa duração.
🚀 Visão Geral

A pipeline foi desenhada para suportar múltiplos ambientes (Multi-Environment) de forma modular. Ela gerencia o estado do Terraform de maneira isolada através de Workspaces e suporta ações de provisionamento (plan/apply) e destruição controlada.

## 🛠️ Tecnologias Utilizadas

•  Terraform (v1.8.3): Provisionamento de infraestrutura.

•  GitHub Actions: Automação de CI/CD.

•  AWS CLI: Configuração de credenciais via OIDC.

•  S3 & DynamoDB: Armazenamento de Statefile e State Locking.

•  JQ: Manipulação de arquivos JSON para configurações de destruição.

## 📂 Estrutura de Arquivos Esperada

Para que a pipeline funcione corretamente, o repositório deve seguir a seguinte estrutura de diretórios:

```
├── .github/
│   └── workflows/
│       ├── terraform.yml       # Workflow Reutilizável
│       ├── dev-deploy.yml      # Trigger para Desenvolvimento
│       └── prod-deploy.yml     # Trigger para Produção
├── infra/
│   ├── main.tf
│   ├── variables.tf
│   ├── destroy_config.json     # Flag de controle para destruição
│   └── envs/
│       ├── dev/
│       │   └── terraform.tfvars
│       └── prod/
│           └── terraform.tfvars
└── README.md
```
## ⚙️ Funcionamento da Pipeline

1. Autenticação Segura (OIDC)

A pipeline utiliza a permissions: id-token: write. Isso permite que o GitHub Actions solicite um token temporário à AWS, baseado em uma Role IAM pré-configurada, aumentando drasticamente a segurança.
2. Controle de Destruição

A pipeline possui uma lógica inteligente para evitar destruições acidentais. Ela lê o arquivo infra/destroy_config.json:

 •   Se o valor para o ambiente for "true", a pipeline executa o terraform destroy.

 •   Se for "false" ou outro valor, ela segue o fluxo padrão de plan e apply.

Exemplo do destroy_config.json:

```
JSON

{
  "dev": "false",
  "prod": "false"
}

```
# 3. Workspaces e Isolamento

A pipeline automaticamente seleciona ou cria um Terraform Workspace baseado no nome do ambiente (ex: dev ou prod), garantindo que os estados de diferentes ambientes nunca se sobreponham no S3.

***

# 🚀 Como Utilizar

Gatilhos (Triggers)

 •   Ambiente de DEV: Disparado automaticamente ao realizar um push na branch develop.

 •   Ambiente de PROD: Disparado automaticamente ao realizar um push na branch main.

Configuração de Inputs

Ao chamar o workflow reutilizável, os seguintes parâmetros são necessários:

```
 Input,Descrição

environment,"Nome do ambiente (ex: dev, prod)"

aws-assume-role-arn,ARN da Role IAM configurada com OIDC

aws-region,Região da AWS (ex: us-east-1)

aws-statefile-s3-bucket,Nome do bucket S3 para o Remote State

aws-lock-dynamodb-table,Tabela DynamoDB para o State Lock

```

📝 Notas de Implementação

•   O Backend é configurado dinamicamente durante o terraform init via argumentos -backend-config.

•   O arquivo de variáveis (.tfvars) deve estar localizado obrigatoriamente em ./infra/envs/${environment}/terraform.tfvars.
